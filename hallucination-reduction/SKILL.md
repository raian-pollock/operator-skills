---
name: hallucination-reduction
description: Apply validated pipeline techniques (few-shot anchoring, temperature dial, Chain-of-Verification critic, targeted repair) to reduce LLM hallucination on factual content. Auto-trigger when generating citations, primary sources, statistics, named entities, or any output with verifiable claims.
last_verified: 2026-04-15
freshness_sources:
  - Internal A/B test on a primary-source generation pipeline (2026-04-15)
  - Meta AI 2023 Chain-of-Verification paper
---

# Hallucination Reduction — Pipeline Techniques

Apply the right combination of techniques per content type. See `TEMPLATES.md` for ready-to-paste few-shot suffixes and CoVe rubric templates. See `REFERENCE.md` for the empirical basis (measured wins, failure modes, research citations).

This skill is **referential**: rules + a content-type decision matrix + templates. No rigid step sequence — pick the techniques that match your content type and apply them inside whatever pipeline you're already running.

## When this skill applies

Auto-trigger when about to generate any of:
- Citations, DOIs, bibliographies, academic references
- Primary-source documents (history, science, legal artifacts) — letters, decrees, transcripts, manifests, ledgers
- Statistics, named researchers, quoted experts, dated events
- Code that calls specific library APIs, version-pinned packages, or named functions you have not just read
- Synthetic data generation that has to look real (fake-but-plausible records)

Do NOT trigger for:
- Pure analysis or reasoning over content you already have
- Translations (over-anchoring with few-shot reduces fluency)
- Marketing copy / brainstorming / creative writing (temperature dial is the only knob; few-shot kills voice)
- Code where the test suite is the critic (let the suite do its job; do not add CoVe)

## The four techniques (validated 2026-04-15)

| Technique | Cost | When | Typical lift |
|---|---|---|---|
| 1. Few-shot anchoring (3+ diverse archetypes) | One-time prompt cost | First defense for any factual content | +15-40% structural fidelity (biggest gain 0 to 1) |
| 2. Temperature dial (0.3-0.4 vs 0.7) | Free | Anything where structure matters more than diversity | Moderate; pairs with #1 |
| 3. Chain-of-Verification critic | ~$0.006/call (Flash-tier) | High-stakes claims you cannot verify against ground truth | Catches what slips past #1+#2 |
| 4. Targeted repair (rewrite only flagged units) | ~$0.012/call | After CoVe flags ≥1 unit | Cheaper than full regen, schema-preserving |

**Validated stack (internal A/B test, primary-source generation task, Gemini 3 Flash):**
- Baseline (Flash, temp 0.7, no few-shot): 1/3 test cases pass, 93% primary-source rate
- +Few-shot (3 archetypes) + temp 0.4: **3/3 pass, 100% PSR, +5% cost** — clears quality floor
- +CoVe critic + targeted repair: 2/3 pass (one regression — see "When CoVe hurts" below), 100% PSR
- Net win: techniques #1+#2 substituted for an 11.6× more expensive model swap.

## Content-type decision matrix

| Content type | Apply | Notes |
|---|---|---|
| Academic citations / bibliographies | 1 + 2 + 3 | CoVe rubric: each citation has author + year + journal + DOI? Cross-check via CrossRef before publish. |
| Primary-source artifacts (letters, decrees, ledgers) | 1 + 2 + 3 + 4 | Few-shot with 3+ archetypes diverse enough to cover the artifact range. Temp 0.4. CoVe per artifact. |
| Statistics in body copy | 1 + 3 | Few-shot with correct citations of real statistics. CoVe asks: is this number from a real source? |
| Named experts / quoted researchers | 3 only | Few-shot is risky (creates a template the LLM fills with plausible names). CoVe asks: is this a real person who said this? |
| Code touching real APIs / libraries | 2 only | Use temp 0.3. Skip CoVe — the test suite is the real critic. Do not add LLM-judging-LLM here. |
| Translations | 2 only | Temp 0.4 reduces drift. Few-shot kills target-language fluency. CoVe is overkill (let humans review). |
| Brainstorming / creative copy | None | Hallucination is the point. Do not constrain. |
| Plan / spec documents | 3 only | CoVe asks: does each claim trace to a stated requirement? No few-shot anchoring (homogenizes plans). |

## Hard rules

- **Diversity beats quantity in few-shot examples.** 3 different archetypes (e.g., letter / table / decree) beats 5 examples of the same type. Same-type examples teach the LLM to copy form, not range.
- **Pair every correct example with its wrong-form counterpart.** "Correct: <real artifact>. NOT: \`*A description of what such an artifact would look like.*\`" The contrast is the lesson.
- **End few-shot with 3-5 named rules.** Numbered, declarative, anchored to the examples. The LLM ignores rules without examples; rules WITH examples become enforceable.
- **CoVe critic must use a different prompt path than the generator.** Same prompt + same model is just rolling the dice again. Use a critic-specific system prompt, lower temperature (0.1), and structured JSON output.
- **Targeted repair MUST schema-lock fields the upstream system pins.** This is the bug that broke one pipeline variant in an internal A/B test (2026-04-15): the LLM-returned value for a schema-pinned field overrode the original, breaking a required structural constraint. When merging repaired output back, accept content fields but lock structural fields to the original values.
- **Never let the same model judge its own output without a different rubric.** Use a critic-specific rubric that decomposes "is this good" into 3-5 yes/no questions. Open-ended self-grading is worthless.
- **CoVe is not a citation verifier.** It catches structural and voice problems but cannot tell you whether "Smith, 2019, Nature 567:234" is a real paper. For citations, pair CoVe with a corpus check (CrossRef, Semantic Scholar, OpenAlex) before publishing.

## Anti-patterns (3 worst mistakes)

1. **"More few-shot examples = better."** Wrong. 5 examples of the same archetype overfits the LLM to one form. Diversity matters more than count. An internal baseline prompt already shipped one example of a single archetype; adding 3 more diverse archetypes (letter, table, decree) was what moved the needle, not adding a 5th example of the same type.
2. **"CoVe critic + repair is always a win."** Wrong. One pipeline variant in an internal A/B test made one output WORSE because the targeted-repair merge overwrote a schema-locked field. CoVe only earns its keep when you have schema-lock in the merge AND the content being verified actually has hallucination risk that few-shot+temp didn't catch.
3. **"Lower temperature always reduces hallucinations."** Wrong for creative or translation tasks — drops below 0.3 collapse fluency. The validated win was at 0.4, not 0.1. Temperature dial is a structure-vs-diversity tradeoff, not a quality knob.

## Tools

There's no ready-made CLI shipped with this skill — the pattern is simple enough to reimplement in whatever stack you already have:

**Chain-of-Verification critic, as a standalone step:**
- Input: the generated text/file + a rubric type (see `TEMPLATES.md` for rubric JSON schemas — `primary-source`, `citation`, `statistics`, `code-api`, or a custom one).
- Call a cheap model (Flash-tier or equivalent) with a critic-specific system prompt, temperature 0.1, and the rubric's yes/no questions as structured output.
- Output: structured JSON with per-unit pass/fail + reasons. The critic does not modify the input — it only returns findings. Pair with a separate repair step if you want fixes applied.
- Cost: roughly $0.006 per call at Flash-tier pricing (~10k tokens in, ~500 out) — verify current provider pricing before budgeting at scale.

See `TEMPLATES.md` for the rubric JSON schema and ready-to-paste few-shot suffixes per content type.

## Known failure modes

Logged so future runs can avoid the same traps:

- **Schema-lock bug in targeted repair (2026-04-15).** A repair merge step accepted the LLM's value for a schema-pinned field, breaking a required category-count distribution rule. Fix: always merge content fields but lock schema-pinned fields to the original.
- **Same-archetype few-shot kept the model in one archetype** for early generations in a primary-source-artifact task. Adding diverse archetypes (letter, table, decree) was the actual fix.
- **CoVe critic occasionally false-positives** on legitimate primary-source artifacts that include scholarly footnotes. Bias is on the safe side (over-flag rather than under-flag), but watch for it.

## Updating this skill

Bump `last_verified` and add a row to the validated-stack table whenever new evidence comes in. The empirical basis is what makes this skill load-bearing — generic "best practices" without measured wins erode the rules over time.

---
Part of operator-skills, the skills layer from an AI-native company that ran on 70+ agents. raianpollock.com
