---
name: Check ENGIE CDR data-holder health and outages
description: Poll the Consumer Data Standards discovery endpoints on ENGIE's registered CDR
  Public Base URI to determine whether the data holder is operational and whether an outage is
  scheduled. Fully anonymous.
api: openapi/simply-energy-cds-common-openapi.yml
base_url: https://public.cdr.engie.com.au/cds-au/v1
auth: none
operations: [getStatus, getOutages]
generated: '2026-07-27'
method: generated
---

# Check ENGIE CDR data-holder health and outages

This provider publishes no HTML status dashboard. The Consumer Data Standards discovery pair
**is** its status surface, and it is machine-readable and anonymous. Both endpoints were
confirmed HTTP 200 on 2026-07-27.

## Step 1 — health check (`getStatus`)

```
GET https://public.cdr.engie.com.au/cds-au/v1/discovery/status
x-v: 1
```

Response:

```json
{"data":{"status":"OK","updateTime":"2026-07-27T20:23:29Z","explanation":"All services operational"},
 "links":{"self":"..."},"meta":{}}
```

`data.status` is one of `OK`, `PARTIAL_FAILURE`, `UNAVAILABLE`, `SCHEDULED_OUTAGE`. Treat
anything other than `OK` as a reason to back off rather than retry hard.

## Step 2 — scheduled outages (`getOutages`)

```
GET https://public.cdr.engie.com.au/cds-au/v1/discovery/outages
x-v: 1
```

`data.outages[]` entries carry `outageTime` (ISO 8601 date-time), `duration` (ISO 8601
duration), `isPartial` and `explanation`. An empty array means none scheduled — that is what
was observed on 2026-07-27.

Under the Consumer Data Standards, normal planned outages must be published to data recipient
software products with **at least one week lead time**; unplanned changes to resolve a critical
service or security issue may occur without notice. Planned outages are excluded from the
99.5%-per-month availability obligation, so this endpoint is the only way to distinguish an
excused outage from a breach.

## Rules to follow

- **`x-v: 1` only.** Both endpoints reject `x-v: 2` and above with HTTP 406 and
  `urn:au-cds:error:cds-all:Header/UnsupportedVersion`. Verified live at `x-v: 9`.
- **Correlate your calls.** Send an RFC 4122 UUID in `x-fapi-interaction-id`; the data holder
  must echo it. Log the echoed value against your request.
- **Do not confuse the two hosts.** Discovery lives on `public.cdr.engie.com.au`; generic
  tariff data lives on `cdr.energymadeeasy.gov.au/engie`. Neither serves the other's paths, and
  the mismatch returns a bare nginx 404, not a CDS error body.
- **Both are high-priority endpoints** under the CDS performance tiers — 95% of calls per hour
  must respond within 1000ms. A sustained slower response is itself a signal.
