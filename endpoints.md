# Endpoints

## Overview

Cleared uses a powerful, secure REST API under the SWF Cloud infrastructure, enabling clients to initiate and manage verification requests across multiple services such as identity, address, income, employment, background checks, and company verifications.

This page explains the base URLs, how to make API calls, and general guidelines for using the `/api/v1/merchant` endpoints.

## Base URLs

All API requests must be made to the `/api/v1/merchant` path. There are two separate environments:

### Sandbox Environment (UAT / Testing)

Used for integration testing and QA with test accounts and dummy data.

**Base URL:**
```
https://enterprise-qa.cleared.id/api/v1/merchant
```

**Admin Portal:**
```
https://enterprise-qa.cleared.id/admin
```

**Characteristics:**
- Free to use (no credit charges)
- Test data and dummy results
- Unlimited requests for testing
- Same API structure as production
- No real personal information processed

**Example Request:**
```bash
curl -X GET https://enterprise-qa.cleared.id/api/v1/merchant/identity/results \
  -H "Authorization: Bearer YOUR_SANDBOX_API_KEY" \
  -H "Content-Type: application/json"
```

### Production Environment (Live Data)

Used in the live production environment with real data.

**Base URL:**
```
https://cleared.id/api/v1/merchant
```

**Admin Portal:**
```
https://cleared.id/admin
```

**Characteristics:**
- Credit-based pricing (per verification)
- Real personal and business data
- Production SLA guarantees
- Full compliance and security
- Audit logging and reporting

**Example Request:**
```bash
curl -X GET https://cleared.id/api/v1/merchant/identity/results \
  -H "Authorization: Bearer YOUR_PRODUCTION_API_KEY" \
  -H "Content-Type: application/json"
```

## API Services

The SWF Cloud API provides the following verification and screening services:

### Identity Verification

Verify identities using government-issued ID documents and biometric matching.

**Base Path:** `/api/v1/merchant/identity`

**Capabilities:**
- ID document verification (passport, driver's license, national ID)
- Facial biometric matching
- Liveness detection
- Age verification
- Document authenticity checks

**Documentation:** [Identity Verification API](./services/identity-verification.md)

---

### Address Verification

Validate residential and business addresses against official records.

**Base Path:** `/api/v1/merchant/address`

**Capabilities:**
- Address validation and standardisation
- Proof of address document verification
- Utility bill verification
- Occupancy verification
- Historical address searches

**Documentation:** [Address Verification API](./merchant/address-verification.md)

---

### Income Verification

Verify income and salary information for individuals.

**Base Path:** `/api/v1/merchant/income`

**Capabilities:**
- Bank statement analysis
- Payslip verification
- Tax return verification
- Employment income validation
- Income trend analysis

**Documentation:** [Income Verification API](./merchant/income-verification.md)

---

### Employment Verification

Confirm employment history and current employment status.

**Base Path:** `/api/v1/merchant/employment`

**Capabilities:**
- Current employment verification
- Employment history checks
- Position and salary verification
- Employer reference checks
- Employment gap analysis

**Documentation:** [Employment Verification API](./merchant/employment-verification.md)

---

### Qualifications Verification

Verify academic and professional qualifications.

**Base Path:** `/api/v1/merchant/qualifications`

**Capabilities:**
- Educational qualification verification
- Professional certification checks
- Degree and diploma authentication
- Institution verification
- Transcript validation

**Documentation:** [Qualifications Verification API](./merchant/qualifications-verification.md)

---

### Reference Verification

Verify personal and professional references.

**Base Path:** `/api/v1/merchant/references`

**Capabilities:**
- Reference contact verification
- Reference questionnaire management
- Character reference checks
- Professional reference validation
- Reference response tracking

**Documentation:** [Reference Verification API](./merchant/reference-verification.md)

---

### Background Checks

Comprehensive background screening and criminal record checks.

**Base Path:** `/api/v1/merchant/background`

**Capabilities:**
- Criminal record checks
- Credit history checks
- Court record searches
- Sanctions and PEP screening
- Media and adverse news checks

**Documentation:** [Background Checks API](./merchant/background-checks.md)

---

### Company Verification

Verify corporate entities and business information.

**Base Path:** `/api/v1/merchant/company`

**Capabilities:**
- Company registration verification
- Director and officer checks
- Financial status verification
- TRN/Tax ID validation
- Corporate structure verification

**Documentation:** [Company Verification API](./merchant/company-verification.md)

---

### Digital Signatures

Secure document signing with digital certificates.

**Base Path:** `/api/v1/merchant/signatures`

**Capabilities:**
- Document signature requests
- Multi-party signing workflows
- Digital certificate application
- Template management
- Audit trail and compliance

**Documentation:** [Digital Signatures API](./merchant/digital-signatures.md)

---

## Common Endpoint Patterns

All service endpoints follow consistent patterns for common operations:

### List Resources

Retrieve a list of resources (e.g., verifications, checks, documents).

**Pattern:** `GET /api/v1/merchant/{service}/{resource}`

**Example:**
```bash
GET /api/v1/merchant/identity/verifications
```

**Query Parameters:**
- `page` – Page number (default: 1)
- `limit` – Items per page (default: 20, max: 100)
- `status` – Filter by status
- `search` – Search by keyword
- `sortBy` – Sort field
- `sortOrder` – Sort direction (asc/desc)

**Response:**
```json
{
  "success": true,
  "data": {
    "records": [...],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 156,
      "pages": 8
    }
  }
}
```

---

### Get Resource by ID

Retrieve a specific resource by its unique identifier.

**Pattern:** `GET /api/v1/merchant/{service}/{resource}/{id}`

**Example:**
```bash
GET /api/v1/merchant/identity/verifications/507f1f77bcf86cd799439011
```

**Response:**
```json
{
  "success": true,
  "data": {
    "_id": "507f1f77bcf86cd799439011",
    "status": "completed",
    "result": {...}
  }
}
```

---

### Create Resource

Create a new resource (e.g., initiate verification).

**Pattern:** `POST /api/v1/merchant/{service}/{resource}`

**Example:**
```bash
POST /api/v1/merchant/identity/verifications
Content-Type: application/json

{
  "firstName": "John",
  "lastName": "Smith",
  "documentType": "passport",
  "documentNumber": "A12345678"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "_id": "507f1f77bcf86cd799439011",
    "status": "pending",
    "verificationUrl": "https://verify.cleared.id/v/abc123"
  },
  "message": "Verification initiated successfully"
}
```

---

### Update Resource

Update an existing resource.

**Pattern:** `PUT /api/v1/merchant/{service}/{resource}/{id}` or `PATCH /api/v1/merchant/{service}/{resource}/{id}`

**Example:**
```bash
PATCH /api/v1/merchant/identity/verifications/507f1f77bcf86cd799439011
Content-Type: application/json

{
  "status": "cancelled"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "_id": "507f1f77bcf86cd799439011",
    "status": "cancelled"
  },
  "message": "Verification updated successfully"
}
```

---

### Delete Resource

Delete a resource (soft delete in most cases).

**Pattern:** `DELETE /api/v1/merchant/{service}/{resource}/{id}`

**Example:**
```bash
DELETE /api/v1/merchant/identity/verifications/507f1f77bcf86cd799439011
```

**Response:**
```json
{
  "success": true,
  "message": "Verification deleted successfully"
}
```

---

## Request Format

### Headers

All API requests must include the following headers:

```
Authorization: Bearer YOUR_API_KEY
Content-Type: application/json
```

**Optional Headers:**
```
X-Request-ID: unique-request-identifier
X-Idempotency-Key: unique-idempotency-key
Accept: application/json
```

### Request Body

Request bodies must be valid JSON:

```json
{
  "field1": "value1",
  "field2": "value2",
  "nestedObject": {
    "nestedField": "nestedValue"
  },
  "arrayField": ["item1", "item2"]
}
```

### File Uploads

For endpoints that accept file uploads, use `multipart/form-data`:

```bash
curl -X POST https://cleared.id/api/v1/merchant/identity/documents/upload \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -F "documentType=passport" \
  -F "file=@/path/to/document.pdf"
```

## Response Format

### Success Response

All successful responses follow this structure:

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

All error responses follow this structure:

```json
{
  "success": false,
  "error": "Error message",
  "code": "ERROR_CODE",
  "details": "Additional error details or validation errors"
}
```

### HTTP Status Codes

| Status Code | Description | Use Case |
|------------|-------------|----------|
| 200 | OK | Request successful |
| 201 | Created | Resource created successfully |
| 204 | No Content | Request successful, no content to return |
| 400 | Bad Request | Invalid request parameters |
| 401 | Unauthorised | Invalid or missing API key |
| 403 | Forbidden | Insufficient permissions |
| 404 | Not Found | Resource not found |
| 409 | Conflict | Resource already exists or conflict |
| 422 | Unprocessable Entity | Validation errors |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Server error |
| 503 | Service Unavailable | Service temporarily unavailable |

## Pagination

List endpoints support pagination with the following parameters:

**Query Parameters:**
- `page` – Page number (default: 1)
- `limit` – Items per page (default: 20, max: 100)

**Response:**
```json
{
  "success": true,
  "data": {
    "records": [...],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 156,
      "pages": 8,
      "hasNextPage": true,
      "hasPreviousPage": false
    }
  }
}
```

## Filtering and Sorting

### Filtering

Filter results using query parameters:

```bash
GET /api/v1/merchant/identity/verifications?status=completed&country=GB
```

**Common Filters:**
- `status` – Filter by status
- `dateFrom` – Results from date (ISO 8601)
- `dateTo` – Results to date (ISO 8601)
- `search` – Search across text fields

### Sorting

Sort results using sort parameters:

```bash
GET /api/v1/merchant/identity/verifications?sortBy=createdAt&sortOrder=desc
```

**Parameters:**
- `sortBy` – Field to sort by
- `sortOrder` – Sort direction: `asc` or `desc`

## Idempotency

For POST requests that create resources, use the `X-Idempotency-Key` header to prevent duplicate operations:

```bash
curl -X POST https://cleared.id/api/v1/merchant/identity/verifications \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -H "X-Idempotency-Key: unique-key-123" \
  -d '{"firstName": "John", "lastName": "Smith"}'
```

If the same idempotency key is used again within 24 hours, you'll receive the original response instead of creating a duplicate resource.

## Webhooks

Configure webhooks to receive real-time notifications about verification events:

**Webhook Events:**
- `verification.completed` – Verification completed
- `verification.failed` – Verification failed
- `verification.expired` – Verification expired
- `document.signed` – Document signed
- `background.check.completed` – Background check completed

**Configuration:**
1. Go to Admin Portal > Integrations > Webhooks
2. Add webhook endpoint URL
3. Select events to receive
4. Verify webhook signature

**Webhook Payload:**
```json
{
  "event": "verification.completed",
  "timestamp": "2025-10-19T12:00:00Z",
  "data": {
    "verificationId": "507f1f77bcf86cd799439011",
    "status": "verified",
    "result": {...}
  },
  "signature": "sha256=..."
}
```

**Documentation:** [Webhooks Guide](./webhooks.md)

## Rate Limiting

API requests are rate limited to ensure fair usage:

**Limits:**
- **Sandbox:** 100 requests per minute per API key
- **Production:** 1000 requests per minute per API key

**Response Headers:**
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 987
X-RateLimit-Reset: 1634567890
```

**Rate Limit Exceeded:**
```json
{
  "success": false,
  "error": "Rate limit exceeded",
  "code": "RATE_LIMIT_EXCEEDED",
  "retryAfter": 60
}
```

## Error Handling

Implement proper error handling in your integration:

```javascript
try {
  const response = await axios.post(
    'https://cleared.id/api/v1/merchant/identity/verifications',
    requestData,
    {
      headers: {
        'Authorization': `Bearer ${apiKey}`,
        'Content-Type': 'application/json'
      }
    }
  );
  
  if (response.data.success) {
    // Handle success
    console.log('Verification initiated:', response.data.data);
  }
} catch (error) {
  if (error.response) {
    // API returned error response
    const { status, data } = error.response;
    
    switch (status) {
      case 400:
        console.error('Invalid request:', data.error);
        break;
      case 401:
        console.error('Authentication failed:', data.error);
        break;
      case 429:
        console.error('Rate limit exceeded, retry after:', data.retryAfter);
        break;
      default:
        console.error('API error:', data.error);
    }
  } else if (error.request) {
    // Request made but no response
    console.error('No response from server');
  } else {
    // Request setup error
    console.error('Request error:', error.message);
  }
}
```

## Testing

### Using cURL

```bash
# Test authentication
curl -X GET https://cleared.id/api/v1/merchant/auth/verify \
  -H "Authorization: Bearer YOUR_API_KEY"

# Create verification
curl -X POST https://cleared.id/api/v1/merchant/identity/verifications \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "firstName": "John",
    "lastName": "Smith",
    "email": "john@example.com",
    "documentType": "passport"
  }'
```

### Using Postman

1. Import the [SWF Cloud API Postman Collection](./postman-collection.json)
2. Set environment variables:
   - `api_key` – Your API key
   - `base_url` – Environment base URL
3. Test endpoints from the collection

## Next Steps

Now that you understand the endpoints structure:

1. **Explore service-specific documentation** for detailed endpoint information
2. **Test in Sandbox** before going to production
3. **Implement error handling** and retry logic
4. **Set up webhooks** for real-time notifications
5. **Monitor usage** via the Admin Portal

## Available Service APIs

- [Identity Verification API](./merchant/identity-verification.md)
- [Address Verification API](./merchant/address-verification.md)
- [Income Verification API](./merchant/income-verification.md)
- [Employment Verification API](./merchant/employment-verification.md)
- [Background Checks API](./merchant/background-checks.md)
- [Company Verification API](./merchant/company-verification.md)
- [Digital Signatures API](./merchant/digital-signatures.md)

---

← [Registration](./registration.md) | [Service APIs](./merchant/) →

