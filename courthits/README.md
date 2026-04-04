# CourtHits API

**CourtHits** exposes indexed **court record search**, **hit detail**, and optional **adverse media** / **sanctions & PEP** screening behind the same Cleared gateway as other services.

This folder documents **search and result-retrieval flows only** (no billing administration, coupon, or ledger APIs).

## Audience

Integrators who want to:

- Run **guest** or **member** court searches from a backend service.
- Page results with **`resultSetId`** and **`cursor`**.
- **Hydrate** the first page from a saved result set (same fingerprint) without re-running the search.
- Open **full hit detail** and **related hits** in a credited, auditable way.
- Run **adverse media** / **sanctions** screening and **reload cached snapshots** via token.

## Base URL

CourtHits is mounted on the Cleared API gateway under:

```text
https://<host>/api/v1/services/courthits
```

Replace `<host>` with your environment (e.g. production or QA). All paths below are **relative to that prefix**.

Example full URL for member search:

```text
POST https://<host>/api/v1/services/courthits/search
```

## Authentication

- **Member** routes require a valid **organisation context** (typically the same Bearer session or gateway headers your other Cleared services use). Unauthenticated calls receive `401`.
- **Guest** routes are designed for **device-scoped** usage: the gateway/client should send the **`cleared-device-id`** header (and related guest flows) as configured for your product.

Exact header names mirror the **swf-core-gateway** Courthits proxy and portal client; align with your integration’s existing Cleared auth pattern.

## Documents

| Document | Description |
|----------|-------------|
| **[CourtHits search & results](./courthits-search-api.md)** | Guest search, member search, `resultSetId`, cursors, record detail, screening search & snapshot |

## Related products

- **CourtHits web app** (`cleared-screening-portal`) uses these endpoints from the browser; server-side integrations should call the same paths with appropriate auth.

---

*CourtHits API documentation — maintained alongside `cleared-courthits-api`.*
