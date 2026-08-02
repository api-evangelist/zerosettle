---
name: create-web-checkout
description: Fetch a ZeroSettle product catalog and open a web checkout (payment intent or hosted checkout session) for a purchase.
api: zerosettle
operations:
- getProducts
- createPaymentIntent
- createCheckoutSession
- getTransaction
auth: X-ZeroSettle-Key (zs_pk_test_* in sandbox)
base_url: https://api.zerosettle.io/v1
---

# Create a ZeroSettle Web Checkout

Route an eligible purchase through ZeroSettle web checkout instead of App
Store / Play billing. Prefer the SDKs for real apps; use the raw API for
server-driven or headless flows.

## Steps

1. Call `getProducts` — `GET /iap/products` (optionally with `user_id` to get
   trial eligibility and offer facts) to fetch the catalog. Each product carries
   `web_price[]` and `storekit_price[]`.
2. Create the payment surface:
   - `createPaymentIntent` — `POST /iap/payment-intents` for an embedded
     (Apple Pay / card) sheet. Returns `client_secret`, `payment_intent_id`,
     and `transaction_id`.
   - or `createCheckoutSession` — `POST /iap/checkout-sessions` for a hosted
     checkout URL. Returns `checkout_session_id`, `url`, `transaction_id`.
3. Complete payment client-side (Stripe confirm with the `client_secret`, or
   redirect to the hosted `url`).
4. Confirm the result with `getTransaction` —
   `GET /iap/transactions/{transaction_id}` — and grant access only when its
   `status` is complete.

## Rules

- Sandbox vs live is chosen entirely by the key prefix (`zs_pk_test_` /
  `zs_pk_live_`). Test with Stripe cards (`4242 4242 4242 4242` = success).
- If web checkout is disabled for the buyer's jurisdiction, the API/SDK reports
  `webCheckoutDisabledForJurisdiction` — fall back gracefully, do not error out.
- Idempotency is handled server-side; there is no consumer Idempotency-Key
  header. Do not double-charge by re-creating intents on retry — re-read the
  transaction first.
