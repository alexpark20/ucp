# Shopify MCP: 2026-01-23 → draft implementation guide

> Two major clients are pinned to `2026-01-23` and are not strict schema-driven. The goal is to minimize version-conditional logic while ensuring neither client breaks.

## Column definitions

| Column | Meaning |
| --- | --- |
| **Spec Version** | UCP spec version that introduces this behavior (all entries here are `draft`) |
| **Shopify Impl Version** | Shopify endpoint(s) where this behavior should be active: `2026-01-23, draft` = both; `draft` = draft endpoint only; `—` = deferred |
| **Implemented** | Whether the Shopify implementation currently has this behavior |

---

## Changes

| # | Change | Spec Version | Shopify Impl Version | Implemented |
| --- | --- | --- | --- | --- |
| 2 | Version negotiation: 422 on version mismatch | draft | `draft` | ✅ |
| 4 | MCP results: `oneOf [resource, error_response]` | draft | `draft` | ✅ |
| 5 | `ucp.status: "error"` discriminator on 200 responses | draft | `2026-01-23, draft` | ✅ |
| 6 | New `unrecoverable` severity value | draft | `2026-01-23, draft` | ✅ |
| 7 | Profile `Cache-Control: public, max-age ≥ 60` | draft | `2026-01-23, draft` | ✅ |
| 10 | `adjustment.amount` minimum: 0 enforcement | draft | `—` | ✅ |
| 12 | New error codes (`out_of_stock`, `address_undeliverable`) | draft | `2026-01-23, draft` | ✅ |
| 13 | Cart-to-checkout: discount codes forwarded explicitly | draft | `2026-01-23, draft` | ✅ |
| 16 | Signing headers (`meta.signature`, `meta.idempotency-key`, `meta.ucp-agent`) | draft | `2026-01-23, draft` | ✅ |
| 17 | Eligibility claims & `invalid_eligibility` blocking error | draft | `2026-01-23, draft` | ✅ |
| 18 | `signals` field on cart/checkout | draft | `2026-01-23, draft` | ✅ |
| 19 | Fulfillment method `id`/`type` optional on update | draft | `2026-01-23, draft` | ✅ |
| 20 | `intent` field in context | draft | `2026-01-23, draft` | ✅ |

---

## Notes

### #2 — Version negotiation: 422 on version mismatch

The `draft` spec changes version mismatch responses from `HTTP 200 + body error` to `HTTP 422`. This only affects the `draft` endpoint. The two major clients always talk to the `2026-01-23` endpoint and always send `"version": "2026-01-23"` — they never trigger a mismatch. The `2026-01-23` endpoint keeps the `200 + body error` behavior unchanged.

### #4 — MCP results: `oneOf [resource, error_response]`

The `draft` spec allows MCP method results to be either a resource object or a bare `error_response` object (discriminated union). Returning a bare `error_response` on the `2026-01-23` endpoint would crash clients that assume the result is always the resource type. The `2026-01-23` endpoint must always return a resource-shaped result; errors are expressed via `ucp.status: "error"` within that envelope (see change #5).

### #10 — `adjustment.amount` minimum: 0 enforcement

The `draft` spec adds `minimum: 0` to the `amount` type. This is deferred on both endpoints — clients may be using negative values to represent credit adjustments, and enforcing this constraint without confirming client behavior first would silently break them. Revisit once adjustment usage across both clients is audited.

### #16 — Signing headers

All signing fields (`meta.signature`, `meta.idempotency-key`, `meta.ucp-agent`) are optional in the spec. The implementation should accept them if present and not reject requests that omit them. No active signing verification or emission is required — just pass-through tolerance for both endpoints.
