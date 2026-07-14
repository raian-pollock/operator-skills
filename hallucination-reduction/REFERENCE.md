# Reference — Empirical basis & research

The skill's rules are derived from measured wins, not generic best practices. This file documents the evidence so the rules can be challenged, extended, or rolled back if new data contradicts them.

## Validated wins

### 1. Few-shot anchoring with diverse archetypes (internal A/B test, 2026-04-15)

**Setup:** Gemini 3 Flash on a primary-source-artifact generation task. 3 test cases spanning different audience levels and contexts. Full production pipeline (system prompt + repair loop + strict schema validator, run end-to-end).

**Treatment:** Added 3 diverse primary-source examples (first-person letter, data table/ledger, decree/edict) to the system prompt suffix. Each example paired with a `NOT:` meta-description counter-example. Ended with 5 numbered rules. Combined with temperature drop from 0.7 to 0.4.

**Results:**
- Baseline (no few-shot, temp 0.7): 1 of 3 test cases passed validation; primary-source rate 93%.
- Treatment (3 archetypes + temp 0.4): 3 of 3 test cases passed; primary-source rate 100%; cost increased 5%.
- Effect size: +67 percentage points pass rate, +7 pp PSR, marginal cost overhead.

**Replication path:** Run the same A/B design against your own generator: same system prompt, same repair loop, same strict validator, comparing baseline (no few-shot, temp 0.7) against treatment (3 diverse archetypes + temp 0.4). Hold everything else constant so the intervention is isolated.

### 2. Temperature dial (paired effect from same A/B)

The 0.7-to-0.4 temperature drop is bundled with the few-shot intervention in the validated treatment, so its independent contribution is not isolated. Inference from the CoVe critic flags across pipeline variants: temperature 0.7 generations had 1 critic flag; temperature 0.4 generations had 1 critic flag (a different test case). Roughly equal — the few-shot examples appear to do most of the work, with temperature contributing modest additional fidelity at a 0% cost increase.

### 3. Chain-of-Verification critic (mixed result from same A/B)

CoVe critic correctly flagged 1 of 3 test cases (a document with a "Note" wrapper that broke character). The flag was legitimate. The targeted repair successfully rewrote the flagged document. **However, the merge step clobbered a schema-pinned field, breaking the spec's required category-count distribution rule.** Net: CoVe + repair turned a passing test case into a failing one due to a merge bug, not a CoVe weakness.

**Status:** Detection works; repair-merge needs a schema-lock fix (~2h work) before CoVe + repair earns its keep on this content type.

## Research basis

- **Chain-of-Verification (Dhuliawala et al., 2023, Meta AI).** "Chain-of-Verification Reduces Hallucination in Large Language Models." Reports +23% F1 on list-based tasks via 4-step verify-and-revise pipeline. The skill's CoVe rubrics adapt this to single-call structured JSON to keep cost and latency low.
- **Factual-nucleus sampling.** Lower temperature (0.3-0.5) reduces hallucination on fact-heavy generation at the cost of stylistic diversity. Survey: Lee et al. 2022, "Factuality Enhanced Language Models for Open-Ended Text Generation."
- **Few-shot anchoring research.** The biggest single-shot lift is 0 to 1 example; diminishing returns past 3-5; format matters more than count (Brown et al. 2020, GPT-3 paper, plus subsequent ICL research).

## Failure modes documented in literature

- **Self-consistency without rubric:** Asking the same model to "check if this is good" without a structured rubric produces low-information answers. Always decompose to yes/no questions.
- **Few-shot example contamination:** If the few-shot examples leak into the output (LLM literally copies an example), diversity is the fix, not "tell it not to copy."
- **CoVe over-flagging:** Critics tuned to be strict will over-flag legitimate edge cases. Prefer over-flagging to under-flagging in production but track false-positive rate.
- **Targeted repair bypass loops:** If the repair call sees the same prompt context as the generator, it will produce the same hallucination. Repair prompts MUST include the critic's specific feedback.

## Cost economics

| Operation | Tokens | Cost (Flash @ $0.50/$3.00 per M) |
|---|---|---|
| Generation (typical mid-length article/document) | ~10k in / ~5k out | ~$0.020 |
| Few-shot suffix overhead | +~2k in | +~$0.001 |
| CoVe critic call | ~10k in / ~500 out | ~$0.0065 |
| Targeted repair call | ~3k in / ~2k out | ~$0.0075 |
| Full stack (gen + critic + repair) | as above | ~$0.034 (+70% on baseline) |

For a mid-tier frontier model (~$3.00/$15.00 per M), multiply by ~6×. For a premium frontier model (~$15/$75 per M or higher), multiply by ~30×. Cost-effective stack: cheap-model generator (Flash-tier, Haiku-tier) + cheap-model critic (same). Reserve premium models for content where the full stack is still insufficient. Verify current provider pricing before budgeting at scale — these figures will go stale.

## When a future eval should re-run this skill's recommendations

- New model release. Rules tuned for one model family may not generalize.
- New content type discovered. Add a row to TEMPLATES.md after measuring.
- Hallucination rate creep observed in production. Re-measure before re-tuning.
- Repair-merge schema-lock fix ships. Re-run your own A/B to confirm the full stack (generate + critic + repair) now beats the cheaper few-shot + temperature-only stack.

## File map for this skill

- `SKILL.md` — main: decision matrix, hard rules, anti-patterns
- `TEMPLATES.md` — paste-ready few-shot suffixes + CoVe rubrics by content type
- `REFERENCE.md` — this file: empirical basis + research

---
Part of operator-skills, the skills layer from an AI-native company that ran on 70+ agents. raianpollock.com
