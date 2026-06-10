# Old / Dormant User Re-engagement Flow Spec (tool-neutral)

> For users who signed up weeks or months ago and went quiet. Routed by
> INTENT STAGE (how close they got to an outcome on any tool), with the
> CTA pointed at whichever tool they went deepest on. No tool is
> privileged. Templates only. Not wired up.

---

## Why this is tool-neutral

An old user is a Studojo user, not a Career Coach user. The router does
not check the coach first. It looks at the user's whole footprint across
every tool, weights them equally, and decides:

1. **Intent stage** (the message) = the highest outcome they reached on
   any tool.
2. **Deepest tool** (the CTA destination) = the tool with the strongest
   signal, compared on a normalised scale.

Only the dedicated Career Coach flow is coach-centric. This one is not.

---

## Detection (for the eventual cron)

Enter when:
- `Student.last_seen` older than ~21 days, AND
- `Student.created_at` older than ~21 days.

Suppress entirely:
- `Student.has_active_outreach = True` (paying Outreach customer)
- `Student.email IS NULL`

---

## Outcome signals (evaluated equally across all tools)

| Tool | Signal source | Outcome edge |
|------|---------------|--------------|
| Outreach | main platform order / payment-page events | hit payment page, started an order |
| Resume Maker | finished resume + strength score (same scoring as the Resume Maker flow) | a completed resume |
| Internship Dojo | `fetch_activity_for_email` application count | heavy applying (15+) |
| Career Coach | CheckIn rows, CareerDNA, message depth | completed roadmap actions |

---

## Stage = highest outcome reached on any tool

| Stage | Condition | Message theme |
|-------|-----------|---------------|
| **S1 Nearly converted** | Reached a real outcome edge on ANY tool (payment page, completed resume, completed coach actions, heavy applying) | You were nearly there. The hard part is done. Finish it. |
| **S2 Engaged** | Did meaningful work on some tool but stopped before any outcome | You got real work done, then it stalled. Pick it back up. |
| **S3 Barely started** | Signed up, minimal activity, no meaningful signal anywhere | No guilt. Here is what Studojo does now. One step. |

---

## CTA = deepest tool (the tool-neutral routing)

Within S1 and S2 the message is stage-appropriate; the closing CTA is
swapped at send time based on the user's deepest tool:

| Deepest tool signal | CTA block |
|---------------------|-----------|
| Outreach (payment / used) | `cta-variants/cta-outreach.html` |
| Resume Maker, strong resume | `cta-variants/cta-outreach.html` (resume is ready to use) |
| Resume Maker, weak resume | `cta-variants/cta-coach.html` (find direction first) |
| Internship Dojo (heavy applying) | `cta-variants/cta-two-tool.html` |
| Career Coach (analysis / actions) | `cta-variants/cta-coach.html` (re-anchor, then Outreach) |
| Nothing dominant | `cta-variants/cta-two-tool.html` |

S3 always uses the neutral two-tool CTA (no strong signal to route on).

---

## Stage tracks

### S1 Nearly converted (Jeremy)
| # | Template | Delay |
|---|----------|-------|
| 1 | `s1-nearly/old-s1-1` | Day 0 |
| 2 | `s1-nearly/old-s1-2` | +5 days |
| 3 | `s1-nearly/old-s1-3` | +9 days (CTA swaps by deepest tool) |

### S2 Engaged (Pranav, Jeremy on close)
| # | Template | Delay | Sender |
|---|----------|-------|--------|
| 1 | `s2-engaged/old-s2-1` | Day 0 | Pranav |
| 2 | `s2-engaged/old-s2-2` | +6 days | Pranav |
| 3 | `s2-engaged/old-s2-3` | +9 days (CTA swaps) | Jeremy |

### S3 Barely started (Pranav)
| # | Template | Delay |
|---|----------|-------|
| 1 | `s3-barely/old-s3-1` | Day 0 |
| 2 | `s3-barely/old-s3-2` | +7 days |
| 3 | `s3-barely/old-s3-3` | +7 days |

---

## Rules
- 3 emails per stage, then stop. Medium cadence over ~2 weeks.
- No em dashes. No "last email" language. No "readiness" anywhere.
- Testimonials with metrics in S1-2, S2-2, S3-2.
- Reply rate, if cited, ~10%.
- Variable: `{{first_name}}`.
