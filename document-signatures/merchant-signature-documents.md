# Merchant Signature Documents API

## Overview

The Merchant Signature Documents API provides comprehensive endpoints for merchants to create, manage, configure, and send documents for signature. These endpoints require merchant authentication and operate on documents owned by the authenticated organization.

**Base Path**: `/api/v1/merchant/signatures`

**Authentication**: Merchant JWT token in Authorization header

## Endpoints

### 1. List Documents

Retrieve a paginated list of signature documents with filtering and sorting capabilities.

**Endpoint**: `GET /api/v1/merchant/signatures/documents`

**Query Parameters**:
- `query` (string, JSON encoded) - Contains search/filter/sort parameters:
  ```json
  {
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
  ```

**Success Response** (200):
```json
{
  "records": [
    {
      "_id": "507f1f77bcf86cd799439011",
      "title": "Employment Contract",
      "status": "enqueued",
      "signingParties": [
        {
          "name": "John Smith",
          "email": "john@example.com",
          "status": "pending"
        }
      ],
      "progress": {
        "completed": 0,
        "total": 2
      },
      "sentAt": "2025-10-15T10:00:00Z",
      "expiresAt": "2025-11-14T10:00:00Z",
      "createdAt": "2025-10-15T09:00:00Z"
    }
  ],
  "columns": [
    {
      "header": "Document",
      "path": "title",
      "type": "link",
      "urlPrefix": "/portal/documents/{id}/edit",
      "sortable": true
    },
    {
      "header": "Signers",
      "path": "signingParties",
      "type": "signers"
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
      "options": ["draft", "enqueued", "ready_for_digital_signature", "completed", "expired", "cancelled"]
    },
    {
      "name": "createdAt",
      "label": "Created At",
      "type": "date-range"
    }
  ],
  "appliedFilters": {
    "status": ["draft", "enqueued"]
  },
  "paging": {
    "recordCount": 156,
    "pageCount": 16,
    "currentPage": 1
  }
}
```

**Filter Types**:
- `status` - Multi-select filter for document status
- `createdAt` - Date range filter (use `createdAt_start` and `createdAt_end`)
- `sentAt` - Date range filter for send date

**Sortable Fields**:
- `title` - Document title
- `createdAt` - Creation date
- `sentAt` - Send date
- `expiresAt` - Expiration date
- `status` - Document status

---

### 2. Create Document

Create a new signature document in draft status.

**Endpoint**: `POST /api/v1/merchant/signatures/documents`

**Request Body**:
```json
{
  "title": "Employment Contract",
  "description": "Standard employment agreement for new hires",
  "signingParties": [],
  "fields": []
}
```

**Request Parameters**:
- `title` (string, required) - Document title
- `description` (string, optional) - Document description
- `signingParties` (array, optional) - Array of signers (can be empty for draft)
- `fields` (array, optional) - Array of form fields (can be empty for draft)

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "document": {
      "_id": "507f1f77bcf86cd799439011",
      "title": "Employment Contract",
      "description": "Standard employment agreement for new hires",
      "status": "draft",
      "signingParties": [],
      "fields": [],
      "clientId": "507f1f77bcf86cd799439030",
      "organisationId": "507f1f77bcf86cd799439031",
      "createdAt": "2025-10-19T10:00:00Z",
      "updatedAt": "2025-10-19T10:00:00Z"
    }
  },
  "message": "Document created successfully"
}
```

**Error Responses**:

400 Bad Request:
```json
{
  "error": true,
  "message": "Document title is required"
}
```

401 Unauthorized:
```json
{
  "error": true,
  "message": "Authentication required - no client ID found"
}
```

---

### 3. Get Document by ID

Retrieve a specific document by its ID.

**Endpoint**: `GET /api/v1/merchant/signatures/documents/:documentId`

**URL Parameters**:
- `documentId` (string, required) - Document unique identifier

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "document": {
      "_id": "507f1f77bcf86cd799439011",
      "title": "Employment Contract",
      "description": "Standard employment agreement",
      "status": "draft",
      "signingParties": [
        {
          "_id": "507f1f77bcf86cd799439012",
          "name": "John Smith",
          "email": "john@example.com",
          "role": "Employee",
          "order": 1,
          "status": "pending"
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
        }
      ],
      "totalPages": 3,
      "pageImages": [
        {
          "pageNumber": 1,
          "width": 612,
          "height": 792,
          "imageUrl": "https://s3.amazonaws.com/..."
        }
      ],
      "fileKey": "signatures/documents/507f.../document.pdf",
      "pdfUrl": "https://s3.amazonaws.com/.../document.pdf",
      "createdAt": "2025-10-15T09:00:00Z",
      "updatedAt": "2025-10-19T10:00:00Z"
    }
  }
}
```

**Error Response** (404):
```json
{
  "error": true,
  "message": "Document not found"
}
```

---

### 4. Upload Document PDF

Upload a PDF file for an existing document.

**Endpoint**: `POST /api/v1/merchant/signatures/documents/upload`

**Content-Type**: `multipart/form-data`

**Form Parameters**:
- `file` (file, required) - PDF file to upload
- `documentId` (string, required) - Document ID to associate the file with

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "documentId": "507f1f77bcf86cd799439011",
    "fileKey": "signatures/documents/507f.../document.pdf",
    "totalPages": 3,
    "pageImages": [
      {
        "pageNumber": 1,
        "width": 612,
        "height": 792,
        "imageUrl": "https://s3.amazonaws.com/..."
      }
    ]
  },
  "message": "Document uploaded successfully"
}
```

**Notes**:
- Maximum file size: 10MB
- Accepted format: PDF only
- Automatically generates page images for preview
- Updates document's `fileKey`, `totalPages`, and `pageImages`

---

### 5. Get File Download URL

Get a signed URL for downloading a document file.

**Endpoint**: `GET /api/v1/merchant/signatures/documents/file`

**Query Parameters**:
- `key` (string, required) - S3 file key to download

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "url": "https://s3.amazonaws.com/bucket/path/to/file.pdf?signature=..."
  }
}
```

**Notes**:
- Signed URL expires after 1 hour
- Used for accessing uploaded PDF files
- Requires valid merchant authentication

---

### 6. Get Document Content

Retrieve document content metadata.

**Endpoint**: `GET /api/v1/merchant/signatures/documents/content`

**Query Parameters**:
- `documentId` (string, required) - Document ID

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "fileKey": "signatures/documents/507f.../document.pdf",
    "pdfUrl": "https://s3.amazonaws.com/.../document.pdf",
    "totalPages": 3,
    "pageImages": [...]
  }
}
```

---

### 7. Get Document Pages

Retrieve page images for a document.

**Endpoint**: `GET /api/v1/merchant/signatures/documents/:documentId/pages`

**URL Parameters**:
- `documentId` (string, required) - Document ID

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "pages": [
      {
        "pageNumber": 1,
        "width": 612,
        "height": 792,
        "widthPx": 612,
        "heightPx": 792,
        "imageUrl": "https://s3.amazonaws.com/...",
        "imageKey": "signatures/.../page-1.png"
      }
    ]
  }
}
```

**Notes**:
- Returns fresh signed URLs for all page images
- URLs expire after 1 hour
- Used for document editor/viewer

---

### 8. Save Document

Update document details, fields, and signing parties.

**Endpoint**: `POST /api/v1/merchant/signatures/documents/:documentId/save`

**URL Parameters**:
- `documentId` (string, required) - Document ID

**Request Body**:
```json
{
  "title": "Updated Employment Contract",
  "description": "Updated description",
  "signingParties": [
    {
      "_id": "507f1f77bcf86cd799439012",
      "name": "John Smith",
      "email": "john@example.com",
      "role": "Employee",
      "order": 1
    },
    {
      "name": "Jane Manager",
      "email": "jane@company.com",
      "role": "Employer",
      "order": 2
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
    }
  ],
  "configuration": {
    "enforceSigningOrder": true,
    "requireIdVerification": false,
    "useDigitalSignature": true
  }
}
```

**Request Parameters**:
- `title` (string, optional) - Document title
- `description` (string, optional) - Document description
- `signingParties` (array, optional) - Array of signing parties
  - System automatically finds or creates user accounts for new signers
- `fields` (array, optional) - Array of form fields
- `configuration` (object, optional) - Document configuration

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "_id": "507f1f77bcf86cd799439011",
    "title": "Updated Employment Contract",
    "description": "Updated description",
    "signingParties": [...],
    "fields": [...],
    "updatedAt": "2025-10-19T11:00:00Z"
  },
  "message": "Document saved successfully"
}
```

**Notes**:
- Automatically finds or creates user accounts for signers based on email
- Can only update documents in `draft` status
- All changes are logged in document history

---

### 9. Duplicate Document

Create a copy of an existing document with empty fields and signers.

**Endpoint**: `POST /api/v1/merchant/signatures/documents/:documentId/duplicate`

**URL Parameters**:
- `documentId` (string, required) - Source document ID

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "documentId": "507f1f77bcf86cd799439050",
    "title": "Employment Contract (Copy)",
    "status": "draft",
    "totalPages": 3
  },
  "message": "Document duplicated successfully"
}
```

**Notes**:
- Creates a new PDF file copy in S3
- Reuses page images (they're immutable)
- Clears signing parties and fields
- Title appended with " (Copy)"
- Tracks source document via `templateSourceId`

---

### 10. Send Document for Signing

Send a document to signing parties via email.

**Endpoint**: `POST /api/v1/merchant/signatures/documents/:documentId/send`

**URL Parameters**:
- `documentId` (string, required) - Document ID

**Request Body**:
```json
{
  "signingParties": [
    {
      "name": "John Smith",
      "email": "john@example.com",
      "role": "Employee",
      "order": 1,
      "requireIdVerification": false
    },
    {
      "name": "Jane Manager",
      "email": "jane@company.com",
      "role": "Employer",
      "order": 2,
      "requireIdVerification": false
    }
  ],
  "configuration": {
    "enforceSigningOrder": true,
    "requireIdVerification": false,
    "useDigitalSignature": true,
    "expiration": {
      "type": "relative",
      "relativeDays": 30
    }
  },
  "message": "Please review and sign this employment contract."
}
```

**Request Parameters**:
- `signingParties` (array, required) - Array of signers
  - `name` (string, required) - Signer's full name
  - `email` (string, required) - Signer's email
  - `role` (string, optional) - Signer's role
  - `order` (number, required) - Signing order (1, 2, 3...)
  - `requireIdVerification` (boolean, optional) - Require ID verification
- `configuration` (object, required) - Document configuration
  - `enforceSigningOrder` (boolean) - Require sequential signing
  - `requireIdVerification` (boolean) - Require ID verification
  - `useDigitalSignature` (boolean) - Apply digital certificate after completion
  - `expiration` (object) - Expiration settings
    - `type` (string) - "none", "relative", or "fixed"
    - `relativeDays` (number) - Days until expiration (if type is "relative")
    - `fixedDate` (string) - ISO date (if type is "fixed")
- `message` (string, optional) - Custom message to include in emails

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "_id": "507f1f77bcf86cd799439011",
    "title": "Employment Contract",
    "status": "enqueued",
    "signingParties": [...],
    "expiresAt": "2025-11-18T10:00:00Z"
  },
  "message": "Document sent successfully"
}
```

**Credit Costs**:
- Regular signature: 1 credit
- Digital signature: 5 credits

**Error Responses**:

400 Bad Request - Insufficient credits:
```json
{
  "error": true,
  "message": "Insufficient credits. Required: 5, Available: 2"
}
```

**Process Flow**:
1. Validates document exists and is in draft status
2. Processes signing parties (finds/creates user accounts)
3. Calculates credit cost based on configuration
4. Checks organisation credit balance
5. Deducts credits and creates transaction record
6. Updates document status to `enqueued`
7. Queues document for email sending (handled by machine runner)
8. Creates audit log entries

**Notes**:
- Document status changes from `draft` to `enqueued`
- Signing invitation emails sent asynchronously by machine runner
- Each signer receives unique signing token and URL
- Credits deducted immediately upon sending
- If credit charge fails, document reverts to draft status

---

### 11. Cancel Document

Cancel a document that has been sent for signing.

**Endpoint**: `POST /api/v1/merchant/signatures/documents/:documentId/cancel`

**URL Parameters**:
- `documentId` (string, required) - Document ID

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "_id": "507f1f77bcf86cd799439011",
    "status": "cancelled",
    "cancelledAt": "2025-10-19T12:00:00Z"
  },
  "message": "Document cancelled successfully"
}
```

**Error Response** (404):
```json
{
  "error": true,
  "message": "Document not found or cannot be cancelled"
}
```

**Notes**:
- Can only cancel documents in `pending` or `draft` status
- Cancelled documents cannot be reopened
- Signers are notified of cancellation

---

### 12. Delete Document

Soft delete a document.

**Endpoint**: `POST /api/v1/merchant/signatures/documents/:documentId/delete`

**URL Parameters**:
- `documentId` (string, required) - Document ID

**Success Response** (200):
```json
{
  "success": true,
  "message": "Document deleted successfully"
}
```

**Notes**:
- Soft delete - document marked as deleted but not removed from database
- Cannot delete documents that have been sent or completed
- PDF files remain in S3 for audit purposes

---

### 13. Get Document Signers

Retrieve all signers for a document.

**Endpoint**: `GET /api/v1/merchant/signatures/documents/:documentId/signers`

**URL Parameters**:
- `documentId` (string, required) - Document ID

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "signers": [
      {
        "_id": "507f1f77bcf86cd799439012",
        "name": "John Smith",
        "email": "john@example.com",
        "role": "Employee",
        "order": 1,
        "status": "signed",
        "signedAt": "2025-10-19T12:00:00Z",
        "idVerificationStatus": "verified"
      },
      {
        "_id": "507f1f77bcf86cd799439013",
        "name": "Jane Manager",
        "email": "jane@company.com",
        "role": "Employer",
        "order": 2,
        "status": "pending",
        "idVerificationStatus": "not_required"
      }
    ]
  }
}
```

---

### 14. Get Document History

Retrieve document change history and audit log.

**Endpoint**: `GET /api/v1/merchant/signatures/documents/:documentId/history`

**URL Parameters**:
- `documentId` (string, required) - Document ID

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "history": [
      {
        "_id": "507f1f77bcf86cd799439020",
        "action": "document_created",
        "description": "Document created",
        "timestamp": "2025-10-15T09:00:00Z",
        "userId": "507f1f77bcf86cd799439030"
      },
      {
        "_id": "507f1f77bcf86cd799439021",
        "action": "document_sent",
        "description": "Document sent for signing",
        "timestamp": "2025-10-15T10:00:00Z",
        "userId": "507f1f77bcf86cd799439030"
      },
      {
        "_id": "507f1f77bcf86cd799439022",
        "action": "document_signed",
        "description": "Document signed by John Smith",
        "timestamp": "2025-10-19T12:00:00Z",
        "metadata": {
          "signerId": "507f1f77bcf86cd799439012",
          "signerName": "John Smith"
        }
      }
    ]
  }
}
```

---

### 15. Resend Signing Invitation

Resend signing invitation email to a specific signer.

**Endpoint**: `POST /api/v1/merchant/signatures/documents/:documentId/signers/:signerId/resend`

**URL Parameters**:
- `documentId` (string, required) - Document ID
- `signerId` (string, required) - Signer ID

**Success Response** (200):
```json
{
  "success": true,
  "message": "Signing invitation resent successfully"
}
```

**Notes**:
- Can only resend to signers with `pending` status
- Generates new signing token
- Queues email for sending

---

### 16. Send Reminder to Signer

Send a reminder email to a signer who hasn't completed signing.

**Endpoint**: `POST /api/v1/merchant/signatures/documents/:documentId/signers/:signerId/remind`

**URL Parameters**:
- `documentId` (string, required) - Document ID
- `signerId` (string, required) - Signer ID

**Request Body**:
```json
{
  "message": "Reminder: Please sign the employment contract at your earliest convenience."
}
```

**Success Response** (200):
```json
{
  "success": true,
  "message": "Reminder sent successfully"
}
```

**Notes**:
- Custom message included in reminder email
- Reminder action logged in audit trail
- Can only remind pending signers

---

### 17. Get Document Analytics

Retrieve analytics and statistics for a document.

**Endpoint**: `GET /api/v1/merchant/signatures/documents/:documentId/analytics`

**URL Parameters**:
- `documentId` (string, required) - Document ID

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "views": 12,
    "uniqueViewers": 2,
    "completionRate": 50,
    "averageTimeToSign": "2.5 hours",
    "signerActivity": [
      {
        "signerId": "507f1f77bcf86cd799439012",
        "signerName": "John Smith",
        "views": 8,
        "timeSpent": "15 minutes",
        "status": "signed",
        "signedAt": "2025-10-19T12:00:00Z"
      }
    ]
  }
}
```

---

### 18. Get Document Status

Get current status of a document.

**Endpoint**: `GET /api/v1/merchant/signatures/documents/:documentId/status`

**URL Parameters**:
- `documentId` (string, required) - Document ID

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "documentId": "507f1f77bcf86cd799439011",
    "status": "enqueued",
    "signingStatus": "idle",
    "progress": {
      "completed": 1,
      "total": 2,
      "percentage": 50
    },
    "expiresAt": "2025-11-18T10:00:00Z",
    "isExpired": false
  }
}
```

**Status Values**:
- `draft` - Document being created
- `enqueued` - Sent to signers, awaiting signatures
- `ready_for_digital_signature` - All signed, awaiting digital certificate
- `completed` - Fully signed and processed
- `expired` - Document passed expiry date
- `cancelled` - Document cancelled

---

### 19. Download Document (Merchant)

Download document PDF for merchant.

**Endpoint**: `GET /api/v1/merchant/signatures/documents/:documentId/download`

**URL Parameters**:
- `documentId` (string, required) - Document ID

**Query Parameters**:
- `version` (number, optional) - Specific version to download (default: latest)

**Success Response** (200):
- **Content-Type**: `application/pdf`
- **Content-Disposition**: `attachment; filename="document.pdf"`
- **Body**: PDF binary data

---

### 20. Get Signing Status for Signer

Get the current signing status and progress for a specific signer.

**Endpoint**: `GET /api/v1/merchant/signatures/documents/:documentId/signing-status/:signerId`

**URL Parameters**:
- `documentId` (string, required) - Document ID
- `signerId` (string, required) - Signer ID

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "signerId": "507f1f77bcf86cd799439012",
    "status": "signed",
    "signedAt": "2025-10-19T12:00:00Z",
    "idVerificationStatus": "verified",
    "idVerificationCompletedAt": "2025-10-19T11:30:00Z",
    "signingDetails": {
      "ipAddress": "192.168.1.1",
      "userAgent": "Mozilla/5.0...",
      "fieldCount": 3
    }
  }
}
```

---

### 21. Validate ID Verification Token

Validate an identity verification token for a signer.

**Endpoint**: `POST /api/v1/merchant/signatures/validate-idv-token`

**Request Body**:
```json
{
  "token": "idv_token_xyz123...",
  "documentId": "507f1f77bcf86cd799439011",
  "signerId": "507f1f77bcf86cd799439012"
}
```

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "valid": true,
    "signer": {
      "name": "John Smith",
      "email": "john@example.com",
      "idVerificationStatus": "verified"
    }
  }
}
```

---

### 22. Generate Dropdown Options

Generate dropdown field options for document forms.

**Endpoint**: `POST /api/v1/merchant/signatures/generate-dropdown-options`

**Request Body**:
```json
{
  "fieldType": "country",
  "customValues": []
}
```

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "options": [
      { "value": "US", "label": "United States" },
      { "value": "GB", "label": "United Kingdom" },
      { "value": "CA", "label": "Canada" }
    ]
  }
}
```

---

### 23. Sign Document (Merchant Direct Signing)

Allows merchant to sign a document directly on behalf of a signer (for in-person signing scenarios).

**Endpoint**: `POST /api/v1/merchant/signatures/documents/sign`

**Request Body**:
```json
{
  "documentId": "507f1f77bcf86cd799439011",
  "signerId": "507f1f77bcf86cd799439012",
  "fieldValues": {
    "signature_1": "data:image/png;base64,...",
    "date_1": "2025-10-19"
  }
}
```

**Success Response** (200):
```json
{
  "success": true,
  "message": "Document signed successfully",
  "data": {
    "documentId": "507f1f77bcf86cd799439011",
    "signerId": "507f1f77bcf86cd799439012",
    "pdfVersion": 2,
    "allSigned": false,
    "remainingSigners": 1
  }
}
```

**Notes**:
- Used for in-person signing scenarios
- Requires merchant authentication
- Same workflow as public signing endpoint
- Creates audit log entry with merchant's credentials

---

### 24. Get Envelope Context

Get envelope information for a document (if document is part of an envelope).

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
        "createdAt": "2025-10-15T09:00:00Z"
      }
    ]
  }
}
```

---

## Field Types

When creating or updating document fields, the following types are supported:

### Signature Field
```json
{
  "id": "signature_1",
  "type": "signature",
  "label": "Signature",
  "required": true,
  "assignedTo": "507f1f77bcf86cd799439012",
  "page": 1,
  "position": {
    "x": 100,
    "y": 500,
    "width": 200,
    "height": 50
  }
}
```

### Text Field
```json
{
  "id": "text_1",
  "type": "text",
  "label": "Full Name",
  "required": true,
  "assignedTo": "507f1f77bcf86cd799439012",
  "page": 1,
  "position": {
    "x": 100,
    "y": 550,
    "width": 300,
    "height": 30
  }
}
```

### Date Field
```json
{
  "id": "date_1",
  "type": "date",
  "label": "Date",
  "required": true,
  "assignedTo": "507f1f77bcf86cd799439012",
  "page": 1,
  "position": {
    "x": 450,
    "y": 500,
    "width": 100,
    "height": 30
  },
  "format": "YYYY-MM-DD"
}
```

### Email Field
```json
{
  "id": "email_1",
  "type": "email",
  "label": "Email Address",
  "required": true,
  "assignedTo": "507f1f77bcf86cd799439012",
  "page": 1,
  "position": {
    "x": 100,
    "y": 600,
    "width": 300,
    "height": 30
  }
}
```

### Checkbox Field
```json
{
  "id": "checkbox_1",
  "type": "checkbox",
  "label": "I agree to terms",
  "required": true,
  "assignedTo": "507f1f77bcf86cd799439012",
  "page": 2,
  "position": {
    "x": 100,
    "y": 700,
    "width": 20,
    "height": 20
  }
}
```

### Dropdown Field
```json
{
  "id": "dropdown_1",
  "type": "dropdown",
  "label": "Country",
  "required": true,
  "assignedTo": "507f1f77bcf86cd799439012",
  "page": 1,
  "position": {
    "x": 100,
    "y": 650,
    "width": 200,
    "height": 30
  },
  "options": [
    { "value": "US", "label": "United States" },
    { "value": "GB", "label": "United Kingdom" }
  ]
}
```

## Configuration Options

### Document Configuration
```json
{
  "configuration": {
    "enforceSigningOrder": true,
    "requireIdVerification": false,
    "useDigitalSignature": true,
    "allowComments": true,
    "allowDecline": true,
    "expiration": {
      "type": "relative",
      "relativeDays": 30
    },
    "notifications": {
      "sendReminders": true,
      "reminderDays": [7, 3, 1]
    }
  }
}
```

**Configuration Fields**:
- `enforceSigningOrder` (boolean) - Require signers to sign in specified order
- `requireIdVerification` (boolean) - Require ID verification for all signers
- `useDigitalSignature` (boolean) - Apply digital certificate after completion
- `allowComments` (boolean) - Allow signers to add comments
- `allowDecline` (boolean) - Allow signers to decline signing
- `expiration` (object) - Document expiration settings
- `notifications` (object) - Notification settings

## Common Use Cases

### Creating and Sending a Document

```javascript
// 1. Create document
const createResponse = await fetch('/api/v1/merchant/signatures/documents', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    title: 'Employment Contract',
    description: 'Standard employment agreement'
  })
});
const { data: { document } } = await createResponse.json();

// 2. Upload PDF
const formData = new FormData();
formData.append('file', pdfFile);
formData.append('documentId', document._id);

await fetch('/api/v1/merchant/signatures/documents/upload', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`
  },
  body: formData
});

// 3. Configure fields and signers
await fetch(`/api/v1/merchant/signatures/documents/${document._id}/save`, {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    signingParties: [
      {
        name: 'John Smith',
        email: 'john@example.com',
        role: 'Employee',
        order: 1
      }
    ],
    fields: [
      {
        id: 'signature_1',
        type: 'signature',
        label: 'Employee Signature',
        assignedTo: '...',
        page: 1,
        position: { x: 100, y: 500, width: 200, height: 50 }
      }
    ]
  })
});

// 4. Send document
await fetch(`/api/v1/merchant/signatures/documents/${document._id}/send`, {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    signingParties: [
      {
        name: 'John Smith',
        email: 'john@example.com',
        role: 'Employee',
        order: 1
      }
    ],
    configuration: {
      enforceSigningOrder: false,
      requireIdVerification: false,
      useDigitalSignature: true,
      expiration: {
        type: 'relative',
        relativeDays: 30
      }
    }
  })
});
```

## Error Codes

| Status Code | Description |
|------------|-------------|
| 200 | Success |
| 400 | Bad Request (validation error) |
| 401 | Unauthorized (invalid/missing token) |
| 403 | Forbidden (insufficient permissions) |
| 404 | Not Found |
| 500 | Internal Server Error |

## Rate Limits

- **Document Creation**: 100 per hour
- **Document Sending**: 500 per hour
- **API Calls**: 1000 per hour per organisation

## Related Documentation

- [Envelopes API](./envelopes.md)
- [Document Templates API](./document-templates.md)
- [Envelope Templates API](./envelope-templates.md)
- [API Overview](./README.md)

