# Quickstart & Validation Guide: Storefront Baseline (KrishiDakshina v1)

**Feature**: `001-storefront-baseline` · **Plan**: [plan.md](./plan.md) · **Spec**: [spec.md](./spec.md)

This guide is a **manual smoke validation** any reviewer can run to confirm the baseline still works end-to-end. It is intentionally lightweight because there is no test framework in v1 (see [research.md R-7](./research.md#r-7-testing-strategy)). Automated gates (Lighthouse CI, axe-core, CSP-hygiene, `Math.random(` scan) are the Principle VI gap tracked in the plan's Complexity Tracking table and will be added by `/speckit.tasks`.

## Prerequisites

- A modern evergreen browser (Chrome, Firefox, Safari, or Edge — latest two stable).
- Any static HTTP server. **Do not open `index.html` from the `file://` scheme** — the meta CSP has `default-src 'self'` and `connect-src 'self' https://api.postalpincode.in`, and some browsers treat `file://` origins as opaque, which will make relative asset requests fail.
- No Node.js, no `npm install`, no build step. The repo has no `package.json` by design (Principle III).

## Serve the site locally

Pick any one of the following. All are equivalent for this purpose.

```powershell
# From the repository root:

# Option A — Python 3 (built into most systems)
python -m http.server 8080

# Option B — Node (if installed; no lockfile is created)
npx --yes serve . -l 8080

# Option C — VS Code
# Install the "Live Server" extension, right-click index.html, "Open with Live Server"
```

Then open `http://localhost:8080/index.html` (or `http://127.0.0.1:5500/index.html` for VS Code Live Server's default port).

## Reference implementation files

- [index.html](../../index.html) — single-page markup; meta CSP at line 24; Font Awesome SRI at line 29–32.
- [index.css](../../index.css) — all styles; `prefers-reduced-motion` media block near the end.
- [index.js](../../index.js) — all client script; `WHATSAPP_NUMBER` at line 92; storage keys at lines 96 and 314; length caps at lines 97–101 and 315–320; validators at lines 106, 343–361.
- [CNAME](../../CNAME) — GitHub Pages custom domain (`krishidakshina.in`).

Do **not** modify these files as part of running this quickstart — the guide is read-only validation.

---

## Validation scenarios

Run these in order. Each step lists the acceptance criterion; a failing step is a regression against the referenced spec item.

### 1. Page loads, navbar animates, hero renders (US-6)

- **Do**: Load `http://localhost:8080/index.html` on a viewport ≥ 768 px.
- **Expect**: Desktop navigation is visible, hamburger is hidden. Scrolling triggers the "solid navbar" style transition. Hero copy fades in. Font Awesome icons render (globe, cart, etc.).
- **Fails if**: Any icon shows as a small square (Font Awesome CSS rejected — likely SRI mismatch or CSP violation).

### 2. All 12 product cards + `+` buttons work (US-1)

- **Do**: Scroll to the Products section. Click `+` on every card. Watch the navbar cart badge.
- **Expect**: Badge increments by 1 per click, reaching 12. Cart drawer opens on the cart button. Cards without a real `product_images/<slug>` file render their `data-fallback` emoji (v1 baseline — see [research.md R-9](./research.md#r-9-product-images)).
- **Fails if**: A broken-image icon appears (the delegated `error` listener didn't fire), or the badge doesn't update, or clicking `+` past qty 99 keeps incrementing.

### 3. Cart drawer edits + clear + persist (US-1, US-2)

- **Do**: In the drawer, click `+` and `-` on individual lines. Click "Clear cart". Then re-add 2 products and **reload the tab** (`Ctrl+R`).
- **Expect**: Per-line `+`/`-` adjust each line; hitting `-` at qty 1 removes the line. "Clear cart" empties the drawer and zeroes the badge. After reload, the 2 products come back exactly (persisted under `krishidakshina.cart.v1` — verify in DevTools → Application → Local Storage).
- **Fails if**: Reload loses the cart, the badge shows a different number than the sum of qtys, or a rehydrated cart contains fields other than `{ name, price, unit, qty }`.

### 4. Delivery form validation (US-3)

- **Do**: Open the drawer with items in it. Fill the delivery form with:
  - (a) Valid values (10-digit phone starting `6`–`9`, valid 6-digit pincode starting `1`–`9`, non-empty name / addr1 / city).
  - (b) Invalid phone (e.g., `1234567890`).
  - (c) Invalid pincode (e.g., `000000`).
- **Expect**: "Order via WhatsApp" is **enabled** only under (a). Cases (b) and (c) either disable the button or reject the click with an inline error identifying the field.
- **Fails if**: An invalid phone or pincode gets through, or the button remains disabled after all fields are valid, or the field values are lost after reload (they should persist under `krishidakshina.customer.v1`).

### 5. Pincode lookup (US-3)

- **Do**: Clear the City field. Type `400001` into the Pincode field and tab out.
- **Expect**: A `GET https://api.postalpincode.in/pincode/400001` fires (verify in DevTools → Network). On success the City field is auto-filled with the district (typically "Mumbai" for `400001`). If the API is unreachable, a non-blocking notice shows and City remains editable — the order flow is not blocked.
- **Fails if**: The request goes to any origin other than `api.postalpincode.in`, or the City field is force-overwritten while the user has typed in it, or a lookup failure blocks the order button.

### 6. Geolocation opt-in (US-3)

- **Do**: Click "Use my current location". Grant permission when the browser prompts.
- **Expect**: A hint like "Location captured" appears; the map pin is now attached to the eventual WhatsApp message. Denying permission surfaces a friendly message and does **not** block the order flow.
- **Fails if**: The geolocation prompt fires automatically on page load (Principle IV clause 9 violation), or the captured coordinates persist to `localStorage`.

### 7. WhatsApp handoff (US-4)

- **Do**: With a valid cart + valid delivery form, click "Order via WhatsApp".
- **Expect**: A new tab opens with a URL of the form `https://wa.me/919876543210?text=…`. Decode the `text=` param and verify the message begins with `🛒 *New Order – KrishiDakshina*`, includes each cart line with unit price and line total, includes the customer name/phone/address/pincode/city, and includes the grand total. If a geolocation pin was captured in step 6, a `https://maps.google.com/?q=<lat>,<lng>` link is present.
- **Fails if**: The URL points anywhere other than `wa.me`, the `_blank` window is opened without `noopener,noreferrer` (Principle IV clause 8), or the encoded text length exceeds 3800 chars without the inline error firing first.

### 8. Contact form + reduced motion (US-5, US-6)

- **Do**: Submit the contact form with any values. Then open DevTools → Rendering, toggle "Emulate CSS media feature: `prefers-reduced-motion: reduce`".
- **Expect**: The contact form shows a spinner briefly, then a success toast — **no** network request leaves the browser (verify in DevTools → Network). Under `prefers-reduced-motion: reduce`, the truck animation stops, the particle motion stops, and hover-motion effects are subdued.
- **Fails if**: The contact form fires any HTTP request (Principle V clause 6 violation), or animations continue under reduced-motion (Principle I violation).

---

## Spot-checks for the release gates (Principle VI — currently GAP)

These checks are what the future `.github/workflows/lighthouse.yml` will automate. Running them by hand now gives a numeric baseline the future CI can compare against.

- **CSP hygiene**: In DevTools → Elements, confirm the `<meta http-equiv="Content-Security-Policy">` at [index.html:24](../../index.html#L24) contains neither `'unsafe-inline'` nor `'unsafe-eval'` in `script-src` or `style-src`.
- **`Math.random(` scan**: From the repo root, run `Select-String -Path index.js -Pattern 'Math\.random\('` in PowerShell. **Expected count: 0.**
- **Bundle sizes** (approximate; final measurement uses gzip in CI):

  ```powershell
  Get-ChildItem index.css, index.js | Select-Object Name, Length
  ```

  Confirm `index.js` and `index.css` are within an order of magnitude of the Principle II budget (50 KB gz JS, 30 KB gz CSS). A hand-check with `gzip` locally is optional.

- **Lighthouse mobile audit** (Chrome DevTools → Lighthouse tab → Mobile, 4G Fast, Simulated throttling): record the four category scores. Principle VI requires each ≥ 90; this baseline plan explicitly notes the first measurement has not yet been run (see [plan.md Complexity Tracking](./plan.md#complexity-tracking)).

---

## What this quickstart does **not** cover

- **Deployment**: `git push` to the default branch triggers GitHub Pages auto-publish. There is no preview environment yet — adding one is part of the Principle VI gap.
- **Implementation code**: this document is a validation guide, not a build guide. Model/service/controller bodies, migrations, and full test suites do not exist in v1 (and are largely not applicable — see [data-model.md § Not applicable](./data-model.md#not-applicable-client-only-architecture)). Field-level details live in [data-model.md](./data-model.md); wire-level details live in [contracts/](./contracts/).
- **Business/policy questions**: OQ-01…OQ-05 from `spec.md` (brand vs. domain, placeholder contact details, product-image sourcing, testimonial permissions, contact-form backend). These are the subject of `/speckit.clarify`, not this validation.
