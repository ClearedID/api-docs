# Onboarding Pages API

## Overview

The Onboarding Pages API allows merchants to create customisable, shareable onboarding pages that combine lead capture forms, identity verification, address verification, and other verification types into a single workflow. Perfect for loan applications, rental applications, employee onboarding, or any scenario requiring customer information collection with built-in verification.

**Base Path**: `/api/v1/merchant/screening/onboarding`

**Authentication**: Merchant JWT token in Authorization header

## Key Features

- **Visual Page Builder** – Create branded onboarding pages with custom logos and colors
- **Multi-Step Workflows** – Combine verifications and forms into complete workflows
- **Lead Capture Forms** – Collect custom information before verification
- **Embedded Verifications** – Include identity, address, employment, and other verifications
- **Custom Form Fields** – Create reusable custom fields for your organisation
- **Shareable Links** – Share pages via URL for customer access
- **Submission Tracking** – Monitor all submissions and verification statuses
- **Logo Management** – Reuse uploaded logos across multiple pages
- **Status Management** – Activate or deactivate pages as needed

## Onboarding Page Structure

### Page Components

An onboarding page consists of:

1. **Lead Capture Form** (Optional)
   - Custom fields for collecting initial information
   - Standard fields (name, email, phone)
   - Custom fields specific to your business

2. **Required Verifications** (Optional)
   - Identity verification
   - Address verification
   - Employment verification
   - Income verification
   - Reference checks
   - Qualification verification
   - Background checks

3. **Required Forms** (Optional)
   - Predefined forms from your form library
   - Custom questionnaires
   - Document collection forms

4. **Branding & Styling**
   - Custom logo
   - Primary brand color
   - Button text customization
   - Page title and description

### Customer Journey

```
1. Customer clicks onboarding page link
2. [Optional] Fills out lead capture form
3. Completes required verifications
4. [Optional] Fills out additional forms
5. Submits to merchant for review
6. Merchant receives notification
7. Customer redirected to destination URL
```

## Endpoints

### Page Management

#### 1. Create Onboarding Page

Create a new onboarding page.

**Endpoint**: `POST /api/v1/merchant/screening/onboarding/pages/create`

**Content-Type**: `multipart/form-data` (for logo upload)

**Request Body (Form Data)**:
```
pageTitle: "Loan Application"
description: "Complete your loan application with identity verification"
status: "active"
primaryColor: "blue"
buttonText: "Continue"
submitButtonText: "Submit Application"
destination: "/onboarding/dashboard"
requiredVerifications: ["identity", "address", "income"]
requiredForms: []
logo: [file upload]
```

**Request Parameters**:
- `pageTitle` (string, required) - Page title displayed to customers
- `description` (string, optional) - Page description
- `status` (string, optional) - Status: `active` or `inactive` (default: `active`)
- `primaryColor` (string, optional) - Brand color (default: `blue`)
- `buttonText` (string, optional) - Continue button text (default: `Continue`)
- `submitButtonText` (string, optional) - Final submit button text (default: `Submit to Client`)
- `destination` (string, optional) - Redirect URL after submission (default: `/onboarding/dashboard`)
- `requiredVerifications` (JSON array, optional) - Array of verification types
- `requiredForms` (JSON array, optional) - Array of form IDs
- `logo` (file, optional) - Logo image file (PNG, JPG, JPEG)

**Success Response** (200):
```json
{
  "success": true,
  "message": "Onboarding page created successfully",
  "onboardingPage": {
    "_id": "507f1f77bcf86cd799439100",
    "pageId": "loan-application",
    "pageTitle": "Loan Application",
    "description": "Complete your loan application with identity verification",
    "status": "active",
    "pageLogoUrl": "https://cleared-public.s3.us-east-1.amazonaws.com/onboarding-logos/...",
    "requiredVerifications": [
      {
        "type": "identity",
        "title": "Identity Verification",
        "required": true
      },
      {
        "type": "address",
        "title": "Address Verification",
        "required": true
      },
      {
        "type": "income",
        "title": "Income Verification",
        "required": true
      }
    ],
    "requiredForms": [],
    "clientBranding": {
      "primaryColor": "blue",
      "buttonText": "Continue",
      "submitButtonText": "Submit Application"
    },
    "destination": "/onboarding/dashboard",
    "clientId": "507f1f77bcf86cd799439030",
    "organisationId": "507f1f77bcf86cd799439031",
    "organisationName": "Acme Financial Services",
    "createdBy": "507f1f77bcf86cd799439032",
    "createdAt": "2025-10-19T10:00:00Z"
  }
}
```

**Error Response** (400):
```json
{
  "success": false,
  "message": "Page title is required"
}
```

**Notes**:
- `pageId` is auto-generated (12-character alphanumeric)
- Logo uploaded to S3 public bucket
- Verification types automatically mapped to user-friendly titles
- Organisation ID populated from authenticated user
- Page immediately accessible at public onboarding URL

---

#### 2. List Onboarding Pages

Retrieve all onboarding pages for the authenticated organisation.

**Endpoint**: `GET /api/v1/merchant/screening/onboarding/pages`

**Success Response** (200):
```json
{
  "success": true,
  "pages": [
    {
      "_id": "507f1f77bcf86cd799439100",
      "pageId": "loan-application",
      "pageTitle": "Loan Application",
      "description": "Complete your loan application with identity verification",
      "status": "active",
      "pageLogoUrl": "https://cleared-public.s3.us-east-1.amazonaws.com/onboarding-logos/...",
      "requiredVerifications": [
        {
          "type": "identity",
          "title": "Identity Verification",
          "required": true
        }
      ],
      "requiredForms": [],
      "clientBranding": {
        "primaryColor": "blue",
        "buttonText": "Continue",
        "submitButtonText": "Submit Application"
      },
      "leadCapture": {
        "enabled": true,
        "title": "Get Started",
        "fields": [...]
      },
      "styling": {},
      "destination": "/onboarding/dashboard",
      "views": 245,
      "submissions": 128,
      "organisationId": "507f1f77bcf86cd799439031",
      "createdAt": "2025-10-19T10:00:00Z",
      "updatedAt": "2025-10-19T14:00:00Z"
    }
  ]
}
```

**Notes**:
- Returns only pages for authenticated organisation
- Sorted by creation date (newest first)
- Logo URLs converted from S3 keys to full public URLs
- Includes view and submission counts (if tracking enabled)

---

#### 3. Get Single Onboarding Page

Retrieve details of a specific onboarding page.

**Endpoint**: `GET /api/v1/merchant/screening/onboarding/pages/:id`

**URL Parameters**:
- `id` (string, required) - Page MongoDB ObjectId

**Success Response** (200):
```json
{
  "success": true,
  "onboardingPage": {
    "_id": "507f1f77bcf86cd799439100",
    "pageId": "loan-application",
    "pageTitle": "Loan Application",
    "description": "Complete your loan application with identity verification",
    "status": "active",
    "pageLogoUrl": "https://cleared-public.s3.us-east-1.amazonaws.com/onboarding-logos/...",
    "requiredVerifications": [...],
    "requiredForms": [...],
    "leadCapture": {
      "enabled": true,
      "title": "Get Started",
      "description": "Fill out the form below to begin",
      "fields": [
        {
          "id": "firstName",
          "label": "First Name",
          "type": "text",
          "fieldType": "standard",
          "required": true,
          "order": 1
        }
      ],
      "successMessage": "Thank you! Please continue with verification."
    },
    "styling": {
      "theme": "light",
      "fontFamily": "Inter"
    },
    "clientBranding": {...},
    "destination": "/onboarding/dashboard",
    "backgroundCheckCostSplit": "customer",
    "organisationId": "507f1f77bcf86cd799439031",
    "usedPageIds": ["loan-application"],
    "createdAt": "2025-10-19T10:00:00Z",
    "updatedAt": "2025-10-19T14:00:00Z"
  }
}
```

**Error Response** (404):
```json
{
  "success": false,
  "error": "Onboarding page not found"
}
```

**Notes**:
- Only returns pages owned by authenticated organisation
- Full page configuration including lead capture and styling
- Used for editing page configuration

---

#### 4. Update Onboarding Page

Update an existing onboarding page.

**Endpoint**: `POST /api/v1/merchant/screening/onboarding/pages/:id/update`

**URL Parameters**:
- `id` (string, required) - Page MongoDB ObjectId

**Content-Type**: `multipart/form-data`

**Request Body (Form Data)**:
```
pageId: "loan-app-v2"
pageTitle: "Updated Loan Application"
description: "Updated description"
status: "active"
primaryColor: "#0d2c54"
buttonText: "Next Step"
submitButtonText: "Submit Now"
destination: "/thank-you"
requiredVerifications: ["identity", "address", "income", "employment"]
requiredForms: ["employment-history"]
leadCapture: {JSON string}
styling: {JSON string}
backgroundCheckCostSplit: "merchant"
logo: [file upload or null]
existingLogoId: "onboarding-logos/123456789_logo.png"
```

**Request Parameters**:
- `pageId` (string, optional) - New custom page ID (must be unique)
- `pageTitle` (string, required) - Updated page title
- `description` (string, optional) - Updated description
- `status` (string, optional) - Updated status
- `primaryColor` (string, optional) - Updated brand color
- `buttonText` (string, optional) - Updated button text
- `submitButtonText` (string, optional) - Updated submit button text
- `destination` (string, optional) - Updated redirect URL
- `requiredVerifications` (JSON array, optional) - Updated verifications
- `requiredForms` (JSON array, optional) - Updated forms
- `leadCapture` (JSON string, optional) - Lead capture configuration
- `styling` (JSON string, optional) - Styling configuration
- `backgroundCheckCostSplit` (string, optional) - `customer` or `merchant`
- `logo` (file, optional) - New logo file
- `existingLogoId` (string, optional) - Use existing uploaded logo

**Success Response** (200):
```json
{
  "success": true,
  "message": "Onboarding page updated successfully",
  "onboardingPage": {
    "_id": "507f1f77bcf86cd799439100",
    "pageId": "loan-app-v2",
    "pageTitle": "Updated Loan Application",
    "updatedAt": "2025-10-19T15:30:00Z",
    ...
  }
}
```

**Error Responses**:

400 Bad Request - Invalid Page ID:
```json
{
  "success": false,
  "message": "Page ID can only contain letters, numbers, hyphens, and underscores"
}
```

400 Bad Request - Duplicate Page ID:
```json
{
  "success": false,
  "message": "This Page ID is already taken. Please choose a different one."
}
```

404 Not Found:
```json
{
  "error": "Onboarding page not found"
}
```

**Page ID Management**:
- Can change `pageId` to custom value
- Must be unique across all organisations
- Old pageId saved in `usedPageIds` array
- Page ID format: alphanumeric, hyphens, underscores only

**Logo Handling**:
- Upload new logo: Send file in `logo` field
- Use existing logo: Send S3 key in `existingLogoId`
- Remove logo: Don't send either field
- New logo uploads replace old logo

**Notes**:
- Only updates pages owned by authenticated organisation
- Lead capture and styling sent as JSON strings
- Background check cost can be split between merchant and customer
- Page ID change tracked in `usedPageIds` for URL redirects

---

#### 5. Delete Onboarding Page

Delete an onboarding page.

**Endpoint**: `POST /api/v1/merchant/screening/onboarding/pages/:id/delete`

**URL Parameters**:
- `id` (string, required) - Page MongoDB ObjectId

**Success Response** (200):
```json
{
  "success": true,
  "message": "Onboarding page deleted successfully"
}
```

**Error Response** (404):
```json
{
  "error": "Onboarding page not found"
}
```

**Notes**:
- Hard delete - permanently removes page
- Does not delete associated submissions
- Page URL becomes invalid immediately
- Only deletes pages owned by authenticated organisation
- Consider setting status to `inactive` instead for soft deletion

---

### Logo Management

#### 6. Get Uploaded Logos

Retrieve all logos uploaded by the organisation for reuse.

**Endpoint**: `GET /api/v1/merchant/screening/onboarding/logos`

**Success Response** (200):
```json
{
  "success": true,
  "logos": [
    {
      "_id": "onboarding-logos/1634567890_company-logo.png",
      "name": "company-logo",
      "url": "https://cleared-public.s3.us-east-1.amazonaws.com/onboarding-logos/1634567890_company-logo.png",
      "clientName": "Acme Financial Services",
      "fileKey": "onboarding-logos/company-logo.png"
    },
    {
      "_id": "onboarding-logos/1634567891_alternate-logo.png",
      "name": "alternate-logo",
      "url": "https://cleared-public.s3.us-east-1.amazonaws.com/onboarding-logos/1634567891_alternate-logo.png",
      "clientName": "Acme Financial Services",
      "fileKey": "onboarding-logos/alternate-logo.png"
    }
  ]
}
```

**Response Fields**:
- `_id` - File key (used as identifier)
- `name` - Friendly name extracted from filename
- `url` - Full S3 public URL
- `clientName` - Organisation name
- `fileKey` - S3 file key for updates

**Notes**:
- Returns unique logos from all organisation's onboarding pages
- Logos extracted from existing pages only
- Used for logo picker in page editor
- Prevents duplicate logo uploads

---

### Field Management

#### 7. Get Standard Fields

Get available standard fields for lead capture forms.

**Endpoint**: `GET /api/v1/merchant/screening/onboarding/standard-fields`

**Success Response** (200):
```json
{
  "success": true,
  "fields": [
    {
      "id": "firstName",
      "label": "First Name",
      "type": "text",
      "description": "Person's first name",
      "category": "personal",
      "autoFill": true,
      "readOnly": true
    },
    {
      "id": "lastName",
      "label": "Last Name",
      "type": "text",
      "description": "Person's last name",
      "category": "personal",
      "autoFill": true,
      "readOnly": true
    },
    {
      "id": "emailAddress",
      "label": "Email Address",
      "type": "email",
      "description": "Primary email address",
      "category": "contact",
      "autoFill": true
    },
    {
      "id": "phoneNumber",
      "label": "Phone Number",
      "type": "phone",
      "description": "Primary phone number",
      "category": "contact",
      "autoFill": true
    },
    {
      "id": "dateOfBirth",
      "label": "Date of Birth",
      "type": "date",
      "description": "Person's date of birth",
      "category": "personal",
      "autoFill": true,
      "readOnly": true
    },
    {
      "id": "trn",
      "label": "TRN",
      "type": "text",
      "description": "Tax Registration Number",
      "category": "identification",
      "autoFill": true,
      "readOnly": true
    },
    {
      "id": "address",
      "label": "Address",
      "type": "textarea",
      "description": "Full residential address",
      "category": "location",
      "autoFill": true,
      "readOnly": true
    }
  ]
}
```

**Field Categories**:
- `personal` - Name, DOB, gender
- `contact` - Email, phone
- `identification` - TRN, ID number
- `location` - Address, parish, country

**Field Properties**:
- `autoFill` - Automatically filled from verification data
- `readOnly` - Cannot be edited by customer
- `category` - Grouping for organization

**Notes**:
- Standard fields map to verified customer data
- Auto-fill fields populated after verification
- Read-only fields prevent customer tampering
- Used in lead capture form builder

---

#### 8. Get Custom Fields

Get organisation-specific custom fields.

**Endpoint**: `GET /api/v1/merchant/screening/onboarding/custom-fields`

**Success Response** (200):
```json
{
  "success": true,
  "fields": [
    {
      "_id": "507f1f77bcf86cd799439110",
      "fieldId": "employment-status",
      "label": "Employment Status",
      "type": "select",
      "options": [
        { "value": "employed", "label": "Employed" },
        { "value": "self-employed", "label": "Self-Employed" },
        { "value": "unemployed", "label": "Unemployed" },
        { "value": "retired", "label": "Retired" }
      ],
      "description": "Current employment status",
      "placeholder": "Select employment status",
      "validation": {},
      "organisationId": "507f1f77bcf86cd799439031",
      "createdAt": "2025-10-15T10:00:00Z"
    }
  ]
}
```

**Notes**:
- Returns only custom fields created by organisation
- Reusable across multiple onboarding pages
- Supports all standard HTML input types

---

#### 9. Create Custom Field

Create a new reusable custom field.

**Endpoint**: `POST /api/v1/merchant/screening/onboarding/custom-fields`

**Request Body**:
```json
{
  "fieldId": "monthly-income",
  "label": "Monthly Income",
  "type": "number",
  "description": "Gross monthly income",
  "placeholder": "Enter amount in JMD",
  "validation": {
    "minValue": 0,
    "maxValue": 10000000
  }
}
```

**Request Parameters**:
- `fieldId` (string, required) - Unique field identifier
- `label` (string, required) - Display label
- `type` (string, required) - Field type: `text`, `email`, `phone`, `textarea`, `select`, `checkbox`, `radio`, `date`, `number`
- `description` (string, optional) - Help text
- `placeholder` (string, optional) - Placeholder text
- `options` (array, optional) - Options for select/radio/checkbox
- `validation` (object, optional) - Validation rules

**Success Response** (200):
```json
{
  "success": true,
  "message": "Custom field created successfully",
  "field": {
    "_id": "507f1f77bcf86cd799439111",
    "fieldId": "monthly-income",
    "label": "Monthly Income",
    "type": "number",
    "organisationId": "507f1f77bcf86cd799439031",
    "createdAt": "2025-10-19T16:00:00Z"
  }
}
```

**Validation Options**:
```json
{
  "validation": {
    "minLength": 5,
    "maxLength": 100,
    "minValue": 0,
    "maxValue": 999999,
    "pattern": "^[A-Z]{2}[0-9]{6}$",
    "required": true
  }
}
```

**Notes**:
- Field ID must be unique within organisation
- Supports complex validation rules
- Available immediately in form builder
- Reusable across all organisation's pages

---

#### 10. Delete Custom Field

Delete a custom field.

**Endpoint**: `POST /api/v1/merchant/screening/onboarding/custom-fields/:fieldId/delete`

**URL Parameters**:
- `fieldId` (string, required) - Field ID to delete

**Success Response** (200):
```json
{
  "success": true,
  "message": "Custom field deleted successfully"
}
```

**Error Response** (404):
```json
{
  "error": "Custom field not found"
}
```

**Notes**:
- Does not remove field from existing pages
- Existing pages with this field continue to work
- Field no longer available in form builder
- Consider implications before deletion

---

### Submissions & Analytics

#### 11. Get Page Submissions

Retrieve submissions for a specific onboarding page.

**Endpoint**: `GET /api/v1/merchant/screening/onboarding/submissions/:pageId`

**URL Parameters**:
- `pageId` (string, required) - Page MongoDB ObjectId

**Query Parameters**:
- `query` (JSON string) - Search, filter, and pagination options:
```json
{
  "keywords": "john smith",
  "page": 1,
  "sortField": "submittedAt",
  "sortOrder": "desc",
  "filters": {
    "status": "completed"
  }
}
```

**Success Response** (200):
```json
{
  "records": [
    {
      "_id": "507f1f77bcf86cd799439120",
      "pageId": "507f1f77bcf86cd799439100",
      "requestId": "507f1f77bcf86cd799439013",
      "userId": "507f1f77bcf86cd799439050",
      "contactInfo": {
        "name": "John Smith",
        "emailAddress": "john@example.com",
        "phoneNumber": "+18761234567"
      },
      "formData": {
        "firstName": "John",
        "lastName": "Smith",
        "emailAddress": "john@example.com",
        "phoneNumber": "+18761234567",
        "employmentStatus": "employed",
        "monthlyIncome": 500000
      },
      "verificationStatuses": {
        "identity": "cleared",
        "address": "cleared",
        "income": "pending"
      },
      "status": "in-progress",
      "submittedAt": "2025-10-19T11:00:00Z",
      "completedAt": null,
      "organisationId": "507f1f77bcf86cd799439031"
    }
  ],
  "paging": {
    "recordCount": 128,
    "pageCount": 13,
    "currentPage": 1
  }
}
```

**Submission Status Values**:
- `pending` - Lead captured, verification not started
- `in-progress` - Verifications in progress
- `completed` - All verifications completed
- `submitted` - Submitted to merchant for review
- `approved` - Merchant approved
- `rejected` - Merchant rejected

**Search Capabilities**:
- Keywords search: name, email, phone, request ID
- Filter by status
- Sort by any field
- Pagination support

**Notes**:
- Returns only submissions for pages owned by organisation
- Includes verification statuses for each required verification
- Contact info populated from user record if not in form data
- Default sort: submission date (newest first)

---

### Public Endpoints

#### 12. Submit Lead Capture Form

Submit lead capture form (public endpoint, used by customers).

**Endpoint**: `POST /api/v1/merchant/screening/onboarding/pages/:id/lead-capture`

**URL Parameters**:
- `id` (string, required) - Page MongoDB ObjectId

**Request Body**:
```json
{
  "formData": {
    "firstName": "John",
    "lastName": "Smith",
    "emailAddress": "john@example.com",
    "phoneNumber": "+18761234567",
    "employmentStatus": "employed",
    "monthlyIncome": 500000
  }
}
```

**Success Response** (200):
```json
{
  "success": true,
  "message": "Thank you! We'll be in touch soon.",
  "leadId": "507f1f77bcf86cd799439120"
}
```

**Error Responses**:

400 Bad Request - Lead capture disabled:
```json
{
  "success": false,
  "message": "Lead capture is not enabled for this page"
}
```

400 Bad Request - Missing required field:
```json
{
  "success": false,
  "message": "Email Address is required"
}
```

404 Not Found:
```json
{
  "success": false,
  "message": "Onboarding page not found"
}
```

**Process**:
1. Validates page exists and lead capture enabled
2. Validates all required fields present
3. Creates lead capture record
4. Returns success message (customizable per page)
5. Customer proceeds to verification steps

**Notes**:
- No authentication required (public endpoint)
- Validates required fields based on page configuration
- Success message customizable per page
- Lead ID returned for tracking

---

## Data Models

### Onboarding Page Object

```json
{
  "_id": "507f1f77bcf86cd799439100",
  "pageId": "loan-application",
  "clientId": "507f1f77bcf86cd799439030",
  "organisationId": "507f1f77bcf86cd799439031",
  "organisationName": "Acme Financial Services",
  "pageTitle": "Loan Application",
  "description": "Complete your loan application with identity verification",
  "pageLogoUrl": "onboarding-logos/1634567890_logo.png",
  "requiredVerifications": [
    {
      "type": "identity",
      "title": "Identity Verification",
      "description": "Government-issued ID verification",
      "required": true
    }
  ],
  "requiredForms": [
    {
      "formId": "employment-history",
      "title": "Employment History",
      "description": "Last 3 employers",
      "required": true
    }
  ],
  "leadCapture": {
    "enabled": true,
    "title": "Get Started",
    "description": "Fill out the form below to begin",
    "fields": [
      {
        "id": "firstName",
        "label": "First Name",
        "type": "text",
        "fieldType": "standard",
        "standardFieldMapping": "firstName",
        "required": true,
        "placeholder": "Enter first name",
        "order": 1,
        "validation": {
          "minLength": 2,
          "maxLength": 50
        }
      }
    ],
    "successMessage": "Thank you! Please continue with verification."
  },
  "styling": {
    "theme": "light",
    "fontFamily": "Inter",
    "backgroundColor": "#ffffff",
    "textColor": "#000000"
  },
  "clientBranding": {
    "primaryColor": "#0d2c54",
    "buttonText": "Continue",
    "submitButtonText": "Submit Application"
  },
  "destination": "/onboarding/dashboard",
  "backgroundCheckCostSplit": "customer",
  "status": "active",
  "usedPageIds": ["loan-application", "loan-app-v2"],
  "views": 245,
  "submissions": 128,
  "createdBy": "507f1f77bcf86cd799439032",
  "createdAt": "2025-10-19T10:00:00Z",
  "updatedAt": "2025-10-19T15:30:00Z"
}
```

### Submission Object

```json
{
  "_id": "507f1f77bcf86cd799439120",
  "pageId": "507f1f77bcf86cd799439100",
  "requestId": "507f1f77bcf86cd799439013",
  "userId": "507f1f77bcf86cd799439050",
  "contactInfo": {
    "name": "John Smith",
    "emailAddress": "john@example.com",
    "phoneNumber": "+18761234567"
  },
  "formData": {
    "firstName": "John",
    "lastName": "Smith",
    "emailAddress": "john@example.com",
    "phoneNumber": "+18761234567",
    "employmentStatus": "employed",
    "monthlyIncome": 500000,
    "customField1": "Custom value"
  },
  "verificationStatuses": {
    "identity": "cleared",
    "address": "cleared",
    "income": "pending",
    "employment": "rejected"
  },
  "verificationResults": {
    "identity": "507f1f77bcf86cd799439011",
    "address": "507f1f77bcf86cd799439012"
  },
  "status": "in-progress",
  "submittedAt": "2025-10-19T11:00:00Z",
  "completedAt": null,
  "approvedAt": null,
  "rejectedAt": null,
  "organisationId": "507f1f77bcf86cd799439031"
}
```

## Complete Integration Example

### Full Onboarding Page Creation & Management

```javascript
// 1. Create onboarding page with logo
const createPage = async () => {
  const formData = new FormData();
  formData.append('pageTitle', 'Loan Application');
  formData.append('description', 'Complete your loan application');
  formData.append('status', 'active');
  formData.append('primaryColor', '#0d2c54');
  formData.append('buttonText', 'Next Step');
  formData.append('submitButtonText', 'Submit Application');
  formData.append('destination', '/onboarding/thank-you');
  formData.append('requiredVerifications', JSON.stringify([
    'identity',
    'address',
    'income'
  ]));
  formData.append('requiredForms', JSON.stringify([]));
  
  // Add logo file
  const logoFile = document.querySelector('#logo-input').files[0];
  if (logoFile) {
    formData.append('logo', logoFile);
  }
  
  const response = await fetch(
    '/api/v1/merchant/screening/onboarding/pages/create',
    {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${apiKey}`
      },
      body: formData
    }
  );
  
  const result = await response.json();
  console.log('Page created:', result.onboardingPage);
  
  return result.onboardingPage;
};

// 2. Add lead capture form
const addLeadCapture = async (pageId) => {
  const formData = new FormData();
  formData.append('pageTitle', 'Loan Application');
  formData.append('description', 'Complete your loan application');
  formData.append('status', 'active');
  formData.append('primaryColor', '#0d2c54');
  formData.append('buttonText', 'Next Step');
  formData.append('submitButtonText', 'Submit Application');
  formData.append('destination', '/onboarding/thank-you');
  formData.append('requiredVerifications', JSON.stringify(['identity', 'address', 'income']));
  formData.append('requiredForms', JSON.stringify([]));
  
  // Add lead capture configuration
  const leadCapture = {
    enabled: true,
    title: 'Get Started',
    description: 'Tell us about yourself',
    fields: [
      {
        id: 'firstName',
        label: 'First Name',
        type: 'text',
        fieldType: 'standard',
        required: true,
        order: 1
      },
      {
        id: 'lastName',
        label: 'Last Name',
        type: 'text',
        fieldType: 'standard',
        required: true,
        order: 2
      },
      {
        id: 'emailAddress',
        label: 'Email',
        type: 'email',
        fieldType: 'standard',
        required: true,
        order: 3
      },
      {
        id: 'phoneNumber',
        label: 'Phone',
        type: 'phone',
        fieldType: 'standard',
        required: true,
        order: 4
      },
      {
        id: 'monthly-income',
        label: 'Monthly Income',
        type: 'number',
        fieldType: 'custom',
        required: false,
        order: 5
      }
    ],
    successMessage: 'Thank you! Please continue with verification.'
  };
  
  formData.append('leadCapture', JSON.stringify(leadCapture));
  
  await fetch(
    `/api/v1/merchant/screening/onboarding/pages/${pageId}/update`,
    {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${apiKey}`
      },
      body: formData
    }
  );
};

// 3. List all pages
const listPages = async () => {
  const response = await fetch(
    '/api/v1/merchant/screening/onboarding/pages',
    {
      headers: {
        'Authorization': `Bearer ${apiKey}`
      }
    }
  );
  
  const { pages } = await response.json();
  console.log(`${pages.length} onboarding pages`);
  
  return pages;
};

// 4. Monitor submissions
const monitorSubmissions = async (pageId) => {
  const query = {
    keywords: '',
    page: 1,
    sortField: 'submittedAt',
    sortOrder: 'desc',
    filters: {
      status: 'in-progress'
    }
  };
  
  const response = await fetch(
    `/api/v1/merchant/screening/onboarding/submissions/${pageId}?query=${encodeURIComponent(JSON.stringify(query))}`,
    {
      headers: {
        'Authorization': `Bearer ${apiKey}`
      }
    }
  );
  
  const { records, paging } = await response.json();
  
  console.log(`${records.length} submissions (${paging.recordCount} total)`);
  
  // Process submissions
  records.forEach(submission => {
    console.log(`Submission: ${submission.contactInfo.name}`);
    console.log(`Status: ${submission.status}`);
    console.log(`Verifications:`, submission.verificationStatuses);
  });
  
  return records;
};

// 5. Create custom field
const createCustomField = async () => {
  const response = await fetch(
    '/api/v1/merchant/screening/onboarding/custom-fields',
    {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${apiKey}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        fieldId: 'employment-status',
        label: 'Employment Status',
        type: 'select',
        description: 'Current employment status',
        options: [
          { value: 'employed', label: 'Employed' },
          { value: 'self-employed', label: 'Self-Employed' },
          { value: 'unemployed', label: 'Unemployed' }
        ]
      })
    }
  );
  
  const result = await response.json();
  console.log('Custom field created:', result.field);
};

// Complete workflow
(async () => {
  // Create page
  const page = await createPage();
  console.log('Page URL:', `https://cleared.id/onboarding/${page.pageId}`);
  
  // Add lead capture
  await addLeadCapture(page._id);
  console.log('Lead capture added');
  
  // Monitor submissions every 30 seconds
  setInterval(() => {
    monitorSubmissions(page._id);
  }, 30000);
})();
```

## Common Use Cases

### Use Case 1: Loan Application

```javascript
// Create comprehensive loan application page
const loanApp = await createOnboardingPage({
  pageTitle: 'Loan Application',
  description: 'Apply for a personal loan with instant approval',
  verifications: ['identity', 'address', 'income', 'employment'],
  leadCapture: {
    fields: [
      'firstName', 'lastName', 'emailAddress', 'phoneNumber',
      'dateOfBirth', 'trn', 'monthlyIncome', 'employmentStatus'
    ]
  },
  styling: {
    theme: 'professional',
    primaryColor: '#0d2c54'
  }
});

console.log(`Share link: https://cleared.id/onboarding/${loanApp.pageId}`);
```

### Use Case 2: Rental Application

```javascript
// Create rental application with tenant screening
const rentalApp = await createOnboardingPage({
  pageTitle: 'Tenant Application',
  description: 'Apply to rent at our property',
  verifications: ['identity', 'address', 'income', 'references'],
  leadCapture: {
    fields: [
      'fullName', 'emailAddress', 'phoneNumber',
      'currentAddress', 'employmentStatus', 'monthlyIncome',
      'moveInDate', 'numberOfOccupants'
    ]
  }
});
```

### Use Case 3: Employee Onboarding

```javascript
// Create employee onboarding with background check
const employeeOnboarding = await createOnboardingPage({
  pageTitle: 'New Employee Onboarding',
  description: 'Complete your pre-employment verification',
  verifications: ['identity', 'address', 'employment', 'background', 'qualifications'],
  forms: ['emergency-contact', 'bank-details'],
  backgroundCheckCostSplit: 'merchant'
});
```

### Use Case 4: Quick Lead Capture

```javascript
// Create simple lead capture page
const leadCapture = await createOnboardingPage({
  pageTitle: 'Get a Quote',
  description: 'Tell us about your needs',
  verifications: [], // No verifications
  leadCapture: {
    fields: ['fullName', 'emailAddress', 'phoneNumber', 'companyName', 'industry']
  },
  destination: 'https://mycompany.com/thank-you'
});
```

## Best Practices

### 1. Page Design

- **Clear Title** - Use descriptive, action-oriented titles
- **Brief Description** - Explain what the page is for and what's required
- **Appropriate Verifications** - Only request necessary verifications
- **Logical Flow** - Lead capture → Verifications → Forms → Submit
- **Brand Consistency** - Match your company's visual identity

### 2. Lead Capture Forms

- **Minimal Fields** - Only collect essential information upfront
- **Progressive Disclosure** - Collect detailed info after initial capture
- **Field Validation** - Use appropriate validation rules
- **Clear Labels** - Use simple, unambiguous field labels
- **Help Text** - Provide descriptions for complex fields

### 3. Verification Selection

- **Risk-Based** - Higher risk scenarios require more verifications
- **Cost-Aware** - Each verification type has associated costs
- **Time Consideration** - More verifications = longer completion time
- **Compliance** - Ensure verifications meet regulatory requirements

### 4. Custom Fields

- **Reusable Design** - Create fields that can be used across pages
- **Consistent Naming** - Use clear, consistent field IDs
- **Appropriate Types** - Choose correct input type for data
- **Validation Rules** - Set appropriate constraints

### 5. Submission Management

- **Regular Monitoring** - Check submissions daily
- **Status Updates** - Keep customers informed of progress
- **Follow-up** - Contact customers with incomplete submissions
- **Data Export** - Export submissions for record-keeping

### 6. Page URL Management

- **Memorable IDs** - Use descriptive page IDs when customizing
- **URL Stability** - Avoid changing page IDs frequently
- **Redirect Old URLs** - System tracks old page IDs for redirects
- **Short Links** - Use URL shortener for marketing materials

## Error Codes

| Status Code | Description |
|------------|-------------|
| 200 | Success |
| 400 | Bad Request (validation error) |
| 404 | Not Found (page/field not found) |
| 500 | Internal Server Error |

## Rate Limits

- **Create Page**: 50 per hour per organisation
- **Update Page**: 200 per hour per organisation
- **List Pages**: 500 per hour per organisation
- **Get Submissions**: 500 per hour per organisation
- **Public Lead Capture**: 1000 per hour (across all customers)

## File Upload Limits

- **Logo File Size**: 5 MB maximum
- **Logo File Types**: PNG, JPG, JPEG
- **Recommended Dimensions**: 400x400px minimum, square aspect ratio
- **Storage**: AWS S3 public bucket

## Verification Types Supported

| Type | Description | Typical Use Case |
|------|-------------|------------------|
| `identity` | ID document + biometric | KYC, onboarding |
| `address` | Proof of address | Residency verification |
| `income` | Income verification | Loan applications |
| `employment` | Employment history | Background checks |
| `qualifications` | Education/certifications | Professional onboarding |
| `references` | Reference checks | Rental, employment |
| `background` | Criminal records | Employee screening |

## Related Documentation

- [Verification Links API](./verification-links/verification-links-api.md)
- [Verification Requests API](./verification-requests.md)
- [Merchant Updates API](./merchant-updates.md)
- [Identity Verification API](./identity/identity-api.md)
- [Address Verification API](./address/address-api.md)
- [API Overview](./README.md)

