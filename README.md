# Simply Energy (simply-energy)

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Simply Energy is the former brand of the Australian electricity and gas retailer that now trades as ENGIE, operated by IPower Pty Ltd (ACN 111 267 228) and IPower 2 Pty Ltd (ABN 24 070 374 293) trading as ENGIE (ABN 67 269 241 237), together with Simply Energy Solutions Pty Ltd. The business carried the Simply Energy name for seventeen years before rebranding to ENGIE in April 2024, and supplies more than 700,000 residential and business accounts across Victoria, South Australia, New South Wales, Queensland and Western Australia. It sits on the retail tier of the National Electricity Market value chain, buying wholesale energy and billing end customers, rather than in generation, transmission or distribution. Its API posture is entirely a product of statutory mandate, and in this case the mandate is genuinely implemented rather than merely claimed. The company is a designated Consumer Data Right energy data holder (data holder provider number DH002028) and is listed on the live CDR Register under the ENGIE brand. Its unauthenticated CDR Generic Plans endpoint, hosted for it by the Australian Energy Regulator's Energy Made Easy service, returns 2,452 real ENGIE tariff plans conforming to the Consumer Data Standards energy schemas, and its own registered public base URI serves the Consumer Data Standards discovery endpoints with correct `x-v` version negotiation. Everything else is closed. Customer usage, billing, service point and DER data are available only to ACCC accredited data recipients with explicit consumer consent, there is no developer portal, no self-serve API keys, no published OpenID Connect discovery document, and no open grid, market or system data of any kind.

**APIs.json:** [https://raw.githubusercontent.com/api-evangelist/simply-energy/refs/heads/main/apis.yml](https://raw.githubusercontent.com/api-evangelist/simply-energy/refs/heads/main/apis.yml)

## Tags

- Energy
- Australia
- Utilities
- Electricity
- Gas
- Energy Retail
- Consumer Data Right
- CDR
- Smart Metering
- Energy Markets

## Timestamps

- **Created:** 2026-07-27
- **Modified:** 2026-07-27

## Mandate Posture

| Field | Value |
| --- | --- |
| Mandate regime | `cdr-energy` — Australia's Consumer Data Right, extended from banking into energy |
| Mandate status | `live-implemented` — verified on the wire, not claimed |
| Data standard | CDR Consumer Data Standards (Energy) v1.36.0 |
| Consumer data API | Yes — accredited data recipients only, with consumer consent |
| Open market data | No — no grid, system or wholesale feed published by this company |
| Access gate | `accredited-only` for consumer data; fully anonymous for tariff data |
| Developer portal | None |

The evidence: the public CDR Register lists brand **ENGIE** (ABN 67269241237, `publicBaseUri` `https://public.cdr.engie.com.au`) among 84 energy data holder brands — and lists no brand named "Simply Energy". That registered base URI answers `GET /cds-au/v1/discovery/status` and `/cds-au/v1/discovery/outages` with HTTP 200 at `x-v: 1` and rejects higher versions with `urn:au-cds:error:cds-all:Header/UnsupportedVersion`. The mandated generic tariff data answers with 2,452 real ENGIE plans. See [`review.yml`](review.yml) for every URL probed and every status code returned, including the control test that proves the tariff payload — not the status code — is what establishes the implementation.

## APIs

### Simply Energy (ENGIE) CDR Energy Generic Plans API

Public, unauthenticated Consumer Data Right Generic Tariff Data for the Simply Energy / ENGIE retail brand, conforming to the Data Standards Body Consumer Data Standards energy schemas. Confirmed live on 2026-07-27 — `GET /energy/plans` returns HTTP 200 with `x-v: 1` and `meta.totalRecords` 2452 ENGIE electricity and gas plans (MARKET and STANDING) across Ausgrid, Endeavour, Essential Energy, Energex, SA Power Networks, Jemena and AGN distribution zones. `GET /energy/plans/{planId}` returns HTTP 200 at `x-v: 3`. The endpoint is hosted on behalf of the retailer by the Australian Energy Regulator's Energy Made Easy service; it is not self-hosted.

- **Human URL:** [https://engie.com.au/help-centre/cdr-policy](https://engie.com.au/help-centre/cdr-policy)
- **Base URL:** `https://cdr.energymadeeasy.gov.au/engie/cds-au/v1`

#### Tags

- Energy Plans
- Generic Tariff Data
- Consumer Data Right
- Public

#### Properties

- [OpenAPI](openapi/simply-energy-cds-energy-openapi.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [API Reference](https://consumerdatastandardsaustralia.github.io/standards/#energy-apis)
- [Documentation](https://engie.com.au/help-centre/cdr-policy)

### Simply Energy (ENGIE) CDR Discovery API

The Consumer Data Standards Common Discovery API served from the brand's own registered CDR Public Base URI. Confirmed live on 2026-07-27 — `GET /discovery/status` returns HTTP 200 at `x-v: 1` with status OK, and `GET /discovery/outages` returns HTTP 200 at `x-v: 1` with an empty outages array. Both correctly reject `x-v: 2` and above with `urn:au-cds:error:cds-all:Header/UnsupportedVersion`. This host does **not** serve `/energy/plans` — it returns an nginx HTTP 404 — which matches the behaviour of AGL, Alinta and Origin Energy and reflects the CDR energy design in which generic tariff data is centralised at the Australian Energy Regulator rather than held by each retailer.

- **Human URL:** [https://engie.com.au/help-centre/cdr-policy](https://engie.com.au/help-centre/cdr-policy)
- **Base URL:** `https://public.cdr.engie.com.au/cds-au/v1`

#### Tags

- Discovery
- Status
- Outages
- Consumer Data Right
- Public

#### Properties

- [OpenAPI](openapi/simply-energy-cds-common-openapi.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [API Reference](https://consumerdatastandardsaustralia.github.io/standards/#common-apis)
- [Documentation](https://engie.com.au/help-centre/cdr-policy)

### Simply Energy (ENGIE) CDR Energy Consumer Data API

The mandated Consumer Data Right consumer data sharing surface — energy accounts, balances, billing, invoices, concessions, payment schedules, electricity service points, usage and distributed energy resources — conforming to the shared Consumer Data Standards energy contract. **Not anonymously verifiable and deliberately listed without a base URL:** the CDR InfoSec and resource base URIs for a data holder are published only through the authenticated portion of the CDR Register and are reachable only by an ACCC accredited data recipient presenting a client certificate. Access requires ACCC accreditation, OAuth2 / OpenID Connect under the CDR FAPI profile, mutual TLS, and explicit consumer consent. Recorded here as designated and documented, not as an endpoint confirmed by API Evangelist.

- **Human URL:** [https://engie.com.au/help-centre/cdr-policy](https://engie.com.au/help-centre/cdr-policy)

#### Tags

- Energy Accounts
- Usage
- Billing
- DER
- Consumer Data Right
- Accredited Only

#### Properties

- [OpenAPI](openapi/simply-energy-cds-energy-openapi.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [API Reference](https://consumerdatastandardsaustralia.github.io/standards/#energy-apis)
- [Documentation](https://engie.com.au/help-centre/cdr-policy)

## Common Properties

- [Website](https://www.simplyenergy.com.au/)
- [Website](https://engie.com.au/)
- [Documentation](https://engie.com.au/help-centre/cdr-policy)
- [API Reference](https://consumerdatastandardsaustralia.github.io/standards/#energy-apis)
- [Registry](https://api.cdr.gov.au/cdr-register/v1/energy/data-holders/brands/summary)
- [LinkedIn](https://www.linkedin.com/company/engie-australia-new-zealand/)

## Maintainers

- Kin Lane — kin@apievangelist.com
