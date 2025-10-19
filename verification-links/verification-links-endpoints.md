# Verification Links API - Endpoints

← [Verification Links API Overview](./verification-links-api.md)

---

## Endpoints

### Merchant Verification Links Management

#### 1. List Verification Links

Retrieve a list of verification links for the authenticated organisation.

**Endpoint**: `GET /api/v1/merchant/verification-links`

**Query Parameters**:
- `status` (string, optional) - Filter by status: `active`, `inactive`, `archived`, `all` (default: all)
- `limit` (number, optional) - Number of results per page (default: 50, max: 100)
- `offset` (number, optional) - Pagination offset (default: 0)

**Success Response** (200):
```json
{
  "success": true,
  "data": [
    {
      "_id": "507f1f77bcf86cd799439100",
      "linkId": "employee-verification",
      "linkTitle": "Employee Verification",
      "description": "Identity verification for new employees",
      "status": "active",
      "verificationType": "identity",
      "destination": "https://company.com/onboarding/complete",
      "clientBranding": {
        "companyName": "Acme Corporation",
        "logo": "https://example.com/logo.png",
        "primaryColor": "#007bff"
      },
      "styling": {
        "theme": "light"
      },
      "webhookConfig": {
        "url": "https://api.company.com/webhooks/verification",
        "active": true
      },
      "urlParameters": [
        {
          "key": "source",
          "value": "employee-onboarding"
        }
      ],
      "clientId": "507f1f77bcf86cd799439030",
      "organisationId": "507f1f77bcf86cd799439031",
      "createdBy": "507f1f77bcf86cd799439032",
      "createdAt": "2025-10-15T10:00:00Z"
    }
  ],
  "pagination": {
    "total": 12,
    "limit": 50,
    "offset": 0,
    "hasMore": false
  }
}
```

**Notes**:
- Returns only links owned by the authenticated organisation
- Webhook secrets are removed from response for security
- Results sorted by creation date (newest first)
- Supports pagination for large result sets

---

#### 2. Get Verification Link by ID

Retrieve a specific verification link by its linkId.

**Endpoint**: `GET /api/v1/merchant/verification-links/:linkId`

**URL Parameters**:
- `linkId` (string, required) - The verification link identifier

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "_id": "507f1f77bcf86cd799439100",
    "linkId": "employee-verification",
    "linkTitle": "Employee Verification",
    "description": "Identity verification for new employees",
    "status": "active",
    "verificationType": "identity",
    "destination": "https://company.com/onboarding/complete",
    "clientBranding": {
      "companyName": "Acme Corporation",
      "logo": "https://example.com/logo.png",
      "primaryColor": "#007bff",
      "accentColor": "#0056b3"
    },
    "styling": {
      "theme": "light",
      "buttonStyle": "rounded",
      "fontFamily": "Inter, sans-serif"
    },
    "webhookConfig": {
      "url": "https://api.company.com/webhooks/verification",
      "active": true
    },
    "urlParameters": [
      {
        "key": "source",
        "value": "employee-onboarding"
      },
      {
        "key": "department",
        "value": "engineering"
      }
    ],
    "clientId": "507f1f77bcf86cd799439030",
    "organisationId": "507f1f77bcf86cd799439031",
    "createdBy": "507f1f77bcf86cd799439032",
    "createdAt": "2025-10-15T10:00:00Z",
    "updatedAt": "2025-10-18T14:30:00Z"
  }
}
```

**Error Response** (404):
```json
{
  "success": false,
  "message": "Verification link not found"
}
```

**Notes**:
- Only returns links owned by the authenticated organisation
- Webhook secret is removed from response
- Used to retrieve link configuration for editing

---

#### 3. Create Verification Link

Create a new verification link.

**Endpoint**: `POST /api/v1/merchant/verification-links/create`

**Request Body**:
```json
{
  "linkId": "tenant-screening",
  "linkTitle": "Tenant Verification",
  "description": "Identity and address verification for new tenants",
  "verificationType": "identity",
  "destination": "https://property-mgmt.com/tenants/verified",
  "clientBranding": {
    "companyName": "Premier Property Management",
    "logo": "https://property-mgmt.com/logo.png",
    "primaryColor": "#2563eb",
    "accentColor": "#1e40af"
  },
  "styling": {
    "theme": "light",
    "buttonStyle": "rounded",
    "fontFamily": "Inter, sans-serif"
  },
  "webhookUrl": "https://api.property-mgmt.com/webhooks/verification",
  "webhookSecret": "your-webhook-secret-key",
  "webhookEnabled": true,
  "urlParameters": [
    {
      "key": "propertyId",
      "value": "PROP-456"
    },
    {
      "key": "applicationId",
      "value": "APP-789"
    }
  ],
  "status": "active"
}
```

**Request Parameters**:
- `linkId` (string, optional) - Custom link identifier (auto-generated if not provided)
- `linkTitle` (string, required) - Display title for the link
- `description` (string, required) - Description shown to customers
- `verificationType` (string, required) - Type: `identity`, `address`, `combined`, `company`
- `destination` (string, required) - URL to redirect after verification
- `clientBranding` (object, optional) - Branding configuration
  - `companyName` (string) - Company name to display
  - `logo` (string) - Logo URL
  - `primaryColor` (string) - Primary brand color (hex)
  - `accentColor` (string) - Accent brand color (hex)
- `styling` (object, optional) - Styling preferences
  - `theme` (string) - `light` or `dark`
  - `buttonStyle` (string) - `rounded`, `square`, `pill`
  - `fontFamily` (string) - Font family name
- `webhookUrl` (string, optional) - Webhook endpoint URL
- `webhookSecret` (string, optional) - Webhook HMAC secret
- `webhookEnabled` (boolean, optional) - Enable/disable webhooks
- `urlParameters` (array, optional) - Custom URL parameters
  - `key` (string) - Parameter name
  - `value` (string) - Parameter value
- `status` (string, optional) - Initial status (default: `active`)

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "_id": "507f1f77bcf86cd799439100",
    "linkId": "tenant-screening",
    "linkTitle": "Tenant Verification",
    "description": "Identity and address verification for new tenants",
    "status": "active",
    "verificationType": "identity",
    "destination": "https://property-mgmt.com/tenants/verified",
    "clientBranding": {...},
    "styling": {...},
    "webhookConfig": {
      "url": "https://api.property-mgmt.com/webhooks/verification",
      "secret": "your-webhook-secret-key",
      "active": true
    },
    "urlParameters": [...],
    "clientId": "507f1f77bcf86cd799439030",
    "organisationId": "507f1f77bcf86cd799439031",
    "createdBy": "507f1f77bcf86cd799439032",
    "createdAt": "2025-10-19T10:00:00Z"
  },
  "message": "Verification link created successfully"
}
```

**Error Responses**:

400 Bad Request - Duplicate Link ID:
```json
{
  "success": false,
  "message": "A verification link with this ID already exists"
}
```

500 Internal Server Error:
```json
{
  "success": false,
  "message": "Internal server error"
}
```

**Link ID Generation**:
- If not provided, auto-generated (12 characters)
- Must be unique within organisation
- URL-safe characters only
- Case-sensitive

**Notes**:
- LinkId becomes part of the verification URL
- Choose descriptive, memorable link IDs
- Webhook secret stored securely
- URL parameters passed to destination URL after verification

---

#### 4. Update Verification Link

Update an existing verification link's configuration.

**Endpoint**: `POST /api/v1/merchant/verification-links/:linkId/update`

**URL Parameters**:
- `linkId` (string, required) - Link identifier to update

**Request Body**:
```json
{
  "linkTitle": "Updated Tenant Verification",
  "description": "Updated description for tenant screening",
  "status": "active",
  "destination": "https://property-mgmt.com/tenants/verified-updated",
  "clientBranding": {
    "companyName": "Premier Property Management LLC",
    "logo": "https://property-mgmt.com/new-logo.png",
    "primaryColor": "#1e40af"
  },
  "webhookUrl": "https://api.property-mgmt.com/webhooks/verification-v2",
  "webhookSecret": "new-secret-key",
  "webhookEnabled": true,
  "urlParameters": [
    {
      "key": "source",
      "value": "tenant-portal-v2"
    }
  ]
}
```

**Request Parameters**:
- All fields are optional - only provided fields will be updated
- `linkId` (string, optional) - New link identifier (renames the link)
- `linkTitle` (string, optional) - Updated title
- `description` (string, optional) - Updated description
- `status` (string, optional) - Updated status
- `destination` (string, optional) - Updated destination URL
- `clientBranding` (object, optional) - Updated branding
- `styling` (object, optional) - Updated styling
- `webhookUrl` (string, optional) - Updated webhook URL
- `webhookSecret` (string, optional) - Updated webhook secret (only if changed)
- `webhookEnabled` (boolean, optional) - Enable/disable webhook
- `urlParameters` (array, optional) - Updated URL parameters

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "_id": "507f1f77bcf86cd799439100",
    "linkId": "tenant-screening",
    "linkTitle": "Updated Tenant Verification",
    "description": "Updated description for tenant screening",
    "status": "active",
    "updatedAt": "2025-10-19T14:30:00Z"
  },
  "message": "Verification link updated successfully"
}
```

**Error Responses**:

404 Not Found:
```json
{
  "success": false,
  "message": "Verification link not found or you do not have permission to update it"
}
```

409 Conflict - Duplicate Link ID:
```json
{
  "success": false,
  "message": "A verification link with this ID already exists"
}
```

**Update Behaviour**:
- **Webhook Config**: Preserves existing values, updates only what's provided
- **URL Parameters**: Completely replaces array (provide full array)
- **Branding**: Merges with existing configuration
- **Empty Webhook Secret**: Ignored (preserves existing secret for security)

**Notes**:
- Existing webhook secret preserved if not explicitly updated
- Can rename link by providing new `linkId`
- Organisation ownership verified before update
- All changes logged in audit trail

---

#### 5. Delete Verification Link

Delete a verification link (hard delete).

**Endpoint**: `POST /api/v1/merchant/verification-links/:linkId/delete`

**URL Parameters**:
- `linkId` (string, required) - Link identifier to delete

**Success Response** (200):
```json
{
  "success": true,
  "message": "Verification link deleted successfully"
}
```

**Error Response** (404):
```json
{
  "success": false,
  "message": "Verification link not found"
}
```

**Notes**:
- Hard delete - permanently removes the link
- Cannot be undone
- Existing verification submissions are preserved
- Link URL becomes invalid immediately
- Consider setting status to `inactive` instead for soft deletion

---

#### 6. Get Link Submissions

Retrieve all verification submissions for a specific link.

**Endpoint**: `GET /api/v1/merchant/verification-links/submissions/:linkObjectId`

**URL Parameters**:
- `linkObjectId` (string, required) - Verification link MongoDB ObjectId (not linkId)

**Query Parameters**:
- `status` (string, optional) - Filter by status: `pending`, `cleared`, `rejected`, `all` (default: all)
- `limit` (number, optional) - Number of results per page (default: 50, max: 100)
- `offset` (number, optional) - Pagination offset (default: 0)

**Success Response** (200):
```json
{
  "success": true,
  "data": [
    {
      "_id": "507f1f77bcf86cd799439110",
      "verificationLinkId": "507f1f77bcf86cd799439100",
      "sessionId": "SESSION-abc123",
      "status": "cleared",
      "verificationType": "identity",
      "referenceNumber": "REF-A1B2C3",
      "verificationToken": "token_xyz123",
      "customerInfo": {
        "firstName": "John",
        "lastName": "Smith",
        "email": "john@example.com"
      },
      "verificationResultId": "507f1f77bcf86cd799439011",
      "submittedAt": "2025-10-19T11:00:00Z",
      "clearedAt": "2025-10-19T14:00:00Z",
      "webhookSent": true,
      "webhookSentAt": "2025-10-19T14:00:30Z",
      "clientId": "507f1f77bcf86cd799439030",
      "organisationId": "507f1f77bcf86cd799439031",
      "createdAt": "2025-10-19T11:00:00Z"
    },
    {
      "_id": "507f1f77bcf86cd799439111",
      "verificationLinkId": "507f1f77bcf86cd799439100",
      "sessionId": "SESSION-def456",
      "status": "pending",
      "verificationType": "identity",
      "referenceNumber": "REF-D4E5F6",
      "customerInfo": {
        "firstName": "Jane",
        "lastName": "Doe",
        "email": "jane@example.com"
      },
      "submittedAt": "2025-10-19T12:00:00Z",
      "createdAt": "2025-10-19T12:00:00Z"
    }
  ],
  "pagination": {
    "total": 145,
    "limit": 50,
    "offset": 0,
    "hasMore": true
  }
}
```

**Submission Status Values**:
- `pending` - Submitted, awaiting review
- `processing` - Under operations review
- `cleared` - Verification approved
- `rejected` - Verification rejected
- `expired` - Session expired before completion

**Response Fields**:
- `verificationResultId` - Links to the actual identity/address verification result
- `referenceNumber` - Customer-facing reference code
- `verificationToken` - Token for retrieving results
- `webhookSent` - Whether webhook was successfully delivered
- `customerInfo` - Basic customer information

**Notes**:
- Use link's MongoDB ObjectId (not linkId) as parameter
- Tracks all verifications completed via this link
- Includes webhook delivery status
- Sorted by submission date (newest first)

---

### Public Verification Link Access

#### 7. Get Public Verification Link

Get verification link details for public access (used by verification flow).

**Endpoint**: `GET /api/v1/public/verification-links/:linkId`

**URL Parameters**:
- `linkId` (string, required) - The verification link identifier

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "linkId": "employee-verification",
    "linkTitle": "Employee Verification",
    "description": "Identity verification for new employees",
    "status": "active",
    "verificationType": "identity",
    "clientBranding": {
      "companyName": "Acme Corporation",
      "logo": "https://example.com/logo.png",
      "primaryColor": "#007bff",
      "accentColor": "#0056b3"
    },
    "styling": {
      "theme": "light",
      "buttonStyle": "rounded",
      "fontFamily": "Inter, sans-serif"
    },
    "destination": "https://company.com/onboarding/complete",
    "clientId": "507f1f77bcf86cd799439030",
    "organisationId": "507f1f77bcf86cd799439031",
    "logoUrl": "https://cleared-public.s3.us-east-1.amazonaws.com/logos/acme-logo.png",
    "companyName": "Acme Corporation",
    "_id": "507f1f77bcf86cd799439100",
    "createdAt": "2025-10-15T10:00:00Z",
    "urlParameters": [
      {
        "key": "source",
        "value": "employee-onboarding"
      }
    ]
  }
}
```

**Error Responses**:

400 Bad Request:
```json
{
  "message": "Link ID is required"
}
```

404 Not Found:
```json
{
  "success": false,
  "message": "Verification link not found or inactive"
}
```

**Notes**:
- No authentication required (public endpoint)
- Only returns active links
- Webhook configuration excluded for security
- Fetches organisation logo from database
- Used by verification flow UI to display branding
- Company name fetched from organisation if available

---

## Data Models

### Verification Link Object

```json
{
  "_id": "507f1f77bcf86cd799439100",
  "linkId": "employee-verification",
  "linkTitle": "Employee Verification",
  "description": "Identity verification for new employees",
  "status": "active",
  "verificationType": "identity",
  "destination": "https://company.com/onboarding/complete",
  "clientBranding": {
    "companyName": "Acme Corporation",
    "logo": "https://example.com/logo.png",
    "primaryColor": "#007bff",
    "accentColor": "#0056b3"
  },
  "styling": {
    "theme": "light",
    "buttonStyle": "rounded",
    "fontFamily": "Inter, sans-serif"
  },
  "webhookConfig": {
    "url": "https://api.company.com/webhooks/verification",
    "secret": "your-webhook-secret-key",
    "active": true
  },
  "urlParameters": [
    {
      "key": "orderId",
      "value": "ORD-123"
    }
  ],
  "clientId": "507f1f77bcf86cd799439030",
  "organisationId": "507f1f77bcf86cd799439031",
  "createdBy": "507f1f77bcf86cd799439032",
  "createdAt": "2025-10-15T10:00:00Z",
  "updatedAt": "2025-10-18T14:30:00Z"
}
```

### Verification Session Object

```json
{
  "_id": "507f1f77bcf86cd799439110",
  "verificationLinkId": "507f1f77bcf86cd799439100",
  "sessionId": "SESSION-abc123",
  "status": "cleared",
  "verificationType": "identity",
  "referenceNumber": "REF-A1B2C3",
  "verificationToken": "token_xyz123",
  "customerInfo": {
    "firstName": "John",
    "lastName": "Smith",
    "email": "john@example.com",
    "phone": "+1876xxxxxxx"
  },
  "verificationResultId": "507f1f77bcf86cd799439011",
  "submittedAt": "2025-10-19T11:00:00Z",
  "clearedAt": "2025-10-19T14:00:00Z",
  "webhookSent": true,
  "webhookSentAt": "2025-10-19T14:00:30Z",
  "webhookStatus": "success",
  "clientId": "507f1f77bcf86cd799439030",
  "organisationId": "507f1f77bcf86cd799439031",
  "createdAt": "2025-10-19T11:00:00Z",
  "updatedAt": "2025-10-19T14:00:30Z"
}
```

## Complete Workflow Example

### Creating and Using a Verification Link

```javascript
// 1. Create verification link
const createResponse = await fetch('/api/v1/merchant/verification-links/create', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${apiKey}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    linkId: 'employee-onboarding',
    linkTitle: 'Employee Verification',
    description: 'Complete identity verification for employment',
    verificationType: 'identity',
    destination: 'https://company.com/hr/verification-complete',
    clientBranding: {
      companyName: 'Acme Corporation',
      logo: 'https://company.com/logo.png',
      primaryColor: '#007bff'
    },
    webhookUrl: 'https://api.company.com/webhooks/verification',
    webhookSecret: 'secret-key-xyz123',
    webhookEnabled: true,
    urlParameters: [
      { key: 'employeeId', value: 'EMP-456' },
      { key: 'department', value: 'engineering' }
    ]
  })
});

const { data: link } = await createResponse.json();
console.log('Verification URL:', `https://cleared.id/v/${link.linkId}`);

// 2. Share link with customer
const verificationUrl = `https://cleared.id/v/${link.linkId}`;
// Send via email, SMS, or embed in website

// 3. Customer completes verification
// Customer clicks link, uploads ID, completes verification

// 4. Receive webhook notification
// POST https://api.company.com/webhooks/verification
// {
//   "trn": "123-456-789",
//   "firstName": "John",
//   "lastName": "Smith",
//   "event": "cleared",
//   "referenceNumber": "REF-ABC123",
//   "verificationToken": "token_xyz"
// }

// 5. Retrieve full verification details (using verification token)
const verificationResponse = await fetch(
  `/api/v1/merchant/identity/verify-reference`,
  {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${apiKey}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      referenceNumber: 'REF-ABC123'
    })
  }
);

// 6. Monitor submissions
const submissionsResponse = await fetch(
  `/api/v1/merchant/verification-links/submissions/${link._id}?status=all`,
  {
    headers: { 'Authorization': `Bearer ${apiKey}` }
  }
);

const { data: submissions } = await submissionsResponse.json();
console.log(`Total submissions: ${submissions.length}`);
```

## Webhook Implementation

### Validating Webhook Signatures

```javascript
const crypto = require('crypto');

function validateWebhookSignature(payload, signature, secret) {
  const hmac = crypto.createHmac('sha256', secret);
  const expectedSignature = hmac.update(JSON.stringify(payload)).digest('hex');
  
  return signature === `sha256=${expectedSignature}`;
}

// Express webhook handler
app.post('/webhooks/verification', (req, res) => {
  const signature = req.headers['x-webhook-signature'];
  const payload = req.body;
  const secret = process.env.WEBHOOK_SECRET;
  
  if (!validateWebhookSignature(payload, signature, secret)) {
    return res.status(401).json({ error: 'Invalid signature' });
  }
  
  // Process webhook
  const { event, referenceNumber, verificationToken } = payload;
  
  if (event === 'cleared') {
    // Handle successful verification
    console.log('Verification cleared:', referenceNumber);
    // Store verification token for later retrieval
    // Update your system with customer details
  } else if (event === 'rejected') {
    // Handle rejected verification
    console.log('Verification rejected:', referenceNumber);
  }
  
  res.json({ success: true });
});
```

### Webhook Retry Logic

If webhook delivery fails:
1. **Immediate retry** - Retry after 5 seconds
2. **Second retry** - Retry after 1 minute
3. **Third retry** - Retry after 5 minutes
4. **Mark as failed** - Log webhook failure

Monitor webhook delivery status via submissions endpoint.

## Common Use Cases

### Use Case 1: Employee Onboarding

```javascript
// Create link for employee verification
const link = await createVerificationLink({
  linkId: 'employee-onboarding',
  linkTitle: 'Employee Identity Verification',
  description: 'Complete your identity verification to start employment',
  verificationType: 'identity',
  destination: 'https://hr-portal.com/onboarding/step2',
  webhookUrl: 'https://api.hr-portal.com/webhooks/identity-verified',
  urlParameters: [
    { key: 'step', value: 'identity-complete' }
  ]
});

// Email to new employee
const email = `
  Welcome to Acme Corporation!
  
  Please complete your identity verification:
  https://cleared.id/v/employee-onboarding
  
  This should take about 2 minutes.
`;
```

### Use Case 2: Tenant Screening

```javascript
// Create link for tenant verification
const link = await createVerificationLink({
  linkId: 'tenant-screening',
  linkTitle: 'Tenant Verification',
  description: 'Identity and address verification for rental application',
  verificationType: 'combined',
  destination: 'https://property-mgmt.com/applications/verified',
  urlParameters: [
    { key: 'applicationId', value: 'APP-789' },
    { key: 'propertyId', value: 'PROP-456' }
  ]
});

// Include in rental application
const applicationLink = `https://cleared.id/v/tenant-screening`;
```

### Use Case 3: KYC for Financial Services

```javascript
// Create link for KYC verification
const link = await createVerificationLink({
  linkId: 'customer-kyc',
  linkTitle: 'Customer Verification',
  description: 'Identity verification for account opening',
  verificationType: 'identity',
  destination: 'https://bank.com/accounts/kyc-complete',
  clientBranding: {
    companyName: 'Acme Bank',
    logo: 'https://bank.com/logo.png',
    primaryColor: '#004d40'
  },
  webhookUrl: 'https://api.bank.com/webhooks/kyc',
  webhookSecret: 'bank-secret-xyz',
  webhookEnabled: true
});
```

### Use Case 4: Vendor Onboarding

```javascript
// Create link for vendor verification
const link = await createVerificationLink({
  linkId: 'vendor-verification',
  linkTitle: 'Vendor Verification',
  description: 'Company verification for vendor registration',
  verificationType: 'company',
  destination: 'https://procurement.com/vendors/verified',
  urlParameters: [
    { key: 'vendorId', value: 'VND-123' }
  ]
});
```

## Best Practices

### 1. Link Management

- **Use Descriptive Link IDs** - Choose clear, memorable identifiers
- **One Link Per Purpose** - Create separate links for different use cases
- **Monitor Inactive Links** - Disable unused links to prevent misuse
- **Regular Audits** - Review links quarterly, remove obsolete ones

### 2. Branding

- **Consistent Branding** - Match your company's visual identity
- **High-Quality Logos** - Use PNG with transparent background
- **Accessible Colors** - Ensure sufficient contrast for readability
- **Test Appearance** - Preview link before sharing with customers

### 3. Webhook Configuration

- **Secure Secrets** - Use strong, random webhook secrets
- **HTTPS Only** - Always use HTTPS for webhook URLs
- **Validate Signatures** - Always verify webhook signatures
- **Idempotency** - Handle duplicate webhooks gracefully
- **Error Handling** - Implement retry logic for failed webhooks

### 4. URL Parameters

- **Meaningful Keys** - Use descriptive parameter names
- **Consistent Values** - Maintain naming conventions
- **Track Context** - Pass order IDs, customer IDs, campaign codes
- **Limit Parameters** - Keep to 3-5 parameters maximum

### 5. Security

- **Monitor Submissions** - Regularly check for suspicious activity
- **Rate Limiting** - Implement rate limiting on your webhook endpoint
- **Access Control** - Verify verification tokens before granting access
- **Data Privacy** - Handle customer data according to regulations

## Error Codes

| Status Code | Description |
|------------|-------------|
| 200 | Success |
| 400 | Bad Request (invalid parameters) |
| 401 | Unauthorised (invalid/missing token) |
| 404 | Not Found (link not found) |
| 409 | Conflict (duplicate link ID) |
| 500 | Internal Server Error |

## Rate Limits

- **Link Creation**: 10 per hour per organisation
- **Link Updates**: 50 per hour per organisation
- **Link Retrieval**: 1000 per hour per organisation
- **Submission Retrieval**: 500 per hour per organisation

## Related Documentation

- [Verification Links API Overview](./verification-links-api.md)
- [Identity Verification API](../identity/identity-api.md)
- [Address Verification API](../address/address-api.md)
- [API Overview](../README.md)

---

← [Verification Links API Overview](./verification-links-api.md)

