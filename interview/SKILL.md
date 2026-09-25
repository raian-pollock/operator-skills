---
name: interview
description: >-
  How to interview the principal (the person the agent works for) at the START of a task: the few questions only they can answer, asked as corrections to the agent's own best guess, in one batch, before any drafting or building. AUTO-TRIGGER at the opening of any task that sets a DIRECTION (what gets built, who it is for, what it argues, what "done" means), before leaving plan mode, before drafting a blog post or any outward-facing artifact, and whenever the think-better opener's Interview line cannot honestly say "skipped because". Also fires on "interview me", "ask me first", "what do you need to know". Do NOT fire on mechanical or clear-repro work, on a follow-up whose direction was already set, or inside a sub-agent (its questions never reach the principal).
---

# Interview

**Ask the principal to correct your intent, never to supply it.**

That is the whole skill. Every question below is a version of *"I intend to do X — correct me"*,
never *"what should I do?"*. The first is delegation working. The second is handing the job back
(Oncken's second level of initiative, "ask what to do", from *Who's Got the Monkey?*, HBR 1974:
taking a decision the principal pays you to make and returning it to them with a question mark on it).

## Why this exists

Measured across the transcripts of one production agent fleet:

| Population | Ever asked the principal anything |
|---|---|
| Substantial sessions (over 1 MB of transcript) | 34 of 324 — **10.5%** |
| Genuinely interactive ones (more than 3 principal turns) | 19 of 84 — **22.6%** |

Sessions inside a formal planning mode, by contrast, almost always asked. **The gap is ordinary
task starts: roughly three in four working sessions never put a single question to the principal,
then produced work aimed somewhere the principal had to correct afterwards.** Re-measure on your own
transcripts: count the sessions that ever called the question tool.

## When to interview, and when not

**Interview** when a wrong guess wastes real work: a blog post or any outward artifact, a plan,
a new workstream or feature, a piece of research whose framing decides the answer, anything with an
audience, anything where "done" is a matter of taste.

**Do not interview** — and say which of these applies, out loud, in the think-better opener:
- the principal already gave an exhaustive spec (execute it; do not ask it back);
- mechanical or clear-repro work (a rename, a failing test, a known bug);
- a follow-up whose direction an earlier interview set (that answer is on file — using it is the
  point);
- a headless or scheduled run with nobody at the keyboard.

A kickoff on trivial work is the same disrespect as no kickoff on important work.

## The method

### 1. Earn the right to ask (silent, before any message)

Read what is on file first: the plan of record, persistent notes, past deliverables for this
workstream, `git log`, recent threads, the governing skill. List every fork the work will hit.
**Delete every fork the file already settles.** What survives, usually one to three, is the only
legitimate question material. If nothing survives, do not ask: state your assumption and go.

A question whose answer sits on disk is not a question. It is the tell that nobody read the file,
and principals notice it. If a file half-answers it, quote the half you have: *"On file you wanted
X — still true here?"*

### 2. Write the anchor first

One short paragraph stating the finished thing as if it were already done:

> "I'm going to write X, aimed at Y, arguing Z, about this long, published there."
> "I'm going to build X so Y can Z, shipping A but not B, done when C."

This is the cheapest artifact to correct, orders of magnitude cheaper than the thing itself. It
must be your **honest best inference from the file, never a placeholder**: anchoring is real and
people adjust away from an anchor too little (Tversky & Kahneman), so a lazy default silently
steers the work toward a direction nobody chose. Label it as provisional and expose the assumption
it rests on (*"this assumes it's aimed at buyers, not at other builders"*), so that correcting the
assumption visibly changes the answer. An unlabelled proposal gets rubber-stamped; a labelled one
gets corrected.

If you genuinely cannot pick, say so in one clause — *"genuinely torn, here's the trade"* — and
that admission is itself the question.

### 3. Ask at most four, in one batch

In Claude Code, `AskUserQuestion` takes **at most 4 questions per call**; use the same limit in any
runtime. Three is usually right. Zero is a legitimate and frequent answer. Question five is not
free: burden and satisficing rise with length (Krosnick), and past about sixty seconds of the
principal's attention a long ask does not get a worse answer, it gets *"looks fine"* — which reads
as consent and is not.

Batch once, in one message, answerable in one reply. Every extra round trip costs a full attention
interrupt worth more than the marginal question. Follow-ups belong in the delivery (*"I went with
A, here's why"*), never in a second ask.

**Fixed slots, filled from this task's actual forks:**

1. **The fork** (mandatory) — the one real branch the file does not settle, as two named concrete
   options plus "or neither".
2. **The anchor check** — your one-paragraph intent, offered for correction.
3. **The failure question** (high value, nearly always worth a slot) — in the **past tense**:
   *"Assume this is finished and it's wrong. What was wrong with it?"* The past-tense reframe is
   the active ingredient (Klein, HBR 2007, citing research that imagining the failure has already
   happened raises the number of reasons people identify by about 30%, where "what could go
   wrong?" reliably returns nothing).
4. **The blind spot**, on new workstreams or high stakes only: *"What should I be asking that I'm
   not?"* Drop it on routine work, where it reads as padding.

### 4. Attach a default so silence still ships

Every question carries what you will do if the principal says nothing: *"no reply = I go with A."*
Answering must be how the principal **redirects**, never how they **unblock** you. This is what
keeps interviewing compatible with pressing on: you are not waiting for permission, you are
offering a cheap chance to steer.

## Writing the questions

- **Closed with an escape hatch, never open.** Two concrete named options plus "or neither, and
  here's why". An open field under a sixty-second budget produces a shrug.
- **Your answer first, marked (Recommended).** Correcting a concrete proposal is fast and reliable;
  authoring a preference from a blank page is neither (recognition beats recall — Nielsen Norman
  Group).
- **One concept per question.** An "and" or a second question mark means it is two questions, and
  it will be half-answered (Sawatsky).
- **Neutral wording.** No loaded adjectives, no "don't you think" — you want to be corrected, and a
  planted answer hides the correction.
- **Anchor to real artifacts, not adjectives.** *"Same register as the March memo?"* beats *"how
  formal?"*. *"Like the launch page or like the blog?"* beats *"what style?"*. Things the principal
  has already judged are data; abstractions invite invention.
- **Force the trade-off, not the wish.** *"Rough by Thursday or right by Monday?"* beats *"what do
  you want here?"*, which returns "everything, well".
- **Write for a cold reader.** No file paths, no pipeline nouns, no internal jargon. If a question
  needs your working state to make sense, rewrite it around the consequence to the principal.
- **In Claude Code, use `header` (at most 12 characters) as the chip**, and `multiSelect: true` when
  the options are not mutually exclusive. "Other" is added automatically — never write your own.

## What you must NEVER ask

**The principal owns the OUTCOME; you own the EXECUTION.** Asking about execution bills them for the
work they delegated to you.

| Legitimate (outcome) | Never ask (execution / already yours) |
|---|---|
| Audience, thesis, the angle | Structure, tone, length, wording |
| What would make it wrong | Which model, which tool, which library |
| Scope edges: shipping A but not B | How to illustrate it, how long to spend |
| Priority between two real options | Where a rule or a file should live |
| Anything with external consequence or spend | Whether to proceed — keep going |
| A genuine strategic fork | Anything a file already answers |

Also never: permission to continue, small operational decisions, or a question already asked in an
earlier session. A fork settled once must never be re-asked. If you find yourself wanting to, the
answer was not recorded properly, and fixing that is the task.

## Question banks

### A blog post or any outward-facing artifact

Most blog failures are **standards** a writing skill should already own (the figures, a real
artifact as evidence, the title test, a voice pass). Those you execute; you do not ask about them.
Only these need the principal, because only they hold the raw material:

1. **The value line, offered as your draft, not as a question.** Write your own one-line answers to
   four value questions — what the reader can SEE now that they could not, how much FASTER they
   find out (old lag vs new), what it REMOVED (a role, a meeting, an evening), and what breaks, and
   what that costs, if it is switched off — and ask the principal to correct them. The common miss
   is answering these about the mechanism instead of about the reader.
2. **The thesis in one sentence**, with the assumption it rests on exposed.
3. **The real artifact**: name the screenshot, receipt or listing you plan to use as evidence, and
   ask whether a better one exists. The principal knows what actually happened; you do not.
4. **The failure question**, past tense.

Do not ask: which figures, what title, what length, whether to run the voice pass. Bring a titled,
illustrated draft; bring the direction BEFORE it.

### Cross-cutting — the questions the principal's own corrections keep asking for

Mined in one production fleet from 156 interactive transcripts, which held 25 direction-corrections
the principal had to make AFTER work was built. Counts, where given, are how many separate incidents
each question would have prevented. This is not a template to run every time; pick the ones this
task's forks actually raise.

1. **"Is this in the plan?"** (3 incidents) — before building anything the plan of record does not
   name, confirm it is wanted. Typical miss: changes to a set of pages the plan never mentioned. It
   also fires in reverse: say so if you are stopping short of the actual end goal.
2. **Who is the reader, and at what altitude?** (4) — typical miss: a draft polished small errors
   when the principal wanted it to step back to what the audience cares about.
3. **The disqualifying criteria, not the ideal ones.** Ask what makes a candidate, vendor or option
   a NO. Ideal-profile questions get answered warmly and filter nothing; the typical miss is a
   shortlisted option the principal rules out on sight for a reason nobody asked about.
4. **How much of the principal's time may this consume?** A plan that quietly books their hours gets
   rejected on that alone. Default to the channel that runs without them, and say so.
5. **Exposure and identity constraints**, whenever anything will carry the principal's name, face or
   company. Typical miss: their name appearing throughout a page they wanted it kept off entirely.
   Ask before generating, never after.
6. **The emotional framing of any narrative** — lead with the setback, or with what was done well?
   Typical miss: a story told with a negative spin that left out the work done to keep quality high.
7. **Which artifact is canonical?** when several versions of a design, deck or template exist.
   Building from the drifted one wastes the whole piece.
8. **The autonomy boundary**, when a set of recommendations is about to be executed: which of these
   am I pre-approved to do now, and which need your word first? Typical miss: the principal trusted
   every item and wanted the agent to have asked once, then acted, instead of stalling.

**A recurring correction with a CONSTANT answer is not a question — it is a standard, and it
belongs in a skill.** Typical examples: render the artifact and look at it before declaring it
done; prefer a CLI path over a manual click; verify every contact before sending. Asking about
those every time is the questionnaire failure this skill exists to prevent. The test: *would the
principal's answer be the same on every task?* If yes, encode it once. If it changes per task, ask
it.

### A plan or a build

1. The one architectural or scope fork the plan of record does not settle.
2. Your intent paragraph: shipping A but not B, done when C.
3. The failure question, past tense.
4. Priority against what is already in flight, if this competes for the same week.

### Research or a recommendation

1. The decision this feeds, and what would change if the answer came back the other way.
2. Your framing of the question, offered for correction — framing decides the answer.
3. What evidence would be strong enough to act on.

## After the interview

Reflect the answers **visibly** in what you produce: quote the correction and name where it changed
the work. Then record every answer where the next agent will find it: the plan document, the
workstream's notes, or the project dashboard. An answer that lives only in one transcript will be
asked again, and a repeat ask across sessions is a scaffolding defect to fix, not a question to send.

If some work does not depend on the answers (research, data gathering, setting up tools), start it
before you ask, then feed the answers into its briefs. If the principal answers "I'm not sure,
check X", check X and decide yourself; say what you found and what you chose. Asking again is the
questionnaire failure.

## Enforcement

A pre-tool hook can make this mechanical: refuse to leave plan mode, or to write into publishing
paths, while the session has asked nothing, and explain why so the agent interviews and retries. It
must open on a real question, on the principal waiving the interview in their own words, for
sub-agents, and for headless runs, and it should fail open on any error.

**Sub-agents cannot interview.** A sub-agent's question never reaches the principal, so the
interview happens in the main session, before the fan-out — never inside a worker.

## Sources

Oncken & Wass, *Management Time: Who's Got the Monkey?* (HBR 1974) · Klein, *Performing a Project
Premortem* (HBR 2007) · Krosnick & Presser, *Question and Questionnaire Design* (2010) · Tversky &
Kahneman, anchoring and adjustment (1974) · Nielsen Norman Group, recognition vs recall · Sawatsky
via Poynter, lean and neutral questions · Amazon Working Backwards (the forcing function only) ·
Fitzpatrick, *The Mom Test* (past-behaviour kernel only).

Deliberately NOT adopted: The Mom Test's "never mention your idea" (the principal is the
decision-maker and wants your recommendation, not to be studied as a research subject), story-based
discovery interviews (you already hold a long record of the principal's decisions on disk),
open questions always (right for a reluctant source over 30 minutes, wrong for a cooperative one
over 60 seconds), the full PR/FAQ, ghostwriter voice interviews (a voice corpus and a lint do that
job better), and any fixed kickoff template — a template is a questionnaire by another name, and
its questions are generic, with the answer already on file, by construction.

---
Part of operator-skills, the skills layer from an AI-native company that ran on 70+ agents. raianpollock.com
