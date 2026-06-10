# CTA Variants (tool-neutral routing)

The old-user stage emails carry a stage-appropriate body. The closing
call-to-action is swapped at send time based on the user's DEEPEST tool,
so the destination matches their actual footprint. No tool is privileged.

## The three CTA blocks

| File | Used when deepest tool is | Sends them to |
|------|---------------------------|---------------|
| `cta-outreach.html` | Outreach (payment/used), or a strong finished resume | Outreach checkout |
| `cta-coach.html` | A weak resume, or Career Coach (analysis/actions) | Career Coach |
| `cta-two-tool.html` | Internship Dojo heavy applying, or no dominant signal | Two-tool choice (Outreach or Coach) |

## How integration uses these

At send time the cron picks the deepest-tool block and injects it as the
closing CTA of the stage email. In these static blueprint templates the
stage emails ship with a sensible default inline CTA so they preview
cleanly; these three files document the swap-in variants the team can
review individually.

S3 (barely started) always uses `cta-two-tool.html` because there is no
strong signal to route on.
