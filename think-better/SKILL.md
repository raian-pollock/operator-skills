---
name: think-better
description: >-
  Structured 5-line opener for non-trivial tasks. Forces goal + decision criterion, approach in 2-3 bullets, one-line premortem, named verifier, and a routing decision (which branch / worktree the work lands on) before any tool calls. Evidence-aligned (Plan-and-Solve, Klein premortem, Anthropic evals-first); explicitly NOT a manufacturing-framework cargo cult. AUTO-TRIGGER on any of the following — do not wait for the user to invoke: a task expected to take 2+ tool calls; a decision under uncertainty (which model, which approach, whether to ship); a research, audit, or investigation task; cross-doc or cross-file edits where blast radius isn't obvious; the user asks "what do you think", "how should we", or "what's the best way". Do NOT fire on single-file mechanical edits (rename, format, typo, one-line fix); bug fixes with clear repro steps from the user; tasks already inside a project's own planning-mode process (that process should own its own Definition of Done and premortem); QA runs (a project's own QA process covers verification). The opener is user-visible — write it as the agent's first text output, before tool calls.
---

# think-better

Before tool calls, write six lines (seven for strategic/quantitative tasks):

1. **Goal.** One sentence + decision criterion: "Done when ___." Tag task type when non-obvious: research / build / debug / plan / audit / polish / ship / discussion.
2. **Approach.** 2–3 bullets, in order. Before writing the bullets, scan whatever scaffolding you actually have available: (a) any installed skills whose description matches the task's *domain*, not just its code path (e.g., analytics reasoning → an analytics/BI skill, model selection → a model-eval skill, multi-locale UI work → an i18n skill, a batch job → a batch-processing skill, deploy staging → a push/release-prep skill). If a skill matches, name it in your first bullet ("using the `<skill>` skill") and load + follow its protocol. If no skill matches, say "no matching skill" so a gap is visible. Then broaden the inventory across the other scaffolding substrates you maintain: (b) any load-bearing project rules that bind this turn (a definition-of-done bar, a quality/north-star principle, an inferential-discipline rule, a reliability-first principle — whatever your project's standing rules are called); (c) any persistent notes/memory index you keep, searched by trigger phrase. Cite the top 1–3 of each substrate in your first or second bullet ("scaffolding bound: <rule>, <rule>, <note>"). Do this *before* the first tool call — waiting until the second or third turn is a known failure mode (scaffolding gets bound retroactively, after the mistake it would have prevented).
3. **Scope.** One line: "In scope: X, Y. Out of scope: Z. Assumption I have not yet verified: W." Required when blast radius is non-obvious (multi-file, multi-system, multi-locale, cross-functional, plan affects 3+ surfaces). The "Assumption" half forces externalizing the load-bearing premise *before* spending tokens on it; many failures are silent premises that 60 seconds of `grep` would have falsified.
4. **Premortem.** One line: "If this comes back wrong, most likely reason: ___."
5. **Verifier.** One line: "I'll know it worked because ___."
6. **Routes to.** One line: "Work lands on `<branch>` (or `<new worktree off branch X>`); prereq check: <yes/no/N/A>." Skip when the task doesn't write to disk (pure research, planning, conversation).
7. **Inferential check** (required for any task touching demand/market/ROI/quality claims, candidate ranking, prioritization, A/B reasoning, causal claims, statistical operations, hypothesis framing). One line: "Load-bearing claims tagged at evidence level [X]; biases I'm guarding against: [confirmation / survivorship / sunk-cost-anchoring / self-selection / candidate-independence / base-rate neglect — pick the relevant ones]; one-line falsifier: ___." Skip on mechanical / build / code-quality tasks where no quantitative or strategic claims are made.

Then execute. No further ceremony.

## Why these (and only these)

| Element | Evidence | Why kept |
|---|---|---|
| Goal + criterion | Husain/Yan, *What We Learned from a Year of Building with LLMs* (2024): "good writing is good thinking" | Forces externalization; catches misframings before token spend |
| Plan-then-execute | Plan-and-Solve Prompting (Wang et al., ACL 2023): +5–8% over raw CoT, survives budget-aware evaluation (EMNLP 2024) | Most evidence-backed structural move for LLM agents |
| Premortem | Klein (2007), *The Power of Premortem* (HBR); reinforced by Kahneman | High-leverage cognitive move that survives translation to LLMs; ~10 tokens |
| Verifier | Anthropic *Building Effective Agents* (evals-first); CorrectBench (arXiv 2510.16062): reflexion is −11% without external grounding | Self-critique is theater without external signal |
| Routes to | Origin: two same-day worktree-stomp incidents — a branch shifted mid-session leaving finished work committed to the wrong branch, and a parallel `git stash` wiped uncommitted source files in a sibling session | Forces explicit base-branch + isolation decision *before* code touches disk; ~10 tokens; cheap insurance against shared-checkout race conditions |
| Scope (Assumption check) | Origin: repeated silent-premise failures across separate sessions — an unstated assumption stated once early in a task ("this only affects the internal tool", "this population is out of scope", "this edge case doesn't apply here") silently propagated through many tool calls before being caught, each time by a human, never by the agent itself | Externalizes the load-bearing premise so a 60-second grep can falsify it before token spend; required when blast radius is non-obvious |
| Inferential check | Origin: a strategic-planning task where cited "demand evidence" was later found, on self-audit, to carry nine distinct errors — survivorship bias on the comparison set, sunk-cost anchoring, self-selection in the pilot group, sample-size laxity, base-rate neglect, and failures of candidate independence. Kahneman *Thinking, Fast and Slow*; Klein on naturalistic decision-making; falsifiability per Popper. | Catches the strategic-reasoning failure class that structural premortem + verifier miss: claims that *look* rigorous because they cite numbers, but the numbers are biased / mis-tagged / single-method. Required only on strategic/quantitative tasks; skipped on mechanical work. |

## Explicitly cut (and why)

- **PDCA / DMAIC / 5 Whys / Genchi Genbutsu / Hansei labels.** Vocabulary cargo cult — the original mechanism (statistical loops over weeks of physical processes) doesn't transfer to a single LLM task.
- **CIA Structured Analytic Techniques (ACH, Key Assumptions Check).** Designed for 6–8h human analytic sessions. RAND's own assessment is mixed. Only premortem survived translation.
- **Cynefin domain classification.** Weak evidence even in its native domain.
- **OODA labels.** Vocabulary borrowing; the loop-fast intuition is already how a competent coding agent operates.
- **Forced "think step by step" preamble.** Wharton GAIL 2025: on reasoning models the gain is negligible and may hurt; already implicit in extended/chain-of-thought reasoning modes.
- **Self-consistency / multi-agent debate / Tree-of-Thoughts / generic reflexion loops.** High cost; often negative under budget (EMNLP 2024 budget-aware eval; CorrectBench).

## Plan-mode addendum: world-class approach research

If you run any kind of formal planning mode (a distinct planning phase before implementation) for a task that spans more than ~2 hours of agent time or touches an architecture choice, the plan should include a "World-class research" section with four parts:

1. **References consulted** (2–5 named sources: org + post / repo / talk / RFC). Spend 5–10 min on how well-known engineering teams (pick ones relevant to your domain — e.g. Anthropic, Stripe, Vercel, Linear, Figma, Notion, GitHub, Cloudflare, Supabase) have published on this exact problem, and on who has solved it at much bigger scale than you.
2. **Load-bearing idea(s)** — the 1–3 principles from that research that would still be right at your scale.
3. **Explicitly rejected** — 2–3 specific things from that research you are *not* adopting, named with the scale / context / budget mismatch reason (e.g. "their eval harness assumes a dedicated infra team and a research org; we have neither; cut their model-card pipeline and adopt only the eval-runner shape").
4. **Adapted shape** — how the load-bearing idea lands in your codebase, files, and budget.

Do not take world-class prior art at face value. Translate critically: those teams often have far larger budgets and different constraints. The discipline is extracting the load-bearing idea and cutting the scale-dependent ceremony. Even one paragraph of "X's eval-platform writeup + Y's repair-pipeline RFC informed this" puts the plan on stronger footing than fully original thinking from training data alone.

**Skip only when** the task is mechanical (typo, rename), industry-consensus on the solution (e.g., "use SHA-256"), or genuinely has no external analog (project-specific glue). Even then, declare "no relevant world-class precedent" in the plan — the explicit declaration is the discipline, not the research itself.

This is separate from a general "don't reinvent the wheel" check (scanning for existing OSS/SaaS/internal helpers before building bespoke) — that's tech-tier. World-class research here is approach-tier: design patterns, architecture choices, sequencing.

## Composition with existing scaffolding

- **Formal planning mode:** if your project already has a planning process that requires a cost estimate, a definition of done, an approach-level premortem, behavioral tests, and now world-class research (above), this opener can be skipped once that planning mode is entered — no double-tax. The world-class-research requirement still applies to the plan body itself.
- **Project QA/verification steps:** run at task end. This opener runs at task start. They bracket a task without overlapping.
- **Cost/spend guardrails:** if you already gate metered batch operations on a separate check, don't duplicate it here.

## Portable form (for runtimes without a skills system)

The six/seven-line opener above is specific to how Claude Code loads skills. The underlying discipline is not — see the accompanying `think-better-portable.md` in this package for a copy-pasteable version for ChatGPT custom instructions, an `AGENTS.md`, or any other system-prompt-driven agent runtime.

## Sources

- Anthropic — [Building Effective Agents](https://www.anthropic.com/research/building-effective-agents)
- Anthropic — [Effective Context Engineering for AI Agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- Wharton GAIL — [Decreasing Value of Chain of Thought](https://gail.wharton.upenn.edu/research-and-insights/tech-report-chain-of-thought/) (Meincke, Mollick et al., 2025)
- Wang et al. — [Plan-and-Solve Prompting](https://arxiv.org/abs/2305.04091) (ACL 2023)
- [Budget-Aware Evaluation of LLM Reasoning Strategies](https://aclanthology.org/2024.emnlp-main.1112.pdf) (EMNLP 2024)
- [CorrectBench: Can LLMs Correct Themselves?](https://arxiv.org/html/2510.16062v1) (2025)
- Klein — [The Power of Premortem](https://hbr.org/2007/09/performing-a-project-premortem) (HBR, 2007)
- Husain, Yan et al. — [What We Learned from a Year of Building with LLMs](https://applied-llms.org/) (2024)

---
Part of operator-skills, the skills layer from an AI-native company that ran on 70+ agents. raianpollock.com
