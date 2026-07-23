---
name: Create and track a Balad payout
description: Authenticate, create a cross-border payout transaction into Egypt, and track it to a terminal status via polling or webhook.
api: openapi/balad-corp-gateway-openapi.yml
operations:
  - POST /identity/v1/token
  - POST /remit-api/v1/transactions
  - GET /remit-api/v2/transactions/{remitter_reference_no}/status
  - POST /remit-api/v1/transactions/cancel
generated: '2026-07-18'
method: generated
---

# Create and track a Balad payout

Use this skill to send money into Egypt through Balad Link and follow it to a terminal state.

## Auth
1. `POST /identity/v1/token` with `grant_type=client_credentials`, `client_id`, `client_secret`
   as `application/x-www-form-urlencoded`. Use the returned `access_token` as
   `Authorization: Bearer <token>` (valid ~3600s) on every call.

## Create the transaction
2. `POST /remit-api/v1/transactions` with sender/receiver details, `sending_currency_code`
   (USD or EGP only), `amount`, `receiver_bank_code` or wallet/cash target, `purpose`,
   `source_of_remittance`, and a **unique** `remitter_reference_no`.
   - `remitter_reference_no` is your idempotency key: reusing one returns error `129`
     (Duplicate reference). Generate a fresh unique value per intended transaction.
   - Amount bounds: min 1 USD (error `170`), route/currency mismatch is `185`, unsupported
     currency is `8057`, per-wallet cap is `187`.

## Track to terminal status
3. Poll `GET /remit-api/v2/transactions/{remitter_reference_no}/status`, OR (preferred)
   subscribe to the `Link.Transaction.StatusUpdateV2` webhook (event_type `994019`) and verify
   the `X-HMAC-SIGNATURE` (HMAC-SHA256 of the minified body with your 32-char secret).
4. Terminal statuses: `1` Transferred (success), `9` Failed, `13`/`15` rejected,
   `4` Returned, `7` Canceled. On failure, read `failureDetails.code`
   (e.g. `CMP_001` AML, `ACC_004` invalid account, `BEN_BNK_001` bank unavailable).

## Cancel (only while cancellable)
5. `POST /remit-api/v1/transactions/cancel` — allowed only in Created/Pending; `113` means the
   transaction already completed, `169` means already canceled.

Grounded in errors/balad-corp-error-codes.yml, errors/balad-corp-decline-codes.yml,
conventions/balad-corp-conventions.yml.
