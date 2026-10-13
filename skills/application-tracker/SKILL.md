---
name: application-tracker
description: Track every job application in applications/LOG.md, record new applications, update status (applied/interviewing/rejected/offer), and check which ones are due for a follow-up. Use when the candidate wants to log an application, update an application's status, or asks what needs following up.
---

# Application Tracker

Keeps `applications/LOG.md`: the one place that records every application
actually generated from this project, its current status, and whether it's
due for a follow-up.

This is on-demand only, by the candidate's own explicit choice: nothing
here pings the candidate automatically or on a schedule, since that would
need a real always-on process this repo doesn't have (see
linkedin-discovery's Hard Constraints for why that's not something to
promise lightly). Ask "what needs follow-up" any time instead and get an
accurate answer.

## The log

`applications/LOG.md`: one row per application: Title, Company, Date
Applied, Score, Status, Last Updated, Follow-up Due, Notes. Gitignored, same
as the rest of `applications/`. This is personal job-search data, not
meant to be committed.

Status is always one of: Applied, Interviewing, Rejected, Dropped, Offer,
Withdrawn, Not Applied, Missed (posting closed).

## Adding a row

Two ways a row gets created:

1. **Automatic, via bulk-apply:** the moment bulk-apply (or a single manual
   application) creates `applications/<company-role>/`, it adds a row here:
   status "Applied," Date Applied = today, Follow-up Due = Date Applied + 7
   days (the candidate's chosen default), Score/Title/Company pulled from the
   scan file entry that generated it.
2. **Manual:** if the candidate says something like "log the <company>
   application as applied on [date]" for one that didn't go through
   bulk-apply, add the row the same way, asking for whatever's missing
   (title, company, date, score if known).

Never invent a score or date. Ask rather than guess if either is missing.

## Updating status

When the candidate says something like "mark <company> as interviewing,"
"<company> rejected me," or "got an offer from X": find the matching row (by
company, asking which one if more than one plausible match exists), update
Status and Last Updated to today. Moving to Interviewing, Rejected, Dropped,
Offer, or Withdrawn clears that row's Follow-up Due, no next-step tracking
beyond status changes right now. Any other update doesn't touch Follow-up
Due.

**Moving to Rejected or Dropped triggers two automatic cleanup steps**
(standing approval from the candidate, no need to ask each time):

1. Move the application's folder from `applications/<company-role>/` into
   `applications/REJECTED/<company-role>/` or `applications/DROPPED/<company-role>/`
   to match the new status, if it isn't already there.
2. Delete every generated-document file in that folder: PDFs (CV_*.pdf,
   CL_*.pdf, any merged CV_CL_*.pdf) and docx files (CV_*.docx, CL_*.docx,
   any merged CV_CL_*.docx) alike. Nothing else in the folder is
   touched: cv.md, cover-letter.md, job-posting.txt, hiring-review.md,
   short-answers.txt, interviewer-notes.txt, and any other source or notes
   files all stay, since they're the real source and any PDF/docx is one
   regeneration away if ever needed again.

If the folder was already moved, or the application never had a PDF (e.g.
dropped before drafting), just note that plainly, nothing to do. Always
report back exactly what was moved and what was deleted, and add a short
note to that row's Notes column recording it, so LOG.md stays a complete
record even after the PDFs are gone.

## Checking follow-ups

When the candidate asks "what needs follow-up" (or similar): list every row
still in "Applied" status whose Follow-up Due date is today or earlier,
oldest first: title, company, days since applied, days overdue. If nothing
qualifies, say so plainly rather than padding the list. This only runs when
asked; it does not run automatically at any interval, and nothing in this
repo checks it unprompted.

## Showing the log

On request, print the whole log back as a table, optionally filtered by
status (e.g. "show me everything still open" = Applied + Interviewing).

## Honesty

If `applications/LOG.md` doesn't exist yet, say so and create it (with the
header row) rather than assuming it's empty by inference. If a company name
is ambiguous against more than one row, ask which one rather than guessing.
