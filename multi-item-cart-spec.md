# Multi-Item Cart, Cancellation & Reconciliation — Technical Spec

**Context:** Livagora allows guests to add multiple experiences (day passes, cabanas, spa treatments, day rooms) — potentially across different cancellation policies — into a single cart and complete one checkout. This spec defines how to do that safely for a revenue-share marketplace without inheriting reconciliation risk.

**Cart size assumption:** Realistic cart size is 2–6 items (couple/family booking a pass + a treatment). This is *not* retail-scale, so the design below intentionally avoids over-engineering a generic multi-vendor ledger.

---

## 1. Core principle

> **One checkout, many independent bookings.**
> The guest pays once. Internally, every cart item becomes its own `booking` row with its own status, cancellation policy, and payout lifecycle. Cancelling or refunding one item never touches the others.

---

## 2. Data model

```
orders
├── id (pk)
├── guest_id (fk)
├── hotel_id (fk)            -- v1: single-hotel cart (see §6)
├── status                   -- pending | paid | partially_refunded | refunded | completed
├── payment_intent_id        -- Stripe/Adyen PaymentIntent (one charge for the whole cart)
├── subtotal
├── total
├── created_at

booking_items
├── id (pk)
├── order_id (fk)
├── experience_id (fk)       -- references the specific pass/cabana/spa product
├── experience_name          -- snapshot at time of purchase (don't rely on live product data)
├── category                 -- pool | spa | day_room | activity
├── price                    -- snapshot at time of purchase
├── quantity
├── visit_date
├── cancellation_policy_id   -- snapshot of the policy that applied at purchase time
├── cancellation_deadline    -- computed absolute timestamp (visit_date - policy window)
├── status                   -- confirmed | cancelled | refunded | completed | no_show
├── refund_amount            -- nullable, filled if refunded
├── refunded_at
├── payout_status            -- held | released | clawed_back
├── payout_released_at

cancellation_policies
├── id (pk)
├── hotel_id (fk)
├── category                 -- policy can be hotel-wide or per-category
├── window_hours_before_visit
├── refund_type              -- full | credit_only | none
```

**Key decision:** `cancellation_policy_id` and `cancellation_deadline` are **snapshotted onto the booking_item at purchase time**, not looked up live. If a hotel changes its policy tomorrow, it must not retroactively change what a guest already agreed to.

---

## 3. Checkout flow

1. Guest adds items to cart (client-side state, matches current Hotel Page behavior).
2. On "Proceed to Checkout": create **one `order`** + **one `booking_item` per cart line**.
3. Create **one Stripe `PaymentIntent`** for the order total, with `metadata` listing each `booking_item_id` and its amount (Stripe supports structured metadata for exactly this).
4. On successful payment: mark `order.status = paid`, all `booking_items.status = confirmed`.
5. Confirmation email lists each item separately, **with its own cancellation deadline** (not one blended policy for the order).

---

## 4. Cancellation & refund flow

1. Guest cancels a **single `booking_item`** (from "My Bookings" — never a whole order at once unless every item is cancelled).
2. Backend checks `now() < cancellation_deadline` for that item only.
   - If within window → issue a **partial refund** against the original `PaymentIntent`, scoped to that item's amount (Stripe partial refund by amount, referencing the item in metadata).
   - If past window → block self-cancel; item stays `confirmed` (matches ResortPass's enforced behavior).
3. `order.status` is derived, not manually set:
   - All items refunded → `refunded`
   - Some items refunded → `partially_refunded`
   - None refunded → unchanged

---

## 5. Payout / revenue-share timing

> **Never release payout to the hotel before that item's cancellation window closes.**

- `booking_item.payout_status` starts as `held`.
- A scheduled job flips it to `released` once `now() > cancellation_deadline` **and** `status = confirmed`.
- Only `released` items are included in the hotel's payout batch.
- This eliminates clawbacks entirely for the common case — money never moves to the hotel until it's no longer refundable.
- If a hotel cancels/reschedules a `released` item (rare), that's the one case requiring a manual clawback — flag separately, don't design the whole system around it.

---

## 6. Scope guardrails for MVP

- **Single-hotel cart only.** Don't allow mixing items from two different hotels in one order — keeps payout logic simple (one `hotel_id` per order) and matches how guests actually shop (they're on one hotel's page).
- **Max 6 items per checkout, single hotel only.** Confirmed limit — enforced both client-side (Hotel Page cart UI) and server-side (reject order creation above 6 `booking_items`, or with more than one distinct `hotel_id`). Purely a UX/abuse guardrail, not a technical constraint.
- **One discount code per order**, applied to subtotal before tax/fees — matches ResortPass's observed behavior, avoids stacking edge cases.

---

## 7. What this buys us vs. the single-item ("override") approach

| | Single-item cart | Multi-item, per-item ledger (this spec) |
|---|---|---|
| Engineering cost | Lower | +1–2 days (one extra table, snapshot fields) |
| Conversion | Lower (forces repeat checkouts) | Higher |
| Refund complexity | Trivial | Trivial (Stripe partial refund per item) |
| Reconciliation risk | None | None, because payout is held until each item's window closes |

The extra cost is small and bounded because cart size is small and bounded. The single-item approach trades away conversion to avoid a risk that this design removes anyway.
