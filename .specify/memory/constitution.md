<!--
Sync Impact Report
==================
Version change: 2.0.0 → 2.1.0
Ratification date: 2026-07-18 (unchanged)
Last amended date: 2026-07-18

Rationale for MINOR bump: two new governance rules were added (testimonial
permission rule + contact-form-migration procedure), one open question was
resolved (canonical brand = "KrishiDakshina"), and a namespace rename
(`gutpoint.*` → `krishidakshina.*`) was cascaded through the constitution and
its dependent templates. No existing principle was removed or redefined; the
change strictly expands governance guidance and updates identifiers.

Resolved open questions:
  - OQ-01 / BRAND_DOMAIN_RECONCILIATION → RESOLVED 2026-07-18: canonical brand
    is "KrishiDakshina" (single word, capital K and D). Site markup, meta tags,
    WhatsApp message header, and localStorage namespace have been rebranded
    from "Gut Point" / `gutpoint.*` to "KrishiDakshina" / `krishidakshina.*`.
  - OQ-04 / TESTIMONIALS → RESOLVED 2026-07-18 (as placeholders): the three
    testimonials in `index.html` (Priya Sharma / Rahul Mehta / Ananya Iyer)
    are confirmed placeholders and MUST NOT be published as production content
    without written permission from real customers (see new Governance rule).
  - OQ-05 / CONTACT_FORM_FATE → RESOLVED 2026-07-18 (same-day post-amendment
    closure, no version bump): the visible contact form now submits via a
    WhatsApp handoff (`window.open` on `https://wa.me/${WHATSAPP_NUMBER}?text=`)
    consistent with the order flow. This path is expressly permitted by
    Principle V clause 6 sub-clause (a) as "a WhatsApp handoff consistent
    with #4"; it introduces no new external origin (`wa.me` was already on
    the runtime allow-list under clause 4), requires no CSP change, and
    WhatsApp's native phone-number verification serves as the anti-spam
    mechanism the "Contact-form migration procedure" governance rule
    requires. Any *future* migration to a third-party form endpoint
    (Web3Forms / Formspree / Netlify Forms / etc.) still triggers the full
    procedure (constitution MINOR amendment + CSP allow-list amendment +
    declared anti-spam mechanism) and is scoped as post-v1 task T020a in
    `specs/001-storefront-baseline/tasks.md`.

Added governance rules (new since v2.0.0):
  - Testimonial permission requirement — placeholder testimonials MUST be
    labeled as example content in a source-file comment and MUST NOT ship as
    production copy without documented, written permission from the named
    customer. (MINOR-level governance expansion.)
  - Contact-form migration procedure — any migration of the contact form off
    the current client-side simulation REQUIRES (a) a constitution MINOR
    amendment listing the new origin(s), (b) a CSP allow-list amendment, and
    (c) an anti-spam mechanism whose provider (if any) is also listed in the
    constitution. (MINOR-level governance expansion.)

Namespace rename (identifier update, not a schema change):
  - `gutpoint.cart.v1` → `krishidakshina.cart.v1`
  - `gutpoint.customer.v1` → `krishidakshina.customer.v1`
  - The value shape is unchanged; the `.v1` suffix is preserved. Any browser
    that previously stored data under `gutpoint.*` will present an empty cart
    under the new key (pre-launch, acceptable). Documented as a one-time reset
    event dated 2026-07-18 in
    `specs/001-storefront-baseline/contracts/localstorage-schema.md`
    (Migration policy section).

Templates requiring updates:
  - ✅ .specify/templates/plan-template.md — storage key references updated
       to `krishidakshina.cart.v1` / `krishidakshina.customer.v1`.
  - ✅ .specify/templates/tasks-template.md — storage key pattern hint updated
       to `krishidakshina.<name>.v<N>` and current keys renamed accordingly.
  - ✅ .specify/templates/spec-template.md — no change required
       (technology-agnostic).
  - ✅ .specify/templates/checklist-template.md — no change required
       (content-agnostic scaffold).
  - ⚠  README.md / docs/quickstart.md — not present in repository; propagate
       when authored (unchanged from v2.0.0).

Feature artifacts updated (cascade):
  - ✅ specs/001-storefront-baseline/spec.md — narrative rebrand, storage-key
       rename, OQ-01 marked RESOLVED, OQ-04 marked RESOLVED-AS-PLACEHOLDER.
  - ✅ specs/001-storefront-baseline/plan.md — title, narrative, and storage
       key references rebranded.
  - ✅ specs/001-storefront-baseline/research.md — title + R-* storage key
       references rebranded.
  - ✅ specs/001-storefront-baseline/data-model.md — title, table entries,
       Mermaid diagram labels, WhatsApp message header rebranded.
  - ✅ specs/001-storefront-baseline/quickstart.md — title, storage key
       mentions, WhatsApp message header rebranded.
  - ✅ specs/001-storefront-baseline/tasks.md — title, description, narrative,
       Section A storage-key and WhatsApp-header references rebranded; T016
       marked DONE (2026-07-18); T017 description updated to note the email
       placeholder retention; T019 marked DONE (2026-07-18 — testimonials
       confirmed placeholders); Section B totals adjusted.
  - ✅ specs/001-storefront-baseline/contracts/localstorage-schema.md — all
       key names rebranded (table, section headers, examples, migration rules,
       "What is not stored"); Migration policy notes the 2026-07-18 one-time
       reset.
  - ✅ specs/001-storefront-baseline/contracts/whatsapp-handoff.md — message
       header example and storage-key narrative rebranded.
  - ✅ specs/001-storefront-baseline/checklists/requirements.md — title
       rebranded.

Follow-up TODOs (deferred, not blocking amendment):
  - RESOLVED 2026-07-18 (was BRAND_DOMAIN_RECONCILIATION): canonical brand is
    KrishiDakshina.
  - TODO(CONTACT_DETAILS_CENTRALIZATION): DEFERRED — owner will supply
    production contact details; centralization work is blocked until then.
    Note: the email address `hello@gutpoint.com` intentionally retains the
    old brand string as a placeholder and will be updated when the owner
    provides the production address.
  - TODO(PRODUCT_IMAGES): DEFERRED — owner will provide product images later;
    the `product_images/` folder is still not populated and the emoji
    fallback remains the interim behaviour.
  - RESOLVED 2026-07-18 (was CONTACT_FORM_FATE): the contact form submits
    via a WhatsApp handoff (Principle V clause 6(a)). Path E — migration to
    a third-party form endpoint — remains available and is scoped as
    post-v1 task T020a (`specs/001-storefront-baseline/tasks.md`); it would
    still trigger the full "Contact-form migration procedure" governance
    rule below.

Prior amendment (v1.0.0 → v2.0.0) rationale is preserved in the repository's
Git history; the substantive changes it introduced (Security-by-Default,
Client-Only Architecture, removal of Bilingual Content, allow-list codification)
remain in force under v2.1.0.
-->

# krishidakshina Constitution

## Core Principles

### I. Accessibility-First (NON-NEGOTIABLE)

Every page, component, and user-facing change MUST meet WCAG 2.2 Level AA. Semantic HTML
is the default — use landmark elements (`<header>`, `<nav>`, `<main>`, `<footer>`), correct
heading order, and native form controls before reaching for ARIA. All interactive
elements MUST be reachable and operable by keyboard alone, with visible focus states
that meet WCAG 2.4.11 (Focus Not Obscured) and 2.4.13 (Focus Appearance) contrast rules.
Images MUST have meaningful `alt` text (or empty `alt=""` if purely decorative), color
MUST NOT be the sole means of conveying information, and the page MUST remain usable at
200% zoom.

Motion and animation MUST respect `@media (prefers-reduced-motion: reduce)` — the
current codebase disables the truck animation and hover-motion effects under that
query, and every future animated affordance MUST provide the same fallback.

**Rationale**: The site targets farmers and rural users on varied devices and assistive
technologies; excluding any of them undermines the site's purpose. Accessibility bolted
on after the fact costs more and delivers less.

### II. Performance Budget

The following budgets are hard limits, measured on a simulated 4G Fast connection
(9 Mbps down / 1.5 Mbps up, 150 ms RTT) with 4× CPU throttling using Lighthouse mobile:

- Largest Contentful Paint (LCP): < 2.0 s
- Total shipped JavaScript per page: ≤ 50 KB gzipped
- Total shipped CSS per page: ≤ 30 KB gzipped
- Cumulative Layout Shift (CLS): < 0.1
- Interaction to Next Paint (INP): < 200 ms

Images MUST be served as AVIF or WebP with a JPEG/PNG fallback via `<picture>`, sized
via `width`/`height` attributes to reserve layout space, and lazy-loaded
(`loading="lazy"`) when below the fold. Product images MUST be optimized to under
200 KB each at 600×600 (documented in the HTML author comment next to the product
grid). Fonts MUST be self-hosted OR loaded from an allow-listed origin (see
Additional Constraints), subset to required glyphs, and loaded with
`font-display: swap`. Any change that regresses a budget MUST be reverted or
accompanied by a justified amendment.

**Rationale**: The audience is bandwidth- and battery-constrained. Enforcing budgets in
CI is cheaper than reclaiming performance after regressions accumulate.

### III. Static-First & Framework-Minimal

The site MUST remain deployable to GitHub Pages via the existing `CNAME` file with no
server-side runtime. The current baseline is **vanilla HTML, CSS, and JavaScript with
zero runtime dependencies fetched at build time and no `package.json`**. That baseline
is the default and MUST be preserved.

Adding a framework, bundler, transpiler, or build step (including but not limited to
React, Vue, Svelte, Astro, Vite, webpack, Babel, TypeScript compilation, or a CSS
preprocessor) requires an amendment PR that documents:

1. The concrete problem that vanilla HTML/CSS/JS cannot solve.
2. The impact on the Performance Budget (Principle II).
3. The impact on the GitHub Pages deployment path.
4. A simpler alternative that was considered and rejected, with reasons.

Third-party runtime dependencies (CDN scripts, hosted widgets, embed iframes) are
subject to the same review and to Principles IV and V.

**Rationale**: Simplicity is a feature. Frameworks introduce bytes, complexity, and
supply-chain surface area; each addition must earn its place.

### IV. Security-by-Default (NON-NEGOTIABLE)

Security postures already encoded in the source MUST be preserved and MUST NOT be
weakened without a MAJOR amendment. Every change MUST comply with the following:

1. **Content Security Policy**: The page MUST ship a strict CSP (declared via
   `<meta http-equiv="Content-Security-Policy">` or an HTTP header) with:
   - `default-src 'self'`
   - `script-src 'self'` — **no `'unsafe-inline'`, no `'unsafe-eval'`, no inline
     `<script>` bodies, no inline event handlers (`onclick=`, etc.)**
   - `style-src 'self'` plus only origins on the runtime allow-list
   - `img-src 'self' data: https:`
   - `connect-src 'self'` plus only origins on the runtime allow-list
   - `frame-ancestors 'none'`
   - `object-src 'none'`
   - `form-action 'self'`
   - `base-uri 'self'`
   - `Referrer-Policy: strict-origin-when-cross-origin`

2. **No inline JS / CSS / event handlers.** All scripts and styles live in external
   files. The source comments already state this is a deliberate choice "so the page
   can drop `unsafe-inline`" — that choice is now constitutional.

3. **SRI-pinned CDN scripts and stylesheets.** Any resource fetched from a
   third-party origin that could execute code MUST carry an `integrity=` hash,
   `crossorigin="anonymous"`, and (where practical) `referrerpolicy="no-referrer"`.

4. **XSS-safe DOM APIs.** Any string that could originate from user input, network
   responses, or persisted storage MUST be inserted with `textContent` /
   `createElement` / `setAttribute`. `innerHTML`, `outerHTML`, `document.write`, and
   `insertAdjacentHTML` are BANNED for user-controlled data.

5. **Prototype-pollution defense.** Data hydrated from `localStorage`,
   `sessionStorage`, URL params, or the network MUST be validated by a strict schema
   that (a) whitelists known keys, (b) enforces per-field type checks, and (c) caps
   string lengths. `isValidCartItem` and `isValidCustomer` in `index.js` are the
   reference pattern.

6. **Input validation.** Every persisted or transmitted field MUST have a length cap
   and, where format matters, a regex gate. The current baseline is:
   phone `/^[6-9]\d{9}$/`, pincode `/^[1-9]\d{5}$/`,
   `MAX_QTY_PER_ITEM = 99`, `MAX_CART_ITEMS = 50`, `MAX_MSG_LEN = 3800`.

7. **Randomness.** `crypto.getRandomValues()` MUST be used in place of
   `Math.random()`, even for decorative values (Sonar-clean). `Math.random()` is
   BANNED in production JS.

8. **External links.** Every `target="_blank"` link MUST carry
   `rel="noopener noreferrer"`.

9. **Opt-in privacy-sensitive APIs.** Geolocation, camera, microphone,
   notifications, clipboard, and any similar API MUST be triggered by an explicit
   user action (button click) and MUST NOT be auto-requested on page load.

Any deviation from the above MUST be (a) justified in the plan under Complexity
Tracking, (b) documented in the CSP allow-list, and (c) reviewed as a
security-impacting amendment.

**Rationale**: The site handles addresses, phone numbers, and hands orders off to a
messaging service; a small security regression translates directly to user harm.
Encoding the current hardened posture as a principle prevents silent drift.

### V. Client-Only Architecture (WhatsApp Handoff) (NON-NEGOTIABLE)

The site is a purely client-side static application. The following MUST hold:

1. **No backend, no server-side runtime.** GitHub Pages serves static files only.
   Introducing any server-side component (API, function-as-a-service, database,
   auth service, form-submission backend) requires a **MAJOR** constitution
   amendment.
2. **No analytics, no tracking pixels, no cookies, no service worker.** These are
   banned by construction. If any of them is later needed, it requires (a) a MAJOR
   amendment, (b) explicit user opt-in that defaults to reject, (c) a CSP
   amendment, and (d) an update to the runtime allow-list.
3. **State lives in `localStorage` with versioned keys.** The current keys are
   `krishidakshina.cart.v1` and `krishidakshina.customer.v1`; any schema change
   MUST bump the version segment and ship a migration or a fresh-start path.
   Hydrated state MUST pass the strict validators from Principle IV. A pure
   namespace rename (e.g., 2026-07-18: `gutpoint.*` → `krishidakshina.*`) is
   NOT a schema change and preserves the `.vN` suffix; the previous keys are
   simply orphaned and any browser that held them presents an empty cart under
   the new namespace (pre-launch acceptable — see
   `specs/001-storefront-baseline/contracts/localstorage-schema.md`
   Migration policy).
4. **Order placement is a WhatsApp handoff.** Orders are placed by opening a
   `wa.me` URL with a formatted message. The generated message MUST respect
   `MAX_MSG_LEN = 3800`. The destination number is currently hard-coded as
   `WHATSAPP_NUMBER = "919876543210"` in `index.js`; it MUST be centralized (see
   Governance follow-up) but MUST NOT be moved to a server-side lookup without a
   MAJOR amendment.
5. **Third-party API calls.** Any `fetch()` to a third-party origin MUST hit an
   origin that is (a) on the runtime allow-list in Additional Constraints, (b)
   listed in `connect-src`, and (c) invoked with a no-referrer policy where the
   origin does not require the referrer. The current allow-listed API is
   `https://api.postalpincode.in/pincode/` for pincode → city/state lookup.
6. **Contact form submissions.** The visible contact form submits via a
   WhatsApp handoff (spinner → `window.open` on
   `https://wa.me/${WHATSAPP_NUMBER}?text=<encoded enquiry>` → success toast
   "WhatsApp opened with your message"). It MUST NOT silently transmit data
   to any other endpoint. Any migration off this WhatsApp handoff to a
   different real-delivery mechanism (email, third-party form service,
   in-house endpoint, etc.) MUST either (a) remain a WhatsApp handoff
   consistent with #4 (the current path — pre-approved, no amendment
   required), or (b) trigger this principle's MAJOR-amendment clause
   (backend / serverless / analytics) AND the Governance-level
   "Contact-form migration procedure" rule (constitution MINOR amendment +
   CSP allow-list amendment + declared anti-spam mechanism).

**Rationale**: The zero-backend posture is what makes the site cheap to run, private
by construction, and immune to server outages. Codifying it prevents the usual creep
toward "just add a tiny endpoint".

### VI. Verifiable Releases

A change is not "done" until it passes automated release gates. Every PR MUST:

1. Produce a preview build reachable at a stable URL (a preview deployment or
   published artifact reviewers can open).
2. Pass a Lighthouse audit (mobile profile, 4G Fast throttling) with each of
   Performance, Accessibility, Best Practices, and SEO scoring **≥ 90**.
3. Pass an automated accessibility check (e.g., axe-core, Pa11y) with zero
   serious/critical violations.
4. Pass a **CSP hygiene gate**: the served page's CSP MUST contain neither
   `'unsafe-inline'` nor `'unsafe-eval'` in `script-src` or `style-src`.
5. Pass a **`Math.random` gate**: static scan of production JS MUST find zero
   occurrences of `Math.random(` (Principle IV, clause 7).
6. Include, in the PR description, the principles the change impacts and a
   before/after Lighthouse summary when Principle II or VI thresholds are near
   the limit.

Failing gates MUST block merge. Waivers are permitted only via an amendment PR
that records the deviation and its expiry.

**Rationale**: Manual review misses regressions; gates encode the constitution in CI
so principles survive contributor churn.

## Additional Constraints & Standards

- **Hosting**: GitHub Pages, custom domain via the tracked `CNAME` file
  (`krishidakshina.in`). Any change that would break the GitHub Pages deploy
  (server, dynamic routing, non-static APIs) is out of scope without an amendment
  (see Principle V).
- **Browser support**: Latest two stable versions of Chrome, Firefox, Safari, and
  Edge, plus Chrome on Android and Safari on iOS. Graceful degradation, not
  polyfill bloat, is the strategy for older engines.
- **Runtime origin allow-list** (MUST match the CSP in `index.html`):
  - `cdnjs.cloudflare.com` — Font Awesome 6.5.0 CSS, **SRI-pinned**
    (`style-src`).
  - `fonts.googleapis.com` — Inter font CSS (`style-src`).
  - `fonts.gstatic.com` — Inter font files (`font-src`).
  - `api.postalpincode.in` — pincode → city/state lookup (`connect-src`).
  - `wa.me` — WhatsApp order handoff (navigation only; not a `connect-src`).

  Adding a new external origin requires ALL of: (a) a CSP amendment in
  `index.html`, (b) a constitution amendment (MINOR if it introduces a new
  dependency category, MAJOR if it violates Principle V, e.g., analytics),
  and (c) an SRI hash where the origin serves executable content (scripts).
- **Repository hygiene**: `index.html`, `index.css`, `index.js`, and `CNAME` are
  the canonical roots. New assets live under predictable directories
  (`product_images/`, `assets/`) rather than the repository root. No hidden
  build outputs are committed.
- **SEO baseline**: Every page has a unique `<title>`, a meta description, Open
  Graph tags, and a canonical URL. A `sitemap.xml` and `robots.txt` MUST be
  maintained.

## Development Workflow & Quality Gates

- **Change model**: All non-trivial changes flow through a PR against the
  default branch. Direct pushes are limited to typo/README fixes.
- **PR requirements**:
  - Reference the principles the change impacts.
  - Include a preview URL or deploy artifact.
  - Attach the Lighthouse summary produced by CI.
  - For any change touching `index.html`, `index.js`, or the CSP: confirm the
    Security-by-Default gates (Principle IV) still pass.
- **CI gates** (enforced automatically):
  1. HTML validation (nu HTML checker or equivalent).
  2. Link check for internal links.
  3. Lighthouse CI with the thresholds in Principle VI.
  4. Automated accessibility scan (Principle I).
  5. Bundle-size check against Principle II budgets.
  6. CSP hygiene scan and `Math.random` static scan (Principle VI, clauses
     4–5).
- **Reviewer checklist**: `.specify/templates/checklist-template.md` can be used
  to generate feature-specific review checklists that expand on these gates.
- **Complexity justification**: Any deviation from Principles I–VI, or any
  addition of a framework/build tool under Principle III, or any new external
  origin under the allow-list, MUST be recorded in the plan under "Complexity
  Tracking" with the simpler alternative that was rejected.

## Governance

- **Authority**: This constitution supersedes ad-hoc decisions, prior
  conventions, and undocumented preferences. Where any document conflicts with
  this constitution, the constitution wins until amended.
- **Amendments**: Amendments are made via a PR that (a) edits this file, (b)
  updates the version and dates below, (c) prepends/updates the Sync Impact
  Report comment at the top of this file, and (d) updates any dependent
  templates in `.specify/templates/`. Amendments require review and merge like
  any other change.
- **Versioning policy** (Semantic Versioning for governance):
  - **MAJOR**: Backwards-incompatible removal or redefinition of a principle,
    introduction of a backend/analytics/cookies (violating Principle V), or a
    change that invalidates existing plans/specs.
  - **MINOR**: Addition of a new principle or section, materially expanded
    guidance within an existing principle, or a new external origin on the
    runtime allow-list.
  - **PATCH**: Wording, typo, or clarifying edits that do not change intent.
- **Compliance review**: Every PR reviewer MUST confirm the change is
  consistent with this constitution before approval. Repeated violations of a
  principle are grounds for either (a) tightening enforcement in CI, or (b)
  amending the principle — never for silent tolerance.
- **Testimonial permission requirement** (NEW in v2.1.0): Any customer
  testimonial rendered in the site MUST either (a) be signed by a real,
  named customer with documented, written permission on file, OR (b) be
  labeled as example content in a source-file comment adjacent to the
  markup that renders it. Placeholder testimonials MUST NOT ship as
  production copy — the site is BLOCKED from launch while any
  testimonial-shaped block visible on the page lacks either the
  permission record or the explicit "example content" comment. The three
  testimonials currently in `index.html` (Priya Sharma, Rahul Mehta,
  Ananya Iyer) are confirmed placeholders per OQ-04 (RESOLVED 2026-07-18)
  and are already labeled with a source comment marking them as example
  content.
- **Contact-form migration procedure** (NEW in v2.1.0): The visible contact
  form submits via a WhatsApp handoff in v1 (Principle V clause 6, resolved
  2026-07-18 — see Sync Impact Report). Any migration off that WhatsApp
  handoff to a real inbox / third-party endpoint delivery mechanism (email,
  form service, in-house endpoint, etc.) REQUIRES ALL of:
  1. A **constitution MINOR amendment** listing the new origin(s) on the
     runtime allow-list (or, if data leaves via an origin that is not a
     fetch target, on an equivalent explicit inventory).
  2. A **CSP allow-list amendment** in `index.html` — at minimum an update
     to `form-action` and/or `connect-src` and, where relevant, an SRI
     hash for any third-party script the endpoint requires.
  3. An **anti-spam mechanism** whose provider (if any) is itself listed
     in the constitution as an external origin (honeypot fields do not
     require an origin entry; CAPTCHA providers do). If no anti-spam
     control is added, the amendment MUST justify that choice.

  Skipping any of the three steps voids the amendment. A migration that
  introduces server-side order storage, an in-house backend, or any form
  of tracking simultaneously triggers Principle V's MAJOR-amendment
  clause and cannot proceed as a MINOR change.
- **Deferred / open questions**:
  - **Brand vs. domain** — **RESOLVED 2026-07-18**: canonical brand is
    **"KrishiDakshina"** (single word, capital K and D, spelled exactly
    that way). The site markup, meta tags, WhatsApp message header, and
    `localStorage` namespace have been rebranded accordingly. Historical
    references to "Gut Point" / `gutpoint.*` are being cascaded out of the
    Spec Kit artifacts under this amendment (see Sync Impact Report). The
    email address `hello@gutpoint.com` is preserved verbatim as a
    placeholder pending owner-supplied contact details.
  - **Hindi / multilingual support**: the site currently ships in English
    only. Bilingual (Hindi) support is a **deferred, post-v1 goal** and is
    intentionally not a principle in v2.x. Reintroducing it later would
    be a MINOR amendment that adds a Bilingual Content principle back and
    updates the plan/tasks templates.
  - **Hard-coded contact details** — **DEFERRED**: owner will supply
    production contact details; centralization work is blocked until
    then. Note: the email address `hello@gutpoint.com` intentionally
    retains the old brand string as a placeholder and will be updated
    when the owner provides the production address. Until the owner
    supplies the values, the WhatsApp number `919876543210`, phone
    `+91 98765 43210`, email `hello@gutpoint.com`, and Mumbai address
    remain in place duplicated across `index.html` / `index.js`.
  - **Contact-form fate** — **RESOLVED 2026-07-18**: the contact form
    submits via a WhatsApp handoff (Principle V clause 6(a)). This
    reuses the already-declared `wa.me` origin — no new origin, no CSP
    change. A future migration to a third-party form endpoint
    (Web3Forms recommended pre-audit) is scoped as post-v1 task T020a
    in `specs/001-storefront-baseline/tasks.md` and would trigger the
    full "Contact-form migration procedure" governance rule.
- **Runtime guidance**: Contributor-facing "how to work in this repo"
  instructions live in `README.md` (to be authored) and in agent/IDE guidance
  files under `.github/` or `.specify/`. Those documents MUST cite this
  constitution and MUST NOT contradict it.

**Version**: 2.1.0 | **Ratified**: 2026-07-18 | **Last Amended**: 2026-07-18
