---
name: manage-subscription-retention
description: Run a ZeroSettle cancel flow (retention questionnaire + save offer) and cancel, pause, or resume a subscription.
api: zerosettle
operations:
- getCancelFlow
- submitCancelFlowResponse
- acceptCancelFlowOffer
- cancelSubscription
- pauseSubscription
- resumeSubscription
auth: X-ZeroSettle-Key
base_url: https://api.zerosettle.io/v1
---

# Manage a ZeroSettle Subscription (Retention + Lifecycle)

Build a headless cancellation experience: present the dashboard-configured
retention questionnaire and save offer before actually cancelling.

## Steps

1. Call `getCancelFlow` — `GET /iap/cancel-flow` to load the configured
   questionnaire (`CancelFlowQuestion[]`) and any retention `CancelFlowOffer`.
2. Record the user's answers with `submitCancelFlowResponse` —
   `POST /iap/cancel-flow/respond`.
3. If the user takes the save offer, call `acceptCancelFlowOffer` —
   `POST /iap/cancel-flow/accept-offer` and stop — the subscription stays active.
4. Otherwise complete the lifecycle action:
   - `cancelSubscription` — `POST /iap/subscriptions/cancel` (access remains
     until the end of the current billing period).
   - `pauseSubscription` — `POST /iap/subscriptions/pause`.
   - `resumeSubscription` — `POST /iap/subscriptions/resume`.

## Rules

- Always present the cancel flow before `cancelSubscription`; skipping it loses
  the configured retention offer.
- On cancel, the entitlement stays `is_active: true` until `expires_at`; do not
  revoke access immediately.
- A `404` from a lifecycle call means no active subscription for that product —
  verify with the Entitlements API first (see verify-entitlement skill).
