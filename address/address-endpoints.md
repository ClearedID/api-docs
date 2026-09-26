# Address Verification API - Endpoints

← [Address API Overview](./address-api.md)

---

## Endpoints

### Dashboard & Summary

#### 1. Get Dashboard Summary

Get comprehensive dashboard statistics for address verifications.

**Endpoint**: `GET /api/v1/merchant/address/dashboard/summary`

**Success Response** (200):
```json
{
  "totalAddressResults": [
    {
      "_id": "507f1f77bcf86cd799439020",
      "name": "John Smith",
      "status": "cleared",
      "documentType": "utilityBill",
      "documentAddress": {
        "addressLine1": "123 Main Street",
        "addressLine2": "Kingston 6",
        "town": "Kingston",
        "parish": "St. Andrew"
      },
      "createdAt": "2025-10-15T10:00:00Z"
    }
  ],
  "pendingRequests": [
    {
      "_id": "507f1f77bcf86cd799439021",
      "name": "Jane Doe",
      "types": "address",
      "status": "pending",
      "originator": "HR Department",
      "createdAt": "2025-10-19T09:00:00Z"
    }
  ],
  "recentlyVerifiedAddresses": [
    {
      "_id": "507f1f77bcf86cd799439022",
      "name": "Bob Wilson",
      "status": "cleared",
      "documentAddress": {...},
      "sharedAt": "2025-10-18T14:00:00Z"
    }
  ],
  "processingAddresses": [
    {
      "_id": "507f1f77bcf86cd799439023",
      "name": "Alice Brown",
      "status": "processing",
      "createdAt": "2025-10-19T11:00:00Z"
    }
  ],
  "expiredAddresses": [
    {
      "_id": "507f1f77bcf86cd799439024",
      "name": "Charlie Davis",
      "status": "expired",
      "createdAt": "2025-09-15T10:00:00Z"
    }
  ]
}
```

**Categories**:
- `totalAddressResults` - All cleared address verifications
- `pendingRequests` - Verification requests awaiting customer response
- `recentlyVerifiedAddresses` - Addresses verified in last 7 days
- `processingAddresses` - Currently under review
- `expiredAddresses` - Expired verifications

**Notes**:
- Non-operations users see only their organisation's data
- Operations users see all verifications across all organisations
- Recently verified includes items from last 7 days

---

#### 2. Get Category Summary

Get detailed summary for a specific category with date filtering.

**Endpoint**: `POST /api/v1/merchant/address/dashboard/summary/:category`

**URL Parameters**:
- `category` (string, required) - Category: `recentlyVerified`, `verified`, `processing`, `expired`

**Request Body**:
```json
{
  "fromDate": "2025-10-01",
  "toDate": "2025-10-31"
}
```

**Request Parameters**:
- `fromDate` (string, optional) - ISO date string (filter start date)
- `toDate` (string, optional) - ISO date string (filter end date)

**Success Response** (200):
```json
{
  "recentlyVerifiedAddresses": [
    {
      "_id": "507f1f77bcf86cd799439020",
      "name": "John Smith",
      "status": "cleared",
      "documentType": "utilityBill",
      "documentAddress": {
        "addressLine1": "123 Main Street",
        "addressLine2": "Kingston 6",
        "town": "Kingston",
        "parish": "St. Andrew"
      },
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

**Valid Categories**:
- `recentlyVerified` - Recently verified addresses (status: cleared)
- `verified` - All verified addresses (status: cleared)
- `processing` - Under review (status: processing)
- `expired` - Expired verifications (status: expired)

---

### Verification Requests

#### 3. List Address Verification Requests

Retrieve paginated list of address verification requests.

**Endpoint**: `GET /api/v1/merchant/address/requests`

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
      "_id": "507f1f77bcf86cd799439021",
      "name": "John Smith",
      "types": "address",
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
      "header": "Originator",
      "path": "originator"
    },
    {
      "header": "Date",
      "path": "createdAt",
      "type": "date",
      "tooltip": "The date the request was sent to the customer."
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
      "name": "createdAt",
      "label": "Date",
      "type": "date-range"
    }
  ],
  "appliedFilters": {},
  "paging": {
    "recordCount": 34,
    "pageCount": 4,
    "currentPage": 1
  }
}
```

**Request Status Values**:
- `pending` - Awaiting customer response
- `approved` - Customer approved sharing
- `denied` - Customer denied request
- `awaitingClearance` - Waiting for operations clearance

**Notes**:
- Requires `view_address_verification_requests_list` privilege
- Non-operations users only see their organisation's requests
- Operations users see internal fields (organisationId, userId, clientId)
- Supports keyword search on name and originator

---

#### 4. Get Request by ID

Retrieve a specific address verification request.

**Endpoint**: `GET /api/v1/merchant/address/requests/:requestId/get`

**URL Parameters**:
- `requestId` (string, required) - Request unique identifier

**Success Response** (200):
```json
{
  "request": {
    "_id": "507f1f77bcf86cd799439021",
    "name": "John Smith",
    "types": "address",
    "status": "pending",
    "createdAt": "2025-10-19T09:00:00Z"
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
- Requires `view_address_verification_requests_list` privilege
- Returns customer contact information
- Access controlled by organisation
- Non-operations users see sanitised data

---

#### 5. Get Request Details

Get detailed information about an address verification request including linked results.

**Endpoint**: `GET /api/v1/merchant/address/requests/:requestId/details`

**URL Parameters**:
- `requestId` (string, required) - Request ID

**Success Response** (200):
```json
{
  "_id": "507f1f77bcf86cd799439021",
  "name": "John Smith",
  "types": "address",
  "status": "approved",
  "originator": "HR Department",
  "links": [
    {
      "url": "/address/report/507f1f77bcf86cd799439020/details",
      "text": "Address Verification - cleared"
    }
  ],
  "createdAt": "2025-10-19T09:00:00Z"
}
```

**Notes**:
- Requires `view_address_verification_request_details` privilege
- Includes links to related verification results
- Used to navigate from request to actual verification report

---

### Address Verification Results

#### 6. List Address Verification Results

Retrieve paginated list of address verification results for individuals.

**Endpoint**: `GET /api/v1/merchant/address/results`

**Query Parameters**:
- `query` (string, JSON encoded) - Search/filter/sort parameters:
  ```json
  {
    "keywords": "kingston",
    "page": 1,
    "sortField": "createdAt",
    "sortOrder": "desc",
    "filters": {
      "status": ["cleared", "processing"],
      "createdAt_start": "2025-10-01",
      "createdAt_end": "2025-10-31"
    },
    "customerId": "507f1f77bcf86cd799439050",
    "temporaryUserCode": "temp-abc123",
    "userId": "507f1f77bcf86cd799439050",
    "includeMedia": true
  }
  ```

**Success Response** (200):
```json
{
  "records": [
    {
      "_id": "507f1f77bcf86cd799439020",
      "name": "John Smith",
      "status": "cleared",
      "documentType": "utilityBill",
      "documentNumber": "BILL-123456",
      "documentAddress": {
        "addressLine1": "123 Main Street",
        "addressLine2": "Kingston 6",
        "town": "Kingston",
        "parish": "St. Andrew"
      },
      "gpsAddress": "18.0179° N, 76.8099° W",
      "manualAddress": "123 Main Street, Kingston 6, Kingston, St. Andrew",
      "createdAt": "2025-10-15T10:00:00Z",
      "updatedAt": "2025-10-16T12:00:00Z",
      "meta": {
        "accountNumber": "ACC-12345",
        "phoneNumber": "+1876xxxxxxx",
        "emailAddress": "john@example.com",
        "platform": "iOS",
        "fullAddress": "123 Main Street, Kingston 6, Kingston"
      },
      "media": [
        {
          "label": "Utility Bill",
          "url": "https://s3.amazonaws.com/...",
          "type": "document"
        }
      ]
    }
  ],
  "columns": [
    {
      "header": "Name",
      "path": "name",
      "type": "link",
      "urlPrefix": "/address/report/{id}/details"
    },
    {
      "header": "Status",
      "path": "status",
      "type": "status",
      "align": "text-center"
    },
    {
      "header": "Address",
      "path": "meta.fullAddress"
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
      "name": "createdAt",
      "label": "Created At",
      "type": "date-range"
    }
  ],
  "appliedFilters": {},
  "paging": {
    "recordCount": 89,
    "pageCount": 9,
    "currentPage": 1
  }
}
```

**Status Values**:
- `new` - Just created, not yet submitted
- `submitted` - Customer submitted documents
- `processing` - Under review by operations
- `flagged` - Flagged for manual review
- `cleared` - Verified and approved
- `rejected` - Verification rejected
- `deleted` - Deleted/archived

**Search Features**:
- Keyword search across name, address fields, GPS, manual address
- Email address search (finds user then filters results)
- Filter by customer ID, user ID, or temporary user code
- Optional media inclusion (with signed URLs)

**Notes**:
- Requires `view_address_verification_reports_list` privilege
- Operations users see additional fields (account number, contact info, platform)
- Default filter: `["processing", "submitted", "flagged", "cleared"]`
- Deleted records show masked information
- Names auto-resolved and cached

---

#### 7. Get Address Verification Report

Get complete details of an address verification result.

**Endpoint**: `GET /api/v1/merchant/address/report/:requestId`

**URL Parameters**:
- `requestId` (string, required) - Verification result ID

**Success Response** (200):
```json
{
  "_id": "507f1f77bcf86cd799439020",
  "name": "John Michael Smith",
  "status": "cleared",
  "documentType": "utilityBill",
  "documentNumber": "BILL-123456",
  "clearedAt": "2025-10-16T12:00:00Z",
  "personalInfo": {
    "biography": {
      "firstName": "John",
      "middleName": "Michael",
      "lastName": "Smith",
      "dateOfBirth": "1990-05-15"
    }
  },
  "documentAddress": {
    "addressLine1": "123 Main Street",
    "addressLine2": "Kingston 6",
    "town": "Kingston",
    "parish": "St. Andrew"
  },
  "verifiedAddress": {
    "addressLine1": "123 Main Street",
    "addressLine2": "Kingston 6",
    "town": "Kingston",
    "parish": "St. Andrew",
    "coordinates": {
      "latitude": 18.0179,
      "longitude": -76.8099
    }
  },
  "gpsAddress": "18.0179° N, 76.8099° W",
  "manualAddress": "123 Main Street, Kingston 6, Kingston, St. Andrew",
  "documentInfo": {
    "institution": "Jamaica Public Service Company",
    "documentNumber": "BILL-123456",
    "issueDate": "2025-09-15",
    "documentAge": 31,
    "country": "Jamaica"
  },
  "reason": "Proof of residence for employment verification",
  "media": [
    {
      "label": "Utility Bill - Front",
      "url": "https://s3.amazonaws.com/...",
      "type": "document"
    },
    {
      "label": "Photo of Property",
      "url": "https://s3.amazonaws.com/...",
      "type": "image"
    }
  ],
  "decisionFields": [
    {
      "path": "personalInfo.biography.firstName",
      "label": "First Name"
    },
    {
      "path": "documentAddress.addressLine1",
      "label": "Document Address"
    },
    {
      "path": "documentInfo.institution",
      "label": "Institution"
    }
  ],
  "meta": {
    "temporaryUserCode": "temp-abc123",
    "submissionChannel": "mobile_app",
    "ipAddress": "192.168.1.1"
  }
}
```

**Document Types**:
- `utilityBill` - Electricity, water, gas bill
- `bankStatement` - Bank statement
- `governmentLetter` - Official government correspondence
- `rentalAgreement` - Lease or rental contract
- `mortgageStatement` - Mortgage documentation
- `taxDocument` - Tax assessment or bill
- `insuranceDocument` - Insurance policy or statement

**Response Fields**:
- `personalInfo` - Personal details (linked from identity verification)
- `documentAddress` - Address extracted from document
- `verifiedAddress` - Verified/corrected address
- `gpsAddress` - GPS coordinates if available
- `manualAddress` - Manually entered address
- `documentInfo` - Document details (institution, number, age)
- `media` - Document images with signed URLs
- `decisionFields` - Fields available for editing during review

**Notes**:
- Requires `view_address_verification_report_details` privilege
- Auto-updates status from `submitted` to `processing` when operations views
- Personal info auto-populated from linked identity verification
- Media URLs are signed and time-limited
- Creates audit log entry

---

#### 8. Update Address Verification Decision

Make a clearance or rejection decision on an address verification.

**Endpoint**: `POST /api/v1/merchant/address/report/decision`

**Request Body**:
```json
{
  "requestId": "507f1f77bcf86cd799439020",
  "decision": {
    "documentAddress": {
      "addressLine1": "123 Main Street",
      "addressLine2": "Kingston 6",
      "town": "Kingston",
      "parish": "St. Andrew"
    },
    "personalInfo": {
      "biography": {
        "firstName": "John",
        "middleName": "Michael",
        "lastName": "Smith"
      }
    }
  },
  "status": "cleared"
}
```

**Request Parameters**:
- `requestId` (string, required) - Verification result ID
- `decision` (object, required) - Decision details (corrected fields)
- `status` (string, required) - Decision status: `cleared` or `rejected`

**Success Response** (200):
```json
{}
```

**Process Flow**:

**When Status = `cleared`:**
1. Updates address verification with corrected details
2. Marks verification as cleared with timestamp
3. Publishes verification update to merchant webhook
4. Sends clearance email to customer
5. Updates verification status records
6. Closes related action items
7. Creates contact log entry

**When Status = `rejected`:**
1. Marks verification as rejected with timestamp
2. Sends rejection email to customer
3. Logs rejection in contact history
4. Updates verification status records

**Email Templates**:
- Clearance: Welcome message with success confirmation
- Rejection: Explanation and retry instructions

**Notes**:
- Requires operations role
- Updates name from linked identity verification if needed
- Publishes to merchant systems via webhook
- Sets Command Centre status to completed
- Sends optional copy to operations team

---

#### 9. Request Additional Information (Due Diligence)

Request additional documentation or information from customer.

**Endpoint**: `POST /api/v1/merchant/address/report/diligence-request`

**Request Body**:
```json
{
  "reportId": "507f1f77bcf86cd799439020",
  "note": "Please provide a more recent utility bill dated within the last 3 months.",
  "type": "address-document"
}
```

**Request Parameters**:
- `reportId` (string, required) - Address verification result ID
- `note` (string, required) - Message to customer explaining what's needed
- `type` (string, required) - Type of request:
  - `address-document` - Upload additional document
  - `address-gps` - Update GPS location
  - `address-photo` - Provide photo of location
  - `address-video` - Provide video of location

**Success Response** (201):
```json
{
  "success": true,
  "message": "Due diligence request submitted successfully.",
  "actionId": "507f1f77bcf86cd799439100"
}
```

**Error Response** (500):
```json
{
  "success": false,
  "message": "An error occurred while submitting the due diligence request.",
  "error": "Error details"
}
```

**Process Flow**:
1. Creates action item for customer with high priority
2. Sends email to customer with specific instructions
3. Creates Command Centre action for tracking
4. Customer sees action in their dashboard
5. Customer fulfils request
6. Operations team reviews new submission

**Action Types**:
- **address-document** - "Upload Additional Document"
- **address-gps** - "Update GPS Location"
- **address-photo** - "Provide Photo of Location"
- **address-video** - "Provide Video of Location"

**Email Content**:
Includes:
- Personalised greeting
- Specific instruction based on type
- Agent's note/reason for request
- Instructions to access Actions in Cleared app
- Reference number for tracking

**Notes**:
- Requires operations role
- Creates high-priority action item
- Sends email with agent's note
- Optional copy sent to operations team
- Links action to verification report

---

#### 10. Delete Address Verification Result

Soft delete an address verification result.

**Endpoint**: `POST /api/v1/merchant/address/report/:reportId/delete`

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
- Creates audit log entry

---

### Company Address Verification

#### 11. List Company Address Verification Results

Retrieve paginated list of company address verification results.

**Endpoint**: `GET /api/v1/merchant/address/companies/results`

**Query Parameters**:
- `query` (string, JSON encoded) - Search/filter/sort parameters:
  ```json
  {
    "keywords": "acme corporation",
    "page": 1,
    "sortField": "createdAt",
    "sortOrder": "desc",
    "filters": {
      "status": ["cleared", "processing"]
    }
  }
  ```

**Success Response** (200):
```json
{
  "records": [
    {
      "_id": "507f1f77bcf86cd799439030",
      "companyName": "Acme Corporation Ltd",
      "status": "cleared",
      "documentType": "utilityBill",
      "verifiedAddress": {
        "addressLine1": "45 Business Plaza",
        "addressLine2": "New Kingston",
        "town": "Kingston",
        "parish": "St. Andrew"
      },
      "createdAt": "2025-10-15T10:00:00Z",
      "updatedAt": "2025-10-16T12:00:00Z",
      "meta": {
        "accountNumber": "ACC-67890",
        "phoneNumber": "+1876xxxxxxx",
        "emailAddress": "admin@acme.com",
        "platform": "web",
        "fullAddress": "45 Business Plaza, New Kingston, Kingston"
      }
    }
  ],
  "columns": [
    {
      "header": "Name",
      "path": "companyName",
      "type": "link",
      "urlPrefix": "/address/companies/report/{id}/details"
    },
    {
      "header": "Status",
      "path": "status",
      "type": "status",
      "align": "text-center"
    },
    {
      "header": "Address",
      "path": "verifiedAddress.addressLine1"
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
      "options": ["cleared", "rejected", "processing", "submitted", "new"]
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

**Search Capabilities**:
- Company name
- Document number
- Address fields (line1, line2, town, parish)

**Notes**:
- Requires `view_address_verification_reports_list` privilege
- Auto-populates company name from company verification result if missing
- Operations users see contact information
- Supports same filtering as individual address verification

---

#### 12. Get Company Address Report

Get detailed company address verification result.

**Endpoint**: `GET /api/v1/merchant/address/companies/report/:reportId`

**URL Parameters**:
- `reportId` (string, required) - Company address verification result ID

**Success Response** (200):
```json
{
  "_id": "507f1f77bcf86cd799439030",
  "companyName": "Acme Corporation Ltd",
  "companyId": "507f1f77bcf86cd799439040",
  "status": "cleared",
  "documentType": "utilityBill",
  "documentNumber": "BILL-789456",
  "clearedAt": "2025-10-16T12:00:00Z",
  "personalInfo": {
    "biography": {
      "firstName": "John",
      "middleName": "Michael",
      "lastName": "Smith",
      "dateOfBirth": "1990-05-15"
    }
  },
  "documentAddress": {
    "addressLine1": "45 Business Plaza",
    "addressLine2": "New Kingston",
    "town": "Kingston",
    "parish": "St. Andrew"
  },
  "verifiedAddress": {
    "addressLine1": "45 Business Plaza",
    "addressLine2": "New Kingston",
    "town": "Kingston",
    "parish": "St. Andrew"
  },
  "documentInfo": {
    "institution": "Jamaica Public Service Company",
    "documentNumber": "BILL-789456",
    "issueDate": "2025-09-15",
    "documentAge": 31,
    "country": "Jamaica"
  },
  "reason": "Business address verification for company registration",
  "media": [
    {
      "label": "Utility Bill",
      "url": "https://s3.amazonaws.com/...",
      "type": "document"
    },
    {
      "label": "Business Location Photo",
      "url": "https://s3.amazonaws.com/...",
      "type": "image"
    }
  ],
  "company": {
    "_id": "507f1f77bcf86cd799439040",
    "name": "Acme Corporation Ltd",
    "registrationNumber": "12345678",
    "status": "verified"
  },
  "decisionFields": [
    {
      "path": "personalInfo.biography.firstName",
      "label": "First Name"
    },
    {
      "path": "documentAddress.addressLine1",
      "label": "Document Address"
    },
    {
      "path": "documentInfo.institution",
      "label": "Institution"
    },
    {
      "path": "reason",
      "label": "Reason for Verification"
    }
  ]
}
```

**Response Includes**:
- Complete company information
- Document address and verified address
- Document details (institution, age, number)
- Media with signed URLs
- Linked company verification details
- Editable decision fields

**Notes**:
- Requires `view_address_verification_report_details` privilege
- Auto-updates status from `submitted` to `processing` when operations views
- Media URLs regenerated as signed URLs
- Creates audit log entry
- Includes linked company data

---

#### 13. Update Company Address Verification Decision

Make a clearance or rejection decision on a company address verification.

**Endpoint**: `POST /api/v1/merchant/address/companies/report/decision`

**Request Body**:
```json
{
  "requestId": "507f1f77bcf86cd799439030",
  "decision": {
    "documentAddress": {
      "addressLine1": "45 Business Plaza",
      "addressLine2": "New Kingston",
      "town": "Kingston",
      "parish": "St. Andrew"
    },
    "verifiedAddress": {
      "addressLine1": "45 Business Plaza",
      "addressLine2": "New Kingston",
      "town": "Kingston",
      "parish": "St. Andrew"
    }
  },
  "status": "cleared"
}
```

**Request Parameters**:
- `requestId` (string, required) - Company address verification result ID
- `decision` (object, required) - Decision details (corrected fields)
- `status` (string, required) - Decision status: `cleared` or `rejected`

**Success Response** (200):
```json
{}
```

**Process Flow**:

**When Status = `cleared`:**
1. Updates company address verification with corrected details
2. Marks verification as cleared with timestamp
3. Finds company owner from CompanyPersonnel
4. Sends clearance email to company owner
5. Updates Command Centre status
6. Creates audit log entry

**When Status = `rejected`:**
1. Marks verification as rejected with timestamp
2. Sends rejection email to company owner
3. Updates Command Centre status

**Notes**:
- Requires operations role
- Finds company owner via CompanyPersonnel collection
- Sends email to company owner's address
- Optional copy sent to operations team
- Updates name from user record

---

