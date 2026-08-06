# Livagora — Clickable Design Prototype

This folder contains a **static, click-through prototype** of the Livagora guest flow — plain HTML/CSS/vanilla JS, no build step, no framework, no backend. It exists to communicate layout, interaction, content structure, and edge-case behavior to the dev team before real implementation begins.

**⚠️ This is not production code.** Do not import these files into `/src`, extend them in place, or treat them as a starting point to "just clean up." See `dev-handoff-checklist.md` for exactly what needs to be rebuilt for real (payments, auth, database, QR codes, wallet passes, etc.) before any of this can go live.

## How to view it

No install needed — every file is self-contained. Just open `livagora-landing.html` in a browser, or drag the whole folder onto a static host (GitHub Pages, Vercel, Netlify) for a shareable link the whole team can click through.

## Guest flow, in order

```
livagora-landing.html   → homepage, search
  ↓
search-results.html     → filtered list of hotels
  ↓
hotel-page.html          → hotel detail, pass selection, mini cart
  ↓
cart.html                → (optional) standalone cart review
  ↓
checkout.html             → guest info, payment, confirmation
  ↓
create-account.html       → registration (new guests)
  ↓
profile.html               → bookings, favorites, reviews, payment methods,
                              notifications, wallet-style pass + QR

contact.html   → "Become a Host" partner application
support.html   → general guest support contact form
```

## Data persistence

There's no backend, so the prototype fakes persistence with the browser's `localStorage` (cart contents, session, bookings, favorites, reviews, saved cards, notification preferences). This is **only good for demoing in one browser** — nothing here is shared across devices or survives a cache clear. Every one of these keys maps to a real database table that needs to be designed — see the table in `dev-handoff-checklist.md`.

## Key docs in this folder

- **`dev-handoff-checklist.md`** — start here. Full list of what's real vs. fake in this prototype, and what needs to be built for each: payments, QR codes, wallet passes, images, forms, auth, etc.
- **`multi-item-cart-spec.md`** — backend architecture for the cart/checkout system: one PaymentIntent but one row per cart item, per-item cancellation windows, payout timing, refund logic. This is the data model backbone for the whole booking system — read before designing the database schema.

## Design system quick reference

```css
--cream: #FBF2E6;      /* page background */
--cream-soft: #FCF6EC; /* card/section background */
--ink: #0D1B24;        /* primary text */
--navy: #0F1F2B;        /* dark sections, primary buttons */
--gold: #C9A227;        /* accent ONLY — ratings, small badges. Never large text or backgrounds. */
--line: #E7DCC9;        /* borders */
--muted: #6B6459;        /* secondary text */
```
Fonts: **Fraunces** (serif, headings) + **Inter** (sans, body), via Google Fonts.

## Not yet designed

About page, and the full host/admin side (partner login, dashboard, inbox, reservation management, guest profile, amenities, inventory, finance/invoicing, host-side reviews).
