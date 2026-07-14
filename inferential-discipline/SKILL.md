---
name: inferential-discipline
description: >-
  Default-on statistical and inferential reasoning hygiene for strategic plans, candidate rankings, demand assessments, audit interpretations, ROI estimates, hypothesis framing, and any quantitative claim. AUTO-TRIGGER mandatory on any task touching demand/market/ROI/quality claims, candidate ranking, prioritization, "why X first" arguments, A/B reasoning, causal claims, statistical operations on collected data, hypothesis-shaped reasoning ("if X then Y"), or any percentage/probability language. Pairs with a sample-sizing discipline (n-vs-CI math) and a factual-citation-accuracy discipline; this skill covers the broader epistemic hygiene of how strategic conclusions are drawn from evidence. The 8 checks: evidence-level tagging, named-bias check, hypothesis framing with falsifier, numerical literacy (mean/median/distribution/outliers/Simpson's), causal-reasoning discipline, uncertainty quantification (P50+P90, CI), common-failures self-review (anecdote, conjunction, ad hoc rescue, moving goalposts), and the inferential-check one-liner. PLUS a research-specific addendum: competing-hypotheses tournament + adversarial dive + ACH adjudication when the claim is drawn from agent-mediated research subagent dives. SKIP on mechanical/build/single-file work where no quantitative or strategic claims are made.
---

# Inferential Discipline

Default agent behavior. The user shouldn't have to call out confirmation bias, survivorship bias, sunk-cost anchoring, sample-size laxity, base-rate neglect, or causal-vs-correlational confusion. The agent catches these BEFORE drafting strategic conclusions or numerical claims.

## When to apply (auto-trigger)

Fire on FIRST occurrence per session, stay loaded after, on any task that involves:

- Strategic plans, roadmaps, candidate rankings, prioritization, "why X first" arguments
- Demand assessment, market sizing, ROI estimates, conversion projections, growth claims
- Quality claims drawn from samples (audit pass-rate, eval scores, pilot readings)
- Causal claims ("X drove Y", "this lifts conversion", "users prefer X because Y")
- Probability/risk language (likely, probable, low/medium/high) AND any percentage
- Statistical operations on collected data (means, rates, deltas, "the average user")
- Hypothesis-shaped reasoning ("if we ship X, then Y")
- A/B test design or interpretation
- Pilot design or interpretation

**Skip on:** mechanical/build/code-quality tasks where no quantitative or strategic claims are made. Bug fixes with clear repro steps. Single-file refactors. Typo fixes.

## The 8 checks

### 1. Evidence-level tag every load-bearing claim

Every demand / market / feasibility / quality / ROI claim gets a tag in square brackets. From weakest to strongest:

| Tag | Meaning |
|---|---|
| `[untested]` | Intuition or analogy, no data gathered |
| `[existing-rank-only]` | Internal data conditioned on what you already shipped (search-console impressions on your own pages, analytics on your own content). **Heavily biased toward queries/features you already built for.** |
| `[tam-ceiling]` | Top-line TAM number. Sets upper bound, not addressable demand |
| `[search-volume]` | Third-party keyword data showing total volume across the market, not just your slice |
| `[stated-preference]` | Survey/interview data where users say they would use/pay for X. Inflates real demand 1.5-3× |
| `[revealed-preference]` | Behavioral signal (test-page conversion, cold-outreach reply, prior similar-feature adoption) |
| `[shipped-cohort]` | Real product data on a similar shipped feature/cohort |
| `[multi-method]` | 3+ of the above pointing in the same direction |

**Hard rule:** No commit may rest on `[untested]`, `[existing-rank-only]`, or `[tam-ceiling]` alone. At minimum `[search-volume]` + ONE behavioral signal.

### 2. Named-bias check before any strategic claim

State which of these biases plausibly threaten the claim AND how you've defended:

- **Confirmation bias on internal-data sources** (your own search-console/analytics data → only shows what you built).
- **Survivorship bias on benchmarks/case studies** (citing only successful programs; what about failed ones with the same tactics?).
- **Sunk-cost anchoring on existing infrastructure** ("we already have X → X is the right next bet" — sunk costs lower attempt cost; they don't pick the highest-ROI move).
- **Self-selection bias on pilot populations** (friendly cohorts ≠ skeptical-user generalization).
- **Candidate-independence assumption** (treating multiple candidates as if outcomes don't share bandwidth, brand, or user-journey effects).
- **Base-rate neglect** (most features don't move primary metrics; cite a reference base rate when one exists).

One or two sentences in any strategic plan. Explicit, not implicit.

### 3. Hypothesis framing — alternatives + falsifier + pre-registered prediction

For any "if we ship X, then Y" claim, in the same paragraph:

- **Alternatives:** what's at least one other plausible explanation if Y happens? What if Y doesn't happen?
- **Falsifier:** what observation would convince you the claim is wrong? If nothing would, the claim is vibes.
- **Pre-registered prediction:** state expected magnitude + timeframe BEFORE shipping, so post-hoc "we knew it" rationalization is harder.

### 4. Numerical literacy on collected data

When working with numbers from any source:

- **Mean vs median.** Don't report a mean on heavy-tailed distributions. Power-laws (revenue per user, time on page, transactions per customer) often need median + p90 + p99.
- **Distribution shape awareness.** Eyeball the distribution before reducing to a summary statistic. Variance + skew + outliers matter.
- **Outlier handling.** Decide BEFORE looking at the data: include all / winsorize / exclude. Don't pick post-hoc.
- **Regression to the mean.** Extreme readings tend to moderate; don't over-credit "interventions" applied right after extreme baselines.
- **Composition effects (Simpson's paradox).** Aggregate trends can reverse within segments. Always slice by major segments before claiming the aggregate.
- **Normalization choices.** Per-user vs per-session vs per-day-active gives different stories; state the denominator.

### 5. Causal reasoning

- **Correlation ≠ causation.** State the causal mechanism explicitly. If you can't, say "correlated, mechanism unclear."
- **Confounders.** Name at least one alternative variable that could produce the same correlation.
- **Selection effects.** Which users / cohorts / pages are NOT in your data, and could that change the conclusion?
- **Reverse causation.** Could Y be causing X instead of the reverse?
- **A/B as gold standard for causal claims** when feasible. Otherwise label "associational, not causal."

### 6. Uncertainty quantification

When you state a number, state its uncertainty in the same breath:

- Estimates → P50 AND P90 (not just the mean).
- Sample-derived rates → CI width at the actual N, not just the point estimate. Common at p=0.5: n=15 ≈ ±26pp; n=25 ≈ ±20pp; n=50 ≈ ±14pp; n=100 ≈ ±10pp; n=200 ≈ ±7pp.
- Forecasts / projections → state assumptions; show how the answer changes if a key assumption shifts ±50%.
- "Order of magnitude" claims → say so; don't pretend to two significant figures when the input is one.

### 7. Common reasoning failures to catch in self-review

- **Anecdote-as-evidence** (one user said X → "users want X"). N=1 is not a population.
- **Conjunction fallacy** (P(A and B) treated as ≥ P(A); always P(A and B) ≤ min(P(A), P(B))). Compound plans with multiple required successes have lower joint probability than any individual step suggests.
- **Ad hoc rescue** (data contradicts hypothesis → adding caveats to save it; if it can survive any data, it predicts nothing).
- **Moving the goalposts** (success criterion shifts after results land).
- **Appeals to "everyone knows"** without source or base rate.
- **Apophenia** (finding patterns in N=2 or N=3 data points).
- **Substitution** (asked a hard question, answered an easier one).

### 8. Inferential check one-liner (visible to user)

When presenting a strategic conclusion or numerical claim, append one line:

> **Inferential check:** load-bearing claims tagged at evidence level [X]; biases checked: [confirmation / survivorship / sunk-cost-anchoring / self-selection / candidate-independence / base-rate neglect — pick relevant]; unstated assumptions: [list]; one-line falsifier: ___.

This makes the discipline visible AND auditable. The user sees the reasoning before approving.

## When the claim comes from agent-mediated research

A specialization of checks 2, 3, and 5 above. Agent-driven research has a structural confirmation-bias failure mode: a subagent given a hypothesis to "test" almost always returns a credible pile of supporting evidence regardless of truth. Five additional requirements when the conclusion was generated by one or more research subagent dives:

1. **Name 3-5 competing hypotheses up front** (mutually incompatible), not one hypothesis tested in isolation. Evidence consistent with H is nearly meaningless unless you've also checked whether it's consistent with H', H'', H'''.

2. **Run a deliberate adversarial dive.** A subagent whose only job is to marshal the strongest disconfirming evidence and source the smartest critics. One-sided contrary framing, no balancing. Pair with a one-sided pro dive; they cancel.

3. **Pre-register falsifiers per hypothesis before looking.** "If finding X appears, H1 is dead." Lock in writing so the goalposts can't move in synthesis.

4. **Adjudicate via ACH** (Analysis of Competing Hypotheses, Heuer, CIA, 1999). Evidence × hypothesis matrix. Cells C / I / A. Pick the hypothesis with the **fewest I cells**, not the most C. Identify cells that *discriminate* and spend the next research budget there, not on stacking more confirmation on the leader.

5. **Report calibrated confidence with prior + update.** "Started at 60%, evidence moved me to 75%, would revise below 50% if [X]." Do not declare "confirmed."

**Prompting convention for research subagents:**
- Do NOT open with "the user hypothesizes...". Use neutral framing.
- Explicitly require sourcing the strongest critics + contrarian voices.
- Require listing "what would contradict each finding" alongside each finding.
- For adversarial dives, mandate one-sided framing in the opposite direction with no balancing.

**Triggers:** any strategic / market / GTM / pricing / segmentation / prioritization conclusion drawn from research subagent dives. **Skip:** mechanical lookups, fact retrieval, well-established technical references.

Origin: a 4-subagent same-direction research dive on a product-market-fit hypothesis returned a credible pile of confirming evidence. Reframing the same research question in the opposite direction would very likely have returned an equally credible pile pointing the other way. That's a methodology failure, not a discovery — the fix is durable scaffolding, not a one-off reminder.

### Self-derived numbers count too

The competing-hypotheses + adversarial discipline above is about claims from web research. The SAME verification rigor applies to numbers YOU derived from local data:

- Use a rigorous parser (Python `csv` / `json` / pandas), not awk / grep / sed, for structured data. Awk on CSV with quoted commas silently miscounts because `"a,b"` shifts every downstream column.
- Cite source FRESHNESS: live API or database query vs dated snapshot. Stale snapshots get an explicit caveat ("as of <date>, may have updated").
- Cross-check load-bearing counts via one independent extraction (different tool or angle) before publishing.
- For shareable artifacts, re-derive load-bearing numbers from a fresh pull immediately before finalizing the doc.
- When the user pushes back on a number ("are you sure that's all the data?"), do not defend, re-check.

Origin: a claim in a strategic memo rested on a stale, days-old static snapshot showing n=6 records; a live database pull the same day showed n=16. A related count in the same memo was an awk artifact from a CSV with quoted commas (the embedded comma shifted every downstream column); re-parsing properly in Python changed the count from 17 to 14. Both errors were caught only by the reviewer pushing back, not by internal verification — the lesson became a standing rule, not a one-off fix.

**Subagent confidence labels need orchestrator spot-checks.** When a subagent labels output HIGH-confidence, sample 2-3 instances and verify against ground truth before propagating. Subagents have fresh context and local scoring; their confidence is not calibrated to project stakes. The verify-self-derived discipline applies AS STRONGLY to subagent-derived findings as to your own. Origin: an enrichment subagent labeled roughly 1 in 7 cohort members HIGH-confidence; the findings were propagated to a summary without spot-check. Manual review found the underlying record-matching itself was wrong (people matched on loose keyword/topic overlap rather than verified identity).

**Name the selection chain before extrapolating from a non-random sample.** When you cite an N-of-X finding from any cohort, explicitly list the selection layers between that cohort and the population you're inferring about. Each layer reduces the weight. Default rule: small-N sample + one selection bias = hypothesis-shaped evidence; two or more selection biases = hypothesis-shaped only, should not load-bear on posterior shifts > 1-2pp. Origin: a finding from a single design-partner cohort (one site, opt-in recruited, roughly half of an already-narrow group responding) was cited as evidence a core strategy was "weakening," moving a posterior estimate several points. Four selection layers separated that cohort from the general population being reasoned about — corrected treatment capped the shift at roughly 1 point and reframed it as "consistent with a hypothesis," not "disconfirms a hypothesis."

**Identity-resolution requires identity-resolution tools, not search engines.** "Find person X" tasks (LinkedIn, contact details, decision-maker mapping) need purpose-built identity APIs (Proxycurl, RocketReach, Lix, Hunter, Surfe), NOT generic search (Google, Exa.ai, WebSearch). Search-engine-based identity inference must include a last-mile name-overlap check (reject if surname does not match or first-name character-overlap is below ~60%). Common-name professionals and low-web-presence populations have especially high false-match rates. Firecrawl explicitly does not support LinkedIn. Origin: a cohort-enrichment pass using search-engine-based identity inference (instead of a purpose-built identity API) produced roughly 70% false-positive matches at its medium-confidence tier.

## Pairs with

- **A sample-sizing discipline** for measurement tasks (how many participants/records before a rate estimate is meaningful) — this skill covers the broader reasoning-quality half of the same problem.
- **A factual-citation-accuracy discipline** for citations, named entities, and other atomic factual claims — this skill is one level up: how strategic conclusions get drawn FROM verified facts.
- **A "think before acting" opener** for non-trivial tasks — that opener should carry a one-line inferential-check trigger; this skill is the deep version invoked when that check surfaces a real risk.
- **A render-and-check iteration loop** for anything you can visually verify — same underlying discipline: default agent behavior, fires automatically rather than waiting to be asked.

## Origin

A strategic roadmap cited internal search-console data (a low-double-digit click count on a query the team already ranked for) as "real demand signal" for a prioritization decision. Push-back surfaced that this only measures queries already authored for — confirmation bias. A self-audit of the same document then surfaced 9 related errors, all variations on the same handful of failure modes:

1. A second demand claim reused the same existing-rank-only data as if it were independent evidence.
2. "We already built infrastructure for X" was cited as strategic justification for building more on X (sunk-cost anchoring).
3. A benchmarks section cited only successful comparable programs, no failures with the same tactics (survivorship bias).
4. A pilot readout (n=25) was presented with no confidence-interval math (n=25 ≈ ±20pp).
5. A second pilot readout (n=5) was presented with no CI math either (n=5 ≈ ±44pp).
6. A partner cohort was used as a pilot population without flagging self-selection bias.
7. A risk matrix used low/medium/high labels with no stated base rates.
8. Phasing estimates carried no planning-fallacy disclaimer or kill criteria.
9. A multi-candidate ranking treated every candidate's outcome as independent, when in reality they shared distribution channels, brand, and user journeys.

Encoded as default agent behavior so the same family of errors doesn't recur. The scope deliberately expanded beyond demand/ROI to broader statistical and inferential hygiene: numbers, hypotheses, and reasoning under uncertainty generally.

## Sources

- Kahneman — *Thinking, Fast and Slow* (2011) — anchoring, base-rate neglect, conjunction fallacy, substitution, regression to mean.
- Klein — *The Power of Premortem* (HBR 2007) — pre-mortem as default cognitive tool.
- Popper — *The Logic of Scientific Discovery* — falsifiability requirement on hypotheses.
- Pearl — *The Book of Why* (2018) — causal-reasoning discipline.
- Rosenbaum — *Observational Studies* (Springer 2002) — selection effects, confounders.
- Tetlock & Gardner — *Superforecasting* (2015) — base-rate awareness, calibration discipline, Bayesian updating in practice.
- Wagenmakers et al. — pre-registration discipline (Nature 2018) — alternative-hypothesis discipline before data lands.
- Anthropic — *Building Effective Agents* — evals-first verification (the same discipline applied at the agent-output layer).

---
Part of operator-skills, the skills layer from an AI-native company that ran on 70+ agents. raianpollock.com
