# Address Verification API

## Overview

The Address Verification API provides comprehensive address verification services for both individuals and companies. This service enables businesses to verify residential and business addresses using proof of address documents, GPS coordinates, photos, and video evidence.

**Base Path**: `/api/v1/merchant/address`

**Authentication**: Merchant JWT token in Authorization header

## Key Features

- **Proof of Address Verification** – Utility bills, bank statements, government letters
- **Document Analysis** – Extract and verify address details from documents
- **GPS Verification** – Validate physical location coordinates
- **Photo/Video Evidence** – Visual confirmation of address
- **Company Address Verification** – Business address validation
- **Due Diligence Requests** – Request additional documentation from customers
- **Multi-format Support** – PDF, images, various document types
- **Integration with Identity** – Auto-populate customer information

## Verification Workflow

### Standard Address Verification Flow

```
1. Merchant initiates verification request → Status: pending
2. Customer receives verification link → Opens verification flow
3. Customer uploads proof of address → Status: submitted
4. System processes documents → Status: processing
5. Operations team reviews → Status: flagged (if needed)
6. Decision made → Status: cleared or rejected
7. Merchant receives result → Webhook notification sent
8. Customer notified → Email confirmation
```

## Accepted Document Types

### Proof of Address Documents

#### Utility Bills
- **Electricity Bill** – JPS, JPS Co., Jamaica Public Service
- **Water Bill** – NWC, National Water Commission
- **Telephone Bill** – Flow, Digicel
- **Internet/Cable Bill** – Flow, Digicel

#### Financial Documents
- **Bank Statements** – Any licensed financial institution
- **Credit Card Statements** – Major credit providers
- **Mortgage Statements** – Financial institutions
- **Loan Statements** – Banks and credit unions

#### Government Documents
- **Tax Assessment Notice** – Tax Administration Jamaica
- **Property Tax Bill** – Local municipal authority
- **Government Correspondence** – Official government letters
- **Voter Registration Card** – Electoral Office of Jamaica

#### Rental/Property Documents
- **Rental Agreement/Lease** – Official rental contract
- **Tenancy Agreement** – Signed by landlord
- **Property Deed** – Title documents
- **Homeowners Insurance** – Insurance policy

#### Other Accepted Documents
- **Insurance Statements** – Home/property insurance
- **Employer Letters** – On official letterhead
- **School/University Letters** – Educational institutions

### Document Requirements

All documents must meet these criteria:

**Essential Requirements**:
- ✅ Show the customer's full name
- ✅ Show the complete address
- ✅ Be dated within last 3-6 months (depending on type)
- ✅ Be legible and clearly visible
- ✅ Include institution/issuer information
- ✅ Not be altered or tampered with

**Quality Standards**:
- High resolution (minimum 1000px width)
- Clear text (no blur or distortion)
- Complete document (all corners visible)
- Proper lighting (no glare or shadows)
- Authentic appearance (no obvious edits)

## Verification Methods

### 1. Document Verification

**Document Analysis**:
- Extract name from document
- Extract address details
- Validate institution/issuer
- Check document age/recency
- Assess document authenticity

**Address Extraction**:
- Address line 1 (street number and name)
- Address line 2 (area, community, apartment)
- Town/city
- Parish/county/region
- Country
- Postal code (if applicable)

**Institution Validation**:
- Verify issuer is legitimate
- Check institution format
- Validate document layout
- Confirm logo and branding

### 2. GPS Verification

**Location Validation**:
- Customer provides GPS coordinates
- System validates coordinates are in Jamaica
- Converts to address format
- Cross-references with document address
- Calculates distance/accuracy

**GPS Requirements**:
- Accuracy within 50 meters
- Located in Jamaica
- Matches general area of document address
- Recent coordinates (< 24 hours old)

### 3. Photo/Video Evidence

**Visual Verification**:
- Photo of property exterior
- Photo showing street address/number
- Video walkthrough of property
- Photo of mailbox with address

**Photo Requirements**:
- Clear and recent (< 7 days)
- Shows identifiable features
- Includes visible address marker
- Good lighting and quality

### 4. Manual Address Entry

Customer can provide:
- Manually typed address
- GPS picked location
- Map-selected address

**Validation**:
- Format validation
- Completeness check
- Cross-reference with document
- Parish/town validation

## Due Diligence Requests

When additional information is needed:

### Request Types

**Document Request** (`address-document`):
- Request more recent utility bill
- Request different document type
- Request clearer document image
- Request additional proof

**GPS Request** (`address-gps`):
- Request GPS coordinates
- Request location update
- Verify physical presence

**Photo Request** (`address-photo`):
- Request property photo
- Request street view
- Request address marker photo

**Video Request** (`address-video`):
- Request property walkthrough
- Request neighbourhood video
- Request detailed evidence

### Due Diligence Workflow

```
1. Operations identifies need for additional info
2. Creates due diligence request with note
3. System creates high-priority action for customer
4. Customer receives email with instructions
5. Customer uploads additional information
6. Operations reviews new submission
7. Decision made (cleared/rejected/more info needed)
```

## Company Address Verification

### Company vs Individual

**Differences**:
- Links to company verification result
- Validates business address
- Checks company ownership
- Verifies authorised representative
- Uses company letterhead/documents

**Company-Specific Documents**:
- Company utility bills
- Business bank statements
- Commercial lease agreements
- Business license/permits
- Tax registration documents

### Company Data Model

```json
{
  "companyId": "company_id",
  "companyName": "Acme Corporation Ltd",
  "documentAddress": {
    "addressLine1": "45 Business Plaza",
    "addressLine2": "New Kingston",
    "town": "Kingston",
    "parish": "St. Andrew"
  },
  "verifiedAddress": {...},
  "company": {
    "_id": "company_id",
    "name": "Acme Corporation Ltd",
    "registrationNumber": "12345678",
    "status": "verified"
  }
}
```

## Integration with Identity Verification

### Automatic Name Resolution

When customer has identity verification:
- Name auto-populated from identity result
- Personal info synced automatically
- Biography details inherited
- Updates cached when identity changes

**Resolution Process**:
1. System checks for identity verification
2. Extracts personal information
3. Populates name and biography fields
4. Caches in address verification result
5. Marks as `nameUpdated` to prevent re-resolution

### Temporary User Codes

For customers without full accounts:
- System assigns temporary user code
- Links address to identity via code
- Enables verification without registration
- Name resolved when identity cleared

**Example Linking**:
```json
{
  "userId": "catch-all-user-id",
  "meta": {
    "temporaryUserCode": "temp-abc123-def456",
    "linkedIdentityVerificationId": "identity_verification_id"
  }
}
```

## Jamaican Address Standards

### Standard Address Format

```
[Street Number] [Street Name]
[Area/Community]
[Town/City], [Parish]
Jamaica
```

### Parish Names

The 14 parishes of Jamaica:
- Clarendon
- Hanover
- Kingston
- Manchester
- Portland
- St. Andrew
- St. Ann
- St. Catherine
- St. Elizabeth
- St. James
- St. Mary
- St. Thomas
- Trelawny
- Westmoreland

### Common Towns/Cities

- Kingston
- Spanish Town
- Montego Bay
- Portmore
- May Pen
- Mandeville
- Old Harbour
- Savanna-la-Mar
- Ocho Rios
- Negril
- Port Antonio

## Status Lifecycle

### Address Verification Status Flow

```
new → submitted → processing → flagged (optional) → cleared/rejected
                              ↓
                           deleted
```

**Status Transitions**:
1. `new` - Verification created, awaiting submission
2. `submitted` - Customer uploaded documents
3. `processing` - Operations team reviewing
4. `flagged` - Requires additional review or information
5. `cleared` - Approved and verified
6. `rejected` - Verification failed/rejected
7. `deleted` - Soft deleted

### Due Diligence Workflow

When additional information is needed:

```
processing → flagged → due diligence request → customer action → 
resubmitted → processing → cleared/rejected
```

## Best Practices

### 1. Document Selection

- **Prefer recent documents** - Within 3 months is ideal
- **Use official sources** - Government or major institutions
- **Clear visibility** - All text must be readable
- **Complete documents** - Show all corners and edges

### 2. Address Formatting

- Use standard Jamaican address format
- Include proper parish names
- Spell out street names fully
- Use consistent capitalisation

### 3. Quality Control

- Ensure document quality before submission
- Verify all information is visible
- Check document is not expired
- Confirm institution is legitimate

### 4. Processing Efficiency

- Review verifications promptly
- Request additional info early if needed
- Provide clear instructions to customers
- Follow up on pending actions

## Next Steps

Ready to integrate the Address Verification API?

1. **[Address API Endpoints](./address-endpoints.md)** - Complete endpoint reference
2. **[Authentication Guide](../authentication.md)** - API key setup
3. **[Integration Examples](../examples/)** - Sample code and workflows

## Support

For Address Verification API support:

- **Technical Documentation**: [Address API Endpoints](./address-endpoints.md)
- **Email Support**: [support@cleared.id](mailto:support@cleared.id)
- **Admin Portal**: [https://cleared.id/admin](https://cleared.id/admin)

---

**Continue to**: [Address API Endpoints](./address-endpoints.md) →

