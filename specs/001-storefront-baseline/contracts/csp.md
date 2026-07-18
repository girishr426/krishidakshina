# Contract: Content Security Policy

**Feature**: `001-storefront-baseline` · **Owner**: [index.html](../../../index.html) meta tag at line 24 · **Principles**: P-IV (Security-by-Default) clauses 1–3, P-V (Client-Only Architecture) clause 5

## Verbatim CSP (v1)

The page ships this policy as a `<meta http-equiv="Content-Security-Policy">` — there is no HTTP-header equivalent because GitHub Pages does not let us set arbitrary response headers. This is the single source of truth; the constitution's Additional Constraints allow-list MUST match this string.

```
default-src 'self';
style-src 'self' https://cdnjs.cloudflare.com https://fonts.googleapis.com;
font-src 'self' https://cdnjs.cloudflare.com https://fonts.gstatic.com;
img-src 'self' data: https:;
script-src 'self';
connect-src 'self' https://api.postalpincode.in;
frame-ancestors 'none';
base-uri 'self';
form-action 'self';
object-src 'none'
```

Additional referrer policy (separate `<meta name="referrer">` tag): `strict-origin-when-cross-origin`.

## Directive-by-directive rationale

| Directive | Value | Why this value |
|-----------|-------|----------------|
| `default-src` | `'self'` | Deny-by-default. Every other directive is either a same-origin exception or an explicit allow-list. |
| `style-src` | `'self'` + `cdnjs.cloudflare.com` + `fonts.googleapis.com` | `index.css` from same origin; Font Awesome CSS from `cdnjs` (SRI-pinned); Inter font CSS from Google Fonts. **No `'unsafe-inline'`** — hence zero inline styles anywhere in the codebase. |
| `font-src` | `'self'` + `cdnjs.cloudflare.com` + `fonts.gstatic.com` | Font-family binary URLs for Font Awesome and Inter. |
| `img-src` | `'self'` + `data:` + `https:` | Local product images, inline SVG data-URIs, and any `https:` avatar/testimonial photo. Deliberately broad on the `https:` scheme because product/marketing imagery may be sourced from CDNs later — but see amendment rule (2) below. |
| `script-src` | `'self'` | Only `index.js` from same origin. **No `'unsafe-inline'`, no `'unsafe-eval'`, no external script CDNs, no analytics beacons.** |
| `connect-src` | `'self'` + `api.postalpincode.in` | `fetch()` targets are limited to same-origin (currently unused) and the pincode lookup. |
| `frame-ancestors` | `'none'` | Blocks clickjacking via `<iframe>` embedding. |
| `base-uri` | `'self'` | Prevents `<base href>` hijack changing the resolution of relative URLs. |
| `form-action` | `'self'` | The visible contact form is a client-side simulation ([research.md R-8](../research.md#r-8-observability--error-handling)); any actual `<form action>` would have to hit same-origin. |
| `object-src` | `'none'` | Legacy plugin surface completely disabled. |

## Companion runtime rules (already in the codebase)

- **Every `<link rel="stylesheet">` that points off-origin carries `integrity="sha384-…"`, `crossorigin="anonymous"`, and (where practical) `referrerpolicy="no-referrer"`.** The Font Awesome tag at [index.html:29-32](../../../index.html#L29) is the reference.
- **Every `<a target="_blank">` and every `window.open` call carries `rel="noopener noreferrer"` / the equivalent options string.**
- **Every external navigation destination (`wa.me`, `maps.google.com`) is opened via `window.open(url, '_blank', 'noopener,noreferrer')`**, never via a bare `location = …`.
- **Referrer-Policy meta**: `strict-origin-when-cross-origin`. Prevents leaking full URL paths to third-party origins on outbound requests.

## Amendment procedure

Any change to this CSP is treated as constitution-relevant. To add a new origin or relax a directive:

1. **Justify the origin** in the plan under Complexity Tracking, naming the feature it unlocks and the alternative that was rejected.
2. **Update this file** (`contracts/csp.md`) with the new verbatim string.
3. **Update [index.html](../../../index.html)** to match.
4. **Update the constitution** (`.specify/memory/constitution.md`) — the runtime allow-list under Additional Constraints & Standards must match. This is a **MINOR** amendment if it introduces a new dependency category, **MAJOR** if it violates Principle V (e.g., analytics, tracking pixels).
5. **If the new origin serves executable content (scripts, iframes with script)**, an SRI hash is mandatory for the resource, and the `crossorigin="anonymous"` + `referrerpolicy="no-referrer"` attributes must be added.
6. **CI CSP-hygiene gate** (once wired) MUST continue to pass: the served CSP must not contain `'unsafe-inline'` or `'unsafe-eval'` in `script-src` or `style-src`.

## Explicit denials (do not add without a MAJOR amendment)

The following are **banned** by construction in v1 and appear here so a future maintainer can `Ctrl+F` before proposing them:

- `'unsafe-inline'` in `script-src` or `style-src` — Principle IV clause 1 & 2.
- `'unsafe-eval'` in `script-src` — same.
- Analytics origins (`www.googletagmanager.com`, `plausible.io`, `stats.g.doubleclick.net`, …) — Principle V clause 2.
- Any tracking pixel origin (`facebook.com`, `pixel.…`, `bat.bing.com`) — Principle V clause 2.
- Any origin that would require a service worker — Principle V clause 2.
- Any `worker-src` beyond `'self'` — no service worker in v1.
- `frame-src` — no third-party embeds in v1; if one is ever needed, it MUST be justified and paired with `frame-ancestors 'none'` staying intact.
