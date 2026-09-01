# LivAgora — Refund & Commission Policy Notes
### Reference for the Hotel Partner Agreement (to be drafted)

These are the business rules already implemented in the Finance dashboard prototype. They need to be reflected in the legal terms signed by each hotel partner, since the product enforces them automatically — the contract should describe what the product already does, not the other way around.

---

## 1. Who can request a refund

- Only the hotel's **Admin** role can request or manage refunds in the dashboard. Managers and Staff can view refund status but cannot act on it.
- A refund is always a **request**, not an instant action. The hotel does not hold the merchant-of-record relationship with the card networks — LivAgora does. The hotel submits a request (amount + required note explaining the reason); LivAgora's payments team processes it and updates the status once complete.
- While a request is pending, the booking is marked **"Refund Pending"** and no second request can be opened for the same booking.

## 2. The core distinction: has the pass already been used?

This is the rule that must not get lost when the contract is drafted — **it's the crux of who bears the cost of a refund.**

| Situation | Who decided to refund | LivAgora's commission |
|---|---|---|
| **Pre-service** — pass date is still in the future (genuine cancellation, duplicate charge, service issue caught in advance) | Guest or hotel, within/around the property's stated cancellation policy | **Reversed proportionally.** LivAgora did not successfully deliver a completed transaction, so it does not keep a commission on revenue that didn't materialize. |
| **Post-service** — pass date has already passed (guest completed their visit, and the hotel *chooses* to refund afterward — e.g. to head off a bad review, as a goodwill gesture) | **The hotel, unilaterally** | **Kept in full.** LivAgora already delivered its side of the service (matched the guest, processed payment) successfully. The hotel's later decision to refund for its own reputation/relationship reasons is a hotel business decision — LivAgora should not absorb the cost of a decision it didn't make and that doesn't reflect any failure on its part. |

**Why this matters for the contract:** without this clause, hotels could routinely refund satisfied guests after the fact (e.g. in response to any less-than-five-star review) and expect LivAgora to shoulder part of the cost every time. That would let the hotel unilaterally decide how much commission LivAgora earns on any given booking, after the fact. The contract needs to state plainly that **post-service, goodwill refunds are funded entirely out of the hotel's own share** — LivAgora's fee for a completed, successfully delivered booking is non-refundable once the pass has been used.

## 3. How the commission reversal is calculated (pre-service case only)

- Platform fee: **8% of gross booking value** (this rate should be confirmed/parameterized per contract — the prototype uses a flat rate, but real contracts may tier this by volume).
- If a refund is approved before the pass date, LivAgora reverses **8% of the refunded amount** (not the whole original fee) — so partial refunds get a proportional, not full, fee reversal.
- Example: $300 booking, $24 fee (8%), $276 to hotel. Guest gets a $150 pre-service refund → LivAgora reverses $12 (8% of $150) → hotel's payout is reduced by $138 net, not the full $150.
- **Don't forget:** this rule has no exceptions by *how* the refund happens — cancellation flow or discretionary "Issue Refund" flow, doesn't matter. Anything processed on or after the pass date keeps LivAgora's full fee; anything strictly before it reverses proportionally. Both flows must check the exact same cutoff (`pass date > today`, not `>=`) — an earlier draft of the prototype had cancellation and discretionary refund checking this with different comparisons (`>` vs `>=`), so a same-day booking could get *reversed* through one flow and *kept* through the other. Fixed, but worth re-testing if this logic is ever touched again — same-day bookings are the edge case that exposes a mismatch fastest.

## 4. Timing: refunds always land on the *next* payout

- LivAgora pays hotels on a recurring cycle (weekly, in the current prototype).
- A confirmed refund is **never retroactively deducted from a payout that has already gone out.** It's deducted from whichever payout period is still open/processing at the moment LivAgora confirms the refund — regardless of which payout period the original booking belonged to.
- The contract should set expectations that a refund confirmed today may show up as a deduction on a payout covering a *different, later* date range than the original stay.

## 5. This is separate from card disputes (chargebacks)

- A **refund** (above) is hotel- or guest-initiated and goes through LivAgora's own process.
- A **dispute/chargeback** is filed by the guest directly with their card issuer, and follows card network rules — not LivAgora's refund policy. Typical card network windows (as of 2026): cardholders generally have **120 days** from the transaction (or expected service date) to file, with rare extensions up to 540 days for specific circumstances. LivAgora and the hotel do not control this timeline.
- The contract should make clear that a hotel closing its own "refund window" internally does **not** protect it from a guest disputing the same charge with their bank well after that window — that risk sits with LivAgora's dispute-management process (see the "Credit Card Disputes" section of the dashboard) and may have its own cost-sharing terms, separate from the refund terms above.

## 6. Late cancellations — confirmed against direct competitor precedent (ResortPass), and implemented

**Status: implemented in both Finance and Reservations.** No-Show status still only exists in Finance.

This closes a real risk: if "staff cancels a booking" always auto-refunds regardless of timing, staff becomes a backdoor around the pass's own cancellation policy — a policy with a 24-hour cutoff means nothing if staff can cancel 1 hour out and still trigger a full automatic refund. The final model, after iterating:

- **One consistent flow regardless of timing.** Cancelling a reservation always refunds the guest in full and always requires a note explaining why — same modal, same required field, whether the pass's cancellation window is still open or has already closed. This was a deliberate simplification: earlier drafts blocked the refund entirely outside the window, but that created two different behaviors for staff to remember. Now there's one action; only the underlying commission math changes.
- **What actually changes with timing is who bears LivAgora's fee, not whether the guest gets refunded:**
  - *Within the window:* LivAgora reverses its proportional commission, same as a guest self-cancelling.
  - *Outside the window:* LivAgora keeps its full commission — the guest is still refunded in full, but that cost comes entirely out of the hotel's own share of the payout.
- **Restricted to Manager and Admin roles for cancellations — but Admin-only once the pass date has passed.** Cancelling has real financial consequences (it always triggers a refund), so it sits at the same permission level as issuing a refund. But once the pass date passes, "cancelling" stops being an available action anywhere in the app — the booking is either checked in, no-showed, or left as-is. The *only* remaining path to give money back is a discretionary refund, which at that point requires **Admin specifically**, not Manager — it's a more deliberate, exception-driven decision once the normal cancellation window has fully closed.
- **The required note is the audit trail.** Since the system no longer blocks late cancellations outright, the note field is what keeps this from being abused — every cancellation has a named person and a stated reason attached to it, especially the ones that cost LivAgora its commission.

Sources: ResortPass's own cancellation policy pages state plainly that bookings become non-refundable after the window closes; this note documents a deliberate divergence from that stricter model in favor of always refunding the guest, while still protecting LivAgora's commission via the pre/post-window fee split.

## 7. Payout timing should be anchored to the pass date, not the purchase date

**Status: agreed on concept, not yet implemented.** Today's prototype groups payouts by transaction/purchase date (the code even labels this field "Date booked"), which is the wrong anchor and should be corrected before this becomes real.

- **Industry standard (confirmed via Airbnb):** payout to the host is not released until *after* the service is delivered — for home stays, one business day after check-in; for **Experience/service-style hosts** (the closest match to LivAgora's day-pass model), payout is released after the experience is completed, not when it was booked. No matter how far in advance the guest paid, the host doesn't see that money until the service date has passed.
- **Why:** most legitimate, in-policy cancellations happen *before* the service date. Anchoring payout to the service date means the cancellation window has usually already closed by the time any money moves — sharply reducing how often LivAgora has to claw back a payout already sent to the hotel.
- **This does not replace the late-cancellation/no-show rule in Section 6 — it solves a different problem.** Delaying payout reduces LivAgora's *operational/accounting* exposure to refund clawbacks. It does **not**, by itself, stop staff from cancelling a booking outside its policy window and still expecting a refund — that's a contractual revenue-protection question, not a cash-flow-timing one. Both protections are needed together:
  - Payout anchored to pass date → cleaner accounting, less clawback risk
  - Late-cancellation/no-show non-refundable by default (Section 6) → protects the revenue itself, whether or not a payout has already gone out
- **Supporting precedent:** even Airbnb's own early-payout pilot (partial payout before check-in, for a fee) uses the exact same clawback mechanism already built here — "if a booking is cancelled after an early payout has been received, the amount will be deducted from the host's next booking."
- **Product implication:** the data model needs two separate dates per booking — purchase/transaction date (when the card was charged) and pass/service date (when the guest is scheduled to use it) — rather than the single conflated date field the prototype currently uses for both. Payout grouping should use the pass date; "Date booked" displays should keep using the purchase date.

---

*Prepared as a working note from the Finance dashboard build — not legal language. Legal/business teams should translate this into contract clauses.*
