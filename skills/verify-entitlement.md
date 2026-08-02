---
name: verify-entitlement
description: Verify a ZeroSettle user's purchase/subscription status server-side before granting access to gated content or API features.
api: zerosettle
operations:
- getEntitlements
- getTransaction
auth: X-ZeroSettle-Key (zs_pk_live_* in production)
base_url: https://api.zerosettle.io/v1
---

# Verify a ZeroSettle Entitlement (Server-Side)

Use this when your backend must confirm a user actually owns an active purchase
or subscription before unlocking content. ZeroSettle is Merchant of Record, so
you never handle raw payment webhooks — you poll the Entitlements API.

## Steps

1. Call `getEntitlements` — `GET /iap/entitlements` with query `user_id=<your user id>`
   (or `email=<address>` for anonymous purchasers). Send header
   `X-ZeroSettle-Key: zs_pk_live_...`.
2. Read the `entitlements[]` array. An entitlement grants access when
   `is_active` is `true` and its `product_id` matches the product gating the
   feature. Check `expires_at` / `will_renew` for subscription state.
3. If you need the underlying payment record (audit, support), call
   `getTransaction` — `GET /iap/transactions/{transaction_id}` using the
   entitlement `id`.
4. Deny access (e.g. HTTP 403) when no matching active entitlement is found.

## Rules

- Never trust client-reported entitlement state for billing-critical gates —
  always re-verify server-side with `getEntitlements`.
- On a `502` (upstream Stripe/Apple), retry with backoff; do not revoke access
  on a transient upstream error.
- Errors return `{ "error": ..., "code": ... }` — not RFC 9457 problem+json.
