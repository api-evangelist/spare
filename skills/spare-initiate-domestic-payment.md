---
name: Initiate a domestic payment (PIS)
description: Authenticate, create a domestic payment consent, and track the payment via Spare's Payment Initiation API.
api: openapi/spare-bahrain-pis-openapi-original.json
tenant_note: Payment types vary by jurisdiction; confirm availability with Spare. Bahrain host shown.
operations:
  - GET /api/v1.0/authentication/Token
  - GET /api/v1.0/pis/Provider/List
  - POST /api/v1.0/pis/Consent/Create
  - GET /api/v1.0/pis/Consent/Get
  - POST /api/v1.0/pis/Payment/List
  - GET /api/v1.0/pis/Payment/Get
---

# Initiate a domestic payment (PIS)

Use this skill to initiate a domestic Open Banking payment through Spare's
Payment Initiation API.

## Prerequisites
- Spare Dashboard API keys and the correct tenant host (Bahrain `https://ob.tryspare.com`, sandbox `https://sandbox.tryspare.com`).
- A payment scheme (`IBAN` or `BBAN`) and purpose (e.g. `BIL`, `PERS`) — see the `PaymentPurpose` / `SchemeName` enums.

## Steps
1. **Get a token** — `GET /api/v1.0/authentication/Token`; use `Authorization: Bearer <JWT>` thereafter.
2. **List providers** — `GET /api/v1.0/pis/Provider/List` to select the debtor's bank.
3. **Create a domestic payment consent** — `POST /api/v1.0/pis/Consent/Create` with the amount, creditor account (IBAN/BBAN), purpose and payment type (`SingleInstantPayment`). The response returns an authorisation link (redirection flow `WEB` or `DECOUPLED`).
4. **Have the payer authorise** — send the end user to the authorisation link to approve in their bank app.
5. **Confirm consent** — `GET /api/v1.0/pis/Consent/Get` (or handle the webhook) until authorised; rejection returns `CONSENT_REJECTED`.
6. **Track the payment** — `POST /api/v1.0/pis/Payment/List` and `GET /api/v1.0/pis/Payment/Get` to read `PaymentTransactionStatus`.

## Rules
- International payments are documented as "not yet available" — do not attempt them.
- Handle `PAYMENT_NOT_FOUND`, `PAYMENT_REQUEST_NOT_FOUND`, `CREDITOR_ACCOUNT_NOT_FOUND`, `DEBTOR_ACCOUNT_NOT_FOUND` (see `errors/spare-error-codes.yml`).
- Verify inbound webhook `x-signature` against the tenant JWKS (see `asyncapi/spare-webhooks.yml`).
