# Public Signature Documents API

## Overview

The Public Signature Documents API provides endpoints for signers to interact with documents they've been invited to sign. These endpoints are accessed using signing tokens that are unique to each signer and document combination.

**Base Path**: `/api/v1/public/signatures/documents`

**Authentication**: Signing token (included in the signing URL sent to signers)

## Endpoints

### 1. Get Document for Signing

Retrieve a document's details, fields, and page images for a specific signer.

**Endpoint**: `GET /api/v1/public/signatures/documents/:documentId`

**Authentication**: Query parameters (`signerId` and `token`)

**URL Parameters**:
- `documentId` (string, required) - The document's unique identifier

**Query Parameters**:
- `signerId` (string, required) - The signer's unique identifier
- `token` (string, required) - JWT signing token for this signer

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "document": {
      "_id": "507f1f77bcf86cd799439011",
      "title": "Employment Contract",
      "description": "Standard employment agreement",
      "status": "enqueued",
      "signingParties": [
        {
          "_id": "507f1f77bcf86cd799439012",
          "name": "John Smith",
          "email": "john@example.com",
          "status": "pending",
          "order": 1,
          "role": "Employee",
          "idVerificationStatus": "not_required"
        }
      ],
      "fields": [
        {
          "id": "signature_1",
          "type": "signature",
          "label": "Employee Signature",
          "required": true,
          "assignedTo": "507f1f77bcf86cd799439012",
          "page": 1,
          "position": {
            "x": 100,
            "y": 500,
            "width": 200,
            "height": 50
          }
        },
        {
          "id": "date_1",
          "type": "date",
          "label": "Signature Date",
          "required": true,
          "assignedTo": "507f1f77bcf86cd799439012",
          "page": 1,
          "position": {
            "x": 350,
            "y": 500,
            "width": 100,
            "height": 30
          }
        }
      ],
      "configuration": {
        "requireIdVerification": false,
        "enforceSigningOrder": true,
        "expiresInDays": 30
      },
      "createdAt": "2025-10-15T10:00:00Z",
      "expiresAt": "2025-11-14T10:00:00Z"
    },
    "pageImages": [
      {
        "pageNumber": 1,
        "width": 612,
        "height": 792,
        "widthPx": 612,
        "heightPx": 792,
        "imageUrl": "https://s3.amazonaws.com/bucket/path/to/page-1.png?signature=...",
        "imageKey": "signatures/507f1f77bcf86cd799439011/pages/page-1.png"
      }
    ],
    "signer": {
      "_id": "507f1f77bcf86cd799439012",
      "name": "John Smith",
      "email": "john@example.com",
      "status": "pending",
      "order": 1
    },
    "organizationName": "Acme Corporation"
  }
}
```

**Error Responses**:

400 Bad Request:
```json
{
  "success": false,
  "error": "Signer ID and token are required"
}
```

401 Unauthorized:
```json
{
  "success": false,
  "error": "Invalid or expired signing token"
}
```

404 Not Found:
```json
{
  "success": false,
  "error": "Document not found"
}
```

**Notes**:
- If ID verification is required but not completed, `pageImages` will contain placeholder objects without actual image URLs
- Signed URLs for page images expire after 1 hour
- The JWT token is validated to ensure it matches both the document and signer IDs

---

### 2. Initiate Identity Verification

Generate an identity verification link for a signer who needs to complete ID verification before signing.

**Endpoint**: `POST /api/v1/public/signatures/documents/:documentId/initiate-idv`

**URL Parameters**:
- `documentId` (string, required) - The document's unique identifier

**Request Body**:
```json
{
  "signerId": "507f1f77bcf86cd799439012",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Request Parameters**:
- `signerId` (string, required) - The signer's unique identifier
- `token` (string, required) - JWT signing token

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "verificationLinkUrl": "https://cleared.id/v/signer-identity?documentId=507f1f77bcf86cd799439011&signerId=507f1f77bcf86cd799439012&token=...&entity=Acme+Corporation&destination=...",
    "documentId": "507f1f77bcf86cd799439011",
    "signerId": "507f1f77bcf86cd799439012"
  }
}
```

**Error Responses**:

400 Bad Request:
```json
{
  "success": false,
  "error": "Signer has already signed this document"
}
```

401 Unauthorized:
```json
{
  "success": false,
  "error": "Invalid signing token"
}
```

404 Not Found:
```json
{
  "success": false,
  "error": "Organization not found"
}
```

**Verification URL Parameters**:
The generated verification URL includes:
- `documentId` - Document identifier
- `signerId` - Signer identifier
- `token` - Original signing token
- `entity` - Organization name
- `logo` - Organization logo URL (if available)
- `description` - Description of the verification purpose
- `destination` - Return URL after successful verification

**Notes**:
- This endpoint is only needed when `requireIdVerification` is enabled
- After successful verification, the signer is redirected back to the signing page
- The verification status is stored in the signer's record

---

### 3. Submit Signature

Submit the signer's field values and signature data for a document.

**Endpoint**: `POST /api/v1/public/signatures/documents/sign`

**Request Body**:
```json
{
  "documentId": "507f1f77bcf86cd799439011",
  "signerId": "507f1f77bcf86cd799439012",
  "fieldValues": {
    "signature_1": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAA...",
    "date_1": "2025-10-19",
    "text_1": "John Smith",
    "email_1": "john@example.com"
  }
}
```

**Request Parameters**:
- `documentId` (string, required) - The document's unique identifier
- `signerId` (string, required) - The signer's unique identifier
- `fieldValues` (object, required) - Key-value pairs of field IDs and their values
  - Signature fields: Base64-encoded PNG image data
  - Date fields: ISO date string (YYYY-MM-DD)
  - Text fields: String value
  - Email fields: Email address string

**Success Response** (200):
```json
{
  "success": true,
  "message": "Document signed by John Smith. Waiting for 1 more signer(s).",
  "data": {
    "documentId": "507f1f77bcf86cd799439011",
    "signerId": "507f1f77bcf86cd799439012",
    "signerName": "John Smith",
    "signerEmail": "john@example.com",
    "pdfVersion": 2,
    "pdfUrl": "https://s3.amazonaws.com/bucket/documents/507f1f77bcf86cd799439011_v2.pdf",
    "allSigned": false,
    "status": "enqueued",
    "signingMode": "manual_signature",
    "processingTime": "1250ms",
    "remainingSigners": 1,
    "totalSigners": 2,
    "queuedForDigitalSignature": false,
    "auditLogEntry": {
      "action": "document_signed_manually",
      "timestamp": "2025-10-19T12:00:00Z",
      "ipAddress": "192.168.1.1",
      "userAgent": "Mozilla/5.0..."
    }
  }
}
```

**Success Response - All Signers Completed** (200):
```json
{
  "success": true,
  "message": "All signers completed! Document queued for digital signature processing by CLEARED IDENTITY LIMITED",
  "data": {
    "documentId": "507f1f77bcf86cd799439011",
    "signerId": "507f1f77bcf86cd799439013",
    "signerName": "Jane Doe",
    "signerEmail": "jane@example.com",
    "pdfVersion": 3,
    "pdfUrl": "https://s3.amazonaws.com/bucket/documents/507f1f77bcf86cd799439011_v3.pdf",
    "allSigned": true,
    "status": "ready_for_digital_signature",
    "signingMode": "manual_signature",
    "processingTime": "1450ms",
    "remainingSigners": 0,
    "totalSigners": 2,
    "queuedForDigitalSignature": true,
    "auditLogEntry": {
      "action": "document_queued_for_digital_signature",
      "timestamp": "2025-10-19T12:05:00Z",
      "ipAddress": "192.168.1.2",
      "userAgent": "Mozilla/5.0..."
    }
  }
}
```

**Error Responses**:

400 Bad Request - Missing fields:
```json
{
  "success": false,
  "error": "Document ID, signer ID, and field values are required"
}
```

400 Bad Request - Already signed:
```json
{
  "success": false,
  "error": "Document has already been signed by this signer"
}
```

400 Bad Request - Invalid fields:
```json
{
  "success": false,
  "error": "You can only sign fields assigned to you. Invalid fields: signature_2, date_2"
}
```

404 Not Found:
```json
{
  "success": false,
  "error": "Document not found"
}
```

500 Internal Server Error:
```json
{
  "success": false,
  "error": "Failed to complete manual signature",
  "requestId": "a1b2c3d4e5f6g7h8"
}
```

**Process Flow**:

1. **Validation**: System validates document ID, signer ID, and field values
2. **Field Assignment Check**: Verifies signer can only fill fields assigned to them
3. **PDF Update**: Applies field values to PDF (creates new version)
4. **Signer Status Update**: Marks signer as `signed` with timestamp and metadata
5. **Check Completion**: Determines if all signers have completed
6. **If Complete**: 
   - Status → `ready_for_digital_signature`
   - Document queued for digital signature batch processing
   - CLEARED IDENTITY LIMITED digital certificate will be applied asynchronously
7. **If Incomplete**: 
   - Status remains `enqueued`
   - Waits for remaining signers

**Digital Signature Processing**:
- After all signers complete, the document enters the digital signature queue
- A batch processor applies the CLEARED IDENTITY LIMITED digital certificate
- This creates a legally binding, tamper-evident PDF
- Includes OCSP/CRL for long-term validation
- Final status → `completed`

**Notes**:
- Each signer creates a new PDF version with their field values applied
- The digital signature is applied only after all signers complete
- All actions are logged in the audit trail with IP, user agent, and timestamp
- Field values can include base64-encoded images for signature fields
- Processing time typically ranges from 500ms to 2000ms depending on PDF size

---

### 4. Get Signer Details

Retrieve specific signer information for a document.

**Endpoint**: `GET /api/v1/public/signatures/documents/:documentId/signer/:signerId`

**URL Parameters**:
- `documentId` (string, required) - The document's unique identifier
- `signerId` (string, required) - The signer's unique identifier

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "signer": {
      "_id": "507f1f77bcf86cd799439012",
      "name": "John Smith",
      "email": "john@example.com",
      "status": "signed",
      "order": 1,
      "role": "Employee",
      "signedAt": "2025-10-19T12:00:00Z",
      "idVerificationStatus": "verified",
      "signingDetails": {
        "ipAddress": "192.168.1.1",
        "userAgent": "Mozilla/5.0...",
        "timestamp": "2025-10-19T12:00:00Z",
        "fieldCount": 3,
        "totalFieldsAssigned": 3,
        "signingMode": "manual_signature"
      }
    }
  }
}
```

**Error Responses**:

404 Not Found - Document:
```json
{
  "success": false,
  "error": "Document not found"
}
```

404 Not Found - Signer:
```json
{
  "success": false,
  "error": "Signer not found for this document"
}
```

---

### 5. Download Document

Download the current version of the document PDF.

**Endpoint**: `GET /api/v1/public/signatures/documents/:documentId/download`

**URL Parameters**:
- `documentId` (string, required) - The document's unique identifier

**Query Parameters**:
- `signerId` (string, optional) - Filter audit log by signer
- `version` (number, optional) - Download specific version (default: latest)

**Success Response** (200):
- **Content-Type**: `application/pdf`
- **Content-Disposition**: `attachment; filename="document-title.pdf"`
- **Body**: PDF file binary data

**Error Responses**:

404 Not Found:
```json
{
  "success": false,
  "error": "Document not found"
}
```

**Notes**:
- Returns the latest version of the PDF by default
- If `version` parameter is provided, returns that specific version
- Completed documents include the digital signature
- Draft/in-progress documents return the current working version

---

### 6. Get Document Audit Log

Retrieve the complete audit trail for a document.

**Endpoint**: `GET /api/v1/public/signatures/documents/:documentId/audit-log`

**URL Parameters**:
- `documentId` (string, required) - The document's unique identifier

**Query Parameters**:
- `signerId` (string, optional) - Filter audit log by signer
- `limit` (number, optional) - Limit number of entries (default: 100, max: 1000)
- `offset` (number, optional) - Pagination offset (default: 0)

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "auditLog": [
      {
        "_id": "507f1f77bcf86cd799439020",
        "documentId": "507f1f77bcf86cd799439011",
        "action": "document_created",
        "description": "Document created by Admin User",
        "timestamp": "2025-10-15T10:00:00Z",
        "metadata": {
          "clientId": "507f1f77bcf86cd799439030",
          "title": "Employment Contract"
        }
      },
      {
        "_id": "507f1f77bcf86cd799439021",
        "documentId": "507f1f77bcf86cd799439011",
        "action": "document_sent",
        "description": "Document sent to 2 signers",
        "timestamp": "2025-10-15T11:00:00Z",
        "metadata": {
          "signerCount": 2,
          "expiresAt": "2025-11-14T11:00:00Z"
        }
      },
      {
        "_id": "507f1f77bcf86cd799439022",
        "documentId": "507f1f77bcf86cd799439011",
        "action": "document_signed_manually",
        "description": "Document signed by John Smith (john@example.com) - 1 signers remaining",
        "timestamp": "2025-10-19T12:00:00Z",
        "metadata": {
          "signerId": "507f1f77bcf86cd799439012",
          "signerName": "John Smith",
          "signerEmail": "john@example.com",
          "ipAddress": "192.168.1.1",
          "userAgent": "Mozilla/5.0...",
          "fieldCount": 3,
          "pdfVersion": 2,
          "processingTime": "1250ms"
        }
      },
      {
        "_id": "507f1f77bcf86cd799439023",
        "documentId": "507f1f77bcf86cd799439011",
        "action": "document_queued_for_digital_signature",
        "description": "All signers completed. Document queued for digital signature processing by CLEARED IDENTITY LIMITED",
        "timestamp": "2025-10-19T12:05:00Z",
        "metadata": {
          "signerId": "507f1f77bcf86cd799439013",
          "signerName": "Jane Doe",
          "totalSigners": 2,
          "pdfVersion": 3
        }
      }
    ],
    "totalRecords": 4,
    "hasMore": false
  }
}
```

**Error Responses**:

404 Not Found:
```json
{
  "success": false,
  "error": "Document not found"
}
```

**Audit Log Actions**:
- `document_created` - Document initially created
- `document_updated` - Document details modified
- `document_sent` - Document sent to signers
- `document_viewed` - Signer viewed the document
- `document_signed_manually` - Signer completed their signature
- `document_queued_for_digital_signature` - All signers completed, queued for digital signature
- `document_signed_digitally` - Digital signature applied
- `document_completed` - Document finalized
- `document_cancelled` - Document cancelled
- `signer_reminded` - Reminder sent to signer
- `idv_initiated` - Identity verification started
- `idv_completed` - Identity verification completed

**Notes**:
- Audit log entries are immutable
- All entries include timestamp and relevant metadata
- Use `signerId` query parameter to filter logs for specific signer
- Pagination available for documents with extensive history

---

### 7. Start Identity Verification (Alternative Flow)

Alternative endpoint for starting identity verification process.

**Endpoint**: `POST /api/v1/public/signatures/documents/:documentId/start-idv`

**URL Parameters**:
- `documentId` (string, required) - The document's unique identifier

**Request Body**:
```json
{
  "signerId": "507f1f77bcf86cd799439012",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "returnUrl": "https://app.example.com/sign/flow?documentId=507f1f77bcf86cd799439011"
}
```

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "verificationSessionId": "idv_session_507f1f77bcf86cd799439040",
    "redirectUrl": "https://verify.cleared.id/session/idv_session_507f1f77bcf86cd799439040"
  }
}
```

**Notes**:
- Similar to `/initiate-idv` but creates a verification session
- Returns a session ID and redirect URL
- After verification, user is redirected to `returnUrl` with session result

---

### 8. Identity Verification Webhook

Webhook endpoint for receiving identity verification results from the verification service.

**Endpoint**: `POST /api/v1/public/signatures/documents/:documentId/idv-webhook`

**URL Parameters**:
- `documentId` (string, required) - The document's unique identifier

**Request Body** (from verification service):
```json
{
  "sessionId": "idv_session_507f1f77bcf86cd799439040",
  "signerId": "507f1f77bcf86cd799439012",
  "status": "verified",
  "verificationData": {
    "documentType": "drivers_license",
    "documentNumber": "DL123456",
    "firstName": "John",
    "lastName": "Smith",
    "dateOfBirth": "1990-01-01",
    "matchScore": 98.5
  },
  "timestamp": "2025-10-19T11:30:00Z"
}
```

**Success Response** (200):
```json
{
  "success": true,
  "message": "Identity verification result processed"
}
```

**Notes**:
- This endpoint is called by the verification service, not by client applications
- Updates the signer's `idVerificationStatus` based on the result
- Verification data is stored securely and included in audit log

---

## Common Use Cases

### Case 1: Simple Signing Flow (No ID Verification)

```javascript
// 1. Signer receives email with signing link containing documentId, signerId, and token
const signingUrl = "https://app.example.com/sign?documentId=507f...&signerId=507f...&token=eyJ...";

// 2. Load document details
const response = await fetch(
  `/api/v1/public/signatures/documents/${documentId}?signerId=${signerId}&token=${token}`
);
const { data } = await response.json();

// 3. Signer fills in fields and submits
const signResponse = await fetch('/api/v1/public/signatures/documents/sign', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    documentId,
    signerId,
    fieldValues: {
      signature_1: signatureImageBase64,
      date_1: '2025-10-19',
      name_1: 'John Smith'
    }
  })
});
```

### Case 2: Signing Flow with ID Verification

```javascript
// 1. Load document details
const response = await fetch(
  `/api/v1/public/signatures/documents/${documentId}?signerId=${signerId}&token=${token}`
);
const { data } = await response.json();

// 2. Check if ID verification is required
if (data.document.configuration.requireIdVerification && 
    data.signer.idVerificationStatus !== 'verified') {
  
  // 3. Initiate ID verification
  const idvResponse = await fetch(
    `/api/v1/public/signatures/documents/${documentId}/initiate-idv`,
    {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ signerId, token })
    }
  );
  
  const { data: idvData } = await idvResponse.json();
  
  // 4. Redirect to verification link
  window.location.href = idvData.verificationLinkUrl;
  
  // After verification completes, user returns to signing page
}

// 5. Once verified (or if not required), proceed with signing
const signResponse = await fetch('/api/v1/public/signatures/documents/sign', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    documentId,
    signerId,
    fieldValues: { /* ... */ }
  })
});
```

### Case 3: Download Signed Document

```javascript
// Download the final signed PDF
const downloadUrl = `/api/v1/public/signatures/documents/${documentId}/download`;
window.open(downloadUrl, '_blank');
```

### Case 4: View Audit Trail

```javascript
// Get complete document history
const auditResponse = await fetch(
  `/api/v1/public/signatures/documents/${documentId}/audit-log`
);
const { data } = await auditResponse.json();

console.log('Document history:', data.auditLog);
```

## Security Notes

1. **Token Validation**: All endpoints validate the signing token to ensure it matches the document and signer
2. **Single Use**: Signing tokens should be treated as single-use credentials
3. **Expiration**: Tokens expire when the document expires or is completed
4. **IP Logging**: All signing actions log the signer's IP address and user agent
5. **HTTPS Only**: All API requests must use HTTPS in production
6. **Rate Limiting**: Public endpoints are rate-limited by IP address

## Error Codes Summary

| Status Code | Description |
|------------|-------------|
| 200 | Success |
| 400 | Bad Request (missing or invalid parameters) |
| 401 | Unauthorized (invalid or expired token) |
| 404 | Not Found (document or signer not found) |
| 500 | Internal Server Error |

## Related Documentation

- [Merchant Signature Documents API](./merchant-signature-documents.md)
- [API Overview](./README.md)

