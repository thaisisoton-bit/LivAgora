# Livagora — Click-Through Prototype

A static HTML/CSS/vanilla-JS prototype for **Livagora**, a marketplace where hotels sell day passes (pool, spa, cabanas, day rooms) to local guests who aren't staying overnight. This repo covers three audiences: the **guest-facing site**, the **hotel partner dashboard**, and **LivAgora's own internal platform admin**.

This is a prototype, not a production app. There is no real backend — every "database" is either hardcoded mock data or the browser's `localStorage`. See `dev-handoff-checklist.md` for exactly what a real build-out needs to replace.

---

## Quick start

Everything is static — no build step, no dependencies. Open `index.html` in a browser, or serve the folder with any static file server:

```bash
npx serve .
# or
python3 -m http.server 8000
```

**Deploying to GitHub Pages or similar:** `index.html` must stay at the project root — that's the file convention-based static hosts serve automatically at the root URL. Every other page is linked to by filename, so as long as the whole folder goes up together, relative links resolve correctly.

**Demo logins:** the guest site and partner portal don't check real credentials — any email/password gets you in. On staff pages, use the "Viewing as" switcher in the top bar to preview Admin / Manager / Staff roles without logging in separately.

---

## File map

### Guest-facing (public site)
| File | Purpose |
|---|---|
| `index.html` | Homepage — search, featured hotels, guest stories |
| `search-results.html` | Search results with filters |
| `hotel-page.html` | Hotel detail page — photos, amenities, reviews, pass selection + inline cart |
| `checkout.html` | Guest info, payment, order summary, confirmation |
| `create-account.html` | Guest registration |
| `forgot-password.html` | Guest password reset |
| `profile.html` | Guest account — bookings (with QR check-in), favorites, reviews, payment methods, notifications |
| `contact.html` | "Become a Host" partner application form |
| `support.html` | General guest support contact form |

### Hotel partner portal (staff-facing, per property)
| File | Access | Purpose |
|---|---|---|
| `host-login.html`, `host-forgot-password.html` | — | Partner portal auth |
| `host-dashboard.html` | Admin, Manager, Staff | KPI overview, reservation trend, top passes, revenue share |
| `reservations.html` | Admin, Manager, Staff | Booking list, check-in, cancellation |
| `passes.html` | Admin, Manager | Pass/experience configuration — pricing model, cancellation policy, availability |
| `inventory.html` | Admin, Manager | Capacity calendar and date blocks per pass |
| `finance.html` | **Admin only** | Transactions, refunds, disputes, payouts, late-cancellation tracking |
| `insights.html` | Admin, Manager | Operational health, guest behavior, and pass-performance analytics |
| `reviews.html` | Admin, Manager | Guest review list, rating trend, category breakdown, filter by pass |
| `settings.html` | **Admin only** | Hotel profile, policies (hours, house rules), team & permissions |
| `inbox.html` | Admin, Manager | Support case channel with LivAgora (see "Preparing for a real helpdesk" below) |

### LivAgora platform admin (not scoped to any one hotel)
| File | Purpose |
|---|---|
| `livagora-admin.html` | Platform-wide configuration — currently manages the review category list shown on every property's Reviews tab. Built with room to grow (Fee & Payout Rules, Cancellation Policies, Partner Properties are stubbed in the nav as "Soon"). |

### Reference docs (not app files — read these before building the real backend)
| File | Purpose |
|---|---|
| `dev-handoff-checklist.md` | What `localStorage` stands in for, and what needs a real backend before launch |
| `livagora-refund-policy-notes.md` | The refund/commission/cancellation business rules already enforced in the prototype — meant to become the actual hotel partner contract terms |
| `multi-item-cart-spec.md` | How a multi-pass cart should reconcile into independent bookings server-side |

---

## Design system

- **Fonts:** Fraunces (serif, headlines) + Inter (sans, everything else)
- **Core palette:** `--navy #0F1F2B` (brand/dark), `--cream #FBF2E6` (background), `--ink #0D1B24` (text) — identical across every file
- **Status color meaning:** green/lime = good, red = bad, amber = needs attention — but **never color alone**. Every trend indicator pairs color with an arrow and an explicit word ("Better"/"Worse"), since relying on color alone to convey state fails for colorblind users. If you add new KPI/delta UI, follow this pattern rather than reverting to color-only.
- **Access-blocked pattern:** pages restricted by role (`finance.html`, `settings.html`, `insights.html`, `reviews.html`, `inbox.html`) don't just hide their sidebar link — they check the role on load and swap the entire page body for a lock-icon message if someone lands there directly without permission. Reuse this pattern (`fin-access-blocked` CSS class) for any new restricted page.

---

## Business rules already enforced (not just UI — actual logic)

Full detail lives in `livagora-refund-policy-notes.md`. The short version:

- **Cancellation window is per-pass and hour-precise**, not a single global cutoff — e.g. a 24-hour policy and a 48-hour policy on the same day genuinely resolve differently. See `PASS_CANCELLATION_HOURS` in `finance.html`, `reservations.html`, and `host-dashboard.html` (must stay in sync across all three).
- **Within the window:** guest or staff can cancel; LivAgora reverses its 8% fee proportionally.
- **Outside the window, before the pass date:** only Manager/Admin can cancel; guest is still refunded in full, but LivAgora's fee is not reversed — that cost comes from the hotel's share.
- **After the pass date:** cancelling is no longer offered anywhere in the app. The only path back is a discretionary refund, and only an **Admin** (not Manager) can issue one.
- **Lost disputes (chargebacks)** deduct the disputed amount *plus* a flat chargeback fee from the next payout — never the fee-reversal treatment a voluntary refund gets, since the original sale did happen.
- **Payouts** are computed weekly and only ever adjusted going forward — a refund or lost dispute that happens after a payout period has already gone out is deducted from the *next* payout, never retroactively.

---

## Preparing for a real helpdesk (Inbox)

`inbox.html` is deliberately built so the entire page talks to one object, `InboxAPI` (`list`, `create`, `addMessage`, `setStatus`), instead of touching stored data directly. Today those four methods read/write a JSON blob in `localStorage`. To plug in a real helpdesk (Zendesk, Intercom, Front, Help Scout, or a custom backend), replace those four method bodies with real API calls — no other part of the file needs to change. Field names (`subject`, `status`, `category`, `messages`) were chosen to map closely to how those tools already model a ticket.

---

## Known limitations (see `dev-handoff-checklist.md` for the full list)

- No real backend — everything is mock data or `localStorage`, meaning nothing persists across devices/browsers and nothing is shared between the guest site and the partner portal in a real sense (they're separate mock datasets that happen to reference the same sample names).
- No real authentication — any credentials work; the "Viewing as" role switcher is a demo-only convenience.
- Card payments are not real — checkout does not call any payment processor.
