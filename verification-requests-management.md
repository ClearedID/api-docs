# Verification Requests Management API

## Overview

The Verification Requests Management API allows merchants to view, track, and manage verification requests throughout their lifecycle. Use these endpoints to monitor request status, view details, withdraw pending requests, and extend expiration times.

**Base Path**: `/api/v1/merchant/verifications/requests`

**Authentication**: Merchant JWT token in Authorization header

## Key Features

- **List Requests** – View all verification requests with filtering
- **Request Details** – Get complete information about specific requests
- **Withdraw Requests** – Cancel pending or in-progress requests
- **Extend Expiration** – Give customers more time to complete
- **Status Tracking** – Monitor request lifecycle
- **Search & Filter** – Find requests by customer, status, type, or date
- **Pagination** – Efficient handling of large request lists

## Verification Request Lifecycle

```
pending → awaiting clearance → approved/denied
   ↓
withdrawn (merchant cancels at any time before approved/denied)
```

**Status Flow**:
1. **pending** - Customer has not yet completed verification
2. **awaiting clearance** - Verification submitted, operations reviewing
3. **approved** - Customer approved sharing after clearance
4. **denied** - Customer denied sharing
5. **withdrawn** - Merchant cancelled the request

## Creating Verification Requests

To create new verification requests, see:
- **[Initiate Verification API](./initiate-verification.md)** - Complete guide to creating verification requests

## Endpoints

### 1. List Verification Requests

Retrieve paginated list of verification requests.

**Endpoint**: `GET /api/v1/merchant/verifications/requests`

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
      "types": ["identity", "address"],
      "createdAt_start": "2025-10-01",
      "createdAt_end": "2025-10-31"
    },
    "customerId": "507f1f77bcf86cd799439050"
  }
  ```

**OR direct query parameters**:
- `keywords` (string, optional) - Search keywords
- `page` (number, optional) - Page number (default: 1)
- `sortField` (string, optional) - Sort field (default: `createdAt`)
- `sortOrder` (string, optional) - Sort order: `asc` or `desc` (default: `desc`)
- `filters` (object, optional) - Filter criteria

**Success Response** (200):
```json
{
  "records": [
    {
      "_id": "507f1f77bcf86cd799439013",
      "name": "John Smith",
      "types": ["identity", "address"],
      "originator": "HR Department",
      "status": "pending",
      "expiresAt": "2025-10-20T10:00:00Z",
      "emailAddress": "john@example.com",
      "phoneNumber": "+1876xxxxxxx",
      "createdAt": "2025-10-19T09:00:00Z",
      "userId": "507f1f77bcf86cd799439050",
      "organisationId": "507f1f77bcf86cd799439031"
    }
  ],
  "columns": [...],
  "filters": [...],
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
- `awaiting clearance` - Verification submitted, awaiting operations review
- `withdrawn` - Merchant withdrew request (excluded from default results)

**Notes**:
- Requires `view_identity_verification_requests_list` privilege
- Non-operations users see only their organisation's requests
- Withdrawn requests excluded by default
- Supports keyword search on name and originator

---

### 2. Get Request Details

Get detailed information about a specific verification request.

**Endpoint**: `GET /api/v1/merchant/verifications/requests/:requestId/details`

**URL Parameters**:
- `requestId` (string, required) - Request unique identifier

**Success Response** (200):
```json
{
  "_id": "507f1f77bcf86cd799439013",
  "name": "John Smith",
  "types": ["identity", "address"],
  "status": "approved",
  "originator": "HR Department",
  "organisationId": "507f1f77bcf86cd799439031",
  "userId": "507f1f77bcf86cd799439050",
  "customerId": "507f1f77bcf86cd799439060",
  "emailAddress": "john@example.com",
  "phoneNumber": "+1876xxxxxxx",
  "expiresAt": "2025-10-20T10:00:00Z",
  "createdAt": "2025-10-19T09:00:00Z",
  "approvedAt": "2025-10-19T11:30:00Z",
  "pendingApprovals": {
    "identity": {
      "granted": true,
      "grantedAt": "2025-10-19T11:30:00Z"
    },
    "address": {
      "granted": true,
      "grantedAt": "2025-10-19T11:30:00Z"
    }
  }
}
```

**Error Response** (404):
```json
{
  "message": "Verification not found"
}
```

**Notes**:
- Triggers verification request progress handler
- Auto-populates customer ID if found
- Auto-populates contact info from user if missing
- Access controlled by organisation

---

### 3. Withdraw Verification Request

Withdraw a pending verification request.

**Endpoint**: `POST /api/v1/merchant/verifications/requests/:requestId/withdraw`

**URL Parameters**:
- `requestId` (string, required) - Request ID to withdraw

**Success Response** (200):
```json
{
  "success": true,
  "message": "Verification request withdrawn successfully",
  "notification": {
    "type": "success",
    "title": "Success",
    "text": "Verification request has been withdrawn successfully."
  }
}
```

**Error Responses**:

400 Bad Request - Cannot withdraw:
```json
{
  "notification": {
    "type": "error",
    "title": "Cannot Withdraw",
    "text": "Only pending or awaiting clearance verification requests can be withdrawn."
  },
  "error": true
}
```

403 Forbidden - Access denied:
```json
{
  "notification": {
    "type": "error",
    "title": "Access Denied",
    "text": "You can only withdraw verification requests from your own organisation."
  },
  "error": true
}
```

404 Not Found:
```json
{
  "notification": {
    "type": "error",
    "title": "Not Found",
    "text": "Verification request not found."
  },
  "error": true
}
```

**Process**:
1. Validates request exists and belongs to organisation
2. Checks request can be withdrawn (pending or awaiting clearance only)
3. Updates status to `withdrawn`
4. Sets `withdrawnAt` timestamp and `withdrawnBy` client ID
5. Creates audit log entry
6. Sends notification email to customer (optional)

**Notes**:
- Requires `view_identity_verification_requests_list` privilege
- Can only withdraw pending or awaiting clearance requests
- Cannot withdraw approved, denied, or completed requests
- Access restricted to owning organisation (unless operations user)

---

### 4. Extend Verification Request

Extend the expiration date of a verification request.

**Endpoint**: `POST /api/v1/merchant/verifications/requests/:requestId/extend`

**URL Parameters**:
- `requestId` (string, required) - Request ID to extend

**Request Body**:
```json
{
  "expiration": {
    "expiresAt": "2025-10-25T10:00:00Z"
  }
}
```

**Request Parameters**:
- `expiration` (object, required) - Expiration configuration
  - `expiresAt` (string, required) - New expiration date/time (ISO format)

**Success Response** (200):
```json
{
  "success": true,
  "message": "Verification request extended successfully",
  "data": {
    "expiresAt": "2025-10-25T10:00:00Z",
    "extendedAt": "2025-10-20T11:00:00Z"
  },
  "notification": {
    "type": "success",
    "title": "Success",
    "text": "Verification request expiration has been extended successfully."
  }
}
```

**Error Responses**:

400 Bad Request - Already extended:
```json
{
  "notification": {
    "type": "error",
    "title": "Already Extended",
    "text": "This verification request has already been extended once and cannot be extended again."
  },
  "error": true
}
```

400 Bad Request - Cannot extend expired:
```json
{
  "notification": {
    "type": "error",
    "title": "Cannot Extend",
    "text": "Cannot extend an expired verification request."
  },
  "error": true
}
```

400 Bad Request - Invalid status:
```json
{
  "notification": {
    "type": "error",
    "title": "Cannot Extend",
    "text": "Only pending or awaiting clearance verification requests can be extended."
  },
  "error": true
}
```

**Restrictions**:
- Can only extend once per request
- Can only extend pending or awaiting clearance requests
- Cannot extend expired requests
- Cannot extend approved or denied requests
- New expiration must be valid date/time

**Process**:
1. Validates request exists and belongs to organisation
2. Checks request can be extended (not already extended, not expired, correct status)
3. Validates new expiration date format
4. Updates `expiresAt` timestamp
5. Sets `extendedAt` timestamp and `extendedBy` client ID
6. Creates audit log entry
7. Sends notification email to customer (optional)

**Notes**:
- Requires `view_identity_verification_requests_list` privilege
- One extension allowed per request
- Used when customer needs more time
- Customer notified of extension

---

## Complete Management Workflow

### Example: Monitor and Manage Requests

```javascript
// 1. List all pending requests
const listPendingRequests = async () => {
  const query = {
    keywords: '',
    page: 1,
    sortField: 'createdAt',
    sortOrder: 'desc',
    filters: {
      status: ['pending', 'awaiting clearance']
    }
  };
  
  const response = await fetch(
    `/api/v1/merchant/verifications/requests?query=${encodeURIComponent(JSON.stringify(query))}`,
    {
      headers: {
        'Authorization': `Bearer ${apiKey}`
      }
    }
  );
  
  const { records, paging } = await response.json();
  
  console.log(`${records.length} pending requests (${paging.recordCount} total)`);
  return records;
};

// 2. Get details of specific request
const getRequestDetails = async (requestId) => {
  const response = await fetch(
    `/api/v1/merchant/verifications/requests/${requestId}/details`,
    {
      headers: {
        'Authorization': `Bearer ${apiKey}`
      }
    }
  );
  
  const request = await response.json();
  
  console.log('Request status:', request.status);
  console.log('Expires at:', request.expiresAt);
  console.log('Types:', request.types);
  
  return request;
};

// 3. Extend request that's about to expire
const extendRequestExpiration = async (requestId) => {
  // Extend by 2 days from now
  const newExpiration = new Date(Date.now() + 2 * 24 * 60 * 60 * 1000);
  
  const response = await fetch(
    `/api/v1/merchant/verifications/requests/${requestId}/extend`,
    {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${apiKey}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        expiration: {
          expiresAt: newExpiration.toISOString()
        }
      })
    }
  );
  
  const result = await response.json();
  console.log('Extended expiration to:', result.data.expiresAt);
};

// 4. Withdraw request if customer no longer needs verification
const withdrawRequest = async (requestId) => {
  const response = await fetch(
    `/api/v1/merchant/verifications/requests/${requestId}/withdraw`,
    {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${apiKey}`
      }
    }
  );
  
  const result = await response.json();
  console.log(result.message);
};

// 5. Monitor expiring requests
const monitorExpiringRequests = async () => {
  const now = new Date();
  const tomorrow = new Date(now.getTime() + 24 * 60 * 60 * 1000);
  
  const query = {
    filters: {
      status: ['pending'],
      expiresAt_end: tomorrow.toISOString()
    }
  };
  
  const response = await fetch(
    `/api/v1/merchant/verifications/requests?query=${encodeURIComponent(JSON.stringify(query))}`,
    {
      headers: { 'Authorization': `Bearer ${apiKey}` }
    }
  );
  
  const { records } = await response.json();
  
  console.log(`${records.length} requests expiring within 24 hours`);
  
  // Optionally extend them
  for (const request of records) {
    await extendRequestExpiration(request._id);
  }
};

## Verification Types

### Supported Services

| Service | Description | Typical Cost |
|---------|-------------|--------------|
| `identity` | ID document + biometric | 5 credits |
| `address` | Proof of address | 3 credits |
| `income` | Income verification | 10 credits |
| `employment` | Employment history | 8 credits |
| `qualification` | Education/certification | 7 credits |
| `reference` | Reference checks | 5 credits |
| `company` | Company verification | 7 credits |
| `background` | Background screening | 15+ credits |

### Request Combinations

**Common Combinations**:

**Basic KYC**:
```json
{
  "verificationRequests": [
    { "type": "identity", "required": true }
  ]
}
```

**Enhanced KYC**:
```json
{
  "verificationRequests": [
    { "type": "identity", "required": true },
    { "type": "address", "required": true }
  ]
}
```

**Employment Screening**:
```json
{
  "verificationRequests": [
    { "type": "identity", "required": true },
    { "type": "address", "required": true },
    { "type": "employment", "required": true },
    { "type": "reference", "required": false }
  ]
}
```

**Comprehensive Screening**:
```json
{
  "verificationRequests": [
    { "type": "identity", "required": true },
    { "type": "address", "required": true },
    { "type": "employment", "required": true },
    { "type": "income", "required": false },
    { "type": "background", "required": true }
  ]
}
```

## Request Status Lifecycle

```
pending → awaiting clearance → approved/denied
   ↓
withdrawn (at any time before approved/denied)
```

**Status Flow**:
1. **pending** - Customer has not yet completed verification
2. **awaiting clearance** - Verification submitted, operations reviewing
3. **approved** - Customer approved sharing after clearance
4. **denied** - Customer denied sharing
5. **withdrawn** - Merchant cancelled the request

## Complete Integration Example

### Full Workflow

```javascript
// 1. Initiate verification request
const initiateResponse = await fetch(
  '/api/v1/merchant/identity/verification/initiate',
  {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${apiKey}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      name: 'John Smith',
      emailAddress: 'john@example.com',
      phoneNumber: '+18761234567',
      verificationRequests: [
        {
          type: 'identity',
          required: true,
          description: 'Government-issued ID verification'
        },
        {
          type: 'address',
          required: true,
          description: 'Proof of residential address'
        }
      ],
      summary: 'Identity and address verification for employment',
      expiration: {
        expiresAt: new Date(Date.now() + 48 * 60 * 60 * 1000).toISOString()
      }
    })
  }
);

const { requestId, success } = await initiateResponse.json();

if (success) {
  console.log('Verification request created:', requestId);
  
  // 2. Monitor request status
  const checkStatus = async () => {
    const response = await fetch(
      `/api/v1/merchant/verifications/requests/${requestId}/details`,
      {
        headers: { 'Authorization': `Bearer ${apiKey}` }
      }
    );
    
    const request = await response.json();
    
    if (request.status === 'approved') {
      console.log('Verification approved!');
      // Retrieve verification results
      // ...
    } else if (request.status === 'denied') {
      console.log('Verification denied');
    }
    
    return request.status;
  };
  
  // Poll status every 30 seconds
  const pollInterval = setInterval(async () => {
    const status = await checkStatus();
    
    if (['approved', 'denied', 'withdrawn'].includes(status)) {
      clearInterval(pollInterval);
    }
  }, 30000);
  
  // 3. Optionally extend request if needed
  setTimeout(async () => {
    await fetch(
      `/api/v1/merchant/verifications/requests/${requestId}/extend`,
      {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${apiKey}`,
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          expiration: {
            expiresAt: new Date(Date.now() + 72 * 60 * 60 * 1000).toISOString()
          }
        })
      }
    );
  }, 40 * 60 * 60 * 1000); // Extend after 40 hours if still pending
  
  // 4. Withdraw if needed
  // await fetch(`/api/v1/merchant/verifications/requests/${requestId}/withdraw`, {
  //   method: 'POST',
  //   headers: { 'Authorization': `Bearer ${apiKey}` }
  // });
}
```

## Best Practices

### 1. Request Creation

- **Verify Customer Info** - Check if customer exists before creating
- **Appropriate Expiration** - Use 24-48 hours for most cases
- **Clear Description** - Explain what verification is needed and why
- **Required vs Optional** - Mark critical verifications as required
- **Cost Awareness** - Be mindful of credit costs for multiple verifications

### 2. Request Management

- **Monitor Pending** - Check pending requests daily
- **Extend When Needed** - Extend before expiration if customer requests
- **Withdraw Unused** - Withdraw requests that are no longer needed
- **Track Customer Response** - Monitor approval/denial rates

### 3. Communication

- **Clear Messaging** - Use clear summary and descriptions
- **Timely Requests** - Don't request verifications too early
- **Follow Up** - Send reminders for pending requests
- **Respect Privacy** - Don't over-request verifications

### 4. Error Handling

- **Handle Duplicates** - Check for existing requests before creating
- **Validate Contact Info** - Ensure email/phone is valid
- **Quota Management** - Monitor service quotas
- **Grace Periods** - Allow time for customer to complete

## Error Codes

| Status Code | Description |
|------------|-------------|
| 200 | Success |
| 201 | Created |
| 400 | Bad Request (validation error) |
| 401 | Unauthorised (invalid/missing token) |
| 403 | Forbidden (insufficient permissions/quota exceeded) |
| 404 | Not Found |
| 500 | Internal Server Error |

## Rate Limits

- **Initiate Requests**: 100 per hour per organisation
- **List Requests**: 500 per hour per organisation
- **Withdraw/Extend**: 200 per hour per organisation

## Privileges Required

| Action | Privilege |
|--------|-----------|
| List Requests | `view_identity_verification_requests_list` |
| Get Details | `view_identity_verification_requests_list` |
| Initiate Request | `create_{service}_verification_request` |
| Withdraw Request | `view_identity_verification_requests_list` |
| Extend Request | `view_identity_verification_requests_list` |

Replace `{service}` with actual service name: `identity`, `address`, etc.

## Related Documentation

- [Merchant Updates API](./merchant-updates.md)
- [Identity Verification API](./identity/identity-api.md)
- [Address Verification API](./address/address-api.md)
- [Verification Links API](./verification-links/verification-links-api.md)
- [API Overview](./README.md)

