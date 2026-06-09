# Internship Dojo Email Flow Spec

> Triggered when a student has applied to internships on the job board
> but is not getting results. Offers two tools, branches on their choice.

---

## Trigger

**Condition**: Student has sent N+ applications through the Internship Dojo job board with no replies/callbacks logged.
- Suggested threshold: **>= 15 applications, 0 positive responses, over a 2 week window.**
- Tune against real data.

---

## Email 1: The Two-Tool Offer

| Template | Delay | Sender |
|----------|-------|--------|
| `internship-dojo/id-two-tools.html` | On threshold hit | Jeremy |

Presents two clear paths:
- **Outreach Dojo** for students who know what they want and need to reach the right people
- **Career Coach** for students unsure where they stand

Each tool has its own CTA button. The student's click decides the branch.

---

## Branch on click

- Clicks **Outreach Dojo** → enter the Outreach Dojo flow (used or not-used branch based on whether they actually start the tool)
- Clicks **Career Coach** → enter the Career Coach flow (cc-welcome onward)

If they click neither but the tool is used later, follow the same branching based on which tool they engaged.

---

## Did NOT open / did NOT click: Re-engagement

If the two-tool email is not opened or no CTA is clicked within a few days, run the re-engagement sequence. Each email re-frames the problem from a fresh angle.

| # | Template | Delay | Sender | Angle |
|---|----------|-------|--------|-------|
| 1 | `internship-dojo/id-reengage-1.html` | 3 days after two-tools, no engagement | Jeremy | The 2% reply rate stat. Why portal applications fail. Single CTA. |
| 2 | `internship-dojo/id-reengage-2.html` | +4 days, still no engagement | Jeremy | Rohan testimonial: 60 applications got nothing, 12 outreach messages got 3 interviews. Two CTAs again. |

After re-engagement with no response, the student can fall into the old-user re-engagement flow later.

---

## Notes

- The whole flow exists because the student already showed high intent (they applied a lot). We are redirecting that intent, not generating it.
- No "last email" language.
- Variables: `{{first_name}}`
