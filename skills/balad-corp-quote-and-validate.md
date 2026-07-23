---
name: Quote FX and validate a beneficiary before sending
description: Fetch exchange rates, calculate the receiving amount, list valid banks, and validate the beneficiary account before creating a Balad payout.
api: openapi/balad-corp-gateway-openapi.yml
operations:
  - POST /identity/v1/token
  - GET /core/services/{serviceId}/exchange-rates
  - POST /remit-api/v1/transactions/calculate-receiving-amount
  - GET /remit-api/banks
  - POST /remit-api/v1/transactions/validate-account
generated: '2026-07-18'
method: generated
---

# Quote FX and validate a beneficiary before sending

Run this pre-flight before `createTransaction` to avoid validation declines.

## Auth
1. `POST /identity/v1/token` (client_credentials) and use the Bearer token on all calls.

## Quote
2. `GET /core/services/{serviceId}/exchange-rates` for the current partner rate.
3. `POST /remit-api/v1/transactions/calculate-receiving-amount` with the sending amount
   (and optional transaction type) to get the receiving amount(s) in local currency.

## Resolve the payout target
4. `GET /remit-api/banks` filtered by transaction type and receiving country to get the valid
   `receiver_bank_code` set (38 CBE-licensed banks). Also use the lookups
   (`/remit-api/v1/lookups/purposes`, `/sources`, `/relationships`) to populate required codes.

## Validate the beneficiary
5. `POST /remit-api/v1/transactions/validate-account` to validate the account number / IBAN
   against currency (EGP vs USD), bank code, and length rules **before** sending. This prevents
   `172` (invalid account for bank code) and `ACC_004` decline downstream.

Now proceed to the "Create and track a Balad payout" skill. Currencies are USD or EGP only.
Grounded in conventions/balad-corp-conventions.yml and errors/balad-corp-error-codes.yml.
