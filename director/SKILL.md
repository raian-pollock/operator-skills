---
name: director
description: >-
  Project-director protocol for any multi-step agent task: plan before executing, map dependencies in a file, decide delegate-vs-inline with the spawn test, brief sub-agents with a four-field contract, route models per job, size fan-outs, checkpoint mid-flight, and run two-layer QA at the end. Load when planning any task with 3+ steps, before spawning any sub-agent or scripted multi-agent workflow, when parallelising work, when briefing a delegate on another model or runtime, or when a recurring loop delegates part of its sweep.
---

# Director

The main session is the project director: strategy, planning, orchestration, review, QA. Execution is delegated when delegation buys something concrete; otherwise the director executes directly. A "team" here means a fan-out of ordinary sub-agents under one director, not a group of peers.

## 1. Classify before planning

Label every step **read / write / dependency** before choosing how to run it. Reads parallelise; writes serialise — ONE writer per file or surface, always. Dependency edges define the critical path and show where parallel work is safe. Deterministic work (counts, greps, file existence, renders) goes to scripts, not models.

## 2. The plan artifact

**3 or more steps, or 2 or more workers → the plan and its dependency map go in a FILE before dispatch** (the project's plans directory). A written map survives context compaction and exposes write collisions before they happen. Minimum content: the steps, an owner per step (inline / spawn / script), blocking edges, checkpoint thresholds (section 7), and the project's definition of done.

## 3. The spawn test

Delegate a step ONLY if:
- **(read-only AND context-heavy)** — the worker will chew through far more tokens than its conclusion is worth in the director's context; you keep the conclusion, not the file dumps; OR
- **(write-isolated)** — it owns its file or worktree exclusively, with no shared surface;
- AND you can name **what the fan-out buys**: context isolation, genuinely independent perspectives, or worktree-isolated edits.

Fan-out multiplies token cost many times over, and at an equal compute budget elaborate multi-agent strategies often stop beating a simple baseline (see the budget-aware evaluation cited in [`../think-better/SKILL.md`](../think-better/SKILL.md)). Wall-clock speed alone never justifies a spawn; context isolation does. Do not spawn for a single fact in a known location, for work that needs the conversation's full context, or for a sequential dependency chain.

## 4. The brief — the highest-leverage artifact

In the MAST taxonomy of multi-agent failures (Cemri et al., [*Why Do Multi-Agent LLM Systems Fail?*](https://arxiv.org/abs/2503.13657), 2025), specification issues account for 41.77% of the failures observed across 200+ traces, the largest of its three categories. Every spawn carries four fields:

1. **Objective** — the outcome, not the activity, including what "done" looks like.
2. **Output schema** — an exact output file path or structured shape. The worker's final message is the path plus counts, never a content dump: workers return short summaries, and artifacts go to files.
3. **Tools and sources** — which files, APIs and ground truth to use, plus a provenance contract for claim-bearing output (verbatim quotes; cite it or drop it).
4. **Boundaries** — what NOT to touch, what parallel workers own, read-only scope, stop conditions.

Investigation briefs ("is X broken / missing / empty?") also carry a **verify-the-shape mandate**, verbatim: *"Before reporting that any field is null, missing, or empty in an external API response, run a 5-second pre-flight to verify the actual response shape (e.g. `SELECT properties FROM events WHERE event = '<name>' LIMIT 1`). Singular-vs-plural property-name guesses are a recurring trap. Confirm property names exist before claiming they're empty."*

**Delegates on another model or runtime** never load your rules, notes or skills, so the prompt must name the binding rules and skill disciplines for THAT task. The director QAs every output (confident invention is the signature flaw of fast, economical models): code gets run or type-checked, claims get checked against the source of truth, cited APIs and paths get existence-confirmed, renders get read back. That QA is never delegated back to the delegate.

**Headless spawns:** where the CLI supports a JSON-schema output option, pass the schema, so the spawn returns validated JSON instead of prose you have to parse. Where it can forward sub-agent transcripts, turn that on when you need to QA what the spawn's own sub-agents actually did.

**Parallel builders get a contract, written by the first agent.** When 3 or more agents build parts of one thing, run a core agent first. It builds the shared parts, writes a contract file (who owns which file, the shared interface with examples, layout rules, a "never" list, the checks each part must pass), and leaves a placeholder for every other part that already passes the checks. Each builder's brief then says: read the contract first; you own ONLY these files; anything shared goes in your report under "requests for the core", which the director applies itself.

## 5. Model routing by task

Use the quick rule of thumb in the `model-pick` skill ([`../model-pick/ROUTING.md`](../model-pick/ROUTING.md)). This is a local decision, not a per-task model-research workflow. Scripts handle deterministic work. Small or context-dependent tasks stay in the current capable session. Justified workers can run on any model or runtime the project has authorised, chosen by task evidence, supported tools, data requirements and total cost.

Economical models suit bounded work with a direct check; capable models keep difficult planning and consequential judgement. A planning mode alone does not require research, workers, an evaluation or maximum reasoning. Adjust effort on the same capable model first: low for clear work, medium for ordinary planning, high for difficult judgement; maximum needs a specific reason. Use the runtime's supported controls and preserve the user's explicit setting.

Name the model explicitly when spawning where the runtime allows it; forks inherit. Never pass one provider's model ID into a tool that only runs another's, and never claim the running chat changed models without an actual supported control.

Do not delegate just to reach a cheaper model. Count briefing, context, verification, likely repair and quota pressure. Existing production pipelines keep their validated routes and acceptance checks.

## 6. Fan-out sizing

From a written rule, not vibes: **1 worker** for a lookup or verification; **2-4** for a comparison, multi-angle verification or a moderate sweep; **10+** only for genuinely broad independent research. Set a cap on concurrent background agents and subprocesses from your own machine's limits and send work in waves beyond it; fully remote batches are capped by provider quota instead. **More than 3 workers → dispatch through a script** with schema outputs: deterministic control flow, and the director reads returned schemas, never raw transcripts.

**Broad research runs in two passes.** Pass 1: several sweep agents, each on a different angle, return a fixed schema that includes "verified at the source today: yes/no", under a request budget (a cap on fetches per agent, no paid search unless approved). Merge and rank in the script. Pass 2: one agent per top result (5-10) reads it at the source: opens the site, the posting or the document. Pass 2 overrules the sweep, and the report says where it did; in one production run the second pass corrected 6 of the 10 top results. Never design, rank or recommend from sweep summaries alone.

## 7. Mid-flight checkpoints

Set thresholds at dispatch rather than relying on vigilance. Re-plan when (a) a worker fails or returns empty or malformed output, (b) a named budget line is crossed (tokens, money, wall-clock), and (c) always BEFORE the first irreversible action (a send, a push, a delete, metered spend) — that checkpoint is a hard gate. Long jobs favour resumable state files over watching.

**Quota continuity.** If your plan has usage windows or weekly caps, check before any fan-out that could plausibly exhaust the window (yes *or unsure*), and schedule a one-shot resume shortly after the next reset. Resume briefs are self-contained (task IDs, state-file paths already saved to durable storage, key numbers, ordered next steps, standing constraints); assume the context has been compacted by the time it fires. On a limit error mid-work: stop spending, save state, schedule the resume, post a short status. Use an already-authorised alternative only if it meets the same quality and data requirements; otherwise wait for the reset. Never start an unapproved metered fallback, and never silently downgrade a quality-critical stage: if cap pressure moves bulk or mechanical work to a cheaper model, say so.

**Overlap phases, carefully.** The next phase may start before the previous one has finished, once the part it depends on is in, provided its workers confirm at the source any claim they rely on. While workers run, the director builds the shared pieces and the checks, so the checks exist before the work they judge.

**Finish from existing files.** After a usage limit or crash, first run the checks on whatever is on disk. Then restart unfinished WRITING agents with a new brief: "A previous agent wrote most of these files and was cut off before verifying anything. Do not start over: read the existing files, keep what is good, complete what is missing against the brief, then verify for real." Do not blindly resume a scripted workflow whose unfinished agents write files: they restart from their original "build" brief and rebuild over the work. Start reviewing the finished parts at the same time.

## 8. Terminal QA — two layers, never self-review

- **Layer 1, deterministic:** run the suite, lint or build the work claims to pass; recompute load-bearing numbers from their source; render and read back anything visual. After changing a shared module, template or data file, re-run the checks of EVERY place that uses it, including earlier prototypes and review pages still on show (a fix to a shared module that never reaches the older pages that import it is a common silent miss).
- **Layer 2, fresh-context review:** a SEPARATE agent checks the deliverable against the plan's definition of done (self-review adds little: an agent grading its own work shares its blind spots, and self-correction without external grounding measured negative in CorrectBench, cited in [`../think-better/SKILL.md`](../think-better/SKILL.md)). External or high-stakes artifacts also get a critic anchored on real exemplars; internal artifacts skip the critic, and the director verifies facts itself and ships.
- **Critic and fixer shape (when a critic is justified).** The critic never edits. It uses the work the way the real user will (real mouse, keyboard and phone input for a site; a cold read for a report), compares it with 2-3 named references, and returns: a short verdict naming the biggest weakness; defects as {severity blocker/major/minor, where, what, a buildable fix}; and a KEEP list of what must not break. A separate fixer who owns the files reports per defect: fixed, or not fixed and why; anything outside its files is marked "needs core". The director fixes each "needs core" item or writes it into the plan's open list; none stays only in a sub-agent report. Worker "fixed" reports are claims: the director re-checks them on the real output (in one production run, two builders reported fixes that had not happened).

The director personally verifies sub-agent claims: a report states intent, not deed. Gates cite the artifact (file path, fixture ID, stratified sample, the diff); "tests pass" alone is not a gate. **Five checks, tiered** — low risk (mechanical, docs only) = 1+2 · medium (new code paths, scripts) = 1+2+3 · high (user-facing LLM content, anything other systems consume, gates) = all 5:

1. Re-run the suite it claims passed; cite your own count.
2. Read the diff (`git diff <base>...<branch>`, three dots), never just the file list.
3. Spot-check 3-5 artifacts.
4. Re-verify cited claims against ground truth (re-fetch, re-count, re-query).
5. Diff against the plan's definition of done, quoting the line and citing branch evidence.

"All gates green" → re-run them. "Idempotent" → run it twice and diff. A plan with 3 or more LLM or sub-agent steps names 4-8 self-verification gates up front. Skip only trivial, deterministic-and-test-covered, or read-only steps. Spot-checking N of M is NOT verification; the tell of a sloppy verdict is a uniform 100% same outcome on a varied set. This applies to EVERY delegate: fresh context means untrusted, whatever the model. Triage, git-state and "is X shipped?" verdicts are never delegated: the director runs `git log`, `git diff`, `git show` and `grep` itself (sub-agents may run commands in parallel, never own the verdict).

## 9. Recurring loops

Use scripts or native tools for a cycle's mechanical sweep. Delegate only when the spawn test (section 3) and the host's permissions are met; no fixed worker or cheap-model percentage applies. Keep consequential judgement with a capable model. Routing telemetry gives diagnostic hints, not proof of routing quality: short output does not mean the work was easy, and low delegation can be efficient.

---
Part of operator-skills, the skills layer from an AI-native company that ran on 70+ agents. raianpollock.com
