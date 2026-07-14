# Skill Builder — Reference

## Anatomy of a World-Class Skill

A skill that agents reliably follow has these 7 properties:

### 1. Front-Loaded Description

The description field determines when Claude auto-loads the skill. First 30 words must answer: **"When do I use this?"**

**Good:** "Run comprehensive QA checks after any UI or content change — Playwright screenshots, mobile verification, SSR checks..."

**Bad:** "Advanced comprehensive code analysis and quality assurance methodology for ensuring production readiness..."

Rules:
- Under 250 characters total (skill listing truncates beyond this)
- Primary use case in first clause
- List 2-3 concrete deliverables
- End with trigger condition: "Auto-triggers when..." or "Invoke with /name"

### 2. Scannable Structure

Claude treats numbered steps as a checklist. It treats prose paragraphs as background it may skip.

**Procedural skills:** Numbered steps (`## Step N: Title`), each with checkbox items (`- [ ]`), separated by `---` dividers.

**Referential skills:** Headed sections with tables and lists. No steps needed — just organize by lookup topic.

**Both types:** Opening line states the skill's purpose + cross-references to supporting files, immediately after frontmatter.

### 3. STOP Gates That Work

A STOP gate only works when paired with **explicit, verifiable criteria**.

**Ignored:** `STOP: Check with the user before proceeding.`

**Respected:** `STOP: Present the criticality table. Do not proceed until you have candidates from at least 3 price tiers + baseline + SOTA.`

Pattern: `STOP: {action verb} + {what to present} + {minimum criteria for proceeding}`

Place STOP gates:
- Before expensive operations (API calls, batch processing)
- Before irreversible actions (file creation, external communication)
- After analysis steps where the user should validate direction
- Before the final output (to catch incomplete work)

### 4. Explicit Output Format

If the skill produces structured output, show the EXACT template:

```markdown
## Step N: Report — REQUIRED OUTPUT

\`\`\`
## Report Title
- **Field 1:** [placeholder]
- **Field 2:** [placeholder]

| Col 1 | Col 2 | Col 3 |
|-------|-------|-------|
| [data] | [data] | [data] |
\`\`\`
```

Claude fills in your template. Without one, it invents a new format each time.

### 5. Anti-Patterns From Real Failures

Minimum 3 anti-patterns per skill. Source from:
- User corrections ("don't do X")
- Production incidents ("X broke because...")
- Common agent mistakes observed in practice

Format: **numbered list at the end of SKILL.md**, each item is one sentence stating the failure + why it's bad.

### 6. Modular Split Rules

| Metric | Single file OK | Split to REFERENCE.md |
|--------|---------------|----------------------|
| SKILL.md lines | <150 | 150-300 |
| SKILL.md lines | - | >300 = MANDATORY split |
| Lookup tables | 1-2 small tables | 3+ tables or any table >20 rows |
| Historical lessons | 0-2 inline notes | 3+ lessons = LESSONS.md |
| Tool subcommands | 1-3 commands | 4+ commands = references/*.md per topic |

**Reference rule:** Every file in the skill directory MUST be cited in SKILL.md by exact name. Example: "For architecture details, see `REFERENCE.md` in this directory."

### 7. Cross-Skill Awareness

Before creating a new skill, check for overlap:
- Does an existing skill already cover part of this domain?
- Should this be a section added to an existing skill instead?
- If two skills will interact, declare the dependency in both.

---

## Frontmatter Template

```yaml
---
name: skill-name
description: [Primary use case in first clause]. [2-3 deliverables]. [Trigger: Auto-triggers when X / Invoke with /name].
last_verified: YYYY-MM-DD
freshness_sources:       # Optional: what external data affects this skill
  - source_1
  - source_2
---
```

Optional fields (use only when needed):
```yaml
disable-model-invocation: true   # Side-effect skills (deploy, send-email)
user-invocable: false            # Background knowledge only
paths: "src/**,tests/**"         # Auto-trigger on file edit patterns
context: fork                     # Run in isolated subagent
allowed-tools: "Read, Grep"     # Restrict available tools
```

---

## Skill Skeleton — Procedural

```markdown
---
name: {name}
description: {front-loaded description under 250 chars}
---

# {Title} — Execution Checklist

{One sentence: what this does.}

For {detail topic}, see `REFERENCE.md` in this directory.

---

## Step 1: {Prerequisite / Assessment}

- [ ] 1a. {Verifiable action}
- [ ] 1b. {Verifiable action}

**STOP: {Present X}. {Minimum criteria to proceed}.**

---

## Step 2: {Main Work}

- [ ] 2a. {Action with specific command or instruction}

---

## Step N: {Output} — REQUIRED OUTPUT

\`\`\`
## {Report Title}
- **Field:** [placeholder]
\`\`\`

---

## Anti-Patterns

1. **{Failure}.** {Why it's bad.}
2. **{Failure}.** {Why it's bad.}
3. **{Failure}.** {Why it's bad.}
```

---

## Skill Skeleton — Referential

```markdown
---
name: {name}
description: {front-loaded description under 250 chars}
---

# {Title}

{One sentence: what rules this encodes.}

---

## {Category 1}

| Rule | Example | Why |
|------|---------|-----|
| {rule} | {example} | {reason} |

---

## {Category 2}

- **{Convention}:** {description}
- **{Convention}:** {description}

---

## Banned / Anti-Patterns

- {thing to avoid} — {why}
```

---

## 8. Enforcement Hooks

Advisory rules in skills (anti-patterns, STOP gates, "NEVER do X") have a 0% enforcement rate when the agent is under time pressure or working autonomously. The only reliable enforcement mechanism is a PreToolUse blocking hook that mechanically prevents the violation at tool-call time.

### Decision Tree: Does This Rule Need a Hook?

Ask these four questions about each hard rule in the skill:

1. **Does the rule use absolute language?** (NEVER, ALWAYS, MUST, HARD RULE) If no: advisory is fine.
2. **Has the rule been violated 2+ times across sessions?** If yes: it needs a hook.
3. **Does violation have concrete consequences?** (lost money, broken data, wrong email, hallucinated content) If yes: it needs a hook.
4. **Can the rule be checked mechanically?** (file exists? pattern matches? field present? DB lookup?) If no: keep advisory.

If questions 1+3+4 are all yes, the rule needs a hook regardless of violation count.

### Three-Piece Enforcement Pattern

Every enforcement hook follows three pieces.

| Piece | Purpose | Example |
|---|---|---|
| **PreToolUse gate** (`.claude/hooks/<name>.sh`) | Intercepts Edit/Write/Bash, validates via CLI, blocks with exit 2 if violation detected | a content-quality gate, a cost-preflight gate, a dedup-check gate |
| **Ledger** (`tools/shared/<name>.py`, e.g. a small SQLite store) | Records decisions, approvals, bypasses, actuals. Enables auditing and drift detection | a cost ledger, a contact ledger |
| **PostToolUse recorder** (`.claude/hooks/record-<name>.sh`) | Non-blocking. Records ground truth after the action. Feeds the ledger | a cost recorder, a touch/contact recorder |

### When to Keep Advisory

Not every rule needs a hook. Keep it advisory when:
- The rule requires judgment ("write in a warm tone")
- The rule is context-dependent and can't be reduced to a binary check
- The violation is low-stakes (preferences, not requirements)
- The rule changes frequently (hooks are harder to maintain than anti-patterns)

### Integrating Hooks into a Skill

When a skill needs enforcement:
1. The skill's SKILL.md documents the rule as an anti-pattern (for agent awareness)
2. The companion hook enforces it mechanically (for actual prevention)
3. The skill's frontmatter includes `enforced_by: .claude/hooks/<name>.sh`
4. The hook's block message references the skill for rewrite guidance

This dual-layer approach means the agent understands WHY the rule exists (from the skill) and cannot violate it regardless (from the hook).

---

## Description Quality Examples

| Score | Description | Problem |
|-------|-------------|---------|
| 1/5 | "Code quality and review tool" | No trigger, no deliverables, vague |
| 2/5 | "Comprehensive QA methodology for production readiness" | Buzzwords, no specifics |
| 3/5 | "QA checks for UI changes. Screenshots and mobile verification." | Decent but no trigger condition |
| 4/5 | "Run QA after UI/content changes — screenshots, mobile, SSR. Auto-triggers on src/app/ and src/components/." | Good. Could list one more deliverable. |
| 5/5 | "Run comprehensive QA checks after any UI or content change — Playwright screenshots, mobile verification, SSR checks, Chrome interactive QA, behavioral tests, and spec compliance. Auto-triggers when files in src/app/, src/components/... are modified." | Front-loaded, specific deliverables, trigger condition. |

---

## Common Agent Failure Modes to Guard Against

When writing anti-patterns, consider these universal failure modes:

1. **Skipping steps** — agent jumps to the "interesting" part. Fix: STOP gates after boring-but-critical steps.
2. **Inventing data** — agent generates plausible but wrong numbers. Fix: mandate reading from specific files/commands.
3. **Format drift** — agent changes output format over sessions. Fix: explicit template in code block.
4. **Scope creep** — agent does more than asked. Fix: "this skill does X. It does NOT do Y."
5. **Stale assumptions** — agent uses cached knowledge instead of reading current state. Fix: "always read X before..."
6. **Missing cross-reference** — agent doesn't check related systems. Fix: checklist items for adjacent concerns.
7. **Optimistic completion** — agent reports "done" without verifying. Fix: verification step before output.
8. **Context saturation** — skill is so long the agent loses detail. Fix: split to supporting files.

---
Part of operator-skills, the skills layer from an AI-native company that ran on 70+ agents. raianpollock.com
