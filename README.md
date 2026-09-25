# operator-skills

The skills layer from an AI-native company. These are the written disciplines a fleet of
70+ AI agents inherited before they were allowed to plan, research, pick models, find
emails, or make claims — extracted from production and genericized.

A skill here is a markdown file a coding agent (Claude Code or similar) loads as
operating procedure. Prose guidance decays; skills are the durable form.

| Skill | What it does |
|---|---|
| [think-better](think-better/) | Structured opener before any non-trivial task: goal, interview decision, premortem, verifier |
| [interview](interview/) | Ask the principal to correct your best guess, in one batch of at most four questions, before direction-setting work |
| [director](director/) | Orchestration protocol for multi-step work: plan file, spawn test, four-field briefs, fan-out sizing, two-layer QA |
| [inferential-discipline](inferential-discipline/) | Evidence tagging, falsifiers, competing-hypotheses tournaments for agent research |
| [hallucination-reduction](hallucination-reduction/) | Four validated techniques, a decision matrix, and paste-ready templates |
| [model-pick](model-pick/) | Quick routing rule: adjust effort on the same model first; research only a genuinely new model choice |
| [find-email](find-email/) | Free-first escalation ladder for finding anyone's email, and verifying the mailbox before a first message |
| [seo-cluster](seo-cluster/) | SERP-overlap keyword clustering and hub-and-spoke architecture |
| [seo-geo](seo-geo/) | AI-search (GEO) visibility: what actually matters, what's myth |
| [seo-schema](seo-schema/) | Structured data: validation, active vs deprecated rich results, templates |
| [skill-builder](skill-builder/) | Meta: how to build and audit skills like these |

## Install

Copy any skill folder into your agent's skills directory (for Claude Code:
`~/.claude/skills/<name>/`). Each skill folder is self-contained. A few point to a sibling
(think-better to interview and model-pick; director to model-pick and think-better) and work best installed together.

## License

MIT — see [LICENSE](LICENSE).

---
Part of the operating system behind [raianpollock.com](https://raianpollock.com). The agents' tooling lives in the sibling repos; these skills are what teach them judgment.
