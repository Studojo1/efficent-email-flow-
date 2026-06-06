# Studojo Email Flow Spec — Outreach Dojo (New Flow, Standalone)

> This flow is not wired up. It is a blueprint for future integration.
> Nothing in this folder touches the existing emailer-service or current email flow.

---

## Trigger Variables

| Variable | Description |
|----------|-------------|
| `{{first_name}}` | Student's first name |
| `{{COUPON_CODE}}` | Discount code (set at send time) |

---

## Flow 0 — Universal Welcome

**Trigger**: User signs up on studojo.com (any source)
**Delay**: Within 1 minute of signup
**Template**: `templates/welcome-new-user.html`
**Sender**: Pranav (cofounder)

After sending → branch on `signup_source` tag:

| Source tag | Next flow |
|------------|-----------|
| `outreach` | Outreach Dojo Flow (below) |
| `career-coach` | Career Coach Flow (parked) |
| `internship` | Internship Dojo Flow (parked) |
| `resume-maker` | Resume Maker Flow (parked) |
| `ai-risk` | AI Risk Flow (parked) |
| `reports` | Reports Flow (parked) |

---

## Flow 1 — Outreach Dojo (signup source = `outreach`)

### Branch A — User has NOT used the Outreach tool

**Entry condition**: User signed up via outreach source AND `tool_used` event has NOT fired

| # | Template | Delay | Sender |
|---|----------|-------|--------|
| 1 | `templates/outreach/not-used/outreach-nudge-d1.html` | 7 hrs after welcome | Pranav |
| 2 | `templates/outreach/not-used/outreach-nudge-d2.html` | 24 hrs after d1 | Pranav |
| 3 | `templates/outreach/not-used/outreach-nudge-d3.html` | 32 hrs after d2 | Pranav |
| 4 | `templates/outreach/not-used/outreach-nudge-d4.html` | Day 4 (96 hrs after welcome) | Pranav |

**Exit rules:**
- If `tool_used` event fires at any point → immediately exit Branch A, enter Branch B
- After d4 with no `tool_used` → trigger entry into **Career Coach Flow** (separate standalone flow; week-1 non-use is the backdoor entry point)

---

### Branch B — User HAS used the Outreach tool

**Entry condition**: `tool_used` event fires (at any point — during Branch A or directly)

#### Phase 1 — High intent push (0–50 hrs from `tool_used`)

| # | Template | Delay from `tool_used` | Sender |
|---|----------|------------------------|--------|
| 1 | `templates/outreach/used/outreach-used-push1.html` | ~4 hrs | Jeremy |
| 2 | `templates/outreach/used/outreach-used-push2.html` | ~24 hrs | Jeremy |
| 3 | `templates/outreach/used/outreach-used-push3.html` | ~50 hrs | Jeremy |

#### Phase 2 — Conversion gap (50–100 hrs from `tool_used`)

| # | Template | Delay from `tool_used` | Sender |
|---|----------|------------------------|--------|
| 4 | `templates/outreach/used/outreach-used-convert1.html` | ~60 hrs | Jeremy |
| 5 | `templates/outreach/used/outreach-used-convert2.html` | ~75 hrs | Jeremy |

#### Phase 3 — Payment page triggered

**Entry condition**: `payment_page_viewed` event fires (at any point during Branch B)

When `payment_page_viewed` fires:
1. **Pause** current Phase 1 or Phase 2 sequence
2. Send `templates/outreach/used/outreach-payment-page.html` immediately — Jeremy
3. Wait a few hours
4. Send `templates/outreach/used/outreach-used-coupon.txt` — Jeremy (plain text, no HTML)

**Global exit rule**: If `payment_completed` event fires at any point → stop all emails immediately.

---

## Events to Track (for future integration)

| Event | When it fires |
|-------|---------------|
| `signup` | User creates account |
| `tool_used` | User interacts with the Outreach Dojo tool |
| `payment_page_viewed` | User lands on the payment/checkout page |
| `payment_completed` | User successfully pays |

---

## Parked Flows (build later)

- Career Coach Flow — own complete standalone flow; also backdoor entry after outreach week-1 non-use
- Internship Dojo Flow
- Resume Maker Flow
- AI Risk Flow
- Reports Flow
- "Used outreach but switched to another tool" flow
- Returning / old user re-engagement flows
