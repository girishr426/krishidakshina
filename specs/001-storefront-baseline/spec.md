# Feature Specification: Storefront Baseline (KrishiDakshina v1)

**Feature Branch**: `001-storefront-baseline`

**Created**: 2026-07-18

**Status**: Draft

**Input**: User description: "Reverse-engineer the existing static site (`index.html`, `index.css`, `index.js`, `CNAME`) into a versioned baseline specification so all future changes flow through the Spec Kit workflow. The site is a single-page, zero-dependency static storefront for a farm-fresh grocery brand ('KrishiDakshina') that lets a visitor browse 12 curated products, add them to a persistent cart, enter delivery details, and place an order via a pre-composed WhatsApp message. No backend, no analytics, no cookies. Deployed to GitHub Pages via `CNAME` (`krishidakshina.in`)."

---

## Overview & Business Context

**What this feature is**: The current shipped state of the [`krishidakshina.in`](https://krishidakshina.in) storefront, captured as a spec. This document is *descriptive* of what already exists in `index.html`, `index.css`, and `index.js` — its purpose is to lock in a baseline so subsequent Spec Kit features can amend it deliberately rather than by accident.

**Why**: The business needs a marketing site plus an order-taking mechanism without paying for backend infrastructure. WhatsApp is the incumbent communication channel for the target market, and `localStorage` is sufficient to hold cart state between page loads on a single device. Ratifying the baseline gives every future change a stable reference point traceable to the [constitution v2.1.0](../../.specify/memory/constitution.md).

**Constitutional traceability**: Every user story and requirement in this document is annotated with the constitutional principle(s) it depends on: **P-I** Accessibility-First, **P-II** Performance Budget, **P-III** Static-First & Framework-Minimal, **P-IV** Security-by-Default, **P-V** Client-Only Architecture (WhatsApp Handoff), **P-VI** Verifiable Releases.

---

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Browse and Add to Cart (Priority: P1)

A visitor lands on the home page, scrolls to the Products section, sees a grid of 12 curated products (each with an image or emoji fallback, name, category badge, description, price, and per-unit label), and adds items to a cart drawer that persists across page reloads.

**Why this priority**: Without a browsable, addable catalog there is no order to place. This is the MVP core — the site fails its business purpose if this story does not work.

**Constitutional traceability**: **P-I** (semantic product cards, keyboard-reachable `+` buttons with ARIA labels, `alt` on every product image), **P-II** (product images ≤ 200 KB, lazy loaded below the fold), **P-IV** (XSS-safe DOM insertion of any product name or badge; delegated event listener rather than inline `onclick`), **P-V** (cart state persisted to `localStorage` under `krishidakshina.cart.v1`).

**Independent Test**: Load the site with an empty `localStorage`, click the `+` button on any product card, verify the cart badge shows `1` and the drawer contains one line for that product. Reload the tab and confirm the cart still contains that line.

**Acceptance Scenarios**:

1. **Given** the visitor has never opened the site (empty `localStorage`), **When** the Products section renders, **Then** exactly 12 product cards are shown, each with a name, category, description, price, unit label, and a keyboard-focusable `+` button with an `aria-label`.
2. **Given** a product image file is missing (`404`) from `product_images/`, **When** the card renders, **Then** the emoji from `data-fallback` is displayed in place of a broken-image icon, and the `alt` text remains unchanged.
3. **Given** the cart is empty, **When** the visitor activates the `+` button on a product (mouse click, `Enter`, or `Space`), **Then** a line item for that product is created with `qty = 1`, the navbar cart badge increments to `1`, and the item is persisted to `krishidakshina.cart.v1`.
4. **Given** a product is already in the cart with `qty = n` (where `1 ≤ n < 99`), **When** the visitor activates `+` on the same product card again, **Then** the line item's `qty` becomes `n + 1` and the cart badge increments accordingly.
5. **Given** a product is in the cart with `qty = 99`, **When** the visitor activates `+` on the same product card, **Then** the `qty` remains `99` (silent cap) and no additional line is created.
6. **Given** the cart already contains 50 distinct products, **When** the visitor activates `+` on a 51st distinct product, **Then** the new product is not added (`MAX_CART_ITEMS` cap) and the existing cart is unchanged.
7. **Given** the visitor has items in the cart, **When** the page is reloaded, **Then** the cart drawer and badge reflect the same items and quantities as before the reload.

---

### User Story 2 - Adjust or Clear the Cart (Priority: P1)

Inside the slide-in cart drawer, the visitor can increment or decrement each line, remove a line by decrementing it to `0`, or clear the entire cart with one action.

**Why this priority**: Order accuracy depends on this. A cart that cannot be corrected leads to abandoned orders and support burden.

**Constitutional traceability**: **P-I** (drawer controls keyboard-operable with visible focus; drawer traps focus when open), **P-IV** (all mutations go through the same validator that hydrates `localStorage` — no untrusted qty ever reaches state), **P-V** (every mutation is persisted synchronously to `krishidakshina.cart.v1`).

**Independent Test**: With a cart containing at least two distinct products at various quantities, exercise `+`, `-`, and "Clear cart"; verify the badge, drawer, and `localStorage` value stay consistent at each step.

**Acceptance Scenarios**:

1. **Given** a line in the drawer shows `qty = 3`, **When** the visitor activates the drawer-line `+` button, **Then** the line shows `qty = 4` and the badge increments by 1 (respecting the `99` cap from US-1).
2. **Given** a line in the drawer shows `qty = 2`, **When** the visitor activates the drawer-line `-` button, **Then** the line shows `qty = 1` and the badge decrements by 1.
3. **Given** a line in the drawer shows `qty = 1`, **When** the visitor activates the drawer-line `-` button, **Then** the line is removed from the drawer, the badge decrements by 1, and the line is removed from `krishidakshina.cart.v1`.
4. **Given** the cart contains one or more lines, **When** the visitor activates "Clear cart", **Then** the drawer becomes empty, the badge shows `0`, and `krishidakshina.cart.v1` is either removed or set to an empty object.

---

### User Story 3 - Provide Delivery Details (Priority: P1)

Below the cart lines the visitor fills a delivery form (Name, Phone, Address Line 1, Address Line 2, Pincode, City, Notes) whose values persist between visits. Optionally the visitor taps "Use my current location" to attach a Google Maps pin.

**Why this priority**: Without delivery details there is nowhere to ship, and without persistence the visitor re-types on every visit. This is required for the order handoff.

**Constitutional traceability**: **P-I** (labels associated with inputs, error messages linked via `aria-describedby`, focus moves to the first invalid field on submit attempt), **P-IV** (`phone: /^[6-9]\d{9}$/`, `pincode: /^[1-9]\d{5}$/`, per-field length caps, strict schema validation on load), **P-IV clause 9** (geolocation is opt-in and behind an explicit button — never auto-requested), **P-V** (form values persisted to `krishidakshina.customer.v1`; pincode → city lookup calls `api.postalpincode.in`, an allow-listed `connect-src`).

**Independent Test**: Fill the form with valid values, reload the tab, and confirm the values pre-fill from `krishidakshina.customer.v1`. Enter an invalid phone or pincode and confirm submission is blocked with an inline error.

**Acceptance Scenarios**:

1. **Given** the visitor has never filled the form, **When** the delivery panel renders, **Then** all fields are empty and required fields are visually marked as required.
2. **Given** the visitor types a valid phone (matches `^[6-9]\d{9}$`), Address Line 1, and a valid pincode (matches `^[1-9]\d{5}$`), **When** they leave the pincode field with the City field empty, **Then** the site fetches `https://api.postalpincode.in/pincode/<pincode>` and populates City from the response.
3. **Given** the pincode API is unreachable or returns an error, **When** the lookup fails, **Then** the visitor sees a non-blocking notice, the City field remains editable, and the order flow is not blocked (**Resilience**).
4. **Given** the visitor types values into any field, **When** they navigate away or reload, **Then** the previously-entered values are re-hydrated from `krishidakshina.customer.v1` and pass the strict validator; any value that fails validation is dropped rather than shown.
5. **Given** the visitor activates "Use my current location", **When** the browser prompts for permission and the visitor grants it, **Then** a Google Maps link is stored in the customer state and included in the WhatsApp message. **When** permission is denied or unavailable, **Then** the visitor sees a non-blocking message and can still place the order.

---

### User Story 4 - Place Order via WhatsApp (Priority: P1)

Once the cart has ≥ 1 item AND all required delivery fields validate, the "Order via WhatsApp" button enables. Activating it composes a formatted plain-text message (header, customer, delivery, line items, total, optional map pin) and opens `https://wa.me/<WHATSAPP_NUMBER>?text=<encoded>` in a new tab.

**Why this priority**: This is the actual order handoff — the point where the site delivers business value. Without it, the previous stories are ceremony.

**Constitutional traceability**: **P-IV** (only opens external links with `rel="noopener noreferrer"`; message assembled with pure string concatenation from validated state — no HTML injection surface), **P-V clause 4** (WhatsApp handoff is the sole order-placement path; the encoded message MUST NOT exceed `MAX_MSG_LEN = 3800` chars), **P-V clause 5** (`wa.me` is on the runtime allow-list for navigation).

**Independent Test**: With a valid cart and valid delivery form, click "Order via WhatsApp"; verify the URL is `https://wa.me/<WHATSAPP_NUMBER>?text=...`, the decoded text contains the expected header, all cart lines, and the customer name/phone/address, and the encoded length is ≤ 3800 characters.

**Acceptance Scenarios**:

1. **Given** the cart is empty OR any required field fails validation, **When** the visitor looks at the "Order via WhatsApp" button, **Then** the button is disabled (or the click is rejected with an inline error identifying the missing/invalid fields).
2. **Given** the cart has ≥ 1 item and all required delivery fields validate, **When** the visitor activates "Order via WhatsApp", **Then** a new tab opens with `https://wa.me/<WHATSAPP_NUMBER>?text=<message>`, and the decoded message begins with `🛒 *New Order – KrishiDakshina*`, includes each cart line with unit price and line total, includes the customer name/phone/address/pincode/city, and includes the grand total.
3. **Given** the encoded message length exceeds `MAX_MSG_LEN` (3800 chars), **When** the visitor activates "Order via WhatsApp", **Then** the WhatsApp tab is NOT opened, and an inline error asks the visitor to reduce the cart or shorten the notes.
4. **Given** the visitor previously attached a location pin, **When** the WhatsApp message is composed, **Then** the message includes a `https://maps.google.com/?q=<lat>,<lng>` link.

---

### User Story 5 - Contact the Business (Priority: P2)

The Contact section shows the business address, phone, email, hours, a WhatsApp CTA, and a message form. Submitting the message form runs a client-side simulation (spinner + success toast); there is no backend delivery in v1. This is intentional.

**Why this priority**: The message form is a nice-to-have that reassures visitors the site is staffed; the WhatsApp CTA is the real communication channel and already covered by US-4's origin allow-listing. If US-5 breaks, orders still work.

**Constitutional traceability**: **P-I** (form has labels, error messages, and keyboard support), **P-V clause 6(a)** (contact-form submission is a WhatsApp handoff consistent with the order flow — no third-party form endpoint, no backend, no analytics; reuses the already-declared `wa.me` origin).

**Independent Test**: Fill the contact form with any values (first name + email + message required) and submit; verify a brief spinner, then a new tab opens on `wa.me/919876543210` with the enquiry pre-filled as a WhatsApp message, and confirm (via DevTools Network tab on the origin page) that no `fetch` / `XMLHttpRequest` fires from the page origin — only the external navigation to `wa.me` appears in the newly opened tab.

**Acceptance Scenarios**:

1. **Given** the visitor is on the Contact section, **When** the section renders, **Then** the site's address, phone, email, business hours, and a WhatsApp CTA are visible.
2. **Given** the visitor fills the contact form, **When** they submit it, **Then** a spinner is shown briefly, a new tab opens on `wa.me/919876543210` with a pre-filled enquiry message, a success toast ("WhatsApp opened with your message — press Send there to reach us.") appears, the form is cleared, and no `fetch` / `XMLHttpRequest` fires from the page origin (WhatsApp handoff via `window.open` — P-V clause 6(a)).
3. **Given** the visitor activates the WhatsApp CTA, **When** the click is handled, **Then** a new tab opens to `https://wa.me/<WHATSAPP_NUMBER>` (no pre-composed message required for the CTA path).

---

### User Story 6 - Navigate on Any Device (Priority: P2)

The visitor uses the site on desktop, tablet, and mobile: a fixed navbar with smooth-scroll section links and an active-section indicator; a mobile hamburger menu below 768 px; a floating scroll-to-top button; fade-in-on-scroll animations for content blocks; decorative particle background in the hero; animated truck icon in the features strip. All animations respect `prefers-reduced-motion`.

**Why this priority**: Presentation and navigation quality drive conversion, but the site is still usable if animations degrade. Placing this at P2 signals it is important but not gating.

**Constitutional traceability**: **P-I** (motion respects `@media (prefers-reduced-motion: reduce)`; the navbar is a `<nav>` landmark; mobile menu is keyboard-operable), **P-II** (animations do not thrash layout; no polyfills bloat mobile JS; keeps INP < 200 ms), **P-IV clause 7** (any randomness used for the particle background uses `crypto.getRandomValues()` — `Math.random()` is banned in production JS).

**Independent Test**: Resize the browser below 768 px and confirm the hamburger appears and the desktop nav-links collapse. Enable "Reduce Motion" in OS settings and confirm the truck animation, particle motion, and hover-motion effects stop.

**Acceptance Scenarios**:

1. **Given** the viewport is ≥ 768 px, **When** the visitor loads the page, **Then** the desktop navigation is visible and the hamburger button is hidden.
2. **Given** the viewport is < 768 px, **When** the visitor loads the page, **Then** the hamburger button is visible and the desktop nav-links are hidden until the hamburger is activated.
3. **Given** the visitor clicks a nav link, **When** the click is handled, **Then** the page smooth-scrolls to the target section and that section is marked as the current one in the navbar.
4. **Given** the visitor scrolls past the hero, **When** the scroll position crosses a threshold, **Then** the floating scroll-to-top button becomes visible; clicking it smooth-scrolls to the top.
5. **Given** the OS or browser reports `prefers-reduced-motion: reduce`, **When** the page is rendered, **Then** the truck animation, particle motion, hover-motion effects, and fade-in-on-scroll transitions are disabled or reduced.

---

### Edge Cases

- **Tampered `localStorage`**: A key `krishidakshina.cart.v1` containing malformed JSON, unknown fields, non-integer qty, negative qty, qty above 99, or more than 50 line items MUST be rejected on load. The offending entries are dropped and the site starts fresh rather than crashing.
- **Missing product image**: `product_images/<file>.jpg` returns 404. The `data-fallback` emoji is rendered in its place. No broken-image icon is ever shown.
- **Pincode lookup failure**: `api.postalpincode.in` is unreachable, times out, or returns an unexpected shape. The user is notified non-blockingly and can type the city manually; the order flow is unaffected.
- **Geolocation denial**: The visitor denies the permission prompt or the device has no GPS. The button surfaces a friendly notice; the order flow is unaffected.
- **Oversized WhatsApp message**: The encoded message length exceeds 3800 chars (large cart, long notes). The order button surfaces an inline error and does NOT navigate; nothing is sent.
- **Empty cart at checkout**: The visitor clears the cart while the delivery form is filled. "Order via WhatsApp" becomes disabled; the delivery form remains valid and persisted.
- **Duplicate `+` clicks / rapid interactions**: Cart mutations remain idempotent per event (no double-count from a single click).
- **Missing CSP or weakened CSP**: A misconfigured deployment strips the meta CSP or introduces `'unsafe-inline'`. The verifiable-release gates in Principle VI MUST block the merge.
- **`Math.random` reintroduced in production JS**: A regression adds `Math.random(...)`. The Principle VI static scan MUST fail the release.
- **JavaScript disabled**: The visitor loads the page with JS off. Static content (nav, product images, contact info, hero copy) still renders; cart/order features are inert. This is acceptable in v1.

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001** (P-V, P-III): The site MUST be a single static page (`index.html`) served with two sibling assets (`index.css`, `index.js`) plus a `CNAME`, deployable to GitHub Pages with zero server-side runtime and zero build step.
- **FR-002** (P-III): The shipped page MUST have no `package.json`, no framework, no bundler, and no transpiler at build time. Runtime dependencies fetched at page load are limited to the Additional Constraints allow-list (`cdnjs.cloudflare.com`, `fonts.googleapis.com`, `fonts.gstatic.com`, `api.postalpincode.in`, `wa.me`).
- **FR-003** (P-I): The page MUST use semantic landmarks (`<nav>`, `<main>` implied by the single-page structure, `<section>`, `<footer>`), heading order MUST be correct, every `<img>` MUST have meaningful `alt` (or `alt=""` if purely decorative), and every icon-only button MUST have an `aria-label`.
- **FR-004** (P-I): The page MUST be operable using keyboard only, with visible focus states, and MUST remain usable at 200 % zoom.
- **FR-005** (P-I): All motion/animation MUST respect `@media (prefers-reduced-motion: reduce)` — including the truck animation, particle background, fade-in-on-scroll, and hover-motion effects.
- **FR-006** (P-IV): The page MUST ship a strict CSP via `<meta http-equiv="Content-Security-Policy">` with `default-src 'self'`; `script-src 'self'` (no `'unsafe-inline'`, no `'unsafe-eval'`, no inline `<script>` bodies, no inline event handlers); `connect-src 'self' https://api.postalpincode.in`; `frame-ancestors 'none'`; `object-src 'none'`; `form-action 'self'`; `base-uri 'self'`. The `Referrer-Policy` MUST be `strict-origin-when-cross-origin`.
- **FR-007** (P-IV): All CSS and JavaScript MUST live in external files (`index.css`, `index.js`). Inline `<style>`, `style="..."` attributes, `<script>` bodies, and inline event handlers (`onclick=`, etc.) are BANNED.
- **FR-008** (P-IV): Any third-party stylesheet or script (currently Font Awesome from `cdnjs.cloudflare.com`) MUST carry an `integrity=` (SRI) hash, `crossorigin="anonymous"`, and `referrerpolicy="no-referrer"`.
- **FR-009** (P-IV): All strings that could originate from user input, network responses, or persisted storage MUST be inserted into the DOM via `textContent`, `createElement`, or `setAttribute`. `innerHTML`, `outerHTML`, `document.write`, and `insertAdjacentHTML` are BANNED for user-controlled data.
- **FR-010** (P-IV): Data hydrated from `localStorage` MUST be validated by a strict schema that whitelists keys, enforces per-field types, caps string lengths, and rejects the entire entry on failure. The reference implementations are `isValidCartItem` and `isValidCustomer` in `index.js`.
- **FR-011** (P-IV): The following validation constants MUST hold: phone regex `/^[6-9]\d{9}$/`; pincode regex `/^[1-9]\d{5}$/`; `MAX_QTY_PER_ITEM = 99`; `MAX_CART_ITEMS = 50`; `MAX_MSG_LEN = 3800`.
- **FR-012** (P-IV): `Math.random()` MUST NOT appear anywhere in the production JavaScript. Randomness MUST use `crypto.getRandomValues()`.
- **FR-013** (P-IV): Every `<a target="_blank">` MUST carry `rel="noopener noreferrer"`.
- **FR-014** (P-IV): Privacy-sensitive APIs (currently geolocation) MUST be triggered by explicit user action (button click) and MUST NOT be auto-requested on page load.
- **FR-015** (P-V): Cart state MUST be persisted to `localStorage` under the versioned key `krishidakshina.cart.v1`, and customer state MUST be persisted under `krishidakshina.customer.v1`. A schema change MUST bump the version segment.
- **FR-016** (P-V): The site MUST NOT set cookies, MUST NOT register a service worker, and MUST NOT ship analytics or tracking pixels.
- **FR-017** (P-V): Orders MUST be placed by opening `https://wa.me/<WHATSAPP_NUMBER>?text=<encoded>` in a new tab. The composed message MUST include a header, customer identity, delivery address, per-line items with unit price and subtotal, an order total, and an optional map link. The encoded message MUST be at most `MAX_MSG_LEN = 3800` chars; oversize MUST block the handoff with an inline error.
- **FR-018** (P-V clause 6(a)): The visible contact form MUST submit via a WhatsApp handoff — on submit, the page opens `https://wa.me/${WHATSAPP_NUMBER}?text=${encoded}` in a new tab with a formatted `💬 *New Enquiry – KrishiDakshina*` message. No `fetch` / `XMLHttpRequest` MAY be issued from the page origin. Any migration to a third-party form endpoint (Web3Forms, Formspree, etc.) MUST follow the constitution v2.1.0 "Contact-form migration procedure" governance rule.
- **FR-019** (P-V): The only external HTTP endpoint the JavaScript is permitted to `fetch()` in v1 is `https://api.postalpincode.in/pincode/<pincode>` for the pincode → city/state lookup. Its failure MUST NOT block order placement.
- **FR-020** (P-II): Product images MUST be optimized to ≤ 200 KB each at 600 × 600 (AVIF/WebP preferred with JPEG/PNG fallback). Below-the-fold images MUST use `loading="lazy"` and reserve layout space via `width`/`height`. Total shipped JS per page MUST be ≤ 50 KB gzipped; total shipped CSS MUST be ≤ 30 KB gzipped.
- **FR-021** (P-II): The page MUST meet LCP < 2.0 s, CLS < 0.1, and INP < 200 ms on the simulated 4G Fast profile.
- **FR-022** (P-II): Fonts MUST be loaded with `font-display: swap` and served either from `self` or from the allow-listed origins (`fonts.googleapis.com` / `fonts.gstatic.com`).
- **FR-023** (P-VI): Every release MUST pass the CI gates in Principle VI: HTML validation, internal link check, Lighthouse mobile ≥ 90 on all four categories, zero serious/critical axe-core violations, bundle-size check against the P-II budgets, CSP hygiene scan (no `'unsafe-inline'`, no `'unsafe-eval'`), and a static scan showing zero `Math.random(` occurrences.
- **FR-024** (Product catalog): The Products section MUST render exactly 12 product cards in v1. Each card MUST show a category label, product name, description, price with unit, an image with `data-fallback` emoji, and a keyboard-focusable Add-to-cart button.
- **FR-025** (Cart drawer): The navbar cart button MUST show a badge with the total quantity across all lines. Activating it MUST open a drawer with per-line quantity controls, a running subtotal per line, a grand total, a "Clear cart" action, and the delivery form + "Order via WhatsApp" CTA.
- **FR-026** (Missing image handling): If a `product_images/<file>` request returns 404 or fails, the card MUST show its `data-fallback` emoji rather than a broken-image icon. This behavior MUST be wired via a delegated event listener in `index.js` (no inline `onerror=` handler, per FR-007).
- **FR-027** (Navigation): The site MUST provide smooth-scroll for in-page anchor links, an active-section indicator in the navbar, a mobile hamburger menu for viewports < 768 px, and a floating scroll-to-top button that appears once the visitor scrolls past the hero.

### Key Entities *(include if feature involves data)*

- **Product** *(hard-coded in `index.html` for v1; no admin UI)*: A single catalog item.
  - Attributes: name, category (Vegetables, Dairy & Eggs, Leafy Greens, Bakery, Fruits, Dairy, Grains, Dry Fruits, Spices), description, price (INR), unit label (e.g., `/ kg`, `/ dozen`, `/ 250 g`), image path (`product_images/<file>`), `data-fallback` emoji, optional badge (`Organic`, `Fresh Pick`, `Seasonal`, `Bestseller`).
  - Cardinality in v1: exactly 12.
- **CartItem** *(persisted in `localStorage` under `krishidakshina.cart.v1`)*: A single line in the cart.
  - Attributes: product identity (name/price/unit — sufficient to reconstruct the WhatsApp message), quantity (integer, `1 ≤ qty ≤ 99`).
  - Aggregate constraint: at most 50 distinct lines.
  - Lifecycle: created on first `+` click, incremented on subsequent `+` clicks, decremented via drawer `-` button, removed when qty hits 0 or on "Clear cart".
- **Customer** *(persisted in `localStorage` under `krishidakshina.customer.v1`)*: The delivery-form state.
  - Attributes: name, phone (`/^[6-9]\d{9}$/`), addressLine1 (required), addressLine2 (optional), pincode (`/^[1-9]\d{5}$/`), city (auto-filled from pincode lookup if empty), notes, optional geolocation pin (`{lat, lng}` used to build a Google Maps link).
  - Persistence: all fields have per-field length caps; invalid rehydrated values are dropped.
- **OrderMessage** *(ephemeral; assembled at click time)*: The plain-text payload sent to WhatsApp.
  - Composition: header (`🛒 *New Order – KrishiDakshina*`), customer block, delivery block, cart line-items (name × qty × unit price = line total), grand total, optional map pin link.
  - Length constraint: encoded ≤ `MAX_MSG_LEN` (3800 chars).
  - Delivery: `https://wa.me/<WHATSAPP_NUMBER>?text=<encodeURIComponent(message)>` opened in a new tab.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001** (P-II): On a simulated 4G Fast connection with 4× CPU throttling (Lighthouse mobile), the page reaches Largest Contentful Paint in under 2.0 seconds.
- **SC-002** (P-II): Total JavaScript shipped per page is ≤ 50 KB gzipped, total CSS shipped per page is ≤ 30 KB gzipped, and no product image exceeds 200 KB.
- **SC-003** (P-II): Cumulative Layout Shift stays below 0.1 and Interaction to Next Paint stays below 200 ms on the same profile.
- **SC-004** (P-I, P-VI): An automated accessibility scan (axe-core or Pa11y) reports zero serious or critical violations against the shipped page.
- **SC-005** (P-VI): Lighthouse mobile scores ≥ 90 on Performance, Accessibility, Best Practices, and SEO.
- **SC-006** (P-IV, P-VI): A CSP hygiene scan of the served page finds neither `'unsafe-inline'` nor `'unsafe-eval'` in `script-src` or `style-src`.
- **SC-007** (P-IV, P-VI): A static scan of the shipped JavaScript finds zero occurrences of `Math.random(`.
- **SC-008** (Business flow): A visitor with a stable network can go from landing to opening the pre-composed WhatsApp message in under 3 minutes end-to-end (browse ≥ 1 product, add to cart, fill delivery form, click Order via WhatsApp).
- **SC-009** (P-V): Zero HTTP requests leave the browser during a full session that does NOT involve activating the pincode lookup or the WhatsApp handoff — no analytics beacons, no service-worker fetches, no third-party pixels.
- **SC-010** (Resilience): With `product_images/*` returning 404 and `api.postalpincode.in` unreachable, a visitor can still complete the flow in SC-008 (fallback emojis render, City field is typed manually, order handoff succeeds).
- **SC-011** (P-IV): Rehydrating `localStorage` values that violate the FR-011 constraints (bad qty, unknown keys, non-string address) results in the offending entries being dropped and the app starting in a clean state — never in a crash or a corrupted UI.

## Assumptions

- **A-01** — The site is deployed as static files on GitHub Pages with the tracked `CNAME` (`krishidakshina.in`). No CDN in front, no reverse proxy that could inject headers.
- **A-02** — The target audience uses modern evergreen browsers (latest two stable Chrome/Firefox/Safari/Edge plus Chrome on Android and Safari on iOS). Older engines get graceful degradation, not polyfills.
- **A-03** — Product catalog is small (12 items) and hand-maintained directly in `index.html` for v1. No admin/product-management UI is in scope.
- **A-04** — Order fulfilment (packing, delivery, payment collection) happens over WhatsApp between the business and the visitor. The site is not responsible for payment, tracking, or fulfilment beyond composing the message.
- **A-05** — The contact form is a WhatsApp handoff (consistent with the order flow); migration to a third-party form endpoint is a MINOR-amendment change per constitution v2.1.0 "Contact-form migration procedure", and migration to a backend/serverless endpoint is a MAJOR-amendment change.
- **A-06** — `localStorage` is available in the visitor's browser. Private/incognito sessions may reset it between tabs; that is acceptable in v1.
- **A-07** — Pincode lookup latency and success rate are best-effort. The visitor can always fall back to typing the city manually.
- **A-08** — English is the sole content language in v1. Hindi and any other language are out of scope; reintroducing them is a MINOR-amendment gate per the constitution's Governance section.
- **A-09** — Testimonial names/quotes and any placeholder people-photos shown in the site are treated as non-personally-identifying content pending the clarification below.
- **A-10** — The runtime origin allow-list in the constitution (`cdnjs.cloudflare.com`, `fonts.googleapis.com`, `fonts.gstatic.com`, `api.postalpincode.in`, `wa.me`) is exhaustive for v1. Adding an origin requires a CSP amendment and a constitution amendment.

## Non-Goals (v1)

- **NG-01** — No payment processing anywhere on the site. All orders finalize on WhatsApp.
- **NG-02** — No account system, no login, no per-user history beyond `localStorage`.
- **NG-03** — No server-side order storage or admin dashboard.
- **NG-04** — No multi-language support in v1 (English only). Hindi is deferred per the constitution's Governance section.
- **NG-05** — No direct email/SMTP delivery of the contact form; enquiries route through WhatsApp handoff. Direct-inbox delivery via a third-party form endpoint is scoped as post-v1 (tasks.md T020a).
- **NG-06** — No admin/product-management UI; the product list is hard-coded in `index.html`.
- **NG-07** — No analytics, telemetry, or third-party tracking pixels. No service worker.
- **NG-08** — No native app; this is a single web page.

## Open Questions

The following items were explicitly flagged as unresolved and are intentionally NOT answered here; `/speckit.clarify` should address them before `/speckit.plan`.

- **OQ-01** — **RESOLVED 2026-07-18**: Canonical brand is **"KrishiDakshina"** (single word, capital K and D). The site markup, meta tags, WhatsApp message header, and `localStorage` namespace have been rebranded from "Gut Point" / `gutpoint.*` to "KrishiDakshina" / `krishidakshina.*`. The email address `hello@gutpoint.com` is preserved verbatim as a placeholder pending OQ-02 (owner-supplied contact details). See constitution v2.1.0 Sync Impact Report.
- **OQ-02**: [NEEDS CLARIFICATION — DEFERRED: Are the current contact details real production values or placeholders? Specifically: WhatsApp number `919876543210` (hard-coded in `index.js` as `WHATSAPP_NUMBER`), phone `+91 98765 43210`, email `hello@gutpoint.com`, and street address "123 Green Market Lane, Mumbai". Owner will supply production values later; the email intentionally keeps the old brand string as a placeholder and will be updated together with the other contact details.]
- **OQ-03**: [NEEDS CLARIFICATION — DEFERRED: Product image assets — 12 filenames are wired up (`tomato.jpg`, `eggs.jpg`, `spinach.jpg`, `bread.jpg`, `mango.jpg`, `milk.jpg`, `carrot.jpg`, `yogurt.jpg`, `brown-rice.jpg`, `avocado.jpg`, `mixed-nuts.jpg`, `turmeric.jpg`) but the `product_images/` folder is not yet in the repo. Owner will provide images later; the emoji fallback remains the interim behaviour.]
- **OQ-04** — **RESOLVED 2026-07-18 (as placeholders)**: Testimonials (Priya Sharma, Rahul Mehta, Ananya Iyer at [index.html:494-521](../../index.html)) are confirmed placeholders. Per constitution v2.1.0 Governance ("Testimonial permission requirement") they MUST be labeled as example content in a source-file comment (already done in `index.html`) and MUST NOT be published as production content without documented, written permission from real customers. Launch is BLOCKED from publishing them as-is.
- **OQ-05**: **RESOLVED 2026-07-18** — Contact form behaviour is a **WhatsApp handoff** (Option C), consistent with the order flow (Principle V clause 6(a)). Reuses the already-declared `wa.me` origin; no new external origin, no CSP change, no constitution amendment required (clause 6(a) is the pre-approved escape from the migration procedure). WhatsApp's native phone verification serves as the anti-spam mechanism. A future migration to a third-party inbox endpoint (Web3Forms recommended) is scoped as post-v1 task T020a in tasks.md and would follow the full "Contact-form migration procedure" governance rule (constitution MINOR amendment + CSP allow-list amendment + anti-spam declaration).

## Constitutional Impact Summary

| Principle | Story / Requirement Coverage |
|-----------|------------------------------|
| **P-I** Accessibility-First (NON-NEGOTIABLE) | US-1, US-2, US-3, US-5, US-6; FR-003, FR-004, FR-005; SC-004, SC-005 |
| **P-II** Performance Budget | US-1, US-6; FR-020, FR-021, FR-022; SC-001, SC-002, SC-003 |
| **P-III** Static-First & Framework-Minimal | US-1; FR-001, FR-002 |
| **P-IV** Security-by-Default (NON-NEGOTIABLE) | US-1, US-2, US-3, US-4, US-6; FR-006 – FR-014; SC-006, SC-007, SC-011 |
| **P-V** Client-Only Architecture (NON-NEGOTIABLE) | US-1, US-2, US-3, US-4, US-5; FR-015 – FR-019; SC-009, SC-010 |
| **P-VI** Verifiable Releases | US-6 (indirectly); FR-023; SC-004 – SC-007 |

## Downstream Notes

- This spec is **descriptive** of the current baseline, not prescriptive of new work. `/speckit.plan` should treat it as the "as-is" state and only design work for the Open Questions above or for follow-ups filed against the constitution's Sync Impact Report (`CONTACT_DETAILS_CENTRALIZATION`, `PRODUCT_IMAGES`). `BRAND_DOMAIN_RECONCILIATION` was resolved in constitution v2.1.0 (2026-07-18); `CONTACT_FORM_FATE` was resolved on 2026-07-18 via the WhatsApp handoff (P-V clause 6(a)).
- Any future change that touches `index.html`, `index.css`, or `index.js` MUST cite which functional requirement or user story in this spec it modifies, and MUST re-run the CI gates in FR-023.
