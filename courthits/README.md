# CourtHits API

**CourtHits** exposes indexed **court record search**, **hit detail**, and optional **adverse media** / **sanctions & PEP** screening for **authenticated organisation** integrations.

This folder documents **search and result-retrieval flows only** (no billing administration, coupon, or ledger APIs).

## Audience

Integrators who want to:

- Run **organisation-scoped** court searches from a backend service.
- Page results with **`resultSetId`** and **`cursor`**.
- **Hydrate** the first page from a saved result set (same fingerprint) without re-running the search.
- Open **full hit detail** and **related hits** in a credited, auditable way.
- Run **adverse media** / **sanctions** screening and **reload cached snapshots** via token.

## Base URL

The **CourtHits API** is served under:

```text
https://<host>/api/v1/services/courthits
```

Replace `<host>` with your environment (e.g. production or QA). All paths in the search & results document are **relative to that prefix**.

Example full URL for search:

```text
POST https://<host>/api/v1/services/courthits/search
```

## Authentication

- All documented **CourtHits API** routes require a valid **organisation (member) context** (typically the same Bearer session or gateway headers your Cleared workspace uses to reach services). Unauthenticated calls receive `401`.
- Align header and token handling with your **CourtHits API** / gateway configuration (e.g. **swf-core-gateway** Courthits proxy patterns).

## Documents

| Document | Description |
|----------|-------------|
| **[CourtHits search & results](./courthits-search-api.md)** | Search, `resultSetId`, cursors, record detail, screening search & snapshot |

## Related products

- **CourtHits web app** (`cleared-screening-portal`) calls these endpoints from the browser; server-side integrations should use the same paths with appropriate auth.

---

*CourtHits API documentation — maintained alongside `cleared-courthits-api`.*
