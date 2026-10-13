# Job Search Agent

A personal job-search system I designed and built with Claude Code. It finds job postings, scores each one against a fixed rubric, drafts a tailored CV and cover letter, runs an independent review of the drafts, and tracks every application. A human approves every step that matters: nothing is submitted, built as a final PDF, or sent without my explicit go-ahead.

I built it for my own search in Montreal, as a product: a real user (me), real constraints, and a lot of iteration on what broke.

## What it does

1. **Discovery.** A subagent searches public job boards (general Montreal boards, startup boards, VC portfolio boards, sector boards), removes duplicates against everything already seen or applied to, and writes a short unscored list to a dated scan file. Two sites that do not allow automation are handled differently: separate skills read only the page I already have open, and never log in, navigate, or click.
2. **Scoring.** A skill scores each posting out of 100 against my profile: skills and experience (40), seniority fit (20), preferences such as location, salary, and industry (25), and dealbreakers (15, or a hard cap when one is violated). Every score comes with a rationale that cites the posting and the profile, including what is missing.
3. **Essential-skills gate.** If a posting's own *required* items are not covered by my profile, the score carries a visible flag, and drafting stops until I see the posting's exact wording and say yes.
4. **Tailoring.** Two skills draft the CV and the cover letter. They can reorder, reweight, and rephrase, but they can only use facts that exist in two source-of-truth files. They never add experience, numbers, titles, or scope.
5. **Hiring review.** A separate subagent reads the finished package as a skeptical member of the target company's hiring team. It checks every claim against the source files, quotes both sides of any discrepancy, and returns findings ranked must-fix or nice-to-fix, plus positioning notes. It does not rewrite anything; that stays with the tailoring skills.
6. **Tracking.** A skill keeps the application log current, lists follow-ups when asked, and archives closed applications.

## Design choices

- **Two files are the ground truth.** All facts about the candidate live in a CV file and a preferences file. Every generated document must trace each claim back to a sentence in one of them. Tailoring runs can read these files but never write to them.
- **Verification before writing.** The review step must re-read the source file before quoting it, because quotes rebuilt from memory were sometimes attributed to the wrong document.
- **Caveats survive compression.** When a long bullet is shortened, any limiting detail (for example, "didn't reach commercial launch") must stay. This rule exists because compressed versions kept dropping exactly that part.
- **Human in the loop at the right points.** The agent runs research and drafting unattended, and stops before anything irreversible: drafting past a skills gap, building final documents, or submitting.
- **One application at a time.** Final documents are never batch-built, so every package gets its own review before anything is finalized.
- **Style rules as checks.** Banned buzzwords and certain punctuation are checked on every output, not left to taste.

## What I learned building it

- **Copies drift.** Claude Code loads skills from one folder, and I edited them in another. A stale copy reintroduced outdated wording into generated documents. Now every skill edit is mirrored and checked.
- **The model is good at sounding right.** The most dangerous errors were plausible: a region added to a claim, a date range "corrected" into something false. The fix was structural (ground-truth files, traceability, an isolated reviewer), not a better prompt.
- **Scoring needs to be honest to be useful.** A scorer that flatters every posting wastes application effort. The rubric is tuned to call out gaps and stretches.

## Structure

```
skills/
  job-fit-scorer/        score a posting, set the essential-skills flag
  cv-tailor/             tailored CV, formatting standard, naming
  cover-letter-tailor/   tailored cover letter
  bulk-apply/            pick postings by number, gate, then draft
  application-tracker/   application log and follow-ups
  linkedin-discovery/    read-only reading of a page already open
  indeed-discovery/      same pattern for another board
.claude/agents/
  job-discovery.md       unattended public-board search
  hiring-reviewer.md     isolated, skeptical review of one package
profile/                 (not included: personal data)
applications/            (not included: personal data)
scans/                   (not included: personal data)
```

Personal data, generated documents, scan results, and the application log are excluded from this repository. The skills refer to the profile files by path; `profile/*.example.md` shows their shape with placeholder content.

## Built with

Claude Code (skills and subagents), the Claude API, Node.js (the `docx` library) and LibreOffice for the PDF build, and a Cowork session bridged to my own computer for file access.

Iván Tovar, Montreal
