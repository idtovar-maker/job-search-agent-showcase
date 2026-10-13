---
name: cv-tailor
description: Tailor the candidate's base CV to a specific job posting for ATS keyword match and relevance. Use when the candidate asks to tailor, adapt, or optimize their CV for a specific job.
---

# CV Tailor

## Before tailoring

Read profile/base-cv.md (ground truth for all experience, dates, numbers) and profile/preferences.md (positioning and domain philosophy) before writing anything. Also read the job posting provided in the conversation.

## Ground rules: never violate these

1. Never invent experience, numbers, tools, or responsibilities that aren't in profile/base-cv.md. If the posting wants something the candidate's CV doesn't support, do not fabricate it. Flag it instead (see "Gap flagging" below).
2. Never change dates, company names, titles, or factual claims. Reordering, reweighting, and rephrasing are fine; changing facts is not.
3. Never reorder the sequence of roles in Work Experience. They must always appear in the same strict reverse-chronological order as profile/base-cv.md (most recent role first, by end date then start date), exactly as they appear there. This is non-negotiable even when a less-recent role is more relevant to the posting. Handle relevance by reweighting bullets (more/fewer bullets, which points get emphasized) or by what the Profile paragraph foregrounds, never by moving a role out of chronological position. Before finalizing any tailored CV, diff its role order against profile/base-cv.md's role order and confirm they match exactly. (Added after a tailored CV was caught with two roles swapped out of chronological order across three application folders in the same batch. Claude had read "reorder which roles appear more prominently" in the Tailoring process below as license to resequence roles rather than just reweight bullets within them. That phrasing is corrected below to remove the ambiguity.)
4. If the target role's industry doesn't match the candidate's direct experience, apply the Domain Philosophy from profile/preferences.md: frame the cross-domain track record (<domain A> to <domain B> to <domain C>) as a strength, not a gap to hide.
5. No upfront weakness disclosure (see preferences.md): never state a gap, weakness, or missing qualification anywhere in the CV itself, not even framed as a pivot to a strength. The CV only ever presents the positive case plus whatever an ATS needs to parse correctly. Real gaps still get surfaced, just not on the page, see "Gap flagging" below.
6. Never write to profile/base-cv.md or profile/preferences.md while tailoring. These are read-only source-of-truth inputs for every tailoring run; tailoring only ever produces or edits files inside applications/<company-role>/. (Added after a tailoring run invented an extended, false timeline for a <past employer> and wrote that fabrication directly into base-cv.md and preferences.md, corrupting the ground-truth files every other application's tailoring depends on. If base-cv.md's own wording is ambiguous or seems to need reconciling, do not invent a resolution and do not edit base-cv.md or preferences.md to "fix" it. Flag the ambiguity to the candidate directly and leave the source files untouched.)

## Tailoring process

1. Extract keywords: Pull the job posting's required/preferred skills and exact phrasing (titles, tools, methodologies, metrics language). ATS systems match literal keywords, so mirror the posting's own wording wherever it's truthfully applicable to the candidate's real experience.
2. Reorder and reweight: Reorder bullets *within* each role to foreground whatever is most relevant to this specific posting. Do not delete roles entirely; reduce their bullet count instead if less relevant. Do NOT reorder the roles themselves: the sequence of roles in Work Experience must stay in strict reverse-chronological order, matching profile/base-cv.md exactly (see Ground rule 3). "Prominence" for a less-recent-but-more-relevant role comes from bullet count and what the Profile paragraph foregrounds, never from moving it up the timeline.
3. Adjust the summary: Rewrite the top "Profile" paragraph to speak directly to this role and company, using 1-2 keywords/phrases from the posting naturally. See "Profile paragraph rules" below before writing it.
4. ATS-safe formatting: No tables, columns, text boxes, or graphics. Standard section headers (Profile, Core Competencies, Work Experience, Education). Plain bullet points.

## Profile paragraph rules

Added after repeated corrections on one application. The Profile paragraph blends achievements from several different roles into a few dense sentences, which is exactly where inaccuracy creeps in fastest. What went wrong there, twice, is the template for what to avoid:

**Do:**
- Trace every clause in the Profile paragraph back to a specific sentence in profile/base-cv.md before writing it. If you can't point to the exact source sentence a claim comes from, don't write it.
- When a sentence blends two facts from two different roles or timeframes, write them as two separate sentences, not one joined by a semicolon. A semicolon reads as "these two things are directly connected"; if they're actually unrelated facts about different roles, that connection is misleading even when both halves are individually true.
- Keep the Profile to broad, simple, well-supported claims. A specific initiative with a nuanced outcome (a deal that validated a model but didn't reach commercial launch, a pivot with a caveat, a partial result) belongs in its Work Experience bullet, where there's room for the nuance, not compressed into a single profile clause that can't carry it.
- When in doubt, cut the detail rather than keep it "for color." A geography, a scale qualifier, or a scope word that isn't load-bearing for the sentence's point is a place embellishment sneaks in; if it doesn't change what the reader understands, it doesn't need to be there.

**Don't:**
- Don't add scope that isn't in the source: a region, an adoption scale, a client count, anything that sounds plausible but isn't literally stated in the base-cv.md bullet. (Real example: "adopted by multiple banks and financial institutions" got rewritten as "...across <region>", a region base-cv.md never claims, and the real answer, per the candidate directly, is one country only.)
- Don't reference a specific deal, venture, or initiative vaguely, without naming what it is. If a reader (including the candidate) can't tell which achievement is being described from the sentence alone, that ambiguity is a problem on its own, independent of whether the compressed version is even accurate. If it can't be named concretely in one clause, leave it out of the Profile and let the full Work Experience bullet carry it.
- Don't silently drop a material caveat when compressing a base-cv.md bullet (e.g., dropping "didn't reach commercial launch" from a joint venture with a <partner company>; this pattern recurred across five files in one batch before being caught).

## Gap flagging

After producing the tailored CV, list any requirements from the posting that the candidate's CV doesn't clearly support. Do not silently drop this step, and never put it in the CV itself (see Ground rule 5). Tell the candidate directly, in the conversation and in job-posting.txt, so they can decide whether to raise it themselves in an interview, or not at all. The cover letter is not an option for addressing it either, since cover-letter-tailor also only presents strengths.

## Writing style rules

- Never use the em dash (U+2014) or en dash (U+2013) as a sentence-connecting device (e.g., "did X, dash, which led to Y"). Rewrite as two sentences, or use a comma, colon, or "and"/"which" instead. This applies to all body content in bullets and summaries.
- Date ranges are the one exception: en dashes are fine there (e.g., "Mar 2022 – Oct 2024"), since that's standard resume convention, not the same issue.
- Never claim the candidate personally built visual designs/prototypes in Figma; they direct a designer and review/comment in Figma themselves. Phrase any Figma-related bullet accordingly (e.g., "directing a designer on Figma prototypes," not "building Figma prototypes").

## Core Competencies category rule

**Languages line:** base-cv.md's Core Competencies ends with a Languages line. Keep it in every tailored CV; a multilingual profile is a real advantage in some markets, and some languages are a scoring positive (see preferences.md). Never upgrade a language level beyond what base-cv.md states. Core Competencies uses standard recruiter/ATS category names (Product Strategy, Delivery & Agile, Tools, Domains, Design & UX, Languages); don't reintroduce custom labels like the retired "Creative-Tech Sync". When a Core Competencies line isn't earning its place, the general rule still applies: replace it with a category that's well-evidenced elsewhere in base-cv.md's Work Experience (e.g. "Industries: <industry list>," drawn directly from the Work Experience history) rather than inventing filler. If nothing in the CV actually supports a good replacement category, drop the line entirely instead of keeping a weak one just to fill five bullet categories.

## Voice and authenticity

The candidate asked for output that sounds less "AI-produced." Two things, both permanent:

- Avoid generic AI-résumé buzzwords and filler: "spearheaded," "leveraged," "proven track record," "results-driven," "dynamic," "cutting-edge," "seamlessly," "robust," "passionate about," "synergy," "utilize" (say "use"), "in order to" (say "to"). Prefer plain, concrete verbs.
- Don't force every bullet into an identical "Verb + object, result metric" mold just for symmetry. Real writing varies in length and structure. Uniform, textbook-perfect parallelism across every line is itself a tell.
- If preferences.md notes that the candidate is a fluent but non-native English speaker, the CV should read as clear, confident professional English, not as a native-level, idiomatically polished voice. Plainer word choices and simpler sentence construction are preferred over "impressive" vocabulary that doesn't match how they'd actually write. This is about authenticity, not lowering the bar: grammar stays correct and meaning stays clear, the writing just shouldn't reach for flourish it wouldn't naturally have.

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

Produce the tailored CV as a complete markdown document. Ask the candidate whether they want it finalized as a PDF once they've approved the content. Don't generate the file before content approval.

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

**CV-specific:**
- Name: bold, 20pt
- Section headings: bold, uppercase, 11pt, thin navy bottom border
- Role headings: bold title + company left-aligned, dates right-aligned on the same line via a right tab stop, italic dates in soft grey
- Real bullet-list formatting (numbering config, not literal dashes/asterisks)

## Output filename convention (PDF only)

Every finished CV file is named:

`CV_<First>_<Last>_<Company>_<Month>_<Year>.pdf`

- `<Company>` is the company name as it appears on the posting (e.g. `Acme`, `ExampleCorp`), no spaces, punctuation stripped rather than replaced with underscores mid-word.
- `<Month>_<Year>` is the full month name and four-digit year the document is finalized (e.g. `September_2026`), not the posting date.
- This replaces the generic `cv.pdf` name in the application folder. Keep `cv.md` as the source-of-truth markdown filename inside `applications/<company-role>/` (that folder-naming convention is unchanged), but the finished PDF the candidate sends must use this naming.
- No .docx deliverable at all, only the PDF, per the candidate's explicit instruction. Don't send or commit a .docx file for any application.
