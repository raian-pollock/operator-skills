---
name: find-email
description: >-
  The canonical procedure for locating a specific person's email address for prospecting (investors, partners, journalists, operators, schools, sales targets). AUTO-TRIGGER whenever the agent needs to find, look up, verify, prospect, or otherwise obtain a particular person's email address — do not wait for the user to invoke it explicitly. Stopping at "I couldn't find an email" after only pattern-guessing is exactly the failure mode this skill exists to prevent. Triggers include: (1) "find email" / "find his email" / "find her email" / "find their email" / "look up email" / "verify email" / "what's X's email" / "email for X" / "X's contact info"; (2) "prospect" / "prospecting" / "cold email" combined with a named person; (3) "investor email" / "VC email" / "partner email" / "founder email"; (4) explicit mention of Anymail Finder, Hunter.io, RocketReach, Lead411, VoilaNorbert, Snov.io, Skrapp; (5) frustration like "couldn't find their email" or "is there a way to find this person's email"; (6) any cold-outreach drafting task that needs a verified recipient address before it can proceed. Escalation order, free before paid, verify before send: (0) the local ledgers first, at no cost: a denylist of addresses that actually bounced (a hit ends the search), a ledger of known-good addresses searched by domain, and any mail the person sent you; (1) a free-tier email finder + verifier (Anymail Finder or similar) run across natural name/domain patterns; (2) search-engine dork sweep with site:/filetype:/intext: operators; (3) specialized free sources — GitHub commit-log mining, the Wayback Machine, personal sites, academic corresponding-author pages, conference speaker bios; (4) a second free-tier finder (Hunter.io); (5) a third free-tier enrichment API (e.g. Surfe) that also returns phone; (6) paid unlock (RocketReach / Lead411 / Apollo), only with the user's explicit go-ahead; (6.5) pivot to another fitting, reachable person at the same organisation; (7) LinkedIn/X DM or warm intro as the accepted fallback when a person has intentionally kept their email off the public web. Never stop at brute-force pattern-guessing alone — that is not "trying hard." And never send a first message to an address whose mailbox has not been verified, however the address was found: a bounce is not an acceptable outcome.
---

# find-email: locate a specific person's email for prospecting

## Hard rules

1. **Never stop at "I couldn't find the email" after only running finder pattern-guesses.** The verify sweep is one step of roughly eight. Stopping there is the exact failure mode this skill blocks.
2. **Never cold-email an unverified address.** Treat finder results by status: only an explicit `valid` ships. `risky` never ships in a batch; for a one-to-one message, prefer another channel (phone, a DM), and if the message still goes by email, write the named fallback channel down *before* sending, because `risky` is exactly the band that bounces. `not_found` / `invalid` / `unknown` does not ship. Verify calls are typically free when the result comes back invalid, so run them liberally.
3. **Never fabricate or guess a "likely email" in a draft.** If an address isn't verified, the draft doesn't get a recipient. Either escalate the search or fall back to a DM. Never paste `firstname@likely-domain.com` into an outbound message.
4. **Tag every "verified" claim with the tool that verified it.** `[finder: valid]`, `[finder: risky]`, `[hunter: 95% confidence]`, `[github commit log]`, `[personal site contact page]`. "We found X" without provenance is not acceptable.
5. **Quote the exact canonical email format an organization uses once discovered**, and cache it. `first.last@company.com` vs `flast@company.com` vs `first@company.com` are different patterns — record the pattern per domain so a future lookup at that org skips straight to verification instead of re-running the whole sweep.
6. **Before declaring a target "no email" / DM-only, you must have run every free tier to exhaustion and be able to say which ones ran.** Two lessons worth internalizing: (a) try the person's *own* domain, not just a parent/network/umbrella domain — a boutique operator's real address often lives on their own small domain, not the larger org they're listed under. (b) enrichment-by-profile-URL (when a provider supports it) reliably outperforms name+company lookups for solo operators or ambiguous companies — a name+company match can silently return the wrong person entirely, while passing the actual LinkedIn/profile URL disambiguates. "No email, DM only" is the genuine last resort, only after the free tiers plus an offered paid unlock have all come up empty.
7. **Verify the mailbox before any first message, however the address was found.** An address copied from a website, a directory, a conference list, a colleague or an old thread is a claim, not a verified mailbox. Run the verifier on it before the first send. A DNS MX lookup is not a mailbox check: it proves the domain accepts mail, not that this person's mailbox exists. Replies to mail the person sent you are exempt, and so are addresses already attested in your known-good ledger (Step 0). A send-time hook can enforce this mechanically by refusing a first message to any address that is not on the known-good ledger.

## Decision tree

Run techniques in this order. Stop as soon as you have a verified address. Each tier escalates cost, time, and friction.

```
STEP 0 (free, instant)            -- local ledgers: bounced denylist, then known-good ledger, then inbound mail
   |
   v (no denylist hit, no ledger hit, no inbound mail from them)
TIER 1 (free, fast, ~1-2 min)     -- finder API + pattern verification
   |
   v (if 'not_found' on all natural patterns)
TIER 2 (free, 5-10 min)           -- search-engine dork sweep
   |
   v (if no email surfaces)
TIER 3 (free, 5-15 min)           -- specialized sources (GitHub, Wayback, academic, speaker bios)
   |
   v (if nothing)
TIER 4 (free, 2-3 min)            -- second finder API (e.g. Hunter.io free tier)
   |
   v (if nothing)
TIER 4.5 (free, limited credits)  -- enrichment API with profile-URL lookup (e.g. Surfe)
   |
   v (if nothing)
TIER 5 (paid, ~$5-15)             -- paid unlock, ASK THE USER FIRST
   |
   v (if user declines paid, or address still elusive)
TIER 5.5 (free)                   -- pivot to another fitting, reachable person at the same organisation
   |
   v (no fitting alternative)
TIER 6 (fallback)                 -- LinkedIn/X DM or warm-intro path

REPORT at any tier: verified address + tier where found + source citation.
```

## Step 0: the local ledgers (free, instant, before anything that costs)

Two local stores answer a useful share of lookups for nothing, and one of them can end the search outright. Neither costs a credit, a quota slot or a network call. Keep both as plain text files next to your outreach tooling, git-ignored.

**1. The bounced denylist. A hit ends the search for that address.**
Grep it for the domain and for the surname. A row here is a mailbox that actually rejected mail, and that outranks every verifier verdict, including a fresh `valid`. Do not send, do not re-verify hoping for a different answer, and do not escalate to a paid tier: the address is dead, so the search is over and the question becomes which channel to use instead.

Why a separate denylist: an address already documented as a known verifier false positive bounced, and nobody removed it from the known-good list. A later session re-verified it, the vendor said `valid` again, and the dead mailbox wrote itself straight back into the known-good list. A known-good list can only ever say yes, so a bounce needs somewhere to live that a re-verify cannot undo. Grep the denylist first, or you will re-derive the bounce with your own credits.

**2. The known-good ledger. Grep it for the DOMAIN, not only the person.**
A hit on the person is the answer. A hit on anyone else at the same domain is nearly as good: it shows the organisation's email format for free, which is exactly what Tier 1 would otherwise spend credits to learn. Read the format off the hit and put it at the top of the pattern table below.

**3. If they ever emailed you, that beats every verifier.** Inbound mail proves the mailbox exists in a way no API can, and it costs nothing. Record the address in the known-good ledger with the reason ("they emailed us first") and stop. The recording step should refuse any address on the denylist and fail loudly when it does, so a refusal can never be read as a success.

Only when all three come back empty does Tier 1 start spending.

## Tier 1: finder API + pattern verification (start here once Step 0 is empty)

Wrap a free-tier email-finder API (Anymail Finder is a solid default; similar products work the same way) in a thin script or direct call. Cost discipline on these providers is typically: verifying an address that turns out invalid is free; verifying one that turns out valid, or running a "find" query, burns a small amount of credit.

**Find by name + domain (cheap when the domain is known):**
```bash
# Illustrative shape — check the provider's current API docs for exact params.
curl -s -X POST "https://api.<provider>.com/v5/find-email" \
  -H "Authorization: Bearer $PROVIDER_API_KEY" \
  -d '{"full_name": "Full Name", "domain": "company-domain.com"}'
```
Returns an address plus a status: `valid` / `risky` / `not_found`.

**Bulk-verify candidate addresses (typically free when they come back invalid):**
```bash
for addr in first.last@domain.com flast@domain.com first@domain.com firstl@domain.com; do
  curl -s -X POST "https://api.<provider>.com/v5/verify-email" \
    -H "Authorization: Bearer $PROVIDER_API_KEY" \
    -d "{\"email\": \"$addr\"}"
done
```

**Canonical pattern table (try in this order per domain):**
| Pattern | Example | Common for |
|---|---|---|
| `first.last` | `jane.doe@acmecorp.com` | US VC, EU tech |
| `flast` | `jdoe@acmecorp.com` | US enterprise, banks |
| `first` | `jane@acmecorp.com` | small firms, founder emails |
| `firstl` | `janed@acmecorp.com` | medium firms |
| `first_last` | `jane_doe@acmecorp.com` | rare, academic |
| `flastN` | `jdoe1@acmecorp.com` | name-collision domains |
| `lastfirst` | `doe.jane@acmecorp.com` | very rare, EU |

**Multi-domain fanout:** many people have multiple corporate emails (current employer, board seats, personal domain). Try the primary/firm domain first, then plausible alternatives (parent org, prior employer, personal domain).

**STOP CRITERION:** if the finder returns `valid` on any pattern, stop, log it, use that address. `risky` also stops the search, but it does not automatically ship: it never goes in a batch, and a one-to-one message needs the named fallback channel from Hard rule 2. If everything returns `not_found` after 5-8 attempts across 3-4 plausible domains, proceed to Tier 2.

## Tier 2: search-engine dork sweep

When pattern-guessing fails, the person may still have left an email in public web text — forum posts, speaker bios, press releases, academic papers, GitHub commits, conference programs, podcast show-notes, newsletter signups.

Use a web-search tool and a direct fetch tool for specific URLs. Combinations that work well:

**Direct email pattern dorking:**
```
"Full Name" "@" domain.com
"Full Name" email
"Full Name" contact
intext:"Full Name" intext:"@"
intext:"Full Name" "mailto:"
```

**File-type dorking (resumes, conference programs, speaker decks):**
```
"Full Name" filetype:pdf email
"Full Name" filetype:pdf contact
"Full Name" filetype:ppt OR filetype:pptx
"Full Name" filetype:doc OR filetype:docx
```

**Site-specific dorks for common email-leaking platforms:**
```
site:github.com "Full Name"                # commits expose author email
site:medium.com "Full Name"                # author profile may have email
site:substack.com "Full Name"              # author profile / about
site:academia.edu "Full Name"              # corresponding-author emails
site:researchgate.net "Full Name"
site:about.me "Full Name"
site:keybase.io "Full Name"                # crypto keys with linked emails
site:speakerdeck.com "Full Name"
site:youtube.com "Full Name" description   # channel descriptions
site:notion.site "Full Name"               # personal Notion pages
```

**Press-release + media-contact dorks:**
```
"Full Name" "media contact" OR "press contact"
"Full Name" site:prnewswire.com
"Full Name" site:businesswire.com
"Full Name" site:techcrunch.com email
```

**Conference + panel-speaker dorks:**
```
"Full Name" "speaker" "contact"
"Full Name" "panelist" email
"Full Name" "keynote" site:<some-industry-conference>.com
"Full Name" "panel" filetype:pdf
```

**Personal-domain discovery dorks (if you suspect a vanity domain):**
```
"Full Name" "personal website" OR blog
"Full Name" homepage
inurl:lastname OR inurl:firstname intext:"Full Name"
```

**Second search engine as backup:** a primary search engine sometimes deindexes, or never indexed, a specific page. A second engine (DuckDuckGo, Bing) has different crawl coverage — re-run the same dorks there before giving up on Tier 2.

## Tier 3: specialized free sources

Sources where emails are routinely exposed but require knowing where to look.

### GitHub git-log mining

If the target has a GitHub profile, their commits expose an author-email (unless they've enabled noreply masking).

```bash
# 1. Find their GitHub username via search (site:github.com "Full Name",
#    or `gh search users "Full Name"`)

# 2. List public repos
gh api users/USERNAME/repos --jq '.[].clone_url'

# 3. Clone the most-recently-updated public, non-fork repo
git clone <repo-url> /tmp/email-dig
cd /tmp/email-dig
git log --all --pretty=format:"%an <%ae>" | sort -u | head -20

# 4. Filter to author-matching emails
git log --all --author="Full Name" --pretty=format:"%ae" | sort -u
```

The GitHub public-events API also exposes emails for some users:
```bash
curl -s "https://api.github.com/users/USERNAME/events/public" | grep -oE '"email"\s*:\s*"[^"]+"'
```

### Wayback Machine

Older team/about/contact pages often listed a direct email before it moved behind a contact form.

```bash
# Get all archived snapshots of a page
curl -s "https://web.archive.org/cdx/search/cdx?url=company.com/about&output=json&limit=100" \
  | jq -r '.[1:] | .[] | "https://web.archive.org/web/\(.[1])/\(.[2])"'

# Fetch an older team page
curl -sL "https://web.archive.org/web/2018*/company.com/team"
```

The Wayback CDX API is open, rate-limited, and free.

### Conference speaker bios

Most conference websites post speaker bios with contact info. Search the conference's own site with `site:` restricted to that domain plus the person's name.

### Academic papers

If the person has co-authored a paper, the corresponding-author email is often published in the PDF.
```
site:scholar.google.com "Full Name"
site:arxiv.org "Full Name"
"Full Name" corresponding author filetype:pdf
```

### Personal websites + blogs

If they have a personal site, the contact/about page almost always has an email. Try common vanity-domain patterns (`firstnamelastname.com`, `lastname.com`, `firstlast.io`) and fetch the contact page directly.

### Social profile bios

Bio links on X/Twitter or similar often point to a personal blog, which has a contact page. LinkedIn About sections occasionally list an email for people who've opted in, but are typically behind a login wall.

## Tier 4: a second finder API (e.g. Hunter.io free tier)

Hunter.io's free tier is a reasonable second finder to run after Tier 1-3 come up empty: roughly 50 finds + 50 verifications per month, resetting monthly. Endpoints worth knowing: an account/quota check, a name+domain(s) find, a single-address verify, and a domain-search that lists every email a provider has seen for a domain (useful for confirming the canonical pattern).

**A field worth reading before trusting any returned address is the source classification:**
- Sourced from an actual scraped page → trust the confidence score.
- Pattern-generated with no real evidence behind it → treat as risky even at a high score.
- Domain flagged as "catch-all" (accepts mail to any address) → a "verified" result there is close to meaningless, because every random string at that domain would also "verify."

Origin: a lookup on a small consulting-firm domain returned a plausible address with a decent score, but the source type was "generated" (pattern-guessed, no real evidence) and the domain was catch-all. Flagging it as risky rather than trusting the score was the right call — catch-all plus generated is not verified.

## Tier 4.5: an enrichment API with profile-URL lookup (e.g. Surfe)

A second free-tier enrichment provider that supports lookup by profile URL (not just name+company) is worth keeping in the ladder after Hunter, before any paid step. These free tiers tend to be smaller (tens of credits per month vs. Hunter's ~50), so run the first finder tier before spending them, and reserve them for higher-value targets. A useful property of some of these providers: they return a phone number alongside email, which stays useful for warm intros even when the email itself turns out to be intentionally hidden.

**Why here in the ladder (after Hunter, before paid unlock):** scarcer free credits than Tier 4, so spend them only once Tier 4 has nothing and the target is worth it.

Origin: a profile-URL enrichment for a hard-to-find target returned no email but did surface a mobile number that matched the person's known location, at moderate confidence. That confirmed the email really was intentionally hidden across two separate paid-tier lookups — the mobile number wasn't used for cold outreach, but it's available if a warm intro ever needs it.

## Tier 5: paid unlock (ask the user first)

Any paid step should stay under the spend threshold the principal sets; above it, get explicit approval every time, per occurrence — don't default into a paid unlock silently. Treat anything within about 10% of the threshold as over it. Confirm the price live from the vendor's own price table before quoting it, never from memory, and price it in the currency the threshold is written in (a dollar figure that looks under a euro threshold can be over it). If the estimate said under and the actual cost crosses the threshold, stop, say so, and do not start the next lookup.

| Service | Approx. cost | When useful |
|---|---|---|
| RocketReach | ~$10/lookup or a monthly plan | When a masked preview (`g******@gmail.com`-style) is already visible |
| Lead411 | Cents per contact in bulk; monthly plans for volume | When you have a list of 10+ targets, not a single lookup |
| Apollo.io | Free tier available | General purpose; typically lower hit-rate than RocketReach on hard targets |
| ZoomInfo | Enterprise pricing | B2B sales orgs with existing seats, not ad-hoc single lookups |

**Decision rule:** frame the ask concretely — "Provider X has a masked preview for this person; one unlock is about $Y. Want me to go ahead, or stay on the DM path?"

## Tier 5.5: pivot to another fitting person at the same organisation

When the named target is unreachable (every tier exhausted) but the organisation is a genuine fit, do not fall straight to a DM of the unreachable person. First check whether another fitting, reachable person at the same organisation exists: run Tier 1 on its partners or decision-makers (a decision-maker beats a junior or departed contact). If one verifies (`valid`, or `risky` only under the one-to-one rule in Hard rule 2) and passes your dedupe checks, pivot to them. If the campaign already has approval to contact fitting people at that organisation, proceed; otherwise ask. The goal is a real conversation at a fitting organisation, not a particular name. Typical case: a named founder had no reachable company inbox, while a partner at the same firm verified as `valid`.

## Tier 6: LinkedIn/X DM or warm intro (accepted fallback)

If Tiers 1-5 all fail, the person has likely kept their email off the open web on purpose. Three paths, in rough order of fit:

1. **LinkedIn DM** — usually the best fit for B2B/investor outreach. A connection request with a short personal note, or InMail if you have it.
2. **X/social DM** — better fit if the person is visibly active there. Check the last-post date first; a dormant account won't reply.
3. **Warm intro** — ask your own network for an introduction. Alumni networks, professional communities, and shared-connection paths tend to outperform a cold DM.

When falling back to a DM: tell the user explicitly that the email path is exhausted and a DM is the channel, save the message body somewhere paste-ready, and note the "DM only" status against that person/org so a future session doesn't re-run the same search from scratch. If a sent email later bounces, add the address to the bounced denylist (Step 0) the same day.

## Anti-patterns

1. **Pattern-guessing without verification.** Sending to `firstname@company.com` because "that's usually the pattern" is a hard violation. Verify or don't send.
2. **Stopping after 5-10 verify attempts.** That's Tier 1 only. Tiers 2-5 exist precisely because Tier 1 alone regularly fails on real targets.
3. **Paying for an expensive plan to find one person's email when a DM would have worked about as well.** A single paid unlock for one hard target is fine; subscribing to an enterprise tool for one lookup is not.
4. **Sending to a "best-guess" address without telling the user.** Always surface the verification provenance.
5. **Caching a pattern too broadly.** If one pattern works for one person at an org, don't assume it holds for everyone there — different hiring eras at the same org sometimes use different conventions (founders often get `first@`, later hires `flast@`).
6. **Not respecting the privacy wall.** Some people have deliberately hidden their email. Pushing further with paid unlocks after masking + a dormant social presence is a poor signal on both sides — a warm intro will land far better than a found-and-cold email in that case.
7. **Re-verifying a bounced address.** A fresh `valid` from a verifier does not undo a real bounce. The denylist wins; the search is over for that address.
8. **Trusting an address because of where it was found.** A website, a directory or an old thread does not verify a mailbox. Verify before the first message (Hard rule 7).

## Origin case study (anonymized)

**What happened:** searching for a hard-to-find prospect's email for a cold-outreach campaign. Ran the Tier-1 verify sweep — roughly 30 pattern guesses across the person's natural domains and name variants. All returned invalid. Reported "DM only" and stopped there.

**What was missed:** Tiers 2-5. No search-engine dorks, no GitHub check, no Wayback Machine pass on old team pages, no second finder API, no offer of a paid unlock to the user.

**Lesson:** Tier 1 alone is not "trying hard." Re-running the full ladder on the same target took about five minutes and produced a confident answer instead of an open question: two paid-tier enrichment lookups both came back empty on email (one correctly flagged as an unusable catch-all/pattern-generated result), while a profile-URL enrichment surfaced a phone number that corroborated the person's known location — enough evidence to call the email genuinely, intentionally hidden rather than just "not found yet," and to route confidently to the DM path instead of leaving it as an unresolved thread.

## Tooling reference

- Two local ledgers (Step 0) — a bounced denylist and a known-good ledger, plain text, git-ignored, with a small wrapper that can list, grep and record entries. The denylist is checked first and is never overwritten by a `valid` verdict.
- A free-tier finder API (e.g. Anymail Finder) — find + verify + bulk-verify. Store the API key in a local, git-ignored env file, never inline in a script or committed config.
- Hunter.io — find + verify + domain-search + account/quota check. Free tier: roughly 50 finds + 50 verifies/month.
- An enrichment API with profile-URL lookup (e.g. Surfe) — enrich by LinkedIn URL, domain, or company name; returns email and sometimes phone. Free tiers here tend to be smaller and annual rather than monthly. Enrichment calls are often async (submit, then poll for the result).
- A resilient fetch helper — direct fetch first, falling back through a reader-view proxy, a web-archive lookup, and (if you have one) a paid scraping fallback for anything that 4xx/5xxs or hits a bot-wall (common on LinkedIn, Crunchbase, and other paywalled or gated pages).
- A web-search tool. Caveat: some search tools synthesize a "best-guess" email in their summary text — treat any email that didn't come with a real source link as unverified until confirmed via a finder/verify API.
- `gh` CLI — GitHub's authenticated API, useful for `gh search users` and `gh api users/USERNAME/repos` during Tier 3 git-log mining.

## Composition

- Sits inside a wider outbound-campaign discipline: this skill finds ONE person's address; the campaign discipline owns whether the name belongs on the list at all, the pre-send checks that run after an address is verified, and the dedupe that stops two campaigns contacting the same person. Use that discipline when the question is "should we contact them" rather than "what is their address".
- Pairs with an email-sending skill: once an address is verified, that's where draft creation and send discipline live.
- Pairs with a hallucination-reduction / evidence-tagging discipline: emails are claims, tag them with their source like any other claim.
- Pairs with an inferential-discipline habit: don't claim an email is "the one" without stating its verification provenance.

---
Part of operator-skills, the skills layer from an AI-native company that ran on 70+ agents. raianpollock.com
