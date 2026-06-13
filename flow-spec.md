# Studojo Email Flow — Live Spec

> **This is the flow as actually wired in `emailer-service` (production).**
> Source of truth: `internal/handlers/events.go` (routing + sequences),
> `internal/scheduler/scheduler.go` (drain, catch-up, paid suppression),
> `internal/email/sender.go` (subjects, senders, rate limit).
> Supersedes the old "not wired up / parked flows" design draft.

---

## How it runs

- **Triggers** arrive as `event.cc.*` events (from the platform + career-coach backend) on `POST /v1/email/events`.
- An **instant** email may send immediately; **follow-ups** are written to `scheduled_emails` with a `scheduled_at` and drained by the scheduler each minute.
- **Dedup**: a `UNIQUE (user_id, email_type)` index makes a second send physically impossible.
- **Preferences**: marketing sends respect `product_emails`; one-click unsubscribe sets it false.
- **Paid suppression**: a successful payment (`event.payment.success` or `event.cc.paid`) cancels every pending `cc_*` marketing row, and the scheduler also skips `cc_*` for paid users at send time.
- **Pacing**: one global rate limiter, `EMAIL_RATE_PER_HOUR` (currently **80**, under real ACS capacity). Transactional mail is never throttled.
- **Senders** rotate across `support` / `welcome` / `promotions` domains; the founder coupon blast is pinned to Jeremy.

---

## Trigger variables

| Variable | Meaning |
|----------|---------|
| `{{.UserName}}` | Student's first name (falls back to "there") |
| `{{.CouponCode}}` | Discount code, set at send time |
| `{{.UnsubscribeURL}}` | Signed one-click unsubscribe link |

---

## 1 — Outreach Dojo · NOT used  `event.cc.welcome_new_user`

| Step | Template | Delay | Subject |
|---|---|---|---|
| instant | `cc-welcome-new-user` | 0 | You're in the top 3%. Here's how we know. |
| +7h | `cc-outreach-nudge-d1` | 7h | Did you get a chance to try Outreach Dojo? |
| +31h | `cc-outreach-nudge-d2` | 31h | What one student got after using Outreach Dojo |
| +63h | `cc-outreach-nudge-d3` | 63h | Here's exactly how to get started |
| +96h | `cc-outreach-nudge-d4` | 96h (day 4) | The number that changes everything |

- **Catch-up**: if a user is 7h+ past signup with no `d1`, the scheduler queues it (covers downtime).
- **Exit**: `event.cc.outreach_used` cancels every pending `cc_outreach_nudge*` and enters the USED flow.

## 2 — Outreach Dojo · USED  `event.cc.outreach_used`

| Step | Template | Delay from trigger | Subject |
|---|---|---|---|
| instant | `cc-outreach-push1` | 0 | You started. Here's what happens next |
| +24h | `cc-outreach-push2` | 24h | Students who finished this got real replies |
| +50h | `cc-outreach-push3` | 50h | You're one step away |
| +60h | `cc-outreach-convert1` | 60h | Here's what you actually get |
| +75h | `cc-outreach-convert2` | 75h | After you sign up. Here's exactly what happens |

## 3 — Abandoned checkout  `event.cc.outreach_payment_page` (deferred — nothing sends instantly)

| Step | Template | Delay | Subject |
|---|---|---|---|
| +2h | `cc-outreach-payment-page` | 2h buffer | You were right there |
| +6h | `cc-outreach-coupon` | 6h | Something from me, Jeremy |

- A payment inside the buffer drains these (paid suppression), so a payer never gets "you were right there".
- `event.cc.outreach_coupon` can also fire standalone to send the founder coupon immediately.

## 4 — Career Coach · not started  `event.cc.welcome`

| Step | Template | Delay | Subject |
|---|---|---|---|
| instant | `cc-welcome` | 0 | You asked for an honest look. Good. |
| +8h | `cc-nudge-1` | 8h | Did you get started? |
| +32h | `cc-nudge-2` | 32h | What the coach actually tells you |
| +56h | `cc-nudge-3` | 56h | What changed when she finally started |

## 5 — Post-DNA  `event.cc.dna_ready`

| Step | Template | Delay | Subject |
|---|---|---|---|
| instant | `cc-dna-ready` | 0 | Your career analysis is ready |
| +2d | `cc-dna-confirm-nudge` | 2d | Your analysis needs your confirmation |
| +4d | `cc-checkin-1` | 4d | One action. This week. |
| +7d | `cc-checkin-2` | 7d | What students who act do differently |
| +10d | `cc-checkin-3` | 10d | Have you marked anything complete yet? |

## 6 — Roadmap  `event.cc.roadmap_delivered`

| Step | Template | Delay | Subject |
|---|---|---|---|
| instant | `cc-roadmap-delivered` | 0 | You have your roadmap. Here is how to use it. |
| +7d | `cc-upskill-nudge` | 7d | The coach gets sharper every time you use it |
| +9d | `cc-coupon-unlock` | 9d | Log your progress and unlock something |
| +11d | `cc-dormant` | 11d | Most students stop here |
| +14d | `cc-to-outreach` | 14d | You know where you stand. Here is what to do with it. |

## 7 — Resume Maker  `event.cc.resume_strong` / `event.cc.resume_weak`

**Strong** → `cc-rm-strong-1` (instant) · `cc-rm-strong-2` (+2d) · `cc-rm-strong-3` (+3d)
**Weak** → `cc-rm-weak-1` (instant) · `cc-rm-weak-2` (+2d) · `cc-rm-weak-3` (+3d)

## 8 — Internship Dojo  `event.cc.id_two_tools`

`cc-id-two-tools` (instant) · `cc-id-reengage-1` (+3d) · `cc-id-reengage-2` (+7d)

## 9 — Old / dormant users  `event.cc.old_s1` / `s2` / `s3` (spread-scheduled)

Big dormant batches cascade under a per-day cap (`EMAIL_OLDUSER_PER_DAY`, default 100) so new-user mail is never starved. First email lands at the next free slot; the rest chain off it.

- **S1**: `cc-old-s1-1` (slot) · `cc-old-s1-2` (+5d) · `cc-old-s1-3` (+9d)
- **S2**: `cc-old-s2-1` (slot) · `cc-old-s2-2` (+6d) · `cc-old-s2-3` (+9d)
- **S3**: `cc-old-s3-1` (slot) · `cc-old-s3-2` (+7d) · `cc-old-s3-3` (+14d)
- CTA blocks `cc-old-cta-outreach` / `-coach` / `-two-tool` are swapped into the S1/S2 closers.

## Coach-backend driven (not orchestrated here)

`cc-returning-1..3` and `cc-profiling-idle-1..3` templates exist and are sent by the **career-coach backend cron** via `POST /v1/email/send-template`, not by the emailer's event sequences.

## Transactional (never throttled, not marketing)

`welcome` · `payment-thankyou` · `forgot-password` · `password-changed` · `resume-optimized` · `internship-applied` · `contact-form` · `leads-ready` · `checkin-reminder`

## One-shot / ops

`cc-cart-goat` — founder abandoned-cart blast (GOAT10, 10% off), fired manually via `event.cc.cart_goat`. Pinned to the Jeremy sender.

## Global exit rule

`event.payment.success` / `event.cc.paid` → cancel all pending `cc_*` marketing for that user.
