# Document signature emails — wording for client reference

This document shows **sample messages** as recipients see them, with **`{{placeholder}}`** tokens where values are filled at send time. Use the **Placeholder glossary** to interpret each field.

Technical source paths are in **Appendix** at the bottom.

---

## Placeholder glossary

| Placeholder | Meaning |
|-------------|---------|
| `{{signer_email}}` | Email address of the signer (To-address for invitation and signer completion). |
| `{{merchant_contact_email}}` | Email address of the merchant client (To-address for client completion). |
| `{{document_title}}` | Title of the signature document. |
| `{{signer_name}}` | Full name of the signer receiving the email. |
| `{{organisation_name}}` | Sending organisation’s display name (company). |
| `{{signing_link_url}}` | Time-limited URL to open the signing flow (includes auth token in query string). |
| `{{sender_custom_message}}` | Optional note from the sender; **only included when set** on the document. Plain text, HTML-escaped in production. |
| `{{link_validity_sentence}}` | Either `This link is valid until {{document_expiry_date}}` (UK-style long date) **or** `This link is valid for 7 days` if no document expiry. |
| `{{document_expiry_date}}` | Human-readable expiry date for the signing link (when document expiry is configured). |
| `{{identity_verification_notice}}` | Optional warning sentence(s) when ID verification is required before signing (see sample). |
| `{{pdf_view_url}}` | Time-limited signed URL to **view** the finalized PDF in browser (~7 days). |
| `{{pdf_download_url}}` | Time-limited signed URL to **download** the finalized PDF (~7 days). |
| `{{client_greeting_name}}` | Merchant contact name for greeting (display name or first name; may fall back to a neutral form). |
| `{{signer_count}}` | Number of signing parties on the document (integer). |
| `{{signers_plural_word}}` | Either `person` or `people` (matches `{{signer_count}}`). |
| `{{completed_date}}` | Date the completion email was sent (locale-formatted). |
| `{{merchant_documents_dashboard_url}}` | Link to the merchant portal documents screen for this document. |
| `{{merchant_portal_base_url}}` | Base URL of the merchant portal (environment-dependent). |

---

## Sample 1 — Signature request (email to signer)

**Recipient:** signer (`{{signer_email}}` — not shown in body; used as To-address).

**Subject**

```
Document Signature Request: {{document_title}}
```

**Sample body (plain-text representation of the email)**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📄 Document Signature Request
You have been invited to sign a document
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Hello {{signer_name}},

You have been invited to sign the following document:

── Document Details ──
Document: {{document_title}}
Sender: {{organisation_name}}

[OPTIONAL — only if sender added a message]
── Message from sender ──
{{sender_custom_message}}

[OPTIONAL — only if identity verification is required]
⚠️ IMPORTANT: Identity verification will be required before signing this document.

[ Button: Sign Document Now ]
   → links to: {{signing_link_url}}

── Important Information: ──
 • {{link_validity_sentence}}
 [OPTIONAL — only if IDV required] • You will need to verify your identity before signing
 • Please contact the sender if you have questions

If the button above doesn't work, copy and paste this link into your browser:

{{signing_link_url}}

────────────────────────────────────────
This is an automated message. Please do not reply to this email.
```

**Filled example (illustrative only — links fictional)**

```
Subject: Document Signature Request: Loan Agreement — March 2026

Hello Jane Doe,

You have been invited to sign the following document:

Document: Loan Agreement — March 2026
Sender: Acme Finance Ltd

[ Sign Document Now → https://cleared.id/sign/flow?documentId=…&signerId=…&token=… ]

Important Information:
 • This link is valid until 15 August 2026
 • You will need to verify your identity before signing
 • Please contact the sender if you have questions
```

---

## Sample 2 — Fully signed (email to each signer)

**Recipient:** each signer with an email on the document.

**Subject**

```
Document Signed: {{document_title}}
```

**Sample body**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Document Signing Completed
Your document is ready for download
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Dear {{signer_name}},

Great news! The document "{{document_title}}" has been successfully signed by all parties and is now complete.

You can view or download the fully signed document using the links below:

[ Button: View Document ]   → {{pdf_view_url}}
[ Button: Download Document ] → {{pdf_download_url}}

── Document Information: ──
 • This link will expire in 7 days
 • Document has been digitally signed with an advanced electronic signature (AES)
 • Includes a comprehensive audit trail
 • Open in Adobe to check validity of signatures to detect forgeries and tampering

If you have any questions, please don't hesitate to contact us.

────────────────────────────────────────
Best regards,
Cleared Identity Limited
```

*(Production HTML uses Cleared styling and a Cleared header logo; wording above matches the template.)*

---

## Sample 3 — Fully signed (email to client / merchant)

**Recipient:** client account email (`{{merchant_contact_email}}` — To-address; not shown in body).

**Subject**

```
✅ Document Signing Complete: {{document_title}}
```

**Sample body**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Document Signing Complete!
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Hi {{client_greeting_name}},

Great news! Your document "{{document_title}}" has been successfully signed by all parties.

✓ All Signers Have Completed
Document: {{document_title}}
Signers: {{signer_count}} {{signers_plural_word}}
Completed: {{completed_date}}

The document has been sealed with a cryptographic digital signature for legal validity.

You can view the signed document in your browser or download it to your computer:

[ 👁️ View Document ]   → {{pdf_view_url}}
[ ⬇️ Download Document ] → {{pdf_download_url}}

📁 Document Management: You can also access this document anytime from your Documents Dashboard:
   {{merchant_documents_dashboard_url}}
   (typically under {{merchant_portal_base_url}}/documents?id=<document id>)

The signed document will be available for download for the next 7 days. Please save a copy to your computer.

────────────────────────────────────────
This is an automated notification from Cleared
Digital Document Signatures | Identity Verification
```

**Sender display name:** `Cleared — Document Signatures`

---

## Sample 4 — Fully signed (alternate client email, shorter)

Used when completion is finalized after identity verification clearance on some flows.

**Subject**

```
✅ Document Signing Complete: {{document_title}}
```

**Sample body**

```
Document signing is complete

All required signatures and identity checks are now complete for {{document_title}}.

You can view or download the finalized PDF using the links below.

View document   → {{pdf_view_url}}
Download document → {{pdf_download_url}}
```

**Sender display name:** `Cleared — Document Signatures`

---

## Sample 5 — Push notification (merchant app, paired with completion)

Not an email; included because it ships with the same milestone.

| Field | Sample value |
|-------|----------------|
| Title | `Document Signing Completed` |
| Body | `Your document "{{document_title}}" has been fully signed and is ready for download.` |

---

## Appendix — template inventory (engineering)

### 1. Signature request (email to signer)

**Where:** `swf-machine-runner.ms` → `generateEmailTemplate` / `sendSignerEmail` (`index.js`).

**Subject line**

- `Document Signature Request: ` + document title

**Email title (HTML)**

- `Document Signature Request`

**Header**

- `Document Signature Request` / `You have been invited to sign a document`

**Optional blocks**

- `Message from sender` + custom message (escaped).
- IDV: `IMPORTANT: Identity verification will be required before signing this document.`

**CTA**

- `Sign Document Now`

**Important box**

- `Important Information:` + bullets using `{{link_validity_sentence}}` pattern above + optional IDV bullet + contact sender.

**Footer**

- `This is an automated message. Please do not reply to this email.`

---

### 2. Fully signed — email to each signer

**Where:** `swf-core-gateway` → `services/sslcom-esigner.js` → `notifySignersOfCompletion`.

**Subject:** `Document Signed: ` + title.

**Sender display:** `Cleared` (with Cleared header logo in HTML).

---

### 3. Fully signed — email to client (merchant)

**Where:** `swf-machine-runner.ms` → `notifyClientOfDocumentCompletion` (`index.js`).

---

### 4. Fully signed — alternate client email

**Where:** `swf-core-gateway` → `utils/publicSignatureSigningAuxRoutes.cjs`.

---

## Notes for the client

1. **Resend / remind from API:** In `swf-core-gateway/routes/gatewayHttpRegs.cjs`, “resend” / “remind” helpers are stubs (log only). Live **signer invitation** copy is **Sample 1** (machine-runner).
2. **Links:** Invitation uses `{{signing_link_url}}`; completion emails use `{{pdf_view_url}}` / `{{pdf_download_url}}` (typically **7-day** signed URLs).
3. **Branding:** HTML templates include Cleared colours and some emoji in headings; signer completion sign-off references **Cleared Identity Limited**.
