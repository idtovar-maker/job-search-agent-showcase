---
name: indeed-discovery
description: Read Indeed job search results or a single Indeed posting from a page the candidate is already viewing in their own Chrome, via Claude in Chrome, and feed candidates into the same scoring pipeline as linkedin-discovery. Use when the candidate asks to read an Indeed search page, pull jobs from Indeed, run an "Indeed scan", or score an Indeed posting they have open. Never use to navigate, click, scroll, or get past a verification check on Indeed; reading only.
---

# Indeed Discovery (human-supervised)

Sibling of `linkedin-discovery`, built on the same pattern for the same
reason. Indeed blocks automated access: job-discovery's WebFetch calls get
403 responses, and the site-restricted WebSearch fallback returned nothing
usable across two full runs. Indeed is therefore read only this way: live,
in the main conversation, on a page the candidate opened themselves in their
own Chrome, with the candidate driving the browser.

This is a Skill, not a subagent, on purpose. A subagent runs unattended in an
isolated context with no browser, which is exactly the pattern Indeed's
protection is built to stop. This Skill never tries to work around that
protection. It only reads what a person browsing normally already sees.

## Hard constraints, do not violate these

1. **The candidate drives, Claude reads.** The candidate opens Indeed, runs
   the search, and clicks each job themselves in their own Chrome (via the
   Claude in Chrome extension). Claude never navigates to an Indeed URL,
   never types into the search box, never clicks a job card, "Next", "Apply
   now", "Save", or any other element, and never scrolls to load more
   results. Read-only page tools only (read_page / get_page_text / find).
   Reading never becomes clicking.
2. **Verification checks belong to the candidate.** If the page shows a
   "verify you are human", "Additional verification required", CAPTCHA,
   Cloudflare challenge, or sign-in wall, stop immediately, say so plainly,
   and wait for the candidate to clear it themselves and send the next cue.
   Never attempt to solve, bypass, reload around, or wait out a challenge,
   and never suggest tools or tricks that would (stealth browsers, proxies,
   CAPTCHA services, faked headers). If challenges start appearing on almost
   every card, say so and suggest the candidate slow down or stop for the day
   rather than pushing through.
3. **No account actions.** Never log in, never touch credentials, never apply,
   save, message an employer, upload a resume, or answer screener questions on
   the candidate's behalf. Indeed's "Easily apply" flow is the candidate's
   alone.
4. **One page at a time, cued loop only.** Two supported reading patterns:
   - **Single read:** the candidate asks once, Claude reads once, reports,
     stops.
   - **Cued loop (Mode A, same mechanics as linkedin-discovery's proven
     Mode A):** each check is triggered by a short message from the
     candidate.
   There is no autonomous polling mode here. linkedin-discovery's Mode B was
   never proven, and repeated unattended reads are a worse fit for a site that
   is actively screening for automated traffic. If the candidate asks for an
   autonomous loop, explain this and offer Mode A.
   Moving to a **new results page** always requires the candidate to navigate
   there themselves and then say so ("page 2 now"). Claude never follows
   pagination, even if it notices the candidate clicked "Next".
5. **Job data only.** Record title, company, location, pay, and posting
   content. Do not record recruiter or hiring-manager names, reviewer
   comments, or other people's personal details that may appear on the page.
   Company star ratings may be noted as a number only, no review text.

## Before reading anything

Read profile/preferences.md (target roles, dealbreakers, compensation range,
industries, the LinkedIn Title Filter section, which this Skill reuses as-is
for Indeed titles).

Also read, once per session, the files used for cross-source dedup (see
"Dedup across all sources" below): `scans/seen-postings.md`, today's
`scans/<YYYY-MM-DD>-linkedin.md` and `scans/<YYYY-MM-DD>-discovery.md` if they
exist, today's `scans/<YYYY-MM-DD>-indeed.md` if a run already happened
today, and `applications/LOG.md`. Skip any that don't exist. Re-read
`scans/seen-postings.md` and today's indeed scan file fresh immediately before
each write to them (the project's hard rule on shared files).

## Workflow

1. **Confirm the tab.** Ask for tab context and confirm the candidate has an
   Indeed tab open (ca.indeed.com, or indeed.com). Identify which kind of
   page it is:
   - a search-results page (`/jobs?q=...`): job card list on the left, job
     details pane on the right;
   - a single posting page (`/viewjob?jk=...`);
   - anything else (home page, company page, profile, a verification wall):
     say what it is and ask the candidate to open a job search or posting
     instead.
2. **Check for a verification wall first,** every single read, before
   extracting anything (Hard constraint 2).
3. **Search-results page, read the card list once.** For each visible card
   extract: Title, Company, Location / work mode (Remote, Hybrid, On-site as
   Indeed labels it), Pay as shown, "Easily apply" (yes/no), "Sponsored"
   (yes/no), posted date if shown ("Just posted", "3 days ago"), and the job
   key. The job key is the `jk` value, visible in the card's link
   (`...?jk=abc123...`) or as a `data-jk` attribute; it is Indeed's stable
   job ID and the primary dedup key. If no job key is visible for a card, mark
   it `(no jk)` and fall back to company plus title matching.
   Only extract cards actually rendered on the page. Never pad or infer.
4. **Dedup before anything else** (see the section below). Cards already seen
   are listed as a short reference, not re-processed.
5. **Title prefilter.** Apply the "LinkedIn Title Filter" section of
   preferences.md to the title, exactly as linkedin-discovery step 4 does,
   including the consultancy and marquee-company exceptions. Drop silently
   anything with an excluded seniority word or no matching role keyword. Keep
   genuinely ambiguous titles rather than guessing.
6. **Dealbreaker prefilter.** Drop a card only if the card itself visibly
   states native/bilingual French as required or Canadian citizenship/PR as
   required. When unsure, keep it.
7. **Number and list.** Give each surviving new posting the next number in
   today's Indeed sequence, `I-1`, `I-2`, and so on (see "Numbering and the
   scan file"). Report in the same compact format the other sources use, with
   flags appended:

   `I-N | Title | Company | Location/mode | Pay | URL | Flags`

   Flags (zero or more):
   - `sponsored`: a paid placement. Still a real posting, just labeled.
   - `est. pay`: the pay shown is Indeed's own estimate, not the employer's
     posted figure (Indeed marks these with wording like "Estimated" or
     "Indeed's estimate"). Never present an estimate as the posted pay.
   - `agency / employer undisclosed`: posted by a staffing or recruiting
     agency, or the real employer isn't named ("Confidential", "Our client",
     a recruiter firm name). Same caution as earlier cases where an
     aggregator or agency posting hid the real employer.
   - `French-first`: the card or title is in French. Not a dealbreaker by
     itself; flag it so the candidate decides early (a lesson from an earlier
     French-first application).
   - `easily apply`: noted for information only.

   Then one summary line, for example: "14 new (I-1 to I-14), 6 already seen
   (4 from LinkedIn, 2 from earlier Indeed runs), 3 filtered by title, 1
   filtered by dealbreaker." Append every listed row to the scan file,
   whether or not it ever gets scored.
8. **Scoring a posting (single read or cued loop).** The job details pane on a
   results page, or a `/viewjob` page, normally shows the full description.
   Before scoring, still check for truncation: text ending in an ellipsis, a
   visible collapsed "Show more" / "See full job description" control, or a
   description much shorter than a normal posting. If truncated, say so and
   ask the candidate to expand it (their click, not Claude's), then wait for
   the next cue.
   Also confirm the pane shows the posting the candidate meant: match its
   title and company, and its `jk` when visible, against the scan file row.
   If the full text is present, extract title, company, location, pay (noting
   employer-posted vs. Indeed estimate), description, and requirements, and
   run it through job-fit-scorer exactly as if the candidate had pasted it.
   The Essential-skills gap flag is recorded in the scan file when it
   applies, but it never pauses this scan (job-fit-scorer); the pause
   happens only later, at drafting.
   Output form:
   - One posting asked about on its own: the full job-fit-scorer breakdown.
   - Inside the cued loop: short form only,
     `I-N | Title | Company | Score/100 | Apply/Maybe/Skip [flags]`,
     with the full breakdown available on request.
   Append score, recommendation, any flags, and the full posting text to the
   posting's row in the scan file, so bulk-apply never has to go back to
   Indeed.
9. **Cued loop (Mode A).** Started when the candidate says something like
   "start the Indeed loop" or "Indeed scan, I'll cue you".
   - The candidate clicks a card, waits for the details pane to load, and
     sends any short cue ("n", ".", "next"). The cue's content doesn't
     matter. A posting the candidate skips the cue for simply doesn't get
     read.
   - On each cue: verification check (step 2), then read the details pane.
     - Same posting as the last one scored: say so briefly, don't re-score.
     - Already seen on another source or day: give the reference line, don't
       re-score (unless the candidate explicitly asks for a fresh look).
     - Truncated: say so and wait.
     - New and complete: score it (step 8, short form), append to the scan
       file and the seen-postings index, print the one-liner.
   - A card the candidate opens that wasn't in the original list (for
     example, Indeed inserted it after the first read): number it as the
     next `I-N` and add it to the scan file at that point.
   - **Every response in the loop ends with one explicit status line:**
     `→ ready for next`, `→ still waiting, expand the description`,
     `→ nothing new yet`, or `→ stopped: verification check, clear it and cue
     me`.
   - The loop ends when the candidate says stop, or when they move to a new
     page (then it resumes only after they say "page N now", still numbering
     forward).

## Dedup across all sources

Indeed aggregates heavily: the same job often also appears on LinkedIn, Built
In, or the company's own careers page. Check every card, in this order,
before prefiltering or scoring:

1. **`scans/seen-postings.md` by job key.** Indeed entries are keyed
   `indeed:<jk>`. A match means already processed.
2. **Company plus title, across every source.** Compare against
   `scans/seen-postings.md` (all sources, including LinkedIn rows),
   today's `-linkedin.md` and `-discovery.md` scan files, today's earlier
   Indeed rows, and `applications/LOG.md`. Use job-discovery's fuzzy rule:
   same company, same or near-identical title, ignoring case, punctuation,
   and minor wording differences ("Sr." vs. "Senior"). A different role at the
   same company is not a match. Treat an agency-posted card whose real
   employer isn't named as a non-match (it can't be matched reliably), but
   keep the `agency / employer undisclosed` flag on it.
3. **Reference format for a match**, for example:
   "<company>, Senior PM Checkout, already seen <date> on job-discovery as
   #3 (84, Apply), applied <date> per LOG.md". Don't renumber it, don't
   re-score it.

If a match is uncertain (same company, titles only roughly similar), keep the
card as new and note "possible duplicate of <reference>" next to it, so the
candidate decides rather than the Skill hiding it.

## Numbering and the scan file

New postings are written to `scans/<YYYY-MM-DD>-indeed.md`, one file per day,
created on first use. Numbering is prefixed `I-` and starts at `I-1` each day,
counting up across every Indeed page read that day; it never resets per page.
The prefix keeps Indeed numbers distinct from linkedin-discovery's plain `#N`
and job-discovery's numbers, so "apply to I-3" can never be confused with a
LinkedIn `#3` from the same day.

If today's file already exists (a second Indeed session the same day), keep
every existing row exactly as it is and continue numbering from the highest
`I-N` already there.

File structure:

```
# Indeed Scan: <YYYY-MM-DD>

Human-supervised Indeed reads (indeed-discovery). Numbers use the I- prefix.

| #   | Title | Company | Location/Mode | Pay | Flags | Score | Rec | URL |
|-----|-------|---------|---------------|-----|-------|-------|-----|-----|
| I-1 | ...   | ...     | ...           | ... | ...   | ...   | ... | ... |

## Already seen (not renumbered)
- <reference lines from the dedup step>

## Posting details
### I-1, <Title>, <Company>
- Job key: <jk>
- URL: https://ca.indeed.com/viewjob?jk=<jk>
- Pay: <as shown> (employer-posted | Indeed estimate)
- Score breakdown: Skills x/40, Seniority x/20, Preference x/25, Dealbreakers x/15
- Recommendation: <Apply/Maybe/Skip, plus Essential-skills gap flag if any>
- Rationale: <job-fit-scorer's one or two sentences>
- Full posting text:
  <verbatim text as read>
```

Build the URL from the job key (`https://ca.indeed.com/viewjob?jk=<jk>`)
rather than copying a long tracking link from the card. If the company's own
posting URL (Greenhouse, Lever, Workday, the company site) is visible in the
description, record it too: it is the better apply link and helps later
dedup.

On request, print the day's file back as one ranked table (#, Title, Company,
Score, Recommendation), highest score first, with listed-but-unscored rows
marked "not yet scored" rather than omitted.

## Seen-postings index

Whenever an Indeed posting is listed or scored, add or update its row in the
shared, never-reset `scans/seen-postings.md`, using the same columns
linkedin-discovery uses: key `indeed:<jk>` (or a `(fuzzy)`-flagged
title+company key when no jk was visible), title, company, first-seen date,
the scan file holding the detail (`<date>-indeed.md`), its `I-N` number, and
score/recommendation once scored. Re-read the file fresh immediately before
each write; never write it from a copy read earlier in the conversation.

## Honesty

If the page is still loading, mostly blank, or showing a verification or
sign-in wall, say so plainly; don't invent postings and don't retry on your
own. If the results look personalized or odd (for example, mostly sponsored
cards), say so. If a card's details pane doesn't match the card the candidate
seems to have clicked, say so instead of scoring the wrong posting.

## Relationship to the other discovery sources

- `linkedin-discovery`: same human-supervised pattern, for LinkedIn.
- `job-discovery` (subagent): unattended public-board search. It no longer
  searches Indeed at all; Indeed is covered only by this Skill.
- All three write into `scans/` under different suffixes (`-linkedin.md`,
  `-discovery.md`, `-indeed.md`) and share `scans/seen-postings.md`, so a job
  found on one is recognized on the others.
- `bulk-apply` accepts `I-N` numbers and reads them from the Indeed scan file.

Possible later addition, not built: parsing Indeed job-alert emails through a
Gmail connector as a second, browse-free feed into the same scan file.
