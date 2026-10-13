---
name: linkedin-discovery
description: Read LinkedIn job search results or a single posting from a page the candidate is already viewing in their own logged-in Chrome, via Claude in Chrome, and feed candidates into the same pipeline as job-discovery. Use when the candidate asks to read a LinkedIn search page, pull jobs from LinkedIn, or score a LinkedIn posting they have open. Never use to navigate, click, or scroll through LinkedIn on its own initiative. Reading (including the experimental autonomous loop) is always fine, driving the browser itself never is.
---

# LinkedIn Discovery (human-supervised)

This is a Skill, not a subagent, on purpose. job-discovery runs unattended in an
isolated context and hands back a finished list. LinkedIn's Terms of Service do
not allow that pattern here: any automated crawling, scripted navigation, or
background scraping of LinkedIn is out of bounds regardless of intent. This
Skill only ever runs in the main conversation, live, with the candidate
watching and driving the browser themselves.

## Hard constraints: do not violate these

1. The candidate must already be logged into LinkedIn themselves, in their own
   Chrome, via the Claude in Chrome extension. This Skill never logs in, never
   touches credentials, never posts, applies, messages, or connects on the
   candidate's behalf.
2. Claude only ever reads content the candidate has already put in front of
   it. There is no persistent background process that lives *between
   separate messages*: Claude cannot wake up on its own minutes after a turn
   has already ended, no matter what a SKILL.md says. (Confirmed live: an
   earlier version of this file described an autonomous "~10-15s timer"
   working across messages, and it didn't work. Postings got missed and the
   loop kept stalling.) Within one continuous turn, though, Claude genuinely
   can keep making tool calls in a loop (read, wait a few seconds, read
   again, repeat) without the candidate sending anything new in between;
   that distinction is what the autonomous loop (Workflow step 8, Mode B) is
   built on and testing live.
   Whichever mode is running, Claude never navigates to a LinkedIn URL on
   its own initiative, never clicks "Next," a job card, "...more," a
   profile link, or any other LinkedIn element, and never scrolls to
   trigger additional results loading. Use read-only page-reading tools
   (read_page / get_page_text) only. Reading, however many times and
   however it's triggered, never becomes clicking.
3. One page at a time, with three supported reading patterns on that page:
   - **Single read:** the candidate asks once, Claude reads once, reports
     back, and stops until asked again.
   - **Cued loop (Workflow step 8, Mode A), proven:** once the candidate
     starts it, Claude reads and scores whatever's expanded every time the
     candidate sends a short cue (even a single character). Reliable because
     every single check is explicitly triggered by a message; nothing
     depends on timing or on Claude choosing to keep going.
   - **Autonomous loop (Workflow step 8, Mode B), experimental:** once the
     candidate starts it, Claude keeps polling the page on its own (read,
     wait, read again) within one continuous run, with the candidate just
     clicking through postings and sending nothing in between, until a stop
     condition is hit. This is new and unproven here: an earlier version of
     this Skill assumed autonomous checking worked and it didn't hold up
     live. Treat it as an experiment: if it stalls or misses postings, say
     so and drop back to Mode A for the rest of the page rather than
     troubleshooting live. Mode A stays fully supported regardless of how
     Mode B performs.
   Whichever mode: moving to a **new page** always requires the candidate to
   navigate there themselves first and let it load, then explicitly tell
   Claude (e.g. "page 2 now") before it reads or resumes a loop. Claude
   never follows pagination on its own, even if it notices the candidate
   clicked "Next". A loop only ever covers the single page it was started on.
4. If asked to do anything more autonomous than the patterns above
   (auto-scroll, click through search results itself, open postings one by
   one itself, follow pagination on its own, iterate a list of URLs),
   decline and point back to this rule. Suggest the candidate navigate
   manually instead.
5. Extract job posting data only. Do not record or repeat other members'
   personal details that may appear on the page (names/photos of connections,
   who posted the job, "X people clicked apply," recruiter profiles). Job
   title, company, location, and posting content are in scope; people are not.

## Before reading anything

Read profile/preferences.md for target roles, dealbreakers, compensation
range, and industries, the same source job-discovery and job-fit-scorer use.

## Workflow

1. Confirm the candidate has a LinkedIn tab open and Claude in Chrome can see
   it (ask for tab context if unclear). Confirm whether it's a search-results
   page or a single job posting page; the two are handled differently below.
2. Use read_page or get_page_text on that tab only. Do not navigate.
3. **Search-results page:** extract each visible posting card: Title,
   Company, Location/work mode, Easy Apply (yes/no), posted date or applicant
   count if shown, and the job's URL or job ID if visible in the DOM. LinkedIn
   cards do not usually show pay: write "not listed," never guess. If a
   title doesn't look like a standard job posting (an open call for
   founders, a cohort/program application, a waitlist), keep it per the
   ambiguous-title rule below but flag it explicitly rather than presenting
   it as an ordinary posting. The candidate should decide whether it counts.

   Before running a card through anything below, check it against
   `scans/seen-postings.md` (persistent across days, see "Seen-postings
   index" below). If its job ID (or, lacking one, a fuzzy title+company
   match) is already in that index, don't re-run the prefilters or scoring.
   List it with a short reference instead, e.g. "<company>, Senior PM,
   already seen <date> as #3, Apply (75), see that day's scan file," and
   move on without spending further time on it. Only cards not already in
   the index go through steps 4-7 below.
4. **Title/keyword prefilter, before anything else drops or gets shown:**
   compare each posting's title (and company, for the two exceptions below)
   against the "LinkedIn Title Filter" section of profile/preferences.md.
   Drop it silently, don't list it, don't spend any further time on it, if
   either:
   - it contains an excluded seniority word (Junior, Associate, Intern,
     Entry-level, Coordinator, Trainee, or others listed there), or
   - it doesn't reasonably match any of the included role keywords (Product
     Manager, Head of Product, Director of Product, Business Developer /
     Business Development, Product Owner, VP Product, Chief Product
     Officer, or others listed there).

   Two named exceptions to the seniority-word check, both from
   preferences.md, check company name first:
   - **Consultancy exception:** at a consultancy on preferences.md's list,
     "Associate" and "Coordinator" no longer count as exclusion words
     (Intern/Entry-level/Trainee/Graduate still do). The role-keyword match
     still applies.
   - **Marquee-company exception:** at a company on preferences.md's marquee
     list (named global payments/fintech brands, or any fintech
     headquartered in the target city, named or not), no seniority word
     counts as an exclusion reason at all. The role-keyword match still
     applies, this only waives seniority. Don't hardcode the company names
     here; preferences.md is the source of truth and gets edited
     independently of this file.

   This is a title-text match only, not a fit judgment. When a title is
   genuinely ambiguous (e.g. a title preferences.md doesn't clearly cover),
   keep it rather than guess. The point is to skip obviously irrelevant
   postings before spending any time on them, not to do fit scoring here.
5. Apply the same lightweight dealbreaker prefilter job-discovery uses on
   whatever survived step 4: drop postings with an explicit native/bilingual
   French requirement or an explicit Canadian citizenship/PR requirement, if
   that's visible on the card. Do not attempt full scoring here, that's
   job-fit-scorer's job. When unsure, keep it in the list.
6. Assign each surviving posting the next number in today's running count
   (see "Numbering and the scan file" below; do not restart at 1 per page,
   carry the count forward from wherever the last page left off), then
   return results in the same compact format job-discovery uses plus that
   number, so the two sources still merge cleanly:

   `#N | Title | Company | Location/mode | Pay (if listed) | URL`

   Limit to what's actually visible on the current page. Do not pad the list
   or infer postings that aren't rendered. Mention how many were filtered out
   by the title prefilter (e.g. "6 shown, 4 filtered as junior/off-target")
   so the candidate can sanity-check the filter isn't hiding something it
   shouldn't. Append every listed row to the scan file, whether or not it
   ever gets scored. That's what lets the candidate reference "#14" later
   even for one they haven't opened yet.
7. **Single posting page (scored one at a time, or via the cued loop
   below):** opening a posting is two clicks on LinkedIn's side, not one.
   The candidate clicking a card only opens it partially; a second click on
   "...more" (or however LinkedIn's expand control currently reads) is
   what actually reveals the full description. Before scoring, check
   whether the text looks truncated: it ends in an ellipsis, a "...more" /
   "see more" toggle is visible and still collapsed, or the description is
   noticeably shorter than a normal posting. If so, say so and ask the
   candidate to click that expand control before scoring, rather than
   scoring on partial text. This is still the candidate's own click on
   LinkedIn's UI, no different from them opening the card in the first
   place; Claude still isn't clicking anything itself. Once the full text is
   actually there, extract it (title, company, location, description,
   requirements) and run it through job-fit-scorer, exactly as if the
   candidate had pasted the text themselves.
   Output depends on context:
   - Asked to score one specific posting on its own: give the full
     breakdown, same depth job-fit-scorer normally produces.
   - Inside the cued loop, or scoring several in a row: give the short
     form only, `#N | Title | Company | Score/100 | Apply/Maybe/Skip`,
     and offer the full breakdown for any specific number on request rather
     than producing it unprompted for every posting. This is a display
     choice only, job-fit-scorer's underlying scoring is identical either
     way.
   Either form gets appended to that posting's row in the scan file (score,
   recommendation, and the full posting text, so bulk-apply can reuse it
   without going back to LinkedIn).
8. **Loop modes, for browsing many postings on one page:** two modes exist.
   If the candidate just says "start the loop" without naming one, ask which
   they want (both are worth naming explicitly while Mode B is still
   experimental) unless they've already stated a standing preference this
   session.

   **Mode A, Cued loop (proven):** the candidate says something like "start
   the cued loop, I'll cue you after each one" once they're ready to browse
   a page's postings one by one.
   - The candidate clicks a card in LinkedIn's own UI, then a second click on
     "...more" to reveal the full description (step 7: two clicks, not
     one), then sends Claude a short cue. The cue can be as small as a
     single character ("n", ".", anything short). Its content doesn't
     matter, only that it arrives, since that's literally what triggers
     the next check. Skipping the cue for a posting means that posting
     doesn't get read, full stop. There's no background process that
     will catch it later.
   - On each cue, read the current panel. If it's the same posting already
     scored, there's nothing new: say so briefly rather than re-scoring
     it, in case the candidate cued before actually clicking a new one.
   - If it's a new posting, check it's not still truncated (step 7's
     truncation check). If it is, say so and wait for the next cue rather
     than scoring partial text; the candidate likely hasn't clicked
     "...more" yet for this one.
   - Once the full text is there, read and score it (step 7's short form:
     number, title, company, score, recommendation), append it to the scan
     file, and print the one-liner. Never click, never scroll, never
     navigate to fill any gap. Waiting for the next cue is always the
     right move, not guessing at what changed.
   - **Every response in this loop ends with an explicit "→ ready for
     next" line** (or "→ still waiting, click '...more'" for the
     truncated case, or "→ nothing new yet" for the already-scored case),
     so the candidate has an unambiguous signal for when it's safe to click
     the next card and send the next cue, rather than guessing from whether
     the response looks finished.
   - The loop ends when the candidate says to stop.

   **Mode B, Autonomous loop (experimental, added at the candidate's
   explicit request to test whether it can work at all):** the candidate
   says something like "start the autonomous loop" and then just keeps
   clicking card, then "...more" (step 7), at their own pace, sending Claude
   nothing in between.
   - Claude keeps this one continuous turn going by itself: read the
     current panel; if there's nothing new or it's still truncated, wait
     roughly 5 seconds (a plain timed wait) and read again, repeating
     this read-wait-read cycle on its own, without ending the turn or
     waiting for a message from the candidate, until a new, complete
     posting shows up.
   - When a new, non-truncated posting appears, score it (step 7's short
     form), append it to the scan file immediately (not batched for
     later, so nothing already scored is lost if the loop stops
     unexpectedly), print the one-liner, and go straight back into the
     read-wait-read cycle. No message from the candidate is needed to
     continue.
   - Safety cap so this can't run unbounded: stop and report a summary
     (ranked so far, which numbers are still open) after roughly 8 minutes
     of continuous polling or about 90 read cycles, whichever comes first.
     The candidate can just say "keep going" afterward to start a fresh
     autonomous run. Numbering continues from the scan file, nothing
     restarts from scratch.
   - Also stop and report if many consecutive reads in a row show no
     change at all. That most likely means the candidate paused or moved
     to something else, not that anything is broken.
   - This mode is genuinely untested here. An earlier version of this
     Skill assumed autonomous checking worked and it didn't hold up live:
     it kept stalling and needing a manual nudge. If that happens again
     (the loop stops early, or postings still get missed), say so plainly,
     don't pretend it's still watching, and suggest switching to Mode A for
     the rest of the page rather than troubleshooting live.
   - Same ToS boundary as always: purely repeated read-only calls on the
     tab the candidate is already driving with their own clicks, never a
     navigation, click, or scroll performed by Claude. Continuous unattended
     reads are a slightly different posture than reads triggered one at a
     time by a message, which is exactly why this is a flagged experiment
     rather than a quiet default.

   Either mode ends when the candidate says to stop, or (Mode B only) when
   its safety cap is hit. Neither mode continues onto a new page by itself;
   see the pagination rule in Hard Constraints #3. When the candidate has
   navigated to a new page and told Claude, the loop (whichever mode)
   resumes fresh on that page with the same mechanics, still numbering
   forward.

## Numbering and the scan file

Every *new* posting (one not already in the seen-postings index below) gets
a number, and gets written to a scan file at `scans/<YYYY-MM-DD>-linkedin.md`
(one file per calendar day; create the `scans/` folder and file if they
don't exist yet for today). Numbering starts at 1 for the day and counts up
across every page read that day, it never resets per page. Postings the
seen-postings index already covers aren't renumbered, just referenced back
to whichever day and number originally captured them.

For each posting, the scan file holds: its number, title, company,
location/mode, whether it was Easy Apply, the URL if one was ever visible,
and, once scored, the score breakdown, recommendation, and the full
posting text as read. This file is the lookup the candidate uses later to
say "apply to #1, #5, #12" (see skills/bulk-apply/SKILL.md) without Claude
needing to go back to LinkedIn for anything already captured.

On request, print the day's scan file back as one ranked table (#, Title,
Company, Score, Recommendation) sorted highest score first, skipping rows
that were listed but never actually scored (mark those "not yet scored"
rather than omitting them silently, so the candidate knows they're still
there).

## Seen-postings index (cross-day dedup)

`scans/seen-postings.md` is a second, persistent file. Unlike the daily
scan file, it's never dated and never reset. The candidate doesn't run this
every day, so a search they revisit later would otherwise get re-scored from
scratch, burning real time and tokens on postings already fully evaluated.

Whenever a posting gets listed or scored into today's scan file, add or
update its row in this index too: key (job ID from the URL, or a
`(fuzzy)`-flagged title+company match if no ID is visible), title, company,
first-seen date, which day's scan file has the full detail, its number that
day, and its score/recommendation once scored. Step 3 above checks this
index first for every card on a page. Anything already in it gets
referenced, not re-processed, unless the candidate explicitly asks for it to
be looked at again (in which case, treat it as a fresh single-posting read
and update the index entry with whatever's found).

## Honesty

If the page hasn't finished loading, is mostly blank, or LinkedIn is showing
a login/verification wall, say so plainly rather than inventing postings or
retrying automatically. If the candidate is on a page this Skill can't make
sense of (feed, messaging, a profile), say so and ask them to navigate to a
job search or job posting page instead. If the autonomous loop (Mode B)
stops short of its stated safety cap, or seems to have quietly stopped
watching, say so directly rather than letting the candidate assume it's
still running.

## Relationship to job-discovery

This supplements job-discovery, it doesn't replace it. Public-board coverage
(Built In, Job Bank Canada, Wellfound, the other tiered boards, company
pages) stays with the job-discovery subagent, which can still run
unattended. LinkedIn coverage only ever happens this way: interactively, one
page at a time, with the candidate driving. Indeed moved out of
job-discovery (it blocks automated access) and is now covered by its own
sibling Skill, `skills/indeed-discovery/`, built on this same
human-supervised pattern with `I-N` numbering and its own
`scans/<date>-indeed.md` file. Both Skills share `scans/seen-postings.md`;
Indeed rows there are keyed `indeed:<jk>`, so this Skill's fuzzy
title+company check will also recognize a LinkedIn card already seen on
Indeed.
