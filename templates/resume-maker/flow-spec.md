# Resume Maker Email Flow Spec

> Triggered when a student completes a resume on Studojo Resume Maker.
> The coach scores the resume quality, then branches.

---

## Trigger

**Event**: `mark-resume-ready` fires on the coach backend (main platform calls `POST /public/mark-resume-ready` with the student email when a resume is finished).

**At trigger time**: look up the student by email. Determine resume quality:
- If the student has a `CareerDNA` with `readiness_score`, use it.
- Otherwise score the resume snapshot via `calculate_readiness_score()`.
- Threshold: **readiness_score >= 55 = strong**, below = weak.

---

## Branch: Strong Resume (readiness >= 55)

Goal: push toward Outreach Dojo. The resume is good, the next move is getting it in front of hiring managers.

| # | Template | Delay | Sender | Angle |
|---|----------|-------|--------|-------|
| 1 | `resume-maker/strong/rm-strong-1.html` | 1 hr after resume ready | Jeremy | Your resume is strong, here is how to use it. Bridge to outreach. |
| 2 | `resume-maker/strong/rm-strong-2.html` | +2 days | Jeremy | Reply rate comparison + testimonial. Strong resume plus outreach. |
| 3 | `resume-maker/strong/rm-strong-3.html` | +3 days | Jeremy | The 3-step how outreach uses your resume. Final push. |

If the student starts Outreach at any point, exit and enter the Outreach used flow.

---

## Branch: Weak Resume (readiness < 55)

Goal: redirect to Career Coach first. The resume alone is not enough; they need to understand their gaps and direction before applying.

| # | Template | Delay | Sender | Angle |
|---|----------|-------|--------|-------|
| 1 | `resume-maker/weak/rm-weak-1.html` | 1 hr after resume ready | Pranav | A resume is one part. Know where you stand first. Bridge to coach. |
| 2 | `resume-maker/weak/rm-weak-2.html` | +2 days | Pranav | Testimonial: Meera was targeting the wrong roles. Coach redirected her. |
| 3 | `resume-maker/weak/rm-weak-3.html` | +3 days | Pranav | What the coach gives you before you apply. Final push. |

If the student starts the coach at any point, exit and enter the Career Coach flow.

---

## Notes

- Quality threshold (55) is a starting point. Tune against real conversion data.
- Both branches are pull-based. No "last email" language.
- Strong branch is signed by Jeremy (conversion voice). Weak branch by Pranav (coach voice).
- Variables: `{{first_name}}`
