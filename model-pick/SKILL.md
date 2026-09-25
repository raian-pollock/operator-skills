---
name: model-pick
description: >-
  Choose the model and reasoning effort for a piece of agent work with a quick local rule of thumb, not a market survey. Adjust effort on the same capable model first; consider another model only for a capability gap, a persistent failure or a worthwhile total-cost advantage. Research only a genuinely new model or channel choice, or a relevant failure; run a production evaluation only for consequential recurring or production changes. Routine tasks, effort changes and entering a planning mode need no survey, no benchmark lookup, no eval and no log. AUTO-TRIGGER when: a task needs a model or channel the project has not adopted yet; the current route fails a check or lacks a capability; a known price, access or terms change affects a route in use; someone asks "which model should I use" or "is there a better model"; code is about to introduce or materially change a recurring or production LLM call. Do NOT trigger on a vendor mention, a new release, a calendar anniversary, or entering plan mode by themselves.
---

# model-pick: route by a quick rule, research only a real choice

Use the rule of thumb in `ROUTING.md` (in this folder). Read `REFERENCE.md` (also in this folder) only when a genuinely new selection or adoption decision is open.

**Why this replaced the old version.** An earlier version of this skill ran 5-10 minutes of fresh research before every model choice: registry refresh, benchmark lookup, upcoming-release sweep, pricing news. It addressed a real problem (agents defaulting to whatever their training data named last), but applied to every task it became overhead on routine work, where the right move is usually a small, repeated adjustment of effort on a model already in use. The current rule keeps the research where it changes a decision (new choices, failures, production changes) and drops it everywhere else.

## Routine work

Use tools for deterministic work. Adjust reasoning effort on the same capable model before considering a model switch: low for clear work, medium for ordinary planning and implementation, high for difficult judgement; maximum needs a specific reason. Use the levels the runtime actually supports and keep any explicit setting the user chose. Reassess at meaningful task boundaries, without a separate routing call. Consider another model only for a capability gap, persistent failure or a worthwhile total-cost advantage, and count handoff, review, repair and quota costs when you do.

This is a short decision inside the task. Do not invoke another model to choose a model. No benchmark search, full registry refresh, release forecast, calibration call or routing log is needed merely to start work or enter a planning mode.

Respect the runtime's controls, its delegation rules and the user's model selection. Instructions in a prompt do not switch the running chat's model or reasoning setting. A routine effort adjustment needs no picker, evaluation or log; validated production settings keep their own change requirements.

## When to do more

**Targeted selection** (read `REFERENCE.md`) when the task actually needs a model or channel the project has not adopted, the current route fails or lacks a capability, or the user asks for current model advice. Verify the missing facts for a short shortlist, and reuse valid task evidence you already have.

**Production evaluation** for a consequential recurring or production adoption or migration, a quality regression, or an explicit request for an empirical comparison. If your toolkit has a dedicated eval-harness skill, this is its trigger. If it does not, the minimum viable version is: a small fixture set representative of production inputs (edge cases included, and every locale you serve if the output is localised), the current baseline, a written acceptance rubric, and total cost per accepted result. Existing production acceptance and approval gates still apply. Ordinary plans, vendor mentions and recent releases do not trigger it on their own.

**A new inexpensive candidate** starts with a bounded, authorised pilot whose output has a direct check. A small success is provisional. Keep a capable fallback and escalate on failure without lowering the quality bar.

## Common failures to avoid

1. **Research on every task.** Apply the cached rule of thumb and investigate only a missing fact that changes the decision.
2. **Cheapest by price alone.** Include reasoning tokens, context transfer, repairs and review in the cost of an accepted result.
3. **Automatic default or highest effort.** Choose effort for the task on the same model first. Preserve explicit settings; the model default is a fallback when the right level is unclear.
4. **Forced delegation.** Scripts or finishing the work directly may be the economical choice. Never target a worker percentage.
5. **Stale or unavailable models.** Verify a new model ID and the actual access path before use; respect route configurations that were already validated.
6. **Confusing a listing with permission.** A catalogue entry does not prove account access, acceptable data-use terms or room under the spend threshold the principal sets. Check all three before a paid pilot.
7. **Overstated evidence.** A leaderboard, a short answer or a small pilot does not prove task quality or savings.

## Files in this skill

- `ROUTING.md`: the quick decision, the effort guide, the three levels of selection, runtime and quota boundaries, and how to enumerate free and stealth models when a choice is open.
- `REFERENCE.md`: the targeted-selection procedure, what evidence counts, a benchmark lookup table, and a compact adoption record.

---
Part of operator-skills, the skills layer from an AI-native company that ran on 70+ agents. raianpollock.com
