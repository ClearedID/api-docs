# Merchant Users API

## Overview

The Merchant Users API lets integrators **find or create** Cleared user accounts programmatically. Use it when your system needs to ensure a person exists in Cleared before sending verifications, onboarding, or other flows.

**Endpoint:** `POST /api/v1/merchant/users/create`

**Authentication:** Organisation API key as Bearer token (same as envelope template instantiate and other `/api/v1/merchant` routes). See [Authentication](./authentication.md).

---

## Authentication

Include your API key in the `Authorization` header:

```
Authorization: Bearer YOUR_API_KEY
```

Generate keys in the Admin Portal: **Integrations → API Integrations**.

Invalid or missing keys receive **403** from the gateway before the handler runs.

---

## Request body

Accepts **either** a single subject at the root **or** a `subjects` array.

### Single subject

```json
{
  "firstName": "John",
  "middleName": "Michael",
  "lastName": "Smith",
  "emailAddress": "john@example.com",
  "taxNumber": "123456789",
  "phoneNumber": "+18761234567"
}
```

### Batch

```json
{
  "subjects": [
    {
      "firstName": "John",
      "lastName": "Smith",
      "emailAddress": "john@example.com",
      "phoneNumber": "+18761234567"
    },
    {
      "firstName": "Jane",
      "lastName": "Doe",
      "emailAddress": "jane@example.com",
      "phoneNumber": "+18769876543",
      "taxNumber": "987654321"
    }
  ]
}
```

### Fields

| Field | Type | Required | Notes |
|-------|------|----------|--------|
| `firstName` | string | No | Stored on the user when provided |
| `middleName` | string | No | Stored on TRN verification result bio when `taxNumber` is sent |
| `lastName` | string | No | Stored on the user when provided |
| `emailAddress` | string | Conditional | Required if `phoneNumber` omitted |
| `phoneNumber` | string | Conditional | Required if `emailAddress` omitted; single phone field |
| `taxNumber` | string | No | 9-digit TRN; alias `trn` also accepted |

At least one of `phoneNumber` or `emailAddress` is required per subject.

---

## Behaviour

- **Lookup order:** phone number first (`phoneNumber`, then `phoneNumbers[]`), then email (`emailAddress`, then `emailAddresses[]`).
- **Existing user:** status set to `active`, `confirmed` set to `true`, profile fields updated when sent.
- **New user:** created with `status: active`, `confirmed: true`.
- **Tax number:** when provided, a TRN verification result is created if one does not already exist for that user and number.
- **Provenance:** `meta.source` on the user and TRN result is set to your organisation id.

The API does **not** indicate whether the user already existed or was newly created.

---

## Response

### Success

**HTTP 200**

```json
{
  "success": true
}
```

No `userId`, `isNewUser`, or per-subject details are returned.

### Validation error

**HTTP 400**

```json
{
  "success": false,
  "message": "Each subject requires phoneNumber or emailAddress."
}
```

### Processing error

**HTTP 500**

```json
{
  "success": false,
  "message": "Unable to process request."
}
```

Batch requests are all-or-nothing: any subject failure fails the entire request with a generic error.

---

## Example

```bash
curl -X POST https://cleared.id/api/v1/merchant/users/create \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "firstName": "John",
    "lastName": "Smith",
    "emailAddress": "john@example.com",
    "phoneNumber": "+18761234567",
    "taxNumber": "123456789"
  }'
```

**Response:**

```json
{
  "success": true
}
```

---

## Related documentation

- [Authentication](./authentication.md)
- [Endpoints](./endpoints.md)
- [Initiate Verification](./initiate-verification.md)
