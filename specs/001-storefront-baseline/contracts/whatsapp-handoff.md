# Contract: WhatsApp Order Handoff (`wa.me`)

**Feature**: `001-storefront-baseline` · **Owner**: [index.js](../../../index.js) around lines 92, 535, 541 · **Principles**: P-V (Client-Only Architecture) clause 4, P-IV (Security-by-Default) clauses 6 & 8

The **sole** order-placement path in v1. The site never itself transmits an order; it composes a plain-text message and hands the visitor to WhatsApp via a `wa.me` link. Any change to this contract that would transmit order data through the site's own infrastructure is a MAJOR constitution amendment.

## Destination

- **Base URL**: `https://wa.me/${WHATSAPP_NUMBER}?text=${encoded}`
- **`WHATSAPP_NUMBER`**: string constant declared at [index.js:92](../../../index.js#L92) — currently `"919876543210"`.
- **Open mechanism**: `window.open(url, '_blank', 'noopener,noreferrer')` — the `noopener,noreferrer` string is mandatory per Principle IV clause 8.
- **`wa.me` in the CSP**: not a `connect-src` origin. It is a navigation target only; the browser opens it directly and the site never `fetch()`es it.

## Trigger

The "Order via WhatsApp" button is enabled **only** when all of the following hold:

1. `Object.keys(cart).length >= 1`.
2. Every required delivery field passes its validator: `isNonEmpty` on name / addr1 / city, `isValidPhone` on phone, `isValidPincode` on pincode. See [index.js:405-410](../../../index.js#L405).
3. The composed message's encoded length is `≤ MAX_MSG_LEN` (3800). If the message is oversize, the click is rejected inline and the tab is NOT opened. See [index.js:535](../../../index.js#L535).

If any check fails, no navigation happens and the failing field(s) are surfaced in the drawer.

## Message layout (plain-text)

The message is built by string concatenation over validated state — no template engine, no `innerHTML`, no dynamic evaluation. Structure:

```
🛒 *New Order – KrishiDakshina*

*Customer:*
Name: <customer.name>
Phone: <customer.phone>

*Delivery:*
<customer.addr1>
[<customer.addr2>]
<customer.city> - <customer.pincode>
[Notes: <customer.notes>]
[Location: https://maps.google.com/?q=<geoLoc.lat>,<geoLoc.lng>]

*Items:*
- <name> × <qty> @ ₹<price> <unit> = ₹<qty*price>
- …

*Total: ₹<grand_total>*
```

- Bracketed lines are included only when the value is present.
- `<price>`, `<qty*price>`, and `<grand_total>` are rendered as plain integers or two-decimal numbers depending on the source `price`. Currency symbol is a plain `₹` character; the site never sends HTML entities.
- Line separators are `\n`; WhatsApp preserves them.

## Encoding

```js
const encoded = encodeURIComponent(message);
if (encoded.length > MAX_MSG_LEN) {
  // surface inline error; DO NOT navigate
  return;
}
window.open(
  `https://wa.me/${WHATSAPP_NUMBER}?text=${encoded}`,
  '_blank',
  'noopener,noreferrer'
);
```

- **`encodeURIComponent` only.** Not `encodeURI`, not manual replacement. The message may legally contain any Unicode character allowed by `URLSearchParams` — the encoder handles it.
- **Length cap is measured on the encoded string**, not the raw message, because that is what actually travels in the URL. The 3800-char cap gives headroom against `wa.me`'s roughly-4-KB effective limit.

## Security posture

- **XSS surface = zero.** The message is a plain-text string; it never enters the DOM as HTML.
- **Injection surface = zero.** Every substituted value comes from the strict validators (`isValidCartItem`, `isValidCustomer`, `isValidPhone`, `isValidPincode`) with per-field length caps. Even if a user pastes a raw script tag into `notes`, it lands in the encoded query string as inert text.
- **Referrer leak = bounded.** `Referrer-Policy: strict-origin-when-cross-origin` means `wa.me` sees only `https://krishidakshina.in`, not the query-string body.
- **Tabnabbing = blocked.** `noopener,noreferrer` prevents the opened window from touching `window.opener`.

## Failure modes

| Situation | Outcome |
|-----------|---------|
| Empty cart | Button disabled; no navigation. |
| Any required field missing / invalid | Button disabled; inline error identifies the field. |
| Encoded message > 3800 chars | Click intercepted; inline error suggests reducing cart or shortening notes; **no** navigation. |
| Popup blocker denies `window.open` | The user sees the browser's popup-blocker notice. Nothing to fix on our side. |
| `WHATSAPP_NUMBER` misconfigured | The visitor lands on a `wa.me` chat that does not exist / no reply. This is a governance issue (Sync Impact Report `CONTACT_DETAILS_CENTRALIZATION`), not a code path. |

## What is **not** part of this contract

- **Order confirmation / receipt** — the site does not know whether the message was actually sent; the visitor's WhatsApp client is authoritative from here on.
- **Order storage** — nothing is persisted about a placed order. There is no `krishidakshina.orders.v*` key (see [localstorage-schema.md § What is not stored](./localstorage-schema.md#what-is-not-stored)).
- **Payment collection** — spec NG-01. Payment is negotiated on the WhatsApp thread between the visitor and the business.
- **Delivery tracking** — spec NG-06 / NG-03. Out of scope for v1.
- **Analytics / conversion events** — Principle V clause 2. Not allowed.

## Amendment procedure

Any change that…

1. Points the handoff at a different origin (e.g., Signal, Telegram, an in-house API), **or**
2. Adds a same-origin `fetch()` in the click path (e.g., "record the order in our backend first"), **or**
3. Removes the client-side simulation nature of the flow (e.g., stores the order in a database) …

…is a **MAJOR** constitution amendment (Principle V clause 4). Every other change (message layout tweaks, adding an emoji, adjusting the line format) is a normal spec change and re-runs `/speckit.plan`. If `MAX_MSG_LEN` is raised or lowered, the constitution's Principle IV clause 6 reference constants MUST be updated to match.
