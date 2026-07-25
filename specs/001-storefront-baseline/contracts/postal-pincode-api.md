# Contract: Postal Pincode API (`api.postalpincode.in`)

**Feature**: `001-storefront-baseline` · **Owner**: [index.js](../../../index.js) around line 430 · **Principles**: P-V (Client-Only Architecture) clause 5, P-IV (Security-by-Default) clauses 4–6

The only third-party HTTP endpoint the site is permitted to `fetch()` in v1 (per constitution Additional Constraints and the CSP `connect-src`).

## When it is called

The lookup fires **only** when **all** of the following hold, per the logic around [index.js:430](../../../index.js#L430):

1. The visitor has typed a value into the Pincode field and moved focus away (or the value has stabilised via debounce).
2. `isValidPincode(v)` returns true — i.e., the value matches `/^[1-9]\d{5}$/`.
3. The City field is currently empty (the lookup never overwrites a value the visitor typed).
4. Any in-flight request is cancelled first via `AbortController.abort()` — only the latest lookup wins.

It is **never** called on page load, and it is **never** called from anywhere outside the delivery form.

## Request

- **Method**: `GET`
- **URL**: `https://api.postalpincode.in/pincode/<pincode>`
- **Path parameter**: `<pincode>` — a 6-digit string that already passed `isValidPincode`.
- **Headers sent**: only the browser defaults. No custom headers. No `Authorization`, no `Cookie` (the site does not set cookies).
- **Body**: none.
- **Referrer**: constrained by the page's `Referrer-Policy: strict-origin-when-cross-origin` — the third party only sees `https://krishidakshina.in`, not the full URL.
- **Cancellation**: an `AbortController` is created per pincode edit and its `.signal` is passed into `fetch`. Superseded lookups are aborted.

Example: `GET https://api.postalpincode.in/pincode/400001`

## Response — success shape (contract we consume)

The upstream returns a JSON array with one element. The fields we rely on are:

```json
[
  {
    "Status": "Success",
    "PostOffice": [
      {
        "District": "Mumbai",
        "State":    "Maharashtra",
        "…":        "other fields are ignored"
      }
    ]
  }
]
```

### Consumption rules

- Read `body[0].Status === "Success"`. Any other value (including missing) is treated as a lookup failure.
- Read `body[0].PostOffice[0].District` — this is what fills the City field.
- Apply `.slice(0, MAX_CITY_LEN)` before assignment (`MAX_CITY_LEN = 100`). See [index.js:443](../../../index.js#L443).
- **Do not** trust or persist any other field, and **do not** iterate the full `PostOffice` array — only the first entry is used, and only for the district.
- **Do not** use `innerHTML` / template literals against the response — the value goes into an `<input>` via `.value =` (which does not evaluate HTML).

## Response — failure shapes we handle

The upstream may return, and this contract MUST tolerate:

- **`Status: "Error"`** — e.g., unknown pincode. The site emits a non-blocking notice; the City field stays editable.
- **HTTP non-2xx** — the fetch's `response.ok` is false. Same non-blocking notice.
- **Network / DNS failure** — the fetch rejects. Same non-blocking notice.
- **Timeout / abort** — the `AbortController` fires. No user-facing message (this is a normal supersede).
- **Unexpected shape** — `body[0]` missing, `PostOffice` not an array, `District` not a string. Same non-blocking notice.

**In every failure mode, the order flow is unaffected** — the visitor can type a city manually and place the order. This is required by spec US-3 Acceptance Scenario 3 and by spec FR-019.

## What this contract does **not** allow

- **Sending any payload to the API.** It is a public read-only endpoint; we only issue GETs.
- **Retrying automatically.** A failed lookup does not retry; the visitor either edits the pincode or types the city.
- **Caching results in `localStorage`.** Results are used once and discarded — no persisted geographic data.
- **Any other endpoint on the same origin.** Only `/pincode/{pincode}` is called. Adding a second path from this origin is fine (same `connect-src` entry) but MUST be documented here.
- **Any other origin.** Adding a different pincode / geo provider is a CSP amendment + a constitution allow-list amendment ([contracts/csp.md § Amendment procedure](./csp.md#amendment-procedure)).

## Amendment procedure

To change the endpoint, the response fields we consume, or the failure handling:

1. Update this contract file.
2. Update the code around [index.js:430](../../../index.js#L430).
3. If the origin changes, update [contracts/csp.md](./csp.md), [index.html](../../../index.html)'s meta CSP, and the constitution's runtime allow-list.
4. Update spec FR-019 and US-3 Acceptance Scenario 2/3.
5. Confirm the lookup remains **non-blocking for the order flow** — this is a Principle V property that any amendment must preserve.
