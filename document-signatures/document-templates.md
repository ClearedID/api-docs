# Document Templates API

## Overview

The Document Templates API allows merchants to create reusable document templates with pre-defined roles, fields, and configurations. Templates enable quick document creation by defining the structure once and instantiating it multiple times with different signers.

**Base Path**: `/api/v1/merchant/signatures/templates`

**Authentication**: Merchant JWT token in Authorization header

## Key Concepts

### What is a Document Template?
A document template is a reusable blueprint for creating signature documents. Instead of defining specific signers, templates use **roles** that are mapped to actual signers when instantiating the template.

### Templates vs Documents
| Feature | Document | Template |
|---------|----------|----------|
| **Signers** | Specific people with names/emails | Generic roles (e.g., "Employee", "Manager") |
| **Usage** | Single-use, sent once | Reusable, create multiple documents |
| **Fields** | Assigned to specific signers | Assigned to roles |
| **Status** | Draft, sent, completed | Always reusable |

### Use Cases
- **Recurring Agreements**: Employment contracts, NDAs, vendor agreements
- **Standardized Forms**: Application forms, consent forms, waivers
- **Multi-location Operations**: Same document used across multiple locations
- **Template Library**: Build a library of commonly used documents

## Endpoints

### 1. List Templates

Retrieve a paginated list of document templates with filtering and sorting.

**Endpoint**: `GET /api/v1/merchant/signatures/templates`

**Query Parameters**:
- `query` (string, JSON encoded) - Search/filter/sort parameters:
  ```json
  {
    "keywords": "employment",
    "page": 1,
    "sortField": "usageCount",
    "sortOrder": "desc",
    "filters": {
      "category": ["HR"],
      "tags": ["employment", "contracts"],
      "createdAt_start": "2025-01-01",
      "createdAt_end": "2025-12-31"
    }
  }
  ```

**Success Response** (200):
```json
{
  "records": [
    {
      "_id": "507f1f77bcf86cd799439070",
      "title": "Employment Contract Template",
      "category": "HR",
      "tags": ["employment", "contracts"],
      "roles": [
        {
          "id": "role_1",
          "name": "Employee",
          "order": 1
        },
        {
          "id": "role_2",
          "name": "Employer",
          "order": 2
        }
      ],
      "usageCount": 45,
      "lastUsedAt": "2025-10-19T10:00:00Z",
      "createdAt": "2025-01-15T09:00:00Z"
    }
  ],
  "columns": [
    {
      "header": "Template",
      "path": "title",
      "type": "link",
      "urlPrefix": "/portal/documents/templates/{id}/edit",
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
      "header": "Roles",
      "path": "roles",
      "type": "count"
    },
    {
      "header": "Usage Count",
      "path": "usageCount",
      "type": "number",
      "sortable": true
    }
  ],
  "filters": [
    {
      "name": "tags",
      "label": "Tags",
      "type": "multi-select",
      "options": ["employment", "contracts", "nda", "vendor"]
    },
    {
      "name": "category",
      "label": "Category",
      "type": "multi-select",
      "options": ["HR", "Legal", "Sales", "Operations"]
    },
    {
      "name": "createdAt",
      "label": "Created At",
      "type": "date-range"
    }
  ],
  "appliedFilters": {},
  "paging": {
    "recordCount": 12,
    "pageCount": 2,
    "currentPage": 1
  }
}
```

**Sortable Fields**:
- `title` - Template name
- `category` - Template category
- `usageCount` - Number of times used
- `lastUsedAt` - Last usage date
- `createdAt` - Creation date

---

### 2. Get Template by ID

Retrieve a specific template with all its details.

**Endpoint**: `GET /api/v1/merchant/signatures/templates/:templateId`

**URL Parameters**:
- `templateId` (string, required) - Template unique identifier

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "template": {
      "_id": "507f1f77bcf86cd799439070",
      "title": "Employment Contract Template",
      "description": "Standard employment agreement template",
      "category": "HR",
      "tags": ["employment", "contracts", "onboarding"],
      "roles": [
        {
          "id": "role_1",
          "name": "Employee",
          "order": 1,
          "enforceOrder": false,
          "requireIdVerification": false,
          "description": "The new employee"
        },
        {
          "id": "role_2",
          "name": "Employer",
          "order": 2,
          "enforceOrder": false,
          "requireIdVerification": false,
          "description": "Company representative"
        }
      ],
      "fields": [
        {
          "id": "signature_1",
          "type": "signature",
          "label": "Employee Signature",
          "required": true,
          "assignedToRole": "role_1",
          "pageNumber": 1,
          "x": 100,
          "y": 500,
          "width": 200,
          "height": 50
        },
        {
          "id": "date_1",
          "type": "date",
          "label": "Date",
          "required": true,
          "assignedToRole": "role_1",
          "pageNumber": 1,
          "x": 350,
          "y": 500,
          "width": 100,
          "height": 30
        }
      ],
      "totalPages": 3,
      "pageImages": [...],
      "configuration": {
        "useDigitalSignature": true,
        "enforceSigningOrder": false
      },
      "usageCount": 45,
      "lastUsedAt": "2025-10-19T10:00:00Z",
      "createdAt": "2025-01-15T09:00:00Z",
      "updatedAt": "2025-10-19T10:00:00Z"
    },
    "roles": [...],
    "fields": [...]
  },
  "message": "Template retrieved successfully"
}
```

**Error Response** (404):
```json
{
  "error": true,
  "message": "Template not found"
}
```

---

### 3. Create Template

Create a new empty template.

**Endpoint**: `POST /api/v1/merchant/signatures/templates`

**Request Body**:
```json
{
  "title": "Employment Contract Template",
  "description": "Standard employment agreement",
  "roles": [
    {
      "id": "role_1",
      "name": "Employee",
      "order": 1,
      "enforceOrder": false,
      "requireIdVerification": false,
      "description": "The new employee"
    },
    {
      "id": "role_2",
      "name": "Employer",
      "order": 2,
      "enforceOrder": false,
      "requireIdVerification": false,
      "description": "Company representative"
    }
  ],
  "fields": [],
  "tags": ["employment", "contracts"],
  "category": "HR",
  "configuration": {
    "useDigitalSignature": true,
    "enforceSigningOrder": false
  }
}
```

**Request Parameters**:
- `title` (string, required) - Template title
- `description` (string, optional) - Template description
- `roles` (array, required) - Array of roles
  - `id` (string, required) - Unique role identifier
  - `name` (string, required) - Role name (e.g., "Employee", "Manager")
  - `order` (number, required) - Signing order
  - `enforceOrder` (boolean, optional) - Enforce signing order
  - `requireIdVerification` (boolean, optional) - Require ID verification
  - `description` (string, optional) - Role description
- `fields` (array, optional) - Array of form fields (can add later)
- `tags` (array, optional) - Array of tags for organization
- `category` (string, optional) - Template category
- `configuration` (object, optional) - Template configuration

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "template": {
      "_id": "507f1f77bcf86cd799439070",
      "title": "Employment Contract Template",
      "description": "Standard employment agreement",
      "roles": [...],
      "fields": [],
      "createdAt": "2025-10-19T10:00:00Z"
    }
  },
  "message": "Template created successfully"
}
```

**Error Response** (400):
```json
{
  "error": true,
  "message": "Template title is required"
}
```

---

### 4. Save Template

Update an existing template's details, roles, and fields.

**Endpoint**: `POST /api/v1/merchant/signatures/templates/:templateId/save`

**URL Parameters**:
- `templateId` (string, required) - Template ID

**Request Body**:
```json
{
  "title": "Updated Employment Contract Template",
  "description": "Updated description",
  "roles": [...],
  "fields": [...],
  "tags": ["employment", "contracts", "updated"],
  "category": "HR",
  "configuration": {
    "useDigitalSignature": true,
    "enforceSigningOrder": false
  }
}
```

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "template": {
      "_id": "507f1f77bcf86cd799439070",
      "title": "Updated Employment Contract Template",
      ...
    }
  },
  "message": "Template saved successfully"
}
```

**Notes**:
- Updates all provided fields
- Roles and fields are completely replaced (not merged)
- Usage count and last used date are preserved

---

### 5. Duplicate Template

Create a copy of an existing template.

**Endpoint**: `POST /api/v1/merchant/signatures/templates/:templateId/duplicate`

**URL Parameters**:
- `templateId` (string, required) - Source template ID

**Request Body** (optional):
```json
{
  "title": "New Template Name (Copy)"
}
```

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "templateId": "507f1f77bcf86cd799439071",
    "title": "Employment Contract Template (Copy)",
    "originalTemplateId": "507f1f77bcf86cd799439070"
  },
  "message": "Template duplicated successfully"
}
```

**Notes**:
- Creates exact copy with all roles, fields, and configuration
- PDF file is copied in S3
- Page images are reused
- Usage count reset to 0
- Title appended with " (Copy)" if not provided

---

### 6. Create Template from Document

Convert an existing document into a reusable template.

**Endpoint**: `POST /api/v1/merchant/signatures/templates/from-document/:documentId`

**URL Parameters**:
- `documentId` (string, required) - Source document ID

**Request Body**:
```json
{
  "title": "Employment Contract Template",
  "description": "Template created from existing document",
  "tags": ["employment", "contracts"],
  "category": "HR"
}
```

**Request Parameters**:
- `title` (string, optional) - Template title (defaults to document title + " Template")
- `description` (string, optional) - Template description
- `tags` (array, optional) - Array of tags
- `category` (string, optional) - Template category

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "templateId": "507f1f77bcf86cd799439070",
    "title": "Employment Contract Template",
    "template": {
      "_id": "507f1f77bcf86cd799439070",
      "title": "Employment Contract Template",
      "roles": [
        {
          "id": "role_507f1f77bcf86cd799439012",
          "name": "Employee",
          "order": 1,
          "description": "Role for John Smith"
        }
      ],
      "fields": [
        {
          "id": "signature_1",
          "type": "signature",
          "assignedToRole": "role_507f1f77bcf86cd799439012",
          ...
        }
      ]
    }
  },
  "message": "Template created from document successfully"
}
```

**Conversion Process**:
1. **Signing Parties → Roles**: Each signer becomes a role
   - Signer name → Role description
   - Signer role → Role name
   - Signer order → Role order
2. **Fields**: All fields are copied
   - `assignedTo` (signer ID) → `assignedToRole` (role ID)
   - Field positions and properties preserved
3. **PDF & Images**: Reused from document
4. **Configuration**: Copied from document

**Notes**:
- Document must have at least one signer to create template
- Document must have been uploaded (have PDF file)
- Original document remains unchanged
- Template starts with usage count of 0

---

### 7. Instantiate Template (Create Document from Template)

Create a new document from a template by mapping roles to actual signers.

**Endpoint**: `POST /api/v1/merchant/signatures/templates/:templateId/instantiate`

**URL Parameters**:
- `templateId` (string, required) - Template ID

**Request Body**:
```json
{
  "title": "Employment Contract - John Smith",
  "signingParties": [
    {
      "id": "signer_1",
      "roleId": "role_1",
      "userId": "507f1f77bcf86cd799439080",
      "name": "John Smith",
      "email": "john@example.com",
      "role": "Employee",
      "order": 1,
      "enforceOrder": false,
      "requireIdVerification": false
    },
    {
      "id": "signer_2",
      "roleId": "role_2",
      "userId": "507f1f77bcf86cd799439081",
      "name": "Jane Manager",
      "email": "jane@company.com",
      "role": "Employer",
      "order": 2,
      "enforceOrder": false,
      "requireIdVerification": false
    }
  ]
}
```

**Request Parameters**:
- `title` (string, optional) - Document title (defaults to template title)
- `signingParties` (array, required) - Array of actual signers
  - `id` (string, required) - Unique signer identifier
  - `roleId` (string, required) - Role from template this signer fills
  - `userId` (string, optional) - User ID if known
  - `name` (string, required) - Signer's full name
  - `email` (string, required) - Signer's email
  - `role` (string, optional) - Signer's role/title
  - `order` (number, required) - Signing order
  - `enforceOrder` (boolean, optional) - Enforce signing order
  - `requireIdVerification` (boolean, optional) - Require ID verification

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "documentId": "507f1f77bcf86cd799439011",
    "document": {
      "_id": "507f1f77bcf86cd799439011",
      "title": "Employment Contract - John Smith",
      "status": "draft",
      "signingParties": [
        {
          "id": "signer_1",
          "name": "John Smith",
          "email": "john@example.com",
          "status": "pending",
          ...
        }
      ],
      "fields": [
        {
          "id": "signature_1",
          "type": "signature",
          "assignedTo": "signer_1",
          ...
        }
      ]
    }
  },
  "message": "Document created from template successfully"
}
```

**Field Mapping**:
- Template fields with `assignedToRole: "role_1"` 
- → Document fields with `assignedTo: "signer_1"` (based on role mapping)

**Error Responses**:

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
  "message": "Template not found"
}
```

**Automatic Updates**:
- Template `usageCount` is incremented
- Template `lastUsedAt` is updated to current timestamp
- Created document is in `draft` status and ready to be configured/sent

**Notes**:
- Must provide signer for each role in template
- Role-to-signer mapping is done via `roleId` field
- If role mapping fails, attempts to map by order
- New document inherits template's configuration
- PDF and page images are reused from template

---

### 8. Delete Template

Permanently delete a template.

**Endpoint**: `POST /api/v1/merchant/signatures/templates/:templateId/delete`

**URL Parameters**:
- `templateId` (string, required) - Template ID

**Success Response** (200):
```json
{
  "success": true,
  "message": "Template deleted successfully"
}
```

**Error Response** (404):
```json
{
  "error": true,
  "message": "Template not found"
}
```

**Notes**:
- Permanently deletes template from database
- PDF file remains in S3 (for documents created from template)
- Documents created from this template are NOT affected
- Cannot be undone

---

### 9. Upload PDF to Template

Upload or replace a PDF file for a template.

**Endpoint**: `POST /api/v1/merchant/signatures/templates/:templateId/upload`

**URL Parameters**:
- `templateId` (string, required) - Template ID

**Content-Type**: `multipart/form-data`

**Form Parameters**:
- `file` (file, required) - PDF file to upload

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "templateId": "507f1f77bcf86cd799439070",
    "fileKey": "signatures/templates/507f.../template.pdf",
    "totalPages": 3,
    "pageImages": [
      {
        "pageNumber": 1,
        "width": 612,
        "height": 792,
        "imageUrl": "https://s3.amazonaws.com/..."
      }
    ]
  },
  "message": "Template PDF uploaded successfully"
}
```

**Notes**:
- Maximum file size: 10MB
- Accepted format: PDF only
- Automatically generates page images
- Updates template's `fileKey`, `totalPages`, and `pageImages`
- Existing PDF is replaced

---

## Role Structure

Roles define the signing parties in a template without specifying actual people.

### Role Object
```json
{
  "id": "role_1",
  "name": "Employee",
  "order": 1,
  "enforceOrder": false,
  "requireIdVerification": false,
  "description": "The new employee who will sign the contract"
}
```

**Role Fields**:
- `id` (string, required) - Unique role identifier
- `name` (string, required) - Role name/title
- `order` (number, required) - Signing order
- `enforceOrder` (boolean, optional) - Whether to enforce this role's order
- `requireIdVerification` (boolean, optional) - Whether to require ID verification
- `description` (string, optional) - Role description for clarity

### Common Roles

**HR/Employment**:
- Employee
- Manager/Supervisor
- HR Representative
- Executive/Officer

**Legal**:
- Party A / Party B
- Client
- Service Provider
- Witness
- Notary

**Sales/Vendor**:
- Vendor
- Client/Customer
- Procurement Officer
- Finance Approver

**Real Estate**:
- Buyer
- Seller
- Agent/Broker
- Attorney

---

## Field Assignment in Templates

In templates, fields are assigned to **roles** instead of specific signers.

### Template Field
```json
{
  "id": "signature_1",
  "type": "signature",
  "label": "Employee Signature",
  "required": true,
  "assignedToRole": "role_1",
  "pageNumber": 1,
  "x": 100,
  "y": 500,
  "width": 200,
  "height": 50
}
```

### Document Field (After Instantiation)
```json
{
  "id": "signature_1",
  "type": "signature",
  "label": "Employee Signature",
  "required": true,
  "assignedTo": "signer_1",
  "pageNumber": 1,
  "x": 100,
  "y": 500,
  "width": 200,
  "height": 50
}
```

**Key Difference**: `assignedToRole` → `assignedTo`

---

## Template Configuration

Templates support the same configuration options as documents.

```json
{
  "configuration": {
    "useDigitalSignature": true,
    "enforceSigningOrder": true,
    "requireIdVerification": false,
    "allowComments": true,
    "allowDecline": true,
    "expiration": {
      "type": "relative",
      "relativeDays": 30
    }
  }
}
```

**Configuration is inherited by documents created from the template.**

---

## Template Organization

### Tags
Use tags to categorize and find templates easily:
```json
{
  "tags": ["employment", "contracts", "onboarding", "hr"]
}
```

### Categories
Use categories for high-level organization:
```json
{
  "category": "HR"
}
```

**Common Categories**:
- HR
- Legal
- Sales
- Operations
- Finance
- Real Estate
- Vendor Management

---

## Complete Workflow Example

### Creating and Using a Template

```javascript
// 1. Create template
const createResponse = await fetch('/api/v1/merchant/signatures/templates', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    title: 'Employment Contract Template',
    description: 'Standard employment agreement',
    roles: [
      {
        id: 'employee_role',
        name: 'Employee',
        order: 1,
        description: 'The new employee'
      },
      {
        id: 'manager_role',
        name: 'Manager',
        order: 2,
        description: 'Hiring manager'
      }
    ],
    tags: ['employment', 'hr'],
    category: 'HR',
    configuration: {
      useDigitalSignature: true,
      enforceSigningOrder: false
    }
  })
});
const { data: { template } } = await createResponse.json();

// 2. Upload PDF to template
const formData = new FormData();
formData.append('file', pdfFile);

await fetch(`/api/v1/merchant/signatures/templates/${template._id}/upload`, {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`
  },
  body: formData
});

// 3. Add fields to template (via save endpoint)
await fetch(`/api/v1/merchant/signatures/templates/${template._id}/save`, {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    fields: [
      {
        id: 'employee_signature',
        type: 'signature',
        label: 'Employee Signature',
        assignedToRole: 'employee_role',
        pageNumber: 1,
        x: 100,
        y: 500,
        width: 200,
        height: 50,
        required: true
      },
      {
        id: 'manager_signature',
        type: 'signature',
        label: 'Manager Signature',
        assignedToRole: 'manager_role',
        pageNumber: 1,
        x: 400,
        y: 500,
        width: 200,
        height: 50,
        required: true
      }
    ]
  })
});

// 4. Use template to create document
const instantiateResponse = await fetch(
  `/api/v1/merchant/signatures/templates/${template._id}/instantiate`,
  {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      title: 'Employment Contract - John Smith',
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
const { data: { documentId } } = await instantiateResponse.json();

// 5. Send document (see Document API for details)
// ...
```

---

## Best Practices

### 1. Template Design
- Use clear, descriptive role names
- Limit to 2-5 roles per template
- Include role descriptions for clarity
- Test template with sample signers before production use

### 2. Field Organization
- Group related fields together
- Use consistent field labeling
- Mark critical fields as required
- Position fields logically (reading order)

### 3. Template Library
- Create comprehensive template descriptions
- Use consistent tagging strategy
- Organize with meaningful categories
- Review and update templates quarterly

### 4. Role Mapping
- Always provide `roleId` when instantiating
- Verify all roles are mapped to signers
- Use consistent role IDs across similar templates

### 5. Versioning
- Duplicate template before major changes
- Include version or date in template name
- Document template changes in description

---

## Error Codes

| Status Code | Description |
|------------|-------------|
| 200 | Success |
| 400 | Bad Request (validation error) |
| 401 | Unauthorized (invalid/missing token) |
| 403 | Forbidden (insufficient permissions) |
| 404 | Not Found (template not found) |
| 500 | Internal Server Error |

## Rate Limits

- **Template Creation**: 50 per hour
- **Template Instantiation**: 500 per hour
- **API Calls**: 1000 per hour per organisation

## Related Documentation

- [Merchant Signature Documents API](./merchant-signature-documents.md)
- [Envelope Templates API](./envelope-templates.md)
- [Envelopes API](./envelopes.md)
- [API Overview](./README.md)

