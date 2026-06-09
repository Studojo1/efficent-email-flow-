# Old / Dormant User Re-engagement Flow Spec

> For users who signed up weeks or months ago and went quiet. Segmented
> by how far they got before going cold, routed by deepest engagement.
> Templates only. Not wired up.

---

## Detection (for the eventual cron)

A user enters this flow when:
- `Student.last_seen` is older than ~21 days, AND
- `Student.created_at` is older than ~21 days.

**Suppress entirely** if:
- `Student.has_active_outreach = True` (paying Outreach customer)
- `Student.email IS NULL`

**Handle gently** (do not hard-sell from scratch) if `Student.discount_intent_email` is set: they were already a warm lead.

---

## Routing: deepest engagement wins

Evaluate coach depth first, richest signal first. Only fall to cross-tool data when coach depth is shallow.

| Order | Signal | Segment |
|-------|--------|---------|
| 1 | `CheckIn` rows exist | A acted then vanished |
| 2 | `CareerDNA` exists, no CheckIn | B analysis, never acted |
| 3 | Messages, last `Message.state = PROFILING`, no DNA | C stuck in profiling |
| 4 | No real coach activity BUT `fetch_activity_for_email` shows resume / internship use | D cross-tool |
| 5 | Cold everywhere | E cold |

Cross-tool data via `main_platform_db.fetch_activity_for_email(email)`:
`resume_maker.used`, `internship_applications.count`, `career_applications`. Only consulted for segments D and E.

---

## Segment tracks

### Segment A acted then vanished (Jeremy)
| # | Template | Delay |
|---|----------|-------|
| 1 | `seg-a-acted/olduser-a1` | day 0 |
| 2 | `seg-a-acted/olduser-a2` | +7 days |
| 3 | `seg-a-acted/olduser-a3` | +10 days (handoff to Outreach) |

### Segment B analysis, never acted (Pranav, Jeremy on handoff)
| # | Template | Delay | Sender |
|---|----------|-------|--------|
| 1 | `seg-b-analysis/olduser-b1` | day 0 | Pranav |
| 2 | `seg-b-analysis/olduser-b2` | +7 days | Pranav |
| 3 | `seg-b-analysis/olduser-b3` | +9 days | Pranav |
| 4 | `seg-b-analysis/olduser-b4` | +9 days (handoff) | Jeremy |

### Segment C stuck in profiling (Pranav)
| # | Template | Delay |
|---|----------|-------|
| 1 | `seg-c-profiling/olduser-c1` | day 0 |
| 2 | `seg-c-profiling/olduser-c2` | +8 days |
| 3 | `seg-c-profiling/olduser-c3` | +10 days |

### Segment D used another tool, not the coach (Jeremy)
| # | Template | Delay |
|---|----------|-------|
| 1 | `seg-d-crosstool/olduser-d1` | day 0 |
| 2 | `seg-d-crosstool/olduser-d2` | +8 days |
| 3 | `seg-d-crosstool/olduser-d3` | +10 days |

### Segment E cold everywhere (Pranav, lightest)
| # | Template | Delay |
|---|----------|-------|
| 1 | `seg-e-cold/olduser-e1` | day 0 |
| 2 | `seg-e-cold/olduser-e2` | +9 days |
| 3 | `seg-e-cold/olduser-e3` | +9 days |

---

## Rules
- 3 to 4 emails per segment, then stop. Medium cadence over ~a month.
- No em dashes. No "last email" language.
- Testimonials with metrics in A2, B2, C3, D3, E2.
- Reference where they left off so it reads personal.
- Reply rate, if cited, ~10%.
- Variable: `{{first_name}}`.
