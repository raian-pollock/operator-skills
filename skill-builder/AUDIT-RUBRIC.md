# Skill Audit Rubric

Score each dimension 1-5. A world-class skill scores 35+/40.

---

## Dimension 1: Structure (weight: high)

How well-organized is the skill for agent consumption?

| Score | Criteria |
|-------|----------|
| 1 | Wall of prose. No steps, no sections, no lists. |
| 2 | Has sections but uses prose paragraphs instead of steps/checkboxes. |
| 3 | Numbered steps OR headed sections, but inconsistent (some steps have checkboxes, others don't). |
| 4 | Consistent numbered steps with checkboxes for procedural, or clear headed sections with tables/lists for referential. Dividers between sections. |
| 5 | Perfect structure: every step has verifiable checkbox items, clear dividers, opening line states purpose + cross-refs. Scannable in 10 seconds. |

**How to fix low scores:** Convert prose to numbered steps. Add `- [ ]` checkboxes for each verifiable action. Add `---` dividers between steps.

---

## Dimension 2: Description Quality (weight: high)

Does the description tell Claude WHEN to load this skill?

| Score | Criteria |
|-------|----------|
| 1 | Missing, empty, or generic ("useful tool for code"). |
| 2 | States what it does but buries the use case after 30+ words. |
| 3 | Front-loads use case but missing deliverables or trigger condition. |
| 4 | Front-loaded use case + deliverables + trigger, but over 250 chars or slightly vague. |
| 5 | Under 250 chars. Primary use case in first clause. 2-3 specific deliverables. Explicit trigger condition. |

**How to fix:** Rewrite description starting with the verb: "Run...", "Build...", "Audit...", "Apply...". End with "Auto-triggers when..." or "Invoke with /name".

---

## Dimension 3: STOP Gates (weight: high)

Does the skill prevent the agent from barreling through without validation?

| Score | Criteria |
|-------|----------|
| 1 | No STOP gates anywhere. Agent runs the entire skill uninterrupted. |
| 2 | Has "check with user" type gates but no verifiable criteria. |
| 3 | 1-2 STOP gates with criteria, but missing gates before expensive/irreversible operations. |
| 4 | STOP gates at all critical decision points, each with criteria. Missing 1 that should exist. |
| 5 | STOP gates at every point where direction could change, each with explicit criteria (checkbox list or table). Final STOP before output. |

**How to fix:** Add `STOP: {present X}. {criteria to proceed}.` before: cost decisions, destructive actions, direction-setting analysis, and final output.

**Skill-type calibration:**
- **Procedural skills** (step-by-step execution): gates are mandatory at every decision/cost point. Score normally.
- **Referential skills** (rules, conventions, knowledge): gates are N/A. Score 5 automatically. The agent must adapt these rules to varied scenarios — rigid gates would make the skill brittle.
- **Hybrid skills** (reference rules + optional pipeline): score gates ONLY on the pipeline/execution sections. Reference sections don't need them. A hybrid skill with good gates on its pipeline steps but no gates on its reference sections scores 4-5.
- **Tool reference skills** (command catalogs): gates are N/A. Score 5 automatically. The agent composes commands freely based on the task.

---

## Dimension 4: Output Format (weight: high)

If the skill produces output, is the format explicit?

| Score | Criteria |
|-------|----------|
| 1 | No output format specified. Agent invents format each time. |
| 2 | Output described in prose ("provide a summary with key findings"). |
| 3 | Partial template — shows some fields but not the complete structure. |
| 4 | Complete template in code block with placeholders, but missing tables or edge cases. |
| 5 | Exact template in fenced code block with every field, table, and placeholder. Labeled `REQUIRED OUTPUT`. |

**How to fix:** Add a step titled `## Step N: {Name} — REQUIRED OUTPUT` with a fenced code block showing every field and table.

**Skill-type calibration:**
- **Procedural skills:** output format is mandatory. Score normally.
- **Referential / tool reference skills:** no structured output expected. Score 5 automatically. The agent's output depends on the task it's applying the rules to.
- **Hybrid skills:** score output format ONLY if the skill has an explicit pipeline/execution mode that produces a deliverable. Reference sections don't need output templates.

---

## Dimension 5: Anti-Patterns (weight: medium)

Does the skill guard against known failures?

| Score | Criteria |
|-------|----------|
| 1 | No anti-patterns section. |
| 2 | 1-2 vague anti-patterns ("don't do bad things"). |
| 3 | 3+ anti-patterns but generic (not sourced from real failures). |
| 4 | 3+ specific anti-patterns from real failures, each with why it's bad. |
| 5 | 5+ anti-patterns from real failures, organized by severity, each actionable (tells agent what to do instead). |

**How to fix:** Ask the user: "What are the 3 worst mistakes the agent makes in this domain?" Convert each to: `**{Failure}.** {Why + what to do instead.}`

---

## Dimension 6: Modularity (weight: medium)

Is the skill appropriately split across files?

| Score | Criteria |
|-------|----------|
| 1 | Single file over 300 lines. Context saturation guaranteed. |
| 2 | Single file 200-300 lines with mixed concerns (procedures + reference tables + lessons). |
| 3 | Single file under 200 lines, cohesive domain. Acceptable but could benefit from split. |
| 4 | Main SKILL.md under 200 lines + REFERENCE.md for lookups. All files cross-referenced. |
| 5 | Clean split: SKILL.md (execution) + REFERENCE.md (lookups) + LESSONS.md (failures). Each file focused. No orphan files. Under 300 lines each. |

**How to fix:** Extract lookup tables to REFERENCE.md. Extract failure history to LESSONS.md. Add cross-references: "For X, see `REFERENCE.md` in this directory."

**Exemption:** Skills under 100 lines with a single cohesive concern score 5 as a single file.

---

## Dimension 7: Cross-References (weight: low)

Does the skill connect to the broader skill ecosystem?

| Score | Criteria |
|-------|----------|
| 1 | No mention of related skills or project docs. Operates in isolation. |
| 2 | Mentions CLAUDE.md but not specific related skills. |
| 3 | References 1 related skill or project doc. |
| 4 | References related skills where workflows overlap + relevant project docs. |
| 5 | All overlapping skills referenced. All supporting files cited by name. Dependency direction clear. |

**How to fix:** Check which other skills touch the same domain. Add: "For X, see the `{skill}` skill." Reference project docs where relevant (ARCHITECTURE.md, PLAYBOOKS.md, etc.).

---

## Dimension 8: Freshness (weight: low)

Is the skill maintainable over time?

| Score | Criteria |
|-------|----------|
| 1 | No `last_verified` date. No indication of when content was confirmed. |
| 2 | `last_verified` date but no `freshness_sources`. |
| 3 | `last_verified` + `freshness_sources` listed. Date over 60 days old. |
| 4 | `last_verified` within 60 days + `freshness_sources` + content matches current state. |
| 5 | `last_verified` within 30 days + `freshness_sources` + verified against current code/tools. |

**How to fix:** Add `last_verified: YYYY-MM-DD` and `freshness_sources:` to frontmatter. Update date when auditing.

**Exemption:** Skills covering stable conventions (copy style, UX principles) that rarely change can score 4 without recent verification.

---

## Dimension 9: Enforcement Readiness (weight: high)

Does the skill pair hard rules with mechanical enforcement?

| Score | Criteria |
|-------|----------|
| 1 | Skill contains NEVER/ALWAYS/MUST rules with concrete consequences, but no companion hook. Rules will be ignored. |
| 2 | Hard rules exist but enforcement is noted as "planned" or "TODO" without a working hook. |
| 3 | Skill has a companion hook for some hard rules but not all. Or hook exists but lacks a ledger (no audit trail). |
| 4 | All hard rules have companion PreToolUse hooks. Ledger exists. Missing one piece of the three-piece pattern (gate + ledger + recorder). |
| 5 | Full three-piece enforcement: PreToolUse gate + SQLite ledger + PostToolUse recorder. All hard rules covered. Bypass is audited. Skill frontmatter includes `enforced_by:` field. |

**How to fix:** Identify every NEVER/ALWAYS/MUST rule in the skill. For each, ask: "Does violation have concrete consequences?" and "Can the rule be checked mechanically?" If both yes, write a PreToolUse hook following the three-piece pattern in `REFERENCE.md` > "Enforcement Hooks." Proven examples: an AI-slop content gate, a cost-preflight gate, a duplicate-contact gate, a hallucination gate.

**Skill-type calibration:**
- **Skills with no hard rules** (pure referential knowledge, tool catalogs): Score 5 automatically. Not all skills need enforcement.
- **Skills with judgment-only rules** ("write warmly", "prefer X over Y"): Score 5 automatically. These can't be mechanically checked.
- **Skills with concrete hard rules** (cost limits, naming conventions, factual accuracy, copy bans, security gates): Score normally. These are the skills where enforcement matters.

---

## Scoring Summary

| Grade | Score | Interpretation |
|-------|-------|---------------|
| A | 39-45 | World-class. Agent follows reliably. Hard rules mechanically enforced. |
| B | 31-38 | Solid. Minor gaps that occasionally cause drift. |
| C | 23-30 | Functional but agent misses nuances or skips steps. |
| D | 15-22 | Unreliable. Agent partially follows at best. |
| F | <15 | Rebuild from scratch. |

**Minimum viable skill:** All high-weight dimensions (Structure, Description, Gates, Output, Enforcement) must score 3+. A skill scoring 5/5/5/5/5 on those five and 1/1/1/1 on the rest (total 29) is still a C -- functional.

**Priority order for fixing:** Enforcement > Structure > Gates > Output > Description > Anti-patterns > Modularity > Cross-refs > Freshness.

---
Part of operator-skills, the skills layer from an AI-native company that ran on 70+ agents. raianpollock.com
