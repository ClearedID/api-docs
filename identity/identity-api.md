# Identity Verification API

## Overview

The Identity Verification API provides comprehensive identity verification services including ID document verification, biometric facial matching, liveness detection, TRN (Tax Registration Number) verification, and face authentication. This service enables businesses to verify customer identities with government-issued documents and facial biometrics.

**Base Path**: `/api/v1/merchant/identity`

**Authentication**: Merchant JWT token in Authorization header

## Key Features

- **ID Document Verification** – Passport, driver's licence, national ID card, voter ID
- **Facial Biometric Matching** – Match selfie with ID photo
- **Liveness Detection** – Prevent spoofing with live selfie verification
- **TRN Verification** – Validate Jamaican Tax Registration Numbers
- **Face Authentication** – Authenticate users with facial recognition
- **Duplicate Detection** – Identify duplicate identities across the system
- **Account Recovery** – Handle lost account and device change scenarios
- **Video Verification** – Optional video call verification for complex cases

## Verification Workflow

### Standard Identity Verification Flow

```
1. Merchant initiates verification request → Status: pending
2. Customer receives verification link → Opens verification flow
3. Customer uploads ID documents → Status: submitted
4. System processes documents → Status: processing
5. Operations team reviews → Status: flagged (if needed)
6. Decision made → Status: cleared or rejected
7. Merchant receives result → Webhook notification sent
8. Customer notified → Email confirmation
```

### Document Types Supported

**Identity Documents**:
- **Passport** – Jamaican passport with MRZ
- **Driver's Licence** – Jamaican driver's licence (contains TRN)
- **National ID Card** – Jamaican national identification card (contains TRN)
- **Voter ID** – Jamaican electoral registration card

**TRN Documents**:
- **TRN Card** – Official TRN card from Tax Administration Jamaica
- **TRN Letter** – Official TRN letter/certificate
- **Driver's Licence** – Contains embedded TRN
- **National ID Card** – Contains embedded TRN

## Verification Components

### 1. Document Verification

The system verifies ID documents using multiple checks:

**Authenticity Checks**:
- Document format validation
- Security feature detection
- Barcode/MRZ validation
- Template matching
- Forgery detection

**Data Extraction**:
- Personal information (name, DOB)
- Document details (number, issue/expiry dates)
- Address information
- TRN number (if applicable)
- Photo extraction

**Quality Assessment**:
- Image quality and clarity
- Document completeness
- Text legibility
- No alterations or tampering

### 2. Biometric Verification

**Face Matching**:
- Compare selfie with ID photo
- Calculate similarity score (0-100%)
- Match confidence level
- Return match/no-match decision

**Liveness Detection**:
- Detect live person vs photo/video
- Prevent spoofing attacks
- Validate real-time capture
- Confidence scoring

**Face Indexing**:
- Index verified faces in AWS Rekognition
- Enable duplicate detection
- Cross-reference with existing verifications
- Maintain face collection per country

### 3. TRN Verification

**Automatic TRN Extraction**:
When driver's licence or national ID is verified:
- System automatically extracts TRN
- Creates TRN verification result
- Links to identity verification
- Marks as cleared automatically

**CBS Integration**:
- Query Credit Bureau Services (CBS) database
- Validate TRN number
- Cross-check personal information
- Return official TRN holder details

**TRN Validation**:
- Format validation (XXX-XXX-XXX)
- Duplicate detection
- Active status check
- DOB cross-verification

### 4. Duplicate Detection

**Three-Layer Detection**:

**Layer 1: TRN Matching**
- Searches for users with same TRN
- Checks `uniqueId` field
- Requires `uniqueIdVerified = true`
- Prevents clearing if duplicate found

**Layer 2: Document Matching**
- Matches document type + document number
- Finds other users with same ID
- Status must be `cleared`
- Alerts to potential identity fraud

**Layer 3: Facial Matching**
- Uses AWS Rekognition face search
- Searches indexed face collection
- Returns similarity scores
- Shows matched reports with photos

## Status Values

### Verification Result Status

| Status | Description | User Action | Operations Action |
|--------|-------------|-------------|-------------------|
| `new` | Just created | Upload documents | - |
| `submitted` | Documents uploaded | Wait for review | Review submission |
| `processing` | Under operations review | Wait | Review and decide |
| `flagged` | Requires manual review | Wait | Detailed investigation |
| `cleared` | Verified and approved | Complete | - |
| `rejected` | Verification failed | Retry with better docs | - |
| `deleted` | Archived/removed | - | - |
| `expired` | Verification expired | Restart process | - |

### Verification Request Status

| Status | Description |
|--------|-------------|
| `pending` | Awaiting customer response |
| `approved` | Customer approved sharing |
| `denied` | Customer denied request |
| `awaitingClearance` | Waiting for operations clearance |

### Face Authentication Status

| Status | Description |
|--------|-------------|
| `in_progress` | Authentication in progress |
| `waiting` | Waiting for user action |
| `cleared` | Successfully authenticated |
| `rejected` | Authentication failed |
| `failed` | System error |
| `expired` | Authentication expired |

## Data Extraction

### Personal Information

The system extracts and structures personal information:

```json
{
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
  }
}
```

### Document Information

```json
{
  "documentInfo": {
    "issuer": "Passport Immigration and Citizenship Agency",
    "countryOfIssue": "Jamaica",
    "idNumber": "A1234567",
    "controlNumber": "1410098515",
    "issueDate": "2021-07-29",
    "expirationDate": "2031-07-28",
    "firstIssueDate": "2011-07-30"
  }
}
```

### Analysis Results

```json
{
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
  }
}
```

## Account Recovery

The API includes comprehensive account recovery functionality for two scenarios:

### Scenario 1: Lost Account Access

**When**: Customer loses access to account (forgot password, lost email/phone)

**Process**:
1. Customer attempts verification with new contact info
2. System detects potential duplicate based on ID/face match
3. Creates recovery session with `lost_account` purpose
4. Operations team reviews both verifications side-by-side
5. Decision made: **same person** or **different person**

**Resolution Types**:
- **Same User** - Same person, new contact info → Restore to original account
- **Different User** - Different person → Create new account

### Scenario 2: New Device Access

**When**: Customer tries to access existing account from new device

**Process**:
1. Customer verifies on new device
2. System detects existing verified identity
3. Creates recovery session with `new_device` purpose
4. Operations team compares verifications
5. Decision made: **approve** or **deny** device access

**Resolution Types**:
- **New Device Verified** - Same person → Grant device access
- **New Device Rejected** - Suspicious → Deny access
- **Different Persons** - Different person → Transfer contact info

### Contact Info Management

Recovery process handles:
- Phone number transfers
- Email address transfers
- Device ID preservation
- Replaced contact tracking
- History logging

## Video Verification

For complex cases requiring human verification:

### Video Meeting Flow

```
1. Operations requests video meeting
2. System creates Daily.co video room (24h expiry)
3. Customer receives email with meeting link
4. Customer and agent join video call
5. Agent verifies identity in real-time
6. Meeting closed and logged
```

**Use Cases**:
- Unclear document images
- Suspicious submissions
- High-risk verifications
- Complex identity scenarios
- Additional verification required

## Webhook Integration

### Verification Link Webhooks

When verification is shared via verification link:

**Webhook Sent On**:
- `verification.cleared` - Identity verified and approved
- `verification.rejected` - Identity verification rejected

**Webhook Payload**:
```json
{
  "trn": "123-456-789",
  "firstName": "John",
  "lastName": "Smith",
  "documentNumber": "A1234567",
  "issueDate": "2021-07-29",
  "expiryDate": "2031-07-28",
  "issuingAuthority": "Passport Immigration and Citizenship Agency",
  "event": "cleared",
  "referenceNumber": "REF-ABC123",
  "verificationToken": "token_xyz123"
}
```

**Webhook Security**:
- HMAC signature validation
- Configurable secret key
- Timestamp validation
- Retry mechanism on failure

## AI-Powered Features

### Document Analysis

Uses OpenAI GPT-4 for:
- Text extraction from ID images
- Structured data extraction
- Document type classification
- Authenticity assessment
- Rejection reason generation

**Training Data**:
- Sample ID formats
- Field positioning
- Expected patterns
- Validation rules

### ID Generator (Testing)

For QA and testing purposes:
- Generates realistic test IDs
- Uses AI to create random data
- Follows Jamaican ID formats
- Includes barcodes and MRZ
- Prevents pattern repetition

## Security & Privacy

### Data Protection

- **Sensitive Data Masking** - ID numbers, contact details masked in public views
- **Profile Badge Controls** - User-controlled visibility settings
- **Access Control** - Organisation-based access lists
- **Audit Logging** - Complete audit trail of all actions
- **Temporary User Codes** - Verification without full registration

### Privacy Features

**Profile Badge Visibility Options**:
- **Name**: Full, first + last initial, initials only
- **Address**: Full, parish only, hidden
- **Email**: Full, masked, hidden
- **Phone**: Full, masked, hidden
- **ID Type**: Show or hide

### Compliance

- **GDPR Compliant** - Right to access, erasure, portability
- **Data Minimization** - Only collect necessary information
- **Purpose Limitation** - Use only for stated purpose
- **Retention Policies** - Automatic expiration and cleanup
- **Encryption** - Data encrypted at rest and in transit

## Traffic Ticket Integration

Special integration for Jamaican traffic ticket retrieval:

**Features**:
- Find tickets by driver's licence number
- Calculate total outstanding balance
- Send formatted email summary
- Send SMS with web link
- Mobile-friendly ticket view

**Requires**: `traffic-tickets-onboarding-team` role

## Privileges & Roles

### Required Privileges

| Action | Privilege |
|--------|-----------|
| View verification requests list | `view_identity_verification_requests_list` |
| View request details | `view_identity_verification_request_details` |
| View verification results list | `view_identity_verification_reports_list` |
| View result details | `view_identity_verification_report_details` |
| View public reports | `view_public_identity_verification_report_details` |
| View face auth results | `view_face_auth_results_list` |

### Operations-Only Actions

The following actions require `operations` role:
- Update verification decisions (clear/reject)
- Delete verification results
- Merge duplicate reports
- Request and manage video meetings
- Resolve account recovery cases
- Update TRN verification decisions
- Access all organisations' data
- View internal analysis details

## Rate Limits

- **Verification Requests**: 100 per hour per organisation
- **Result Retrieval**: 1000 per hour per organisation  
- **Decision Updates**: 200 per hour per organisation
- **API Calls**: 2000 per hour per organisation

Exceeding limits returns `429 Too Many Requests` with retry-after header.

## Error Handling

### Common Error Responses

**401 Unauthorized**:
```json
{
  "success": false,
  "error": "Invalid or expired API key"
}
```

**403 Forbidden**:
```json
{
  "notification": {
    "text": "You are not authorised to do that.",
    "type": "error"
  }
}
```

**404 Not Found**:
```json
{
  "message": "Request not found"
}
```

**500 Internal Server Error**:
```json
{
  "message": "Server error.",
  "error": "Error details"
}
```

## Best Practices

### For Merchants

1. **Request Management**
   - Check for existing verifications before requesting new ones
   - Use clear originator names for tracking
   - Set appropriate expiration dates
   - Monitor pending requests regularly

2. **Result Processing**
   - Review processing queue daily
   - Monitor duplicate detection alerts
   - Handle rejections promptly
   - Maintain verification history

3. **Data Security**
   - Never expose full ID numbers in client-side code
   - Use proper access controls
   - Respect profile badge visibility settings
   - Audit all access to sensitive data

### For Operations Teams

1. **Verification Review**
   - Review processing queue daily
   - Flag suspicious submissions for manual review
   - Validate document authenticity carefully
   - Cross-check with CBS for TRN verification
   - Monitor duplicate detection alerts

2. **Duplicate Handling**
   - Always check duplicate warnings before clearing
   - Use merge functionality to consolidate duplicates
   - Investigate face matches for potential fraud
   - Verify TRN uniqueness across system

3. **Account Recovery**
   - Review recovery cases carefully
   - Compare original and new verification side-by-side
   - Validate contact info changes
   - Use appropriate resolution type
   - Document decision rationale

4. **Quality Assurance**
   - Ensure consistent decision-making
   - Document rejection reasons clearly
   - Follow standard operating procedures
   - Escalate complex cases
   - Maintain audit trail

## Integration Examples

### Basic Verification Check

```javascript
// List pending verification results
const response = await fetch('/api/v1/merchant/identity/results?query=' + JSON.stringify({
  page: 1,
  sortField: 'createdAt',
  sortOrder: 'desc',
  filters: { status: ['processing', 'submitted'] }
}), {
  headers: {
    'Authorization': `Bearer ${apiKey}`,
    'Content-Type': 'application/json'
  }
});

const { records } = await response.json();
console.log(`${records.length} verifications awaiting review`);
```

### Get Verification Details

```javascript
// Get full verification report
const report = await fetch(
  `/api/v1/merchant/identity/report/${reportId}`,
  {
    headers: { 'Authorization': `Bearer ${apiKey}` }
  }
);

const data = await report.json();

// Check for duplicates
if (data.duplicateTRNs.length > 0) {
  console.warn('Duplicate TRN detected:', data.duplicateTRNs);
}

if (data.matches.length > 0) {
  console.warn('Face matches found:', data.matches);
}
```

### Clear Verification

```javascript
// Clear an identity verification
await fetch('/api/v1/merchant/identity/report/decision', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${apiKey}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    requestId: reportId,
    decision: {
      firstName: 'John',
      middleName: 'Michael',
      lastName: 'Smith'
    },
    status: 'cleared',
    withTrn: true,
    override: false
  })
});
```

## Next Steps

Ready to integrate the Identity Verification API?

1. **[Identity API Endpoints](./identity-endpoints.md)** - Complete endpoint reference
2. **[Authentication Guide](../authentication.md)** - API key setup
3. **[Integration Examples](../examples/)** - Sample code and workflows

## Support

For Identity Verification API support:

- **Technical Documentation**: [Identity API Endpoints](./identity-endpoints.md)
- **Email Support**: [support@cleared.id](mailto:support@cleared.id)
- **Admin Portal**: [https://cleared.id/admin](https://cleared.id/admin)

---

**Continue to**: [Identity API Endpoints](./identity-endpoints.md) →

