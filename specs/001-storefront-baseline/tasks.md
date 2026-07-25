---

description: "Task list — Storefront Baseline (KrishiDakshina v1). Brownfield / reverse-engineered."
---

# Tasks: Storefront Baseline (KrishiDakshina v1)

**Feature Branch**: `001-storefront-baseline`

**Feature Directory**: [specs/001-storefront-baseline/](./)

**Inputs**:
- Spec: [spec.md](./spec.md)
- Plan: [plan.md](./plan.md)
- Research: [research.md](./research.md)
- Data model: [data-model.md](./data-model.md)
- Contracts: [contracts/csp.md](./contracts/csp.md) · [contracts/localstorage-schema.md](./contracts/localstorage-schema.md) · [contracts/postal-pincode-api.md](./contracts/postal-pincode-api.md) · [contracts/whatsapp-handoff.md](./contracts/whatsapp-handoff.md)
- Quickstart: [quickstart.md](./quickstart.md)
- Constitution: [.specify/memory/constitution.md](../../.specify/memory/constitution.md) (v2.1.0)

**Nature**: This feature is a **brownfield / reverse-engineered baseline**. The site is already shipped ([index.html](../../index.html), [index.css](../../index.css), [index.js](../../index.js)). Section A snapshots what already works; Section B enumerates outstanding v1 gaps (audit + CI + follow-ups from the plan's Complexity Tracking).

**Tests**: Not requested; TDD is not applicable to a shipped baseline. Section B audits produce report artifacts under `specs/001-storefront-baseline/audits/` — these are the "tests" for the outstanding work.

## Legend

- `- [x] DONE` — already shipped; **do not re-execute**. Included for baseline traceability only.
- `- [ ] Txxx` — outstanding v1 task to execute (Section B). Follows the mandated `- [ ] [TaskID] [P?] [Story?] Description` format.
- `[P]` — parallelizable (different files, no in-flight dependency).
- `Depends on: Txxx` — explicit predecessor.
- `Size: S/M/L` — subjective effort marker (no hour estimates per the request).
- **Principle refs**: **P-I** Accessibility · **P-II** Performance · **P-III** Static-First · **P-IV** Security · **P-V** Client-Only · **P-VI** Verifiable Releases.
- **Deferred item IDs** from the plan / constitution Sync Impact Report:
  - `P-VI FAIL` (Complexity Tracking, plan.md)
  - `P-I AUDIT`, `P-II BASELINE` (Complexity Tracking, plan.md)
  - `CONTACT_DETAILS_CENTRALIZATION` (constitution follow-up)
  - `BRAND_DOMAIN_RECONCILIATION` (**RESOLVED 2026-07-18** in constitution v2.1.0 — canonical brand is "KrishiDakshina", localStorage namespace migrated to `krishidakshina.*`)
  - `PRODUCT_IMAGES` (constitution follow-up, spec OQ-03)
  - `CONTACT_FORM_FATE` (**RESOLVED 2026-07-18** — Option C: WhatsApp handoff via `wa.me`, per Principle V clause 6(a); no new origin, no CSP change, no constitution amendment required)
  - `OQ-02` contact details real vs. placeholder · `OQ-04` testimonials **RESOLVED 2026-07-18 as placeholders** (new governance rule in constitution v2.1.0) · `OQ-05` contact-form fate **RESOLVED 2026-07-18** (Option C — WhatsApp handoff)

---

# Section A — Baseline (Already Shipped) — `[x] DONE`

Cited against the current tree. Each item names the file and line range that satisfies it so a reviewer can re-verify against the shipped code. **These are not re-executable tasks — they are the frozen baseline snapshot.**

## A-US1: Browse and Add to Cart (Priority: P1) — DONE

**Story goal**: Visitor lands on the home page, browses the 12-product grid, adds items to a cart drawer that persists in `localStorage`.

**Independent test (already passing manually per [quickstart.md](./quickstart.md))**: Load with empty `localStorage`, click `+` on any product, confirm badge shows `1`, drawer contains the line, reload and confirm persistence.

- [x] DONE A001 [US1] Semantic product grid with 12 `<article class="pcard">` cards — [index.html:252-452](../../index.html) (Cards 1–12, one per commented block)
- [x] DONE A002 [US1] Each card carries category badge, name, description, price/unit, `<img alt=... data-fallback=...>`, and keyboard-focusable `<button class="btn-add" aria-label="Add to cart">` — [index.html:257-268](../../index.html) (representative Card 1)
- [x] DONE A003 [US1] Product-image `error` fallback via delegated capture-phase listener → replaces `<img>` with `data-fallback` emoji via `textContent` (no inline `onerror`, XSS-safe) — [index.js:11-30](../../index.js) (`applyImgFallback` + document-level error listener + catch-up pass)
- [x] DONE A004 [US1] Add-to-cart handler reads name/price/unit from card DOM, caps `qty ≤ MAX_QTY_PER_ITEM` (99), caps distinct lines ≤ `MAX_CART_ITEMS` (50) — [index.js:271-296](../../index.js) (`.btn-add` click handler)
- [x] DONE A005 [US1] Cart state persists to `krishidakshina.cart.v1` with schema validation on load (whitelisted keys, type checks, length caps, prototype-pollution-safe fresh-object copy) — [index.js:94-132](../../index.js) (`isValidCartItem` + hydration loop) and `saveCart()` at [index.js:134-137](../../index.js)
- [x] DONE A006 [US1] Cart badge shows total quantity across all lines; toggles `.has` class when non-zero — [index.js:157-160](../../index.js) (`renderCart`)
- [x] DONE A007 [US1] Card visuals + hover states use only external CSS (no inline `style="..."`) — [index.css:1-2](../../index.css) (file-header comment confirms the deliberate decision to keep styles external)

## A-US2: Adjust or Clear the Cart (Priority: P1) — DONE

**Story goal**: Inside the drawer, `+` / `-` per line, remove on qty→0, "Clear cart" button empties everything.

**Independent test**: With ≥ 2 lines at various qty, exercise `+`, `-`, and Clear; badge/drawer/`localStorage` remain consistent.

- [x] DONE A010 [US2] Cart drawer + overlay markup with `role="dialog" aria-modal="true" aria-label="Shopping cart"` — [index.html:731-736](../../index.html)
- [x] DONE A011 [US2] Drawer open/close handlers (`cartBtn` → open; `cartClose` + overlay click → close) with body-scroll lock — [index.js:139-149](../../index.js)
- [x] DONE A012 [US2] Per-line quantity controls built with `createElement` + `textContent` (no innerHTML for storage-derived name/unit) — [index.js:170-227](../../index.js) (row/info/ctrl/priceEl construction)
- [x] DONE A013 [US2] `+` / `-` handlers respect `MAX_QTY_PER_ITEM` cap; qty→0 removes the line via `delete cart[k]` — [index.js:239-253](../../index.js)
- [x] DONE A014 [US2] "Clear cart" empties in place via `for..of Object.keys(cart) { delete cart[k] }` (no full-replace of the reference) — [index.js:299-302](../../index.js) (`btnClearCart` handler)
- [x] DONE A015 [US2] Every drawer mutation calls `renderCart()` which calls `saveCart()` synchronously (mutations hit `localStorage` immediately) — [index.js:161](../../index.js) (`saveCart()` inside `renderCart`)

## A-US3: Provide Delivery Details (Priority: P1) — DONE

**Story goal**: Delivery form (Name, Phone, Address 1/2, Pincode, City, Notes) persists between visits; pincode → city lookup; opt-in geolocation.

**Independent test**: Fill valid values, reload, confirm re-hydration. Enter invalid phone/pincode, confirm submission is blocked.

- [x] DONE A020 [US3] Delivery-form markup inside cart drawer with `<label for=...>` per field, native `required` / `maxlength` / `autocomplete` / `pattern` — [index.html:748-780](../../index.html)
- [x] DONE A021 [US3] `isValidCustomer` strict validator + hydration from `krishidakshina.customer.v1` with per-field length checks — [index.js:339-378](../../index.js) (`isValidCustomer` + load block)
- [x] DONE A022 [US3] `saveCustomer()` writes back on every `input` event, slicing each field to its `MAX_*_LEN` cap — [index.js:380-391](../../index.js)
- [x] DONE A023 [US3] Regex validators `phone /^[6-9]\d{9}$/`, `pincode /^[1-9]\d{5}$/`; `validateForm()` toggles `.invalid` only after the user has typed (avoids all-red first-open) — [index.js:328-336](../../index.js) and [index.js:407-424](../../index.js)
- [x] DONE A024 [US3] Pincode lookup: fires only when pincode matches regex AND City is empty; uses `AbortController` to cancel in-flight requests; slices `District`/`State` before touching the DOM; failure is non-blocking — [index.js:432-461](../../index.js)
- [x] DONE A025 [US3] Geolocation button (opt-in only; never auto-requested); options `{enableHighAccuracy:true, timeout:8000, maximumAge:60000}`; captures `{lat,lng}` in-memory only (`geoLoc`, never persisted); permission-denied / unavailable / timeout paths each surface a non-blocking hint — [index.js:464-495](../../index.js)
- [x] DONE A026 [US3] `<output aria-live="polite">` sink for hints so screen readers announce the pincode auto-fill and geolocation state — [index.html:782](../../index.html) (`#delivHint output`) + [index.js:401-405](../../index.js) (`setHint`)

## A-US4: Place Order via WhatsApp (Priority: P1) — DONE

**Story goal**: With ≥ 1 cart line AND valid delivery form, "Order via WhatsApp" composes a plain-text message and opens `https://wa.me/<num>?text=...` in a new tab.

**Independent test**: Valid cart + valid form → click Order → new tab opens with the expected URL; decoded text contains header, all lines, customer name/phone/address; encoded length ≤ 3800.

- [x] DONE A030 [US4] `updateOrderBtnState()` disables `btnWaOrder` unless `cart` has items AND `validateForm()` passes — [index.js:427-430](../../index.js)
- [x] DONE A031 [US4] Message composition via pure string concatenation from validated state (no HTML injection surface) — [index.js:501-535](../../index.js)
- [x] DONE A032 [US4] `MAX_MSG_LEN = 3800` guard: oversized encoded messages are blocked with an inline error, no navigation — [index.js:319](../../index.js) (constant) + [index.js:537-540](../../index.js) (guard)
- [x] DONE A033 [US4] `window.open(url, '_blank', 'noopener,noreferrer')` — [index.js:542-545](../../index.js)
- [x] DONE A034 [US4] Optional geolocation pin appended as `https://maps.google.com/?q=<lat>,<lng>` when `geoLoc` is set — [index.js:524-526](../../index.js)
- [x] DONE A035 [US4] Order message layout matches [contracts/whatsapp-handoff.md](./contracts/whatsapp-handoff.md): header `🛒 *New Order – KrishiDakshina*`, customer block, delivery block with notes + map, per-line items, grand total — [index.js:513-533](../../index.js)

## A-US5: Contact the Business (Priority: P2) — DONE

**Story goal**: Contact section shows address / phone / email / hours / WhatsApp CTA plus a client-side-simulated message form (no backend).

**Independent test**: Submit contact form → spinner → new tab opens on `wa.me/919876543210` with a pre-filled enquiry message → no `fetch` / `XMLHttpRequest` fires from the page origin (verify via DevTools Network tab; only the external navigation to `wa.me` appears, and only in the newly opened tab).

- [x] DONE A040 [US5] Contact info cards (Address, Phone, Email, Hours) with icon + `<a href="tel:..." class="link-plain">` and `<a href="mailto:...">` — [index.html:588-618](../../index.html)
- [x] DONE A041 [US5] Direct WhatsApp CTA `<a class="btn-wa" target="_blank" rel="noopener noreferrer">` at [index.html:621-625](../../index.html)
- [x] DONE A042 [US5] `<form id="cf" novalidate>` with labelled inputs and required-marker copy — [index.html:634-676](../../index.html)
- [x] DONE A043 [US5] Submit handler is a **WhatsApp handoff** (spinner → build `💬 *New Enquiry – KrishiDakshina*` message → `window.open('https://wa.me/${WHATSAPP_NUMBER}?text=...')` → success toast "WhatsApp opened with your message") — [index.js:548-630](../../index.js) — **no `fetch()` / no `XMLHttpRequest`** from the page origin; reuses the already-declared `wa.me` handoff (P-V clause 6(a)). Superseded the earlier client-side simulation on 2026-07-18 as part of the OQ-05 resolution.

## A-US6: Navigate on Any Device (Priority: P2) — DONE

**Story goal**: Fixed navbar with smooth-scroll + active-section indicator; mobile hamburger < 768 px; floating scroll-to-top; fade-in-on-scroll; particle background; animated truck — all reduced-motion-safe.

**Independent test**: Resize < 768 px → hamburger appears; enable Reduce Motion → truck/particles/fade-in stop.

- [x] DONE A050 [US6] Semantic `<nav id="nav">` landmark with skip-free anchor list — [index.html:44-84](../../index.html)
- [x] DONE A051 [US6] Sticky-on-scroll navbar via `.on` class toggle at `scrollY > 40` — [index.js:55-58](../../index.js)
- [x] DONE A052 [US6] Active-section indicator: `scroll` listener sets `.cur` on the anchor whose section is under `scrollY + 110` — [index.js:62-67](../../index.js)
- [x] DONE A053 [US6] Mobile hamburger toggle + delegated close-on-link-click for `#mobNav a` (no inline `onclick="mClose()"` — CSP-safe) — [index.js:70-75](../../index.js)
- [x] DONE A054 [US6] Floating scroll-to-top button (`#top-btn`) appears at `scrollY > 300`; click → smooth scroll to top — [index.js:57 + 78](../../index.js)
- [x] DONE A055 [US6] `IntersectionObserver` at threshold `0.10` staggers `.fi` fade-in by `i * 90 ms`; unobserves after first hit (no thrash) — [index.js:81-89](../../index.js)
- [x] DONE A056 [US6] `@media (prefers-reduced-motion: reduce)` in CSS disables the truck animation, particle motion, and hover-motion effects — [index.css:240](../../index.css)
- [x] DONE A057 [US6] Mobile breakpoint at `@media(max-width:768px)` — [index.css:576](../../index.css)
- [x] DONE A058 [US6] Hero particles (20) use `randFloat()` (CSPRNG) — no `Math.random(` anywhere in production JS — [index.js:32-39](../../index.js) (`randFloat`) + [index.js:571-582](../../index.js) (particle loop)

## A-CC: Cross-cutting Security & Performance — DONE

- [x] DONE A060 Strict `<meta http-equiv="Content-Security-Policy">` with `default-src 'self'`, `script-src 'self'` (no `'unsafe-inline'`, no `'unsafe-eval'`), `connect-src 'self' https://api.postalpincode.in`, `frame-ancestors 'none'`, `object-src 'none'`, `form-action 'self'`, `base-uri 'self'` — [index.html:24](../../index.html)
- [x] DONE A061 `<meta name="referrer" content="strict-origin-when-cross-origin">` — [index.html:25](../../index.html)
- [x] DONE A062 Font Awesome 6.5.0 pinned with SRI `integrity`, `crossorigin="anonymous"`, `referrerpolicy="no-referrer"` — [index.html:29-32](../../index.html)
- [x] DONE A063 Google Fonts uses `preconnect` hints and `display=swap` — [index.html:34-36](../../index.html)
- [x] DONE A064 `'use strict'` at top of script; all handlers registered via `addEventListener` (no inline `onclick=`, no inline `<script>` bodies) — [index.js:9](../../index.js) plus every listener registration site
- [x] DONE A065 XSS-safe DOM: every storage-derived / user-derived string is inserted via `textContent` or `.value` (never `innerHTML`); comments explicitly justify this at [index.js:180-183](../../index.js) and [index.js:288](../../index.js)
- [x] DONE A066 Versioned `localStorage` keys `krishidakshina.cart.v1` and `krishidakshina.customer.v1` (schema-change gate) — [index.js:93 + 315](../../index.js)
- [x] DONE A067 Copyright year auto-fills via `textContent = new Date().getFullYear()` (no inline JS) — [index.js:51](../../index.js) + [index.html:713](../../index.html) (`<span id="yr">`)
- [x] DONE A068 Deployment: `CNAME` file pins the custom domain `krishidakshina.in` for GitHub Pages — [CNAME](../../CNAME)
- [x] DONE A069 `product_images/*` requests are permitted to 404 by design; the emoji fallback (A003) is the resilience path — [spec.md `SC-010`](./spec.md) codifies this

**Section A total: 49 DONE items** across 6 user stories + cross-cutting group.

---

# Section B — Outstanding v1 Gaps (To Do)

Every task cites (a) the principle it satisfies, (b) the acceptance criterion or file to touch, and (c) the deferred-item ID from the plan / constitution follow-ups. Dependency-ordered top-to-bottom; a fresh reviewer executing the file in order lands on a compliant v1 release.

## Phase B1 — Release Verification & CI Gates (P-VI FAIL) 🚨 HIGHEST PRIORITY

**Purpose**: Wire the CI gates that the constitution's Principle VI mandates. This is the sole FAIL gate on the plan — the site is otherwise principle-compliant by construction, but without CI the compliance can silently regress. All five tasks below author (or extend) files under `.github/workflows/`; none of them touch `index.html` / `index.js` and can therefore be composed in parallel where indicated, but a single reviewer will typically batch them into one PR.

**Independent test criterion for Phase B1**: Open a throw-away PR that (a) removes the CSP `<meta>`, (b) adds a `Math.random()` call, (c) balloons `index.js` past 50 KB gzipped, (d) introduces a serious a11y violation. Each gate must fail the PR independently.

- [ ] T001 [P] Create `.github/workflows/lighthouse.yml` running Lighthouse mobile against a static-server preview on every PR against the default branch; assert Performance / Accessibility / Best-Practices / SEO **all ≥ 90**; upload the HTML report as a workflow artifact. **Principle**: P-VI clauses 1–2. **Deferred ID**: `P-VI FAIL`. **Acceptance**: [spec.md `FR-023` + `SC-005`](./spec.md). **Test evidence**: `lighthouse-report.html` artifact attached to the PR run. **Size**: M. **Depends on**: none.
- [ ] T002 [P] Add an axe-core (or Pa11y) job to the same `lighthouse.yml` (or a sibling `.github/workflows/a11y.yml`) that fails the build on any **serious** or **critical** violation against the served page. **Principle**: P-I + P-VI clause 3. **Deferred ID**: `P-VI FAIL`, `P-I AUDIT`. **Acceptance**: [spec.md `SC-004`](./spec.md). **Test evidence**: axe JSON report in workflow artifacts; PR annotation on failure. **Size**: M. **Depends on**: T001 (shares the preview-server bring-up step).
- [ ] T003 [P] Add a **CSP-hygiene grep step** to CI that fails the build if `index.html` contains `'unsafe-inline'` or `'unsafe-eval'` anywhere inside the `<meta http-equiv="Content-Security-Policy">` value. Reference regex: `grep -nE "unsafe-(inline|eval)" index.html && exit 1 || exit 0`. **Principle**: P-IV clause 1 + P-VI clause 4. **Deferred ID**: `P-VI FAIL`. **Acceptance**: [spec.md `SC-006`](./spec.md). **Test evidence**: CI log line "CSP hygiene: PASS/FAIL". **Size**: S. **Depends on**: none (writes to a workflow file only).
- [ ] T004 [P] Add a **`Math.random(` grep step** against `index.js` (and any future JS file) that fails the build on any occurrence. Reference regex: `grep -n "Math\.random(" index.js && exit 1 || exit 0`. **Principle**: P-IV clause 7 + P-VI clause 5. **Deferred ID**: `P-VI FAIL`. **Acceptance**: [spec.md `SC-007`](./spec.md). **Test evidence**: CI log line "Math.random gate: PASS/FAIL". **Size**: S. **Depends on**: none.
- [ ] T005 [P] Add a **bundle-size guard** step: `index.js` gzipped **< 50 KB**, `index.css` gzipped **< 30 KB**. Use `gzip -c index.js | wc -c` (or a portable Node equivalent) and fail the build if either exceeds its budget. **Principle**: P-II + P-VI. **Deferred ID**: `P-VI FAIL`, `P-II BASELINE`. **Acceptance**: [spec.md `FR-020` + `SC-002`](./spec.md). **Test evidence**: CI log line reporting the two gz sizes. **Size**: S. **Depends on**: none.

**Checkpoint B1**: All five gates green on a no-op PR; each gate proven to fail on a deliberately regressed PR. Principle VI is now enforced in CI.

---

## Phase B2 — Accessibility Audit (P-I AUDIT)

**Purpose**: Produce the audit artifacts Principle I requires as an artifact of every release. The site is expected to pass by construction, but the audit itself is the deliverable.

- [ ] T006 [P] Run **axe-core** locally against `index.html` (served through `python -m http.server` or `npx serve`); capture every violation with severity; remediate all **serious**/**critical**; write the report to [specs/001-storefront-baseline/audits/a11y-<YYYY-MM-DD>.md](./audits/) including the axe-core version, URL, viewport, and remediation notes for each finding. **Principle**: P-I + P-VI clause 3. **Deferred ID**: `P-I AUDIT`. **Acceptance**: [spec.md `SC-004`](./spec.md) (zero serious/critical). **Test evidence**: the `audits/a11y-<date>.md` file itself + axe JSON checked into the same folder. **Size**: M. **Depends on**: none (can run in parallel with T001/T009). Runs against `index.html`, does not edit it.
- [ ] T007 Manual **keyboard-only walkthrough** covering the full flow: nav links → hero CTAs → product-grid `+` buttons → cart-icon → drawer qty controls → delivery-form fields → "Use my current location" → "Order via WhatsApp" → contact form. Every unreachable control or focus-trap finding becomes a follow-up task appended to Section B under its owning story. Record findings in [specs/001-storefront-baseline/audits/keyboard-<YYYY-MM-DD>.md](./audits/). **Principle**: P-I clauses 2–3 (WCAG 2.4.11, 2.4.13). **Deferred ID**: `P-I AUDIT`. **Acceptance**: [spec.md `FR-004`](./spec.md). **Test evidence**: `audits/keyboard-<date>.md` with pass/fail per control. **Size**: M. **Depends on**: T006 (fix trivial a11y issues first so the walk isn't dominated by low-hanging fruit).
- [ ] T008 [P] Verify the page at **200% zoom** and **400% zoom** in Chrome + Firefox; confirm no critical content clips or reflow-breaks (per WCAG 1.4.10). Record screenshots + findings in [specs/001-storefront-baseline/audits/zoom-<YYYY-MM-DD>.md](./audits/). **Principle**: P-I ("MUST remain usable at 200% zoom"). **Deferred ID**: `P-I AUDIT`. **Acceptance**: [spec.md `FR-004`](./spec.md). **Test evidence**: `audits/zoom-<date>.md` with 200 % + 400 % screenshots. **Size**: S. **Depends on**: none.
- [x] **DONE 2026-07-18** T008a [US6] Mobile-first CSS refactor + touch-target + input-attribute hygiene. Rewrote [index.css](../../index.css) responsive layer from **desktop-first `max-width`** (3 blocks) to **mobile-first `min-width`** (3 tiers: ≥481px, ≥769px, ≥1025px). Every layout that had a multi-column desktop default (`.hero-grid`, `.about-grid`, `.contact-grid`, `.row2`, `.rev-grid`, `.prod-grid`, `.highlights`, `.footer-top`, `.strip-row`, `.prod-hd`) now defaults to 1-col on phones and progressively adds columns via `min-width` queries. Elements previously visible-then-hidden on mobile (`.hero-vis`, `.about-vis`, `.nav-links`, `.nav-cta`) now default to `display:none` and reveal at ≥769px; `.hb` (hamburger) defaults visible and hides at ≥769px. Concrete defect fixes bundled: (a) `.qty-btn` in cart drawer bumped **26×26→ 36×36 px** (WCAG 2.5.5 headroom); (b) `.hb` hamburger gets `min-width:44px; min-height:44px` (WCAG 2.5.5 AAA); (c) [index.html](../../index.html) contact-form inputs (`#fn`, `#ln`, `#em`, `#ph`) gained `autocomplete="given-name|family-name|email|tel"` and `inputmode="email|tel"` matching the delivery-form pattern; (d) `<meta name="theme-color" content="#28a745">` added for Android/PWA browser-chrome color. **Principle**: P-I (mobile viewport support + touch targets) + P-II (single-CSS payload, no framework). **Deferred ID**: `P-I AUDIT`. **Acceptance**: [spec.md `FR-004`](./spec.md) + `SC-004`. **Test evidence**: post-change grep shows zero `max-width` media queries in [index.css](../../index.css); manual smoke-test at 360×640, 414×896, 768×1024, 1440×900. **Size**: L. **Depends on**: T006 (a11y baseline). **Follow-up**: T008b below tracks the axe/Lighthouse re-run after this refactor.
- [ ] T008b [US6] Re-run axe-core + Lighthouse mobile after the T008a mobile-first refactor to confirm no regression in a11y score and to capture the new mobile-viewport performance numbers. Append to the same `audits/a11y-<date>.md` and `audits/perf-<date>.md` files (Phase B3 T009). **Principle**: P-VI clauses 2–3. **Deferred ID**: `P-I AUDIT`, `P-II BASELINE`. **Acceptance**: no new serious/critical a11y findings; LCP ≤ T009 baseline. **Test evidence**: appended audit sections. **Size**: S. **Depends on**: T006, T009, T008a (DONE).

**Checkpoint B2**: `audits/` folder holds the three signed reports; Principle I is auditable per release, not just aspirational.

---

## Phase B3 — Performance Baseline (P-II BASELINE)

**Purpose**: Take the first numeric Lighthouse reading against the shipped site, then eliminate the two known CLS/perf smells (missing `loading="lazy"`, missing intrinsic image sizes). Must precede Phase B5 image work so the "before" numbers are on record.

- [ ] T009 [P] Run **Lighthouse mobile** (4G Fast throttling, 4× CPU) against a local static-server build; record LCP, CLS, INP, gz JS, gz CSS, per-image weight in [specs/001-storefront-baseline/audits/perf-<YYYY-MM-DD>.md](./audits/) with the raw JSON attached. **Principle**: P-II + P-VI clause 2. **Deferred ID**: `P-II BASELINE`. **Acceptance**: [spec.md `SC-001`, `SC-002`, `SC-003`, `SC-005`](./spec.md). **Test evidence**: `audits/perf-<date>.md` plus `audits/perf-<date>.json`. **Size**: S. **Depends on**: none. Read-only against source.
- [ ] T010 [US1] Add `loading="lazy"` **and** `decoding="async"` to below-the-fold `<img>` tags in `index.html` (all 12 product-card images at [index.html:257-452](../../index.html) qualify; the hero has no `<img>`). **Principle**: P-II. **Deferred ID**: `P-II BASELINE`, [spec.md `FR-020`](./spec.md). **Acceptance**: [spec.md `SC-001`](./spec.md) (LCP < 2.0 s). **Test evidence**: post-change diff + a follow-up Lighthouse run showing LCP + CLS steady or improved (appended to the same `perf-<date>.md`). **Size**: S. **Depends on**: T009 (need a "before" reading).
- [ ] T011 [US1] Add explicit `width` and `height` attributes to every `<img>` in `index.html` (the 12 product images MUST get intrinsic dimensions matching the target 600×600 source; SVGs at [index.html:50-55](../../index.html) and [index.html:666-671](../../index.html) already have them). This is the single biggest CLS-prevention lever the site can pull today. **Principle**: P-II clause "sized via `width`/`height` attributes to reserve layout space". **Deferred ID**: `P-II BASELINE`, [spec.md `FR-020`](./spec.md). **Acceptance**: [spec.md `SC-003`](./spec.md) (CLS < 0.1). **Test evidence**: post-change Lighthouse rerun (append to `perf-<date>.md`) showing CLS improvement. **Size**: S. **Depends on**: T010 (both edit `index.html`; **NOT** `[P]`). Do these two in the same commit if possible.

**Checkpoint B3**: One "before" Lighthouse artifact on file; two known perf smells fixed; a "after" run demonstrates the deltas. Numeric baseline is now in the audit trail.

---

## Phase B4 — Security Audit (P-IV verification)

**Purpose**: Confirm every Principle IV posture the code claims to hold — with the results committed as evidence, not just asserted. These are read-only grep + review passes; they do not touch shipped code. Safe to parallelize.

- [ ] T012 [P] Static scan: confirm **zero occurrences** of `innerHTML =`, `.outerHTML =`, `document.write`, or `insertAdjacentHTML` for user-derived data in `index.js`. Method: `grep -nE "innerHTML|outerHTML|document\.write|insertAdjacentHTML" index.js`, then review each hit (as of writing all hits are comments justifying the decision — see [index.js:41, 181-183, 288, 310](../../index.js)). Record the grep output + reviewer signoff in [specs/001-storefront-baseline/audits/xss-safe-dom-<YYYY-MM-DD>.md](./audits/). **Principle**: P-IV clause 4. **Deferred ID**: `P-I AUDIT` (delivered under the same audit umbrella). **Acceptance**: [spec.md `FR-009`](./spec.md). **Test evidence**: `audits/xss-safe-dom-<date>.md`. **Size**: S. **Depends on**: none.
- [ ] T013 [P] Confirm every `window.open` and external `<a target="_blank">` carries `rel="noopener noreferrer"`. Method: `grep -nE "target=\"_blank\"|window\.open" index.html index.js`, then review. Currently expected clean: [index.html:622](../../index.html) (WhatsApp CTA), [index.js:542-544](../../index.js) (`window.open(url, '_blank', 'noopener,noreferrer')`). Record in [specs/001-storefront-baseline/audits/rel-noopener-<YYYY-MM-DD>.md](./audits/). **Principle**: P-IV clause 8. **Deferred ID**: `P-I AUDIT`. **Acceptance**: [spec.md `FR-013`](./spec.md). **Test evidence**: `audits/rel-noopener-<date>.md`. **Size**: S. **Depends on**: none.
- [ ] T014 [P] Verify the **SRI hash** for the pinned Font Awesome 6.5.0 CSS at [index.html:30](../../index.html) matches the current cdnjs hash for that exact URL. Method: `curl -sL <url> | openssl dgst -sha384 -binary | openssl base64 -A` and compare. Record the URL, computed hash, and cdnjs source of truth in [specs/001-storefront-baseline/audits/sri-<YYYY-MM-DD>.md](./audits/). **Principle**: P-IV clause 3. **Deferred ID**: `P-I AUDIT`. **Acceptance**: [spec.md `FR-008`](./spec.md). **Test evidence**: `audits/sri-<date>.md`. **Size**: S. **Depends on**: none.
- [ ] T015 Evaluate migrating meta-CSP to **real HTTP response headers**. GitHub Pages does not support custom headers natively, so this task is scoped as an evaluation: (a) confirm the limitation, (b) enumerate candidate hosts that DO support headers (Cloudflare Pages via `_headers`, Netlify via `netlify.toml`, Vercel via `vercel.json`), (c) record cost / DX / deploy-flow trade-offs in [specs/001-storefront-baseline/audits/csp-http-header-<YYYY-MM-DD>.md](./audits/). No host migration in v1 — that would be a constitution amendment. **Principle**: P-IV clause 1 (real headers > meta). **Deferred ID**: follow-up only; not a v1 blocker. **Acceptance**: [contracts/csp.md](./contracts/csp.md) "amendment procedure". **Test evidence**: `audits/csp-http-header-<date>.md` with the decision recorded. **Size**: M. **Depends on**: T003 (grep gate must exist first so the meta-CSP is protected while the evaluation happens). **[BLOCKED-DECISION]**: the go/no-go on host migration is out of scope for v1.

**Checkpoint B4**: Four security-audit artifacts on file. Principle IV is now demonstrable, not just asserted.

---

## Phase B5 — Content, Brand & Asset Consistency

**Purpose**: Resolve the constitution-follow-up items (`CONTACT_DETAILS_CENTRALIZATION`, `PRODUCT_IMAGES`) plus the testimonial permission gate. `BRAND_DOMAIN_RECONCILIATION` was resolved in constitution v2.1.0 (2026-07-18) and its task T016 is now DONE.

- [x] T016 [US5] `BRAND_DOMAIN_RECONCILIATION` — **DONE (2026-07-18)**: canonical brand set to "KrishiDakshina"; localStorage namespace migrated to `krishidakshina.*`; all site copy, meta, footer, WhatsApp handoff, and testimonials updated in [index.html](../../index.html) and [index.js](../../index.js). Ratified in constitution v2.1.0 Sync Impact Report. **Principle**: cross-cutting; Governance section of the constitution. **Deferred ID**: `BRAND_DOMAIN_RECONCILIATION`, spec `OQ-01` — RESOLVED. **Acceptance met**: single canonical brand across all six touchpoints (title, `.brand`, footer, hero copy, WhatsApp header, `og:title`); post-change grep for "Gut Point" in `index.html` / `index.js` returns zero hits. **Test evidence**: post-change diff + grep output in the PR that landed the rebrand. **Size**: M.
- [ ] T017 [US5] `CONTACT_DETAILS_CENTRALIZATION` — **DEFERRED** pending owner-supplied production contact details. Once available, extract WhatsApp number, phone, email, and street address into a single **`<script type="application/json" id="site-config">…</script>`** block inside `index.html` (JSON in a non-executable script tag is CSP-safe and needs **no** `unsafe-inline`). Update [index.js:92](../../index.js) (`WHATSAPP_NUMBER`) to read `JSON.parse(document.getElementById('site-config').textContent)`. Update the four duplicated locations in `index.html`: [line 601](../../index.html) address, [line 608](../../index.html) tel, [line 615](../../index.html) email, [line 622](../../index.html) WhatsApp CTA URL. **Note on email**: the current address `hello@gutpoint.com` intentionally retains the old brand string as a placeholder per constitution v2.1.0 Governance; it is preserved verbatim and will be updated when the owner provides the production email. **Principle**: P-V clause 4 (constitutional follow-up); Governance. **Deferred ID**: `CONTACT_DETAILS_CENTRALIZATION`, spec `OQ-02`. **Acceptance**: post-change grep for `919876543210` returns exactly ONE hit (the JSON block); same for `hello@gutpoint.com` (or its owner-supplied replacement) and `+91 98765 43210`. **Test evidence**: post-change diff + grep output pasted into the PR description. **Size**: M. **Depends on**: owner-supplied contact values. **[BLOCKED-DECISION]**: owner input required (see [spec.md `OQ-02`](./spec.md)).
- [ ] T018 [P] [US1] `PRODUCT_IMAGES` — source 12 photos at 600 × 600 optimized to ≤ 200 KB each, saved as `product_images/{tomato,eggs,spinach,bread,mango,milk,carrot,yogurt,brown-rice,avocado,mixed-nuts,turmeric}.jpg` (or serve AVIF/WebP with a JPG fallback via `<picture>` — see stretch task T023). File names MUST match the 12 `<img src="product_images/…">` references at [index.html:257-452](../../index.html). Verify the emoji fallback still triggers if any image is later removed (delete one file, hit-refresh the page, confirm `data-fallback` shows). **Principle**: P-II. **Deferred ID**: `PRODUCT_IMAGES`, spec `OQ-03`. **Acceptance**: [spec.md `FR-020`, `FR-024`, `FR-026`](./spec.md); [spec.md `SC-002`](./spec.md) (≤ 200 KB each). **Test evidence**: `ls -lS product_images/` output pasted into the PR + one Lighthouse "Properly size images" audit at score 100. **Size**: L (asset sourcing). **Depends on**: none — touches a new folder, no source-file conflict. **[BLOCKED-DECISION]**: asset provider (see [spec.md `OQ-03`](./spec.md)).
- [x] T019 [US5] **DONE (2026-07-18) — testimonials confirmed placeholders**. The three testimonials in [index.html:494-521](../../index.html) (Priya Sharma, Rahul Mehta, Ananya Iyer) are confirmed placeholders per constitution v2.1.0 (OQ-04 RESOLVED). They are already labeled as example content in a source-file comment in `index.html`. **Governance follow-up**: production launch is BLOCKED from publishing these testimonials until real ones are gathered per the new constitution v2.1.0 "Testimonial permission requirement" governance rule — either signed, written permission from each named customer on file, OR replacement with genuinely permitted testimonials. **Principle**: Governance / privacy hygiene (constitution v2.1.0 new governance rule). **Deferred ID**: spec `OQ-04` — RESOLVED as placeholders. **Acceptance met**: source-file comment labels them as example content; a follow-up task (**T019a below**) tracks the production-launch permission gate. **Test evidence**: source comment in `index.html` + constitution v2.1.0 Sync Impact Report entry.
- [ ] T019a [US5] **Production-launch gate** — before publishing the site with the placeholder testimonials replaced by production copy, either (a) obtain and file documented, written permission from each named customer under `docs/testimonial-permissions.md`, OR (b) replace them with genuinely permitted testimonials. Until one of those is done, the placeholder set (Priya Sharma, Rahul Mehta, Ananya Iyer) MUST retain its "example content" source comment and MUST NOT be presented as real testimonials in production. **Principle**: constitution v2.1.0 "Testimonial permission requirement". **Deferred ID**: spec `OQ-04` follow-up. **Acceptance**: either `docs/testimonial-permissions.md` exists with signed permissions, OR the visible testimonials in `index.html` have been swapped for permitted ones. **Test evidence**: the permissions file OR the diff replacing the copy. **Size**: S. **Depends on**: none. **[BLOCKED-DECISION]**: needs real customer testimonials or a decision to defer the section entirely.

**Checkpoint B5**: Brand is single-sourced (T016 DONE in constitution v2.1.0); contact details are single-sourced once owner supplies production values (T017); product images either shipped or the emoji fallback is explicitly ratified; testimonials are labeled as example content (T019 DONE) and the production-launch gate (T019a) tracks the permission requirement.

---

## Phase B6 — Contact-Form Decision (P-V clause 6)

**Purpose**: Resolve the single largest v1 open question that's not "polish": what happens when a visitor submits the visible contact form? Historically a client-side simulation (former A043); on 2026-07-18 migrated to a WhatsApp handoff consistent with the order flow (P-V clause 6(a)).

- [x] **DONE 2026-07-18** T020 [US5] Fate of the contact form ([spec.md `OQ-05`](./spec.md)) — **Option C selected: WhatsApp handoff** (analogous to the order flow at A034/A035). Rationale: (1) reuses the already-declared `wa.me` origin — no new origin, no CSP change, no constitution amendment required, since Principle V clause 6(a) explicitly permits "a WhatsApp handoff consistent with #4"; (2) WhatsApp's native phone-number verification acts as the anti-spam mechanism the governance rule asks for; (3) consistent UX with the primary order CTA. **Implementation**: contact form handler rewritten to build a `💬 *New Enquiry – KrishiDakshina*` message and open `https://wa.me/${WHATSAPP_NUMBER}?text=...` in a new tab — [index.js:548-630](../../index.js). Inline `#cf-err` error region replaces the previous `alert()` call. Button icon updated to `fa-brands fa-whatsapp` and label to "Send via WhatsApp" — [index.html:648-654](../../index.html). Success toast reworded from "Message sent!" to "WhatsApp opened with your message — press Send there to reach us." **Principle**: P-V clause 6(a). **Deferred ID**: spec `OQ-05`, `CONTACT_FORM_FATE` — both RESOLVED.
- [ ] T020a [US5] **[POST-v1]** Evaluate migration to a real-inbox third-party form endpoint (Web3Forms / Formspree / Netlify Forms / Basin / Formsubmit). **Trigger**: owner requests an inbox trail for enquiries independent of WhatsApp. **Constitutional cost** (per constitution v2.1.0 "Contact-form migration procedure"): ALL of (a) a **constitution MINOR amendment** adding the new endpoint origin to the runtime allow-list, (b) a **CSP allow-list amendment** in [index.html](../../index.html) — add the endpoint origin to `form-action` and, if the endpoint's response is fetched via JS, to `connect-src`; add an SRI hash for any third-party script the provider requires, (c) an **anti-spam mechanism** whose provider (if any) is itself listed in the constitution — minimum viable is a honeypot input (no origin cost); a CAPTCHA provider is a further origin addition. **Recommended provider (pre-audit shortlist)**: **Web3Forms** — cleanest CSP footprint (`https://api.web3forms.com`), free tier 250 msg/mo, honeypot supported natively, no signup-blocked domain. **Fallbacks**: Formspree (well-established but 50 msg/mo free-tier ceiling), Formsubmit (no-signup but adds a captcha-redirect UX). **Runtime shape**: keep the current form markup; on submit, `fetch(endpoint, { method: 'POST', body: FormData })`; on non-OK, fall back to the existing WhatsApp handoff (best-of-both). **Acceptance**: constitution diff at v2.2.0 (or later); CSP diff; honeypot present; DevTools Network capture of a real submit reaching the endpoint; success email received at the owner's inbox. **Test evidence**: preview URL + Network log + email screenshot. **Size**: M. **Depends on**: T017 (contact details finalized so the endpoint is configured against the real owner address), owner request to trigger. **[BLOCKED-DECISION]**: needs owner request; not required for v1.

**Checkpoint B6**: The contact-form behavior is ratified as a WhatsApp handoff (T020 done). A future migration to a third-party inbox endpoint is scoped as T020a with the full constitution + CSP paper trail already itemized.

---

## Phase B7 — Documentation

**Purpose**: The constitution's Governance section says a `README.md` and `docs/quickstart.md` MUST be authored and MUST cite the constitution. Neither exists at the repo root today.

- [ ] T021 [P] Author `README.md` at the repository root. Contents: one-liner about the KrishiDakshina site, "how to serve locally" (`python -m http.server 8000` or `npx serve .`), deployment note (`git push` → GitHub Pages auto-publishes from the default branch via [CNAME](../../CNAME)), and a "Governance" section linking to [.specify/memory/constitution.md](../../.specify/memory/constitution.md) and [specs/001-storefront-baseline/spec.md](./spec.md). **Principle**: Governance ("Runtime guidance"). **Deferred ID**: constitution Sync Impact Report — "README.md / docs/quickstart.md — not present in repository; propagate when authored." **Acceptance**: file exists at `README.md`; passes T001's link-check on the new internal links. **Test evidence**: the file itself + green CI. **Size**: S. **Depends on**: none (canonical brand already resolved in constitution v2.1.0).
- [ ] T022 [P] Confirm [specs/001-storefront-baseline/quickstart.md](./quickstart.md) already covers what a `docs/quickstart.md` would cover. If so, add a one-line pointer in `README.md` (task T021) and record "no separate `docs/quickstart.md` needed for v1 — see feature quickstart" in the T021 PR description. If NOT, author `docs/quickstart.md` citing the constitution per its Governance advisory. **Principle**: Governance. **Deferred ID**: constitution Sync Impact Report same as T021. **Acceptance**: either the pointer added or `docs/quickstart.md` exists and cites the constitution. **Test evidence**: T021's PR + the reviewer's confirmation. **Size**: S. **Depends on**: T021.

**Checkpoint B7**: The repo has entry-point docs; no more implicit tribal knowledge.

---

## Phase B8 — Optional / Stretch (NOT v1 blockers)

Marked explicitly as stretch. Do not gate v1 release on these — they are queued for `/speckit.plan` on a follow-up feature branch.

- [ ] T023 [P] [US1] **Stretch** — migrate the 12 `<img>` tags to `<picture>` with AVIF + WebP + JPG fallback. **Principle**: P-II. **Deferred ID**: [spec.md `FR-020`](./spec.md) ("AVIF/WebP preferred with JPEG/PNG fallback"). **Acceptance**: post-change Lighthouse "Serve images in modern formats" audit ≥ 90. **Test evidence**: appended reading in `audits/perf-<date>.md`. **Size**: M. **Depends on**: T018 (need the JPG masters first).
- [ ] T024 [P] [US1] **Stretch** — extract product data (name, category, description, price, unit, badge, filename, fallback emoji) into a separate `<script type="application/json" id="products">` block **or** a `products.json` file, then have `index.js` render the grid from the data. Reduces the risk of a typo in one of 12 near-identical `<article>` blocks. **Principle**: P-III (minimalism, DRY) + P-IV (still XSS-safe via `textContent` per A012 pattern). **Deferred ID**: none (author's judgment call). **Acceptance**: grid renders identically to today; no new CSP entries needed; the JSON block is `application/json` (non-executable). **Test evidence**: side-by-side screenshot + a follow-up Lighthouse run. **Size**: L. **Depends on**: T017 (same "put data in a script tag" pattern; land the site-config block first).
- [ ] T025 [P] [US4] **Stretch** — a "Save order" print / PDF fallback path for visitors who can't reach WhatsApp (out-of-country visitors, WhatsApp regional bans, corporate blocklists). Trigger a print stylesheet + `window.print()` OR a plain-text `download` attribute on an `<a href="data:text/plain,...">`. **Principle**: P-I + resilience ([spec.md `SC-010`](./spec.md) style). **Deferred ID**: none. **Acceptance**: keyboard-reachable button in the drawer footer produces a printable order summary containing the same fields as the wa.me message. **Test evidence**: manual verification screenshot. **Size**: M. **Depends on**: T017 (WhatsApp number + brand freeze).

---

## Dependencies & Execution Order (Section B)

```mermaid
flowchart TD
  T001[T001 lighthouse.yml] --> T002[T002 axe-core CI]
  T003[T003 CSP grep]
  T004[T004 Math.random grep]
  T005[T005 bundle size]
  T006[T006 axe local]
  T006 --> T007[T007 keyboard walk]
  T008[T008 zoom test]
  T008a[T008a mobile-first — DONE]
  T006 --> T008a
  T008a --> T008b[T008b axe/LH re-run]
  T009 --> T008b
  T009[T009 Lighthouse local] --> T010[T010 loading=lazy]
  T010 --> T011[T011 width/height]
  T012[T012 XSS grep]
  T013[T013 rel=noopener grep]
  T014[T014 SRI verify]
  T003 --> T015[T015 CSP header eval]
  T016[T016 BRAND — DONE]
  T017[T017 CONTACT centralize]
  T018[T018 PRODUCT_IMAGES]
  T019[T019 testimonials — DONE]
  T019a[T019a launch permission gate]
  T020[T020 contact-form — DONE]
  T020a[T020a Web3Forms migration — POST-v1]
  T017 --> T020a
  T021[T021 README]
  T021 --> T022[T022 docs/quickstart]
  T018 --> T023[T023 <picture>]
  T017 --> T024[T024 products.json]
  T017 --> T025[T025 print PDF]
```

### Phase Order

1. **B1 (CI Gates)** — first, because everything downstream benefits from the gates being live.
2. **B2 (a11y)** + **B3 (perf)** + **B4 (security)** — parallel work streams, each producing audit artifacts. Best run in the order shown so a11y fixes land before the manual keyboard pass and before the "after" Lighthouse.
3. **B5 (content/brand)** — T016 is DONE (constitution v2.1.0 landed the brand rename); T017 depends on owner-supplied contact values; T018 / T019 / T019a are parallel.
4. **B6 (contact form)** — T020 is DONE (2026-07-18, WhatsApp handoff); T020a (Web3Forms migration) is post-v1 and gated on an owner request + T017.
5. **B7 (docs)** — can now proceed (brand freeze already landed).
6. **B8 (stretch)** — after v1 ships.

### Parallel Opportunities

- **T001 + T003 + T004 + T005** are all workflow-file edits — trivially parallel; a single PR is fine but they're independent.
- **T006 + T008 + T009** are read-only audits against the shipped code — full parallel.
- **T012 + T013 + T014** are pure grep + review — full parallel.
- **T018 + T019a** touch new files or a small isolated section — parallel with the T017 chain.
- **T023 + T024 + T025** are stretch items with different owners possible — parallel among themselves once T017/T018 land.

### Serial Constraints (do NOT parallelize)

- **T010 and T011** both edit `index.html` — sequential.
- **T016 and T017** — T016 is DONE; T017 remains dependent on owner-supplied contact values. No sequencing concern between them anymore.
- **T009 → T010 → T011** is a "measure, fix, re-measure" chain — sequential.

---

## Implementation Strategy

### MVP-first is not applicable

This is a brownfield baseline: the entire "MVP" (US-1 through US-6) is already shipped in Section A. The MVP-first-slice equivalent for Section B is **Phase B1 alone**: land the five CI gates, then ship v1 with the CI enforcing the constitution. Phases B2–B7 fill in the audit trail; Phase B8 waits for v1.1.

### Suggested v1 execution slice

1. **v1.0 release blockers**: T001 – T005 (CI gates) + T006 – T009 (baseline audits) + T017 (contact freeze, pending owner input). That is 10 tasks (T016 and T020 are already DONE; T019 is DONE). Smallest set that closes P-VI FAIL, produces the audit artifacts P-I and P-II expect, and eliminates the remaining content-drift risk the constitution's Sync Impact Report still flags.
2. **v1.0 nice-to-have**: T010 – T014 (perf polish + security-audit paperwork) + T018 (product images) + T019a (testimonial launch gate) + T021 – T022 (docs). Nine more tasks, none of them structural.
3. **v1.0 decisions to unblock**: T015 (host migration eval — decision recorded, not executed).
4. **Post-v1 backlog**: T020a (Web3Forms/third-party form endpoint migration — owner-triggered) + T023 – T025 (stretch).

### Incremental Delivery

Each Section-B task is independently mergeable. Landing T001 alone already meaningfully improves the release posture. Landing all of Phase B1 constitutes the single biggest constitution-compliance win available.

---

## Section B totals

- **Total tasks tracked**: 29 (T001 – T025 + T019a + T020a + T008a + T008b).
- **Completed since baseline**: 4 (T016, T019, T020, T008a — all closed on 2026-07-18).
- **Outstanding tasks (unchecked)**: 25 (T001 – T008, T008b, T009 – T015, T017, T018, T019a, T020a, T021 – T025).
- **v1 release blockers (Phases B1–B5, unchecked)**: 19 (T001 – T008, T008b, T009 – T015, T017, T018, T019a).
- **v1 decisions with recorded outcomes (Phase B4)**: 1 (T015).
- **Documentation (Phase B7)**: 2 (T021, T022).
- **Post-v1 / stretch (Phase B6 T020a + Phase B8)**: 4 (T020a, T023 – T025).
- **[BLOCKED-DECISION] tasks (need owner input)**: 5 (T017, T018, T019a, T020a, and the go/no-go inside T015).
- **Parallelizable `[P]` tasks**: 15 (T001, T002, T003, T004, T005, T006, T008, T009, T012, T013, T014, T018, T021, T022, T023, T024, T025 — full count of the `[P]` markers above).

---

## Notes

- `[P]` = different files, no dependencies. **NEVER** parallelize edits to `index.html` or `index.js`.
- `[Story]` labels use the spec's US identifiers (US1 – US6); tasks that are pure CI or documentation carry no `[US*]` label per the template.
- Every Section-B task cites (a) the principle it satisfies, (b) the acceptance criterion or file to touch, and (c) any deferred item ID from the plan / constitution Sync Impact Report.
- Section A is **frozen evidence**, not work-in-flight. Removing anything from Section A without also removing the corresponding baseline behavior from the source files is a spec/plan drift and requires an amendment PR.
- Cross-links to design docs: [plan.md `#complexity-tracking`](./plan.md) enumerates the same five deferred items this file operationalises; [contracts/csp.md](./contracts/csp.md) governs T003 + T015; [contracts/whatsapp-handoff.md](./contracts/whatsapp-handoff.md) governs T025.
