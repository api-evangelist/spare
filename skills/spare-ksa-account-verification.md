---
name: Verify a KSA bank account (Account Verification)
description: Use Spare's KSA Account Information API to verify an IBAN and retrieve beneficiary details against a National ID or Commercial Registration.
api: openapi/spare-ksa-ais-openapi-original.json
tenant_note: KSA tenant only — host https://ob.sparefinancial.sa (sandbox https://sandbox.sparefinancial.sa).
operations:
  - GET /api/v1.0/authentication/Token
  - POST /api/v1.0/ais/Customer/Create
  - POST /api/v1.0/ais/Consent/CreateShortLivedConsent
  - GET /api/v1.0/ais/Consent/Get
  - GET /api/v1.0/ais/Beneficiary/Get
  - GET /api/v1.0/ais/Account/Get
---

# Verify a KSA bank account (Account Verification)

Use this skill to verify a Saudi bank account (IBAN) and retrieve the
beneficiary/owner details, on the KSA tenant.

## Prerequisites
- Spare KSA Dashboard API keys; target `https://ob.sparefinancial.sa` (sandbox `https://sandbox.sparefinancial.sa`).

## Steps
1. **Get a token** — `GET /api/v1.0/authentication/Token`; use `Authorization: Bearer <JWT>` thereafter.
2. **Create the customer** — `POST /api/v1.0/ais/Customer/Create` for the subject being verified.
3. **Create a short-lived consent** — `POST /api/v1.0/ais/Consent/CreateShortLivedConsent` requesting beneficiary/account read permissions (`ReadBeneficiariesBasic`, `ReadBeneficiariesDetail`, `ReadAccountsBasic`, `ReadAccountsDetail`). Use the returned authorisation link.
4. **Confirm consent** — `GET /api/v1.0/ais/Consent/Get` until authorised.
5. **Read verification data** — `GET /api/v1.0/ais/Beneficiary/Get` and `GET /api/v1.0/ais/Account/Get` to confirm the IBAN and owner details.

## Notes
- KSA exposes both v1 (short-lived and long-lived consent) and v2 (long-lived consent) of the AIS API; this verification flow uses v1 short-lived consent.
- Handle `BANK_ACCOUNT_NOT_FOUND`, `BENEFICIARY_NOT_FOUND`, `INSUFFICIENT_CONSENT_PERMISSIONS` (see `errors/spare-error-codes.yml`).
