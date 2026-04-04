# CourtHits — Search & results API

Reference for **search**, **pagination**, **result sets**, **hit detail**, and **screening** (adverse media / sanctions) **without** separate billing or admin endpoints.

**CourtHits API base path:** `/api/v1/services/courthits` (relative to your API host; exact routing depends on how you expose the CourtHits service).

---

## Conventions

- **JSON** bodies and responses unless noted.
- **Page size:** court search returns up to **5** hits per request (`items`); `totalHits` is the full match count.
- **`resultSetId`:** opaque **JWT** (or legacy UUID) tying hits to a **credited** search session. Pass it on **pagination**, **detail**, and **hydration** as documented.
- **`cursor`:** opaque **base64** string; encodes the next `skip` offset for the same query core. Omit on first page.

---

## Court search (member)

### `POST /search`

**Purpose:** Organisation-scoped court search; returns full summary fields for each hit and a signed **`resultSetId`**.

**Auth:** Member / organisation context required (`401` if missing).

**Body (JSON):**

| Field | Type | Required | Notes |
|-------|------|----------|--------|
| `q` | string | Yes | Non-empty |
| `country` | string | No | Default `JM` |
| `courtDivisions` | string[] | No | Refinement on same query core |
| `dateBucket` | string | No | `this_year` \| `last_year` \| `older` |
| `recordTypes` | string[] | No | Allow-listed |
| `cursor` | string | No | Pagination |
| `resultSetId` | string | No | From prior response — **pagination** or **filter refinement** or **first-page hydration** |
| `idempotencyKey` | string | No | Idempotent credit debit for this logical search slice |

**Success response (member):**

| Field | Meaning |
|-------|---------|
| `mode` | `"member"` |
| `totalHits` | Total matches for the query |
| `items` | Page of projected hits |
| `nextCursor` | Opaque cursor or `null` |
| `resultSetId` | Pass on next page / detail / hydrate |
| `balance` | Organisation balance after debit (when applicable) |
| `chargedCredits` | Credits charged for this request (0 when duplicate/refinement/hydrate rules apply) |

**Hydration (no new search):** If `resultSetId` is supplied, **`cursor` omitted or first page**, and stored **fingerprint** matches the current body, the service may return the first page from the saved id list with **`chargedCredits: 0`** (same result set as a prior search).

**Refinement:** Changing **court division** / **record types** while keeping the same **query core** (`q`, `country`, `dateBucket`) may extend the same result set **without** a second search charge when `resultSetId` is valid (see API implementation).

**Errors:** `400` validation / invalid result set, `402` insufficient credits, `401` unauthenticated.

---

### `GET /record/:id`

**Purpose:** Full hit detail for a record id.

**Query:**

| Param | Required | Notes |
|-------|----------|--------|
| `resultSetId` | Recommended | Must be a result set that contains this `id` for access control |
| `country` | Optional | Echo / routing helper where used |

**Behaviour:** Validates membership in result set when `resultSetId` is provided; may **debit** a detail credit per org+record (idempotent). Returns full projected fields for members.

---

### `GET /record/:id/related`

**Purpose:** Related hits for the same matter (when configured).

**Query:** Same `resultSetId` / `country` pattern as detail.

---

## Adverse media & sanctions (member)

Paths are under the same CourtHits service prefix.

### `POST /screening/media-search`

**Body:** `{ "name": "<full name>" }` (plus optional `idempotencyKey`).

**Response:** Hits, article details, `balance`, `chargedCredits`, optional **`snapshotToken`** (JWT) to reload cached payload without charging again (until cache/TTL expires).

---

### `POST /screening/sanctions-search`

**Body:** `{ "name": "<full name>" }` (plus optional `idempotencyKey`).

**Response:** PEP + list screening payload, `balance`, `chargedCredits`, optional **`snapshotToken`**.

---

### `GET /screening/member-snapshot?token=<snapshotToken>`

**Purpose:** Return the **same JSON shape** as the corresponding POST from **cache**, **without** a new credit charge.

**Auth:** Member required; token must match organisation and screening type.

**Errors:** `410` if cache entry expired or missing.

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

1. Call **`POST /search`** with a non-empty `q` and valid member authentication.
2. Store **`resultSetId`** and use **`nextCursor`** until exhausted.
3. For hit UI, call **`GET /record/:id?resultSetId=...`**.
4. For parallel / saved flows, use **`resultSetId`** on the **first** member search page to **hydrate** without re-querying Mongo when fingerprint matches.
5. For screening, POST then optionally **`GET /screening/member-snapshot`** with **`snapshotToken`**.

---

*For behaviour tied to credit amounts and feature flags, see your **CourtHits API** deployment configuration and any runtime meta endpoints your gateway exposes — not duplicated here.*
