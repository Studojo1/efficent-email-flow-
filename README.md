# Studojo Email Flows — New Flow (Standalone Blueprint)

> **Status: Not wired up. This is a blueprint only.**
> Nothing here connects to the existing emailer-service or current email flow.
> Designed to be integrated and replace the current flow when ready.

---

## Structure

```
.
├── README.md
├── flow-spec.md                            ← trigger logic, branching rules, event definitions
└── templates/
    ├── welcome-new-user.html               ← universal welcome (all signups, within 1 min)
    └── outreach/
        ├── not-used/                       ← Branch A: signed up for outreach, didn't use tool
        │   ├── outreach-nudge-d1.html      (7 hrs after welcome — soft check-in)
        │   ├── outreach-nudge-d2.html      (24 hrs after d1 — student testimonial)
        │   ├── outreach-nudge-d3.html      (32 hrs after d2 — 3-step walkthrough)
        │   └── outreach-nudge-d4.html      (day 4 — reply rate comparison, final nudge)
        └── used/                           ← Branch B: used the tool (high intent)
            ├── outreach-used-push1.html    (~4 hrs — momentum, what happens next)
            ├── outreach-used-push2.html    (~24 hrs — social proof, student results)
            ├── outreach-used-push3.html    (~50 hrs — honest last nudge)
            ├── outreach-used-convert1.html (~60 hrs — objection handling, what you get)
            ├── outreach-used-convert2.html (~75 hrs — post-payment walkthrough)
            ├── outreach-payment-page.html  (triggered immediately on payment page visit)
            └── outreach-used-coupon.txt    (PLAIN TEXT — triggered a few hrs after payment page email)
```

---

## Sender Split

| Templates | Sender |
|-----------|--------|
| `welcome-new-user` + all `outreach-nudge-*` | **Pranav** (co-founder) |
| All `outreach-used-*` + `outreach-payment-page` | **Jeremy** (co-founder) |

---

## Template Variables

| Variable | Replace with |
|----------|-------------|
| `{{first_name}}` | Student's first name |
| `{{COUPON_CODE}}` | Discount code, set at send time |

---

## Flows Parked for Later

- Career Coach Flow (its own standalone flow; also the exit ramp after Outreach week-1 non-use)
- Internship Dojo Flow
- Resume Maker / AI Risk / Reports Flows
- "Used outreach but switched to another tool" flow
- Returning user re-engagement flows

---

## To Integrate (when ready)

1. Load templates into `emailer-service`
2. Wire backend events: `signup`, `tool_used`, `payment_page_viewed`, `payment_completed`
3. Map events to sequences per `flow-spec.md`
4. Replace `{{first_name}}` and `{{COUPON_CODE}}` at send time
5. Set `payment_completed` as global sequence kill-switch
