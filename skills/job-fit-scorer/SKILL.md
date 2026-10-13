---
name: job-fit-scorer
description: Score a job posting against the candidate's profile (CV + preferences) and recommend apply/maybe/skip. Use whenever the candidate pastes a job posting, a job URL with its text, or asks to evaluate/score a job for fit.
---

# Job Fit Scorer

You are scoring job postings for the candidate, who is targeting Senior Product Manager, Business Developer, or hybrid Product/BD roles (e.g. Head of Product) at scale-up tech companies in <target city>. The candidate also targets mid-level Product Manager (including PM II) and mid-level BD / Partnerships Manager roles, plus contract roles (see preferences.md, "Mid-level widening").

## Before scoring

Read profile/base-cv.md and profile/preferences.md in this repo before scoring anything. Do not ask the candidate to repeat their background, experience, or preferences, they are already in those files. If either file is missing or still contains placeholder text, tell the candidate and stop.

## Scoring rubric

For each job posting, evaluate against these four weighted categories:

1. Skills & experience match (0-40 points)
Compare the posting's required/preferred qualifications against the candidate's CV: product management experience, business development track record, domain knowledge, team leadership scope, tools (Jira, Figma, Amplitude, etc.), and the certifications listed in base-cv.md.

2. Seniority & role fit (0-20 points)
Does the posting's seniority level fall within the candidate's target range? The range runs from mid-level up to Head of Product / GM scope (preferences.md, "Mid-level widening"):
- Head of Product, Director, Principal, Staff, Lead, and Senior PM roles: score against the candidate's <years> of experience and Head of Product / GM-level scope.
- Mid-level Product Manager, PM II, plain "Product Manager", and mid-level BD / Partnerships Manager roles with a strategic or product component: in range. Do not mark these down for being below the candidate's past scope; score 14 to 20 on how well the scope fits.
- Junior, associate, entry-level, analyst, or coordinator roles (outside the consultancy and marquee-company exceptions in preferences.md) stay out of range and score low.
- Still penalize junior/mid-level individual-contributor sales roles mislabeled as "Business Development."
- Contract or fixed-term roles are not penalized for being a contract.
In the rationale for a mid-level role, add one line noting the overqualification risk to prepare for (for example "expect a 'why this level?' question"). It is a note, not a deduction.

3. Preference fit (0-25 points)
Check against profile/preferences.md: location (<target city>, or remote/hybrid/on-site, all acceptable), compensation (at or above <target range> for senior roles; at or above <floor> base for mid-level PM / PM II and BD / Partnerships Manager roles; for contracts, annualize the rate), company stage (scale-up, Series B+ preferred but not required), industry (fintech/cybersecurity ideal; climate/cleantech and e-commerce/marketplaces also good; any software company acceptable). If the posting lists a language the candidate speaks natively (see preferences.md) as a required or preferred language, treat that as a positive within this category: this is real signal in the candidate's favor, never a reason to score lower or skip the posting. Likewise, a local employer with markets, customers, operations, or teams in the candidate's <home region> is a positive within this category: there, the candidate's track record in that region offsets the lack of local experience. Name the regional link in the rationale.

4. Dealbreaker check (0-15 points, or automatic cap if violated)
Flag and heavily penalize (cap total score at 30) if the posting:
- Explicitly requires native/bilingual French
- Explicitly requires Canadian citizenship or permanent residency
- Is a pure individual-contributor sales role with no strategic or product component, disguised under a "Business Development" title

## Essential-skills gap flag

Added after a case where a posting cleared the Apply threshold on the strength of preference and seniority while missing two of its own Essential requirements.

When the Skills & experience sub-score comes in below 26/40 (65% of max) and the shortfall is specifically against items the posting itself labels Essential, Required, or Must-have (not Preferred, Nice-to-have, or Highly Desired), the recommendation must carry an explicit flag, even if the total score clears the Apply threshold on the strength of the other three categories. Format: "Apply, flagged: Essential-skills gap (one-line reason naming the specific Essential item(s) at issue)". This is a structured signal, not just a mention buried in the rationale prose: bulk-apply and any manual drafting step must check for it and pause before generating cv.md/cover-letter.md, see bulk-apply's SKILL.md. A gap against a Preferred or Highly Desired item, or a gap that's genuinely learnable or reframeable via preferences.md's Domain Philosophy approach, does not get this flag; it's specifically for hard mismatches against a posting's own core requirements. This flag doesn't change the scoring math or the Apply/Maybe/Skip tier itself, it only adds a required, visible marker so a downstream drafting step knows to pause.

This flag is about the drafting gate downstream, not about linkedin-discovery's own scanning and scoring pass. Keep scanning and scoring every posting exactly as before, one card at a time, without pausing for the candidate mid-scan; the flag just needs to be present and correctly formatted in the scan file so the drafting step can find it later.

## Output format

For a single job posting, respond with:
- Title, Company, Location/Work mode
- Score breakdown: Skills (x/40), Seniority (x/20), Preference (x/25), Dealbreakers (x/15 or "CAPPED: [reason]")
- Total score (x/100)
- Recommendation: Apply (75+) / Maybe (55-74) / Skip (<55), plus the Essential-skills gap flag above when it applies
- One or two sentence rationale, specific: cite actual phrases from the posting and actual lines from the candidate's CV, not generic statements.

If the candidate pastes multiple postings in one message, return a single table ranked highest score first, with columns: Title | Company | Score | Recommendation | Key reason. Ask if they want the full breakdown for any specific one afterward rather than dumping full breakdowns for all of them unprompted.

## Tone and honesty

Be direct and a little critical, the goal is helping the candidate spend their application effort wisely, not flattering every posting. If a posting is a weak fit, say so plainly and explain why, referencing the specific gap.
