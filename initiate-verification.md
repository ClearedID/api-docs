# Initiate Verification Request API

## Overview

The Initiate Verification API allows merchants to programmatically request identity, address, employment, income, and other verifications from customers. This powerful endpoint handles customer lookup, user creation, request configuration, and notification delivery in a single API call.

**Endpoint**: `POST /api/v1/merchant/:service/verification/initiate`

**Authentication**: Merchant JWT token in Authorization header

## Key Features

- **Multi-Service Support** – Request 8 different verification types
- **Automatic Customer Lookup** – Finds existing customers by TRN, email, or phone
- **User Auto-Creation** – Creates new user accounts automatically
- **Flexible Contact Methods** – Email, SMS, or in-person verification
- **Request Bundling** – Request multiple verification types together
- **Expiration Control** – Set custom expiration times (1 hour to 3 days)
- **Cost Management** – Specify cost allocation and splits
- **Quiet Mode** – Silent request creation without notifications
- **In-Person Mode** – Generate tokens for merchant-assisted verification

## Supported Verification Services

| Service | Description | Typical Cost | Common Use Cases |
|---------|-------------|--------------|------------------|
| `identity` | ID document + biometric verification | 5 credits | KYC, onboarding, account opening |
| `address` | Proof of residential/business address | 3 credits | Compliance, loan applications |
| `income` | Income and employment verification | 10 credits | Lending, rental applications |
| `employment` | Employment history verification | 8 credits | Background checks, hiring |
| `qualification` | Education/professional certifications | 7 credits | Professional licensing |
| `reference` | Personal/professional references | 5 credits | Rental, employment screening |
| `company` | Business registration and verification | 7 credits | B2B onboarding, vendor management |
| `background` | Criminal records, credit history | 15+ credits | Employee screening, tenant checks |

## Endpoint Details

### Initiate Verification Request

Create a verification request for a customer.

**Endpoint**: `POST /api/v1/merchant/:service/verification/initiate`

**URL Parameters**:
- `service` (string, required) - Service type: `identity`, `address`, `income`, `employment`, `qualification`, `reference`, `company`, or `background`

**Request Body**:
```json
{
  "name": "John Michael Smith",
  "emailAddress": "john@example.com",
  "phoneNumber": "+18761234567",
  "trn": "123-456-789",
  "verificationRequests": [
    {
      "type": "identity",
      "required": true,
      "description": "Government-issued ID verification"
    },
    {
      "type": "address",
      "required": true,
      "description": "Proof of address verification"
    }
  ],
  "summary": "Identity and address verification for employment",
  "backgroundCheckCostSplit": null,
  "expiration": {
    "expiresAt": "2025-10-22T10:00:00Z"
  },
  "totalCost": 10,
  "quiet": false,
  "mode": "full"
}
```

### Request Parameters

#### Required Parameters

- **`name`** (string, required)
  - Customer's full name
  - Used for display and matching
  - Example: `"John Michael Smith"`

- **`verificationRequests`** (array, required)
  - Array of verification types to request
  - At least one verification required
  - Each verification object contains:
    - `type` (string) - Verification type
    - `required` (boolean) - Whether verification is mandatory
    - `description` (string, optional) - Description shown to customer

#### Contact Information (At least one required)

- **`emailAddress`** (string, conditional)
  - Customer's email address
  - Required if `phoneNumber` not provided
  - Used for notifications and customer lookup
  - Example: `"john@example.com"`

- **`phoneNumber`** (string, conditional)
  - Customer's phone number
  - Required if `emailAddress` not provided
  - International format recommended
  - Example: `"+18761234567"`

- **`trn`** (string, optional)
  - Tax Registration Number
  - Used for customer lookup
  - Example: `"123-456-789"`

#### Optional Parameters

- **`summary`** (string, optional)
  - Brief description of verification purpose
  - Shown to customer in notification
  - Example: `"Identity verification for employment"`

- **`expiration`** (object, optional)
  - Expiration configuration
  - `expiresAt` (string) - ISO 8601 date/time
  - Default: 24 hours from creation
  - Minimum: 1 hour from now
  - Maximum: 3 days from now
  - Example: `{ "expiresAt": "2025-10-22T10:00:00Z" }`

- **`backgroundCheckCostSplit`** (object, optional)
  - Cost allocation for background checks
  - Specifies merchant vs customer payment split
  - Example: `{ "merchant": 50, "customer": 50 }`

- **`totalCost`** (number, optional)
  - Total credit cost for verification
  - Used for quota tracking
  - Example: `10`

- **`quiet`** (boolean, optional)
  - If `true`, suppresses notification emails/SMS
  - Default: `false`
  - Use for batch processing or custom notifications
  - Example: `true`

- **`mode`** (string, optional)
  - Verification mode: `"full"` or `"in-person"`
  - Default: `"full"`
  - **`full`**: Remote verification via customer's device
  - **`in-person`**: Merchant-assisted verification with guest session
  - Example: `"in-person"`

### Success Response (201)

```json
{
  "message": "Verification requests initiated successfully",
  "success": true,
  "resultId": "result_id",
  "requestId": "request_id",
  "token": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Response Fields**:
- `success` - Boolean indicating success
- `resultId` - ID of the verification result object
- `requestId` - ID of the verification request
- `token` - Guest session token (only present in `in-person` mode)
- `message` - Success message

### Error Responses

**400 Bad Request - No Contact Information**:
```json
{
  "error": "User not found and no contact information provided"
}
```

**400 Bad Request - Invalid Request Format**:
```json
{
  "error": "Invalid request format"
}
```

**400 Bad Request - Invalid Expiration**:
```json
{
  "error": "Expiration must be at least 1 hour from now"
}
```

**400 Bad Request - Invalid Expiration Date**:
```json
{
  "error": "Invalid expiration date format"
}
```

**403 Forbidden - Insufficient Privileges**:
```json
{
  "notification": {
    "type": "error",
    "title": "Access Denied",
    "text": "Based on the privileges assigned to you in this organisation, you cannot create verification requests."
  }
}
```

**403 Forbidden - Quota Exceeded**:
```json
{
  "notification": {
    "type": "error",
    "title": "Quota Exceeded",
    "text": "Your organisation has reached its quota limit for this service."
  }
}
```

## Customer Lookup Logic

The system employs sophisticated customer lookup logic to match verification requests with existing users:

### Lookup Priority

#### 1. TRN Lookup (if provided)

The system attempts to find users by TRN in this order:

1. **Highest Priority**: TRN verified + ID verified + Active status
2. **Second Priority**: TRN verified + Active status
3. **Third Priority**: TRN match + Active status
4. **Fourth Priority**: Any TRN match (any status)

```javascript
// Pseudo-code for TRN lookup
if (trn) {
  user = await User.findOne({ 
    uniqueId: trn, 
    uniqueIdVerified: true, 
    idVerified: true, 
    status: 'active' 
  });
  
  if (!user) {
    user = await User.findOne({ 
      uniqueId: trn, 
      uniqueIdVerified: true, 
      status: 'active' 
    });
  }
  
  // ... and so on
}
```

#### 2. Email Lookup (if TRN not found)

If no user found by TRN, searches by email:

1. **Primary Email**: Active users with matching primary email
2. **Secondary Emails**: Active users with email in secondary emails array
3. **Primary Email (Any Status)**: Users with matching primary email (except replaced/inactive)
4. **Secondary Emails (Any Status)**: Users with email in secondary emails (except replaced/inactive)

```javascript
// Pseudo-code for email lookup
if (emailAddress && !user) {
  user = await User.findOne({ 
    emailAddress, 
    status: 'active' 
  });
  
  if (!user) {
    user = await User.findOne({ 
      emailAddresses: emailAddress, 
      status: 'active' 
    });
  }
  
  // ... and so on
}
```

#### 3. Create New User (if no match)

If no existing user found:
- Creates new user account
- Sets provided contact information
- Marks as new user for special handling
- Triggers onboarding workflow

```javascript
if (!user) {
  user = new User({ 
    uniqueId: trn, 
    emailAddress, 
    phoneNumber 
  });
  await user.save();
  isNewUser = true;
}
```

## Expiration Management

### Default Expiration

If no expiration specified:
- **Default**: 24 hours from request creation
- Provides reasonable window for customer completion

### Custom Expiration

Configure custom expiration time:

```json
{
  "expiration": {
    "expiresAt": "2025-10-22T17:00:00Z"
  }
}
```

**Constraints**:
- **Minimum**: 1 hour from current time
- **Maximum**: 3 days from current time
- Out-of-range values automatically adjusted to min/max

**Examples**:

```javascript
// 48 hours from now
const expiresAt = new Date(Date.now() + 48 * 60 * 60 * 1000);

// Specific date/time
const expiresAt = new Date('2025-10-22T17:00:00Z');

// Request with expiration
const request = {
  name: "John Smith",
  emailAddress: "john@example.com",
  verificationRequests: [...],
  expiration: {
    expiresAt: expiresAt.toISOString()
  }
};
```

**Expiration Handling**:
- System validates expiration is in valid range
- Too short: Adjusted to minimum (1 hour)
- Too long: Adjusted to maximum (3 days)
- Invalid format: Request rejected with error

## Verification Modes

### Full Mode (Remote Verification)

**Default mode** - Customer completes verification remotely on their device.

```json
{
  "mode": "full",
  "name": "John Smith",
  "emailAddress": "john@example.com",
  "verificationRequests": [...]
}
```

**Workflow**:
1. System creates verification request
2. Sends email/SMS to customer with link
3. Customer clicks link and opens on their device
4. Customer uploads documents and completes verification
5. Results returned to merchant via webhook/update

**Use Cases**:
- Remote customer onboarding
- Self-service verification
- Asynchronous workflows
- Large-scale batch processing

### In-Person Mode (Merchant-Assisted)

**Specialized mode** - Merchant assists customer with verification on merchant's device.

```json
{
  "mode": "in-person",
  "name": "Jane Doe",
  "emailAddress": "jane@example.com",
  "verificationRequests": [...]
}
```

**Response Includes Token**:
```json
{
  "success": true,
  "requestId": "request_id",
  "resultId": "result_id",
  "token": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Workflow**:
1. System creates verification request
2. Creates guest session with secure token
3. Returns token to merchant
4. Merchant uses token to access verification UI
5. Customer completes verification on merchant's device
6. Results available immediately

**Token Usage**:
```javascript
// Open verification UI with token
const verificationUrl = `https://cleared.id/verify?token=${token}`;
window.open(verificationUrl, '_blank');
```

**Use Cases**:
- In-branch verification
- Point-of-sale scenarios
- Customer without smartphone
- Immediate verification needed
- Assisted onboarding

**Guest Session**:
- One-time use token
- Linked to specific user and request
- Expires with request
- Secure, non-reusable
- Tracks current verification step

## Request Examples

### Example 1: Basic Identity Verification

```json
{
  "name": "John Smith",
  "emailAddress": "john@example.com",
  "verificationRequests": [
    {
      "type": "identity",
      "required": true,
      "description": "Government-issued ID verification"
    }
  ],
  "summary": "Identity verification for account opening"
}
```

### Example 2: Multi-Service KYC

```json
{
  "name": "Jane Doe",
  "emailAddress": "jane@example.com",
  "phoneNumber": "+18767654321",
  "trn": "456-789-123",
  "verificationRequests": [
    {
      "type": "identity",
      "required": true,
      "description": "Government-issued ID with photo"
    },
    {
      "type": "address",
      "required": true,
      "description": "Proof of residential address"
    },
    {
      "type": "employment",
      "required": false,
      "description": "Employment history verification"
    }
  ],
  "summary": "Comprehensive KYC for mortgage application",
  "expiration": {
    "expiresAt": "2025-10-22T17:00:00Z"
  },
  "totalCost": 15
}
```

### Example 3: Background Check with Cost Split

```json
{
  "name": "Bob Wilson",
  "emailAddress": "bob@example.com",
  "phoneNumber": "+18761234567",
  "verificationRequests": [
    {
      "type": "identity",
      "required": true
    },
    {
      "type": "address",
      "required": true
    },
    {
      "type": "background",
      "required": true,
      "description": "Criminal records and credit history"
    }
  ],
  "summary": "Pre-employment screening",
  "backgroundCheckCostSplit": {
    "merchant": 75,
    "customer": 25
  },
  "totalCost": 20
}
```

### Example 4: In-Person Verification

```json
{
  "name": "Alice Johnson",
  "emailAddress": "alice@example.com",
  "verificationRequests": [
    {
      "type": "identity",
      "required": true
    }
  ],
  "mode": "in-person"
}
```

### Example 5: Silent Request (No Notifications)

```json
{
  "name": "Charlie Brown",
  "emailAddress": "charlie@example.com",
  "verificationRequests": [
    {
      "type": "identity",
      "required": true
    }
  ],
  "quiet": true
}
```

### Example 6: Batch Processing

```javascript
// Batch create verification requests
const customers = [
  { name: "John Smith", email: "john@example.com" },
  { name: "Jane Doe", email: "jane@example.com" },
  { name: "Bob Wilson", email: "bob@example.com" }
];

const results = await Promise.all(
  customers.map(customer => 
    fetch('/api/v1/merchant/identity/verification/initiate', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${apiKey}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        name: customer.name,
        emailAddress: customer.email,
        verificationRequests: [
          { type: 'identity', required: true }
        ],
        quiet: true // Don't send notifications for batch
      })
    })
  )
);

console.log(`Created ${results.length} verification requests`);
```

## Complete Integration Example

### Full Workflow Implementation

```javascript
const initiateVerification = async (customerData) => {
  try {
    // 1. Prepare verification request
    const verificationData = {
      name: customerData.fullName,
      emailAddress: customerData.email,
      phoneNumber: customerData.phone,
      trn: customerData.trn, // Optional
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
      summary: 'KYC verification for loan application',
      expiration: {
        expiresAt: new Date(Date.now() + 48 * 60 * 60 * 1000).toISOString()
      },
      totalCost: 8,
      quiet: false,
      mode: 'full'
    };
    
    // 2. Initiate verification request
    const response = await fetch(
      '/api/v1/merchant/identity/verification/initiate',
      {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${apiKey}`,
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(verificationData)
      }
    );
    
    if (!response.ok) {
      const error = await response.json();
      throw new Error(error.error || error.message);
    }
    
    const result = await response.json();
    
    console.log('Verification initiated:', {
      requestId: result.requestId,
      resultId: result.resultId
    });
    
    // 3. Store request ID for tracking
    await storeRequestId(customerData.customerId, result.requestId);
    
    // 4. Monitor request status
    await monitorRequest(result.requestId);
    
    return result;
    
  } catch (error) {
    console.error('Failed to initiate verification:', error);
    throw error;
  }
};

// Monitor request status
const monitorRequest = async (requestId) => {
  const checkStatus = async () => {
    const response = await fetch(
      `/api/v1/merchant/verifications/requests/${requestId}/details`,
      {
        headers: { 'Authorization': `Bearer ${apiKey}` }
      }
    );
    
    const request = await response.json();
    
    console.log(`Request status: ${request.status}`);
    
    if (request.status === 'approved') {
      console.log('Verification approved!');
      // Retrieve verification results
      await processApprovedVerification(request);
      return true;
    } else if (request.status === 'denied') {
      console.log('Verification denied');
      return true;
    }
    
    return false;
  };
  
  // Poll every 30 seconds
  const intervalId = setInterval(async () => {
    const completed = await checkStatus();
    if (completed) {
      clearInterval(intervalId);
    }
  }, 30000);
};

// In-person verification
const initiateInPersonVerification = async (customerData) => {
  const response = await fetch(
    '/api/v1/merchant/identity/verification/initiate',
    {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${apiKey}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        name: customerData.fullName,
        emailAddress: customerData.email,
        verificationRequests: [
          { type: 'identity', required: true }
        ],
        mode: 'in-person'
      })
    }
  );
  
  const result = await response.json();
  
  // Open verification UI in new window
  const verificationUrl = `https://cleared.id/verify?token=${result.token}`;
  window.open(verificationUrl, '_blank');
  
  return result;
};
```

## Process Flow

### Backend Process

When you initiate a verification request, the system:

1. **Validates Input**
   - Checks required fields present
   - Validates email/phone format
   - Validates expiration range
   - Checks service quota availability

2. **Customer Lookup**
   - Searches by TRN (if provided)
   - Searches by email (if not found by TRN)
   - Creates new user if no match found

3. **Request Creation**
   - Creates verification request record
   - Generates unique request ID
   - Creates verification result placeholders
   - Sets expiration timestamp

4. **Mode-Specific Setup**
   - **Full Mode**: Schedules notification emails/SMS
   - **In-Person Mode**: Creates guest session with token

5. **Notification** (if not quiet)
   - Sends email with verification link
   - Sends SMS with verification link
   - Includes summary and expiration time

6. **Action Items**
   - Creates customer action items
   - Adds to customer's todo list
   - Sets priority and deadlines

7. **Audit Logging**
   - Logs request creation
   - Records initiator details
   - Tracks organisation and client

8. **Credit Deduction**
   - Calculates total cost
   - Deducts from organisation credits
   - Records transaction

9. **Contact Logging**
   - Logs verification request sent
   - Tracks initiator and recipient
   - Records verification types

### Customer Experience

For customers receiving verification requests:

**Email Notification**:
```
Subject: Verification Request from Acme Corporation

Hello John Smith,

Acme Corporation has requested verification of your information.

Verification Types Required:
- Identity Verification
- Address Verification

Please complete your verification within 48 hours.

[Complete Verification] button

This link expires on October 22, 2025 at 5:00 PM.
```

**Verification Flow**:
1. Customer clicks link in email/SMS
2. Opens Cleared verification portal
3. Sees list of required verifications
4. Completes each verification step
5. Uploads required documents
6. Submits for review
7. Receives confirmation email
8. Waits for operations review (if needed)
9. Approves/denies sharing with merchant
10. Merchant receives result notification

## Best Practices

### 1. Customer Identification

- **Always Provide TRN** - If known, reduces duplicate accounts
- **Use Consistent Email** - Helps match existing customers
- **International Phone Format** - Use +country format
- **Full Names** - Include middle names when known
- **Validate Before Sending** - Check email/phone validity

### 2. Request Configuration

- **Appropriate Verifications** - Only request what's needed
- **Clear Descriptions** - Explain why each verification is needed
- **Reasonable Expiration** - Give customers adequate time
- **Cost Transparency** - Be clear about cost splits
- **Bundle Related Verifications** - Request together when related

### 3. Notification Strategy

- **Default to Notifications** - Only use quiet mode when appropriate
- **Custom Summaries** - Provide context in summary field
- **Follow-up** - Monitor pending requests and send reminders
- **Multi-Channel** - Provide both email and phone when possible

### 4. Error Handling

- **Catch Quota Errors** - Handle gracefully, inform customer
- **Validate Input** - Client-side validation before API call
- **Retry Logic** - Implement retry for transient failures
- **User Feedback** - Show clear error messages to users
- **Log Failures** - Track failed requests for debugging

### 5. In-Person Mode

- **Secure Token Handling** - Don't log or expose tokens
- **Session Management** - Token valid only for current session
- **Customer Privacy** - Ensure customer can see screen
- **Document Quality** - Guide customer for clear photos
- **Immediate Feedback** - Validate uploads before submission

### 6. Batch Processing

- **Use Quiet Mode** - Suppress individual notifications
- **Rate Limiting** - Respect API rate limits
- **Error Tracking** - Log failed requests for retry
- **Progress Monitoring** - Track batch completion status
- **Custom Notifications** - Send your own batch summary email

## Privileges Required

To initiate verification requests, the authenticated user must have:

**Service-Specific Privilege**:
- `create_identity_verification_request` - For identity verifications
- `create_address_verification_request` - For address verifications
- `create_income_verification_request` - For income verifications
- `create_employment_verification_request` - For employment verifications
- `create_qualification_verification_request` - For qualification verifications
- `create_reference_verification_request` - For reference verifications
- `create_company_verification_request` - For company verifications
- `create_background_verification_request` - For background checks

**Privilege Format**: `create_{service}_verification_request`

**Checking Privileges**:
```javascript
// System checks privilege before processing
const hasPrivilege = await checkPrivilege(
  `create_${service}_verification_request`,
  req.userId,
  req.organisationId
);

if (!hasPrivilege) {
  return res.status(403).json({
    error: 'Insufficient privileges'
  });
}
```

## Quota Management

### Service Quotas

Each organisation has quotas for verification services:

- Quotas set per service type
- Replenished monthly or as purchased
- Enforced if `ENFORCE_SERVICE_QUOTA_LIMITS` enabled
- Quota check happens before request creation

### Quota Check

```javascript
// System checks quota before processing
const quotaAvailable = await checkServiceQuota(
  organisationId,
  verificationType
);

if (!quotaAvailable && config.ENFORCE_SERVICE_QUOTA_LIMITS) {
  return res.status(403).json({
    notification: {
      type: 'error',
      title: 'Quota Exceeded',
      text: 'Your organisation has reached its quota limit.'
    }
  });
}
```

### Cost Calculation

```json
{
  "verificationRequests": [
    { "type": "identity", "cost": 5 },
    { "type": "address", "cost": 3 },
    { "type": "income", "cost": 10 }
  ],
  "totalCost": 18
}
```

## Rate Limits

- **Initiate Verification**: 100 requests per hour per organisation
- **In-Person Mode**: 50 requests per hour per organisation
- **Batch Processing**: 500 requests per hour per organisation

## Error Codes

| Status Code | Description |
|------------|-------------|
| 201 | Created successfully |
| 400 | Bad Request (validation error, invalid format) |
| 401 | Unauthorised (invalid/missing token) |
| 403 | Forbidden (insufficient privileges, quota exceeded) |
| 500 | Internal Server Error |

## Related Documentation

- [Verification Requests Management API](./verification-requests-management.md)
- [Merchant Updates API](./merchant-updates.md)
- [Identity Verification API](./identity/identity-api.md)
- [Address Verification API](./address/address-api.md)
- [Verification Links API](./verification-links/verification-links-api.md)
- [API Overview](./README.md)

