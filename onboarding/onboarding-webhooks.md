# Onboarding (IDV) webhooks

Receive **HTTP POST** notifications when onboarding / verification lifecycle events are delivered for a subscribed organisation. Events use **camelCase** ids (unlike document signing webhooks, which use snake_case).

For the full live list, call the portal **event-catalog** API; this guide covers configuration, request format, and a deep dive on **`initialReviewCompleted`**.

## Configuration

| Method | Where |
|--------|--------|
| **Onboarding page / organisation binding** | Cleared Verification portal → Webhooks (or onboarding page webhook settings) |
| **Event filter** | Optional `subscribedEvents` on each binding — omit or leave unset to receive **all** catalog events; use a string array to whitelist specific camelCase event names |
| **Secret** | Per-endpoint secret used for HMAC (`X-Webhook-Signature`) |

Each binding targets one HTTPS URL. Multiple bindings may fire for the same event.

## Request format

Cleared sends an **HTTP POST** with:

| Header | Value |
|--------|--------|
| `Content-Type` | `application/json` |
| `X-Webhook-Signature` | `sha256=<hex>` — HMAC-SHA256 of the **raw JSON body** using your webhook secret (when a secret is configured) |

Your endpoint should:

- Respond with **2xx** promptly
- Verify the signature before processing
- Treat deliveries as **at-least-once**; use `eventName` + `eventOccurredAt` + `onboarding.urlParameters` (e.g. application id) for idempotency

### Envelope (VerificationStatus style)

Typical body fields:

| Field | Description |
|-------|-------------|
| `topic` | Usually `VerificationStatus` |
| `customerName` / `customerEmailAddress` | Customer identity |
| `verifications` | Array of verification rows (`type`, `status`, type-specific fields) |
| `onboarding` | `{ pageId, onboardingPageId, urlParameters }` when the flow came from an onboarding page |
| `eventName` | Catalog id (camelCase), e.g. `identityVerificationCleared`, `initialReviewCompleted` |
| `eventContext` | Event-specific object (see deep dive below) |
| `eventOccurredAt` | ISO 8601 timestamp |

#### `verifications[]` — identity row fields

When a row has `type: "identity"`, Cleared includes status fields plus person identifiers resolved from the latest identity result (and tax number fallbacks) at **delivery** time:

| Field | When present | Notes |
|-------|----------------|-------|
| `documentType` | When known on the identity result | e.g. passport, driver licence |
| `clearedAt` / `expiresAt` | When known | ISO 8601 |
| `taxNumber` | When a 9-digit Jamaica TRN can be resolved | From identity TRN metadata, else the user’s unique id, else a TRN verification result |
| `idNumber` | When known | From `documentInfo.idNumber` (or non-TRN `documentNumber`) — the **document / national ID number**, not the tax number |
| `dateOfBirth` | When known | `YYYY-MM-DD` from the identity biography |
| `initialReviewStatus` | Always on identity rows | `completed` or `pending` |

TRN verification rows (`type: "trn"`) also expose `taxNumber` when available.

#### `onboarding.urlParameters`

Configured landing / link query keys captured when the subject starts the flow (stored on the verification request). Empty maps are omitted. Progress events such as `identityVerificationStarted` are pinned to the **flow** verification request when the client sends that request id, so the first webhook can include these parameters for the current visit.

**Verification:** recompute HMAC-SHA256 on the raw request body and compare with `X-Webhook-Signature` (value after `sha256=`). If invalid, respond with 401.

## Event catalog (selected)

All onboarding event names use **camelCase**. Below are common decision / IR events; the portal catalog is the SSOT for the full set.

| Event | Category | When Cleared sends it |
|-------|----------|------------------------|
| `identityVerificationStarted` | progress | Customer starts identity capture for a flow request (pinned to that request so `onboarding.urlParameters` match the visit) |
| `identityVerificationSubmitted` | progress | Identity documents submitted |
| `initialReviewCompleted` | decisions | Ops completes **Initial Review** (triage advance or legacy mark-complete). **Not** sent on IR reject. **Does not** start due diligence. |
| `dueDiligenceStarted` | decisions | Ops starts **due diligence** after Initial Review (separate Start due diligence action). |
| `identityVerificationCleared` | decisions | Identity cleared by ops (or ensured on consent share of an already-cleared result) |
| `identityVerificationRejected` | decisions | Identity rejected by ops |
| `identityVerificationApproved` | decisions | Customer / share approval path for identity |
| `verificationRequestApproved` | session | Verification request approved for share |
| `verificationStatusUpdated` | progress | Fallback / status refresh |

For signing lifecycle events (`document_signed`, etc.), see [Document signing webhooks](../document-signatures/document-webhooks.md).

## Deep dive: `initialReviewCompleted`

Emitted when Initial Review is **advanced** (or legacy mark-complete), while final clear/reject may still happen later. `eventContext.finalStatus` is always `"pending"`.

### `eventContext` fields

| Field | Rule |
|-------|------|
| `resultType` / `verificationType` / `resultId` | Always present for IR |
| `source` | `ops_initial_review_triage_advance` (Control Centre triage) or `ops_initial_review_complete` (legacy mark-complete) |
| `level` | `1` \| `2` \| `3`, or `null` (legacy mark-complete) |
| `riskCategory` | Derived: `1→low`, `2→medium`, `3→high`, else `null` |
| `reasonCodes` | string[] (may be empty) |
| `caseStatus` | Actual identity result **status** after advance (`processing` / `flagged`, etc.) |
| `nextCaseStatus` | Triage choice when available (`continue_review`, `awaiting_resubmission`, …). **Not** `due_diligence` — that is a separate Start due diligence action. |
| `resubmissionRequired` | boolean |
| `resubmission` | Present when required: `{ requestId, labels? }` — omitted when not required |
| `reviewedAt` | ISO string |
| `finalStatus` | Always `"pending"` |
| `reviewDurationMs` | number or `null` |
| `taxNumber` | When resolvable — same rules as the identity `verifications[]` row |
| `idNumber` | When known on the identity result biography / document info |
| `dateOfBirth` | When known — `YYYY-MM-DD` |

**Never** included: ops `internalNotes`, or other PII beyond the envelope + the person fields above. Document / tax numbers appear only as the dedicated `idNumber` / `taxNumber` fields (not as free-form notes).

### Example `eventContext` (triage advance)

```json
{
  "resultType": "IdentityVerificationResult",
  "verificationType": "identity",
  "resultId": "64f1a2b3c4d5e6f7a8b9c0d1",
  "source": "ops_initial_review_triage_advance",
  "level": 2,
  "riskCategory": "medium",
  "reasonCodes": ["blurry_photo"],
  "caseStatus": "processing",
  "nextCaseStatus": "continue_review",
  "resubmissionRequired": false,
  "reviewedAt": "2026-07-24T12:00:00.000Z",
  "finalStatus": "pending",
  "reviewDurationMs": 180000,
  "taxNumber": "123456789",
  "idNumber": "A1234567",
  "dateOfBirth": "1990-05-15"
}
```

## Deep dive: `identityVerificationCleared`

Emitted when ops **clears** identity (final clear path). The VerificationStatus envelope’s identity row and `eventContext` both carry person identifiers when available.

### `eventContext` person fields

| Field | Rule |
|-------|------|
| `taxNumber` | When a 9-digit TRN can be resolved (identity metadata → user unique id → TRN verification) |
| `idNumber` | Document / national ID number from the identity result when present |
| `dateOfBirth` | `YYYY-MM-DD` when present on the biography |

Same fields appear on the matching `verifications[]` identity row for this and other lifecycle events delivered via the VerificationStatus payload builder.

### Example identity row snippet

```json
{
  "type": "identity",
  "status": "cleared",
  "documentType": "passport",
  "clearedAt": "2026-09-15T12:00:00.000Z",
  "taxNumber": "123456789",
  "idNumber": "A1234567",
  "dateOfBirth": "1990-05-15",
  "initialReviewStatus": "completed"
}
```

## Action → event matrix (Initial Review Triage)

| Ops / triage action | Client onboarding event | Ops PA topic (Teams / Power Automate) |
|---------------------|-------------------------|----------------------------------------|
| claim / assign / release / start / reassign | *(none — audit only)* | *(none)* |
| advance (`continue_review` / `awaiting_resubmission`) | `initialReviewCompleted` | `initial_review_completed` |
| start due diligence (post-IR) | `dueDiligenceStarted` | `due_diligence_started` |
| reject | **No** `initialReviewCompleted` — identity owns `identityVerificationRejected` | `initial_review_completed` (ops) + later `identity_decided` from identity decision path |

## Deep dive: `dueDiligenceStarted`

Emitted when an agent starts due diligence **after** Initial Review is complete. Separate from `initialReviewCompleted`.

### `eventContext` fields

| Field | Rule |
|-------|------|
| `resultType` / `verificationType` / `resultId` | Always present |
| `source` | `ops_due_diligence_started` |
| `caseStatus` | `due_diligence` |
| `dueDiligenceCategory` | string or `null` |
| `reasonCodes` | string[] (may be empty) |
| `startedAt` | ISO string |
| `finalStatus` | Always `"pending"` |

## Related

- [Client loan / onboarding workflow](../client-loan-application-workflow.md) — end-to-end IDV + webhooks
- [Document signing webhooks](../document-signatures/document-webhooks.md) — snake_case signing events
- [API endpoints overview](../endpoints.md) — webhooks section
- Ops engineering runbook (internal): `docs/ops-webhook-publishing.md` — topics `initial_review_completed`, `due_diligence_started`
