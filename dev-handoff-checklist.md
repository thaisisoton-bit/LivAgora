# Livagora — Dev Handoff Checklist

This is a **static, click-through prototype** (HTML/CSS/vanilla JS). It's built to communicate layout, interaction, and content structure — not to be shipped as-is. Below is what needs to happen to turn it into a real, production application.

---

## 1. What you're handing off

| File | Purpose |
|---|---|
| `index.html` | Public homepage |
| `search-results.html` | Search results (filters, list of hotels) |
| `hotel-page.html` | Hotel detail + mini cart (cart lives inline here, no separate cart.html) |
| `checkout.html` | Guest info + payment + order confirmation |
| `create-account.html` | New guest registration |
| `profile.html` | Guest account: bookings, favorites, reviews, payment methods, notifications, wallet-style pass + QR |
| `contact.html` | "Become a Host" partner application form |
| `support.html` | General guest support contact form |
| `multi-item-cart-spec.md` | Backend architecture spec for cart/refunds/payouts — **read this first** |
| `host-login.html`, `host-forgot-password.html` | Partner portal auth |
| `host-dashboard.html` | Staff/Admin/Manager KPI dashboard |
| `reservations.html` | Booking management, check-in, cancellation |
| `passes.html` | Pass/experience configuration (pricing, cancellation policy) |
| `inventory.html` | Capacity/blocks calendar per pass |
| `finance.html` | Payments, refunds, disputes, payouts (Admin only) |
| `insights.html` | Operational/guest/pass analytics (Admin + Manager) |
| `reviews.html` | Guest review management (Admin + Manager) |
| `settings.html` | Hotel profile, policies, team & permissions (Admin only) |
| `inbox.html` | Support case channel with LivAgora (Admin + Manager) |
| `livagora-admin.html` | LivAgora's own platform-level config (separate from any one hotel) |

**See `README.md` in the project root for the full file map, the design system, and the business rules enforced throughout the prototype (cancellation windows, role permissions, payout math).** This section is left as historical context from an earlier pass — the host/admin side referenced below as "not yet built" has since been built out in full; see `README.md` instead for current state.

**Originally not yet built (now complete, see README.md):** ~~About page, host/admin side (partner login, dashboard, inbox, reservations, inventory, finance/invoicing, host-side reviews)~~.

---

## 2. The big one: everything runs on `localStorage`, not a real backend

Since this is a static prototype, every "database" is actually the browser's `localStorage`. None of this persists across devices, browsers, or after a user clears their cache — **all of it needs a real backend and database before launch.**

| localStorage key | Stands in for | Needs to become |
|---|---|---|
| `livagoraSession` | Logged-in session | Real auth (session cookie / JWT) |
| `livagoraProfile`, `livagoraProfilePhoto` | Guest profile + avatar | `users` table + image upload to real storage (S3/Cloudinary) |
| `livagoraCart` | Cart contents | Server-side cart tied to session, per the multi-item cart spec |
| `livagoraCartEverSet` | Internal flag so demo data doesn't reappear after a real cart empties | Delete — this was only a prototype workaround |
| `livagoraBookings` | Booking history | `orders` / `booking_items` tables — see spec doc |
| `livagoraFavorites` | Saved hotels | `favorites` table (user_id, hotel_id) |
| `livagoraReviews` | Guest reviews | `reviews` table, tied to a completed `booking_item` |
| `livagoraSavedCards` | Saved payment methods | **Do not build this yourself** — use Stripe Customer + saved Payment Methods (or Adyen equivalent). Never store real card numbers. |
| `livagoraNotifPrefs` | Email preference toggles | `notification_preferences` table |

---

## 3. Things that are visually complete but functionally fake

- **Payment form** (`checkout.html`) — no real payment processing. Needs Stripe/Adyen Elements (or similar) dropped in; card fields should never touch your own servers.
- **QR codes** — currently a deterministic pattern generated from the confirmation code for visual purposes only, **not a real scannable QR.** Swap in a real QR library (e.g. `qrcode` npm package) encoding a real check-in URL/token.
- **"Add to Apple Wallet / Google Wallet"** buttons — decorative. Real integration requires Apple PassKit (`.pkpass` file generation, signed with an Apple certificate) and the Google Wallet API.
- **"Send Message" / "Apply to Become a Host"** forms — show a success state but don't send anything. Needs a real backend endpoint + email service (SendGrid, Postmark, etc.) to actually notify your team and confirm to the user.
- **Share pass** (copy link) — the generated link (`livagora.com/pass/...`) doesn't resolve to anything real yet.
- **Login modal / "Log In" everywhere** — UI only, no real authentication check. Any "logged in" state is simulated by the mock login button in `checkout.html`/`profile.html`.

---

## 4. Images

Every photo in this prototype is a **CSS gradient**, not a real image (this was a constraint of the design environment, not a design decision). Before launch:
- Replace every `background:linear-gradient(...)` used as a "photo" with real `<img>` tags or CSS `background-image`
- Decide on an image pipeline (aspect ratios, responsive `srcset`, lazy loading, CDN/optimization)
- The hotel gallery lightbox, hero images, hotel cards, and the wallet-style pass all need real photography

---

## 5. Content that's placeholder text

- All hotel names, descriptions, prices, and reviews are fictional example data
- Cancellation policy text is illustrative — legal/ops should confirm actual policy wording per hotel
- **Terms of Service** and **Privacy Policy** links are all `href="#"` — these need real legal pages before this can go live in any real capacity
- FAQ content, "About" section copy, and footer links to `#about`, `#pricing` etc. need real destinations or should be removed

---

## 6. Design system reference (already consistent — reuse it)

```css
--cream: #FBF2E6;       /* page background */
--cream-soft: #FCF6EC;  /* card/section background */
--ink: #0D1B24;         /* primary text */
--navy: #0F1F2B;        /* dark sections, primary buttons */
--navy-2: #0A161F;      /* navy hover state */
--gold: #C9A227;        /* accent only — ratings, small badges. Never large text/backgrounds. */
--line: #E7DCC9;        /* borders */
--muted: #6B6459;        /* secondary text */
```
Fonts: **Fraunces** (serif, headings/display) + **Inter** (sans, body). Both loaded via Google Fonts — consider self-hosting for production performance/reliability.

---

## 7. Before writing a line of production code

1. **Read `multi-item-cart-spec.md` in full** — it defines the data model, checkout flow, cancellation/refund logic, and payout timing. This is the architectural backbone of the whole booking system.
2. **Decide on your stack** — this prototype has no opinion on React vs. server-rendered vs. anything else. The HTML/CSS can be used as a direct visual reference (or even component starting point), but the JS logic (all the `localStorage` code) should be rewritten against real APIs, not ported as-is.
3. **Design the API** — every `localStorage.getItem/setItem` call in the code marks a place where a real API call needs to go. Search each HTML file for `localStorage` to find every touchpoint.
4. **Set up payments early** — Stripe Connect (or Adyen for Platforms) is a bigger integration than it looks, especially with the per-item payout-hold logic in the cart spec. Start this early.

---

## 8. QA before launch

- [ ] Cross-browser check (this was tested in Chromium only during design)
- [ ] Real mobile device testing (not just responsive resize — test on actual iOS/Android)
- [ ] Accessibility pass: keyboard navigation, screen reader labels, color contrast
- [ ] Form validation duplicated server-side (all current validation is client-side JS only, which is trivial to bypass)
- [ ] Rate limiting on public forms (Become a Host, Support, Login) to prevent spam/abuse
- [ ] Real error states — this prototype has no "something went wrong" handling anywhere

---

## 9. Still needs to be designed

- About page
- Host/admin side in full: partner login & registration, dashboard, inbox (guest messaging), reservation management, guest profile (host-side view), amenities management, inventory, finance/invoicing & payout reporting, host-side review management

If useful, I can keep building these next.
