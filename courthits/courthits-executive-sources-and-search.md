# CourtHits — executive overview: sources and how we search

**Audience:** leadership, compliance, and partners who need a concise picture of what CourtHits is built on and how searches work—without implementation detail.

**Scope:** This reflects the **CourtHits** product as implemented today: the **cleared-courthits-api** service, **cleared-screening-portal** (CourtHits web surface), **swf-court-index-api.ms** (ingestion), and **swf-core-gateway** routing. Credit amounts, TTLs, and feature flags are **environment-specific**; behaviour described here is architectural.

---

## 1. What CourtHits does

CourtHits lets users **discover and review court-related records** that Cleared has already **ingested, normalised, and indexed** in our systems. It is a **research and triage** layer: hits point to **published** material and structured fields; customers remain responsible for verifying outcomes on **official** court or tribunal registers where that matters for their use case.

Alongside core court search, the same product surface exposes **optional screening lanes** (company registry, adverse media, sanctions / PEP) for **authenticated organisations**, using separate providers and caching rules.

---

## 2. Where court hit data comes from (sources)

### 2.1 Indexed court records (`CourtRecords`)

- **System of record for search:** MongoDB collection **`CourtRecords`** (court-index database). CourtHits **does not** crawl the open web at query time for the primary court search.
- **Ingestion:** Dedicated **court-index** pipelines ( **`swf-court-index-api.ms`** ) populate and update `CourtRecords` from **authorised channels**—for example official publishers, open registers, and approved feeds—depending on jurisdiction and source configuration.
- **Judgement-style and list material:** Traditional court-index flows produce structured records (e.g. mentions, lists, orders, judgements where published) with fields such as title, parties, case numbers, divisions, dates, and links back to **source URLs** where applicable.
- **Legal and enforcement news (Jamaica-focused example in code):** A separate **legal-events** pipeline discovers URLs via **Google SERP** ( **Apify** ), crawls article pages, extracts and classifies content, stores article text in **S3**, and **upserts** `CourtRecords` with record types such as **`reported_legal_event`** and **`official_police_notice`**. Registered domains in implementation include major Jamaican news sites and **JCF** (`jcf.gov.jm`) as an official police notice source—see **`LEGAL_EVENTS_INGESTION.md`** in the court-index repo.
- **Quality gate:** Records can be marked **`courthitsSearchExcluded`** when ingest classifies material as **non-legal noise** relative to CourtHits; those documents are **omitted from search** results.

### 2.2 Optional screening sources (not the court index)

These run **on demand** when an organisation calls the screening endpoints; responses are cached under configurable TTLs (shared cache fingerprint across orgs until expiry):

| Capability | Upstream / implementation (as wired in service) |
|------------|--------------------------------------------------|
| **Company registry search** | **UK:** Companies House API (`UK_COMPANY_HOUSE_API_KEY`). **US:** Enigma (`ENIGMA_API_KEY`). **Jamaica:** optional; gated by **`COURTHITS_COMPANY_SEARCH_JM_ENABLED`**. |
| **Adverse media** | **Dilisense** (`DIL_API_KEY`). |
| **Sanctions & PEP** | **Cleared PEP** index (global scope **`all`** in API) **plus** **Dilisense** person scan branch. |

Privacy posture for screening: audit collections avoid storing raw search names where minimisation rules apply; see **`screening.js`** comments.

---

## 3. How court search runs (end-to-end)

### 3.1 API and routing

- Browser and integrators reach CourtHits under **`/api/v1/services/courthits`** via **swf-core-gateway** (hosting and auth as per environment).
- **Court** endpoints include **`POST /search`** (member), **`POST /guest/search`** (limited guest), **`GET /record/:id`**, related hits, disputes, optional enrichment, and result-set helpers documented in **`courthits-search-api.md`**.

### 3.2 Query execution (member and guest)

- Search is implemented as a **MongoDB query** over `CourtRecords`: the query string is **tokenised** (whitespace / commas); **each token** must match (case-insensitive **regex**) in at least one of **`searchText`**, **`title`**, **`caseNumber`**, or **`parties.name`**—so **word order is flexible**.
- **Filters** refine the same logical search: **country** (with Jamaica aliasing), **court divisions**, optional **date bucket** (`this_year` / `last_year` / `older`), and allow-listed **`recordTypes`** (including legal-event types when enabled in the UI).
- **Pagination:** Responses return a small **page of hits** (implementation uses a fixed page size, e.g. **5** per page), **`totalHits`**, **`nextCursor`**, and a **`resultSetId`** that ties pagination, detail access, and refinement together.
- **Atlas Search:** Documented as **optional / future**; current production pattern is **regex on Mongo**, not `$search`, so search does not depend on a live Atlas Search index (**`ATLAS_SEARCH.md`** in cleared-courthits-api).

### 3.3 Member vs guest

| Aspect | Guest | Member (organisation) |
|--------|--------|-------------------------|
| **Auth** | Unauthenticated / device identity | Organisation session / gateway auth |
| **Quota** | Daily cap per **device id** and **hashed IP** (UTC day); separate **per-IP rate limit** window | Credits and org balance |
| **Abuse** | Optional **VPN / proxy / hosting** block unless **`ENABLE_VPN_SEARCH`** is enabled | N/A (same IP intel pattern not applied as guest gate) |
| **Result set** | No refinement / hydration model equivalent to member **`resultSetId`** | **Signed `resultSetId` (JWT)** stores allowed hit ids; supports **filter refinement** without a second search charge when query core unchanged; **hydrate** first page without re-query when fingerprint matches |
| **Detail** | **Redacted** guest record view | Full detail; **detail credits** may apply per org+record with a **paid-access TTL** (repeat views in window free) |

### 3.4 Credits (court)

- **Search:** Typically **one debit** per distinct **query core** (`q` + `country` + `dateBucket`) for the first slice; refinement and pagination extensions are designed to avoid duplicate search charges when **`resultSetId`** is valid (see **`index.js`** header comments).
- **Hit detail:** Separate **detail** credit policy with **TTL-based** repeat access.
- **Optional LLM enrichment:** Separate enrichment credit path where implemented.

### 3.5 Disputes

- Members may lodge **hit disputes**; server persists a snapshot and returns a **reference code**; operations may be notified by email when configured.

---

## 4. How parallel screening searches run (summary)

- **Company:** `POST` with country + name (US requires narrowing fields such as **state** or **website**). Fingerprinted **cache**; credits on cache miss path per config.
- **Adverse media / sanctions:** `POST` with **person name**; returns payload + optional **`snapshotToken`**; **`GET /screening/member-snapshot`** reloads cached JSON **without** a second credit until cache expiry (**410** when gone).

---

## 5. What this document does not claim

- It does **not** list every court or tribunal worldwide or guarantee completeness per jurisdiction.
- It does **not** replace **privacy policy**, **DPA**, or customer contracts; Trust Centre remains authoritative for policy text.
- **Provider names, keys, and costs** are deployment-specific; validate against your environment’s master config and runbooks.

---

## 6. Related technical documents

| Document | Location |
|----------|----------|
| CourtHits API overview | `api-docs/courthits/README.md` |
| Search & screening API reference | `api-docs/courthits/courthits-search-api.md` |
| Courthits service README (env, routes, providers) | `cleared-courthits-api/README.md` |
| Search implementation note (Mongo regex) | `cleared-courthits-api/docs/ATLAS_SEARCH.md` |
| Legal-events ingestion (domains, Apify, flows) | `swf-court-index-api.ms/docs/LEGAL_EVENTS_INGESTION.md` |

---

*Last updated to match repository behaviour as of the document’s authoring; update when ingestion or provider wiring changes materially.*
