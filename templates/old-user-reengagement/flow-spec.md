# Old User Re-engagement Flow Spec

> Platform-wide re-engagement for users who signed up, used any tool,
> then went completely cold. Distinct from the Career Coach returning
> flow (which is coach-specific). This is for any dormant Studojo user.

---

## Trigger

**Condition**: User has had no activity on any Studojo product for an extended period.
- Suggested: **30+ days since last activity** (any tool: outreach, coach, resume maker, internship dojo).
- Exclude users who have an active paid Outreach order in progress.

---

## Sequence

| # | Template | Delay | Sender | Angle |
|---|----------|-------|--------|-------|
| 1 | `old-user-reengagement/old-1.html` | 30 days dormant | Jeremy | No guilt. A lot has changed. Here is what Studojo can do now. Two tools introduced. |
| 2 | `old-user-reengagement/old-2.html` | +7 days still dormant | Jeremy | Tanvi testimonial: a break did not set her back, coming back focused got her further. |
| 3 | `old-user-reengagement/old-3.html` | +7 days still dormant | Jeremy | Short. One step, whenever ready. Two CTAs (Outreach / Coach). |

If the user re-engages at any point, exit this flow and enter the relevant product flow based on which tool they open.

---

## Notes

- Tone is warm and low-pressure. These users left for a reason; guilt pushes them further away.
- No "last email" language.
- Distinct from `career-coach/returning/` which is for coach-specific dormancy with their analysis still saved.
- Variables: `{{first_name}}`
