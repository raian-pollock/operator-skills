---
name: seo-cluster
description: >
  SERP-based semantic topic clustering for content architecture planning. Groups
  keywords by actual Google SERP overlap (not text similarity), designs hub-and-spoke
  content clusters with internal link matrices, and produces content briefs ready
  to hand to a writer or a content-generation pipeline. Use when the user says
  "topic cluster", "content cluster", "semantic clustering", "pillar page",
  "hub and spoke", "content architecture", "keyword grouping", or "cluster plan".
user-invokable: true
argument-hint: "<seed-keyword or url>"
---

# SEO Topic Clustering (Hub-and-Spoke)

SERP-overlap-driven keyword clustering for content architecture. Groups keywords
by how Google actually ranks them (shared top-10 results), not by text similarity
or stemming. Designs a hub-and-spoke content cluster with an internal link matrix,
then either drives content creation directly or generates ready-to-write briefs.

---

## Why SERP overlap, not text similarity

Two keywords that return the same Google results should be targeted by the same
page. Two keywords that return completely different results need separate pages.
This is the whole methodology: use Google's own ranking decisions to determine
content architecture, rather than trusting that similar-looking text implies the
same intent.

"Dog training tips" and "dog training classes" can have completely different
SERPs despite near-identical text. "Run" and "running" can target different
intents entirely. Never cluster on text similarity or stemming alone — check
the actual top-10 overlap. Full scoring algorithm and anti-patterns:
`references/serp-overlap-methodology.md`.

---

## Workflow

| Phase | What happens |
|-------|-------------|
| 1. Expand | Seed keyword -> 30-50 keyword variants |
| 2. Cluster | Group variants by SERP overlap (the core differentiator) |
| 3. Classify | Tag each keyword's intent; drop navigational ones |
| 4. Architect | Design the pillar + spoke clusters |
| 5. Link matrix | Design the mandatory/recommended/optional internal links |
| 6. Execute | Write content directly, or generate briefs for a writer |
| 7. Score | Run the cluster scorecard against the plan |

### Step 1: Seed Keyword Expansion

Expand the seed keyword into 30-50 variants using web search:

1. **Related searches** — search the seed, extract "related searches" and "people also search for"
2. **People Also Ask (PAA)** — extract all PAA questions from the results
3. **Long-tail modifiers** — append common modifiers: "best", "how to", "vs", "for beginners", "tools", "examples", "guide", "template", "mistakes", "checklist"
4. **Question mining** — generate who/what/when/where/why/how variants
5. **Intent modifiers** — add commercial modifiers: "pricing", "review", "alternative", "comparison", "free", "top"

**Deduplication:** normalize variants (lowercase, strip articles), remove exact
duplicates. Target 30-50 unique keyword variants. If under 30, run a second
expansion pass seeded with the top PAA questions.

**If the user already has a keyword list or a strategy doc** (a spreadsheet or
markdown table of keywords / page types / pillars), parse it and use those
keywords as the starting set instead of running Step 1 from scratch.

### Step 2: SERP Overlap Clustering

This is the core differentiator. Load `references/serp-overlap-methodology.md`
for the full algorithm.

**Process:**
1. Group keywords by initial intent guess (reduces pairwise comparisons)
2. For each candidate pair within a group, search both keywords
3. Count shared URLs in the top 10 organic results (ignore ads, featured snippets, PAA)
4. Apply thresholds:

| Shared results | Relationship | Action |
|---------------|-------------|--------|
| 7-10 | Same post | Merge into a single target page |
| 4-6 | Same cluster | Group under the same spoke cluster |
| 2-3 | Interlink | Place in adjacent clusters, add cross-links |
| 0-1 | Separate | Assign to different clusters, or exclude |

**Optimization:** with 40 keywords, full pairwise comparison is 780 SERP
fetches. Instead:
- Pre-group by intent (4 groups of ~10 = 4 x 45 = 180 comparisons)
- Only cross-check group-boundary keywords
- Skip pairs where both are long-tail variants of the same head term (assume same cluster)
- Spot-check ~20% of skipped pairs to verify the assumption held

**Data source:** plain web search is adequate for clustering, but result sets
can shift run to run — if a score lands near a threshold boundary, re-fetch
once and use the more common result set. If a dedicated SERP-data API (e.g.
DataForSEO) is available, prefer it for consistency, and check your own
cost/rate budget before running a large batch of lookups.

### Step 3: Intent Classification

Classify each keyword into one of four intent categories:

| Intent | Signals | Include in clusters? |
|--------|---------|---------------------|
| Informational | how, what, why, guide, tutorial, learn | Yes |
| Commercial | best, top, review, comparison, vs, alternative | Yes |
| Transactional | buy, price, discount, coupon, order, sign up | Yes |
| Navigational | brand names, specific product names, login | No (exclude) |

Remove navigational keywords from clustering. Flag borderline cases for manual
review. Keywords can have mixed intent (e.g., "best CRM software" is both
commercial and informational) — classify by dominant intent.

### Step 4: Hub-and-Spoke Architecture

Load `references/hub-spoke-architecture.md` for full specifications.

**Design the cluster structure:**

1. **Select the pillar keyword** — highest volume, broadest intent, most SERP overlap with the other keywords
2. **Group spokes into clusters** — each cluster is a subtopic area (2-5 clusters per pillar)
3. **Assign posts to clusters** — each cluster gets 2-4 spoke posts
4. **Select a template per post**, based on intent classification:

| Intent pattern | Template options |
|---------------|-----------------|
| Informational (broad) | ultimate-guide |
| Informational (how) | how-to |
| Informational (list) | listicle |
| Informational (concept) | explainer |
| Commercial (compare) | comparison |
| Commercial (evaluate) | review |
| Commercial (rank) | best-of |
| Transactional | landing-page |

5. **Set word count targets:** pillar page 2,500-4,000 words; spoke posts 1,200-1,800 words.

6. **Cannibalization check** — no two posts share the same primary keyword. If
   SERP overlap is 7+, merge those keywords into a single post targeting both.

### Step 5: Internal Link Matrix

Design the bidirectional linking structure:

| Link type | Direction | Requirement |
|-----------|-----------|-------------|
| Spoke to pillar | spoke -> pillar | Mandatory (every spoke) |
| Pillar to spoke | pillar -> spoke | Mandatory (every spoke) |
| Spoke to spoke (within cluster) | spoke <-> spoke | 2-3 links per post |
| Cross-cluster | spoke -> spoke (other cluster) | 0-1 links per post |

**Rules:**
- Every post must have a minimum of 3 incoming internal links
- No orphan pages (every post reachable from the pillar in 2 clicks)
- Anchor text must use the target keyword or a close variant (no "click here")
- Place links within body content, not just navigation/sidebar

Generate the link matrix as a JSON adjacency list:
```json
{
  "links": [
    { "from": "pillar", "to": "cluster-0-post-0", "type": "mandatory", "anchor": "keyword" },
    { "from": "cluster-0-post-0", "to": "pillar", "type": "mandatory", "anchor": "keyword" }
  ]
}
```

### Step 6: Turning the Plan into Content

**If you have a content-generation skill, writing pipeline, or downstream LLM
call available:**

1. Execute in priority order: pillar first, then spokes by search volume
   (highest first), then by cluster index, then by post index within a
   cluster. Rationale: the pillar establishes the topical-authority
   foundation the spokes link back to; high-volume spokes compound traffic
   fastest, so they should publish earliest.
2. For each post, pass a structured context block to the writing step:
   ```json
   {
     "cluster_context": {
       "role": "pillar|spoke",
       "pillar_title": "The Complete Guide to ...",
       "pillar_url": "/guide/...",
       "cluster_name": "Cluster Name",
       "cluster_index": 0,
       "post_index": 0,
       "primary_keyword": "target keyword",
       "secondary_keywords": ["variant 1", "variant 2"],
       "template": "how-to",
       "word_count_target": 1500,
       "outgoing_links": [
         { "url": "/pillar-url", "anchor": "main topic guide", "type": "mandatory" },
         { "url": "/sibling-post", "anchor": "related subtopic", "type": "recommended" }
       ],
       "incoming_link_placeholder": "<!-- cluster-link:cluster-0-post-1 -->",
       "differentiation_note": "This post should focus on X, while the sibling post covers Y"
     }
   }
   ```
3. After each post is written, scan previously-written posts for backward-link
   placeholders (`<!-- cluster-link:POST_ID -->`) and inject the new post's
   URL. If no placeholder exists in an already-written post that should link
   to the new one, append a contextual link in the most relevant section.
4. Support resuming a partially-executed plan: scan the output directory for
   already-written posts, match them against the plan (filename/slug or
   frontmatter keyword), mark those `"status": "written"`, and continue from
   the next unwritten post using the same priority order. If the pillar is
   missing but spokes exist, write the pillar first, then backfill links into
   the existing spokes. Treat a spoke under 50% of its word-count target as
   unwritten and recreate it.
5. After all posts are written, run the cluster scorecard (below).

**If no content-generation step is available**, generate a detailed content
brief per post instead, written to a `cluster-briefs/` directory as individual
markdown files. Each brief includes:
- Title and meta description
- Primary keyword and secondary keywords
- Template type and a suggested H2/H3 outline
- Word count target
- Internal links to include, with anchor text
- Key points to cover
- Competing pages to differentiate from

Reference `references/execution-workflow.md` for the full priority algorithm,
context-injection schema, backward-link mechanics, and resume logic.

### Step 7: Cluster Scorecard

Post-execution quality report. Run automatically after content is created (or
on demand against a partially-executed plan):

| Metric | Target | How measured |
|--------|--------|-------------|
| Coverage | 100% | Posts written / posts planned |
| Link density | 3+ per post | Internal links per post |
| Orphan pages | 0 | Posts with < 1 incoming link |
| Cannibalization | 0 conflicts | Duplicate primary keywords |
| Image count | 1+ per post | Posts with at least one image |
| Pillar links | 100% | All spokes link to pillar and vice versa |
| Cross-links | 80%+ | Recommended spoke-to-spoke links implemented |
| Content gaps | 0 | Planned posts skipped or incomplete |

Do not silently pass a failing cluster — if any gate fails, flag it with a
specific remediation step (e.g., "add a link from cluster-1-post-2 to the
pillar" rather than just "orphan page found").

### Optional: cluster visualization

A small standalone HTML page (nodes = pillar/spokes, edges = the link matrix)
is a useful sanity check but not required for the plan to be usable. If you
want one, it is simplest to generate it inline: build a small self-contained
HTML file with the cluster data serialized as JSON and a lightweight SVG or
canvas render — no external template or build step needed.

---

## Output files

| File | Description |
|------|-------------|
| `cluster-plan.json` | Machine-readable cluster plan (schema below) |
| `cluster-plan.md` | Human-readable cluster plan summary |
| `cluster-briefs/` | Content briefs (if no content-generation step is wired up) |
| `cluster-scorecard.md` | Post-execution quality report |

`cluster-plan.json` schema:
```json
{
  "seed_keyword": "string",
  "created_at": "ISO-8601",
  "pillar": {
    "title": "string",
    "keyword": "string",
    "volume": 0,
    "template": "ultimate-guide",
    "wordCount": 4000,
    "url": "string",
    "status": "planned|written"
  },
  "clusters": [
    {
      "name": "Cluster Name",
      "posts": [
        {
          "title": "string",
          "keyword": "string",
          "volume": 0,
          "template": "string",
          "wordCount": 1500,
          "url": "string",
          "status": "planned|written"
        }
      ]
    }
  ],
  "links": [
    { "from": "pillar", "to": "cluster-0-post-0", "type": "mandatory", "anchor": "keyword" }
  ],
  "serp_matrix": {
    "keywords": ["string"],
    "scores": [[0]]
  },
  "scorecard": {
    "coverage": 0.0,
    "linkDensity": 0.0,
    "orphanPages": 0,
    "cannibalization": 0,
    "contentGaps": 0
  }
}
```

---

## Composes with

- **Keyword research tooling** — if you already have a keyword-research skill
  or a rank-tracking tool, use its output as the starting keyword set and
  skip Step 1.
- **A content-generation skill or writing pipeline** — hand off each post's
  `cluster_context` block; see Step 6.
- **Schema-markup tooling** — apply `Article` + `BreadcrumbList` + `ItemList`
  JSON-LD to the pillar, and `Article` + `BreadcrumbList` to each spoke.
  Templates are in `references/hub-spoke-architecture.md`.

---

## Error Handling

| Error | Cause | Resolution |
|-------|-------|------------|
| No seed keyword provided | Missing argument | Prompt the user for a seed keyword or URL |
| Insufficient keyword variants | Expansion yielded < 15 keywords | Run a second expansion pass with PAA questions |
| SERP data unavailable | Search failing or rate-limited | Retry after ~30s; if persistent, fall back to intent-only clustering with a warning |
| No strategy doc found | Told to import from one but none exists | Prompt for a seed keyword instead |
| `cluster-plan.json` not found | Execution attempted without a plan | Run the planning workflow first |
| SERP API budget/quota exceeded | Using a paid SERP data source | Fall back to plain web search; inform the user |
| Duplicate primary keywords | Cannibalization detected | Merge affected posts, or reassign keywords |
| Orphan page detected | Post missing incoming links | Add links from the nearest cluster siblings |
| Resume state corrupted | Mismatch between plan and output directory | Rebuild state from an output-directory scan |

---

## Anti-patterns

1. **Never cluster by text similarity alone.** Verify with SERP overlap.
2. **Never use stemming-only grouping.** "Run" and "running" can target different intents entirely.
3. **Never assume related searches belong in the same cluster.** Verify with SERP data.
4. **Never ignore SERP feature differences.** A local pack vs. a featured snippet on otherwise-similar keywords often means different content types are needed.
5. **Never treat all domains equally when scoring overlap.** Wikipedia and other ubiquitous domains appear in many SERPs regardless of topic — consider filtering the top few most common domains out before scoring, or weighting topic-specific results higher.
6. **Never silently pass a failing scorecard.** Name the specific gate and the specific remediation.

## Security

- Output files should contain no PII, credentials, or API keys.
- If your implementation fetches pages directly (rather than via search
  snippets), validate/normalize URLs before fetching to guard against SSRF.

---
Part of operator-skills, the skills layer from an AI-native company that ran on 70+ agents. raianpollock.com
