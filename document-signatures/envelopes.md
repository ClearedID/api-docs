# Envelopes API

## Overview

The Envelopes API allows merchants to group multiple related documents together and send them as a package to signers. All documents in an envelope share the same signers and are sent together, providing a unified signing experience.

**Base Path**: `/api/v1/merchant/signatures/envelopes`

**Authentication**: Merchant JWT token in Authorization header

## Key Concepts

### What is an Envelope?
An envelope is a container for multiple related documents that need to be signed together by the same set of signers. Benefits include:

- **Unified Experience**: Signers receive one email for all documents
- **Consistent Signers**: All documents share the same signing parties
- **Group Status Tracking**: Monitor completion across all documents
- **Simplified Management**: Manage related documents as a single unit

### Use Cases
- **Employee Onboarding**: Employment contract, NDA, handbook acknowledgment
- **Real Estate Transactions**: Purchase agreement, disclosures, addendums
- **Vendor Agreements**: Service contract, insurance certificate, W9 form
- **Loan Packages**: Loan agreement, promissory note, security agreement

## Endpoints

### 1. List Envelopes

Retrieve a paginated list of envelopes with filtering and sorting.

**Endpoint**: `GET /api/v1/merchant/signatures/envelopes`

**Query Parameters**:
- `query` (string, JSON encoded) - Search/filter/sort parameters:
  ```json
  {
    "keywords": "onboarding",
    "page": 1,
    "sortField": "createdAt",
    "sortOrder": "desc",
    "filters": {
      "status": ["draft", "sent"],
      "createdAt_start": "2025-01-01",
      "createdAt_end": "2025-12-31"
    }
  }
  ```

**Success Response** (200):
```json
{
  "records": [
    {
      "_id": "507f1f77bcf86cd799439040",
      "name": "Employee Onboarding Package",
      "status": "sent",
      "documentCount": 3,
      "signerCount": 2,
      "completedSignatures": 3,
      "totalSignatures": 6,
      "createdAt": "2025-10-15T09:00:00Z",
      "updatedAt": "2025-10-19T12:00:00Z"
    }
  ],
  "columns": [
    {
      "header": "Envelope Name",
      "path": "name",
      "type": "text",
      "sortable": true
    },
    {
      "header": "Documents",
      "path": "documentCount",
      "type": "number",
      "sortable": true
    },
    {
      "header": "Signers",
      "path": "signerCount",
      "type": "number",
      "sortable": true
    },
    {
      "header": "Status",
      "path": "status",
      "type": "status",
      "sortable": true
    }
  ],
  "filters": [
    {
      "name": "status",
      "label": "Status",
      "type": "multi-select",
      "options": ["draft", "sent", "completed", "cancelled"]
    },
    {
      "name": "createdAt",
      "label": "Created At",
      "type": "date-range"
    }
  ],
  "appliedFilters": {},
  "paging": {
    "recordCount": 45,
    "pageCount": 5,
    "currentPage": 1
  }
}
```

**Status Values**:
- `draft` - Envelope being created
- `sent` - Envelope sent to signers
- `completed` - All documents signed
- `cancelled` - Envelope cancelled

---

### 2. Create Envelope

Create a new empty envelope.

**Endpoint**: `POST /api/v1/merchant/signatures/envelopes`

**Request Body**:
```json
{
  "name": "Employee Onboarding Package"
}
```

**Request Parameters**:
- `name` (string, required) - Envelope name

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "id": "507f1f77bcf86cd799439040",
    "name": "Employee Onboarding Package",
    "documentCount": 0,
    "createdAt": "2025-10-19T10:00:00Z"
  }
}
```

**Error Response** (400):
```json
{
  "error": true,
  "message": "Envelope name is required"
}
```

---

### 3. Get Envelope by ID

Retrieve a specific envelope with all its details.

**Endpoint**: `GET /api/v1/merchant/signatures/envelopes/:envelopeId`

**URL Parameters**:
- `envelopeId` (string, required) - Envelope unique identifier

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "envelope": {
      "_id": "507f1f77bcf86cd799439040",
      "name": "Employee Onboarding Package",
      "status": "sent",
      "clientId": "507f1f77bcf86cd799439030",
      "organisationId": "507f1f77bcf86cd799439031",
      "createdBy": "507f1f77bcf86cd799439030",
      "createdAt": "2025-10-15T09:00:00Z",
      "updatedAt": "2025-10-19T12:00:00Z",
      "sentAt": "2025-10-15T10:00:00Z"
    },
    "documents": [
      {
        "_id": "507f1f77bcf86cd799439011",
        "title": "Employment Contract",
        "status": "enqueued",
        "signingParties": [...]
      },
      {
        "_id": "507f1f77bcf86cd799439012",
        "title": "Non-Disclosure Agreement",
        "status": "completed",
        "signingParties": [...]
      }
    ],
    "progress": {
      "totalDocuments": 3,
      "completedDocuments": 1,
      "totalSignatures": 6,
      "completedSignatures": 3,
      "percentage": 50
    }
  }
}
```

**Error Response** (404):
```json
{
  "error": true,
  "message": "Envelope not found"
}
```

---

### 4. Add Document to Envelope

Add an existing document to an envelope.

**Endpoint**: `POST /api/v1/merchant/signatures/envelopes/:envelopeId/documents`

**URL Parameters**:
- `envelopeId` (string, required) - Envelope ID

**Request Body**:
```json
{
  "documentId": "507f1f77bcf86cd799439011"
}
```

**Request Parameters**:
- `documentId` (string, required) - ID of document to add

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "envelopeId": "507f1f77bcf86cd799439040",
    "documentId": "507f1f77bcf86cd799439011",
    "documentCount": 3
  },
  "message": "Document added to envelope successfully"
}
```

**Error Responses**:

400 Bad Request:
```json
{
  "error": true,
  "message": "Document ID is required"
}
```

404 Not Found - Envelope:
```json
{
  "error": true,
  "message": "Envelope not found"
}
```

404 Not Found - Document:
```json
{
  "error": true,
  "message": "Document not found"
}
```

**Notes**:
- Document must be in `draft` status
- Document cannot be in another envelope already
- Document signers will be synchronized with envelope signers when sent

---

### 5. Remove Document from Envelope

Remove a document from an envelope.

**Endpoint**: `DELETE /api/v1/merchant/signatures/envelopes/:envelopeId/documents/:documentId`

**URL Parameters**:
- `envelopeId` (string, required) - Envelope ID
- `documentId` (string, required) - Document ID to remove

**Success Response** (200):
```json
{
  "success": true,
  "message": "Document removed from envelope successfully"
}
```

**Error Response** (404):
```json
{
  "error": true,
  "message": "Document not found in envelope"
}
```

**Notes**:
- Document's `envelopeIds` array is updated
- Document remains in the system, just no longer associated with envelope
- Cannot remove documents from sent or completed envelopes

---

### 6. Get Envelope Documents

Retrieve all documents in an envelope.

**Endpoint**: `GET /api/v1/merchant/signatures/envelopes/:envelopeId/documents`

**URL Parameters**:
- `envelopeId` (string, required) - Envelope ID

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "documents": [
      {
        "_id": "507f1f77bcf86cd799439011",
        "title": "Employment Contract",
        "status": "enqueued",
        "totalPages": 3,
        "signingParties": [
          {
            "name": "John Smith",
            "email": "john@example.com",
            "status": "pending"
          }
        ]
      },
      {
        "_id": "507f1f77bcf86cd799439012",
        "title": "Non-Disclosure Agreement",
        "status": "completed",
        "totalPages": 2,
        "signingParties": [...]
      }
    ]
  }
}
```

---

### 7. Get Envelope Signers

Retrieve all signers for an envelope (aggregated from all documents).

**Endpoint**: `GET /api/v1/merchant/signatures/envelopes/:envelopeId/signers`

**URL Parameters**:
- `envelopeId` (string, required) - Envelope ID

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "signers": [
      {
        "name": "John Smith",
        "email": "john@example.com",
        "role": "Employee",
        "totalDocuments": 3,
        "completedDocuments": 1,
        "overallStatus": "in_progress",
        "documentStatuses": [
          {
            "documentId": "507f1f77bcf86cd799439011",
            "documentTitle": "Employment Contract",
            "status": "signed",
            "signedAt": "2025-10-19T12:00:00Z"
          },
          {
            "documentId": "507f1f77bcf86cd799439012",
            "documentTitle": "Non-Disclosure Agreement",
            "status": "pending"
          }
        ]
      }
    ]
  }
}
```

**Overall Status Values**:
- `not_started` - Haven't signed any documents
- `in_progress` - Signed some but not all documents
- `completed` - Signed all documents in envelope

---

### 8. Add Signer to Envelope

Add a signing party to an envelope (will be applied to all documents).

**Endpoint**: `POST /api/v1/merchant/signatures/envelopes/:envelopeId/signers`

**URL Parameters**:
- `envelopeId` (string, required) - Envelope ID

**Request Body**:
```json
{
  "name": "John Smith",
  "email": "john@example.com",
  "role": "Employee",
  "order": 1,
  "requireIdVerification": false
}
```

**Request Parameters**:
- `name` (string, required) - Signer's full name
- `email` (string, required) - Signer's email address
- `role` (string, optional) - Signer's role
- `order` (number, required) - Signing order
- `requireIdVerification` (boolean, optional) - Require ID verification

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "signerId": "507f1f77bcf86cd799439012",
    "name": "John Smith",
    "email": "john@example.com",
    "addedToDocuments": 3
  },
  "message": "Signer added to envelope successfully"
}
```

**Notes**:
- Signer is automatically added to all documents in the envelope
- Can only add signers to envelopes in `draft` status
- System finds or creates user account for the signer

---

### 9. Remove Signer from Envelope

Remove a signing party from an envelope (removes from all documents).

**Endpoint**: `POST /api/v1/merchant/signatures/envelopes/:envelopeId/signers/:signerId/remove`

**URL Parameters**:
- `envelopeId` (string, required) - Envelope ID
- `signerId` (string, required) - Signer ID to remove

**Success Response** (200):
```json
{
  "success": true,
  "message": "Signer removed from envelope successfully",
  "data": {
    "removedFromDocuments": 3
  }
}
```

**Error Response** (400):
```json
{
  "error": true,
  "message": "Cannot remove signers from sent envelope"
}
```

**Notes**:
- Signer is removed from all documents in the envelope
- Cannot remove signers after envelope is sent
- All fields assigned to the signer are also removed

---

### 10. Send Envelope

Send an envelope and all its documents to signers.

**Endpoint**: `POST /api/v1/merchant/signatures/envelopes/:envelopeId/send`

**URL Parameters**:
- `envelopeId` (string, required) - Envelope ID

**Request Body**:
```json
{
  "configuration": {
    "enforceSigningOrder": true,
    "requireIdVerification": false,
    "useDigitalSignature": true,
    "expiration": {
      "type": "relative",
      "relativeDays": 30
    }
  },
  "message": "Please review and sign these onboarding documents."
}
```

**Request Parameters**:
- `configuration` (object, required) - Envelope configuration
  - `enforceSigningOrder` (boolean) - Require sequential signing
  - `requireIdVerification` (boolean) - Require ID verification
  - `useDigitalSignature` (boolean) - Apply digital certificate
  - `expiration` (object) - Expiration settings
- `message` (string, optional) - Custom message for signers

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "envelopeId": "507f1f77bcf86cd799439040",
    "documentsSent": 3,
    "signerCount": 2,
    "expiresAt": "2025-11-18T10:00:00Z",
    "creditCost": 15
  },
  "message": "Envelope sent successfully"
}
```

**Error Responses**:

400 Bad Request - No documents:
```json
{
  "error": true,
  "message": "Envelope must contain at least one document"
}
```

400 Bad Request - No signers:
```json
{
  "error": true,
  "message": "Envelope must have at least one signer"
}
```

400 Bad Request - Insufficient credits:
```json
{
  "error": true,
  "message": "Insufficient credits. Required: 15, Available: 10"
}
```

**Credit Costs**:
- Regular signature: 1 credit per document
- Digital signature: 5 credits per document
- Total cost = (number of documents) × (credit per document)

**Process Flow**:
1. Validates envelope has documents and signers
2. Applies configuration to all documents
3. Synchronizes signers across all documents
4. Calculates total credit cost
5. Checks credit balance
6. Deducts credits
7. Updates envelope and document statuses
8. Queues signing invitation emails

**Notes**:
- All documents in envelope sent simultaneously
- Signers receive one email with links to all documents
- Configuration applied consistently across all documents
- Status changes from `draft` to `sent`

---

### 11. Cancel Envelope

Cancel an envelope and all its documents.

**Endpoint**: `POST /api/v1/merchant/signatures/envelopes/:envelopeId/cancel`

**URL Parameters**:
- `envelopeId` (string, required) - Envelope ID

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "envelopeId": "507f1f77bcf86cd799439040",
    "status": "cancelled",
    "documentsCancelled": 3
  },
  "message": "Envelope cancelled successfully"
}
```

**Error Response** (400):
```json
{
  "error": true,
  "message": "Cannot cancel completed envelope"
}
```

**Notes**:
- Cancels all documents in the envelope
- Can only cancel `draft` or `sent` envelopes
- Signers are notified of cancellation
- Cancelled envelopes cannot be reopened

---

### 12. Get Envelope History

Retrieve complete history and audit trail for an envelope.

**Endpoint**: `GET /api/v1/merchant/signatures/envelopes/:envelopeId/history`

**URL Parameters**:
- `envelopeId` (string, required) - Envelope ID

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "history": [
      {
        "_id": "507f1f77bcf86cd799439050",
        "action": "envelope_created",
        "description": "Envelope created",
        "timestamp": "2025-10-15T09:00:00Z",
        "userId": "507f1f77bcf86cd799439030"
      },
      {
        "_id": "507f1f77bcf86cd799439051",
        "action": "document_added",
        "description": "Document 'Employment Contract' added to envelope",
        "timestamp": "2025-10-15T09:15:00Z",
        "metadata": {
          "documentId": "507f1f77bcf86cd799439011",
          "documentTitle": "Employment Contract"
        }
      },
      {
        "_id": "507f1f77bcf86cd799439052",
        "action": "envelope_sent",
        "description": "Envelope sent to 2 signers with 3 documents",
        "timestamp": "2025-10-15T10:00:00Z",
        "metadata": {
          "signerCount": 2,
          "documentCount": 3,
          "creditCost": 15
        }
      }
    ]
  }
}
```

**History Action Types**:
- `envelope_created` - Envelope initially created
- `document_added` - Document added to envelope
- `document_removed` - Document removed from envelope
- `signer_added` - Signer added to envelope
- `signer_removed` - Signer removed from envelope
- `envelope_sent` - Envelope sent to signers
- `envelope_cancelled` - Envelope cancelled
- `envelope_completed` - All documents completed

---

### 13. Get Envelope Signature History

Retrieve detailed signature history for all documents in an envelope.

**Endpoint**: `GET /api/v1/merchant/signatures/envelopes/:envelopeId/signature-history`

**URL Parameters**:
- `envelopeId` (string, required) - Envelope ID

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "signatures": [
      {
        "documentId": "507f1f77bcf86cd799439011",
        "documentTitle": "Employment Contract",
        "signerId": "507f1f77bcf86cd799439012",
        "signerName": "John Smith",
        "signerEmail": "john@example.com",
        "status": "signed",
        "signedAt": "2025-10-19T12:00:00Z",
        "ipAddress": "192.168.1.1",
        "userAgent": "Mozilla/5.0..."
      },
      {
        "documentId": "507f1f77bcf86cd799439012",
        "documentTitle": "Non-Disclosure Agreement",
        "signerId": "507f1f77bcf86cd799439012",
        "signerName": "John Smith",
        "signerEmail": "john@example.com",
        "status": "pending"
      }
    ],
    "summary": {
      "totalSignatures": 6,
      "completedSignatures": 3,
      "pendingSignatures": 3,
      "percentComplete": 50
    }
  }
}
```

---

### 14. Download All Envelope Documents

Download a ZIP file containing all documents in an envelope.

**Endpoint**: `GET /api/v1/merchant/signatures/envelopes/:envelopeId/download-all`

**URL Parameters**:
- `envelopeId` (string, required) - Envelope ID

**Query Parameters**:
- `version` (string, optional) - Which version to download
  - `latest` (default) - Latest version of each document
  - `original` - Original unsigned versions
  - `final` - Final signed versions only (if completed)

**Success Response** (200):
- **Content-Type**: `application/zip`
- **Content-Disposition**: `attachment; filename="envelope-name.zip"`
- **Body**: ZIP file containing all PDFs

**ZIP File Structure**:
```
Employee-Onboarding-Package.zip
├── 1-Employment-Contract.pdf
├── 2-Non-Disclosure-Agreement.pdf
└── 3-Employee-Handbook-Acknowledgment.pdf
```

**Error Responses**:

404 Not Found:
```json
{
  "error": true,
  "message": "Envelope not found"
}
```

500 Internal Server Error:
```json
{
  "error": true,
  "message": "Failed to generate ZIP file",
  "details": "..."
}
```

**Notes**:
- Files are numbered by their order in the envelope
- Filenames sanitized to be filesystem-safe
- Maximum ZIP size: 100MB
- Generation can take 10-30 seconds for large envelopes

---

### 15. Get Documents Associated with Envelope

Get all documents that reference a specific envelope.

**Endpoint**: `GET /api/v1/merchant/signatures/documents/:documentId/envelopes`

**URL Parameters**:
- `documentId` (string, required) - Document ID

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "envelopes": [
      {
        "_id": "507f1f77bcf86cd799439040",
        "name": "Employee Onboarding Package",
        "status": "sent",
        "documentCount": 3,
        "createdAt": "2025-10-15T09:00:00Z"
      }
    ]
  }
}
```

**Notes**:
- Documents can technically be in multiple envelopes
- Most common use case: checking if document is part of an envelope
- Used by UI to show envelope context when editing document

---

### 16. Get Envelope Context for Document

Get envelope context information when viewing/editing a specific document.

**Endpoint**: `GET /api/v1/merchant/signatures/documents/:documentId/envelope-context`

**URL Parameters**:
- `documentId` (string, required) - Document ID

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "isPartOfEnvelope": true,
    "envelopes": [
      {
        "_id": "507f1f77bcf86cd799439040",
        "name": "Employee Onboarding Package",
        "status": "sent",
        "documentCount": 3,
        "siblingDocuments": [
          {
            "_id": "507f1f77bcf86cd799439012",
            "title": "Non-Disclosure Agreement"
          },
          {
            "_id": "507f1f77bcf86cd799439013",
            "title": "Employee Handbook"
          }
        ]
      }
    ]
  }
}
```

**Notes**:
- Shows which envelope(s) contain this document
- Includes sibling documents in the same envelope
- Used in document editor to show envelope context

---

## Envelope Workflow

### Complete Envelope Creation and Sending Flow

```javascript
// 1. Create envelope
const createResponse = await fetch('/api/v1/merchant/signatures/envelopes', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    name: 'Employee Onboarding Package'
  })
});
const { data: { id: envelopeId } } = await createResponse.json();

// 2. Create documents (see Document API for details)
const doc1 = await createDocument('Employment Contract');
const doc2 = await createDocument('Non-Disclosure Agreement');
const doc3 = await createDocument('Employee Handbook');

// 3. Add documents to envelope
await fetch(`/api/v1/merchant/signatures/envelopes/${envelopeId}/documents`, {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({ documentId: doc1._id })
});

await fetch(`/api/v1/merchant/signatures/envelopes/${envelopeId}/documents`, {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({ documentId: doc2._id })
});

await fetch(`/api/v1/merchant/signatures/envelopes/${envelopeId}/documents`, {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({ documentId: doc3._id })
});

// 4. Add signers to envelope
await fetch(`/api/v1/merchant/signatures/envelopes/${envelopeId}/signers`, {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    name: 'John Smith',
    email: 'john@example.com',
    role: 'Employee',
    order: 1
  })
});

// 5. Send envelope
await fetch(`/api/v1/merchant/signatures/envelopes/${envelopeId}/send`, {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    configuration: {
      enforceSigningOrder: false,
      requireIdVerification: false,
      useDigitalSignature: true,
      expiration: {
        type: 'relative',
        relativeDays: 30
      }
    },
    message: 'Welcome! Please sign these onboarding documents.'
  })
});
```

### Monitoring Envelope Progress

```javascript
// Get envelope with progress
const response = await fetch(
  `/api/v1/merchant/signatures/envelopes/${envelopeId}`,
  {
    headers: { 'Authorization': `Bearer ${token}` }
  }
);
const { data } = await response.json();

console.log(`Progress: ${data.progress.completedSignatures}/${data.progress.totalSignatures}`);
console.log(`Percentage: ${data.progress.percentage}%`);

// Get detailed signature history
const historyResponse = await fetch(
  `/api/v1/merchant/signatures/envelopes/${envelopeId}/signature-history`,
  {
    headers: { 'Authorization': `Bearer ${token}` }
  }
);
const { data: historyData } = await historyResponse.json();

// Show which documents each signer has completed
historyData.signatures.forEach(sig => {
  console.log(`${sig.signerName}: ${sig.documentTitle} - ${sig.status}`);
});
```

## Best Practices

### 1. Document Organization
- Group related documents that need the same signers
- Use clear, descriptive envelope names
- Add envelope description explaining the package purpose

### 2. Signer Experience
- Limit envelopes to 5-10 documents maximum
- Order documents logically (most important first)
- Include helpful message explaining the envelope contents

### 3. Configuration
- Apply consistent configuration across envelope documents
- Use signing order enforcement for dependent documents
- Set appropriate expiration dates (15-30 days typical)

### 4. Monitoring
- Check envelope progress regularly via the GET endpoint
- Use signature history to identify bottlenecks
- Send reminders to pending signers

### 5. Credit Management
- Calculate total credit cost before sending
- Consider using regular signatures for non-critical documents
- Use digital signatures only where legally required

## Error Codes

| Status Code | Description |
|------------|-------------|
| 200 | Success |
| 400 | Bad Request (validation error) |
| 401 | Unauthorized (invalid/missing token) |
| 403 | Forbidden (insufficient permissions) |
| 404 | Not Found (envelope or document not found) |
| 500 | Internal Server Error |

## Rate Limits

- **Envelope Creation**: 50 per hour
- **Envelope Sending**: 100 per hour
- **Document Operations**: 500 per hour
- **API Calls**: 1000 per hour per organisation

## Related Documentation

- [Merchant Signature Documents API](./merchant-signature-documents.md)
- [Envelope Templates API](./envelope-templates.md)
- [Document Templates API](./document-templates.md)
- [API Overview](./README.md)

