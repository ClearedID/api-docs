# ABC Company LLC – Cleared Workflow Documentation

This document describes the end-to-end workflows for the two Cleared features used by the ABC Company LLC: **Identity Verification** via onboarding pages and **Digital Signatures** via document templates. It is intended for both business stakeholders and technical integrators (ABC Company LLC student portal and Loan Origination System).

---
## Table of Contents
1. [Overview and system roles](#overview-and-system-roles)
2. [Identity verification workflow](#identity-verification-workflow)
3. [Digital signatures workflow](#digital-signatures-workflow)
4. [Webhook contracts and security](#webhook-contracts-and-security)
5. [Failure handling and retries](#failure-handling-and-retries)
6. [Reference: key endpoints and files](#reference-key-endpoints-and-files)

---
## Overview and system roles
| System | Role |
|--------|------|
| **ABC Company LLC student portal** | Initiates login/onboarding; redirects students to Cleared for IDV; can offer document signing entry and phase-1 document upload. |
| **Cleared Screening Portal** | Where ABC Company LLC configures onboarding pages (redirect URL, webhooks, URL parameters) and document templates. |
| **Cleared (Onboarding web app + API)** | Hosts the onboarding wizard and IDV; issues signing links and collects signatures. |
| **Loan Origination System** | Consumes Cleared APIs to create document instances from portal-configured templates and send for signature; receives webhooks for IDV completion and document signing events. |

---
## Identity verification workflow
Student completes identity verification (IDV) as part of ABC Company LLC onboarding. The student starts on the ABC Company LLC portal, is sent to Cleared for IDV, and is returned to a URL configured by ABC Company LLC. ABC Company LLC (or Loan Origination System) receives verification outcomes via webhooks.

### High-level flow
1. Student initiates login (or “Start verification”) on the ABC Company LLC portal.
2. ABC Company LLC redirects the student to Cleared’s onboarding page, optionally passing a **unique reference** (e.g. loan application ID) in the URL.
3. Cleared shows the configured onboarding page (branding, steps) and runs the IDV wizard.
4. Student completes IDV (and any other configured verification steps).
5. Cleared redirects the student back to the **redirect URL** configured on the onboarding page, with the same URL parameters (e.g. application ID) preserved.
6. Cleared sends **webhooks** to ABC Company LLC/Loan Origination System for verification lifecycle events (e.g. request created, approved, identity cleared).

### Configuration in Cleared Screening Portal
ABC Company LLC configures each onboarding page in the **Cleared Screening Portal**:

- **Redirect (destination)**
  - **Destination URL**: Where the student is sent after completing all steps (e.g. `https://abccompanyllc.com/portal/onboarding/complete`).
  - **Completion behavior**: `redirect`, `callback`, or `both`.
  - **Auto-redirect**: Whether to redirect immediately when all steps are complete.

- **URL parameters**
  - List of `{ key, description }` (e.g. `applicationId`).
  - Values are taken from the **current browser URL** when the student finishes; they are appended to the destination URL.
  - Use this to pass the **unique reference** (e.g. loan application ID) from ABC Company LLC into Cleared and back to ABC Company LLC on redirect.

- **Webhook**
  - **Enable webhook on completion**: ON.
  - **Webhook URL**: ABC Company LLC/Loan Origination System endpoint that will receive events (e.g. `https://api.los.example.com/webhooks/cleared-verification`).
  - **Webhook secret**: Shared secret used for HMAC signing (see [Webhook contracts and security](#webhook-contracts-and-security)).
  - **Secret format**: e.g. `Authorization: Bearer {secret}`, `Authorization: {secret}`, or a custom header name.

In the Cleared Screening Portal, admins set the page redirect destination, completion behavior, URL parameters to pass back, and webhook configuration for events.

### Unique reference (student / application identification)
- **Passing into Cleared**
  ABC Company LLC builds the onboarding link with query parameters, e.g.
  `https://cleared.id/pages/<pageId>?applicationId=APP-12345`
  The student (or ABC Company LLC) opens this URL; the onboarding app sends these to the backend as `urlParamValues` when calling the route endpoint.

- **Backend**
  When the student hits the start/route, the frontend calls
  `POST /cleared/onboarding/:pageId/route`
  with optional `flowRequestId` and `urlParamValues` (key–value from the current URL). The backend creates or links the verification request and returns `flowStateId` and `flowRequestId`. These identify the session/request in Cleared.

- **In webhooks**
  The webhook payload includes an `onboarding` block with `pageId`, `onboardingPageId`, and `urlParameters`. The `urlParameters` are taken from the verification request’s stored `meta.urlParamValues` (the keys and values passed when the student started the flow). ABC Company LLC/Loan Origination System can use this to tie the event to the correct student/application (e.g. `applicationId`).

- **On redirect back**
  When the student completes all steps, Cleared redirects to the configured **destination** URL and appends the same URL parameter keys with values taken from the current page URL (e.g. `applicationId=APP-12345`). So the unique reference is preserved end-to-end.

### Identity verification steps
- The onboarding wizard includes an **identity** verification step.
- The IDV step uses `GET /cleared/identity/random-string` and submits via `POST /cleared/identity/submit?flowStateId=...` (with `randomString` and document/selfie data).
- Other steps (address, employment, documents to sign, etc.) are configurable on the onboarding page.

### Redirect back to ABC Company LLC
After all steps (and any consent to share results), when the user clicks the final “Continue” or equivalent:
- The app reads `metadata.destination` and `metadata.urlParameters`.
- It builds the final URL: `destination` plus query parameters from the current URL for each `urlParameters[].key`.
- It performs `window.location.href = url.toString()` so the student is sent back to ABC Company LLC (e.g. `https://abccompanyllc.com/portal/onboarding/complete?applicationId=APP-12345`).

---
## Digital signatures workflow
ABC Company LLC maintains **document templates** in Cleared (one per document type). The Loan Origination System (or ABC Company LLC) **instantiates** a template to create a document, assigns the student as signer, and **sends** the document. The student receives an invitation, signs (and can upload attachments in phase 1). When the document is fully signed, Cleared applies a digital seal, sends the signed document to the student, and notifies the ABC Company LLC administrator (client). Optional **outbound webhooks** can notify the Loan Origination System on each sign event.

### High-level flow
1. **ABC Company LLC** administers document templates in the Cleared Screening Portal.
2. **Loan Origination System** (or ABC Company LLC), for a given loan application, calls the **instantiate** API with the template ID and the **signing parties** (student name and email for each role).
3. Loan Origination System then calls the **send** API for the created document (with expiration, optional message). Cleared enqueues the document; **Cleared processing** sends **invitation emails** to signers with signing links.
4. **Student** opens the link, optionally completes IDV if required, fills/signs fields, and optionally uploads attachments (`uploadedAttachments`). On submit, Cleared updates the document and, if a **document webhook** is configured, POSTs a `document_signed` event to the Loan Origination System.
5. When **all** signers have signed, the document status becomes `ready_for_digital_signature`. **Cleared processing** applies the digital signature (eSeal), then:
   - Sends **completion emails** to signers (with link to the signed PDF).
   - Sends **push + email** to the **ABC Company LLC administrator** (client) with **view** and **download** links (7-day expiry) and a link to the Documents Dashboard.
6. **Phase 1 option**: Student may also upload a document via the ABC Company LLC portal; that flow is separate (e.g. upload into the loan application); Cleared supports **attachment requirements** on documents so the student can upload files at signing time.

### Maintaining templates (ABC Company LLC)
- Templates are created and edited in the **Cleared Screening Portal** (Documents → Templates).
- Each template has: **roles** (e.g. “Borrower”), **fields** (signature, text, date, etc.) assigned to roles, and **configuration** (e.g. use digital signature, expiration).
- ABC Company LLC keeps one template per document type (e.g. loan agreement, promissory note). The Loan Origination System stores the **template ID** for each type and uses it when creating a document for a loan application.

### Instantiating a document (Loan Origination System)
- **Endpoint**: `POST /api/v1/merchant/signatures/templates/:templateId/instantiate`
- **Body**: `title` (optional), `signingParties`: array of `{ id, roleId, name, email, role, order, ... }`.
- **Response**: `documentId` and the created document (status `draft`). Template roles are mapped to the provided signers; fields are assigned to those signers.

Example: one signer (student) for role “Borrower”:
```json
{
  "title": "Loan Agreement - Jane Smith",
  "signingParties": [
    {
      "id": "signer_1",
      "roleId": "role_borrower",
      "name": "Jane Smith",
      "email": "jane.smith@example.com",
      "role": "Borrower",
      "order": 1
    }
  ]
}
```

### Sending the document (Loan Origination System)
- **Endpoint**: `POST /api/v1/merchant/signatures/documents/:documentId/send`
- **Body**: `signingParties` (name, email, role, order, requireIdVerification, etc.), `configuration` (e.g. `useDigitalSignature`, `expiration`, `enforceSigningOrder`), optional `message`.
- **Effect**: Document status becomes `enqueued`. Credits are deducted. Cleared processing later picks it up, sends **invitation emails** to signers with signing links, and sets status to `pending`.

### Student signs and optional attachments (phase 1)
- **Endpoint** (public): `POST /api/v1/public/signatures/documents/sign`
- **Body**: `documentId`, `signerId`, `fieldValues` (field id → value, e.g. signature image, date), and optionally **`uploadedAttachments`**.
- **`uploadedAttachments`**: Object keyed by **attachment requirement ID** (from the document’s `attachmentRequirements`). Each value is an array of `{ url, name, size, type }` (files already uploaded to storage; the signing UI uploads first then passes references). These are stored on the document as `signerAttachments`.

So for “phase 1” document upload by the student: either the student uploads a file in the ABC Company LLC portal (handled by ABC Company LLC/Loan Origination System) or the document is configured with **attachment requirements** in Cleared and the student attaches files during the signing step; Cleared persists them as part of the signed document.

### Document signing webhook (to Loan Origination System)
If the **document** has `webhookConfig.url` and `webhookConfig.active`, the gateway sends a **POST** to that URL when a signer **submits** their signature (each signer submission triggers one webhook).

- **Headers**: `Content-Type: application/json`, `X-Webhook-Signature: sha256=<hex>` (HMAC-SHA256 of the **raw JSON body** using `webhookConfig.secret`).
- **Body** (example):
```json
{
  "event": "document_signed",
  "documentId": "507f1f77bcf86cd799439011",
  "signerId": "507f1f77bcf86cd799439012",
  "signerName": "Jane Smith",
  "signerEmail": "jane.smith@example.com",
  "signedAt": "2025-10-19T12:00:00.000Z",
  "allSigned": false,
  "documentStatus": "pending"
}
```

When the last signer signs, `allSigned` is `true` and `documentStatus` may be `ready_for_digital_signature` (or `completed` if digital signature is not used). Loan Origination System can use this to update the loan application (e.g. “document signed” or “all signed, awaiting seal”).

### Document completion: signer and ABC Company LLC administrator notifications
After all signers have signed:
1. If the document uses **digital signature**, status becomes `ready_for_digital_signature`; Cleared processing runs the eSeal process, then sets status to `completed`.
2. **Signers**: Cleared sends each signer a **completion email** with a link to view/download the signed PDF (e.g. 7-day expiry).
3. **ABC Company LLC administrator (client)**: Cleared sends:
   - A **push notification** (in-app),
   - An **email** with **view** and **download** links (signed PDF, 7-day expiry) and a link to the **Documents Dashboard** (`MERCHANT_PORTAL_URL/documents?id=<documentId>`).

So the “document gets sent to ABC Company LLC administrator” is implemented as this **email + push** with view/download links and dashboard link; there is no separate “forward document” API.

---
## Webhook contracts and security

### Onboarding (IDV) webhook
- **When**: After verification request creation (with onboarding meta) and when events are marked pending (e.g. customer approve, ops decision: identity/address/reference cleared or rejected). A batch job sends pending events to the configured URL.
- **Method**: POST.
- **Headers**: `Content-Type: application/json`. If a secret is configured: `Authorization` (or custom header) per secret format, and **`X-Webhook-Signature: sha256=<hex>`** where the HMAC-SHA256 is computed over the **raw JSON body string** (canonical) using the webhook secret.
- **Body** (VerificationStatus style): includes `customerName`, `verifications` (array with `type`, `status`, and type-specific fields such as identity `documentType`, `clearedAt`, `expiresAt`), and **`onboarding`**: `{ pageId, onboardingPageId, urlParameters }`, plus **`eventName`** (e.g. `verificationRequestApproved`, `identityVerificationCleared`, `identityVerificationRejected`; default `verificationStatusUpdated`), **`eventContext`**, **`eventOccurredAt`** (ISO).

**Verification**: Recipient should recompute HMAC-SHA256 on the raw request body and compare with `X-Webhook-Signature` (value after `sha256=`). If invalid, respond with 401.

### Document signing webhook
- **When**: On each signer submission (if `document.webhookConfig.url` and `document.webhookConfig.active`).
- **Method**: POST.
- **Headers**: `Content-Type: application/json`, **`X-Webhook-Signature: sha256=<hex>`** (HMAC-SHA256 of the **raw JSON body** with `webhookConfig.secret`).
- **Body**: `event`, `documentId`, `signerId`, `signerName`, `signerEmail`, `signedAt`, `allSigned`, `documentStatus` (see example above).

**Verification**: Same as onboarding—recompute HMAC on the raw body and compare to the header.

---
## Failure handling and retries

### Onboarding webhooks
- **One attempt per event**: Each `WebHookEvent` is sent once by the batch job. On success (2xx), state is set to `success`. On failure (non-2xx, timeout, or error), state is set to `failed` and the run is recorded in `lastRuns`.
- **No automatic retry**: The same event is not retried. A **new** event is created when a new occurrence happens (e.g. another approval or ops decision). ABC Company LLC/Loan Origination System should implement the endpoint to be **idempotent** (e.g. by `eventOccurredAt` + `onboarding.urlParameters.applicationId`) and return 2xx quickly.

### Verification Links webhooks (if used)
If ABC Company LLC uses Verification Links instead of (or in addition to) onboarding pages, the documented retry policy for that product is: immediate retry after 5 s, second after 1 min, third after 5 min, then mark failed.

### Document signing webhook
- The gateway fires the webhook once per sign submission. There is no built-in retry documented for this outbound call; implement the receiver to be idempotent and return 2xx.

---
## Reference: key endpoints and files
| Area | Endpoint / file | Purpose |
|------|------------------|--------|
| Onboarding route | `POST /cleared/onboarding/:pageId/route` | Start/link session (urlParamValues, flowStateId, flowRequestId) |
| Document instantiation (workflow) | `POST /api/v1/merchant/signatures/templates/:templateId/instantiate` | Create a document instance from a portal-configured template |
| Documents | `POST /api/v1/merchant/signatures/documents/:documentId/send` | Send document for signing |
| Public sign | `POST /api/v1/public/signatures/documents/sign` | Submit signature (fieldValues, uploadedAttachments) |

