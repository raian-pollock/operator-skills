# Targeted model selection

Use this only after a trigger in `ROUTING.md` applies. It does not run on every task or plan. A routine effort adjustment on the same approved model does not start this procedure. Choose a suitable supported effort first, and keep validated production settings and their change requirements.

## The procedure

1. **Name the gap.** State the capability or economic gap in the current route. Name the acceptance check and what a failure would cost. Keep the user's chosen model and any validated route decisions.
2. **Keep the shortlist short.** The baseline plus up to two alternatives that could close the gap. Start with the host's supported models, your own model configuration, existing adoption records and relevant task evidence. Query a vendor catalogue only when it answers a missing fact. Broader discovery is appropriate when no known candidate fits or the user asks for it. When a choice is genuinely open, list the free and stealth tier too (see `ROUTING.md`).
3. **Confirm each new candidate.** Exact model ID, channel and account eligibility, required tools, modalities and context, data handling, and supported effort levels. For metered use, fetch live pricing from the vendor's own price page or the endpoint record, and current spend or balance where available. State uncertainty when a fact is unavailable. A subscription does not imply API credit.
4. **Estimate the bounded cost before any chargeable call.** Include input, output, billed reasoning tokens, tool fees, repeats, review and likely repair. Show the maximum number of calls, token caps and the cash equivalent. Stay under the spend threshold the principal sets, or ask. When per-call cost is unknown, measure it with one approved calibration call before the batch. No calibration call is needed for a known routine subscription worker.
5. **Get task evidence.** Where adoption lacks task evidence, run a small representative pilot with a clear check. For a consequential recurring or production change, run a production evaluation with the project's required checks. Compare the existing baseline on normal inputs, difficult cases and relevant locales. Choose the sample size to answer the decision; a small N does not prove statistical parity.
6. **Record the adoption once.** In the existing route configuration or pick log: baseline, chosen ID and channel, effort, acceptance result, limitations, cost evidence and source URLs with dates. Routine reuse needs no new entry. Revisit after a material change or a failure, not because time passed.

## Evidence that matters

Public benchmarks can shortlist candidates; they do not replace task evidence. Check only the benchmarks relevant to the decision. There is no mandatory all-vendor survey, no quota of new-release candidates, no upcoming-release rumour sweep and no rule to wait for a future release. Do not delay useful work for an unconfirmed launch.

Where a benchmark does help, pick the one that matches the task type:

| Task type | Benchmarks worth checking |
|---|---|
| Agentic coding (multi-file, tool use) | SWE-Bench Verified, Terminal-Bench, Aider Polyglot |
| Code completion | HumanEval, MBPP, BigCodeBench |
| Math and hard reasoning | GPQA Diamond, AIME, MMLU-Pro |
| Instruction following, structured output | IFEval, BFCL (function calling) |
| Tool use | BFCL, Tau-Bench |
| Long context (over 100K tokens) | RULER, LongBench |
| Multimodal and vision | MMMU, ChartQA |
| Factuality | SimpleQA, TruthfulQA |
| Creative writing, empathetic chat | EQ-Bench, the Arena creative-writing category |
| General chat | LMArena overall |
| Domain-specific professional text | No good public benchmark; build a small custom rubric |

Cross-check a high benchmark score against crowd-voted results before betting on it; a model trained hard on a benchmark can score well there and poorly on your task.

**Cost per accepted result, not price per token.** For repeated production work, compare cost per accepted result with retries and repairs included. Use actual usage or billing data where available. Credit-funded API calls consume the credit pool even when the cash charged today is zero: report the gross cost at list price and what share of the pool a run will use, or a run can quietly exhaust the pool and spill into paid billing partway through. Subscription usage consumes quota; API list-price equivalents do not measure that quota.

**Localised output.** For multilingual work, inspect every locale the pipeline actually serves, including the weaker ones. Do not extrapolate English quality to other languages. A blended average can hide a real regression on a minority locale: in one many-locale evaluation, all candidates scored within a point of each other overall, and only the per-locale breakdown showed the cheapest one was the worst on two specific languages. Keep task-specific numerical and factual acceptance thresholds.

**Data terms.** For new gateway routes carrying non-public data, keep the data-collection opt-out on and pin an approved provider. Contributor and free tiers need their own terms check. If those requirements cannot be met, use an eligible route; never weaken them to reach a cheaper endpoint.

## Compact adoption record

Use the project's existing format. If there is none, the decision can be a paragraph in the existing plan with: date; task; baseline; chosen model, channel and effort; reason; acceptance evidence and its limits; bounded cost; data-use decision; verified source URLs; conditions for reconsidering. Do not build a new logging system for this.

---
Part of operator-skills, the skills layer from an AI-native company that ran on 70+ agents. raianpollock.com
