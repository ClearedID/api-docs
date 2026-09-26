# Document signature emails — wording for client reference

This document shows **sample messages** as recipients see them, with **`{{placeholder}}`** tokens where values are filled at send time. Use the **Placeholder glossary** to interpret each field.

---

## Placeholder glossary

| Placeholder | Meaning |
|-------------|---------|
| `{{signer_email}}` | Email address of the signer (used as the recipient address for invitation and signer completion messages). |
| `{{merchant_contact_email}}` | Email address of the organisation contact (used as the recipient address for merchant completion messages). |
| `{{document_title}}` | Title of the signature document. |
| `{{signer_name}}` | Full name of the signer receiving the email. |
| `{{organisation_name}}` | Sending organisation’s display name (company). |
| `{{signing_link_url}}` | Secure link to open the signing flow. |
| `{{sender_custom_message}}` | Optional note from the sender; **only included when provided** for that document. |
| `{{link_validity_sentence}}` | Either `This link is valid until {{document_expiry_date}}` (long date style) **or** `This link is valid for 7 days` if no document expiry is set. |
| `{{document_expiry_date}}` | Human-readable expiry date for the signing link (when configured). |
| `{{identity_verification_notice}}` | Optional warning when identity verification is required before signing (see Sample 1). |
| `{{pdf_view_url}}` | Time-limited link to **view** the finalized PDF in a browser (typically about **7 days**). |
| `{{pdf_download_url}}` | Time-limited link to **download** the finalized PDF (typically about **7 days**). |
| `{{client_greeting_name}}` | Merchant contact name for the greeting (display name or first name; may use a neutral greeting if needed). |
| `{{signer_count}}` | Number of signing parties on the document. |
| `{{signers_plural_word}}` | Either `person` or `people` (matches `{{signer_count}}`). |
| `{{completed_date}}` | Date the completion message was sent (locale-formatted). |
| `{{merchant_documents_dashboard_url}}` | Link to open this document in the merchant documents area. |
| `{{merchant_portal_base_url}}` | Base web address of the merchant portal for your environment. |

---

## Sample 1 — Signature request (email to signer)

**Recipient:** signer (`{{signer_email}}` — used as the To-address; not repeated inside the email body).

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

[ Sign Document Now → https://… ]

Important Information:
 • This link is valid until 15 August 2026
 • You will need to verify your identity before signing
 • Please contact the sender if you have questions
```

---

## Sample 2 — Fully signed (email to each signer)

**Recipient:** each signer on the document who has an email address.

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

The live email may use a branded layout (colours, logo); the **wording** matches the sample above.

---

## Sample 3 — Fully signed (email to merchant / organisation contact)

**Recipient:** organisation contact (`{{merchant_contact_email}}` — To-address; not shown in body).

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

The signed document will be available for download for the next 7 days. Please save a copy to your computer.

────────────────────────────────────────
This is an automated notification from Cleared
Digital Document Signatures | Identity Verification
```

**Sender display name:** `Cleared — Document Signatures`

---

## Sample 4 — Fully signed (alternate merchant email, shorter)

You may receive this shorter version in some completion scenarios (for example when everything finishes after identity checks).

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

Not an email; sent around the same milestone as the merchant completion email.

| Field | Sample value |
|-------|----------------|
| Title | `Document Signing Completed` |
| Body | `Your document "{{document_title}}" has been fully signed and is ready for download.` |

---

## Notes

1. **Links:** The invitation uses `{{signing_link_url}}`. Completion messages use `{{pdf_view_url}}` and `{{pdf_download_url}}`; those PDF links are usually valid for **about 7 days**.
2. **Branding:** Messages may use Cleared styling in HTML (colours, logo). Signer completion closes with **Cleared Identity Limited**.
3. **Samples 3 and 4:** Both notify the merchant when a document is fully complete; which layout you see can depend on how the flow completes.
