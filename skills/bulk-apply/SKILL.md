---
name: bulk-apply
description: Generate tailored CV + cover letter application packages for multiple scanned postings at once, referenced by the numbers linkedin-discovery (#N) or indeed-discovery (I-N) assigned them. Use when the candidate says something like "apply to #1, #5, #12", "apply to I-3, I-7", or "generate applications for these numbers."
---

# Bulk Apply

Turns a set of numbers from a linkedin-discovery or indeed-discovery scan
into finished application packages, without re-reading anything from
LinkedIn or Indeed: everything needed is already saved in that day's scan
file.

## Before doing anything

1. Pick the scan file by each number's form:
   - plain `#N` (or a bare number): linkedin-discovery's
     `scans/<YYYY-MM-DD>-linkedin.md`;
   - `I-N`: indeed-discovery's `scans/<YYYY-MM-DD>-indeed.md`.
   A single batch may mix both, for example "apply to #4, I-2, I-9"; resolve
   each number against its own file. If the candidate references a number
   but doesn't say which date, assume today's file; if there's no matching
   scan file for today, ask which date they mean rather than guessing or
   scanning every file in the folder.
2. Look up each number the candidate gave in that file. If a number isn't
   there, or is listed but was never actually scored (no full posting text
   saved), say so plainly and drop it from the batch rather than inventing
   content or guessing which posting they meant.

## Confirm before generating anything

Show the resolved list back to the candidate before creating any files
(number, title, company, score if it has one) and wait for an explicit
go-ahead. This is deliberate: it's the one chance to catch a typo'd number
before it spends a CV/cover-letter generation on the wrong posting.
Something like:

> About to generate CV + cover letter for:
> #1, <company>, Senior Product Manager (84/100, Apply)
> #5, <company>, Senior Product Manager, New Products
> #12, <company>, Digital Product Manager
> I-3, <company>, Senior Product Manager (Indeed)
>
> Proceed?

Only move on once the candidate confirms. If they want to drop or add one
at this point, adjust the list and confirm again rather than proceeding on
the old list.

## Essential-skills gap check

Before Phase 1 drafting starts on any confirmed posting whose scan-file
entry carries job-fit-scorer's "Essential-skills gap" flag (see
job-fit-scorer's SKILL.md), stop and show the candidate that posting's own
Essential-requirements language verbatim for the flagged item(s), quoted
from the saved posting text, and ask for an explicit yes on that posting
specifically before drafting it. This is separate from, and in addition to,
the resolved-list confirmation above: that step catches typo'd numbers,
this step is a real fit judgment and needs its own answer.

Other postings in the same batch that don't carry the flag proceed straight
to Phase 1 as usual; only the flagged ones pause here. If the candidate says
no to a flagged posting, drop it from the batch (log it to
`applications/LOG.md` as Dropped, per application-tracker's schema, with the
Essential-skills gap as the stated reason) and continue with the rest of the
batch.

This check is specific to the drafting gate. It never applies to
linkedin-discovery's or indeed-discovery's own scanning and scoring pass,
which keeps running exactly as before, uninterrupted, one card at a time.

## Generating each package, two phases, never skip the pause

Content approval always comes before any docx/pdf gets built, no exceptions
for batch size or momentum. This rule exists because a 5-application batch
once went straight from scoring to finished docx/pdf with no markdown-review
pause, when the candidate wanted the chance to read and comment before files
got built, consistent with what cv-tailor and cover-letter-tailor already
say on their own: "ask before converting to docx". Bulk-apply's job is to
make that gate work across a whole batch, not to route around it for speed.

**Phase 1, draft only, for every confirmed posting:**

For each confirmed posting, in order, using the posting text saved in the
scan file (not a fresh LinkedIn or Indeed read):

1. Run cv-tailor against it, same as a normal single application, stopping
   at the markdown draft (do not ask cv-tailor to convert to docx yet).
2. Run cover-letter-tailor against it the same way, markdown only.
3. Create `applications/<company-role-slug>/` with `cv.md` / `cover-letter.md`
   (source markdown) and `job-posting.txt` (posting text, URL if the scan
   file has one, the score/recommendation from scoring, and any gaps
   flagged during tailoring). For Indeed postings, also carry over the scan
   file's source flags (`est. pay`, `agency / employer undisclosed`,
   `French-first`) and, when recorded, the company's own posting URL as the
   preferred apply link over the Indeed link. Do not create the docx/pdf pair yet.
4. If `applications/<company-role-slug>/` already exists, ask before
   overwriting it rather than silently replacing an existing application.

Work through the list one posting at a time rather than trying to batch the
generation steps together. If one posting fails (missing data, a tailoring
error), report that clearly and continue with the rest rather than
abandoning the whole batch.

When every posting in the batch has a markdown draft, stop and tell the
candidate plainly that the batch is drafted and waiting on their review. Do
not proceed to Phase 2 on your own judgment, even if the content looks solid.

**Phase 2, only after the candidate reviews:**

If the candidate comes back with no comments (or says something like "looks
good, go ahead"), convert every drafted posting in the batch to docx/pdf per
cv-tailor's and cover-letter-tailor's output filename convention:
`CV_<First>_<Last>_<Company>_<Month>_<Year>.docx`/`.pdf` and
`CL_<First>_<Last>_<Company>_<Month>_<Year>.docx`/`.pdf`. Render each to
PDF/JPEG and visually check before presenting as done, per the shared docx
formatting standard.

If the candidate comes back with edits instead, apply them to the markdown,
show the updated draft, and wait for confirmation again before converting
that posting to docx. A batch can move to Phase 2 posting by posting rather
than all at once if some postings clear review before others.

Once a posting's docx/pdf is built and confirmed, add a row to
`applications/LOG.md` following application-tracker's schema (see
`skills/application-tracker/SKILL.md`): status "Applied," Date Applied =
today, Follow-up Due = today + 7 days, Score/Title/Company from the scan
file entry. Skip this only if the folder already existed and the candidate
declined the overwrite back in Phase 1.

## Wrap-up

After Phase 1, confirm which postings are drafted and waiting on review.
After Phase 2, end with a short summary: which folders got finished docx/pdf
files, which (if any) were skipped and why, and confirm `applications/LOG.md`
was updated for each one created. Same rule as every other part of this
project: nothing gets submitted anywhere automatically. This only produces
the materials; the candidate still submits each application themselves.
