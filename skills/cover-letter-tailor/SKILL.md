---
name: cover-letter-tailor
description: Write a tailored cover letter for a specific job posting, based on the candidate's CV and preferences. Use when the candidate asks to write, draft, or tailor a cover letter for a job.
---

# Cover Letter Tailor

## Before writing

Read profile/base-cv.md and profile/preferences.md (especially the Domain Philosophy note) before writing anything. Also read the job posting provided in the conversation.

## Ground rules

1. Never invent experience, numbers, or claims not in profile/base-cv.md.
2. Address the company and role by name. No generic "Dear Hiring Manager" filler beyond what's necessary if no contact name is available.
3. Keep it to 3-4 short paragraphs, under 350 words total. Recruiters skim.
4. Do not restate the CV. Each paragraph should add context, reasoning, or a story the CV bullets can't fully carry on their own.
5. No upfront weakness disclosure (see preferences.md): the letter's only job is to get the candidate called for an interview. Never name a gap, weakness, or missing qualification anywhere in the letter, not even as a setup for a positive pivot (no "I don't have X, but I do have Y," no "I should be direct about one gap"). Present only the strongest, truest case for why they fit. A real gap still gets surfaced, just never in the letter, see "Gap flagging" below.
6. Never write to profile/base-cv.md or profile/preferences.md while drafting. These are read-only source-of-truth inputs for every drafting run; drafting only ever produces or edits files inside applications/<company-role>/. (See cv-tailor/SKILL.md's Ground rule 6 for the full incident: a tailoring run invented a false extended timeline for a <past employer> and wrote it directly into base-cv.md and preferences.md, corrupting the ground-truth files every other application depends on. If base-cv.md's own wording seems ambiguous or in need of reconciling, do not invent a resolution and do not edit base-cv.md or preferences.md to "fix" it. Flag the ambiguity to the candidate directly and leave the source files untouched.)

## Structure

1. Opening: Name the role and company, and state the single strongest reason the candidate fits, ideally the one closest to the posting's core need. If there's an industry/domain mismatch between the posting and the candidate's direct experience, use the Domain Philosophy from preferences.md to frame their cross-domain track record (<domain A> to <domain B> to <domain C>) as evidence of adaptability, standing on its own as a strength. Never introduce this by naming the mismatch or gap first (see Ground rule 5). Lead straight with the positive framing.
2. Proof paragraph: One concrete story (not a bullet list) that demonstrates the fit, over a generic list of skills. Actively weigh the full available story pool before defaulting to the same one: the CV bullets plus the fuller background stories in preferences.md that are marked available for cover-letter use (for example: a from-scratch partnership story, a first-client commercial/pricing story, an internal-user-discovery story for internal-tooling postings, a client-service-depth note for customer-facing asks, a hands-on technical story for technical-fluency asks, the Job Search Agent story for AI-fluency asks). Choose whichever story most directly maps to what THIS posting specifically emphasizes, not the story used most recently. A strong, versatile default story is fine to keep winning on its own merits, just make sure it won it, rather than skipping the comparison.
3. Culture/motivation paragraph: Briefly say why this company/role specifically (use real details from the posting: mission, product, growth stage), not generic enthusiasm.
4. Close: Confident, brief call to action. No begging, no "I hope to hear from you soon" filler.

If a posting has a requirement the candidate's real experience doesn't clearly support and there's no genuine, truthful strength to lead with instead, simply don't address that requirement in the letter at all. Silence on an unmet requirement is always preferable to naming it. This sometimes means a letter has only 3 solid paragraphs instead of the full structure above, that's fine, a shorter letter beats one that talks the candidate out of an interview.

## Blending experience accurately

The same rule that governs cv-tailor's Profile paragraph applies here, especially in the opening paragraph, which also compresses multiple roles into a few sentences. Trace every clause back to a specific sentence in profile/base-cv.md before writing it; don't add scope (a region, a scale, an outcome) that isn't literally stated there. When a proof-paragraph story references a specific deal or initiative (e.g. a joint venture that "didn't reach commercial launch" and only validated the business model and technical feasibility), keep any material caveat from the source bullet intact. Don't let a tighter, punchier sentence quietly drop the part that makes it accurate. See cv-tailor/SKILL.md's "Profile paragraph rules" for the full pattern of what went wrong and why.

## Tone

Direct, confident, and specific, mirroring the candidate's own communication style. Avoid corporate cliches ("synergy," "results-driven," "team player," "spearheaded," "leveraged," "passionate about," "seamlessly," "robust," "cutting-edge"). Avoid over-explaining; trust the reader.

If preferences.md notes that the candidate is a fluent but non-native English speaker, the letter should sound like them: clear, direct, and plainly worded, not like a native-level, idiomatically polished voice. Simpler sentence structures and plainer vocabulary are preferred over "impressive" phrasing that doesn't match how they actually write or speak. This is about authenticity, not correctness; grammar stays clear and professional, the letter just shouldn't reach for polish the candidate wouldn't naturally use.

## Gap flagging

If the posting has a requirement the CV/letter doesn't clearly support, never address it in the letter itself (see Ground rule 5 and Structure above). Instead mention it to the candidate directly after drafting, and it should already be captured in that application's job-posting.txt, so they can decide whether to raise it themselves in an interview, entirely separate from what gets sent to the employer.

## Writing style rules

- Never use the em dash (U+2014) or en dash (U+2013) as a sentence-connecting device (e.g., "did X, dash, which led to Y"). Rewrite as two sentences, or use a comma, colon, or "and"/"which" instead. This applies to all body content in bullets, summaries, and cover letter paragraphs.
- Date ranges are the one exception: en dashes are fine there (e.g., "Mar 2022 – Oct 2024"), since that's standard resume convention, not the same issue.
- Never claim the candidate personally built visual designs/prototypes in Figma; they direct a designer and review/comment in Figma themselves. Phrase any Figma-related bullet accordingly (e.g., "directing a designer on Figma prototypes," not "building Figma prototypes").

## Job posting record

Whenever generating application materials in an `applications/<company-role>/` folder, also create (or update, if it already exists) a file called `job-posting.txt` in that same folder, containing:
- Job Title
- Company
- Original posting URL
- Location/Work mode
- Compensation (if listed)
- Application deadline (if listed)
- How to apply (portal link, email, specific instructions, whatever the posting specifies)
- Date the posting was found/scored

Pull this information from the job-discovery or job-fit-scorer output already in the conversation. If the URL or application instructions aren't available in what's already been discussed, say so explicitly in the file rather than leaving it blank silently, and flag it to the candidate.

Only create this file once per folder; if it already exists, leave it as-is rather than overwriting.

## Application form Q&A

Some job applications include short-answer questions beyond the CV and cover letter (e.g. "Why do you want to work here?", "Describe a time you..."). When the candidate asks for help drafting these, save each question and its drafted answer to a file in that application's `applications/<company-role>/` folder:
- `short-answer.txt` if there's a single question
- `short-answers.txt` if there are multiple (list each as its own Question/Answer pair in the same file)

This preserves the drafted answer after the session ends, the same way `job-posting.txt` preserves posting details.

## Output

Produce the letter as plain text/markdown first for the candidate's approval. Ask if they want it finalized as a PDF only after they've approved the content.

## PDF formatting standard (docx is an internal build step only)

The candidate only wants the finished PDF. Build the .docx directly with the `docx` npm library (not a generic Word template) purely as an internal step to get every style detail below under precise control, convert it to PDF, then deliver and commit only the PDF, not the .docx. Don't send the .docx into the conversation or write it to the device folder; it can stay in the scratch/build directory or be discarded once the PDF is rendered.

**Shared style:**
- Font: Calibri throughout
- Colors: navy `#1F2A44` for name/headings, ink `#222222` for body text, soft grey `#555555` for meta/contact/dates
- Body text: 11pt (size 22 in docx-js half-points)
- Page: US Letter, 0.6–0.9in margins
- Must stay ATS-safe: no tables, columns, text boxes, or images
- Prevent orphaned headings: set `keepNext` (and `keepLines` for role headings) on every heading-style paragraph so it never lands alone at the bottom of a page
- Always render to PDF/JPEG and visually check before presenting as done

**Cover-letter-specific, formatted as a formal business letter:**
- Name header with contact line, navy bottom border
- Sender city + date
- Recipient block (name/team, title if known, company). If no specific contact name is available, use "[Company] Hiring Team"
- Bold subject line: "Subject: Application for [Role Title]"
- "Dear [Recipient]," salutation
- Body paragraphs
- Closing: "Sincerely," then name, then contact details on the line below

## Output filename convention (PDF only)

Every finished cover letter file is named:

`CL_<First>_<Last>_<Company>_<Month>_<Year>.pdf`

- `<Company>` is the company name as it appears on the posting (e.g. `Acme`, `ExampleCorp`), no spaces, punctuation stripped rather than replaced with underscores mid-word.
- `<Month>_<Year>` is the full month name and four-digit year the document is finalized (e.g. `September_2026`), not the posting date.
- This replaces the generic `cover-letter.pdf` name in the application folder. Keep `cover-letter.md` as the source-of-truth markdown filename inside `applications/<company-role>/` (that folder-naming convention is unchanged), but the finished PDF the candidate sends must use this naming.
- No .docx deliverable at all, only the PDF, per the candidate's explicit instruction. Don't send or commit a .docx file for any application.
