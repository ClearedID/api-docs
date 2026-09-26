# Verification Links API

## Overview

The Verification Links API enables merchants to create shareable verification links that customers can use to complete identity or address verification. These links provide a branded, embeddable verification experience that can be integrated into any website or application.

**Base Path**: `/api/v1/merchant/verification-links`

**Authentication**: Merchant JWT token in Authorization header

## Key Features

- **Shareable Verification Links** – Create unique URLs for verification flows
- **Custom Branding** – Add company logo, name, and styling
- **Webhook Integration** – Receive real-time verification result notifications
- **URL Parameters** – Pass custom parameters through verification flow
- **Reference Numbers** – Track verifications with unique reference codes
- **Verification Tokens** – Secure tokens for result retrieval
- **Multi-Type Support** – Identity, address, company, and combined verifications
- **Submission Tracking** – Monitor all verifications completed via each link

## How Verification Links Work

### Link Creation Flow

```
1. Merchant creates verification link via API
2. System generates unique linkId
3. Merchant embeds link in website/email/app
4. Customer clicks link and completes verification
5. System processes verification
6. Operations team reviews (if needed)
7. Result sent to merchant via webhook
8. Customer redirected to merchant's destination URL
```

### Link Structure

**Verification Link URL**:
```
https://cleared.id/v/{linkId}
```

**Example**:
```
https://cleared.id/v/signer-kyc
https://cleared.id/v/employee-verification
https://cleared.id/v/tenant-screening
```

### Customer Experience

1. **Clicks Link** → Opens branded verification page
2. **Sees Company Branding** → Logo, name, description
3. **Completes Verification** → Uploads ID, address proof, etc.
4. **Receives Confirmation** → Email with reference number
5. **Redirected** → Returns to merchant site with verification token

### Merchant Experience

1. **Creates Link** → Configures branding and settings
2. **Embeds Link** → Adds to website/app/email
3. **Receives Webhook** → Real-time notification when verification completes
4. **Retrieves Results** → Uses verification token or reference number
5. **Monitors Submissions** → Tracks all verifications via link

## Verification Types

Links can be configured for different verification types:

### Identity Verification
- **Purpose**: Verify customer identity with government-issued ID
- **Required**: ID document upload, selfie with liveness
- **Returns**: Full name, DOB, ID number, TRN (if applicable)

### Address Verification
- **Purpose**: Verify residential or business address
- **Required**: Proof of address document (utility bill, bank statement)
- **Returns**: Verified address, document details

### Combined Verification
- **Purpose**: Both identity and address in one flow
- **Required**: ID documents + proof of address
- **Returns**: Complete customer profile

### Company Verification
- **Purpose**: Verify business/company information
- **Required**: Company registration docs, business address proof
- **Returns**: Company details, directors, address

## Branding & Customization

### Client Branding

```json
{
  "clientBranding": {
    "companyName": "Acme Corporation",
    "logo": "https://example.com/logo.png",
    "primaryColor": "#007bff",
    "accentColor": "#0056b3"
  }
}
```

### Styling Options

```json
{
  "styling": {
    "theme": "light",
    "buttonStyle": "rounded",
    "fontFamily": "Inter, sans-serif"
  }
}
```

### Custom Description

Personalize the verification page with:
- Welcome message
- Instructions
- Purpose explanation
- Required documents list

## Webhook Configuration

### Webhook Setup

```json
{
  "webhookConfig": {
    "url": "https://your-api.com/webhooks/verification",
    "secret": "your-webhook-secret-key",
    "active": true
  }
}
```

### Webhook Events

System sends webhooks for:
- `verification.submitted` - Customer submitted documents
- `verification.cleared` - Verification approved
- `verification.rejected` - Verification rejected

### Webhook Payload

**Identity Verification Cleared**:
```json
{
  "trn": "123-456-789",
  "firstName": "John",
  "lastName": "Smith",
  "documentNumber": "A1234567",
  "issueDate": "2021-07-29",
  "expiryDate": "2031-07-28",
  "issuingAuthority": "Passport Immigration and Citizenship Agency",
  "event": "cleared",
  "referenceNumber": "REF-ABC123",
  "verificationToken": "token_xyz123"
}
```

### Webhook Security

- **HMAC Signature**: SHA-256 HMAC of payload with secret
- **Header**: `X-Webhook-Signature`
- **Validation**: Verify signature matches payload
- **Retry**: Auto-retry on failure (3 attempts)

## URL Parameters

Pass custom parameters through the verification flow:

### Parameter Configuration

```json
{
  "urlParameters": [
    {
      "key": "orderId",
      "value": "ORD-12345"
    },
    {
      "key": "customerId",
      "value": "CUST-67890"
    }
  ]
}
```

### Parameter Usage

Parameters are:
1. Appended to destination URL after verification
2. Included in webhook payload
3. Stored with verification session
4. Returned in submission data

**Example Destination URL**:
```
https://your-site.com/verification/complete?
  verificationToken=token_xyz&
  referenceNumber=REF-ABC&
  orderId=ORD-12345&
  customerId=CUST-67890
```

## Reference Numbers & Verification Tokens

### Reference Numbers

**Purpose**: Human-readable identifier for customer

**Format**: `REF-XXXXXX` (e.g., `REF-A1B2C3`)

**Usage**:
- Displayed to customer in email
- Used for customer support inquiries
- Validates verification ownership
- Provides to merchant in webhook

### Verification Tokens

**Purpose**: Secure token for retrieving verification results

**Format**: JWT token with verification details

**Usage**:
- Returned in destination URL
- Used by merchant to retrieve full results
- Expires based on link configuration
- Single-use or multi-use depending on settings

## Link Status

Links can have the following statuses:

| Status | Description | Accepts Verifications |
|--------|-------------|----------------------|
| `active` | Link is active and accepting verifications | ✅ Yes |
| `inactive` | Link is temporarily disabled | ❌ No |
| `archived` | Link is archived/deprecated | ❌ No |
| `deleted` | Link is deleted | ❌ No |

## Next Steps

Ready to integrate Verification Links?

1. **[Verification Links Endpoints](./verification-links-endpoints.md)** - Complete endpoint reference
2. **[Identity API](../identity/identity-api.md)** - Identity verification details
3. **[Address API](../address/address-api.md)** - Address verification details

## Support

For Verification Links API support:

- **Technical Documentation**: [Verification Links Endpoints](./verification-links-endpoints.md)
- **Email Support**: [support@cleared.id](mailto:support@cleared.id)
- **Admin Portal**: [https://cleared.id/admin](https://cleared.id/admin)

---

**Continue to**: [Verification Links Endpoints](./verification-links-endpoints.md) →

