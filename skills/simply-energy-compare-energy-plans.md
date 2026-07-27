---
name: Compare ENGIE (formerly Simply Energy) retail energy plans
description: Retrieve and compare the live ENGIE electricity and gas tariff plans published as
  Consumer Data Right generic tariff data. Fully anonymous - no key, no registration.
api: openapi/simply-energy-cds-energy-openapi.yml
base_url: https://cdr.energymadeeasy.gov.au/engie/cds-au/v1
auth: none
operations: [listEnergyPlans, getEnergyPlanDetail]
generated: '2026-07-27'
method: generated
---

# Compare ENGIE retail energy plans

This is the only genuinely open API surface this provider has. It is Consumer Data Right
**generic tariff data**, hosted on the retailer's behalf by the Australian Energy Regulator's
Energy Made Easy service. Confirmed live on 2026-07-27 returning **2,452 plans**.

## Before you start

- No credential of any kind. Do not attempt to send an API key or Authorization header.
- The `x-v` header is **mandatory** and differs per operation. Getting it wrong returns
  HTTP 406, not 400.
- Base URL: `https://cdr.energymadeeasy.gov.au/engie/cds-au/v1`

## Step 1 — list plans (`listEnergyPlans`)

```
GET /energy/plans?page=1&page-size=100
x-v: 1
```

Useful query parameters from the spec: `type` (ALL | STANDING | MARKET | REGULATED),
`fuelType` (ALL | ELECTRICITY | GAS | DUAL), `effective` (CURRENT | ALL | FUTURE),
`updated-since`, `brand`, `page`, `page-size`.

Read the envelope, not just `data`:
- `data.plans[]` — each plan carries `planId`, `type`, `fuelType`, `brand`, `brandName`,
  `displayName`, `customerType`, `geography.distributors`, `effectiveFrom`, `lastUpdated`.
- `meta.totalRecords` and `meta.totalPages` — page through with `links.next` until it is absent.
- `page-size` defaults to 25 and **must not exceed 1000**; exceeding it returns
  `urn:au-cds:error:cds-all:Field/InvalidPageSize`.

## Step 2 — fetch the contract for a plan (`getEnergyPlanDetail`)

```
GET /energy/plans/{planId}
x-v: 3
```

Take `planId` verbatim from step 1 — it contains an `@` (observed form
`ENG1145433MRE1@EME`), so **URL-encode it** before substituting into the path.

**This operation is served only at `x-v: 3`.** `x-v: 1`, `2`, `4` and `5` all return HTTP 406.
This is the single most common integration mistake against this API: `listEnergyPlans` is v1
and `getEnergyPlanDetail` is v3.

The response `data.electricityContract` / `data.gasContract` carries `pricingModel`,
`tariffPeriod[]`, `controlledLoad[]`, `discounts[]`, `incentives[]`, `fees[]`, `eligibility[]`
and `solarFeedInTariff[]`. Compare on the tariff periods and the solar feed-in tariff, not on
`displayName`.

## Step 3 — filter to the right customer

A plan is only comparable if the customer's `geography.distributors` matches. Observed
distribution zones for this brand: Ausgrid, Endeavour, Essential Energy, Energex,
SA Power Networks, Jemena and AGN. Also filter on `customerType` (RESIDENTIAL vs BUSINESS)
and `type` (MARKET vs STANDING).

## Rules to follow

- **Cache.** Generic tariff data is explicitly a low-velocity data set under the Consumer Data
  Standards; use `lastUpdated` and `updated-since` rather than re-walking all 2,452 plans.
- **Respect the threshold.** Public unauthenticated traffic is capped at 300 TPS total across
  all consumers. Honour `Retry-After` if you receive it.
- **Errors are not RFC 9457.** The body is `{"errors":[{"code":"urn:au-cds:error:...",
  "title":..., "detail":...}]}` on `application/json`. See
  `errors/simply-energy-problem-types.yml`.
- **Do not treat an HTTP 200 as proof of registration.** An arbitrary unknown retailer slug on
  `cdr.energymadeeasy.gov.au` also returns HTTP 200, with `meta.totalRecords: 0`. The signal is
  a non-empty payload.
- **Do not call the retailer's own host for plans.**
  `https://public.cdr.engie.com.au/cds-au/v1/energy/plans` returns an nginx HTTP 404 by design —
  CDR energy centralises generic tariff data at the regulator.
