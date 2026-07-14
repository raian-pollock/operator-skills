# Templates — Few-shot suffixes + CoVe rubrics by content type

Paste-ready templates. Each pairs a few-shot suffix (technique #1 from `SKILL.md`) with a CoVe rubric (technique #3) for the same content type. Adapt freely; the structure is what matters more than the exact examples.

## Format conventions

- Few-shot suffixes are prepended to the LLM's system or user prompt.
- Each example pairs a CORRECT artifact with an explicit `NOT:` counter-example.
- Rubrics return `{ per_unit: [...], overall_pass: bool }` JSON. Each unit has 3-5 boolean dimensions plus an `issue` string.
- Critic temperature is 0.1; generator temperature for the paired few-shot is 0.4.

---

## 1. Primary-source artifacts (history, science, legal docs)

**When:** Generating letters, decrees, transcripts, manifests, ledgers, diary entries, schematics.

**Validated basis:** This structure is the text that lifted an internal generator from 33% to 100% pass rate in an A/B test (see `REFERENCE.md`).

### Few-shot suffix structure

Three diverse archetypes (letter, table, decree), each paired with a `NOT:` meta-description counter-example, ending with 5 numbered rules. The structural template is:

```
ADDITIONAL PRIMARY-SOURCE EXAMPLES — READ BEFORE GENERATING:

EXAMPLE — First-person letter (~700 chars)
  [Letter from <named author> to <named recipient>
   <Place>, <Date>
   Dear <name>,
   <Specific concrete content with sustained voice>
   Your loving <relationship>, <name>]

NOT: "<meta-description wrapped in italics>" — that is a meta-description, not a letter.

EXAMPLE — Data table / ledger (~700 chars)
  [<Title> — <Date range>
   Compiled by <named compiler>
   <Tabular data with real-looking numbers and units>]

NOT: "<meta-description>" — that is a meta-description, not a table.

EXAMPLE — Edict / decree (~700 chars)
  [<Document name> — <article/section>
   Promulgated by <named ruler/body>
   Given at <place>, <date>
   "<Quoted text in period-appropriate register>"]

NOT: "<meta-description>" — that is a meta-description, not a decree.

WHAT THESE EXAMPLES DEMONSTRATE:
1. Every document has a NAMED source, date, and place.
2. Voice is specific and sustained.
3. Concrete detail that a reader can point to.
4. The document IS the artifact, not a summary describing it.
5. Italic markdown is NEVER used for stage directions or placeholders.
```

### CoVe rubric

```json
{
  "rubric_id": "primary-source",
  "version": "1.0",
  "questions": [
    { "id": "is_primary_source", "prompt": "Is this an actual primary-source artifact (letter, memo, table, decree, transcript, manifest, diary entry, etc.) with named source + date + place + concrete specific detail?" },
    { "id": "voice_is_consistent", "prompt": "Does it maintain ONE voice throughout — no switching from first-person to third-person narration, no breaking character mid-document?" },
    { "id": "no_stage_directions", "prompt": "Does it contain ZERO italic-wrapped *...* placeholders, ZERO [TBD]/[TODO]/[PLACEHOLDER] markers, and ZERO meta-descriptions like 'A map showing...'?" },
    { "id": "no_fabrication_red_flags", "prompt": "Are the named people/dates/numbers historically anchored (real era + real place + plausible detail), NOT obviously invented modern names or impossible dates?" }
  ],
  "fail_if_any_false": true
}
```

---

## 2. Academic citations / bibliographies

**When:** Generating reference lists, in-text citations, DOIs, journal names.

**Critical:** CoVe alone is not enough. Pair with a corpus-check step (CrossRef, Semantic Scholar, OpenAlex APIs) wired directly into the pipeline before publish, rather than relying on CoVe alone.

### Few-shot suffix structure

```
CITATION EXAMPLES — STRUCTURED CORRECTLY:

Every citation has: Author(s) (Year). Title. Journal/Venue, Volume(Issue), Pages. DOI.

EXAMPLE — Journal article (APA)
  Hattie, J., & Timperley, H. (2007). The power of feedback. Review of Educational Research, 77(1), 81-112. https://doi.org/10.3102/003465430298487

EXAMPLE — Book chapter
  Bransford, J. D., Brown, A. L., & Cocking, R. R. (Eds.). (2000). How people learn: Brain, mind, experience, and school (Expanded ed.). National Academy Press.

EXAMPLE — Government report
  OECD. (2019). PISA 2018 results (Volume I): What students know and can do. OECD Publishing. https://doi.org/10.1787/5f07c754-en

NEVER do any of these:
1. Cite a paper you cannot name the journal for. If you cannot remember the journal, the citation is fabricated.
2. Use generic DOIs like "10.1000/xxx" or fake-looking patterns like "10.1234/abcdef".
3. Cite an author's surname alone without first names and full publication info.
4. Pair a real author with a paper they did not write. If you are not sure, omit the citation.
5. Include URLs to PDFs you have not visited (use the DOI instead — DOIs resolve, fake URLs 404).

If you are uncertain about ANY citation, mark it [UNVERIFIED] and let a human resolve it.
```

### CoVe rubric

```json
{
  "rubric_id": "citation",
  "version": "1.0",
  "questions": [
    { "id": "has_full_attribution", "prompt": "Does this citation include author(s), year, title, and venue (journal/publisher)?" },
    { "id": "has_resolvable_doi_or_url", "prompt": "Does it include a DOI in the format 10.XXXX/... or a URL to a known publisher domain (not a generic Google search)?" },
    { "id": "author_paper_pairing_plausible", "prompt": "Is the author known for working in this topic area? Flag if the topic seems outside the author's field." },
    { "id": "journal_year_pairing_plausible", "prompt": "Is the journal real and was it publishing in the cited year? Flag obviously fictitious journal names." },
    { "id": "no_unverified_marker_needed", "prompt": "Is there NO [UNVERIFIED] tag suggesting the generator already flagged this?" }
  ],
  "fail_if_any_false": true,
  "post_process": "Pipe all flagged citations through CrossRef/Semantic Scholar before manual review."
}
```

---

## 3. Statistics in body copy

**When:** Generating stat-driven content (blog, report, slide deck).

### Few-shot suffix structure

```
STATISTICS — FORMAT AND ATTRIBUTION:

Every statistic must be: NUMBER + UNIT + POPULATION + SOURCE + YEAR. Missing any element = unverifiable.

GOOD: "61% of US adults own a smartphone (Pew Research Center, 2024)."
GOOD: "PISA 2018 reports that 23% of 15-year-olds in OECD countries scored below Level 2 in reading literacy (OECD, 2019)."
GOOD: "England saw a 53% drop in detentions for mobile-phone use in classrooms after the 2024 ban (UK DfE, 2025)."

UNVERIFIABLE (banned patterns):
- Statistic with no source attribution
- Hedge phrases with no specific citation (the "research shows" / "studies have found" pattern)
- Brand-name handwaving (e.g., "A Harvard study found...")
- Population missing (40% of WHO?)

RULES:
1. If you do not know the source of a statistic, do not include the statistic.
2. Round numbers carefully — 53% should not become "more than half" if the precision matters.
3. Check temporal coherence — a 2020 statistic cannot describe 2025 conditions.
4. Either name the study/source or cut the claim.
```

### CoVe rubric

```json
{
  "rubric_id": "statistics",
  "version": "1.0",
  "questions": [
    { "id": "has_number_and_unit", "prompt": "Does the statistic include both a number and a unit (% / count / dollars / years)?" },
    { "id": "has_named_source", "prompt": "Is the source named specifically (e.g., 'Pew Research', 'OECD PISA 2018'), not a generic hedge phrase?" },
    { "id": "has_year_or_date", "prompt": "Is the data's year or collection date stated?" },
    { "id": "magnitude_plausible", "prompt": "Is the magnitude in the realm of physical possibility for the cited population?" },
    { "id": "temporal_coherence", "prompt": "Does the statistic's date align with the time period being discussed in the surrounding text?" }
  ],
  "fail_if_any_false": true
}
```

---

## 4. Named experts / quoted researchers

**When:** Generating quoted text attributed to a named person.

**Risk:** Few-shot is RISKY here. Examples teach the LLM to write convincing fake quotes. Rely primarily on CoVe + name-existence verification.

### CoVe rubric (no few-shot suffix recommended)

```json
{
  "rubric_id": "named-expert-quote",
  "version": "1.0",
  "questions": [
    { "id": "person_is_real", "prompt": "Is the named person a real public figure / researcher / practitioner you have prior knowledge of? Flag if the name sounds generic or constructed." },
    { "id": "quote_is_attributed_with_source", "prompt": "Is the quote attributed to a specific paper, talk, interview, or book — not just a name floating in air?" },
    { "id": "quote_voice_matches_person", "prompt": "Does the language register match what this person would actually say given their field?" },
    { "id": "no_fabrication_red_flags", "prompt": "Does the quote avoid suspiciously rounded numbers, generic platitudes, or claims the person is unlikely to have actually made?" }
  ],
  "fail_if_any_false": true,
  "post_process": "Any flagged quote requires manual verification via the cited source before publication. If the source cannot be located, REMOVE the quote — do not paraphrase a fabricated quote into a hedged 'experts say' construction."
}
```

---

## 5. Code touching real APIs / libraries

**When:** Generating code that imports specific libraries or calls named APIs.

**Strategy:** Temperature dial only. NO CoVe. Reason: a test suite is the real critic; LLM-judging-LLM on code adds cost without lift, and real API surfaces change too fast for the LLM critic to be authoritative.

### Few-shot is NOT recommended

Reason: API surfaces shift between versions; a few-shot example anchored to v1 can mislead generation for v2. Read the actual library source / docs and put THAT in the prompt context (RAG-style), not crystallized examples.

### Best practice

1. Set temperature to 0.3 (lower than 0.4 because code is more structural).
2. Read the relevant library file or doc page and include it in the prompt as `## Library context (verbatim from <path>)`.
3. Run the generated code through a real test before trusting it. Do not skip this with `--no-verify` style shortcuts.

---

## 6. Plan / spec documents

**When:** Generating implementation plans, design specs, ADRs.

**Strategy:** CoVe only — checks every claim traces to a stated requirement.

### CoVe rubric

```json
{
  "rubric_id": "plan-traceability",
  "version": "1.0",
  "questions": [
    { "id": "has_explicit_requirement_link", "prompt": "Does each numbered task trace back to a stated user requirement, constraint, or upstream decision?" },
    { "id": "no_invented_features", "prompt": "Does the plan avoid introducing features or capabilities the user did not ask for?" },
    { "id": "no_invented_dependencies", "prompt": "Are all referenced files, services, and APIs ones the user has confirmed exist or that the agent has read directly?" },
    { "id": "scope_matches_request", "prompt": "Is the plan scoped to what was requested — neither over-engineered nor a partial answer?" }
  ],
  "fail_if_any_false": true
}
```

---

## Custom rubrics

For project-specific rubrics, write a JSON file matching the schema above and wire it into your own critic step. Each rubric needs:

- `rubric_id`: lowercase-hyphen identifier
- `version`: bump on changes
- `questions`: array of `{ id, prompt }` — each prompt must be a yes/no question
- `fail_if_any_false`: usually `true`; set `false` for soft-rubric scoring
- `post_process` (optional): human-readable note on what to do with flagged items

Co-locate custom rubric files with the skill (e.g., a `rubrics/` subfolder) or with the project that owns the rubric.

---
Part of operator-skills, the skills layer from an AI-native company that ran on 70+ agents. raianpollock.com
