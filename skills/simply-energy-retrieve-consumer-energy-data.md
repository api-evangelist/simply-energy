---
name: Retrieve a consumer's ENGIE energy accounts, usage and billing (accredited only)
description: The CDR consumer data-sharing flow for this data holder - accounts, service
  points, usage, DER, balances, invoices and billing. Requires ACCC accreditation, a CDR client
  certificate and explicit consumer consent. There is no self-serve tier and no sandbox key.
api: openapi/simply-energy-cds-energy-openapi.yml
base_url: null
auth: oauth2 + openIdConnect + mutualTLS (CDR FAPI 1.0 Advanced profile)
operations: [getCustomer, listEnergyAccounts, getEnergyAccountDetail,
  listElectricityServicePoints, getElectricityServicePointDetail,
  getElectricityServicePointUsage, listElectricityUsageBulk, getElectricityDERForServicePoint,
  getEnergyAccountBalance, getEnergyAccountInvoices, getBillingForEnergyAccount,
  getEnergyAccountPaymentSchedule, getEnergyAccountConcessions]
generated: '2026-07-27'
method: generated
---

# Retrieve a consumer's ENGIE energy data (accredited data recipients only)

**Read this first.** This flow cannot be run by an unaccredited developer, and no part of it
can be tested anonymously. If you are not an ACCC-accredited data recipient, stop here and use
`simply-energy-compare-energy-plans.md` instead — that is the surface you can actually call.

## Prerequisites (all mandatory, none skippable)

1. ACCC accreditation as a CDR data recipient, meeting the information security controls in
   Schedule 2 of the CDR Rules.
2. A CDR client certificate issued under the CDR PKI.
3. A software product registered on the CDR Register via Dynamic Client Registration.
4. The CDR Security Profile implemented: OAuth 2.0 authorization code + OpenID Connect under
   **FAPI 1.0 Advanced**, with **PAR**, **PKCE**, **private_key_jwt** client authentication,
   **MTLS** holder-of-key token binding and **JARM**.
5. An explicit, scoped, time-limited consumer consent collected through the CDR consent model.

The data holder's InfoSec and resource base URIs are **not** anonymously discoverable — they
are published only through the authenticated portion of the CDR Register. Anonymous
`/.well-known/openid-configuration` probes on `public.cdr.engie.com.au` and
`cdr.energymadeeasy.gov.au` both returned HTTP 404. Resolve the base URIs from the Register
with your client certificate; do not hard-code a host.

## Step 0 — request the right scopes

Ask only for what the use case needs. Scope → data mapping is in
`scopes/simply-energy-scopes.yml`; the consumer sees the CDR data-cluster language, so
over-asking visibly costs you consent.

| Need | Scope |
|---|---|
| Who the customer is | `common:customer.basic:read` / `common:customer.detail:read` |
| Accounts and plans | `energy:accounts.basic:read` / `energy:accounts.detail:read` |
| Balances, invoices, billing | `energy:billing:read` |
| Payment preferences | `energy:accounts.paymentschedule:read` |
| Concessions | `energy:accounts.concessions:read` |
| NMI / connection point | `energy:electricity.servicepoints.basic:read` / `.detail:read` |
| Meter reads | `energy:electricity.usage:read` |
| Solar and battery | `energy:electricity.der:read` |

## Step 1 — identify the consented customer

`getCustomer` → `GET /common/customer` (`x-v: 1`) or `getCustomerDetail` →
`GET /common/customer/detail` (`x-v: 2`).

## Step 2 — enumerate accounts

`listEnergyAccounts` → `GET /energy/accounts?open-status=OPEN` (`x-v: 2`). Then
`getEnergyAccountDetail` → `GET /energy/accounts/{accountId}` (`x-v: 4`) for the plan the
account is on.

## Step 3 — enumerate connection points

`listElectricityServicePoints` → `GET /energy/electricity/servicepoints` (`x-v: 2`), then
`getElectricityServicePointDetail` → `GET /energy/electricity/servicepoints/{servicePointId}`
(`x-v: 2`) for the NMI standing data and meter details.

## Step 4 — pull usage

Per service point: `getElectricityServicePointUsage` →
`GET /energy/electricity/servicepoints/{servicePointId}/usage` with `oldest-date`,
`newest-date` and `interval-reads` (`NONE` | `MIN_30` | `FULL`).

Across the consent: `listElectricityUsageBulk` → `GET /energy/electricity/servicepoints/usage`,
or `listElectricityUsageForServicePoints` → `POST /energy/electricity/servicepoints/usage` with
the service point ids in the body.

**`interval-reads=FULL` is the expensive call.** Request `MIN_30` unless you genuinely need
sub-interval granularity, and window your date range.

## Step 5 — solar and storage

`getElectricityDERForServicePoint` →
`GET /energy/electricity/servicepoints/{servicePointId}/der`, or the bulk/POST variants
`listElectricityDERBulk` and `listElectricityDERForSpecificServicePoints`.

## Step 6 — money

- `getEnergyAccountBalance` → `GET /energy/accounts/{accountId}/balance`
- `getEnergyAccountInvoices` → `GET /energy/accounts/{accountId}/invoices`
- `getBillingForEnergyAccount` → `GET /energy/accounts/{accountId}/billing`
- bulk and POST-by-id variants exist for all three
- `getEnergyAccountPaymentSchedule`, `getEnergyAccountConcessions`

## Rules to follow

- **Every call needs `x-v`, and the version differs per operation.** Check
  `x-version` in the spec for each operation before you call it. A wrong version is a 406, not
  a 400.
- **Send the FAPI context headers.** `x-fapi-auth-date` is required on all resource calls;
  `x-fapi-customer-ip-address` and `x-cds-client-headers` are mandatory for customer-present
  calls and must be absent for unattended calls.
- **There is no idempotency key.** The Consumer Data Standards define none. The five POST
  operations are POST-as-query and mutate nothing, so a retry is safe — but do not invent an
  `Idempotency-Key` header; it will be ignored.
- **Respect the unattended thresholds.** 20 sessions per day per customer per software product,
  100 calls per session, 5 TPS per session. Schedule unattended pulls away from peak.
- **Cache and honour ID permanence.** `accountId` and `servicePointId` are arbitrary, immutable
  per consent, and **not transferable across software products**. Never key a cross-product
  join on them.
- **Handle consent failure distinctly.** `Authorisation/RevokedConsent` and
  `Authorisation/InvalidConsent` (HTTP 403) mean re-consent, not retry.
  `Authorisation/UnavailableEnergyAccount` (404/422) is transient; `InvalidEnergyAccount` is
  permanent. Full registry in `errors/simply-energy-problem-types.yml`.
