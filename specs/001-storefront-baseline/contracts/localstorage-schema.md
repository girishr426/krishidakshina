# Contract: `localStorage` Schema

**Feature**: `001-storefront-baseline` · **Owner**: [index.js](../../../index.js) validators at lines 106, 353 · **Principles**: P-IV (Security-by-Default) clauses 4–6, P-V (Client-Only Architecture) clause 3

The client keeps exactly two `localStorage` records. Both use versioned keys per Principle V clause 3 so a future schema change bumps the `.vN` suffix and preserves back-compat.

## Keys (v1)

| Key | Purpose | Reference validator | Size ceiling |
|-----|---------|---------------------|--------------|
| `krishidakshina.cart.v1` | Cart lines keyed by product identifier | `isValidCartItem` at [index.js:106](../../../index.js#L106) | 50 keys × ~250 chars ≈ 12 KB |
| `krishidakshina.customer.v1` | Delivery-form values | `isValidCustomer` at [index.js:353](../../../index.js#L353) | ~1.5 KB (sum of per-field caps) |

Both records are stringified JSON. No other keys under the `krishidakshina.` prefix are read or written in v1.

---

## `krishidakshina.cart.v1`

### Shape

```json
{
  "<productKey>": {
    "name":  "<string, non-empty, ≤ 200 chars>",
    "price": <number, finite, 0 .. 1_000_000>,
    "unit":  "<string, ≤ 50 chars>",
    "qty":   <integer, 1 .. 99>
  }
}
```

- `<productKey>` is a stable per-product identifier chosen by the render pass in `index.html` (in v1: the product's `name`). Keys are validated on hydrate: non-string keys or keys longer than 100 chars cause the entry to be dropped.
- Aggregate cap: `Object.keys(cart).length ≤ 50` (`MAX_CART_ITEMS`).

### Hydrate algorithm (prototype-pollution-safe)

Reproduced from [index.js](../../../index.js) around lines 120–130:

```js
const parsed = JSON.parse(raw);
const cart = {};                          // fresh plain object — never Object.assign onto untrusted
let count = 0;
for (const k of Object.keys(parsed)) {    // whitelist iteration only
  if (count >= MAX_CART_ITEMS) break;
  if (typeof k === 'string' && k.length > 0 && k.length <= 100
      && isValidCartItem(parsed[k])) {
    cart[k] = {
      name:  parsed[k].name,              // per-field copy — no spread of untrusted keys
      price: parsed[k].price,
      unit:  parsed[k].unit,
      qty:   parsed[k].qty
    };
    count++;
  }
}
```

Any of the following causes the offending entry to be silently dropped (the app never crashes):

- The parsed value is not an object.
- A key is not a string, is empty, or exceeds 100 chars.
- Any field is missing, has the wrong type, is out of range, or violates its length cap.
- `qty` is not a positive integer ≤ 99.
- Adding this entry would push `count` past `MAX_CART_ITEMS`.

### Write algorithm

Every mutation calls `JSON.stringify(cart)` and writes to `localStorage.setItem('krishidakshina.cart.v1', …)`. Failures (`QuotaExceededError`, private-mode denials) are caught and `console.warn`'d; the UI proceeds ([research.md R-8](../research.md#r-8-observability--error-handling)).

### Migration policy

- If any field's shape changes (e.g., adding a `sku` field or changing `qty` to a float), the key MUST be bumped to `krishidakshina.cart.v2` and the v1 key MUST be either migrated on read or discarded with a `console.warn` and a fresh-start UX.
- Reintroducing analytics or per-item timestamps is a Principle V concern before it is a schema concern.
- **2026-07-18 — one-time namespace reset**: the storage namespace was renamed from `gutpoint.*` to `krishidakshina.*` as part of constitution v2.1.0 (canonical brand resolution). This is a **pure namespace rename, not a schema change**: the value shape is unchanged and the `.v1` suffix is preserved. Any browser that previously held data under `gutpoint.cart.v1` / `gutpoint.customer.v1` will present an empty cart under the new namespace on the next visit; the old keys are simply orphaned (they may be manually cleared from DevTools → Application → Local Storage, but no code path reads them). Pre-launch reset was deemed acceptable; no migration reader was written.

---

## `krishidakshina.customer.v1`

### Shape

```json
{
  "name":    "<string, non-empty, ≤ 100 chars>",
  "phone":   "<string, ≤ 20 chars raw; matches /^[6-9]\\d{9}$/ after normalizePhone at submit>",
  "addr1":   "<string, non-empty, ≤ 200 chars>",
  "addr2":   "<string, ≤ 200 chars; may be empty>",
  "city":    "<string, non-empty, ≤ 100 chars>",
  "pincode": "<string, matches /^[1-9]\\d{5}$/ at submit>",
  "notes":   "<string, ≤ 500 chars; may be empty>"
}
```

- The record has exactly these seven keys. Extra keys in a stored blob are ignored during hydrate; missing keys cause the entire record to be dropped.
- The `phone` and `pincode` **format regexes are enforced at submit time, not at hydrate time**, so a partially-filled form can rehydrate without triggering a validation failure.

### Hydrate algorithm

Reproduced from [index.js](../../../index.js) around lines 365–380:

```js
const parsed = JSON.parse(raw);
if (isValidCustomer(parsed)) {
  dName.value    = parsed.name    || '';
  dPhone.value   = parsed.phone   || '';
  dAddr1.value   = parsed.addr1   || '';
  dAddr2.value   = parsed.addr2   || '';
  dCity.value    = parsed.city    || '';
  dPincode.value = parsed.pincode || '';
  dNotes.value   = parsed.notes   || '';
}
```

`isValidCustomer` verifies:

- `parsed` is an object (not `null`, not an array).
- Every field is a string.
- Every string satisfies its per-field length cap via `isStrLen`.

If the check fails, no field is populated and the form starts empty. This is the "clean-start" behaviour required by Principle IV clause 5.

### Write algorithm

Every form change slices each input value to its per-field cap **before** JSON-stringifying, so the persisted blob is bounded even if a caller (or a browser autofill) sends a longer string:

```js
{
  name:  dName.value.slice(0, MAX_CNAME_LEN),   // 100
  phone: dPhone.value.slice(0, MAX_PHONE_LEN),  // 20
  addr1: dAddr1.value.slice(0, MAX_ADDR_LEN),   // 200
  addr2: dAddr2.value.slice(0, MAX_ADDR_LEN),   // 200
  city:  dCity.value.slice(0, MAX_CITY_LEN),    // 100
  pincode: /* raw text */,
  notes: dNotes.value.slice(0, MAX_NOTES_LEN)   // 500
}
```

The result is written to `localStorage.setItem('krishidakshina.customer.v1', …)`. Failures are `console.warn`'d, never surfaced.

### Migration policy

- Adding a required field (e.g., `state`) is a MINOR schema change and requires bumping to `krishidakshina.customer.v2` with a hydrate-and-migrate path.
- Removing a field is a MAJOR schema change (existing records need a rewrite path or an explicit discard).

---

## What is **not** stored

- **Cart timestamps, session IDs, correlation IDs.** Not persisted — would enable cross-visit tracking, which Principle V clause 2 forbids.
- **Geolocation** (`geoLoc = { lat, lng }`). In-memory only; see [data-model.md § GeoLoc](../data-model.md#entity-geoloc-in-memory-only).
- **Order history.** No `krishidakshina.orders.v*` key exists. Once the WhatsApp handoff opens, the site is out of the loop — the order lives in the WhatsApp thread between the visitor and the business (spec NG-03).
- **User identity of any kind.** No accounts, no auth tokens, no OAuth state (spec NG-02).

## Cross-references

- Constitution Principle IV clauses 5–6 — this contract is one of the reference implementations named there.
- Constitution Principle V clause 3 — this contract defines what "versioned localStorage keys" means for v1.
- [data-model.md § CartItem](../data-model.md#entity-cartitem) and [§ Customer](../data-model.md#entity-customer) — field-level docs.
