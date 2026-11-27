# Onboarding (IDV) webhooks

Receive **HTTP POST** notifications when onboarding / verification lifecycle events are delivered for a subscribed organisation. Events use **camelCase** ids (unlike document signing webhooks, which use snake_case).

For the full live list, call the portal **event-catalog** API; this guide covers configuration, request format, and a deep dive on **`initialReviewCompleted`**.

## Configuration

| Method | Where |
|--------|--------|
| **Onboarding page / organisation binding** | Cleared Verification portal → Webhooks (or onboarding page webhook settings) |
| **Event filter** | Optional `subscribedEvents` on each binding — omit or leave unset to receive **all** catalogue events; use a string array to whitelist specific camelCase event names |
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
| `eventName` | Catalogue id (camelCase), e.g. `identityVerificationCleared`, `initialReviewCompleted` |
| `eventContext` | Event-specific object (see deep dive below) |
| `eventOccurredAt` | ISO 8601 timestamp |

#### `verifications[]` — identity row fields

When a row has `type: "identity"`, Cleared includes status fields plus person identifiers resolved from the latest identity result (and tax number fallbacks) at **delivery** time:

| Field | When present | Notes |
|-------|----------------|-------|
| `documentType` | When known on the identity result | e.g. passport, driver licence |
| `clearedAt` / `expiresAt` | When known | ISO 8601 |
| `taxNumber` | When a 9-digit Jamaica TRN can be resolved | From identity TRN metadata, else the user’s unique id, else a TRN verification result |
| `idNumber` | When known | Document / national ID number (not the tax number) |
| `dateOfBirth` | When known | `YYYY-MM-DD` from the identity biography |
| `initialReviewStatus` | Always on identity rows | `completed` or `pending` |

TRN verification rows (`type: "trn"`) also expose `taxNumber` when available.

#### `onboarding.urlParameters`

Configured landing / link query keys captured when the subject starts the flow. Empty maps are omitted. Progress events such as `identityVerificationStarted` are pinned to the **flow** verification request when the client sends that request id, so the first webhook can include these parameters for the current visit.

**Verification:** recompute HMAC-SHA256 on the raw request body and compare with `X-Webhook-Signature` (value after `sha256=`). If invalid, respond with 401.

## Event catalogue (selected)

All onboarding event names use **camelCase**. Below are common decision / IR events; the portal catalogue is the authoritative list for the full set.

| Event | Category | When Cleared sends it |
|-------|----------|------------------------|
| `identityVerificationStarted` | progress | Customer starts identity capture for a flow request (pinned to that request so `onboarding.urlParameters` match the visit) |
| `identityVerificationSubmitted` | progress | Identity documents submitted |
| `identityResubmissionRequested` | progress | Cleared asks the customer to resubmit identity photos or documents for a verification (reviewer or automated review). Sent to every client the verification result is shared with that has an onboarding webhook configured. |
| `identityResubmissionReminderSent` | progress | Cleared sends the customer a reminder about an outstanding identity resubmission request (separate from the original request). Same audience as `identityResubmissionRequested`. |
| `initialReviewCompleted` | decisions | Cleared completes **Initial Review** (triage advance or legacy mark-complete). **Not** sent on IR reject. **Does not** start due diligence. |
| `dueDiligenceStarted` | decisions | Cleared starts **due diligence** after Initial Review (separate Start due diligence action). |
| `identityVerificationCleared` | decisions | Identity cleared (or ensured on consent share of an already-cleared result) |
| `identityVerificationRejected` | decisions | Identity rejected |
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
| `source` | `ops_initial_review_triage_advance` (triage advance) or `ops_initial_review_complete` (legacy mark-complete) |
| `level` | `1` \| `2` \| `3`, or `null` (legacy mark-complete) |
| `riskCategory` | Derived: `1→low`, `2→medium`, `3→high`, else `null` |
| `caseStatus` | Actual identity result **status** after advance (`processing` / `flagged`, etc.) |
| `nextCaseStatus` | Triage choice when available (`continue_review`, `awaiting_resubmission`, …). **Not** `due_diligence` — that is a separate Start due diligence action. |
| `resubmissionRequired` | boolean — when `true`, Cleared will contact the customer to resubmit; use **`identityResubmissionRequested`** (and reminders) for the client-safe resubmission signal |
| `reviewedAt` | ISO string |
| `finalStatus` | Always `"pending"` |
| `reviewDurationMs` | number or `null` |
| `taxNumber` | When resolvable — same rules as the identity `verifications[]` row |
| `idNumber` | When known on the identity result biography / document info |
| `dateOfBirth` | When known — `YYYY-MM-DD` |

Webhook payloads include the fields documented above. Person identifiers appear only as the dedicated `idNumber` / `taxNumber` / `dateOfBirth` fields (not as free-form notes). `initialReviewCompleted` does **not** include reason codes, missing-media labels, internal request ids, or upload links.

### Example `eventContext` (triage advance)

```json
{
  "resultType": "IdentityVerificationResult",
  "verificationType": "identity",
  "resultId": "64f1a2b3c4d5e6f7a8b9c0d1",
  "source": "ops_initial_review_triage_advance",
  "level": 2,
  "riskCategory": "medium",
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

Emitted when Cleared **clears** identity (final clear path). The VerificationStatus envelope’s identity row and `eventContext` both carry person identifiers when available.

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

| Review action | Client onboarding event |
|---------------------|-------------------------|
| claim / assign / release / start / reassign | *(none — no client webhook)* |
| advance (`continue_review` / `awaiting_resubmission`) | `initialReviewCompleted` |
| start due diligence (post-IR) | `dueDiligenceStarted` |
| reject | **No** `initialReviewCompleted` — identity owns `identityVerificationRejected` |

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

## Deep dive: `identityResubmissionRequested`

Emitted when Cleared asks the customer to resubmit identity photos or documents, whether the request comes from a Cleared reviewer or from automated review. Cleared delivers the event to every client that shares the verification result and has an onboarding webhook configured. Use this event (not internal ops identifiers) to mark your application as awaiting customer re-upload.

`eventContext.finalStatus` is always `"pending"`.

### `eventContext` fields

| Field | Rule |
|-------|------|
| `resultType` / `verificationType` / `resultId` | Always present |
| `requestedAt` | ISO string — when Cleared requested resubmission |
| `finalStatus` | Always `"pending"` |
| `taxNumber` | When resolvable — same rules as the identity `verifications[]` row |
| `idNumber` | When known on the identity result biography / document info |
| `dateOfBirth` | When known — `YYYY-MM-DD` |

Payloads do **not** include which captures are missing, reasons, notes, upload links, or who initiated the request.

### Example payload

```json
{
  "topic": "VerificationStatus",
  "customerName": "Jane Doe",
  "customerEmailAddress": "jane@example.com",
  "verifications": [
    {
      "type": "identity",
      "status": "processing",
      "documentType": "passport",
      "initialReviewStatus": "completed",
      "taxNumber": "123456789",
      "idNumber": "A1234567",
      "dateOfBirth": "1990-05-15"
    }
  ],
  "onboarding": {
    "pageId": "page_abc",
    "onboardingPageId": "page_abc",
    "urlParameters": { "applicationId": "LOAN-10042" }
  },
  "eventName": "identityResubmissionRequested",
  "eventContext": {
    "resultType": "IdentityVerificationResult",
    "verificationType": "identity",
    "resultId": "64f1a2b3c4d5e6f7a8b9c0d1",
    "requestedAt": "2026-07-26T09:00:00.000Z",
    "finalStatus": "pending",
    "taxNumber": "123456789",
    "idNumber": "A1234567",
    "dateOfBirth": "1990-05-15"
  },
  "eventOccurredAt": "2026-07-26T09:00:00.000Z"
}
```

## Deep dive: `identityResubmissionReminderSent`

Emitted **each time** Cleared sends the customer a reminder about an outstanding identity resubmission request. This is separate from the original `identityResubmissionRequested` delivery. Audience and envelope match the request event.

`eventContext.finalStatus` is always `"pending"`.

### `eventContext` fields

| Field | Rule |
|-------|------|
| `resultType` / `verificationType` / `resultId` | Always present |
| `reminderNumber` | Positive integer — sequence for this resubmission request (`1` = first reminder after the original request) |
| `reminderSentAt` | ISO string — when Cleared sent this reminder |
| `finalStatus` | Always `"pending"` |
| `taxNumber` | When resolvable — same rules as the identity `verifications[]` row |
| `idNumber` | When known on the identity result biography / document info |
| `dateOfBirth` | When known — `YYYY-MM-DD` |

Reminders do **not** include missing-media labels, reasons, notes, or retake links.

### Example payload

```json
{
  "topic": "VerificationStatus",
  "customerName": "Jane Doe",
  "customerEmailAddress": "jane@example.com",
  "verifications": [
    {
      "type": "identity",
      "status": "processing",
      "documentType": "passport",
      "initialReviewStatus": "completed"
    }
  ],
  "onboarding": {
    "pageId": "page_abc",
    "onboardingPageId": "page_abc",
    "urlParameters": { "applicationId": "LOAN-10042" }
  },
  "eventName": "identityResubmissionReminderSent",
  "eventContext": {
    "resultType": "IdentityVerificationResult",
    "verificationType": "identity",
    "resultId": "64f1a2b3c4d5e6f7a8b9c0d1",
    "reminderNumber": 2,
    "reminderSentAt": "2026-07-28T14:00:00.000Z",
    "finalStatus": "pending"
  },
  "eventOccurredAt": "2026-07-28T14:00:00.000Z"
}
```

## Related

- [Client loan / onboarding workflow](../client-loan-application-workflow.md) — end-to-end IDV + webhooks
- [Document signing webhooks](../document-signatures/document-webhooks.md) — snake_case signing events
- [API endpoints overview](../endpoints.md) — webhooks section

## Changelog

### November 2026

- Documented onboarding webhook events **`identityResubmissionRequested`** and **`identityResubmissionReminderSent`** (catalogue, `eventContext`, and examples).
- Aligned **`initialReviewCompleted`** client docs: removed `reasonCodes` and the `resubmission` object; kept `resubmissionRequired` and pointed integrators to **`identityResubmissionRequested`**.
