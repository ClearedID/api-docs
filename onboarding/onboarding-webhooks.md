# Onboarding (IDV) webhooks

Receive **HTTP POST** notifications when onboarding / verification lifecycle events are delivered for a subscribed organisation. Events use **camelCase** ids (unlike document signing webhooks, which use snake_case).

For the full live list, call the portal **event-catalog** API; this guide covers configuration, request format, and deep dives on **`initialReviewCompleted`** (including automated Initial Review) and the **Due Diligence** case stage.

## Configuration

| Method | Where |
|--------|--------|
| **Onboarding page / organisation binding** | Cleared Screening Portal → Webhooks (or onboarding page webhook settings) |
| **Event filter** | Optional `subscribedEvents` on each binding — omit or leave unset to receive **all** catalog events; use a string array to whitelist specific camelCase event names |
| **Secret** | Per-endpoint secret used for HMAC (`X-Webhook-Signature`) |

Each binding targets one HTTPS URL. Multiple bindings may fire for the same event.

Subscribe to **`initialReviewCompleted`** (or leave `subscribedEvents` unset) to receive Initial Review and Due Diligence stage notifications described below.

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
| `verifications` | Array of verification rows (`type`, `status`, `initialReviewStatus`, type-specific fields) |
| `onboarding` | `{ pageId, onboardingPageId, urlParameters }` when the flow came from an onboarding page |
| `eventName` | Catalog id (camelCase), e.g. `identityVerificationCleared`, `initialReviewCompleted` |
| `eventContext` | Event-specific object (see deep dives below) |
| `eventOccurredAt` | ISO 8601 timestamp |

**Verification:** recompute HMAC-SHA256 on the raw request body and compare with `X-Webhook-Signature` (value after `sha256=`). If invalid, respond with 401.

### Identity row in `verifications[]`

For `type: "identity"`, clients commonly use:

| Field | Meaning |
|-------|---------|
| `status` | Current identity result status (`submitted`, `processing`, `flagged`, `due_diligence`, `awaiting_resubmission`, `cleared`, `rejected`, …) |
| `initialReviewStatus` | `"completed"` after Initial Review has finished; otherwise `"pending"` |
| `documentType` / `clearedAt` / `expiresAt` / `taxNumber` | Present when available on the result |

**Important:** `initialReviewStatus: "completed"` means Initial Review finished — **not** that identity is cleared. Final outcomes arrive as `identityVerificationCleared` or `identityVerificationRejected`.

## Event catalog (selected)

All onboarding event names use **camelCase**. Below are common decision and Initial Review events; the portal catalog is authoritative for the full set.

| Event | Category | When Cleared sends it |
|-------|----------|------------------------|
| `identityVerificationSubmitted` | submissions | Identity documents submitted |
| `initialReviewCompleted` | decisions | Initial Review finished (manual review or automated high-confidence pass after submit). **Not** sent when identity is rejected at Initial Review. Also the signal when the case moves into **Due Diligence** (see below). |
| `identityVerificationCleared` | decisions | Identity cleared |
| `identityVerificationRejected` | decisions | Identity rejected |
| `identityVerificationApproved` | decisions | Customer / share approval path for identity |
| `verificationRequestApproved` | session | Verification request approved for share |
| `verificationStatusUpdated` | progress | Fallback / status refresh |

For signing lifecycle events (`document_signed`, etc.), see [Document signing webhooks](../document-signatures/document-webhooks.md).

## Typical identity lifecycle

```text
identityVerificationSubmitted
        │
        ▼
initialReviewCompleted          ← Initial Review done (manual or automated). finalStatus always "pending"
        │
        ├─ continue_review / processing / flagged  → further review
        ├─ due_diligence                           → Due Diligence stage (same event; see below)
        └─ awaiting_resubmission                   → applicant must resubmit
        │
        ▼
identityVerificationCleared  OR  identityVerificationRejected
```

There is **no** separate catalog event named `dueDiligence*`. Entry into Due Diligence is communicated on **`initialReviewCompleted`** with Due Diligence status fields.

---

## Deep dive: `initialReviewCompleted`

Emitted when Initial Review is complete, while final clear/reject may still happen later. `eventContext.finalStatus` is always `"pending"`.

### When it fires

| Trigger | `eventContext.source` | Typical `nextCaseStatus` / `caseStatus` |
|---------|----------------------|----------------------------------------|
| Manual Initial Review advance | `ops_initial_review_triage_advance` | `continue_review`, `due_diligence`, `awaiting_resubmission`, … |
| Automated high-confidence pass after submit | `image_check_auto_ir` | Usually `continue_review` → identity `status` `processing` |
| Initial Review completed (other completion path) | `ops_initial_review_complete` | As applicable; `level` may be `null` |

**Automated pass:** After identity is submitted, Cleared may complete Initial Review automatically when submitted evidence meets Cleared’s high-confidence checks. You still receive **`initialReviewCompleted`**. Treat `source: "image_check_auto_ir"` like any other Initial Review completion — do **not** treat it as cleared.

**Rejection at Initial Review:** Cleared does **not** send `initialReviewCompleted`. Expect `identityVerificationRejected`.

### `eventContext` fields

| Field | Rule |
|-------|------|
| `resultType` / `verificationType` / `resultId` | Always present for Initial Review |
| `source` | See table above |
| `level` | `1` \| `2` \| `3`, or `null`. Automated pass uses `1`. |
| `riskCategory` | Derived: `1→low`, `2→medium`, `3→high`, else `null` |
| `reasonCodes` | string[] (may be empty) |
| `caseStatus` | Actual identity result **status** after Initial Review (`processing` / `flagged` / `due_diligence` / `awaiting_resubmission`, etc.) |
| `nextCaseStatus` | Workflow choice when available (`continue_review`, `due_diligence`, `awaiting_resubmission`, …). For `continue_review`, `caseStatus` is typically `processing`. |
| `resubmissionRequired` | boolean |
| `resubmission` | Present when required: `{ requestId, labels? }` — omitted when not required |
| `reviewedAt` | ISO string |
| `finalStatus` | Always `"pending"` |
| `reviewDurationMs` | number or `null` |

Internal review notes and full document numbers are **not** included beyond what the VerificationStatus envelope already exposes.

### Example — automated Initial Review pass (`image_check_auto_ir`)

```json
{
  "topic": "VerificationStatus",
  "eventName": "initialReviewCompleted",
  "eventOccurredAt": "2026-07-25T05:12:00.000Z",
  "customerName": "Jane Example",
  "customerEmailAddress": "jane.example@example.com",
  "onboarding": {
    "pageId": "acme-onboarding",
    "onboardingPageId": "64f1c0a2e4b0a1b2c3d4e510",
    "urlParameters": { "applicationId": "APP-10042" }
  },
  "verifications": [
    {
      "type": "identity",
      "status": "processing",
      "documentType": "drivers_licence",
      "initialReviewStatus": "completed"
    }
  ],
  "eventContext": {
    "resultType": "IdentityVerificationResult",
    "verificationType": "identity",
    "resultId": "64f1a2b3c4d5e6f7a8b9c0d1",
    "source": "image_check_auto_ir",
    "level": 1,
    "riskCategory": "low",
    "reasonCodes": [],
    "caseStatus": "processing",
    "nextCaseStatus": "continue_review",
    "resubmissionRequired": false,
    "reviewedAt": "2026-07-25T05:12:00.000Z",
    "finalStatus": "pending",
    "reviewDurationMs": null
  }
}
```

### Example — manual Initial Review (`continue_review`)

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
  "reviewDurationMs": 180000
}
```

### How to consume (recommended)

1. Match the person/application via `onboarding.urlParameters` (and/or customer email).
2. Branch on `eventName === "initialReviewCompleted"`.
3. Optionally read **`eventContext.source`** to distinguish manual vs automated Initial Review.
4. Read **`eventContext.caseStatus`** / **`nextCaseStatus`** for the workflow stage (see Due Diligence below).
5. Confirm the identity row: `initialReviewStatus === "completed"` and `status` is **not** yet `cleared` / `rejected`.
6. Keep the application in a **pending / in-review** state until you receive `identityVerificationCleared` or `identityVerificationRejected`.

---

## Deep dive: Due Diligence stage

**Due Diligence** (`due_diligence`) is an identity case status after Initial Review: extended review is required before a final clear or reject. It is **not** a terminal outcome.

### How you learn the case entered Due Diligence

Cleared sends the same event: **`initialReviewCompleted`**, with:

| Signal | Value |
|--------|--------|
| `eventName` | `initialReviewCompleted` |
| `eventContext.nextCaseStatus` | `"due_diligence"` |
| `eventContext.caseStatus` | `"due_diligence"` |
| `eventContext.finalStatus` | `"pending"` (still not cleared/rejected) |
| Identity `verifications[].status` | `"due_diligence"` |
| Identity `verifications[].initialReviewStatus` | `"completed"` |

`source` is typically `ops_initial_review_triage_advance`. Automated Initial Review pass does **not** place cases in Due Diligence (it completes with `continue_review`).

### Example `eventContext` (Due Diligence)

```json
{
  "resultType": "IdentityVerificationResult",
  "verificationType": "identity",
  "resultId": "64f1a2b3c4d5e6f7a8b9c0d1",
  "source": "ops_initial_review_triage_advance",
  "level": 3,
  "riskCategory": "high",
  "reasonCodes": ["DOCUMENT_SUSPECT"],
  "caseStatus": "due_diligence",
  "nextCaseStatus": "due_diligence",
  "resubmissionRequired": false,
  "reviewedAt": "2026-07-25T06:00:00.000Z",
  "finalStatus": "pending",
  "reviewDurationMs": 420000
}
```

### How to consume

1. On `initialReviewCompleted`, if `eventContext.caseStatus === "due_diligence"` **or** `eventContext.nextCaseStatus === "due_diligence"` (or identity `verifications[].status === "due_diligence"`), mark the application as **Due Diligence / extended review**.
2. Do **not** treat the applicant as cleared or rejected.
3. Wait for a later **`identityVerificationCleared`** or **`identityVerificationRejected`** webhook for the final decision.
4. Resubmission is a separate path: `awaiting_resubmission` on `initialReviewCompleted`.

### Leaving Due Diligence

| Outcome | Client event |
|---------|--------------|
| Cleared | `identityVerificationCleared` |
| Rejected | `identityVerificationRejected` |

No additional `initialReviewCompleted` is required for the final decision.

---

## Event matrix (Initial Review)

| Situation | Client event |
|-----------|--------------|
| Initial Review completed (`continue_review` / `due_diligence` / `awaiting_resubmission`) | `initialReviewCompleted` |
| Automated high-confidence Initial Review after submit | `initialReviewCompleted` (`source: image_check_auto_ir`) |
| Rejected at Initial Review | `identityVerificationRejected` (no `initialReviewCompleted`) |

## Related

- [Client loan / onboarding workflow](../client-loan-application-workflow.md) — end-to-end IDV + webhooks
- [Document signing webhooks](../document-signatures/document-webhooks.md) — snake_case signing events
- [API endpoints overview](../endpoints.md) — webhooks section
