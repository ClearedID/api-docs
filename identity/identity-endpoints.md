# Identity Verification API - Endpoints

← [Identity API Overview](./identity-api.md)

---

## Endpoints

### Dashboard & Summary

#### 1. Get Dashboard Summary

Get comprehensive dashboard statistics for identity and TRN verifications.

**Endpoint**: `GET /api/v1/merchant/identity/dashboard/summary`

**Success Response** (200):
```json
{
  "totalIdResults": [
    {
      "_id": "507f1f77bcf86cd799439011",
      "name": "John Smith",
      "status": "cleared",
      "documentType": "passport",
      "documentNumber": "A1234567",
      "createdAt": "2025-10-15T10:00:00Z"
    }
  ],
  "totalTrnResults": [
    {
      "_id": "507f1f77bcf86cd799439012",
      "name": "Jane Doe",
      "status": "cleared",
      "documentNumber": "123-456-789",
      "createdAt": "2025-10-15T11:00:00Z"
    }
  ],
  "pendingRequests": [
    {
      "_id": "507f1f77bcf86cd799439013",
      "name": "Bob Wilson",
      "types": "identity",
      "status": "pending",
      "createdAt": "2025-10-19T09:00:00Z"
    }
  ],
  "recentlyVerifiedIds": [...],
  "recentlyVerifiedTrns": [...],
  "processingIds": [...],
  "processingTrns": [...],
  "expiredIds": [...]
}
```

**Notes**:
- Includes counts and lists for all verification statuses
- Recently verified items from last 7 days
- Processing items include organisation information for operations users
- Non-operations users only see their organisation's data

---

#### 2. Get Category Summary

Get detailed summary for a specific category and document type.

**Endpoint**: `POST /api/v1/merchant/identity/dashboard/summary/:documentType/:category`

**URL Parameters**:
- `documentType` (string, required) - Type of document: `id` or `trn`
- `category` (string, required) - Category: `recentlyVerified`, `verified`, `processing`, `expired`

**Request Body**:
```json
{
  "fromDate": "2025-10-01",
  "toDate": "2025-10-31"
}
```

**Success Response** (200):
```json
{
  "recentlyVerifiedIds": [
    {
      "_id": "507f1f77bcf86cd799439011",
      "name": "John Smith",
      "status": "cleared",
      "documentType": "passport",
      "documentNumber": "A1234567",
      "createdAt": "2025-10-15T10:00:00Z",
      "clearedAt": "2025-10-16T12:00:00Z"
    }
  ]
}
```

**Error Response** (400):
```json
{
  "message": "Invalid category"
}
```

---

### Verification Requests

#### 3. List Verification Requests

Retrieve paginated list of identity verification requests.

**Endpoint**: `GET /api/v1/merchant/identity/requests`

**Query Parameters**:
- `query` (string, JSON encoded) - Search/filter/sort parameters:
  ```json
  {
    "keywords": "john smith",
    "page": 1,
    "sortField": "createdAt",
    "sortOrder": "desc",
    "filters": {
      "status": ["pending", "approved"],
      "types": ["identity"],
      "createdAt_start": "2025-10-01",
      "createdAt_end": "2025-10-31"
    }
  }
  ```

**Success Response** (200):
```json
{
  "records": [
    {
      "_id": "507f1f77bcf86cd799439013",
      "name": "John Smith",
      "types": "identity",
      "originator": "HR Department",
      "status": "pending",
      "createdAt": "2025-10-19T09:00:00Z"
    }
  ],
  "columns": [
    {
      "header": "Name",
      "path": "name",
      "tooltip": "This is the name that was entered when the request was being made."
    },
    {
      "header": "Type",
      "path": "types",
      "tooltip": "The type of verification request."
    },
    {
      "header": "Status",
      "path": "status",
      "type": "status",
      "align": "text-center"
    }
  ],
  "filters": [
    {
      "name": "status",
      "label": "Status",
      "type": "multi-select",
      "options": ["approved", "denied", "pending", "awaitingClearance"]
    },
    {
      "name": "types",
      "label": "Type",
      "type": "multi-select",
      "options": ["identity", "address", "income", "qualification", "reference", "company"]
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
- `pending` - Awaiting customer response
- `approved` - Customer approved sharing
- `denied` - Customer denied request
- `awaitingClearance` - Waiting for operations clearance

**Notes**:
- Requires `view_identity_verification_requests_list` privilege
- Non-operations users only see their organisation's requests
- Operations users see all requests

---

#### 4. Get Request by ID

Retrieve a specific verification request.

**Endpoint**: `GET /api/v1/merchant/identity/requests/:requestId/get`

**URL Parameters**:
- `requestId` (string, required) - Request unique identifier

**Success Response** (200):
```json
{
  "request": {
    "_id": "507f1f77bcf86cd799439013",
    "name": "John Smith",
    "types": "identity",
    "status": "pending",
    "createdAt": "2025-10-19T09:00:00Z",
    "userId": "507f1f77bcf86cd799439050"
  },
  "customerInfo": {
    "phoneNumber": "+1876xxxxxxx",
    "emailAddress": "john@example.com"
  }
}
```

**Error Responses**:

400 Bad Request:
```json
{
  "message": "Invalid request ID format"
}
```

404 Not Found:
```json
{
  "message": "Request not found"
}
```

**Notes**:
- Requires `view_identity_verification_requests_list` privilege
- Returns customer contact information
- Access controlled by organisation

---

#### 5. Get Request Details

Get detailed information about a verification request including linked results.

**Endpoint**: `GET /api/v1/merchant/identity/requests/:requestId/details`

**URL Parameters**:
- `requestId` (string, required) - Request ID

**Success Response** (200):
```json
{
  "_id": "507f1f77bcf86cd799439013",
  "name": "John Smith",
  "types": "identity",
  "status": "approved",
  "links": [
    {
      "url": "/identity/report/507f1f77bcf86cd799439011/details",
      "text": "Identity Verification - Passport - cleared"
    }
  ],
  "createdAt": "2025-10-19T09:00:00Z"
}
```

**Notes**:
- Requires `view_identity_verification_request_details` privilege
- Includes links to related verification results
- Used to navigate from request to actual verification report

---

### Identity Verification Results

#### 6. List Identity Verification Results

Retrieve paginated list of identity verification results.

**Endpoint**: `GET /api/v1/merchant/identity/results`

**Query Parameters**:
- `query` (string, JSON encoded) - Search/filter/sort parameters:
  ```json
  {
    "keywords": "john smith",
    "page": 1,
    "sortField": "createdAt",
    "sortOrder": "desc",
    "filters": {
      "status": ["cleared", "processing"],
      "documentType": ["passport", "driverLicence"],
      "createdAt_start": "2025-10-01",
      "createdAt_end": "2025-10-31"
    },
    "customerId": "507f1f77bcf86cd799439050",
    "temporaryUserCode": "temp-abc123",
    "userId": "507f1f77bcf86cd799439050",
    "addressResultId": "507f1f77bcf86cd799439060"
  }
  ```

**Success Response** (200):
```json
{
  "records": [
    {
      "_id": "507f1f77bcf86cd799439011",
      "name": "John Smith",
      "status": "cleared",
      "documentNumber": "A1234567",
      "uniqueIdNumber": "123-456-789",
      "documentType": "passport",
      "createdAt": "2025-10-15T10:00:00Z",
      "media": [
        {
          "label": "Thumbnail",
          "url": "https://s3.amazonaws.com/...",
          "type": "image"
        }
      ],
      "meta": {
        "accountNumber": "ACC-12345",
        "phoneNumber": "+1876xxxxxxx",
        "emailAddress": "john@example.com",
        "deviceInfo": {
          "platform": "iOS",
          "browser": "Safari",
          "device": "iPhone 14"
        },
        "channel": "mobile"
      }
    }
  ],
  "columns": [
    {
      "header": "Name",
      "path": "name",
      "type": "link",
      "urlPrefix": "/identity/report/{id}/details",
      "classification": "sensitive"
    },
    {
      "header": "Status",
      "path": "status",
      "type": "status",
      "align": "text-center"
    },
    {
      "header": "Document Type",
      "path": "documentType",
      "type": "documentType"
    }
  ],
  "filters": [
    {
      "name": "status",
      "label": "Status",
      "type": "multi-select",
      "options": ["cleared", "rejected", "flagged", "processing", "submitted", "new"]
    },
    {
      "name": "documentType",
      "label": "Document Type",
      "type": "multi-select",
      "options": ["voterId", "passport", "driverLicence", "nationalIdCard"]
    }
  ],
  "appliedFilters": {},
  "paging": {
    "recordCount": 156,
    "pageCount": 16,
    "currentPage": 1
  }
}
```

**Document Types**:
- `voterId` - Voter ID
- `passport` - Passport
- `driverLicence` - Driver Licence
- `nationalIdCard` - National ID Card

**Status Values**:
- `new` - Just created, not yet submitted
- `submitted` - Customer submitted documents
- `processing` - Under review by operations
- `flagged` - Flagged for manual review
- `cleared` - Verified and approved
- `rejected` - Verification rejected
- `deleted` - Deleted/archived
- `expired` - Verification expired

**Notes**:
- Requires `view_identity_verification_reports_list` privilege
- Operations users see additional fields (account number, contact info, device info)
- Results can be filtered by customer ID, user ID, or temporary user code
- Supports keyword search across name, document number, and email
- Deleted records show masked information

---

#### 7. Get Verification Result Details

Get complete details of an identity verification result.

**Endpoint**: `GET /api/v1/merchant/identity/report/:reportId`

**URL Parameters**:
- `reportId` (string, required) - Verification result ID

**Success Response** (200):
```json
{
  "_id": "507f1f77bcf86cd799439011",
  "name": "John Michael Smith",
  "status": "cleared",
  "documentType": "passport",
  "documentNumber": "A1234567",
  "uniqueIdNumber": "123-456-789",
  "clearedAt": "2025-10-16T12:00:00Z",
  "personalInfo": {
    "biography": {
      "firstName": "John",
      "middleName": "Michael",
      "lastName": "Smith",
      "dateOfBirth": "1990-05-15"
    },
    "address": {
      "line1": "123 Main Street",
      "line2": "Kingston 6",
      "town": "Kingston",
      "parish": "St. Andrew"
    },
    "contact": {
      "emailAddress": "john@example.com",
      "phoneNumber": "+1876xxxxxxx"
    }
  },
  "documentInfo": {
    "issuer": "Passport Immigration and Citizenship Agency",
    "countryOfIssue": "Jamaica",
    "idNumber": "A1234567",
    "controlNumber": null,
    "issueDate": "2021-07-29",
    "expirationDate": "2031-07-28",
    "firstIssueDate": "2011-07-30"
  },
  "media": [
    {
      "label": "Front of ID",
      "url": "https://s3.amazonaws.com/...",
      "type": "image"
    },
    {
      "label": "Selfie with date",
      "url": "https://s3.amazonaws.com/...",
      "type": "image"
    }
  ],
  "analysis": {
    "faceMatch": {
      "confidence": 98.5,
      "similarity": 96.2,
      "status": "match"
    },
    "liveness": {
      "confidence": 97.8,
      "status": "live"
    },
    "documentAuthenticity": {
      "confidence": 95.0,
      "status": "authentic"
    }
  },
  "decisionFields": [
    {
      "path": "personalInfo.biography.firstName",
      "label": "First Name"
    },
    {
      "path": "personalInfo.biography.lastName",
      "label": "Last Name"
    }
  ],
  "duplicateTRNs": [
    {
      "_id": "507f1f77bcf86cd799439070",
      "firstName": "John",
      "lastName": "Smith",
      "emailAddress": "otherjohn@example.com",
      "uniqueId": "123-456-789",
      "reports": [
        {
          "_id": "507f1f77bcf86cd799439071",
          "status": "cleared",
          "name": "John Smith"
        }
      ]
    }
  ],
  "duplicateIDs": [],
  "matches": [
    {
      "reportId": "507f1f77bcf86cd799439072",
      "reportStatus": "cleared",
      "documentNumber": "A1234567",
      "documentType": "passport",
      "similarity": 98.5,
      "confidence": 97.0,
      "name": "John Michael Smith",
      "selfieUrl": "https://s3.amazonaws.com/...",
      "idPhotoUrl": "https://s3.amazonaws.com/...",
      "found": true
    }
  ]
}
```

**Response Fields**:
- `personalInfo` - Personal details extracted from ID
- `documentInfo` - Document details (issue date, expiry, issuer)
- `media` - Array of document images and selfies with signed URLs
- `analysis` - AI analysis results (face match, liveness, authenticity)
- `duplicateTRNs` - Users with same TRN (potential duplicates)
- `duplicateIDs` - Users with same ID document number
- `matches` - Facial recognition matches from database

**Notes**:
- Requires `view_identity_verification_report_details` privilege
- Non-operations users see sanitised version (no internal analysis)
- Auto-updates status from `submitted`/`flagged` to `processing` when operations views
- Media URLs are signed and expire after configured time
- Duplicate detection helps identify fraud

---

#### 8. Get Public Verification Report

Get public verification report (for display to third parties).

**Endpoint**: `GET /api/v1/merchant/identity/report/public/:requestId`

**URL Parameters**:
- `requestId` (string, required) - Report ID or profile ID

**Success Response** (200):
```json
{
  "name": "John Smith",
  "status": "cleared",
  "clearedAt": "2025-10-16T12:00:00Z",
  "documentType": "passport",
  "documentInfo": {
    "idNumber": "A12****7"
  },
  "meta": {
    "requestExpired": false
  },
  "addresses": ["Kingston, St. Andrew"],
  "emailAddresses": ["j***@example.com"],
  "phoneNumbers": ["+1876***xxx"]
}
```

**Notes**:
- Requires `view_public_identity_verification_report_details` privilege (optional enforcement)
- Masks sensitive information (ID numbers, contact details)
- Respects profile badge visibility settings
- Can lookup by report ID or user profile ID
- Returns limited information for privacy

**Profile Badge Visibility Options**:
- **Name Visibility**: `full`, `firstLastInitial`, `initials`
- **Address Visibility**: `full`, `parish`, `none`
- **Email Visibility**: `full`, `masked`, `none`
- **Phone Visibility**: `full`, `masked`, `none`
- **ID Type Visibility**: Show or hide document type

---

#### 9. Update Verification Decision

Make a clearance or rejection decision on an identity verification.

**Endpoint**: `POST /api/v1/merchant/identity/report/decision`

**Request Body**:
```json
{
  "requestId": "507f1f77bcf86cd799439011",
  "decision": {
    "firstName": "John",
    "middleName": "Michael",
    "lastName": "Smith"
  },
  "status": "cleared",
  "withTrn": true,
  "override": false,
  "rejectionReason": ""
}
```

**Request Parameters**:
- `requestId` (string, required) - Verification result ID
- `decision` (object, required) - Decision details (corrected name fields)
- `status` (string, required) - Decision status: `cleared` or `rejected`
- `withTrn` (boolean, optional) - Include TRN verification
- `override` (boolean, optional) - Override duplicate ID check
- `rejectionReason` (string, optional) - Reason for rejection

**Success Response** (200):
```json
{
  "success": true
}
```

**Duplicate Detection Response** (200):
```json
{
  "success": true,
  "message": "A user has already been verified with this ID.",
  "duplicate": true
}
```

**Process Flow**:

**When Status = `cleared`:**
1. Updates personal info with corrected details
2. Marks verification as cleared with timestamp
3. Updates user's `idVerified` status
4. If driver's licence/national ID: Auto-creates TRN verification
5. If TRN exists: Links TRN to user and marks as verified
6. Publishes verification update to merchant webhook
7. Sends clearance email to customer
8. Sends webhook notification (if configured)
9. Updates verification status records

**When Status = `rejected`:**
1. Marks verification as rejected with timestamp
2. Uses AI to rewrite rejection reason professionally
3. Sends rejection email with reason to customer
4. Logs rejection in contact history
5. Updates verification status records

**Notes**:
- Requires operations role
- Checks for duplicate IDs (unless override = true)
- Automatically handles TRN verification for driver's licence/national ID
- Sends email notifications to customer
- Publishes updates to merchant systems via webhook
- Creates comprehensive audit trail

---

#### 10. Delete Verification Result

Soft delete an identity verification result.

**Endpoint**: `POST /api/v1/merchant/identity/report/:reportId/delete`

**URL Parameters**:
- `reportId` (string, required) - Report ID

**Success Response** (200):
```json
{}
```

**Error Response**:
```json
{
  "notification": {
    "text": "Deletion failed. Please try again.",
    "type": "error"
  }
}
```

**Notes**:
- Requires operations role
- Soft delete - marks as deleted but retains data
- Sets status to `deleted` with timestamp
- Used for removing test/duplicate/invalid submissions

---

#### 11. Merge Identity Reports

Merge multiple identity verification reports into one primary report.

**Endpoint**: `POST /api/v1/merchant/identity/merge-reports`

**Request Body**:
```json
{
  "reportId": "507f1f77bcf86cd799439011",
  "faceMatches": [
    "507f1f77bcf86cd799439072",
    "507f1f77bcf86cd799439073"
  ],
  "facesToDelete": [
    "507f1f77bcf86cd799439074"
  ]
}
```

**Request Parameters**:
- `reportId` (string, required) - Primary report ID (merge target)
- `faceMatches` (array, optional) - Array of report IDs to merge
- `faceToMerge` (string, optional) - Single report ID to merge
- `facesToDelete` (array, optional) - Array of report IDs to delete

**Success Response** (200):
```json
{
  "success": true,
  "message": "Successfully processed 5 faces (3 merged, 2 deleted)"
}
```

**Merge Process**:
1. Validates all reports exist
2. Merges media from matching documents
3. Copies missing personal info fields
4. Consolidates access lists
5. Marks merged reports as `duplicated`
6. Deletes reports in facesToDelete array
7. Handles user assignment for catch-all cases

**Notes**:
- Requires operations role
- Used to consolidate duplicate verifications
- Preserves all media and information
- Maintains audit trail
- Face matches only merge selfies and matching document types

---

### Face Authentication

#### 12. List Face Authentication Results

Retrieve paginated list of face authentication attempts.

**Endpoint**: `GET /api/v1/merchant/identity/authentication/results`

**Query Parameters**:
- `query` (string, JSON encoded) - Search/filter/sort parameters

**Success Response** (200):
```json
{
  "records": [
    {
      "_id": "507f1f77bcf86cd799439080",
      "name": "John Smith",
      "status": "cleared",
      "documentNumber": "A1234567",
      "documentType": "passport",
      "createdAt": "2025-10-19T11:00:00Z",
      "meta": {
        "accountNumber": "ACC-12345",
        "phoneNumber": "+1876xxxxxxx",
        "emailAddress": "john@example.com"
      }
    }
  ],
  "columns": [...],
  "filters": [
    {
      "name": "status",
      "label": "Status",
      "type": "multi-select",
      "options": ["cleared", "rejected", "in_progress", "waiting", "failed", "expired"]
    }
  ],
  "paging": {
    "recordCount": 45,
    "pageCount": 5,
    "currentPage": 1
  }
}
```

**Status Values**:
- `in_progress` - Authentication in progress
- `waiting` - Waiting for user action
- `cleared` - Successfully authenticated
- `rejected` - Authentication failed
- `failed` - System error
- `expired` - Authentication expired

---

#### 13. Submit Face Authentication Request

Submit a face authentication request with ID document image.

**Endpoint**: `POST /api/v1/merchant/identity/authentication/request`

**Content-Type**: `multipart/form-data`

**Form Parameters**:
- `file` (file, required) - ID document image
- `resultId` (string, required) - Face authentication result ID
- `requestId` (string, required) - Verification request ID

**Success Response** (201):
```json
{
  "message": "Face authentication request created successfully.",
  "fileUrl": "https://s3.amazonaws.com/..."
}
```

**Error Responses**:

400 Bad Request:
```json
{
  "error": "Sorry, we cannot accept that document. Please use another ID."
}
```

500 Internal Server Error:
```json
{
  "error": "Failed to extract text from ID image."
}
```

**Process Flow**:
1. Uploads ID image to S3
2. Extracts text using AWS Textract
3. Validates document is not a visa
4. Uses AI to extract structured ID details
5. Updates face authentication result with extracted data
6. Returns signed URL for uploaded image

**Extracted Details**:
- First name, middle name, last name
- ID type, ID number, control number
- Issue date, expiry date, first issue date
- Date of birth
- Address details
- Issuer and country of issue

**Notes**:
- Rejects visa documents automatically
- Uses AI (GPT-4) for intelligent data extraction
- Validates document authenticity
- Links to verification request and updates name

---

### TRN Verification

#### 14. List TRN Verification Results

Retrieve paginated list of TRN verification results.

**Endpoint**: `GET /api/v1/merchant/identity/trn/results`

**Query Parameters**:
- `query` (string, JSON encoded) - Search/filter/sort parameters

**Success Response** (200):
```json
{
  "records": [
    {
      "_id": "507f1f77bcf86cd799439012",
      "name": "John Smith",
      "status": "cleared",
      "documentNumber": "123-456-789",
      "documentType": "trnCard",
      "createdAt": "2025-10-15T11:00:00Z",
      "updatedAt": "2025-10-16T12:00:00Z",
      "meta": {
        "accountNumber": "ACC-12345",
        "emailAddress": "john@example.com",
        "phoneNumber": "+1876xxxxxxx"
      }
    }
  ],
  "columns": [
    {
      "header": "Name",
      "path": "name",
      "type": "link",
      "urlPrefix": "/identity/trn/report/{id}/details"
    },
    {
      "header": "Status",
      "path": "status",
      "type": "status",
      "align": "text-center"
    },
    {
      "header": "Document #",
      "path": "documentNumber"
    }
  ],
  "filters": [
    {
      "name": "status",
      "label": "Status",
      "type": "multi-select",
      "options": ["cleared", "rejected", "flagged", "processing", "submitted", "new"]
    },
    {
      "name": "documentType",
      "label": "Document Type",
      "type": "multi-select",
      "options": ["trnCard", "trnLetter", "driverLicence", "nationalIdCard"]
    }
  ],
  "paging": {
    "recordCount": 89,
    "pageCount": 9,
    "currentPage": 1
  }
}
```

**TRN Document Types**:
- `trnCard` - TRN Card
- `trnLetter` - TRN Letter
- `driverLicence` - Driver Licence (contains TRN)
- `nationalIdCard` - National ID Card (contains TRN)

**Notes**:
- Requires operations role
- TRN verifications can be standalone or extracted from driver's licence/national ID
- Operations users see contact information

---

#### 15. Get TRN Verification Report

Get detailed TRN verification result.

**Endpoint**: `GET /api/v1/merchant/identity/trn/report/:requestId`

**URL Parameters**:
- `requestId` (string, required) - TRN verification result ID

**Success Response** (200):
```json
{
  "_id": "507f1f77bcf86cd799439012",
  "name": "John Michael Smith",
  "status": "cleared",
  "documentType": "trnCard",
  "documentNumber": "123-456-789",
  "bio": {
    "firstName": "John",
    "middleName": "Michael",
    "lastName": "Smith"
  },
  "trnDetails": {
    "dateOfBirth": "1990-05-15",
    "gender": "M",
    "status": "Active",
    "homeAddress": {
      "streetName": "Main Street",
      "streetNo": "123",
      "parish": "St. Andrew"
    }
  },
  "media": [
    {
      "label": "Front of TRN Card",
      "url": "https://s3.amazonaws.com/...",
      "type": "image"
    }
  ],
  "htmlResponse": "...",
  "reviewNotes": "TRN verification processed from driverLicence.",
  "decisionFields": [
    {
      "path": "bio.firstName",
      "label": "First Name"
    },
    {
      "path": "documentNumber",
      "label": "TRN Number"
    }
  ],
  "clearedAt": "2025-10-16T12:00:00Z",
  "submittedAt": "2025-10-15T11:00:00Z"
}
```

**Notes**:
- Requires operations role
- Auto-updates status from `submitted` to `processing`
- Media URLs regenerated as signed URLs
- Duplicate media removed

---

#### 16. Validate TRN Against CBS

Validate a TRN number against CBS (Credit Bureau Services) database.

**Endpoint**: `POST /api/v1/merchant/identity/trn/validate`

**Request Body**:
```json
{
  "trn": "123-456-789",
  "reportId": "507f1f77bcf86cd799439012"
}
```

**OR**:
```json
{
  "trn": "id-check",
  "reportId": "507f1f77bcf86cd799439011"
}
```

**Request Parameters**:
- `trn` (string, required) - TRN number to validate, or `"id-check"` to auto-detect from linked ID
- `reportId` (string, required) - Report ID (TRN or Identity verification)

**Success Response** (200):
```json
{
  "success": true,
  "trnReport": {
    "id": "507f1f77bcf86cd799439012",
    "result": {
      "trnNumber": "123-456-789",
      "firstName": "John",
      "middleName": "Michael",
      "surname": "Smith",
      "dateOfBirth": "1990-05-15",
      "gender": "M",
      "status": "Active",
      "homeAddress": {
        "streetName": "Main Street",
        "streetNo": "123",
        "parish": "St. Andrew",
        "parishCode": "01"
      },
      "lastUpdate": "2025-10-15"
    },
    "status": "completed"
  },
  "reportedDateOfBirth": "1990-05-15"
}
```

**Notes**:
- Queries CBS database for TRN information
- Compares CBS data with ID verification data
- Returns date of birth from ID for cross-validation
- If `trn = "id-check"`, automatically finds linked TRN for ID verification

---

#### 17. Update TRN Verification Decision

Make a clearance or rejection decision on a TRN verification.

**Endpoint**: `POST /api/v1/merchant/identity/trn/report/decision`

**Request Body**:
```json
{
  "requestId": "507f1f77bcf86cd799439012",
  "decision": {
    "firstName": "John",
    "middleName": "Michael",
    "lastName": "Smith",
    "documentNumber": "123-456-789"
  },
  "status": "cleared"
}
```

**Request Parameters**:
- `requestId` (string, required) - TRN verification result ID
- `decision` (object, required) - Decision details
- `status` (string, required) - `cleared` or `rejected`

**Success Response** (200):
```json
{}
```

**Duplicate Detection Response** (200):
```json
{
  "success": true,
  "message": "A user has already been verified with this ID.",
  "duplicate": true
}
```

**Process Flow**:

**When Status = `cleared`:**
1. Updates TRN verification status and details
2. Sets user's `uniqueId` (TRN) and marks as verified
3. Closes TRN verification onboarding action
4. Updates all identity verifications for user with TRN
5. Sends clearance email to customer
6. Processes pending verification requests that required TRN

**When Status = `rejected`:**
1. Marks TRN as rejected
2. Sends rejection email
3. Logs retry request in contact history

**Notes**:
- Requires operations role
- Checks for duplicate TRN usage
- Auto-links TRN to all identity verifications for the user
- Email notifications sent to customer

---

#### 18. Delete TRN Verification Result

Soft delete a TRN verification result.

**Endpoint**: `POST /api/v1/merchant/identity/trn/report/:reportId/delete`

**URL Parameters**:
- `reportId` (string, required) - TRN report ID

**Success Response** (200):
```json
{}
```

**Notes**:
- Requires operations role
- Soft delete with timestamp

---

### Meetings & Video Verification

#### 19. List Video Meetings

Get list of scheduled video verification meetings.

**Endpoint**: `GET /api/v1/merchant/identity/meetings`

**Query Parameters**:
- `page` (number, optional) - Page number (default: 1)

**Success Response** (200):
```json
{
  "records": [
    {
      "_id": "507f1f77bcf86cd799439090",
      "meetingId": "MTG-abc123",
      "userId": "507f1f77bcf86cd799439050",
      "customerName": "John Smith",
      "title": "Video Call [MTG-abc123] - Identity Verification",
      "status": "scheduled",
      "startTime": "2025-10-20T14:00:00Z",
      "externalMeetingIds": {
        "dailyCoMeetingId": "daily-room-xyz",
        "dailyCoMeetingUrl": "https://daily.co/room-xyz"
      }
    }
  ],
  "paging": {
    "recordCount": 12,
    "pageCount": 2,
    "currentPage": 1
  }
}
```

**Meeting Status Values**:
- `scheduled` - Meeting scheduled, awaiting participants
- `meeting` - Meeting in progress
- `closed` - Meeting completed
- `cancelled` - Meeting cancelled

**Notes**:
- Requires operations role
- Only shows meetings with status `meeting`
- Uses Daily.co for video conferencing

---

#### 20. Request Verification Meeting

Request a video verification meeting for a customer.

**Endpoint**: `POST /api/v1/merchant/identity/requests/:requestId/request-meeting`

**URL Parameters**:
- `requestId` (string, required) - Identity verification result ID

**Request Body**:
```json
{
  "externalMeetingId": "videosdk-meeting-xyz"
}
```

**Success Response** (200):
```json
{
  "notification": {
    "type": "info",
    "text": "Meeting request was sent."
  }
}
```

**Error Responses**:

400 Bad Request:
```json
{
  "notification": {
    "text": "A meeting was already scheduled for this customer.",
    "type": "error"
  }
}
```

404 Not Found:
```json
{
  "message": "Request not found."
}
```

**Process Flow**:
1. Validates user and request exist
2. Checks for existing scheduled meetings
3. Cancels any existing scheduled meetings
4. Creates Daily.co video room (expires in 24 hours)
5. Creates meeting record in database
6. Creates action item for customer
7. Sends email invitation to customer

**Notes**:
- Requires operations role
- Only one active meeting per customer
- Meeting link expires after 24 hours
- Customer receives email with meeting link

---

#### 21. Get Meeting Details

Get meeting details for an identity verification request.

**Endpoint**: `GET /api/v1/merchant/identity/meetings/:requestId`

**URL Parameters**:
- `requestId` (string, required) - Identity verification result ID

**Success Response** (200):
```json
{
  "meetingId": "MTG-abc123"
}
```

**Empty Response** (200):
```json
{}
```

**Notes**:
- Returns meeting ID if active meeting exists
- Returns empty object if no active meeting
- Active = status `scheduled` or `meeting`

---

#### 22. Get Meeting by Meeting ID

Get meeting details by meeting ID.

**Endpoint**: `GET /api/v1/merchant/identity/meetings/:meetingId/meeting-id`

**URL Parameters**:
- `meetingId` (string, required) - Meeting ID

**Success Response** (200):
```json
{
  "externalMeetingIds": {
    "videoSdkMeetingId": "videosdk-xyz",
    "dailyCoMeetingId": "daily-room-xyz",
    "dailyCoMeetingUrl": "https://daily.co/room-xyz"
  },
  "customerName": "John Smith"
}
```

**Empty Response** (200):
```json
{}
```

---

#### 23. Join Meeting (Agent)

Mark that an agent has joined the meeting.

**Endpoint**: `POST /api/v1/merchant/identity/meetings/:meetingId/join`

**URL Parameters**:
- `meetingId` (string, required) - Meeting ID

**Success Response** (200):
```json
{
  "notification": {
    "type": "info",
    "text": "Meeting was joined by agent."
  }
}
```

**Notes**:
- Requires operations role
- Sets `hasAgent` and `agentAttended` flags on meeting
- Used to track agent participation

---

#### 24. Close Meeting

Close an active meeting.

**Endpoint**: `POST /api/v1/merchant/identity/meetings/:meetingId/close`

**URL Parameters**:
- `meetingId` (string, required) - Meeting ID

**Success Response** (200):
```json
{
  "notification": {
    "type": "info",
    "text": "Meeting was closed."
  }
}
```

**Process**:
1. Updates meeting status to `closed`
2. Sets `hasAgent` to false
3. Closes related action items

**Notes**:
- Requires operations role
- Automatically closes related action items

---

### ID Generator (Testing/QA)

#### 25. Get ID Templates

Get list of available ID templates for testing.

**Endpoint**: `GET /api/v1/merchant/identity/id-generator/templates`

**Success Response** (200):
```json
{
  "templates": [
    {
      "id": "driverLicence",
      "name": "Driver Licence",
      "description": "Standard Jamaican driver's licence template",
      "fileKeys": {
        "front": "specimen-ids/driverLicenceFront.png",
        "back": "specimen-ids/driverLicenceBack.png"
      }
    },
    {
      "id": "passport",
      "name": "Passport",
      "description": "Jamaican passport template",
      "fileKeys": {
        "front": "specimen-ids/passportFront.png",
        "back": "specimen-ids/passportBack.png"
      }
    },
    {
      "id": "voterId",
      "name": "Voter ID",
      "description": "Jamaican voter registration card template",
      "fileKeys": {
        "front": "specimen-ids/voterIdFront.png",
        "back": "specimen-ids/voterIdBack.png"
      }
    },
    {
      "id": "nationalIdentificationCard",
      "name": "National Identification Card",
      "description": "Jamaican national ID card template",
      "fileKeys": {
        "front": "specimen-ids/nationalIdentificationCardFront.png",
        "back": "specimen-ids/nationalIdentificationCardBack.png"
      }
    },
    {
      "id": "trn",
      "name": "TRN",
      "description": "Jamaican TRN card template",
      "fileKeys": {
        "front": "specimen-ids/trnFront.png",
        "back": null
      }
    }
  ]
}
```

---

#### 26. Get ID Template

Get a specific ID template with images and photo placement configuration.

**Endpoint**: `GET /api/v1/merchant/identity/id-generator/template/:idType`

**URL Parameters**:
- `idType` (string, required) - ID type: `driverLicence`, `passport`, `voterId`, `nationalIdentificationCard`, `trn`

**Success Response** (200):
```json
{
  "frontImageUrl": "https://s3.amazonaws.com/.../driverLicenceFront.png?signature=...",
  "backImageUrl": "https://s3.amazonaws.com/.../driverLicenceBack.png?signature=...",
  "idType": "driverLicence",
  "photoPlacement": {
    "x": 800,
    "y": 200,
    "width": 120,
    "height": 150
  }
}
```

**Notes**:
- Returns signed URLs for template images
- Photo placement coordinates for overlaying user photo
- URLs expire after 8 hours
- Used in ID generator testing tool

---

#### 27. Generate ID Text Fields

Generate realistic text field data for ID documents using AI.

**Endpoint**: `POST /api/v1/merchant/identity/id-generator/generate-text`

**Request Body**:
```json
{
  "idType": "driverLicence",
  "name": "John Smith"
}
```

**Request Parameters**:
- `idType` (string, required) - Type of ID to generate
- `name` (string, optional) - Specific name to use (otherwise random)

**Success Response** (200):
```json
{
  "frontTextFields": [
    {
      "id": "trn",
      "text": "456-789-123",
      "x": 320,
      "y": 185,
      "fontSize": 16,
      "color": "#333333"
    },
    {
      "id": "lastName",
      "text": "SMITH",
      "x": 30,
      "y": 425,
      "fontSize": 16,
      "color": "#222222"
    },
    {
      "id": "firstAndMiddleNames",
      "text": "JOHN MICHAEL",
      "x": 30,
      "y": 460,
      "fontSize": 16,
      "color": "#222222"
    }
  ],
  "backTextFields": [
    {
      "id": "controlNumber",
      "text": "1410098515",
      "x": 32,
      "y": 530,
      "fontSize": 16,
      "color": "#333333"
    },
    {
      "id": "barcode",
      "type": "barcode",
      "text": "4567891231410098515",
      "x": 24,
      "y": 610,
      "width": 1000,
      "height": 85
    }
  ]
}
```

**Generated Data Includes**:
- Names (highly randomised)
- ID numbers (realistic format)
- Dates (issue, expiry, birth)
- Addresses (Jamaican format)
- Barcodes and QR codes
- MRZ (Machine Readable Zone) for passports/IDs

**Notes**:
- Uses AI (GPT-4) to generate realistic data
- Highly randomised to avoid patterns
- Follows Jamaican ID document formats
- Includes positioning coordinates for rendering
- Prevents consecutive number sequences (e.g., 123, 456)

---

#### 28. Refresh ID Text Fields

Regenerate text fields for an ID template.

**Endpoint**: `POST /api/v1/merchant/identity/id-generator/refresh-text`

**Request Body**:
```json
{
  "idType": "passport"
}
```

**Success Response** (200):
```json
{
  "frontTextFields": [...],
  "backTextFields": [...]
}
```

**Notes**:
- Same as generate-text but for refreshing existing data
- Generates completely new random data

---

### Ticket Retrieval (Traffic Tickets)

#### 29. Find Tickets by Driver Licence

Retrieve traffic tickets for a driver's licence number.

**Endpoint**: `POST /api/v1/merchant/identity/ticket-retrieval/find-by-licence`

**Request Body**:
```json
{
  "driverLicenceNumber": "123456789",
  "identityVerificationResultId": "507f1f77bcf86cd799439011"
}
```

**Request Parameters**:
- `driverLicenceNumber` (string, optional) - Driver's licence number
- `identityVerificationResultId` (string, optional) - Identity verification result ID
- **Note**: Provide either licence number or verification result ID

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "userId": "507f1f77bcf86cd799439050",
    "tickets": [
      {
        "ticketNumber": "TKT-123456",
        "status": "Outstanding",
        "issueDate": "2025-09-15T00:00:00Z",
        "dueDate": "2025-10-15T00:00:00Z",
        "fineAmount": 5000,
        "demeritPoints": 3,
        "description": "Speeding - Exceeded limit by 20 km/h",
        "assignedCourt": "Kingston Traffic Court"
      },
      {
        "ticketNumber": "TKT-789456",
        "status": "Paid",
        "issueDate": "2025-08-01T00:00:00Z",
        "fineAmount": 3000,
        "description": "Parking violation"
      }
    ],
    "totalOutstandingBalance": 5000,
    "identityInfo": {
      "name": "John Smith",
      "driverLicenceNumber": "123456789"
    }
  }
}
```

**Error Responses**:

403 Forbidden:
```json
{
  "success": false,
  "message": "You are not authorised to access this resource"
}
```

404 Not Found:
```json
{
  "success": false,
  "message": "No identity verification result found"
}
```

**Notes**:
- Requires `traffic-tickets-onboarding-team` role
- Queries JCF (Jamaica Constabulary Force) system
- Returns tickets with various statuses
- Calculates total outstanding balance
- Grants consent to view tickets automatically

---

#### 30. Send Ticket Summary Email

Send traffic ticket summary to customer via email.

**Endpoint**: `POST /api/v1/merchant/identity/ticket-retrieval/send-email`

**Request Body**:
```json
{
  "driverLicenceNumber": "123456789",
  "totalBalance": 5000,
  "identityVerificationResultId": "507f1f77bcf86cd799439011"
}
```

**Success Response** (200):
```json
{
  "success": true,
  "message": "Email sent successfully"
}
```

**Email Contains**:
- Summary information (licence number, total outstanding)
- Individual ticket cards with details
- Formatted mobile-friendly HTML
- All ticket statuses (outstanding, paid, disposed)
- Important notices and disclaimers

**Notes**:
- Requires `traffic-tickets-onboarding-team` role
- Sends to user's registered email address
- Beautiful mobile-responsive HTML email
- Includes all ticket details with formatting

---

#### 31. Send Ticket Summary SMS

Send traffic ticket summary to customer via SMS with link.

**Endpoint**: `POST /api/v1/merchant/identity/ticket-retrieval/send-sms`

**Request Body**:
```json
{
  "driverLicenceNumber": "123456789",
  "phoneNumber": "+1876xxxxxxx",
  "totalBalance": 5000,
  "identityVerificationResultId": "507f1f77bcf86cd799439011"
}
```

**Success Response** (200):
```json
{
  "success": true,
  "message": "SMS sent successfully",
  "token": "abc123def456"
}
```

**Process**:
1. Finds identity verification by licence number or result ID
2. Updates user's phone number
3. Retrieves traffic tickets
4. Generates unique token (expires in 24 hours)
5. Stores token in user's record
6. Creates ticket view URL
7. Sends SMS with link

**SMS Message**:
```
View your traffic ticket printout: https://cleared.id/tickets/view/abc123def456
```

**Notes**:
- Requires `traffic-tickets-onboarding-team` role
- Updates user's phone number
- Token expires after 24 hours
- Customer can view tickets via link without login

---

### Account Recovery

#### 32. Resolve Recovery Case

Resolve account recovery cases (lost account or new device).

**Endpoint**: `POST /api/v1/merchant/identity/resolve-recovery`

**Request Body**:
```json
{
  "identityVerificationId": "507f1f77bcf86cd799439011",
  "resolutionType": "same_user",
  "recoverySessionId": "507f1f77bcf86cd799439100"
}
```

**Request Parameters**:
- `identityVerificationId` (string, required) - Identity verification result ID
- `resolutionType` (string, required) - Type of resolution:
  - `same_user` - Same person, different contact info
  - `different_user` - Different person
  - `new_device_verified` - New device access approved
  - `new_device_rejected` - New device access denied
  - `different_persons` - Different persons detected (for new device)
- `recoverySessionId` (string, required) - Recovery session ID

**Success Response - Same User** (200):
```json
{
  "success": true,
  "message": "Recovery case resolved successfully for same person. Pending user deleted, contact info restored to original user.",
  "recoverySessionId": "507f1f77bcf86cd799439100",
  "identityVerificationId": "507f1f77bcf86cd799439011",
  "resolutionType": "same_user",
  "idvResultAction": "deleted",
  "reason": "Same person confirmed - pending user deleted, contact info restored",
  "pendingUserDeleted": true,
  "contactInfoTransferred": true,
  "originalUserUpdated": true
}
```

**Success Response - Different User** (200):
```json
{
  "success": true,
  "message": "Recovery case resolved successfully for different person. New user account activated and contact info transferred.",
  "recoverySessionId": "507f1f77bcf86cd799439100",
  "identityVerificationId": "507f1f77bcf86cd799439011",
  "resolutionType": "different_user",
  "idvResultAction": "cleared",
  "reason": "Different person confirmed - new user account created",
  "newUserAccount": {
    "userId": "507f1f77bcf86cd799439110",
    "firstName": "Jane",
    "lastName": "Doe",
    "status": "active",
    "contactType": "phone",
    "newContactInfo": "+1876xxxxxxx"
  }
}
```

**Success Response - New Device Verified** (200):
```json
{
  "success": true,
  "message": "New device access verified successfully. User can now access their account from this device.",
  "recoverySessionId": "507f1f77bcf86cd799439100",
  "identityVerificationId": "507f1f77bcf86cd799439011",
  "resolutionType": "new_device_verified",
  "idvResultAction": "cleared",
  "reason": "New device access verified by operations team"
}
```

**Recovery Scenarios**:

**1. Same User (Lost Account)**:
- Same person with new contact info
- Deletes pending user account
- Restores contact info to original user
- Moves old contact to replaced arrays
- Copies device ID to original user
- Marks IDV as deleted/closed

**2. Different User (Lost Account)**:
- Different person using same contact info
- Activates new user account
- Removes contact from original user
- Adds old contact to replaced arrays
- Links IDV to new user
- Marks IDV as cleared

**3. New Device Verified**:
- Same person on new device
- Adds device ID to user's device list
- Grants access to account
- Marks IDV as cleared/closed

**4. New Device Rejected**:
- Access denied for new device
- Marks IDV as rejected
- User needs additional verification

**5. Different Persons (New Device)**:
- Different person detected
- Transfers phone/email appropriately
- Activates new user account
- Removes contact from original user

**Error Responses**:

400 Bad Request:
```json
{
  "success": false,
  "error": "Invalid resolution type. Must be \"same_user\", \"different_user\", \"new_device_verified\", \"new_device_rejected\", or \"different_persons\"."
}
```

403 Forbidden:
```json
{
  "success": false,
  "error": "Access denied. Only operations team members can resolve recovery cases."
}
```

404 Not Found:
```json
{
  "success": false,
  "error": "Recovery session not found."
}
```

**Notes**:
- Requires operations role
- Complex workflow handling contact info transfer
- Manages device ID preservation
- Updates recovery session with resolution details
- Creates comprehensive audit trail
- Sends email notifications

---

#### 33. Get Recovery Session by ID

Get recovery session details by MongoDB ID.

**Endpoint**: `GET /api/v1/merchant/identity/report/recovery-session-by-id/:recoverySessionId`

**URL Parameters**:
- `recoverySessionId` (string, required) - Recovery session MongoDB ID

**Success Response** (200):
```json
{
  "success": true,
  "recoveryId": "REC-abc123",
  "recoverySessionId": "507f1f77bcf86cd799439100",
  "status": "pending"
}
```

**Notes**:
- Returns human-readable recovery ID
- Used to lookup recovery sessions

---

#### 34. Get Original User for Recovery

Get original user's identity verification for recovery comparison.

**Endpoint**: `GET /api/v1/merchant/identity/report/original-user/:userId`

**URL Parameters**:
- `userId` (string, required) - Original user ID

**Query Parameters**:
- `newCaseId` (string, optional) - New case ID to exclude from search

**Success Response** (200):
```json
{
  "success": true,
  "user": {
    "_id": "507f1f77bcf86cd799439050",
    "firstName": "John",
    "lastName": "Smith",
    "status": "active",
    "createdAt": "2024-01-15T10:00:00Z",
    "uniqueId": "123-456-789",
    "country": "Jamaica"
  },
  "identityVerification": {
    "_id": "507f1f77bcf86cd799439011",
    "status": "cleared",
    "documentType": "passport",
    "documentNumber": "A1234567",
    "personalInfo": {...},
    "media": [...]
  }
}
```

**Notes**:
- Requires operations role
- Finds most recent identity verification for user
- Prefers verifications with status `cleared`, `submitted`, or `processing`
- Returns media with signed URLs
- Used for side-by-side comparison during recovery resolution

---

### Reference Verification

#### 35. Verify Reference Number

Verify a reference number against identity verification records.

**Endpoint**: `POST /api/v1/merchant/identity/verify-reference`

**Request Body**:
```json
{
  "referenceNumber": "REF-ABC123"
}
```

**Request Parameters**:
- `referenceNumber` (string, required) - Reference number to verify

**Success Response - Verified** (200):
```json
{
  "success": true,
  "status": "verified",
  "message": "Reference number verified successfully",
  "data": {
    "id": "507f1f77bcf86cd799439011",
    "status": "cleared",
    "createdAt": "2025-10-15T10:00:00Z",
    "updatedAt": "2025-10-16T12:00:00Z"
  }
}
```

**Error Response - Not Found** (400):
```json
{
  "success": false,
  "status": "not_found",
  "message": "Reference number not found in verification records"
}
```

**Notes**:
- Searches across organisation's access list
- Reference number must match organisation ID and reference number
- Used to verify identity shared via reference number
- Returns basic verification info without sensitive details

---

