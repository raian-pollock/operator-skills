---
name: model-pick
description: >-
  Lightweight model-selection research that fires BEFORE any code change that adds or changes an external-LLM call. Always considers (1) current frontier per Arena.ai ELO + Artificial Analysis Intelligence Index, (2) new releases in the last 30 days (mandatory candidates), (3) upcoming releases in the next 4 weeks via web search (mandatory), (4) channels + service tiers per surface, (5) recent vendor pricing moves. Runs in 5-10 minutes. A full production bake-off (a larger eval harness with a scoring rubric run against real production data) is a separate, heavier exercise and only warranted for large user-facing batches — see "Escalating to a full bake-off" below. AUTO-TRIGGER on any of: mentioning using or calling an external LLM (Gemini, OpenAI, OpenRouter, Qwen, DeepSeek, direct Anthropic API, Vertex AI, etc.); "let's use <model>" / "let's call <vendor>" / "via OpenRouter" / "via the Gemini API"; "which model should I use" / "is there a better model" / "should we wait for <upcoming model>"; about to write new code that introduces an external-LLM API call (a `model: '...'` string in an OpenRouter call, a `generativelanguage.googleapis.com` URL, `openai.chat.completions.create`, etc.); a cost-preflight step detects a new metered-API code path; a new or recent model release is mentioned; renewing a periodic batch job that calls an external LLM (revisit every 30 days).
---

# Model Pick — Lightweight Selection Before External LLM Use

This skill ensures every external-LLM choice is informed by **current frontier intelligence**, not by whatever was best when the agent's training data ended.

**Why it exists:** an agent's training data is a snapshot. The LLM landscape ships monthly — new models, new prices, new leaderboards. Without a fresh selection step, the agent silently defaults to whatever was SOTA at training cutoff and never notices a cheaper or better option shipped since. Origin: a model-selection task defaulted to three models the agent "knew" from training data, missing two vendor releases from the prior week — one with a 1M-token context window and reasoning at roughly half the price of the incumbent premium pick, the other a same-day cheap/fast release from a different vendor. Both were sitting in a live model registry the whole time; nobody queried it.

**Relationship to a full bake-off:** this skill is the lightweight, always-on step (5–10 minutes, research only). A full production bake-off — scoring candidates against a rubric with real production data, typically warranted only for batches ≥50 items that are user-facing with a high quality bar — is heavier and situational. If your toolkit has a dedicated eval-harness skill, this is the selection step that feeds it; if not, the "Escalating to a full bake-off" section below tells you when to build one ad hoc.

## The 8-step lightweight flow (5–10 min budget)

### Step 1: Surface lens (1 sentence)

Classify the surface using one of four lenses — **Quality / Cost / Speed / Capability** — and write down which is primary and which is secondary, plus any hard floor:

> "Primary lens: Quality. Secondary: Cost. Quality regression unacceptable on factual accuracy, structure compliance. Cost ceiling: $X/run. Speed: batch/async-tolerant."

### Step 1.5: Task-type classification + fit-for-purpose benchmarks (MANDATORY)

**A model that wins on average isn't the right pick for your specific task.** Before identifying candidates, classify the task type and look up which external benchmarks are most predictive for it. Specialists outperform generalists when the task aligns with their training emphasis.

**Canonical task-type → benchmark map:**

| Task type | Primary benchmarks (look up scores per candidate) | Where to find |
|-----------|---------------------------------------------------|---------------|
| **Code completion / autocomplete** | HumanEval, MBPP, BigCodeBench | Papers With Code, HF leaderboards |
| **Agentic coding (multi-file, tool-use)** | SWE-Bench Verified, SWE-Bench Pro, DeepSWE, Terminal-Bench 2.0, Aider Polyglot | swebench.com, aider.chat, vendor leaderboards |
| **Math / hard reasoning** | GPQA Diamond, MATH-500, AIME, HMMT, MMLU-Pro | HF, vendor cards |
| **Creative writing** | EQ-Bench Creative-Writing, LMSYS Arena Creative-Writing subcategory | eqbench.com, lmarena.ai |
| **Instruction-following / structured output** | IFEval, BFCL (function-calling), MT-Bench | HF |
| **Long context (>100K)** | RULER, LongBench, Needle-in-a-Haystack | HF, vendor cards |
| **Multimodal / vision** | MMMU, MathVista, AI2D, ChartQA | HF |
| **Translation / multilingual** | FLORES, WMT24, MultilingualMMLU, XCOPA | HF |
| **Factuality / hallucination** | TruthfulQA, HaluEval, FreshQA, SimpleQA | HF |
| **Tool use / agentic** | BFCL, Tau-Bench, ToolBench, AppWorld | gorilla.cs.berkeley.edu |
| **General chat / open-ended Q&A** | LMSYS Arena Overall ELO, MT-Bench, MMLU | lmarena.ai |
| **Domain-specific / pedagogical / professional text** | No widely-adopted public benchmark — build a small custom rubric. Adjacent public signals: MMLU (knowledge), GPQA (depth), IFEval (structured-output adherence), TruthfulQA (factuality) | Custom rubric + adjacent benchmarks |
| **Customer support / empathetic chat** | EQ-Bench, MT-Bench, your own CSAT-style eval | eqbench.com |

**Web-search for emerging benchmarks (MANDATORY)**: at every model-pick, also run:
```
"<task type> LLM benchmark 2026 new leaderboard"
"<task type> evals 2026"
```
The benchmark landscape shifts as fast as the models do. Anything in the last six months matters; task-specific leaderboards for niches like science QA or retrieval-grounding are still new and easy to miss.

**Anti-patterns to resist:**
- Defaulting to "MMLU is the benchmark" for tasks where MMLU has nothing to do with the work (e.g., picking a coder by MMLU score).
- Trusting only Arena ELO when the task is in a niche the crowd-vote underweights (long-context, multimodal, niche languages).
- Using benchmarks the candidate has plausibly been heavily trained on ("benchmaxing") — cross-reference with Arena ELO to catch this.

**Output**: record the task classification + relevant benchmarks in your pick log (see Step 7).

### Step 1.6: Locale-scope classification (MANDATORY for any task touching localized output)

Classify the task's locale-scope before picking candidates. Models that win on average can still regress on minority locales — and that regression compounds across volume when the deployment is multi-locale.

| Scope | When | Per-locale N (in a subsequent eval) | Per-locale slicing in the pick verdict |
|-------|------|----------------------------------|-------------------------------------|
| **Single-locale** | English-only internal classifier; single-language marketing copy | N=20+ on the single locale | N/A |
| **Few-locale (≤3)** | Support replies in two or three languages; transcreation into one target language | N=10+ per locale | Mandatory |
| **Many-locale (4+)** | Content generation, product copy, or support serving many languages/markets at once | N=5+ per minority locale, N=10+ per primary locale, ≥30 total | Mandatory + per-locale-drop flag rule |

**Per-locale drop flag rule (many-locale tasks):** any model showing **>3pp drop from its best-locale score on any minority locale** is flagged as a multilingual-regression risk. This overrides an "average wins" verdict in favor of language-consistent performers. Cost ceiling rises proportionally — paying several times sticker for consistent quality across every locale you serve is often worth it.

**Non-Western models with possible Anglo-centric or non-English-centric training emphasis** (worth extra scrutiny per locale): Qwen (Chinese-centric), DeepSeek (Chinese/code-centric), Llama (mostly English). Even when their composite intelligence index is high, their performance on Romance, Germanic, Nordic, or South-Asian languages may lag. **Always slice per locale; never extrapolate from English.**

**Before running a many-locale eval, pre-register your own locale list** (the exact set of languages/markets your product serves) and make sure fixture coverage includes all of them before declaring the eval statistically valid.

**Anti-pattern this prevents:** in one many-locale eval, all candidate models scored within a point of each other on the overall average, and the naive read was "ship the cheapest." Slicing by locale revealed the cheapest model was the worst scorer on two specific minority languages — a 4–6pp gap hidden entirely by the blended average. The "free" choice would have shipped a real regression on those markets. It was caught only by someone asking to see the per-locale breakdown, not by the aggregate score.

### Step 2: Refresh your model registry

Before picking candidates, refresh a live view of what's actually available and what it costs. At minimum:

- Pull current model + pricing inventory from OpenRouter's public models endpoint (`GET https://openrouter.ai/api/v1/models`) or directly from each vendor's model-listing page/API.
- Pull current Arena.ai ELO and Artificial Analysis Intelligence Index scores for your candidates.
- Cache the result locally (e.g., a `registry/models.json` file) so Step 3's mechanical query has something fresh to run against.

```bash
# Minimal viable refresh — adapt to whatever registry/cache shape your project uses.
curl -s https://openrouter.ai/api/v1/models -o registry/models.json
```

Required before candidate identification. Skipping this step is exactly how stale training-data defaults creep back in.

### Step 3: Identify 3–5 candidates across tiers (LIVE-QUERY the registry, NEVER from memory)

**The agent's training-data mental model of "available models" is always stale.** Don't pick candidates from memory. Mechanically query the refreshed registry.

**Mandatory mechanical pattern (run this, don't paraphrase from memory):**

```bash
# After Step 2 (registry refresh), select the latest 3 per vendor family + any <30d new releases.
python3 - <<'PY'
import json
reg = json.load(open('registry/models.json'))  # adjust path to wherever you cached the registry
models = reg if isinstance(reg, list) else reg.get('models', [])
FAMILIES = ['deepseek/', 'qwen/', 'google/gemini', 'x-ai/grok', 'anthropic/', 'openai/', 'mistralai/', 'meta-llama/']
for f in FAMILIES:
    matches = sorted([m for m in models if m['id'].startswith(f)], key=lambda x: x.get('created', 0) or 0, reverse=True)[:3]
    for m in matches:
        p = m.get('pricing', {})
        pin, pout = p.get('inputPer1M', '?'), p.get('outputPer1M', '?')
        print(f'  {m["id"]:<55} ${pin}/${pout}')
PY
```

Then pick from that output (the live registry), not from memory. Required tier coverage:
- **Baseline** — whatever is currently used for this surface (or an analogous surface). Check your own pick-log history or the production code path.
- **At least one ultra-cheap** (≤$0.50/M out) — latest of: Gemini Flash family, DeepSeek Flash family, Qwen Flash family, Mistral small family.
- **At least one mid-range** ($0.50–3/M out) — latest of: Claude Sonnet, DeepSeek V*-Pro, mid-tier reasoning models.
- **At least one premium / SOTA** ($3–15/M out) — latest of: Claude Opus, GPT-5 family, Gemini Pro, Qwen Max.
- **Any new release in the last 30 days** — automatic candidate if it slots into a relevant tier. Sort by `created` descending in the registry.

**The "select latest by family" pattern is non-negotiable.** It's mechanical, deterministic, and immune to training-cutoff drift. Origin: an agent picked five candidates entirely from memory and missed two newer releases from a vendor whose family it had already shortlisted — both were sitting in the same registry, under the same vendor prefix, and would have surfaced automatically with this pattern.

### Step 4: Cross-reference benchmarks

For each candidate, log:
- **Arena.ai ELO** (crowdsourced, hardest to benchmark-game)
- **Artificial Analysis Intelligence Index** (synthetic composite)
- **Per-task benchmark** if relevant (e.g., MMLU for factual, HumanEval for code, SWE-Bench for agentic — per the Step 1.5 map)

**Flag benchmaxing risk:** any model with high benchmark scores but low Arena ELO. Cross-check before betting on it.

### Step 5: Upcoming-release + vendor-pricing-announcement research (MANDATORY — never skip)

The agent's training cutoff doesn't see what's coming, or what just got cheaper. Always run two web-search passes before picking.

**Pass A — Upcoming releases (next 4 weeks):**
```
"<top vendor> next model release date 2026"
"<vendor> roadmap 2026"
"<top candidate's vendor> next release rumor"
```

Identify any model **rumored or confirmed within the next 4 weeks**. For each, note:
- Estimated release window (week, month)
- Confidence (vendor-confirmed / leaked / pure rumor — anything under roughly 30% market probability, ignore)
- Expected differential vs. current frontier (what specifically would it improve)

**Wait-vs-now decision rule:**
- Wait IF: (a) an upcoming release is rumored within 4 weeks with vendor confirmation or ≥40% market probability, AND (b) the current frontier shows real quality headroom on the task (pilot N=10 if uncertain), AND (c) nothing else is blocked by deferring.
- Otherwise: ship with current frontier; re-evaluate when the upcoming release lands.

**Pass B — Vendor pricing announcements (last 30 days):**
```
"<vendor> pricing discount 2026"
"AI API price cut 2026"
"<vendor> price announcement permanent discount"
"<vendor> batch API free tier expansion 2026"
```

Vendors compete on price as hard as on quality. Permanent discounts, free-tier expansions, batch-API rate changes, and region-specific pricing routinely shift a candidate's cost tier by 30–75% — and a registry snapshot from even a week ago may not have caught it yet.

**Re-pricing trigger rule:** any vendor announcement that changes a candidate's cost tier by more than 25% in the last 30 days requires immediate re-pricing in the verdict and, if material, a re-ranking of candidates.

**Origin:** a vendor made a large permanent price cut on one of its models days before a scheduled eval ran. The registry refresh picked up the new price automatically, but nothing separately flagged the announcement — it was caught only because someone asked "did anything change recently?" The same vendor shipped a new free tier the same week. Running Pass B on a schedule (not just relying on the registry to quietly reflect it) closes that gap.

### Step 6: Channel + service-tier enumeration

For each shortlisted candidate, enumerate channels:

| Channel | Marginal cost | Notes |
|---------|---------------|-------|
| Direct vendor API (pay-as-you-go) | Sticker rate | Default |
| OpenRouter passthrough | Sticker + small markup | Convenient cross-vendor |
| Consumer subscription (e.g., a flat-rate chat-app plan) | Flat-rate | Usually no API quota — verify per vendor |
| BYOK / credit pool (cloud-provider credits, vendor startup credits) | Sticker, paid from pool | Bounded by balance |
| Subprocess flat-rate (a CLI tool billed under a flat subscription rather than metered API) | $0 marginal | Quota window |

Then service tiers:

| Tier | Multiplier | Use case |
|------|-----------|----------|
| Flex / batch | ~0.5× | Batch, async, latency-tolerant |
| Default | 1× | Standard |
| Priority | ~1.8× | Latency-critical, interactive |

Pick the cheapest channel × tier combination that meets the surface's lens requirements from Step 1.

**MANDATORY for BYOK / credit-pool / flat-rate channels:** always compute and report the **gross-cost-equivalent** (sticker rate × actual token consumption × project volume) in addition to the $0-marginal headline. "Free via BYOK" is misleading when the credit pool is finite. Required outputs per candidate using a BYOK or flat-rate channel:

1. **Marginal $** (often $0 for BYOK).
2. **Gross-cost-equivalent** at sticker rate (what the call would cost if billed pay-as-you-go).
3. **Credit-pool depletion estimate** as a percentage of your known pool size (verify actual pool size with your cloud billing console if uncertain — don't guess).
4. **Flag if gross cost exceeds the pool** — that means the BYOK channel runs out and silently spills to paid billing mid-batch.

**Anti-pattern this prevents:** a candidate model showed `testCost=$0` in a batch job because it was billed through BYOK reporting. Its real gross cost at sticker rate for the full run: roughly $0.024/call × 270,000 calls ≈ $6,477 — about 3.2× the size of a typical $2K startup cloud-credit pool. Left unchecked, that run would have quietly exhausted the credit pool partway through and spilled to metered paid billing. It was caught only because someone asked "how much does this actually deduct?" before running it at scale.

### Step 7: Pick + log + decide escalation

Append a structured entry to a project-local pick log (e.g., `model-pick-log.jsonl`, one JSON object per line):

```json
{
  "ts": "2026-05-27T12:34:56Z",
  "task": "Brief task description",
  "lens_primary": "Quality | Cost | Speed | Capability",
  "lens_secondary": "...",
  "quality_floor": "Regression unacceptable on: ...",
  "candidates_considered": [
    {"id": "vendor/ultra-cheap-model", "tier": "ultra-cheap", "arena_elo": null, "aa_index": null, "cost_per_M_out": 0.5},
    {"id": "vendor/premium-model", "tier": "premium", "arena_elo": null, "aa_index": 56.6, "cost_per_M_out": 7.5}
  ],
  "new_releases_last_30d": ["vendor/model-a (2026-05-19)", "vendor/model-b (2026-05-20)"],
  "rumored_upcoming_4wk": [
    {"id": "vendor/next-gen-pro", "window": "June 2026", "confidence": "vendor-confirmed"}
  ],
  "wait_decision": "no",
  "wait_rationale": "...",
  "chosen": "vendor/premium-model via OpenRouter, Default tier",
  "chosen_rationale": "...",
  "channel": "openrouter",
  "service_tier": "default",
  "escalate_to_full_eval": false,
  "escalate_rationale": "Batch size under 50, not user-facing critical"
}
```

**Then decide:** does this need a full production bake-off (see below)?
- YES if: batch ≥50 items + production user-facing + quality bar high (publishing, generation, anything scored by a downstream quality gate you don't control).
- NO otherwise — the model-pick verdict suffices.

## When to skip

**Skip ONLY** when:
- The task is "use a flat-rate subscription channel for a model tier you already have access to" — no external-vendor choice is being made, so there's nothing to pick.
- It's a throwaway one-off script: deleted after single use AND under roughly 1K total tokens. Document the skip ("no model-pick: throwaway, under 1K tokens").

**ALWAYS invoke** when:
- New code path introduces an external-LLM API call.
- New script that calls an external LLM.
- Renewing a periodic batch job (re-decide every 30 days).
- Someone asks a model-selection question.
- An upcoming release crosses the 4-week threshold (worth re-deciding even mid-batch).

## Anti-patterns (read these every time)

1. **Defaulting to whatever the training data named last.** The agent's cutoff drifts; a live registry doesn't. Always refresh the registry + check the last 30 days.
2. **Skipping upcoming-release research because "I already know what's coming."** You don't. Vendors ship weekly. Always run the web-search sweep.
3. **Picking without a channel + service-tier check.** A batch/flex tier alone is often half the cost on any latency-tolerant surface — a large structural saving left on the table.
4. **Treating Arena ELO as the only signal.** Cross-reference with a composite intelligence index + task-specific benchmark + the per-surface lens.
5. **Locking in "wait for X release" without measuring current frontier first.** It's cheaper to pilot 10 items with the current frontier than to wait blind for four weeks.
6. **Re-running model-pick redundantly within the same session for the same surface.** Once you've picked and logged for a surface, the verdict is good for about 30 days. Re-invoke when a new release lands or the surface changes materially.

## Escalating to a full bake-off

If your toolkit includes a dedicated production-eval skill or harness, this skill's Step 7 escalation criteria are the trigger for it. If it doesn't, and the escalation criteria are met (batch ≥50 items, user-facing, high quality bar), the minimum viable version is: build a small fixture set representative of production inputs (including edge cases and, if localized, every locale you serve), score each shortlisted candidate against a rubric with a stratified sample, and only then lock in the choice. Don't skip this step by rationalizing that the lightweight pick "was thorough enough" — the lightweight flow is research-only and was never meant to substitute for scored evaluation at production scale.

## Where the log lives

Keep an append-only, JSONL pick log — one line per verdict — wherever your project keeps this kind of operational record. It becomes searchable history for "what model did we pick for this kind of task last time, and why."

---
Part of operator-skills, the skills layer from an AI-native company that ran on 70+ agents. raianpollock.com
