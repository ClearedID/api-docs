# Document Signatures API Documentation

## Overview

The Document Signatures API provides a comprehensive solution for creating, managing, and signing documents digitally. The system supports single documents, document envelopes, templates, and digital signature workflows with identity verification.

## Base URL

```
https://cleared.id/api/v1
```

## Authentication

All merchant endpoints require authentication via JWT token. Include the token in the request headers:

```
Authorization: Bearer YOUR_JWT_TOKEN
Content-Type: application/json
```

Public endpoints (signing flow) use document-specific signing tokens that are generated when a document is sent to signers.

## API Categories

### 1. Public Signature Documents API
Endpoints used by signers to view and sign documents they've been invited to sign.

- **Documentation**: [Public Signature Documents API](./public-signature-documents.md)
- **Base Path**: `/api/v1/public/signatures/documents`
- **Authentication**: Signing token (provided in signing URL)

### 2. Merchant Signature Documents API
Endpoints for merchants to create, manage, and send documents for signature.

- **Documentation**: [Merchant Signature Documents API](./merchant-signature-documents.md)
- **Base Path**: `/api/v1/merchant/signatures/documents`
- **Authentication**: Merchant JWT token

### 3. Envelopes API
Endpoints for managing document envelopes (collections of related documents).

- **Documentation**: [Envelopes API](./envelopes.md)
- **Base Path**: `/api/v1/merchant/signatures/envelopes`
- **Authentication**: Merchant JWT token

### 4. Document Templates API
Endpoints for creating and managing reusable document templates.

- **Documentation**: [Document Templates API](./document-templates.md)
- **Base Path**: `/api/v1/merchant/signatures/templates`
- **Authentication**: Merchant JWT token

### 5. Envelope Templates API
Endpoints for creating and managing reusable envelope templates.

- **Documentation**: [Envelope Templates API](./envelope-templates.md)
- **Base Path**: `/api/v1/merchant/signatures/envelope-templates`
- **Authentication**: Merchant JWT token

## Key Concepts

### Documents
A document represents a single PDF file that requires signatures. Documents can contain:
- **Signing Parties**: People who need to sign the document
- **Fields**: Form fields, signature boxes, text fields, date fields, etc.
- **Configuration**: Settings like signing order enforcement, ID verification requirements
- **Audit Trail**: Complete history of all actions taken on the document

### Envelopes
An envelope is a container for multiple related documents that are sent together to the same set of signers. All documents in an envelope share:
- The same signers
- The same sending event
- A unified completion status

### Templates
Templates are reusable document configurations that include:
- Pre-defined roles (instead of specific signers)
- Pre-positioned fields
- Configuration settings
- When instantiated, roles are mapped to actual signers

### Signing Workflow

1. **Merchant creates document** → Status: `draft`
2. **Merchant uploads PDF and configures fields**
3. **Merchant sends document** → Status: `enqueued`
4. **Signer receives email with signing link**
5. **Signer views document** (with signing token)
6. **(Optional) Identity verification** if required
7. **Signer fills and signs fields** → Status: `signed` for that signer
8. **All signers complete** → Status: `ready_for_digital_signature`
9. **Digital signature applied by CLEARED IDENTITY LIMITED** → Status: `completed`
10. **Final signed PDF available for download**

### Digital Signature Implementation

The system uses a two-phase signing approach:

**Phase 1: Manual Signature Collection**
- Signers fill in their signature fields, text fields, dates, etc.
- Document status is updated as each signer completes
- No digital certificate applied yet

**Phase 2: Digital Signature Application**
- After all signers complete, document is queued for digital signature
- CLEARED IDENTITY LIMITED's digital certificate is applied
- Creates a legally binding, tamper-evident PDF
- Includes OCSP/CRL for long-term validation (LTV)

### Status Values

#### Document Status
- `draft` - Document is being created/edited
- `enqueued` - Document has been sent to signers
- `ready_for_digital_signature` - All signers completed, awaiting digital signature
- `completed` - Digital signature applied, document finalized
- `expired` - Document passed expiry date
- `cancelled` - Document cancelled by merchant

#### Signer Status
- `pending` - Awaiting signer's action
- `signed` - Signer has completed their signature
- `declined` - Signer declined to sign (if enabled)

#### Signing Status (for digital signature processing)
- `idle` - No active signing processing
- `pending_digital_signature` - Queued for digital signature batch processing
- `processing_signature` - Digital signature being applied
- `otp_sent` - OTP sent for verification (if using OTP flow)
- `signature_completed` - Digital signature successfully applied
- `signature_failed` - Digital signature processing failed

## Common Response Format

### Success Response
```json
{
  "success": true,
  "data": {
    // Response data
  },
  "message": "Operation completed successfully"
}
```

### Error Response
```json
{
  "success": false,
  "error": "Error message describing what went wrong",
  "details": "Additional error details (optional)"
}
```

## Rate Limiting

API requests are rate limited to:
- **Merchant endpoints**: 1000 requests per hour per organization
- **Public endpoints**: 100 requests per hour per IP address

Rate limit information is included in response headers:
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 987
X-RateLimit-Reset: 1634567890
```

## Pagination

List endpoints support pagination with the following parameters:
- `page` - Page number (default: 1)
- `limit` or `pageSize` - Items per page (default: 10, max: 100)

Paginated responses include:
```json
{
  "records": [...],
  "paging": {
    "recordCount": 156,
    "pageCount": 16,
    "currentPage": 1
  }
}
```

## Filtering and Sorting

List endpoints support:
- **Keywords**: Search across relevant text fields
- **Filters**: Filter by status, dates, tags, categories, etc.
- **Sorting**: Sort by any sortable field in ascending or descending order

Example query parameter:
```json
{
  "query": {
    "keywords": "employment contract",
    "page": 1,
    "sortField": "createdAt",
    "sortOrder": "desc",
    "filters": {
      "status": ["draft", "enqueued"],
      "createdAt_start": "2025-01-01",
      "createdAt_end": "2025-12-31"
    }
  }
}
```

## Webhooks

Document signature events can trigger webhooks to your configured endpoint:
- `document.sent` - Document sent to signers
- `document.viewed` - Signer viewed document
- `document.signed` - Signer completed signature
- `document.completed` - All signers completed and digital signature applied
- `document.declined` - Signer declined to sign
- `document.expired` - Document expired
- `document.cancelled` - Document cancelled

Webhook payload example:
```json
{
  "event": "document.completed",
  "timestamp": "2025-10-19T12:00:00Z",
  "data": {
    "documentId": "doc_123456789",
    "title": "Employment Contract",
    "status": "completed",
    "organisationId": "org_123456789"
  }
}
```

## Security Considerations

1. **Token Management**
   - JWT tokens expire after 24 hours
   - Signing tokens are document-specific and single-use
   - Store tokens securely, never expose in client-side code

2. **Identity Verification**
   - Enable `requireIdVerification` for sensitive documents
   - Signers must complete ID verification before accessing document content
   - Verification results stored with signer record

3. **Audit Trail**
   - All actions are logged with IP address, user agent, and timestamp
   - Audit logs are immutable and stored permanently
   - Access via the audit log endpoint

4. **PDF Security**
   - Final PDFs are digitally signed with tamper-evident seal
   - OCSP/CRL embedded for long-term validation
   - Original unsigned versions preserved for audit purposes

## Support

For API support, contact:
- **Email**: api-support@cleared.id
- **Documentation**: https://docs.cleared.id
- **Status Page**: https://status.cleared.id

## Changelog

### Version 2.0 (October 2025)
- Added envelope template support
- Enhanced digital signature with LTV support
- Added identity verification integration
- Improved batch processing for digital signatures

### Version 1.5 (June 2025)
- Added document templates
- Enhanced signing order enforcement
- Added webhook support
- Improved error handling and logging

### Version 1.0 (January 2025)
- Initial release
- Basic document signing workflow
- Single document management
- Manual signature collection

