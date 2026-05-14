# ABC Company LLC – Cleared Workflow Documentation

This document describes the end-to-end workflows for the Cleared features used by the ABC Company LLC: **Identity Verification** via onboarding pages and **Digital Signatures** via **document templates** (single document) or **envelope templates** (multi-document package). It is intended for both business stakeholders and technical integrators (ABC Company LLC student portal and Loan Origination System).

---
## Table of Contents
1. [Overview and system roles](#overview-and-system-roles)
2. [Identity verification workflow](#identity-verification-workflow)
3. [Digital signatures workflow](#digital-signatures-workflow)
4. [Digital signatures: envelope pathway (multi-document)](#digital-signatures-envelope-pathway-multi-document)
5. [Webhook contracts and security](#webhook-contracts-and-security)
6. [Failure handling and retries](#failure-handling-and-retries)
7. [Reference: key endpoints and files](#reference-key-endpoints-and-files)

---
## Overview and system roles
| System | Role |
|--------|------|
| **ABC Company LLC student portal** | Initiates login/onboarding; redirects students to Cleared for IDV; can offer document signing entry and phase-1 document upload. |
| **Cleared Screening Portal** | Where ABC Company LLC configures onboarding pages (redirect URL, webhooks, URL parameters) and document templates. |
| **Cleared (Onboarding web app + API)** | Hosts the onboarding wizard and IDV; issues signing links and collects signatures. |
| **Loan Origination System** | Consumes Cleared APIs to create document instances from portal-configured **document** or **envelope** templates and send for signature; receives webhooks for IDV completion and document signing events. |

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
ABC Company LLC can maintain **document templates** in Cleared (one per document type) and use the flow below for a **single** document. For **loan packages** (agreement + note + disclosure, etc.), ABC Company LLC may instead configure **envelope templates** and use the [envelope pathway](#digital-signatures-envelope-pathway-multi-document)—same signing and webhook behaviour per document, with one envelope send covering all documents.

### High-level flow (single document)
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
- **Body**: `title` (optional), `signingParties`: array of `{ id, roleId, name, email, role, order, ... }`, optional `sendImmediately`, optional `configuration` (e.g. `useDigitalSignature`, `expiration`), and optional top-level **`useDigitalSignature`** (boolean) merged with `configuration` for the created document.
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
## Digital signatures: envelope pathway (multi-document)
When a loan (or other case) requires **several** related documents signed by the same parties, ABC Company LLC configures an **envelope template** in the Cleared Screening Portal (Documents → Templates → Envelope templates). The template lists **document templates** in order and defines roles per document. The Loan Origination System **instantiates** the envelope template (creating one envelope and one **draft** document per included document template), then **sends the envelope** so signers get a unified experience. Per-document **webhooks** and completion behaviour match the single-document flow above.

### High-level flow (envelope)
1. **ABC Company LLC** builds an **envelope template** in the portal (e.g. loan agreement + promissory note + disclosure), each slot pointing at an existing **document template** with roles.
2. **Loan Origination System** stores the **envelope template ID** for that product (parallel to storing per-document template IDs).
3. For a given application, LOS calls **instantiate envelope template** with a display **title**, which documents are **included**, and **signing parties per included document** (names, emails, `roleId` aligned to each document template’s roles where possible).
4. LOS calls **get envelope documents** to read created documents and their IDs (needed for the next step).
5. LOS calls **send envelope** with an optional **message** and **per-document digital signature flags** (`documentConfigurations`). Cleared deducts credits, sets the envelope and documents to **sent** / **enqueued**, and queues invitation emails.
6. **Student** signs each document (same public sign endpoint and webhooks as single-document flow). When all documents are complete, notifications follow the same completion rules as for individual documents.

### Instantiating an envelope from an envelope template (Loan Origination System)
- **Endpoint**: `POST /api/v1/merchant/signatures/envelope-templates/:templateId/instantiate`
- **Body** (gateway contract):
  - **`title`** (string, optional): Used as the envelope **name**; if omitted, the envelope template’s title is used.
  - **`configuration`** (object, optional) and/or **`useDigitalSignature`** (boolean, optional) at **root**: merged into **every** included document’s configuration (on top of each document template’s published configuration).
  - **`sendImmediately`** (boolean, optional): when **`true`**, after documents are created the gateway marks the envelope **sent** and each document **enqueued** (same idea as **`sendImmediately`** on single-document template instantiate). Optional **`message`** is stored on each document, matching **`POST …/signatures/envelopes/:envelopeId/send`**. When **`false`** or omitted, the envelope stays **draft** until you call send separately.
  - **`documentAssignments`** (array, required): One entry per document template in the package you want to drive.
    - **`templateId`**: ID of the **document template** slot (must match a template in the envelope template).
    - **`isIncluded`**: If `true`, a document is created from that template; at least one must be included.
    - **`signingParties`**: Same shape as single-document instantiate—`id`, `name`, `email`, `role`, `order`, optional `roleId` (maps template roles to signers for field placement), `enforceOrder`, `requireIdVerification`, etc.
    - **`configuration`** (object, optional) and/or **`useDigitalSignature`** (boolean, optional) on the **assignment**: merged into **that** document only; overrides the root `configuration` / `useDigitalSignature` for the same keys.

**Response** (success): `data.envelopeId` and `data.envelope` (draft envelope). Document rows are created server-side; use **`GET /api/v1/merchant/signatures/envelopes/:envelopeId/documents`** — response `data` is an **array** of `{ _id, title, status, signingParties, ... }` — to build **`documentConfigurations`** before send.

Example body (borrower only; two documents included, roles must match what each document template defines—use `GET .../envelope-templates/:id` to inspect `documentTemplates` and role ids):

```json
{
  "title": "Loan package – Jane Smith – APP-12345",
  "documentAssignments": [
    {
      "templateId": "507f1f77bcf86cd799439070",
      "isIncluded": true,
      "signingParties": [
        {
          "id": "signer_borrower_1",
          "roleId": "role_borrower",
          "name": "Jane Smith",
          "email": "jane.smith@example.com",
          "role": "Borrower",
          "order": 1
        }
      ]
    },
    {
      "templateId": "507f1f77bcf86cd799439071",
      "isIncluded": true,
      "signingParties": [
        {
          "id": "signer_borrower_1",
          "roleId": "role_borrower",
          "name": "Jane Smith",
          "email": "jane.smith@example.com",
          "role": "Borrower",
          "order": 1
        }
      ]
    }
  ]
}
```

### Sending the envelope (Loan Origination System)
- **Endpoint**: `POST /api/v1/merchant/signatures/envelopes/:envelopeId/send`
- **Body** (gateway contract):
  - **`message`** (string, optional): Shown to signers / stored on the envelope and documents.
  - **`documentConfigurations`** (array, optional): `{ "id": "<documentMongoId>", "useDigitalSignature": true|false }` per document in the envelope. If omitted for a document, the gateway defaults **`useDigitalSignature` to `true`** when calculating credits.

Credits are summed from all documents (e.g. digital vs regular per-document), checked against the organisation balance, then deducted. All documents move to **enqueued** for outbound processing.

### Sample code (Node.js, merchant API)
Replace `BASE_URL`, `MERCHANT_JWT`, template and signer values with your environment. Uses `fetch` and the instantiate → get → send sequence.

```javascript
const BASE_URL = 'https://cleared.id'; // or your Cleared API host
const MERCHANT_JWT = process.env.CLEARED_MERCHANT_JWT; // Bearer token

const envelopeTemplateId = '507f1f77bcf86cd799439090';

// 1) Create draft envelope + documents from envelope template
const instRes = await fetch(
  `${BASE_URL}/api/v1/merchant/signatures/envelope-templates/${envelopeTemplateId}/instantiate`,
  {
    method: 'POST',
    headers: {
      Authorization: `Bearer ${MERCHANT_JWT}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      title: 'Loan package – Jane Smith – APP-12345',
      documentAssignments: [
        {
          templateId: '507f1f77bcf86cd799439070',
          isIncluded: true,
          signingParties: [
            {
              id: 'signer_borrower_1',
              roleId: 'role_borrower',
              name: 'Jane Smith',
              email: 'jane.smith@example.com',
              role: 'Borrower',
              order: 1
            }
          ]
        },
        {
          templateId: '507f1f77bcf86cd799439071',
          isIncluded: true,
          signingParties: [
            {
              id: 'signer_borrower_1',
              roleId: 'role_borrower',
              name: 'Jane Smith',
              email: 'jane.smith@example.com',
              role: 'Borrower',
              order: 1
            }
          ]
        }
      ]
    })
  }
);
const instJson = await instRes.json();
if (!instRes.ok || !instJson.success) {
  throw new Error(instJson.message || 'Instantiate failed');
}
const envelopeId = instJson.data.envelopeId;

// 2) Load envelope documents to obtain Mongo IDs for send()
const getRes = await fetch(
  `${BASE_URL}/api/v1/merchant/signatures/envelopes/${envelopeId}/documents`,
  {
    headers: { Authorization: `Bearer ${MERCHANT_JWT}` }
  }
);
const getJson = await getRes.json();
if (!getRes.ok || !getJson.success) {
  throw new Error(getJson.message || 'Get envelope documents failed');
}
const docs = Array.isArray(getJson.data) ? getJson.data : [];

const documentConfigurations = docs.map((d) => ({
  id: d._id,
  useDigitalSignature: true // set false where you want regular signature pricing
}));

// 3) Send entire envelope (invitations for all documents)
const sendRes = await fetch(
  `${BASE_URL}/api/v1/merchant/signatures/envelopes/${envelopeId}/send`,
  {
    method: 'POST',
    headers: {
      Authorization: `Bearer ${MERCHANT_JWT}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      message: 'Please review and sign your loan documents.',
      documentConfigurations
    })
  }
);
const sendJson = await sendRes.json();
if (!sendRes.ok || !sendJson.success) {
  throw new Error(sendJson.message || 'Send envelope failed');
}
console.log('Envelope sent:', sendJson.data);
```

### Webhooks and student experience (envelope)
- Each **document** in the envelope can still have **`webhookConfig`**; signer submissions continue to POST **`document_signed`** events per document (see [Digital signatures workflow](#digital-signatures-workflow)).
- Signers typically receive coordinated invitations for the package; tracking at envelope level is available via `GET /api/v1/merchant/signatures/envelopes/:envelopeId` (see [Envelopes API](./document-signatures/envelopes.md)).

### Further reading
- [Envelope templates API](./document-signatures/envelope-templates.md) — list, get, create, save, duplicate, **instantiate**, delete.
- [Envelopes API](./document-signatures/envelopes.md) — list, get, **send**, cancel, history, and related operations.

**Note:** Some older API reference examples may show alternate request shapes for instantiate/send; the **title** + **documentAssignments** instantiate body and **message** + **documentConfigurations** send body above match the current **swf-core-gateway** merchant routes.

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
| Envelope template → envelope | `POST /api/v1/merchant/signatures/envelope-templates/:templateId/instantiate` | Create draft envelope and documents from an envelope template (`title`, `documentAssignments`) |
| Envelopes | `GET /api/v1/merchant/signatures/envelopes/:envelopeId/documents` | List documents in the envelope (`data` is an array; use each `_id` in `documentConfigurations`) |
| Envelopes | `GET /api/v1/merchant/signatures/envelopes/:envelopeId` | Read envelope metadata (status, counts) |
| Envelopes | `POST /api/v1/merchant/signatures/envelopes/:envelopeId/send` | Send all documents in the envelope (`message`, `documentConfigurations`) |
| Public sign | `POST /api/v1/public/signatures/documents/sign` | Submit signature (fieldValues, uploadedAttachments) |

