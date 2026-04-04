# CourtHits API

**CourtHits** exposes indexed **court record search**, **hit detail**, and optional **adverse media** / **sanctions & PEP** screening for **authenticated organisation** integrations.

This folder documents **search and result-retrieval flows only** (no billing administration, coupon, or ledger APIs).

## Audience

Integrators who want to:

- Run **organisation-scoped** **court** search and open **hit detail**, **related hits**, and **disputes**.
- Search **company registries** (JM / UK / US) and load **UK/US company profiles** by registration number.
- Run **adverse media** and **sanctions & PEP** name screening, then **reload cached snapshots** without a second charge.
- Store **saved screening queries** for company / media / sanctions flows.

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
| **[CourtHits API reference & examples](./courthits-search-api.md)** | Court records, disputes, company registry & profile, adverse media, sanctions, snapshots, saved queries, `GET /health` |

## Related products

- **CourtHits web app** (`cleared-screening-portal`) calls these endpoints from the browser; server-side integrations should use the same paths with appropriate auth.

---

*CourtHits API documentation — maintained alongside `cleared-courthits-api`.*
