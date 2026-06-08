# Signing notification controls

Control **signer-facing** emails for document signing: invitations, reminders, and completion/update emails to signers.

Merchant **webhooks**, **client (merchant) push**, and **client completion emails** are **not** affected by these settings.

## `notifications` object

**Mute everything:**

```json
{
  "notifications": {
    "mute": true
  }
}
```

**Granular (mute one category only):**

```json
{
  "notifications": {
    "invitations": "mute",
    "signingUpdates": null
  }
}
```

| Field | Type | Meaning |
|-------|------|---------|
| `mute` | `true` only | Suppresses **all** signer-facing signing notifications (invitations, resends, reminders, signer completion emails). |
| `invitations` | `"mute"` \| `null` | `"mute"` suppresses invitation and resend invitation emails; `null` leaves invitations enabled. |
| `signingUpdates` | `"mute"` \| `null` | `"mute"` suppresses reminders and signer-facing completion/update emails; `null` leaves them enabled. |

Omitted granular keys behave as **enabled** (same as `null`).

**Legacy:** boolean `false` / `true` on `invitations` and `signingUpdates` is still accepted (`false` → `"mute"`, `true` → `null`).

### Validation

Invalid shapes return **400**:

```json
{
  "error": true,
  "message": "Invalid notifications configuration. mute must be true when set; invitations and signingUpdates must be \"mute\", null, or a legacy boolean."
}
```

## Where to set notifications

| Level | Where | Persists on |
|-------|--------|-------------|
| **Template default** | `configuration.notifications` on template save | `SignatureTemplate` pending/published snapshot |
| **Request (instantiate / send)** | Top-level `notifications` and/or `configuration.notifications` on instantiate or send body | `SignatureDocument.configuration` (and envelope when applicable) |
| **Per signer** | `signingParties[].notifications` on instantiate or send | Each `signingParties[]` entry on the document |
| **Envelope send** | Top-level `notifications` on `POST …/envelopes/:envelopeId/send` | Envelope + merged onto each document in the envelope |
| **Batch instantiate** | Top-level `notifications` / `configuration` on `POST …/batch-instantiate` | Applied to every row in the batch |

Legacy **`configuration.quietMode: true`** (read-only) is treated as **`notifications.mute: true`** when evaluating sends. New writes should use `notifications`; `quietMode` is not written on save.

## Precedence

1. **Request `mute: true`** — nothing signer-facing is sent; signer-level settings cannot re-enable.
2. **Request `invitations: "mute"`** or **`signingUpdates: "mute"`** — that category is off for the whole request; signers cannot override back to `null`.
3. **Signer-level `notifications`** — applies only when the request does not already suppress that category (signer can suppress further, not enable what the request disabled).

Envelope configuration is merged with document configuration when resolving effective settings for a document in an envelope.

## `invitationLinks` in responses

When invitations are suppressed (`mute: true` or `invitations: "mute"`), Cleared does **not** email signers. The API returns **`invitationLinks`** synchronously on:

- Document send — `POST …/documents/:documentId/send`
- Template instantiate with `sendImmediately: true` — `POST …/templates/:templateId/instantiate`
- Envelope template instantiate with `sendImmediately: true`
- Envelope send — `POST …/envelopes/:envelopeId/send`
- Batch instantiate when `sendImmediately: true` (per successful row, same as single instantiate)

Each link entry includes a ready-to-use **`signingUrl`**, plus `signerEmail`, `signerName`, `documentId`, `signerId` (and for envelopes: `envelopeId`, `documentTitles` where applicable).

Deliver these URLs via your own channel (portal, WhatsApp, SMS, etc.).

Example fragment:

```json
{
  "notificationsMuted": true,
  "notifications": { "mute": true },
  "invitationLinks": [
    {
      "signerEmail": "john@example.com",
      "signerName": "John Smith",
      "signingUrl": "https://cleared.id/sign/flow?documentId=...&signerId=...&token=...",
      "documentId": "507f1f77bcf86cd799439011",
      "signerId": "507f1f77bcf86cd799439012"
    }
  ]
}
```

Granular example response:

```json
{
  "notificationsMuted": false,
  "notifications": {
    "invitations": "mute",
    "signingUpdates": null
  },
  "invitationLinks": [ ... ]
}
```

## Resend and remind when muted

If you call resend or remind while notifications are suppressed:

**Resend invitation** — `POST …/documents/:documentId/signers/:signerId/resend`

```json
{
  "error": true,
  "message": "Signer invitation notifications are suppressed. Use invitation link below.",
  "code": "NOTIFICATIONS_MUTED",
  "invitationLink": {
    "signerEmail": "john@example.com",
    "signingUrl": "https://cleared.id/sign/flow?...",
    "documentId": "...",
    "signerId": "..."
  }
}
```

**Remind signer** — `POST …/documents/:documentId/signers/:signerId/remind`

```json
{
  "error": true,
  "message": "Signer signing update notifications are suppressed.",
  "code": "NOTIFICATIONS_MUTED"
}
```

Use **`invitationLinks`** from the original send/instantiate response, or call send again in a context that returns fresh links.

## Examples

### Mute all signer emails (deliver links yourself)

```json
{
  "signingParties": [
    { "name": "Jane Smith", "email": "jane@example.com", "order": 1 }
  ],
  "notifications": {
    "mute": true
  }
}
```

### Invitations muted, reminders still sent

```json
{
  "notifications": {
    "invitations": "mute",
    "signingUpdates": null
  }
}
```

### Per-signer: one signer gets no invitations

```json
{
  "notifications": { "invitations": null, "signingUpdates": null },
  "signingParties": [
    {
      "name": "Partner A",
      "email": "a@company.com",
      "order": 1,
      "notifications": { "invitations": "mute", "signingUpdates": null }
    },
    {
      "name": "Partner B",
      "email": "b@company.com",
      "order": 2
    }
  ]
}
```

## Related endpoints

- [Merchant Signature Documents — Send document](./merchant-signature-documents.md#15-send-document-for-signing)
- [Document Templates — Instantiate](./document-templates.md#instantiate-template)
- [Envelopes — Send envelope](./envelopes.md#10-send-envelope)
- [Envelope Templates — Instantiate](./envelope-templates.md)
- [Overview — Notification controls](./README.md#notification-controls-notifications)
