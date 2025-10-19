# Envelope Templates API

## Overview

The Envelope Templates API allows merchants to create reusable envelope templates that contain multiple document templates. Envelope templates enable quick creation of document packages by defining the structure once and instantiating it multiple times with different signers.

**Base Path**: `/api/v1/merchant/signatures/envelope-templates`

**Authentication**: Merchant JWT token in Authorization header

## Key Concepts

### What is an Envelope Template?
An envelope template is a reusable blueprint for creating envelopes (document packages). It contains:
- Multiple document templates
- Shared roles across all documents
- Consistent configuration
- Predefined structure and ordering

### Envelope Templates vs Envelopes
| Feature | Envelope | Envelope Template |
|---------|----------|-------------------|
| **Documents** | Specific documents with signers | Document templates with roles |
| **Usage** | Single-use | Reusable, create multiple envelopes |
| **Signers** | Actual people | Generic roles |
| **Status** | Draft, sent, completed | Always reusable |

### Use Cases
- **Employee Onboarding**: Employment contract + NDA + handbook acknowledgment
- **Vendor Setup**: Service agreement + insurance certificate + W9
- **Client Onboarding**: Engagement letter + privacy policy + payment authorisation
- **Loan Packages**: Loan agreement + promissory note + security agreement
- **Real Estate**: Purchase agreement + disclosure forms + inspection addendum

## Endpoints

### 1. List Envelope Templates

Retrieve a paginated list of envelope templates with filtering and sorting.

**Endpoint**: `GET /api/v1/merchant/signatures/envelope-templates`

**Query Parameters**:
- `search` (string, optional) - Search by title, description, or tags
- `category` (string, optional) - Filter by category
- `tags` (string, optional) - Comma-separated list of tags to filter
- `page` (number, optional) - Page number (default: 1)
- `limit` (number, optional) - Items per page (default: 20, max: 100)
- `sortBy` (string, optional) - Sort field (default: "createdAt")
- `sortOrder` (string, optional) - Sort order: "asc" or "desc" (default: "desc")

**Example Request**:
```
GET /api/v1/merchant/signatures/envelope-templates?search=employee&category=HR&page=1&limit=20&sortBy=usageCount&sortOrder=desc
```

**Success Response** (200):
```json
{
  "records": [
    {
      "_id": "507f1f77bcf86cd799439090",
      "title": "Employee Onboarding Package",
      "category": "HR",
      "tags": ["onboarding", "employment", "hr"],
      "documentTemplates": [
        {
          "templateId": "507f1f77bcf86cd799439070",
          "title": "Employment Contract",
          "order": 1
        },
        {
          "templateId": "507f1f77bcf86cd799439071",
          "title": "Non-Disclosure Agreement",
          "order": 2
        },
        {
          "templateId": "507f1f77bcf86cd799439072",
          "title": "Employee Handbook Acknowledgment",
          "order": 3
        }
      ],
      "usageCount": 125,
      "lastUsedAt": "2025-10-19T10:00:00Z",
      "createdAt": "2025-01-15T09:00:00Z"
    }
  ],
  "columns": [
    {
      "header": "Template",
      "path": "title",
      "type": "link",
      "urlPrefix": "/portal/documents/envelopes/templates/{id}/edit",
      "sortable": true
    },
    {
      "header": "Category",
      "path": "category",
      "type": "text",
      "sortable": true
    },
    {
      "header": "Tags",
      "path": "tags",
      "type": "tags"
    },
    {
      "header": "Documents",
      "path": "documentTemplates",
      "type": "count"
    },
    {
      "header": "Usage",
      "path": "usageCount",
      "type": "number",
      "sortable": true
    }
  ],
  "filters": [
    {
      "name": "category",
      "label": "Category",
      "type": "select",
      "options": []
    },
    {
      "name": "tags",
      "label": "Tags",
      "type": "multi-select",
      "options": []
    }
  ],
  "appliedFilters": {
    "search": "employee",
    "category": "HR",
    "page": 1,
    "limit": 20,
    "sortBy": "usageCount",
    "sortOrder": "desc"
  },
  "paging": {
    "recordCount": 8,
    "pageCount": 1,
    "currentPage": 1
  },
  "message": "Envelope templates retrieved successfully"
}
```

**Sortable Fields**:
- `title` - Template name
- `category` - Template category
- `createdAt` - Creation date
- `lastUsedAt` - Last usage date
- `usageCount` - Number of times used

---

### 2. Get Envelope Template by ID

Retrieve a specific envelope template with all its details.

**Endpoint**: `GET /api/v1/merchant/signatures/envelope-templates/:templateId`

**URL Parameters**:
- `templateId` (string, required) - Envelope template unique identifier

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "template": {
      "_id": "507f1f77bcf86cd799439090",
      "title": "Employee Onboarding Package",
      "description": "Complete onboarding package for new employees",
      "category": "HR",
      "tags": ["onboarding", "employment", "hr"],
      "documentTemplates": [
        {
          "templateId": "507f1f77bcf86cd799439070",
          "title": "Employment Contract",
          "order": 1,
          "required": true
        },
        {
          "templateId": "507f1f77bcf86cd799439071",
          "title": "Non-Disclosure Agreement",
          "order": 2,
          "required": true
        },
        {
          "templateId": "507f1f77bcf86cd799439072",
          "title": "Employee Handbook Acknowledgment",
          "order": 3,
          "required": true
        }
      ],
      "configuration": {
        "enforceSigningOrder": false,
        "requireIdVerification": false,
        "useDigitalSignature": true,
        "expiration": {
          "type": "relative",
          "relativeDays": 30
        }
      },
      "usageCount": 125,
      "lastUsedAt": "2025-10-19T10:00:00Z",
      "createdAt": "2025-01-15T09:00:00Z",
      "updatedAt": "2025-10-19T10:00:00Z",
      "clientId": "507f1f77bcf86cd799439030",
      "organisationId": "507f1f77bcf86cd799439031"
    },
    "documentTemplates": [
      {
        "_id": "507f1f77bcf86cd799439070",
        "title": "Employment Contract",
        "roles": [...]
      },
      {
        "_id": "507f1f77bcf86cd799439071",
        "title": "Non-Disclosure Agreement",
        "roles": [...]
      },
      {
        "_id": "507f1f77bcf86cd799439072",
        "title": "Employee Handbook Acknowledgment",
        "roles": [...]
      }
    ]
  },
  "message": "Envelope template retrieved successfully"
}
```

**Error Response** (404):
```json
{
  "error": true,
  "message": "Envelope template not found"
}
```

---

### 3. Create Envelope Template

Create a new envelope template.

**Endpoint**: `POST /api/v1/merchant/signatures/envelope-templates`

**Request Body**:
```json
{
  "title": "Employee Onboarding Package",
  "description": "Complete onboarding package for new employees",
  "documentTemplates": [
    {
      "templateId": "507f1f77bcf86cd799439070",
      "order": 1,
      "required": true
    },
    {
      "templateId": "507f1f77bcf86cd799439071",
      "order": 2,
      "required": true
    }
  ],
  "tags": ["onboarding", "employment"],
  "category": "HR",
  "configuration": {
    "enforceSigningOrder": false,
    "requireIdVerification": false,
    "useDigitalSignature": true,
    "expiration": {
      "type": "relative",
      "relativeDays": 30
    }
  }
}
```

**Request Parameters**:
- `title` (string, required) - Envelope template title
- `description` (string, optional) - Template description
- `documentTemplates` (array, required) - Array of document templates
  - `templateId` (string, required) - Document template ID
  - `order` (number, required) - Display order
  - `required` (boolean, optional) - Whether document is required (default: true)
- `tags` (array, optional) - Array of tags for organisation
- `category` (string, optional) - Template category
- `configuration` (object, optional) - Envelope configuration

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "template": {
      "_id": "507f1f77bcf86cd799439090",
      "title": "Employee Onboarding Package",
      "description": "Complete onboarding package for new employees",
      "documentTemplates": [...],
      "tags": ["onboarding", "employment"],
      "category": "HR",
      "usageCount": 0,
      "createdAt": "2025-10-19T10:00:00Z"
    }
  },
  "message": "Envelope template created successfully"
}
```

**Error Responses**:

400 Bad Request - No title:
```json
{
  "error": true,
  "message": "Envelope template title is required"
}
```

400 Bad Request - No documents:
```json
{
  "error": true,
  "message": "At least one document template is required"
}
```

**Notes**:
- Must include at least one document template
- Document templates must exist and belong to the organisation
- Document template IDs are validated during creation

---

### 4. Save Envelope Template

Update an existing envelope template.

**Endpoint**: `POST /api/v1/merchant/signatures/envelope-templates/:templateId/save`

**URL Parameters**:
- `templateId` (string, required) - Envelope template ID

**Request Body**:
```json
{
  "title": "Updated Employee Onboarding Package",
  "description": "Updated description",
  "documentTemplates": [
    {
      "templateId": "507f1f77bcf86cd799439070",
      "order": 1,
      "required": true
    },
    {
      "templateId": "507f1f77bcf86cd799439071",
      "order": 2,
      "required": true
    },
    {
      "templateId": "507f1f77bcf86cd799439073",
      "order": 3,
      "required": false
    }
  ],
  "tags": ["onboarding", "employment", "updated"],
  "category": "HR",
  "configuration": {
    "enforceSigningOrder": true,
    "requireIdVerification": false,
    "useDigitalSignature": true
  }
}
```

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "template": {
      "_id": "507f1f77bcf86cd799439090",
      "title": "Updated Employee Onboarding Package",
      ...
    }
  },
  "message": "Envelope template saved successfully"
}
```

**Notes**:
- All provided fields are updated
- Document templates array is completely replaced (not merged)
- Usage count and last used date are preserved

---

### 5. Duplicate Envelope Template

Create a copy of an existing envelope template.

**Endpoint**: `POST /api/v1/merchant/signatures/envelope-templates/:templateId/duplicate`

**URL Parameters**:
- `templateId` (string, required) - Source envelope template ID

**Request Body** (optional):
```json
{
  "title": "Employee Onboarding Package (Copy)"
}
```

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "templateId": "507f1f77bcf86cd799439091",
    "title": "Employee Onboarding Package (Copy)",
    "originalTemplateId": "507f1f77bcf86cd799439090",
    "documentTemplatesCount": 3
  },
  "message": "Envelope template duplicated successfully"
}
```

**Notes**:
- Creates exact copy with all document templates and configuration
- Document template references are preserved (not duplicated)
- Usage count reset to 0
- Title appended with " (Copy)" if not provided

---

### 6. Instantiate Envelope Template (Create Envelope from Template)

Create a new envelope with documents from an envelope template.

**Endpoint**: `POST /api/v1/merchant/signatures/envelope-templates/:templateId/instantiate`

**URL Parameters**:
- `templateId` (string, required) - Envelope template ID

**Request Body**:
```json
{
  "name": "John Smith - Onboarding Package",
  "signingParties": [
    {
      "id": "signer_1",
      "roleId": "employee_role",
      "name": "John Smith",
      "email": "john@example.com",
      "role": "Software Engineer",
      "order": 1
    },
    {
      "id": "signer_2",
      "roleId": "manager_role",
      "name": "Jane Manager",
      "email": "jane@company.com",
      "role": "Engineering Manager",
      "order": 2
    }
  ],
  "customConfiguration": {
    "expiration": {
      "type": "fixed",
      "fixedDate": "2025-11-30"
    }
  }
}
```

**Request Parameters**:
- `name` (string, required) - Envelope name
- `signingParties` (array, required) - Array of actual signers
  - `id` (string, required) - Unique signer identifier
  - `roleId` (string, required) - Role from templates this signer fills
  - `name` (string, required) - Signer's full name
  - `email` (string, required) - Signer's email
  - `role` (string, optional) - Signer's role/title
  - `order` (number, required) - Signing order
- `customConfiguration` (object, optional) - Override template configuration

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "envelopeId": "507f1f77bcf86cd799439040",
    "envelope": {
      "_id": "507f1f77bcf86cd799439040",
      "name": "John Smith - Onboarding Package",
      "status": "draft",
      "createdAt": "2025-10-19T10:00:00Z"
    },
    "documents": [
      {
        "_id": "507f1f77bcf86cd799439011",
        "title": "Employment Contract",
        "status": "draft"
      },
      {
        "_id": "507f1f77bcf86cd799439012",
        "title": "Non-Disclosure Agreement",
        "status": "draft"
      },
      {
        "_id": "507f1f77bcf86cd799439013",
        "title": "Employee Handbook Acknowledgment",
        "status": "draft"
      }
    ],
    "documentsCreated": 3
  },
  "message": "Envelope created from template successfully"
}
```

**Process**:
1. Creates new envelope with provided name
2. Instantiates each document template with provided signers
3. Associates all documents with the envelope
4. Applies configuration (template config + custom overrides)
5. Updates template usage statistics

**Error Responses**:

400 Bad Request - No name:
```json
{
  "error": true,
  "message": "Envelope name is required"
}
```

400 Bad Request - No signers:
```json
{
  "error": true,
  "message": "At least one signing party is required"
}
```

404 Not Found:
```json
{
  "error": true,
  "message": "Envelope template not found"
}
```

**Automatic Updates**:
- Template `usageCount` is incremented
- Template `lastUsedAt` is updated
- Each document template's usage count is incremented
- Created envelope is in `draft` status

**Notes**:
- All document templates in the envelope template are instantiated
- Signers are applied consistently across all documents
- Role mapping must work for all document templates
- Created documents are automatically added to the envelope

---

### 7. Delete Envelope Template

Permanently delete an envelope template.

**Endpoint**: `POST /api/v1/merchant/signatures/envelope-templates/:templateId/delete`

**URL Parameters**:
- `templateId` (string, required) - Envelope template ID

**Success Response** (200):
```json
{
  "success": true,
  "message": "Envelope template deleted successfully"
}
```

**Error Response** (404):
```json
{
  "error": true,
  "message": "Envelope template not found"
}
```

**Notes**:
- Permanently deletes envelope template from database
- Document templates referenced by this envelope template are NOT deleted
- Envelopes created from this template are NOT affected
- Cannot be undone

---

## Document Template Configuration

Each document template in an envelope template can have specific settings.

### Document Template Object
```json
{
  "templateId": "507f1f77bcf86cd799439070",
  "title": "Employment Contract",
  "order": 1,
  "required": true,
  "customConfiguration": {
    "useDigitalSignature": true
  }
}
```

**Fields**:
- `templateId` (string, required) - Document template ID
- `title` (string, optional) - Display title (defaults to template title)
- `order` (number, required) - Display/signing order
- `required` (boolean, optional) - Whether document is required (default: true)
- `customConfiguration` (object, optional) - Document-specific configuration overrides

---

## Envelope Template Configuration

Envelope templates support comprehensive configuration that applies to all documents.

```json
{
  "configuration": {
    "enforceSigningOrder": true,
    "requireIdVerification": false,
    "useDigitalSignature": true,
    "allowComments": true,
    "allowDecline": false,
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

**Configuration is inherited by all documents in envelopes created from the template.**

When instantiating, you can override specific configuration values:
```json
{
  "customConfiguration": {
    "expiration": {
      "type": "fixed",
      "fixedDate": "2025-12-31"
    }
  }
}
```

---

## Role Consistency Across Documents

When creating envelope templates, ensure roles are consistent across all document templates:

### Example: Consistent Roles

**Document Template 1: Employment Contract**
- Role: `employee_role` (Employee)
- Role: `manager_role` (Manager)

**Document Template 2: NDA**
- Role: `employee_role` (Employee)
- Role: `manager_role` (Manager)

**Document Template 3: Handbook Acknowledgment**
- Role: `employee_role` (Employee)

**Result**: When instantiating, map:
- `employee_role` → John Smith
- `manager_role` → Jane Manager

All documents will have the correct signers assigned.

### Example: Inconsistent Roles (Problematic)

**Document Template 1: Employment Contract**
- Role: `employee` (Employee)

**Document Template 2: NDA**
- Role: `new_hire` (New Hire)

**Problem**: When instantiating, you'd need to map both `employee` and `new_hire` to the same person, which is confusing.

**Solution**: Use consistent role IDs and names across related templates.

---

## Complete Workflow Example

### Creating and Using an Envelope Template

```javascript
// 1. Create individual document templates (if not already created)
// See Document Templates API documentation

// 2. Create envelope template
const createResponse = await fetch('/api/v1/merchant/signatures/envelope-templates', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    title: 'Employee Onboarding Package',
    description: 'Complete onboarding for new employees',
    documentTemplates: [
      {
        templateId: 'employment_contract_template_id',
        order: 1,
        required: true
      },
      {
        templateId: 'nda_template_id',
        order: 2,
        required: true
      },
      {
        templateId: 'handbook_template_id',
        order: 3,
        required: true
      }
    ],
    tags: ['onboarding', 'hr', 'employment'],
    category: 'HR',
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
const { data: { template } } = await createResponse.json();

// 3. Use template to create envelope with documents
const instantiateResponse = await fetch(
  `/api/v1/merchant/signatures/envelope-templates/${template._id}/instantiate`,
  {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      name: 'John Smith - Onboarding',
      signingParties: [
        {
          id: 'signer_1',
          roleId: 'employee_role',
          name: 'John Smith',
          email: 'john@example.com',
          role: 'Software Engineer',
          order: 1
        },
        {
          id: 'signer_2',
          roleId: 'manager_role',
          name: 'Jane Manager',
          email: 'jane@company.com',
          role: 'Engineering Manager',
          order: 2
        }
      ]
    })
  }
);
const { data: { envelopeId, documents } } = await instantiateResponse.json();

console.log(`Created envelope ${envelopeId} with ${documents.length} documents`);

// 4. Send envelope (see Envelopes API documentation)
await fetch(`/api/v1/merchant/signatures/envelopes/${envelopeId}/send`, {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    configuration: {
      enforceSigningOrder: false,
      requireIdVerification: false,
      useDigitalSignature: true,
      expiration: {
        type: 'relative',
        relativeDays: 30
      }
    },
    message: 'Welcome! Please review and sign these onboarding documents.'
  })
});
```

---

## Best Practices

### 1. Template Organisation
- Use clear, descriptive envelope template names
- Group related document templates together
- Maintain consistent role definitions across document templates
- Document the purpose in the description field

### 2. Document Selection
- Include 3-7 documents per envelope template (optimal)
- Order documents logically (most important first)
- Mark critical documents as required
- Consider signing complexity when grouping

### 3. Role Management
- Use consistent role IDs across all document templates in the envelope
- Document role expectations in descriptions
- Limit to 2-5 roles per envelope template
- Test role mappings before production use

### 4. Configuration
- Set sensible default expiration (15-30 days)
- Enable digital signatures for legally binding documents
- Configure appropriate reminder schedules
- Consider ID verification requirements

### 5. Template Maintenance
- Review template usage statistics quarterly
- Update outdated templates
- Remove unused templates
- Version template names when making major changes

### 6. Tagging Strategy
- Use consistent tags across related templates
- Include functional tags (e.g., "onboarding", "vendor")
- Include departmental tags (e.g., "hr", "legal")
- Limit to 3-5 tags per template

---

## Common Use Cases

### HR/Employment
```json
{
  "title": "New Hire Package",
  "documentTemplates": [
    "Employment Contract",
    "NDA",
    "Employee Handbook Acknowledgment",
    "Direct Deposit Authorisation",
    "Emergency Contact Form"
  ]
}
```

### Vendor Onboarding
```json
{
  "title": "Vendor Setup Package",
  "documentTemplates": [
    "Service Agreement",
    "Insurance Certificate",
    "W9 Form",
    "Banking Information",
    "Background Check Consent"
  ]
}
```

### Client Onboarding
```json
{
  "title": "Client Engagement Package",
  "documentTemplates": [
    "Engagement Letter",
    "Privacy Policy",
    "Payment Authorisation",
    "Terms of Service"
  ]
}
```

### Real Estate Transaction
```json
{
  "title": "Home Purchase Package",
  "documentTemplates": [
    "Purchase Agreement",
    "Property Disclosure",
    "Inspection Addendum",
    "Financing Contingency",
    "Closing Documents"
  ]
}
```

---

## Error Codes

| Status Code | Description |
|------------|-------------|
| 200 | Success |
| 400 | Bad Request (validation error) |
| 401 | Unauthorised (invalid/missing token) |
| 403 | Forbidden (insufficient permissions) |
| 404 | Not Found (envelope template not found) |
| 500 | Internal Server Error |

## Rate Limits

- **Envelope Template Creation**: 25 per hour
- **Envelope Template Instantiation**: 200 per hour
- **API Calls**: 1000 per hour per organisation

## Related Documentation

- [Document Templates API](./document-templates.md)
- [Envelopes API](./envelopes.md)
- [Merchant Signature Documents API](./merchant-signature-documents.md)
- [API Overview](./README.md)

