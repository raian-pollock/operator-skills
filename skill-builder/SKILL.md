---
name: skill-builder
description: Build new skills or audit existing ones against world-class patterns. Use when creating a new skill, refactoring an existing one, or running a retroactive audit across all skills. Invoke with /skill-builder.
---

# Skill Builder — Execution Checklist

Two modes: **CREATE** (new skill) and **AUDIT** (existing skills). Pick based on user intent.

For the anatomy of a world-class skill, templates, and structural patterns, see `REFERENCE.md`.
For the detailed scoring rubric used in audits, see `AUDIT-RUBRIC.md`.

---

## MODE: CREATE

### Step 1: Interview — Understand the Domain

Before writing anything, ask the user:

- [ ] 1a. **What problem does this skill solve?** What goes wrong when the agent doesn't have it?
- [ ] 1b. **What's the trigger?** When should it activate — file patterns, keywords, manual only?
- [ ] 1c. **What type of skill is it?**
  - **Procedural** = numbered steps with gates. The sequence matters. (e.g. a deploy checklist, a QA runbook, a strategy-review pipeline)
  - **Referential** = rules, conventions, knowledge the agent applies with judgment across varied scenarios. No rigid steps. (e.g. copy-style rules, brand-voice conventions, analytics definitions)
  - **Tool reference** = command catalog the agent composes freely. (e.g. a CLI wrapper reference)
  - **Hybrid** = referential rules + an optional pipeline/execution mode. (e.g. localization rules with an optional translation pipeline)
  - **Critical:** Referential and tool-reference skills must NOT have rigid gates or output formats — these make the skill brittle and unable to adapt. Anti-patterns and clear rules are the right enforcement mechanism instead.
- [ ] 1d. **What are the 3 worst mistakes** the agent makes without this skill? These become anti-patterns.
- [ ] 1e. **Does it need gates?** Any step where the agent should STOP and get approval before proceeding?
- [ ] 1f. **Does it contain hard rules that agents must not violate?** Rules using NEVER/ALWAYS/MUST language, where violation has concrete consequences (lost money, broken data, wrong emails). If yes, the skill needs mechanical enforcement via PreToolUse hooks, not just advisory anti-patterns. See `REFERENCE.md` > "Enforcement Hooks" for the decision tree and three-piece pattern.
- [ ] 1g. **What's the output?** Report, code change, structured data, or just behavioral guidance?
- [ ] 1h. **Does it overlap with existing skills?** Check existing skills in `.claude/skills/` for conflicts.

**STOP: Present your understanding of the skill's purpose, trigger, structure, and output format. Get confirmation before proceeding to research.**

---

### Step 2: Domain Research & Synthesis

Before designing the skill, research how world-class teams and AI agent systems approach this specific domain. Then synthesize those findings through the lens of the user's reality and interview answers. The research is a baseline — its value comes from adaptation, not adoption.

**Phase A — External Research**

- [ ] 2a. **Practitioner research:** Web-search for how top teams handle this domain. What checklists, frameworks, runbooks, or standards do leading organizations use? Look for the teams that are genuinely world-class at this specific thing — not generic "best practices" listicles. (e.g., for a deploy-safety skill: how do Stripe, Vercel, Google SRE approach deploy safety?)
- [ ] 2b. **AI agent research:** Search for how other AI agent frameworks, coding assistants, and autonomous agent teams encode this domain. What do Devin, Cursor, open-source agent repos, or AI-native toolchains do? How do other Claude Code power users structure skills for this domain?
- [ ] 2c. **Failure research:** Search for public post-mortems, common pitfalls, and failure modes in this domain. What goes wrong when teams get this wrong at scale?

**Phase B — Synthesis & Adaptation**

The research is a baseline, not a blueprint. Now adapt it to the user's actual situation:

- [ ] 2d. **Filter for relevance:** Which findings actually apply to this user's scale, stack, and constraints? Discard patterns that solve problems the user doesn't have or that assume infrastructure they lack.
- [ ] 2e. **Cross-reference with interview:** Map research findings against the user's answers from Step 1 — their real problems (Q1a), their real mistakes (Q1d), their actual constraints. Where do world-class patterns address the user's specific pain points? Where are they irrelevant or overkill?
- [ ] 2f. **Identify gaps:** What did the research NOT cover that the interview revealed? The user's domain-specific failures are often not in any public playbook — these gaps are where the skill adds the most unique value.
- [ ] 2g. **Draft a synthesis brief:** 5-10 bullet points of adapted principles that will shape the skill design. Each bullet should trace to either a research finding adapted to context, or an interview insight no external source covers.

**STOP: Present the synthesis brief. The user must confirm that the adapted principles reflect their actual reality, not generic best practices. Adjust before proceeding to skill design.**

---

### Step 3: Determine Structure

Based on interview answers and the synthesis brief, pick the architecture:

| Skill Type | Structure | When |
|-----------|-----------|------|
| **Simple referential** | Single `SKILL.md` (<100 lines) | Rules, conventions, banned patterns. No steps. |
| **Standard procedural** | `SKILL.md` + `REFERENCE.md` | Step-by-step with lookup details that don't belong inline. |
| **Complex procedural** | `SKILL.md` + `REFERENCE.md` + `LESSONS.md` | Multi-step with known failure modes and historical gotchas. |
| **Tool-heavy** | `SKILL.md` + `references/*.md` | CLI tool with many subcommands needing focused docs per topic. |

**Hard rules:**
- `SKILL.md` MUST be under 300 lines. Over 300 = split to REFERENCE.md.
- Every supporting file MUST be referenced by name in SKILL.md.
- Never create supporting files "just in case." Only when content exceeds main file budget.

---

### Step 4: Write the Skill

Apply every pattern from `REFERENCE.md` > "Anatomy of a World-Class Skill." The synthesis brief from Step 2 should visibly inform the skill's structure, anti-patterns, and domain knowledge. Specifically:

- [ ] 4a. **Frontmatter** — name, description (front-load use case in first 30 words), `last_verified`, `freshness_sources` if data-dependent.
- [ ] 4b. **Opening line** — single sentence stating what this skill does, immediately followed by cross-references to supporting files.
- [ ] 4c. **Steps or sections** — numbered steps for procedural, headed sections for referential. Each step has checkboxes for verifiable actions.
- [ ] 4d. **STOP gates** — before any expensive, irreversible, or decision-dependent step. Every STOP paired with explicit criteria (checkbox list or table).
- [ ] 4e. **Enforcement hooks** — if the skill contains hard rules (interview Q1f), write the companion PreToolUse hook using the three-piece pattern from `REFERENCE.md` > "Enforcement Hooks." Advisory anti-patterns alone have a 0% enforcement rate; only hooks prevent violations reliably.
- [ ] 4f. **Output format** — if the skill produces output, show the EXACT format in a fenced code block with placeholders.
- [ ] 4g. **Anti-patterns** — minimum 3. Sourced from interview Q1d, the failure research from Step 2c, and the synthesis brief. Every anti-pattern should trace to a real failure, not imagination.
- [ ] 4h. **No orphan references** — every file in the skill directory is mentioned in SKILL.md.

**STOP: Present the draft skill for review. Do not create files until the user approves the structure.**

---

### Step 5: Validate Against Rubric

Before finalizing, score the new skill against `AUDIT-RUBRIC.md`. All 9 dimensions must score 3+ (out of 5).

- [ ] 5a. Run the audit rubric on your own draft.
- [ ] 5b. Fix any dimension scoring below 3.
- [ ] 5c. Present the scorecard alongside the final skill.

---

### Step 6: Write Files & Register

- [ ] 6a. Create the skill directory and files.
- [ ] 6b. Verify the description is under 250 characters (for the skill listing).
- [ ] 6c. Add an entry to the Skills Reference table in `CLAUDE.md` if the skill should be documented there.
- [ ] 6d. Test: mentally walk through 2 realistic scenarios and verify the skill handles both.

**REQUIRED OUTPUT:**

```
## Skill Created: {name}
- **Files:** {list of files with line counts}
- **Type:** {simple referential / standard procedural / complex procedural / tool-heavy}
- **Trigger:** {auto / manual / both} — {trigger description}
- **Research sources:** {count of external sources consulted in Step 2}
- **Synthesis brief:** {count of adapted principles}
- **Audit score:** {9 dimensions, all 3+}
- **Anti-patterns:** {count}
- **STOP gates:** {count}
```

---

## MODE: AUDIT

### Step 1: Scope

- [ ] 1a. **Single skill or all?** User specifies, or default to all skills.
- [ ] 1b. **Read every SKILL.md** in `.claude/skills/*/`.
- [ ] 1c. **Read supporting files** for context.

---

### Step 2: Score Each Skill

Apply `AUDIT-RUBRIC.md` to each skill. Score all 9 dimensions (1-5).

For each skill, note:
- Overall score (sum of 9 dimensions, max 45)
- Dimensions scoring below 3 (these are the fixable issues)
- Specific fix recommendations

---

### Step 3: Classify Issues

Group findings into:

| Priority | Criteria | Action |
|----------|---------|--------|
| **Critical** | Score 1-2 on Structure, Gates, or Output | Fix immediately — skill is unreliable |
| **High** | Score 1-2 on Description or Anti-patterns | Fix soon — skill triggers wrong or misses failures |
| **Medium** | Score 1-2 on Modularity or Cross-refs | Refactor when touching the skill next |
| **Low** | Score 1-2 on Freshness only | Update metadata |

---

### Step 4: Present Audit Report

**STOP: Present the full audit before making any changes.**

**REQUIRED OUTPUT:**

```
## Skill Audit Report — {date}

### Summary
- **Skills audited:** {N}
- **Average score:** {X}/45
- **Critical issues:** {N}
- **High issues:** {N}

### Scorecard

| Skill | Structure | Description | Gates | Output | Anti-patterns | Modularity | Cross-refs | Freshness | Enforcement | Total | Grade |
|-------|-----------|-------------|-------|--------|---------------|------------|------------|-----------|-------------|-------|-------|
| {skill-name} | 5 | 5 | 4 | 5 | 4 | 4 | 5 | 3 | 5 | 40 | A |
| ... | | | | | | | | | | | |

### Grading Scale
- A (39-45): World-class, hard rules mechanically enforced
- B (31-38): Solid, minor improvements
- C (23-30): Functional but unreliable in edge cases
- D (15-22): Needs significant rework
- F (<15): Rebuild from scratch

### Critical & High Issues
| Skill | Dimension | Score | Issue | Recommended Fix |
|-------|-----------|-------|-------|-----------------|
| ... | | | | |

### Recommended Refactoring Order
1. {skill} — {reason} (estimated effort: {S/M/L})
2. ...
```

---

### Step 5: Fix (with approval)

After user approves:
- [ ] 5a. Fix critical issues first, then high.
- [ ] 5b. Re-score each fixed skill to confirm improvement.
- [ ] 5c. Present before/after scores.

---

## Anti-Patterns

1. **Over-proceduralizing a referential skill.** Adding rigid STOP gates and output templates to a skill that provides rules/knowledge makes it brittle. The agent can't adapt to different scenarios. Referential skills enforce through anti-patterns and clear rules, not checklists.
2. **Writing a skill without interviewing the user.** You'll build for the wrong problem. Interview first.
3. **Monolithic SKILL.md over 300 lines.** Claude loses nuance mid-file. Split to REFERENCE.md.
4. **STOP gates without criteria.** "STOP: Check with user" is ignored. "STOP: Confirm X, Y, Z are present" is respected.
5. **Missing output format.** Claude invents its own format, inconsistent every time. Show the exact template.
6. **Anti-patterns from imagination.** Source them from real failures. Ask the user what went wrong before.
7. **Supporting files nobody references.** Orphan files waste context. Every file must be cited in SKILL.md.
8. **Description that buries the use case.** First 30 words must answer "when do I use this?" — not background.
9. **Auditing without the rubric.** Vibes-based audits miss the same issues every time. Use `AUDIT-RUBRIC.md`.
10. **Advisory-only rules for hard constraints.** If a skill says NEVER/ALWAYS/MUST and violation has consequences (cost, broken data, wrong emails), an anti-pattern list will not prevent it. Advisory rules have a 0% enforcement rate across dozens of observed sessions. The rule needs a PreToolUse blocking hook. See `REFERENCE.md` > "Enforcement Hooks" for the three-piece pattern.
11. **Adopting research findings verbatim without adaptation.** World-class team practices are a baseline, not a blueprint. A skill that copies a large org's runbook without adapting it to a solo developer's reality is worse than useless — it creates false confidence and cargo-cult procedures. Every research finding must pass through the synthesis filter: does it solve a problem the user actually has, at their actual scale?
12. **Skipping domain research entirely.** Building a skill purely from the interview produces a skill that only encodes what the user already knows. The research step surfaces patterns, failure modes, and frameworks the user hasn't encountered — then the synthesis step makes them relevant. Without research, the skill has a low ceiling.

---
Part of operator-skills, the skills layer from an AI-native company that ran on 70+ agents. raianpollock.com
