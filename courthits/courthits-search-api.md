# CourtHits API — reference & examples

The **CourtHits API** covers more than **court record search**. Organisation-authenticated endpoints include:

| Area | What it does |
|------|----------------|
| **Court records** | Search indexed courts, paginate with `resultSetId` / `cursor`, open full hit detail, related hits, lodge a **hit dispute**. |
| **Company registry** | Search JM / UK / US registers; fetch a **full UK/US profile** by registration number (cached). |
| **Adverse media** | Name search against configured media sources; cache + **snapshot token** for free reload. |
| **Sanctions & PEP** | Name search (PEP + watchlist scope per your environment); cache + **snapshot token**. |
| **Saved queries** | Store labelled screening queries (`company`, `adverse_media`, `sanctions`) for reuse in your UI. |

**CourtHits API base path:** `/api/v1/services/courthits` (prepend your API host; routing matches how you expose the Courthits service).

**Auth:** All endpoints in this document expect a valid **organisation (member) context** unless noted. Exact headers (`Authorization`, session cookies, gateway forwards) depend on your deployment.

---

## cURL pattern

Replace `<HOST>` and the `Authorization` header with whatever your gateway expects (Bearer JWT, forwarded session, etc.):

```bash
BASE="https://<HOST>/api/v1/services/courthits"
```

---

## Court records

### `POST /search`

**Purpose:** Organisation-scoped court search; returns hit summaries and a signed **`resultSetId`** for pagination, detail, and hydration.

**Body (JSON):**

| Field | Type | Required | Notes |
|-------|------|----------|--------|
| `q` | string | Yes | Non-empty |
| `country` | string | No | Default `JM` |
| `courtDivisions` | string[] | No | Refinement on same query core |
| `dateBucket` | string | No | `this_year` \| `last_year` \| `older` |
| `recordTypes` | string[] | No | Allow-listed |
| `cursor` | string | No | From previous `nextCursor` |
| `resultSetId` | string | No | From prior response — pagination, filter refinement, or first-page hydration |
| `idempotencyKey` | string | No | Idempotent credit debit for this logical search slice |

**Example request:**

```bash
curl -sS -X POST "$BASE/search" \
  -H "Authorization: Bearer <MEMBER_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{
    "q": "R v Crown",
    "country": "JM",
    "dateBucket": "this_year"
  }'
```

**Example response (shape; field names match your deployed API):**

```json
{
  "mode": "member",
  "totalHits": 12,
  "items": [
    {
      "id": "507f1f77bcf86cd799439011",
      "title": "Example v Example — Parish Court",
      "country": "JM",
      "courtDivision": "Parish Court — Criminal Division",
      "recordType": "mention",
      "caseNumber": "2025-CR-001"
    }
  ],
  "nextCursor": "eyJza2lwIjo1fQ==",
  "resultSetId": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "balance": 995,
  "chargedCredits": 5
}
```

**Hydration:** With a valid `resultSetId`, **first page** (no `cursor`), and a body **fingerprint** matching the saved result set, the service may return the first page from the stored id list with **`chargedCredits: 0`**.

**Refinement:** Changing **court division** or **record types** while keeping the same query core (`q`, `country`, `dateBucket`) may extend the same result set without a second search charge when `resultSetId` is valid.

**Errors:** `400` validation / invalid result set, `402` insufficient credits, `401` unauthenticated.

---

### `GET /record/:id`

**Purpose:** Full hit detail for a court record id.

**Query:**

| Param | Required | Notes |
|-------|----------|--------|
| `resultSetId` | Recommended | Must be a result set that contains this `id` for access control |
| `country` | Optional | Echo / routing helper |

**Example:**

```bash
curl -sS -G "$BASE/record/507f1f77bcf86cd799439011" \
  -H "Authorization: Bearer <MEMBER_TOKEN>" \
  --data-urlencode "resultSetId=<JWT_FROM_SEARCH>"
```

**Example response (abbreviated):**

```json
{
  "mode": "member",
  "item": {
    "id": "507f1f77bcf86cd799439011",
    "title": "…",
    "summary": "…",
    "parties": [],
    "sourceUrl": "https://…"
  },
  "entitledViaResultSet": true,
  "balance": 975,
  "chargedCredits": 20
}
```

May **debit** a detail credit per organisation + record when the last such debit for that hit is older than **`COURTHITS_HIT_PAID_ACCESS_TTL_DAYS`** (default 30 days); repeat views inside the window are free.

---

### `GET /record/:id/related`

**Purpose:** Related hits (same case number and/or court division + country, best-effort).

**Query:** Same `resultSetId` / `country` pattern as detail.

**Example:**

```bash
curl -sS -G "$BASE/record/507f1f77bcf86cd799439011/related" \
  -H "Authorization: Bearer <MEMBER_TOKEN>" \
  --data-urlencode "resultSetId=<JWT_FROM_SEARCH>"
```

**Example response:**

```json
{
  "mode": "member",
  "items": []
}
```

---

### `POST /dispute`

**Purpose:** Lodge a **hit dispute** (member). Persists a snapshot and returns a human-readable reference; operations are notified by email when configured.

**Body (JSON):**

| Field | Type | Required | Notes |
|-------|------|----------|--------|
| `recordId` | string | Yes | Mongo id of the court record |
| `resultSetId` | string | No | If sent, record must appear in that result set |
| `searchQuery` | string | No | Original query text (stored, max length enforced server-side) |
| `country` | string | No | Optional echo |

**Example:**

```bash
curl -sS -X POST "$BASE/dispute" \
  -H "Authorization: Bearer <MEMBER_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{
    "recordId": "507f1f77bcf86cd799439011",
    "resultSetId": "<JWT_FROM_SEARCH>",
    "searchQuery": "R v Crown",
    "country": "JM"
  }'
```

**Example success:**

```json
{
  "success": true,
  "referenceCode": "CH-20260404-A1B2C3",
  "disputeFormPath": "/pages/courthits-dispute-entry"
}
```

---

## Company registry (screening namespace)

Paths share the same `/api/v1/services/courthits` prefix.

### `POST /screening/company-search`

**Purpose:** Search **Jamaica**, **UK** (`GB`/`UK`), or **US** company registers. Credits apply; responses are cached by fingerprint.

**Body (JSON):**

| Field | Type | Required | Notes |
|-------|------|----------|--------|
| `country` | string | Yes | `JM`, `GB`, `UK`, or `US` |
| `name` | string | Yes | Company name |
| `state` | string | No | **US:** narrow by state (often required with `website`) |
| `website` | string | No | **US:** alternative narrow field |
| `idempotencyKey` | string | No | Dedupes credit for same logical search |

**US:** Either **`state`** or **`website`** is required to narrow large universes.

**Example (UK):**

```bash
curl -sS -X POST "$BASE/screening/company-search" \
  -H "Authorization: Bearer <MEMBER_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{"country":"GB","name":"Acme Holdings Ltd"}'
```

**Example response (shape):**

```json
{
  "success": true,
  "results": [
    {
      "name": "ACME HOLDINGS LTD",
      "registrationNumber": "12345678",
      "status": "active"
    }
  ],
  "mode": "member",
  "type": "company",
  "cacheHit": false,
  "balance": 990,
  "chargedCredits": 5
}
```

Exact `results[]` fields depend on registry provider.

---

### `POST /screening/company-profile`

**Purpose:** Full **UK or US** registry profile (officers, address, etc.) for one company, keyed by **registration number**. Cached; does not use the same credit path as `company-search` (see server implementation).

**Body (JSON):**

| Field | Type | Required |
|-------|------|----------|
| `country` | string | Yes — `GB` or `US` (`UK` normalised to `GB`) |
| `registrationNumber` | string | Yes |

**Example:**

```bash
curl -sS -X POST "$BASE/screening/company-profile" \
  -H "Authorization: Bearer <MEMBER_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{"country":"GB","registrationNumber":"12345678"}'
```

**Example response:**

```json
{
  "mode": "member",
  "type": "company_profile",
  "success": true,
  "profile": {}
}
```

`profile` is provider-specific JSON.

---

## Adverse media

### `POST /screening/media-search`

**Purpose:** Adverse **media** search for a **person name** (configured upstream). Returns hits/details plus credit metadata; may include **`snapshotToken`** to reload from cache.

**Body (JSON):**

| Field | Type | Required |
|-------|------|----------|
| `name` | string | Yes (max length enforced server-side) |
| `idempotencyKey` | string | No |

**Example:**

```bash
curl -sS -X POST "$BASE/screening/media-search" \
  -H "Authorization: Bearer <MEMBER_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{"name":"Jane Marie Doe"}'
```

**Example response (abbreviated):**

```json
{
  "hits": 3,
  "details": [],
  "name": "Jane Marie Doe",
  "mode": "member",
  "type": "adverse_media",
  "cacheHit": false,
  "expiresAt": "2026-04-05T12:00:00.000Z",
  "servedAt": "2026-04-04T10:00:00.000Z",
  "balance": 980,
  "chargedCredits": 10,
  "snapshotToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

---

## Sanctions & PEP

### `POST /screening/sanctions-search`

**Purpose:** **PEP** and **watchlist** style screening for a **person name** (scope fixed by server config). Returns a combined payload (`pep`, `dilisense`, optional `errors`) plus credits and optional **`snapshotToken`**.

**Body (JSON):**

| Field | Type | Required |
|-------|------|----------|
| `name` | string | Yes |
| `idempotencyKey` | string | No |

**Example:**

```bash
curl -sS -X POST "$BASE/screening/sanctions-search" \
  -H "Authorization: Bearer <MEMBER_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{"name":"Jane Marie Doe"}'
```

**Example response (abbreviated):**

```json
{
  "pep": { "hits": 0, "pepRecords": [] },
  "dilisense": { "hits": 0, "details": {} },
  "mode": "member",
  "type": "sanctions",
  "cacheHit": false,
  "balance": 970,
  "chargedCredits": 10,
  "snapshotToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

Partial upstream failures may appear under `errors` — treat as operational signals, not a substitute for policy.

---

## Reload cached screening (no extra credits)

### `GET /screening/member-snapshot?token=<snapshotToken>`

**Purpose:** Return the **same JSON shape** as the corresponding **`media-search`** or **`sanctions-search`** POST from **cache**, **without** charging credits again.

**Auth:** Member required; token must match organisation and screening type.

**Example:**

```bash
curl -sS -G "$BASE/screening/member-snapshot" \
  -H "Authorization: Bearer <MEMBER_TOKEN>" \
  --data-urlencode "token=<snapshotToken_FROM_POST>"
```

**Errors:** `410` if the cache entry expired or is missing.

---

## Saved screening queries

Stored per organisation for UI convenience (re-run flows in your app).

### `GET /screening/saved-queries`

Returns up to **100** items, newest first.

**Example response:**

```json
{
  "items": [
    {
      "id": "6625bcf6462398fa588cc1bf",
      "label": "Onboarding — media",
      "type": "adverse_media",
      "query": { "name": "Jane Doe" },
      "createdAt": "2026-04-01T12:00:00.000Z"
    }
  ]
}
```

---

### `POST /screening/saved-queries`

**Body (JSON):**

| Field | Type | Required | Notes |
|-------|------|----------|--------|
| `label` | string | Yes | Non-empty |
| `type` | string | Yes | `company` \| `adverse_media` \| `sanctions` |
| `query` | object | Yes | Opaque JSON your client can replay (e.g. `country`+`name` for company) |

**Example:**

```bash
curl -sS -X POST "$BASE/screening/saved-queries" \
  -H "Authorization: Bearer <MEMBER_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{
    "label": "US vendor check",
    "type": "company",
    "query": { "country": "US", "name": "Acme LLC", "state": "NY" }
  }'
```

---

### `DELETE /screening/saved-queries/:id`

Deletes one saved row if it belongs to the caller’s organisation.

```bash
curl -sS -X DELETE "$BASE/screening/saved-queries/6625bcf6462398fa588cc1bf" \
  -H "Authorization: Bearer <MEMBER_TOKEN>"
```

**Response:** `{ "ok": true }` or `404` if not found.

---

## Operations

### `GET /health`

Liveness JSON for the Courthits service process (no business data).

```json
{
  "ok": true,
  "service": "courthits",
  "package": "cleared-courthits-api"
}
```

---

## Conventions (all JSON endpoints)

- **JSON** request bodies and responses unless noted.
- **Court search page size:** up to **5** hits per `items` page; `totalHits` is the full match count.
- **`resultSetId`:** opaque **JWT** (or legacy UUID) for credited court search sessions.
- **`cursor`:** opaque **base64** string for the next page of the same court query core.

---

## Error shape

Typical error JSON:

```json
{
  "error": true,
  "message": "Human-readable message",
  "code": "machine_code_optional"
}
```

---

## Integration checklist

1. **Court:** `POST /search` → keep **`resultSetId`** + **`nextCursor`** for pagination.
2. **Court:** `GET /record/:id?resultSetId=...` for detail; related via **`/record/:id/related`**.
3. **Court:** Optional **`POST /dispute`** with `recordId` (+ `resultSetId` when applicable).
4. **Company:** `POST /screening/company-search` → optional **`POST /screening/company-profile`** for GB/US depth.
5. **Media / sanctions:** `POST` screening endpoints → persist **`snapshotToken`** and use **`GET /screening/member-snapshot`** to refresh UIs without re-charging (until TTL).
6. **Saved queries:** `GET` / `POST` / `DELETE` under **`/screening/saved-queries`** if your product needs starred searches.

---

*Credit costs, feature flags, cache TTLs, and upstream provider availability are controlled by your **CourtHits API** deployment — not duplicated here.*
