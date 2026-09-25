# Agent model routing

The canonical routing policy for coding agents that load it (Claude Code, Codex, Gemini-based agents and similar). Adjust reasoning effort on the same capable model first, using a small local rule of thumb. Routine routing needs no extra model call, web research, benchmark lookup or routing log. Load this file only when the choice needs more detail than the summary in `SKILL.md`.

## The quick decision

1. **Use a tool when the work is deterministic.** File searches, counts, formatting, validation and known transformations belong in scripts or native tools.
2. **Keep the capable model and choose effort for this task first.** Lower effort can be the useful adjustment without a new model. Use the effort guide below. A handoff can cost more tokens and time than finishing small work yourself; keep context-heavy follow-ups in the current session too.
3. **Consider a different model only when there is a reason.** A capability gap, persistent failure after suitable effort, or a worthwhile total-cost advantage can justify a switch. For substantial work, choose by difficulty and by the cost of an undetected error. Use an economical model with relevant task evidence for bounded work that has a clear check. Keep ambiguous planning, architecture, difficult debugging and consequential judgement on a capable model. An ordinary plan does not need the strongest model or the highest reasoning setting just because it is a plan.
4. **Delegate only when it buys useful context isolation or independent work.** Include briefing, context, review and likely repair in the cost. Wanting a different provider is not by itself a reason to create a worker. The host's delegation permissions and tool constraints still govern. (For the full delegation protocol, see the `director` skill: [`../director/SKILL.md`](../director/SKILL.md).)
5. **Escalate on evidence.** A failed check, a missing capability, unresolved reasoning or higher stakes can justify more capable reasoning. Allow at most one focused repair on an economical candidate before changing approach or escalating. A rate-limit error is not evidence of low quality.

Quality is the constraint. Minimise the total cost of an accepted result: input and context, reasoning and output, tool calls, retries, review, delay and subscription quota. A low price or a benchmark rank alone does not establish fitness. Never lower acceptance checks to justify a cheaper model.

## Adjust effort before changing models

Choose the lowest supported effort that suits the task while keeping the same capable model. These are starting heuristics, not measured claims of equal quality:

| Task | Starting effort on the same model |
|---|---|
| Clear, bounded work; straightforward edits or lookups | Low, where supported; use tools for deterministic operations |
| Ordinary implementation or planning with clear constraints | Medium, where supported |
| Difficult debugging, architecture, ambiguous synthesis or consequential judgement | High; extra high when the difficulty warrants it |
| The hardest unresolved problems, where more reasoning is useful | Maximum or another supported higher level, with a specific reason |

Effort names and behaviour vary by model and runtime; use the controls the runtime actually has. The model default is a fallback when the right level is unclear. Never silently override the user's explicit model or effort setting. Do not reach for the top setting automatically because of a planning mode, a model family or a long context. Some top settings spawn sub-agents and must respect delegation permissions.

Reassess at a meaningful task or stage boundary, without a separate call or log. Raise effort when reasoning stays unresolved or a check fails because of a reasoning error; missing data or tool errors need their own fix. Lower it again when the work becomes straightforward. Do not infer difficulty from answer length. Keep acceptance checks intact, and give consequential judgement enough effort from the start.

A routine effort adjustment on an approved model does not trigger model-pick research or a production evaluation. Material changes to validated recurring or production effort settings keep their own evidence and approval requirements. A cost/quality chart measured at one effort per model says nothing about the same model at a lower effort.

## Task guide

These are starting points by capability tier, not cross-vendor quality rankings. Keep any task-specific route already validated against the same requirements, and bind the tiers to concrete models in your own configuration.

| Work | Starting point | Quality check |
|---|---|---|
| Mechanical work | A native tool or script; the current session supervises | Deterministic result |
| Bounded extraction, classification, straightforward summaries or small isolated edits | An available economical model, as the task warrants | Schema, sources, tests or direct inspection |
| Narrow judgements that code branches on, at volume (classify, route, rank, verify, gate) | An economical model, or a purpose-built classifier if you have adopted one | Agreement with the current route on a labelled sample; thresholds fitted on your own data; production evaluation before a production switch |
| Ordinary implementation and planning with clear constraints | The current capable model | Project checks and acceptance criteria |
| Complex architecture, subtle debugging, synthesis, high-stakes decisions or final quality-critical review | A capable model with enough reasoning | Independent sources and the project's required domain checks |
| Repetitive work on public or synthetic data, large enough to justify a paid route | A verified inexpensive metered candidate | A bounded pilot before wider adoption; output must pass the same checks |

## Channels

For a new bounded worker, prefer a channel the project has already authorised (a native agent runtime or a flat-rate subscription) when it fits. There is no universal provider order: account quota, task evidence, context transfer and tooling all matter. A subscription still consumes shared quota, and it does not include arbitrary vendor API calls.

A metered multi-vendor gateway (OpenRouter or similar) is an optional channel for useful alternatives. Existing budget, data-use and permission rules apply. Production request-path inference keeps its approved metered channel; never move it onto a personal or developer subscription.

## Three levels of selection

| Level | Trigger | Work |
|---|---|---|
| Routine routing | A task, a follow-up, a same-model effort adjustment, entering a planning mode, a known worker, or an unchanged approved pipeline | Adjust effort first. No picker research, no eval. For a substantial plan, one sentence in the existing plan naming the model, the effort and the check is enough. |
| Targeted selection | A new model or channel is actually needed; the existing route fails or lacks a capability; a known price, access or terms change matters; the user asks for current options | Read `REFERENCE.md`. Compare the current route with one or two plausible alternatives. Verify only the missing facts. Record a durable adoption decision once. |
| Production evaluation | Introducing or materially changing a recurring or production model path, a quality regression, or an explicit request for a comparison where task evidence is needed | Representative inputs, a baseline, acceptance criteria, failure cases, relevant locale slices and total cost. Reuse valid existing evidence; a material prompt, tool or channel change may invalidate it. |

A vendor mention, a new release, a 30-day anniversary, a new file in a tools folder, or entering a planning mode does not by itself trigger research or an evaluation. Discovery may surface an option; adoption needs task evidence in proportion to the consequences. A small pilot supports a provisional route, not a claim of universal quality parity.

For unchanged recurring work, reuse its approved configuration and evidence. Reconsider when failures, availability, a meaningful price or terms change, or new task requirements justify it. Do not install a recurring model-review job as part of ordinary routing.

## Runtime and quota boundaries

A prompt policy cannot change the model or reasoning effort already running a chat. Change either only through a control the runtime actually exposes and the task authorises. A request to be concise does not change the configured reasoning budget. If the control is unavailable, finish small work in place; for substantial work, state a useful effort adjustment once when it matters, without pretending it happened or starting another task just to change it. Some runtimes' native workers can only run that runtime's own models; never pass another provider's model ID to such a tool.

Where a spawn supports choosing a model, choose a supported one explicitly when permitted. If the host requires inheritance, use it deliberately or pick a supported isolation mode; do not invent an override.

At a quota wall, use an already-authorised alternative only when it meets the task's quality and data requirements. Otherwise save the work and wait using the host's supported mechanism. Never silently downgrade consequential work, buy more usage, or start a metered fallback nobody approved.

## Enumerate the free and stealth tier when a choice is open

This is not a scheduled sweep. It runs at the moment a model choice is actually open: when this skill's targeted selection or a production evaluation runs, when a route is being chosen for a large or repeated workload, and whenever a candidate shortlist is drawn up. A routine effort adjustment on an already-chosen model does not trigger it. When it does run, list the free tier instead of pricing only the models you already know. Two commands, both free and read-only:

```sh
# every model whose prompt AND completion price is zero
curl -s https://openrouter.ai/api/v1/models | node -e "let s='';process.stdin.on('data',d=>s+=d).on('end',()=>{const j=JSON.parse(s);for(const m of j.data) if(Number(m.pricing?.prompt||1)===0&&Number(m.pricing?.completion||1)===0) console.log(m.id, m.context_length, m.name);})"

# the capabilities of one candidate before trusting it
curl -s https://openrouter.ai/api/v1/models/<id>/endpoints
```

Providers sometimes run frontier models in stealth under neutral names before launch. They are free because the provider keeps the prompts and completions. So:

- **Check `supported_parameters` before testing quality.** A free model with no `structured_outputs` and no `reasoning` cannot hold a strict JSON contract or take an effort setting. That is a capability finding, not a quality finding, and it can disqualify the route on its own.
- **Expect rate limits.** Free and stealth endpoints return 429 under modest load. Measure achievable throughput, not just per-call latency, before planning a large batch around one. A free model that cannot finish the job in time is not cheaper.
- **Data terms decide eligibility before quality does.** A stealth or data-contributing route is a candidate for public or synthetic data only. Send it input scrubbed of the principal's personal and financial data and of third-party identities, or do not send it.
- **A stealth model has no stable identity.** It can be renamed, repriced or withdrawn without notice, and the provider behind it may be undisclosed. Never make it the only route for a recurring pipeline, and record the exact dated model ID with any result.

The failure this prevents: a route comparison priced paid candidates against each other and never listed the free tier at all, missing a free, large-context stealth model that the principal then found by hand. Enumerate first, then shortlist, at the point of choosing and not on a calendar.

## New inexpensive models and data use

Before the first use of a new candidate, verify the exact model ID, account and channel access, needed capabilities, data-use terms and price. A catalogue listing does not prove account or geographic eligibility. Keep confidential or personal material on routes approved for that data. For non-public input through a multi-vendor gateway, turn on the gateway's data-collection opt-out (on OpenRouter, `provider.data_collection = "deny"`) and pin an approved provider; if no eligible endpoint remains, stop using that route rather than relaxing the filter. Some low-cost tiers are cheap because the vendor may use your prompts and outputs to improve its products: treat those as public or synthetic-data candidates only.

## Evidence and upkeep

- OpenAI, [Model selection](https://developers.openai.com/api/docs/guides/model-selection), accessed 10 September 2026: establish accuracy first, then keep it while reducing cost and latency.
- Anthropic, [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents), accessed 10 September 2026: start simple; routing helps distinct task classes when classification is reliable.
- OpenAI, [Codex models](https://developers.openai.com/codex/models), accessed 10 September 2026: use the lowest reasoning effort that produces the needed result, and raise it for deeper planning, analysis or checking. Model and effort controls depend on the runtime and account.

Keep the policy stable and update model bindings when a relevant change is verified. Transcript measurements are diagnostic hints only: output length does not establish difficulty, low delegation is not failure, and logs from one provider cannot measure efficiency across all of them. Assess accepted results, repairs and actual bill or quota data before claiming savings. No fixed delegation percentage is a quality target.

---
Part of operator-skills, the skills layer from an AI-native company that ran on 70+ agents. raianpollock.com
