# Public Signer API

## Overview

Endpoints used by the Cleared signer app when a recipient opens a signing link from email (or when your integration embeds signing). **No merchant JWT** — access is via a short-lived **signing token** in the URL.

**Base Path**: `/api/v1/public/signatures`

**Authentication**: Query/body `token` (JWT) scoped to `documentId` + `signerId`.

---

## Envelope signing behaviour

When a document belongs to an **envelope**:

1. **Invitation email** — One email per signer listing all documents they must sign, with a **single link** to the first pending document (not one email per document).
2. **After each signature** — The signer app **automatically advances** to the next unsigned document for that signer (via `signingContext.nextSigning` from the sign response).
3. **Revisiting an old link** — If the linked document is already signed by that signer, the load endpoint returns **`redirectTo`** (next unsigned doc) or **`envelopeComplete`** (all done), instead of an error.

---

## 1. Load document for signing

**Endpoint**: `GET /api/v1/public/signatures/documents/:documentId/get-document-for-signing`

**Query parameters**:
- `signerId` (string, required)
- `token` (string, required) — Signing access JWT from the invitation link

**Success — ready to sign** (200):
```json
{
  "success": true,
  "data": {
    "document": { "...": "..." },
    "pageImages": [ "..." ],
    "signer": { "...": "..." },
    "signingContext": {
      "hasEnvelope": true,
      "envelopeId": "507f1f77bcf86cd799439040",
      "hasMoreDocumentsForSigner": true,
      "nextSigning": {
        "documentId": "507f1f77bcf86cd799439012",
        "signerId": "507f1f77bcf86cd799439081",
        "token": "...",
        "signingUrl": "https://cleared.id/sign/flow?documentId=...&signerId=...&token=..."
      }
    }
  }
}
```

**Already signed — envelope has more documents** (200):
```json
{
  "success": true,
  "alreadySigned": true,
  "redirectTo": {
    "documentId": "507f1f77bcf86cd799439012",
    "signerId": "507f1f77bcf86cd799439081",
    "token": "...",
    "signingUrl": "https://cleared.id/sign/flow?..."
  },
  "signingContext": { "...": "..." },
  "message": "This document is already signed. Redirecting to your next document."
}
```

**Already signed — all envelope documents complete** (200):
```json
{
  "success": true,
  "alreadySigned": true,
  "envelopeComplete": true,
  "signingContext": { "...": "..." },
  "message": "You have completed all documents in this envelope."
}
```

**Already signed — ID verification still required** (200): Same as before — minimal payload with `idVerificationRequired` / `alreadySigned`; no redirect until IDV is cleared.

**Notes**:
- Signer clients should follow `redirectTo.signingUrl` (or `signingContext.nextSigning.signingUrl`) automatically when present.
- `signingContext.nextSigning` on a normal load describes the **next** document after the current one (used after signing).

---

## 2. Submit signature

**Endpoint**: `POST /api/v1/public/signatures/documents/sign`

**Request body** (summary):
```json
{
  "documentId": "507f1f77bcf86cd799439011",
  "signerId": "507f1f77bcf86cd799439081",
  "fieldValues": { "signature_1": "data:image/png;base64,..." },
  "uploadedAttachments": {},
  "idVerificationToken": "optional"
}
```

**Success** (200) — includes envelope continuation:
```json
{
  "success": true,
  "data": {
    "allSigned": false,
    "signingContext": {
      "hasEnvelope": true,
      "hasMoreDocumentsForSigner": true,
      "nextSigning": {
        "documentId": "507f1f77bcf86cd799439012",
        "signerId": "507f1f77bcf86cd799439081",
        "token": "...",
        "signingUrl": "https://cleared.id/sign/flow?..."
      }
    }
  }
}
```

**Notes**:
- When `signingContext.nextSigning.signingUrl` is present, the signer UI redirects immediately (no intermediate “choose next step” screen).
- When absent and the document is fully executed for this signer in the envelope, show completion.

---

## Related documentation

- [Envelopes API](./envelopes.md) — send, `quietMode`, `invitationLinks`
- [Merchant Signature Documents API](./merchant-signature-documents.md) — merchant send/resend
- [API Overview](./README.md)
