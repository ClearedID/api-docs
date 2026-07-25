# Onboarding (IDV) webhooks

Receive **HTTP POST** notifications when onboarding / verification lifecycle events are delivered for a subscribed organisation. Events use **camelCase** ids (unlike document signing webhooks, which use snake_case).

For the full live list, call the portal **event-catalog** API; this guide covers configuration, request format, and a deep dive on **`initialReviewCompleted`**.

## Configuration

| Method | Where |
|--------|--------|
| **Onboarding page / organisation binding** | Cleared Screening Portal → Webhooks (or onboarding page webhook settings) |
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

**Verification:** recompute HMAC-SHA256 on the raw request body and compare with `X-Webhook-Signature` (value after `sha256=`). If invalid, respond with 401.

## Event catalog (selected)

All onboarding event names use **camelCase**. Below are common decision / IR events; the portal catalog is the SSOT for the full set.

| Event | Category | When Cleared sends it |
|-------|----------|------------------------|
| `initialReviewCompleted` | decisions | Ops completes **Initial Review** (triage advance or legacy mark-complete). **Not** sent on IR reject. |
| `identityVerificationCleared` | decisions | Identity cleared by ops |
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
| `caseStatus` | Actual identity result **status** after advance (`processing` / `flagged` / `due_diligence`, etc.) |
| `nextCaseStatus` | Triage choice when available (`continue_review`, `due_diligence`, `awaiting_resubmission`, …) |
| `resubmissionRequired` | boolean |
| `resubmission` | Present when required: `{ requestId, labels? }` — omitted when not required |
| `reviewedAt` | ISO string |
| `finalStatus` | Always `"pending"` |
| `reviewDurationMs` | number or `null` |

**Never** included: ops `internalNotes`, full document numbers, or other PII beyond what the VerificationStatus envelope already exposes.

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
  "reviewDurationMs": 180000
}
```

## Action → event matrix (Initial Review Triage)

| Ops / triage action | Client onboarding event | Ops PA topic (Teams / Power Automate) |
|---------------------|-------------------------|----------------------------------------|
| claim / assign / release / start / reassign | *(none — audit only)* | *(none)* |
| advance (`continue_review` / `due_diligence` / `awaiting_resubmission`) | `initialReviewCompleted` | `initial_review_completed` |
| reject | **No** `initialReviewCompleted` — identity owns `identityVerificationRejected` | `initial_review_completed` (ops) + later `identity_decided` from identity decision path |

## Related

- [Client loan / onboarding workflow](../client-loan-application-workflow.md) — end-to-end IDV + webhooks
- [Document signing webhooks](../document-signatures/document-webhooks.md) — snake_case signing events
- [API endpoints overview](../endpoints.md) — webhooks section
- Ops engineering runbook (internal): `docs/ops-webhook-publishing.md` — topic `initial_review_completed`
