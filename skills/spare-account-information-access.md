---
name: Access a customer's bank account data (AIS)
description: Authenticate, create a customer and consent, then read accounts, balances and transactions from Spare's Account Information API.
api: openapi/spare-bahrain-ais-openapi-original.json
tenant_note: Use the Bahrain host (ob.tryspare.com / sandbox.tryspare.com) or KSA host (ob.sparefinancial.sa) per the customer's jurisdiction.
operations:
  - GET /api/v1.0/authentication/Token
  - GET /api/v1.0/ais/Provider/List
  - POST /api/v1.0/ais/Customer/Create
  - POST /api/v1.0/ais/Connection/Create
  - POST /api/v1.0/ais/Consent/Create
  - GET /api/v1.0/ais/Account/List
  - GET /api/v1.0/ais/Balance/Get
  - POST /api/v1.0/ais/Transaction/List
---

# Access a customer's bank account data (AIS)

Use this skill to pull a customer's consented bank data from Spare's Account
Information API. Every step targets one tenant host; do not mix tenants.

## Prerequisites
- A Spare Dashboard account with API keys (ECC secp256r1 PKCS#8 key pair) and, if used, an IP allow-list.
- The correct tenant base host: Bahrain `https://ob.tryspare.com` (sandbox `https://sandbox.tryspare.com`), KSA `https://ob.sparefinancial.sa` (sandbox `https://sandbox.sparefinancial.sa`).

## Steps
1. **Get a token** — `GET /api/v1.0/authentication/Token` with your client id/secret. Store the JWT access token and refresh token. Send `Authorization: Bearer <JWT>` on every subsequent call. Refresh via `GET /api/v1.0/authentication/Refresh` when it expires.
2. **List providers** — `GET /api/v1.0/ais/Provider/List` to choose the customer's bank.
3. **Create the customer** — `POST /api/v1.0/ais/Customer/Create` (a `CUSTOMER_ALREADY_EXISTS` error means reuse the existing customer).
4. **(Optional) create a connection** — `POST /api/v1.0/ais/Connection/Create` to bind the customer to the chosen provider.
5. **Create consent** — `POST /api/v1.0/ais/Consent/Create` requesting the `AccountInformationPermission` values you need (e.g. `ReadAccountsBasic`, `ReadAccountsDetail`, `ReadBalances`, `ReadTransactionsBasic`, `ReadTransactionsDetail`). The response returns an authorisation link — send the end user to it to authorise in their bank app.
6. **Wait for authorisation** — poll `GET /api/v1.0/ais/Consent/Get` until authorised (or handle the consent webhook). Reading before authorisation returns `CONSENT_UNAUTHORIZED`.
7. **Read data** — `GET /api/v1.0/ais/Account/List`, then `GET /api/v1.0/ais/Balance/Get` and `POST /api/v1.0/ais/Transaction/List` (paginated via `PagesModel`: `currentPageNumber`, `pageSize`, `totalNumberOfPages`).

## Rules
- Each endpoint enforces the required consent permissions; missing ones return `INSUFFICIENT_CONSENT_PERMISSIONS` (see `scopes/spare-scopes.yml`).
- Handle consent lifecycle errors: `CONSENT_EXPIRED`, `CONSENT_REVOKED`, `CONSENT_REVOKED_OR_EXPIRED` (see `errors/spare-error-codes.yml`).
- There is no idempotency-key header; safety is bound to the single authorised consent (see `conventions/spare-conventions.yml`).
