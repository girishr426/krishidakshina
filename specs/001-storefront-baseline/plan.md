# Implementation Plan: Storefront Baseline (KrishiDakshina v1)

**Branch**: `001-storefront-baseline` | **Date**: 2026-07-18 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from [specs/001-storefront-baseline/spec.md](./spec.md)

**Note**: This plan is a **brownfield / reverse-engineered baseline**. It documents the technical choices already shipping in [index.html](../../index.html), [index.css](../../index.css), and [index.js](../../index.js) so that all future changes have an authoritative reference point. It is descriptive, not prescriptive — no new architecture is proposed here.

## Summary

KrishiDakshina v1 is a single-page, zero-dependency static storefront deployed to GitHub Pages under the tracked domain `krishidakshina.in` (via the [CNAME](../../CNAME) file). A visitor browses a 12-item hard-coded product grid, builds a cart persisted in versioned `localStorage`, fills a delivery form (with optional pincode lookup and opt-in geolocation), and places the order by opening a pre-composed `wa.me` message in a new tab. The technical approach is deliberately minimal: vanilla HTML5 + CSS3 + ES2022+ JS, three sibling files at the repo root, no `package.json`, no build step, and a strict `<meta>` CSP that pins Font Awesome via SRI and allow-lists only `api.postalpincode.in` for `connect-src`. See [research.md](./research.md) for the decisions consolidated per-topic.

## Technical Context

**Language/Version**: Vanilla HTML5, CSS3, and JavaScript ES2022+. Single [index.html](../../index.html), single [index.css](../../index.css), single [index.js](../../index.js). `'use strict'` is declared at the top of `index.js`; no transpiler, no minifier, no bundler.

**Primary Dependencies**:

- Runtime, browser-loaded (allow-listed in the CSP in [index.html](../../index.html#L24)):
  - `cdnjs.cloudflare.com` — Font Awesome 6.5.0 CSS, loaded via `<link rel="stylesheet">` with SRI `integrity="sha384-/o6I2CkkWC//PSjvWC/eYN7l3xM3tJm8ZzVkCOfp//W05QcE3mlGskpoHB6XqI+B"`, `crossorigin="anonymous"`, and `referrerpolicy="no-referrer"` at [index.html:29](../../index.html#L29).
  - `fonts.googleapis.com` + `fonts.gstatic.com` — Inter font weights 300–900 with `preconnect` hints.
  - `api.postalpincode.in` — `GET /pincode/{pincode}` for pincode → city/state; called only when the pincode passes `isValidPincode` **and** the City field is empty; cancelable via `AbortController`.
  - `wa.me` — navigation target for `window.open(...)` on the WhatsApp handoff; not a `connect-src` origin.
- Build-time dependencies: **none**. There is no `package.json`, no bundler, no transpiler, no CSS preprocessor. Adding any of these is a constitution-level (Principle III) decision.

**Storage**:

- `localStorage` keys, versioned per constitution Principle V:
  - `krishidakshina.cart.v1` — `{ [key: string]: { name: string, price: number, unit: string, qty: integer 1..99 } }`, capped at `MAX_CART_ITEMS = 50` line items.
  - `krishidakshina.customer.v1` — `{ name, phone, addr1, addr2, city, pincode, notes }` with per-field length caps (see [contracts/localstorage-schema.md](./contracts/localstorage-schema.md)).
- In-memory only, **never persisted**: `geoLoc = { lat, lng }` (captured on explicit "Use my current location" click).
- No cookies. No `sessionStorage`. No IndexedDB. No service worker.

**Testing**:

- Manual smoke checklist (documented; no framework). Steps 1–8 in [quickstart.md](./quickstart.md#validation-scenarios).
- Automated (planned, not yet implemented): Lighthouse CI gating Performance / Accessibility / Best Practices / SEO ≥ 90 on every PR (see the deferred gap tasks under Constitution Check).

**Target Platform**: Browser only. Latest two stable versions of Chrome, Firefox, Safari, and Edge, plus Chrome on Android and Safari on iOS. GitHub Pages serves the files as-is; there is no Node.js at runtime, no server-side rendering, and no dev-time toolchain requirement beyond a static file server (e.g., `python -m http.server`, `npx serve`, or VS Code Live Server).

**Project Type**: Static single-page marketing site with client-only cart + order-handoff. This is a **client-only web application** in constitution Principle III + V terms.

**Performance Goals** (constitution Principle II, measured on Lighthouse mobile with 4G Fast throttling and 4× CPU slowdown):

- Largest Contentful Paint (LCP) < 2.0 s
- Cumulative Layout Shift (CLS) < 0.1
- Interaction to Next Paint (INP) < 200 ms
- Total shipped JS ≤ 50 KB gzipped per page
- Total shipped CSS ≤ 30 KB gzipped per page
- Product images ≤ 200 KB each at 600 × 600 (AVIF/WebP with JPEG/PNG fallback)

**Constraints**:

- **CSP** (verbatim from [index.html:24](../../index.html#L24)): `default-src 'self'; style-src 'self' https://cdnjs.cloudflare.com https://fonts.googleapis.com; font-src 'self' https://cdnjs.cloudflare.com https://fonts.gstatic.com; img-src 'self' data: https:; script-src 'self'; connect-src 'self' https://api.postalpincode.in; frame-ancestors 'none'; base-uri 'self'; form-action 'self'; object-src 'none'`.
- **Referrer-Policy**: `strict-origin-when-cross-origin` (declared as a `<meta name="referrer">`).
- **No inline JS / CSS / event handlers.** All handlers registered via `addEventListener`; zero `onclick=`, zero `<script>` bodies, zero `style="…"` attributes.
- **XSS-safe DOM only.** Every user-derived string is inserted via `textContent`, `createElement`, or `setAttribute` — `innerHTML` / `outerHTML` / `document.write` / `insertAdjacentHTML` are banned for user data.
- **Prototype-pollution defense.** `localStorage` hydration iterates `Object.keys`, whitelists known field names, type-checks each field, caps string lengths, and constructs a fresh plain object (`{ … }`) — never `Object.assign` onto untrusted input.
- **Input validation** (constants defined in [index.js](../../index.js)):
  - Regex: `phone /^[6-9]\d{9}$/`, `pincode /^[1-9]\d{5}$/`.
  - Length caps: `MAX_QTY_PER_ITEM=99`, `MAX_CART_ITEMS=50`, `MAX_NAME_LEN=200`, `MAX_UNIT_LEN=50`, `MAX_PRICE=1_000_000`, `MAX_CNAME_LEN=100`, `MAX_ADDR_LEN=200`, `MAX_CITY_LEN=100`, `MAX_NOTES_LEN=500`, `MAX_PHONE_LEN=20`, `MAX_MSG_LEN=3800`.
- **Randomness**: `crypto.getRandomValues()` wrapped in `randFloat()` at [index.js:37](../../index.js#L37). `Math.random()` is banned across the codebase (constitution Principle IV, clause 7 + Principle VI, clause 5).
- **External anchors / `window.open`**: every one carries `rel="noopener noreferrer"`.
- **Geolocation options**: `{ enableHighAccuracy: true, timeout: 8000, maximumAge: 60000 }`, fired **only** on an explicit user click, never on load.
- **Observers**: one `IntersectionObserver` for `.fi` fade-in animations (threshold `0.10`, staggered ~90 ms); one capture-phase `error` listener on `document` for `<img>` fallback via `data-fallback`.

**Scale/Scope**:

- 12 products (hard-coded in [index.html](../../index.html)).
- Single page, single markup file, single stylesheet, single script.
- Cart cap: 50 distinct lines × 99 qty each; WhatsApp message hard-capped at 3800 encoded chars.
- Traffic: single-tenant static host on GitHub Pages; concurrency limits are the CDN's, not ours.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

Evaluated against [.specify/memory/constitution.md](../../.specify/memory/constitution.md) v2.1.0.

- [x] **P-I. Accessibility-First** — **GAP (documented)**. By construction the site uses semantic landmarks, native form controls, and honors `prefers-reduced-motion` (see [index.css](../../index.css) media query). It has **not yet** been through an automated axe-core / Pa11y scan or a full manual keyboard/focus/zoom audit. Recorded as a gap task; the shipped baseline is expected to pass, but the audit itself is deferred to `/speckit.tasks`.
- [x] **P-II. Performance Budget** — **GAP (measurement pending)**. Design decisions align with the budget (no framework, product images planned at 600 × 600 / ≤ 200 KB, self-hosted or allow-listed fonts with `font-display: swap`), but LCP / CLS / INP and gz JS+CSS budgets have not been measured yet because Lighthouse CI is not wired. Recorded as a gap task; measurable at the first Lighthouse run.
- [x] **P-III. Static-First & Framework-Minimal** — **PASS by construction**. Three files at repo root (`index.html`, `index.css`, `index.js`) plus `CNAME`. No `package.json`, no framework, no bundler, no transpiler. Deploys via GitHub Pages default-branch auto-publish. Any change to this posture is a MINOR (framework/build tool) or MAJOR (backend) amendment.
- [x] **P-IV. Security-by-Default** — **PASS by inspection, AUDIT GAP recorded**. The strict CSP is present at [index.html:24](../../index.html#L24); Font Awesome is SRI-pinned at [index.html:30](../../index.html#L30); all handlers use `addEventListener`; `isValidCartItem`/`isValidCustomer` in [index.js](../../index.js) enforce schema + length caps; `crypto.getRandomValues()` is used at [index.js:37](../../index.js#L37); no `Math.random(` occurrences remain. The formal CSP-hygiene scan, XSS-safe-DOM audit, and prototype-pollution audit as CI gates are deferred to `/speckit.tasks`.
- [x] **P-V. Client-Only Architecture (WhatsApp Handoff)** — **PASS by construction**. No backend, no analytics, no cookies, no service worker. State lives in `krishidakshina.cart.v1` and `krishidakshina.customer.v1`. Orders open `https://wa.me/${WHATSAPP_NUMBER}?text=…` (WHATSAPP_NUMBER declared at [index.js:92](../../index.js#L92)); message length is bounded by `MAX_MSG_LEN = 3800` at [index.js:535](../../index.js#L535). The only allow-listed `fetch()` origin is `api.postalpincode.in`.
- [ ] **P-VI. Verifiable Releases** — **FAIL (documented as GAP → task in `/speckit.tasks`)**. Deployment today is `git push` → GitHub Pages auto-publish from the default branch; there is no `.github/workflows/lighthouse.yml`, no automated a11y scan, no CSP-hygiene gate, and no `Math.random(` static scan in CI. This is the single blocking gap for the baseline; adding a `lighthouse.yml` workflow (with the four Lighthouse thresholds ≥ 90, an axe-core step, a CSP-hygiene step, and a `Math.random(` grep) is captured under Complexity Tracking below and will be broken out as tasks in `/speckit.tasks`.

**Gate outcome**: PASS with two informational gaps (P-I audit, P-II first measurement) and **one FAIL (P-VI)**. The FAIL is deliberate for this baseline plan — the site is already shipped and the CI gates are the very next work item. Recorded in Complexity Tracking so the gap is not silently tolerated.

## Project Structure

### Documentation (this feature)

```text
specs/001-storefront-baseline/
├── plan.md                       # This file (/speckit.plan output)
├── research.md                   # Phase 0 output — reverse-engineered decisions
├── data-model.md                 # Phase 1 output — Product / CartItem / Customer / OrderMessage
├── quickstart.md                 # Phase 1 output — manual smoke validation guide
├── contracts/
│   ├── csp.md                    # meta CSP verbatim + amendment procedure
│   ├── localstorage-schema.md    # krishidakshina.cart.v1 and krishidakshina.customer.v1
│   ├── postal-pincode-api.md     # GET /pincode/{pincode} shape + error handling
│   └── whatsapp-handoff.md       # wa.me URL construction + message layout
├── checklists/
│   └── requirements.md           # (pre-existing) requirements review checklist
└── spec.md                       # (pre-existing) feature specification
```

### Source Code (repository root)

The **canonical, frozen** layout for this baseline is:

```text
./
├── CNAME                         # krishidakshina.in — GitHub Pages custom domain
├── index.html                    # single-page markup; strict meta CSP lives here (Principle IV)
├── index.css                     # all styles; no inline styles anywhere (Principle IV)
├── index.js                      # all client script; 'use strict'; addEventListener only
│                                 # (state keys: krishidakshina.cart.v1, krishidakshina.customer.v1)
├── product_images/               # (TO BE CREATED) 12 images at 600×600, ≤ 200 KB each
│                                 # filenames must match index.html src attributes:
│                                 # tomato / eggs / spinach / bread / mango / milk /
│                                 # carrot / yogurt / brown-rice / avocado /
│                                 # mixed-nuts / turmeric (.jpg or .webp/.avif)
├── .github/                      # Copilot prompts, agents; CI workflows to be added
│                                 # (future: .github/workflows/lighthouse.yml — Principle VI)
├── .specify/                     # Spec Kit engine, templates, memory, workflows
├── .vscode/                      # workspace settings
└── .gitignore
```

**Structure Decision**: **Static site (DEFAULT — Principle III)**. Three sibling files (`index.html`, `index.css`, `index.js`) at repo root plus `CNAME` for GitHub Pages. The `src/` + `dist/` alternative from the template is explicitly rejected — it would require a build step, which is a Principle III amendment. Adding `product_images/` and `.github/workflows/lighthouse.yml` are the only structural changes contemplated by this baseline; both fit inside the existing decision.

## Complexity Tracking

The baseline itself introduces **no** deviations from the constitution. The tracker below records the one open gap the plan carries forward and the specific follow-ups already flagged in the constitution's Sync Impact Report so they can't drift.

| Violation / Gap | Why Needed | Simpler Alternative Rejected Because |
|-----------------|------------|--------------------------------------|
| **P-VI FAIL: no CI, no Lighthouse gate, no CSP-hygiene / `Math.random(` scans** | Constitution v2.1.0 requires these gates to block merges; the current `git push` → Pages flow has zero automated gates. | "Rely on manual review before every merge" — rejected because Principle VI explicitly says gates encode the constitution so principles survive contributor churn. Task carried to `/speckit.tasks`: add `.github/workflows/lighthouse.yml` with Lighthouse CI thresholds ≥ 90 on all four categories, an axe-core step, a CSP-hygiene grep (no `'unsafe-inline'` / `'unsafe-eval'` in `script-src`/`style-src`), and a `Math.random(` grep against `index.js`. |
| **P-I audit not yet run** (informational) | Principle I requires an axe/Pa11y-clean scan and a manual keyboard/focus/zoom pass on every shipped state. The current baseline has never been formally audited. | "Assume the semantic HTML is enough" — rejected because the constitution explicitly requires the automated + manual audit as an artifact of every release. Task carried to `/speckit.tasks`. |
| **P-II first Lighthouse measurement not yet run** (informational) | Principle II budgets (LCP/CLS/INP/bundle sizes) are hard limits but have never been measured against the shipped site. | "Assume vanilla is fast enough" — rejected: budgets exist to catch the *next* regression, so we need a numeric baseline recorded in a Lighthouse artifact. Task carried to `/speckit.tasks`. |
| **`WHATSAPP_NUMBER`, email, phone, address duplicated across `index.html` and `index.js`** | Governance follow-up `CONTACT_DETAILS_CENTRALIZATION` in the constitution's Sync Impact Report says this MUST be centralized before v1 release. | "Leave duplicated" — rejected by the constitution's Governance section; drift risk is real. Not a plan-level violation (the current code is compliant with every principle), but tracked here so `/speckit.tasks` picks it up. |
| **`product_images/` folder does not yet exist** | Governance follow-up `PRODUCT_IMAGES`; 12 `<img src="product_images/…">` references currently 404 and fall back to emoji. | "Ship without images" — the emoji fallback is intentional and safe (constitution FR-026), so this is *not* a defect, but launch quality requires either shipping the images or documenting the emoji fallback as final. Deferred to `/speckit.tasks` under the constitution's open question OQ-03. |

Everything else is **PASS by construction** — no framework, no bundler, no build step, no backend, no cookies, no analytics, no service worker, strict CSP with SRI, versioned `localStorage`, `crypto.getRandomValues()` only.

## Phase 0 (Research) — see [research.md](./research.md)

Phase 0 is intentionally short: this is a reverse-engineered baseline, not a new architecture. `research.md` consolidates each already-taken decision (language, hosting, external deps, state, security posture, testing, observability) with a one-line rationale and the alternative rejected. Where the plan template would normally ask for POCs or spikes, the entry reads: **"Not applicable — reverse-engineered from shipped code; reference implementation is `index.html` / `index.css` / `index.js`."**

## Phase 1 (Design & Contracts) — outputs

- [data-model.md](./data-model.md) — `Product`, `CartItem`, `Customer`, `GeoLoc`, and the ephemeral `OrderMessage` entity with field types, validation rules, cardinality, and state transitions.
- [contracts/](./contracts/) — four contracts:
  1. [contracts/csp.md](./contracts/csp.md) — the verbatim CSP + amendment procedure.
  2. [contracts/localstorage-schema.md](./contracts/localstorage-schema.md) — `krishidakshina.cart.v1` and `krishidakshina.customer.v1` shapes + rejection rules.
  3. [contracts/postal-pincode-api.md](./contracts/postal-pincode-api.md) — request/response shape for `api.postalpincode.in/pincode/{pincode}` and failure modes.
  4. [contracts/whatsapp-handoff.md](./contracts/whatsapp-handoff.md) — `wa.me` URL layout, message template, encoding rules, and the 3800-char cap.
- [quickstart.md](./quickstart.md) — manual smoke validation guide (8 steps) that a reviewer can run against a local static server to verify the baseline still works end-to-end.

The agent-context refresh script referenced by the plan template (Windows/PowerShell variant) is not present in `.specify/scripts/powershell/` (only `check-prerequisites.ps1`, `common.ps1`, `create-new-feature.ps1`, `setup-plan.ps1`, and `setup-tasks.ps1` exist), so no agent-context file was regenerated in this pass. If a Copilot-facing agent context is added later, it should cite this plan and the constitution.

## Post-Design Re-evaluation of the Constitution Check

Re-checked after generating `research.md`, `data-model.md`, `quickstart.md`, and the four `contracts/` files. No design decision changed the gate outcomes above:

- **P-I / P-II** — still informational gaps; the artifacts explicitly hand the audit + first Lighthouse run to `/speckit.tasks`.
- **P-III / P-V** — the artifacts confirm no build step, no backend, no analytics, no service worker were introduced.
- **P-IV** — the CSP contract, the `localStorage` schema contract, the pincode contract, and the WhatsApp contract each restate the Principle IV rules (SRI, XSS-safe DOM, schema validation, `noopener noreferrer`, `crypto.getRandomValues()`) as their own amendment procedures.
- **P-VI** — still the sole FAIL; unchanged and captured in Complexity Tracking.

## Downstream

- `/speckit.tasks` picks up: the P-VI CI gap (`.github/workflows/lighthouse.yml`), the P-I audit, the P-II first Lighthouse measurement, `CONTACT_DETAILS_CENTRALIZATION`, and the `product_images/` decision (ship images vs. codify emoji fallback).
- Any future change touching `index.html`, `index.css`, or `index.js` MUST cite which spec FR/US it modifies and re-run the CI gates once they exist.
