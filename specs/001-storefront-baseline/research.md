# Phase 0 Research: Storefront Baseline (KrishiDakshina v1)

**Feature**: `001-storefront-baseline` · **Plan**: [plan.md](./plan.md) · **Spec**: [spec.md](./spec.md) · **Constitution**: [.specify/memory/constitution.md](../../.specify/memory/constitution.md) v2.1.0

**Nature of this research**: reverse-engineered from the shipped code. Every decision below is **already in effect** in [index.html](../../index.html), [index.css](../../index.css), and [index.js](../../index.js) at the time this plan was authored. No POCs, spikes, or comparative benchmarks were run for this baseline — that work is **Not applicable — reverse-engineered from shipped code**. The reference implementation is the three source files above.

Each entry uses the same shape:

> **Decision** — what was chosen.
> **Rationale** — why that choice fits the constitution and the business context.
> **Alternatives considered** — what else was on the table and why it was rejected.

---

## R-1. Language, runtime, and build toolchain

- **Decision**: Vanilla HTML5 + CSS3 + JavaScript ES2022+. One [index.html](../../index.html), one [index.css](../../index.css), one [index.js](../../index.js). `'use strict'` at the top of `index.js`. No transpiler, no minifier, no bundler, no `package.json`. Files are served by the host as-is.
- **Rationale**: The constitution's Principle III makes zero-dependency vanilla the default; every framework/build tool has to earn its place via an amendment that documents the concrete problem it solves. The site has no such problem — 12 products, one form, one WhatsApp handoff — so the default wins.
- **Alternatives considered**:
  - React / Vue / Svelte + Vite — rejected: adds runtime JS, bundler, and node dependency tree; would blow the 50 KB gz JS budget (Principle II) and require an amendment (Principle III).
  - Astro / 11ty static generator — rejected: still adds a build step and a node toolchain; the maintainer can hand-edit a single `index.html` faster than they can run a build.
  - TypeScript with `tsc` — rejected: introduces a build step even without a bundler; the existing JS is small enough to hold in one head.

## R-2. Runtime host and deployment pipeline

- **Decision**: GitHub Pages with a custom domain declared in the tracked [CNAME](../../CNAME) file (`krishidakshina.in`). Deploy today is `git push` → GitHub Pages auto-publish from the default branch. No GitHub Actions workflow is wired yet.
- **Rationale**: Principle III + Principle V make server-side runtime a MAJOR amendment; GitHub Pages is free, low-latency-enough for the target audience, and requires zero maintenance. The `CNAME` file is the whole deploy config.
- **Alternatives considered**:
  - Netlify / Vercel — rejected: adds a vendor and features (functions, edge middleware) the constitution forbids; not worth the switching cost.
  - S3 + CloudFront — rejected: same static outcome for higher operational overhead.
  - Adding a GitHub Actions workflow *now* — deferred: the workflow is needed for Principle VI gates, but the baseline plan explicitly records the CI gap and hands it to `/speckit.tasks`.

## R-3. External runtime origins (CDN, fonts, APIs)

- **Decision**: The runtime allow-list, matched to the CSP at [index.html:24](../../index.html#L24), is exactly:
  - `cdnjs.cloudflare.com` — Font Awesome 6.5.0 CSS, SRI-pinned at [index.html:29-32](../../index.html#L29) with `integrity="sha384-/o6I2CkkWC//PSjvWC/eYN7l3xM3tJm8ZzVkCOfp//W05QcE3mlGskpoHB6XqI+B"`, `crossorigin="anonymous"`, `referrerpolicy="no-referrer"`.
  - `fonts.googleapis.com` + `fonts.gstatic.com` — Inter 300–900 with `preconnect`.
  - `api.postalpincode.in` — `GET /pincode/{pincode}` for city/state auto-fill; called only when pincode passes `isValidPincode` and City is empty; cancelable via `AbortController`.
  - `wa.me` — outbound `window.open` for the order handoff (navigation only, not a `connect-src` origin).
  No other origins. **No CDN JavaScript, no analytics, no A/B testing SDK, no CMS, no serverless functions, no service worker.**
- **Rationale**: Every entry maps to a concrete user-visible feature (icons, typography, city auto-fill, order placement). Every executable resource is SRI-pinned per Principle IV clause 3. Every new origin requires (a) a CSP amendment in `index.html`, (b) a constitution amendment on the runtime allow-list, and (c) an SRI hash where the origin serves executable content.
- **Alternatives considered**:
  - Self-hosting Font Awesome — deferred: the SRI-pinned CDN is compliant with Principle IV and saves the maintenance of icon-font versioning; a future task can revisit if bundle-budget pressure grows.
  - Self-hosting Inter — deferred for the same reason (Google Fonts + `font-display: swap` is compliant with Principle II clause on fonts).
  - Adding Google Analytics / Plausible / Fathom — rejected outright: Principle V clause 2 bans analytics without a MAJOR amendment.

## R-4. Client-side architecture

- **Decision**: Single script file [index.js](../../index.js), `'use strict'`, all handlers registered via `addEventListener` (zero inline `onclick=`, zero `<script>` bodies). Imperative DOM APIs — `createElement`, `textContent`, `appendChild`, `setAttribute` — for every insertion. `innerHTML` / `outerHTML` / `document.write` / `insertAdjacentHTML` are **not used for user data**. One `IntersectionObserver` for `.fi` fade-in animations at [index.js:81](../../index.js#L81) (threshold `0.10`, staggered ~90 ms). One capture-phase `error` listener on `document` for `<img>` fallback via `data-fallback` attributes.
- **Rationale**: Constitution Principle IV clauses 2 and 4 make this mandatory. Event delegation on `document` is used sparingly to keep the listener count low and to survive DOM re-renders in the cart drawer.
- **Alternatives considered**:
  - Framework virtual DOM — rejected: same as R-1; adds bytes and complexity for no user-visible gain.
  - Web Components — rejected: overkill for 12 hard-coded products; would still need imperative DOM inside the component.
  - Template literals with `innerHTML` — rejected outright: XSS-risk, violates Principle IV clause 4 for user-controlled data.

## R-5. State management (cart, customer, geolocation)

- **Decision**:
  - Cart: in-memory object `cart`, persisted synchronously to `localStorage` under `krishidakshina.cart.v1`. Schema: `{ [key: string]: { name, price, unit, qty } }` — see [contracts/localstorage-schema.md](./contracts/localstorage-schema.md).
  - Customer: persisted to `localStorage` under `krishidakshina.customer.v1`. Fields: `name`, `phone`, `addr1`, `addr2`, `city`, `pincode`, `notes`.
  - Geolocation: held in-memory only (`geoLoc = { lat, lng }`) — **never persisted**. Fires only on explicit "Use my current location" button click with `{ enableHighAccuracy: true, timeout: 8000, maximumAge: 60000 }`.
  - Hydration: `Object.keys` iteration + per-field whitelist + `isValidCartItem` / `isValidCustomer` schema validators construct a fresh plain object; on any validation failure the entire entry is dropped and the app starts clean.
- **Rationale**: Principle V clause 3 requires versioned `localStorage` keys and Principle IV clause 5 requires prototype-pollution defense. Keeping geolocation in memory only limits the blast radius if `localStorage` is inspected on a shared device.
- **Alternatives considered**:
  - `sessionStorage` for the cart — rejected: users expect cart persistence across a tab close; `localStorage` is the right primitive.
  - `IndexedDB` — rejected: overkill for two small objects.
  - Persisting geolocation — rejected: privacy-sensitive; user consent is per-session, so persisting past the tab is a policy overreach.

## R-6. Security posture (CSP, XSS, SRI, randomness, external links)

- **Decision**: The security posture is the constitution's Principle IV verbatim:
  - Meta CSP at [index.html:24](../../index.html#L24) with `default-src 'self'`, `script-src 'self'`, `frame-ancestors 'none'`, `object-src 'none'`, `form-action 'self'`, `base-uri 'self'`, and the allow-list from R-3. No `'unsafe-inline'`, no `'unsafe-eval'`.
  - `Referrer-Policy: strict-origin-when-cross-origin` (meta).
  - `rel="noopener noreferrer"` on every external anchor and every `window.open`.
  - Input validators from [index.js](../../index.js): `isValidCartItem`, `isValidCustomer`, `isValidPhone` (`/^[6-9]\d{9}$/`), `isValidPincode` (`/^[1-9]\d{5}$/`), `isNonEmpty`, `isStrLen`.
  - Length caps (constants): `MAX_QTY_PER_ITEM=99`, `MAX_CART_ITEMS=50`, `MAX_NAME_LEN=200`, `MAX_UNIT_LEN=50`, `MAX_PRICE=1_000_000`, `MAX_CNAME_LEN=100`, `MAX_ADDR_LEN=200`, `MAX_CITY_LEN=100`, `MAX_NOTES_LEN=500`, `MAX_PHONE_LEN=20`, `MAX_MSG_LEN=3800`.
  - Randomness: `crypto.getRandomValues()` wrapped in `randFloat()` at [index.js:37](../../index.js#L37). `Math.random(` count in production JS: **zero**.
- **Rationale**: Constitution Principle IV is NON-NEGOTIABLE; each clause maps to a concrete risk the site actually faces (address entry, order handoff via WhatsApp, shared devices, tampered `localStorage`).
- **Alternatives considered**: None viable — every relaxation ties directly to a Principle IV clause that would require an amendment.

## R-7. Testing strategy

- **Decision**: Manual smoke checklist only, documented in [quickstart.md](./quickstart.md#validation-scenarios). Steps 1–8 cover navbar, product cards, cart drawer, form validation, pincode API, geolocation, WhatsApp handoff, contact form, and `prefers-reduced-motion`. Automated testing (Lighthouse CI, axe-core, CSP-hygiene grep, `Math.random(` grep) is **planned but not implemented** — this is the P-VI gap tracked in the plan's Complexity Tracking table.
- **Rationale**: For a 12-product single-page site with no logic branches beyond validators and DOM assembly, a documented manual pass is proportional. The automated gates from Principle VI are still required and are the first item on the `/speckit.tasks` queue.
- **Alternatives considered**:
  - Jest / Vitest unit tests — deferred: no code module boundaries yet; test doubles for `localStorage` and `fetch` would cost more than they'd save at 12 products. Revisit if the codebase grows.
  - Playwright end-to-end — deferred: valuable but requires a CI runner; adopting it can happen alongside the Lighthouse workflow.

## R-8. Observability & error handling

- **Decision**: Silent-fail with `console.warn` for non-critical `localStorage` / `fetch` errors. All warnings are intentional and documented as inline comments in [index.js](../../index.js). **No remote error reporting.**
- **Rationale**: Principle V clause 2 bans analytics and by extension any remote error/reporting SDK (Sentry, Datadog RUM, etc.) without a MAJOR amendment. `console.warn` gives the maintainer visibility during local debugging without leaking data.
- **Alternatives considered**:
  - Sentry / Bugsnag — rejected under Principle V.
  - `window.onerror` → self-hosted endpoint — rejected: would require a backend (Principle V) or a new `connect-src` origin (allow-list amendment).

## R-9. Product images

- **Decision**: The `product_images/` folder does not yet exist. The 12 `<img>` elements in [index.html](../../index.html) reference filenames `tomato`, `eggs`, `spinach`, `bread`, `mango`, `milk`, `carrot`, `yogurt`, `brown-rice`, `avocado`, `mixed-nuts`, `turmeric`. Each `<img>` carries a `data-fallback` emoji, and a delegated `error` listener on `document` swaps in the emoji when the image request fails.
- **Rationale**: The delegated-error pattern lets the site ship (and pass Principle IV, since no inline `onerror=`) even before real assets exist. When assets are ready, they MUST match those filenames and MUST satisfy Principle II (≤ 200 KB at 600 × 600, AVIF/WebP with fallback via `<picture>` if introduced, `loading="lazy"` below the fold).
- **Alternatives considered**:
  - Ship emoji only, permanently — deferred to `/speckit.tasks`; the constitution's Sync Impact Report open item `PRODUCT_IMAGES` requires an explicit decision before v1 launch.

## R-10. WhatsApp handoff

- **Decision**: `WHATSAPP_NUMBER = "919876543210"` at [index.js:92](../../index.js#L92). The order button composes a plain-text message from validated cart + customer state and opens `https://wa.me/${WHATSAPP_NUMBER}?text=${encoded}` in a new tab. Length is checked with `if (encoded.length > MAX_MSG_LEN)` at [index.js:535](../../index.js#L535); oversize blocks the open and surfaces an inline error.
- **Rationale**: Principle V clause 4 makes this the sole order-placement path in v1. The hard cap of 3800 chars keeps the URL well under `wa.me`'s ~4 KB soft limit. See [contracts/whatsapp-handoff.md](./contracts/whatsapp-handoff.md) for the full message template.
- **Alternatives considered**:
  - Direct form POST to a backend — rejected under Principle V (would require MAJOR amendment).
  - Email `mailto:` — rejected: unreliable on mobile, no attachment/formatting parity with WhatsApp.

---

**Consolidated status**: All decisions are already implemented and stable. There are **no `NEEDS CLARIFICATION` markers** left in the Technical Context after Phase 0. Open questions surfaced by `spec.md` (OQ-01 brand vs. domain, OQ-02 contact placeholders, OQ-03 product image sourcing, OQ-04 testimonial permissions, OQ-05 contact-form backend) are business/policy decisions, not tech-context decisions — they are handled by `/speckit.clarify` and by the constitution's Sync Impact Report follow-ups, not by this plan.
