# Implementation Plan: [FEATURE]

**Branch**: `[###-feature-name]` | **Date**: [DATE] | **Spec**: [link]

**Input**: Feature specification from `/specs/[###-feature-name]/spec.md`

**Note**: This template is filled in by the `/speckit.plan` command. See `.specify/templates/plan-template.md` for the execution workflow.

## Summary

[Extract from feature spec: primary requirement + technical approach from research]

## Technical Context

<!--
  ACTION REQUIRED: Replace the content in this section with the technical details
  for the project. The structure here is presented in advisory capacity to guide
  the iteration process.
-->

**Language/Version**: [e.g., Python 3.11, Swift 5.9, Rust 1.75 or NEEDS CLARIFICATION]

**Primary Dependencies**: [e.g., FastAPI, UIKit, LLVM or NEEDS CLARIFICATION]

**Storage**: [if applicable, e.g., PostgreSQL, CoreData, files or N/A]

**Testing**: [e.g., pytest, XCTest, cargo test or NEEDS CLARIFICATION]

**Target Platform**: [e.g., Linux server, iOS 15+, WASM or NEEDS CLARIFICATION]

**Project Type**: [e.g., library/cli/web-service/mobile-app/compiler/desktop-app or NEEDS CLARIFICATION]

**Performance Goals**: [domain-specific, e.g., 1000 req/s, 10k lines/sec, 60 fps or NEEDS CLARIFICATION]

**Constraints**: [domain-specific, e.g., <200ms p95, <100MB memory, offline-capable or NEEDS CLARIFICATION]

**Scale/Scope**: [domain-specific, e.g., 10k users, 1M LOC, 50 screens or NEEDS CLARIFICATION]

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

Evaluate the plan against each principle in `.specify/memory/constitution.md`. For every
gate, mark PASS / FAIL / N/A and add a one-line justification. Any FAIL blocks the phase
and must either be resolved or recorded in Complexity Tracking with an amendment plan.

- [ ] **I. Accessibility-First** — Change preserves WCAG 2.2 AA, semantic HTML, keyboard
  operability, and visible focus states. Animated affordances honor
  `prefers-reduced-motion`. New UI ships with an axe/Pa11y-clean scan.
- [ ] **II. Performance Budget** — Change keeps LCP < 2.0 s on simulated 4G, JS ≤ 50 KB
  gz/page, CSS ≤ 30 KB gz/page, CLS < 0.1, INP < 200 ms. Product images ≤ 200 KB at
  600×600, AVIF/WebP with fallbacks, `width`/`height` reserved, `loading="lazy"`
  below the fold.
- [ ] **III. Static-First & Framework-Minimal** — Change deploys to GitHub Pages via the
  existing `CNAME` with no server runtime and no `package.json`. Any new
  framework/build tool/CDN dep is justified in Complexity Tracking with a rejected
  simpler alternative.
- [ ] **IV. Security-by-Default** — CSP remains strict (`default-src 'self'`,
  `script-src 'self'`, no `'unsafe-inline'`/`'unsafe-eval'`, `frame-ancestors 'none'`,
  `object-src 'none'`, `form-action 'self'`, `base-uri 'self'`); no new inline
  JS/CSS/event handlers; any new CDN executable is SRI-pinned with
  `crossorigin="anonymous"`; user-controlled strings use `textContent`/`createElement`
  only; hydrated/persisted data passes a strict schema validator with length caps;
  regex + length caps on all persisted fields (phone `/^[6-9]\d{9}$/`, pincode
  `/^[1-9]\d{5}$/`, `MAX_QTY_PER_ITEM=99`, `MAX_CART_ITEMS=50`, `MAX_MSG_LEN=3800`);
  `crypto.getRandomValues()` used in place of `Math.random()`; every external
  `target="_blank"` link has `rel="noopener noreferrer"`; geolocation and other
  privacy-sensitive APIs are opt-in / button-triggered.
- [ ] **V. Client-Only Architecture (WhatsApp Handoff)** — No backend, no analytics,
  no cookies, no service worker. State lives in versioned `localStorage`
  (`krishidakshina.cart.v1`, `krishidakshina.customer.v1`) and passes the strict
  validators from Principle IV; schema changes bump the version segment. Orders open a
  `wa.me` URL with a message under `MAX_MSG_LEN=3800`. Any third-party `fetch()`
  hits an origin on the runtime allow-list and listed in `connect-src`. Contact
  form remains a client-side simulation or moves to a WhatsApp handoff — a real
  backend triggers a MAJOR constitution amendment.
- [ ] **VI. Verifiable Releases** — Preview URL is available; Lighthouse (mobile,
  4G Fast) scores ≥ 90 across Performance, Accessibility, Best Practices, SEO;
  automated a11y scan has zero serious/critical issues; CSP hygiene scan finds no
  `'unsafe-inline'`/`'unsafe-eval'` in `script-src`/`style-src`; static scan of
  production JS finds zero `Math.random(` occurrences.

## Project Structure

### Documentation (this feature)

```text
specs/[###-feature]/
├── plan.md              # This file (/speckit.plan command output)
├── research.md          # Phase 0 output (/speckit.plan command)
├── data-model.md        # Phase 1 output (/speckit.plan command)
├── quickstart.md        # Phase 1 output (/speckit.plan command)
├── contracts/           # Phase 1 output (/speckit.plan command)
└── tasks.md             # Phase 2 output (/speckit.tasks command - NOT created by /speckit.plan)
```

### Source Code (repository root)
<!--
  ACTION REQUIRED: Replace the placeholder tree below with the concrete layout
  for this feature. This project is a static, GitHub-Pages-hosted marketing site
  (Principle III: Static-First & Framework-Minimal). Prefer the DEFAULT static
  layout unless an amendment justifies a build pipeline.
-->

```text
# [REMOVE IF UNUSED] Static site (DEFAULT — Principle III)
./
├── index.html                # canonical entry (strict CSP lives here — Principle IV)
├── index.css                 # global styles (≤ 30 KB gz — Principle II); no inline styles
├── index.js                  # progressive-enhancement scripts (≤ 50 KB gz);
│                             #   no inline handlers; localStorage keys
│                             #   `krishidakshina.cart.v1`, `krishidakshina.customer.v1` (Principle V)
├── CNAME                     # GitHub Pages custom domain (do not remove)
├── product_images/           # product photography (AVIF/WebP + fallback, ≤ 200 KB @ 600×600)
├── assets/                   # optimized non-product images, self-hosted fonts, icons
└── .github/workflows/        # CI: HTML validate, link check, Lighthouse, a11y,
                              #   bundle size, CSP hygiene, Math.random scan (Principle VI)

# [REMOVE IF UNUSED] Static site with a justified build step (requires constitution
# amendment per Principle III)
src/                          # authoring sources (templates, partials, tokens)
dist/                         # generated static output published to Pages
tools/                        # build/lint scripts
```

**Structure Decision**: [Document the selected structure and reference the real
directories captured above. Confirm that `CNAME`, `index.html` (with its strict CSP),
`index.css`, and `index.js` remain intact, and that GitHub Pages deployment stays
static-only (Principles III + V).]

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| [e.g., 4th project] | [current need] | [why 3 projects insufficient] |
| [e.g., Repository pattern] | [specific problem] | [why direct DB access insufficient] |
