# Merchant Updates API

## Overview

The Merchant Updates API provides a real-time notification system for merchants to receive updates about verification statuses, system announcements, and customer activities. Updates are delivered instantly when verifications are completed, requests are approved/denied, or system events occur.

**Base Path**: `/api/v1/merchant/updates`

**Authentication**: Merchant JWT token in Authorization header

## Key Features

- **Real-Time Updates** – Instant notifications for verification events
- **Unread Tracking** – Track which updates have been read
- **Filtering & Search** – Filter by type, status, priority
- **Pagination** – Efficient loading of large update lists
- **Update Types** – Verification, system, request, and custom updates
- **Priority Levels** – Normal, high, urgent for prioritisation
- **Action Links** – Direct links to relevant pages/actions
- **Custom Metadata** – Attach custom data to updates

## Update Types

### Verification Updates

Notifications about verification status changes:
- **Identity verification cleared** - Customer's identity verified
- **Address verification cleared** - Customer's address verified
- **Verification rejected** - Verification was rejected
- **Verification submitted** - Customer submitted documents
- **Verification expired** - Request expired without completion

**Example Update**:
```json
{
  "updateType": "verification",
  "status": "cleared",
  "title": "Identity Verification Cleared",
  "message": "John Smith's identity verification has been approved",
  "verificationType": "identity",
  "customer": {
    "name": "John Smith",
    "email": "john@example.com"
  },
  "priority": "normal",
  "actionRequired": true,
  "actionUrl": "/portal/identity/report/507f...",
  "actionText": "View Report"
}
```

### System Updates

Platform announcements and system notifications:
- **Maintenance notices** - Scheduled maintenance alerts
- **Feature announcements** - New features available
- **Policy changes** - Terms or policy updates
- **Service disruptions** - Downtime or issues
- **Security alerts** - Important security information

**Example Update**:
```json
{
  "updateType": "system",
  "title": "Scheduled Maintenance",
  "message": "The platform will be offline for maintenance on Oct 25, 2025 from 2-4 AM EST",
  "priority": "high",
  "category": "maintenance",
  "actionRequired": false
}
```

### Request Updates

Customer request status changes:
- **Request approved** - Customer approved sharing
- **Request denied** - Customer denied request
- **Request expired** - Request expired
- **Request withdrawn** - Merchant withdrew request

**Example Update**:
```json
{
  "updateType": "request",
  "title": "Verification Request Approved",
  "message": "Jane Doe approved your verification request",
  "requestId": "507f1f77bcf86cd799439013",
  "customer": {
    "name": "Jane Doe",
    "email": "jane@example.com"
  },
  "priority": "normal"
}
```

### Custom Updates

Merchant-created custom notifications:
- Application-specific events
- Workflow milestones
- Custom business logic triggers

## Priority Levels

| Priority | Description | Use Case |
|----------|-------------|----------|
| `normal` | Standard updates | Regular verification completions |
| `high` | Important updates | Verification rejections, approaching deadlines |
| `urgent` | Critical updates | Security alerts, service disruptions, compliance issues |

## Endpoints

### 1. List Updates

Retrieve paginated list of updates for the authenticated organisation.

**Endpoint**: `GET /api/v1/merchant/updates`

**Query Parameters**:
- `limit` (number, optional) - Results per page (default: 20, max: 100)
- `skip` (number, optional) - Pagination offset (default: 0)
- `updateType` (string, optional) - Filter by type: `verification`, `system`, `request`, `custom`
- `status` (string, optional) - Filter by status (depends on update type)
- `priority` (string, optional) - Filter by priority: `normal`, `high`, `urgent`
- `unreadOnly` (boolean, optional) - Show only unread updates (default: false)
- `search` (string, optional) - Search in title and message

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "updates": [
      {
        "_id": "507f1f77bcf86cd799439200",
        "organisationId": "507f1f77bcf86cd799439031",
        "updateType": "verification",
        "status": "cleared",
        "title": "Identity Verification Cleared",
        "message": "John Smith's identity verification has been approved",
        "description": "The identity verification for John Smith (passport A1234567) has been successfully verified and cleared by our operations team.",
        "verificationType": "identity",
        "verificationId": "507f1f77bcf86cd799439011",
        "customer": {
          "name": "John Smith",
          "firstName": "John",
          "lastName": "Smith",
          "email": "john@example.com",
          "phone": "+1876xxxxxxx",
          "userId": "507f1f77bcf86cd799439050"
        },
        "priority": "normal",
        "category": "verification",
        "tags": ["identity", "cleared"],
        "actionRequired": true,
        "actionUrl": "/portal/identity/report/507f1f77bcf86cd799439011",
        "actionText": "View Report",
        "metadata": {
          "documentType": "passport",
          "documentNumber": "A12****7",
          "verificationDuration": "2.5 hours"
        },
        "read": false,
        "readAt": null,
        "createdBy": {
          "system": true,
          "name": "Cleared System"
        },
        "createdAt": "2025-10-19T14:00:00Z"
      },
      {
        "_id": "507f1f77bcf86cd799439201",
        "organisationId": "507f1f77bcf86cd799439031",
        "updateType": "system",
        "title": "New Feature Available",
        "message": "Document signature templates are now available",
        "description": "You can now create reusable document templates to streamline your signing workflows.",
        "priority": "normal",
        "category": "feature",
        "tags": ["feature", "documents"],
        "actionRequired": false,
        "read": true,
        "readAt": "2025-10-19T15:00:00Z",
        "createdAt": "2025-10-19T10:00:00Z"
      }
    ],
    "unreadCount": 12,
    "pagination": {
      "limit": 20,
      "skip": 0,
      "hasMore": true
    }
  }
}
```

**Notes**:
- Returns updates for authenticated organisation only
- Sorted by creation date (newest first)
- Includes unread count
- Supports filtering by multiple criteria
- Search queries title and message fields

---

### 2. Mark Updates as Read

Mark one or more updates as read.

**Endpoint**: `POST /api/v1/merchant/updates/mark-read`

**Request Body**:
```json
{
  "updateIds": [
    "507f1f77bcf86cd799439200",
    "507f1f77bcf86cd799439201",
    "507f1f77bcf86cd799439202"
  ]
}
```

**Request Parameters**:
- `updateIds` (array, required) - Array of update MongoDB ObjectIds to mark as read

**Success Response** (200):
```json
{
  "success": true,
  "message": "Updates marked as read"
}
```

**Error Response** (400):
```json
{
  "success": false,
  "message": "updateIds array is required"
}
```

**Notes**:
- Only marks updates owned by authenticated organisation
- Sets `read: true` and `readAt: current timestamp`
- Decrements unread count
- Used to keep notification badge accurate

---

### 3. Get Unread Count

Get count of unread updates for the authenticated organisation.

**Endpoint**: `GET /api/v1/merchant/updates/unread-count`

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "unreadCount": 12
  }
}
```

**Notes**:
- Returns count of updates where `read: false`
- Used for notification badge
- Very fast query (uses index)

---

### 4. Create Custom Update

Create a custom update (for system/integration use).

**Endpoint**: `POST /api/v1/merchant/updates`

**Request Body**:
```json
{
  "updateType": "custom",
  "status": "completed",
  "title": "Background Check Completed",
  "message": "Background check for John Smith has been completed",
  "description": "The comprehensive background check including criminal records, credit history, and employment verification has been completed successfully.",
  "verificationId": "507f1f77bcf86cd799439011",
  "requestId": "507f1f77bcf86cd799439013",
  "userId": "507f1f77bcf86cd799439050",
  "customerName": "John Smith",
  "customerEmail": "john@example.com",
  "customerPhone": "+1876xxxxxxx",
  "priority": "high",
  "category": "background-check",
  "tags": ["background", "criminal-records"],
  "actionRequired": true,
  "actionUrl": "/portal/background/report/507f1f77bcf86cd799439011",
  "actionText": "View Background Report",
  "metadata": {
    "checkType": "comprehensive",
    "duration": "3 days",
    "findings": "clear"
  }
}
```

**Request Parameters**:
- `updateType` (string, required) - Type: `verification`, `system`, `request`, `custom`
- `status` (string, optional) - Status relevant to update type
- `title` (string, required) - Update title
- `message` (string, required) - Brief message
- `description` (string, optional) - Detailed description
- `verificationId` (string, optional) - Related verification ID
- `requestId` (string, optional) - Related request ID
- `userId` (string, optional) - Related user ID
- `customerName` (string, optional) - Customer's name
- `customerEmail` (string, optional) - Customer's email
- `customerPhone` (string, optional) - Customer's phone
- `priority` (string, optional) - Priority level (default: `normal`)
- `category` (string, optional) - Custom category
- `tags` (array, optional) - Array of tags
- `actionRequired` (boolean, optional) - Whether action is required (default: false)
- `actionUrl` (string, optional) - URL for action button
- `actionText` (string, optional) - Text for action button
- `metadata` (object, optional) - Custom metadata object

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "_id": "507f1f77bcf86cd799439200",
    "organisationId": "507f1f77bcf86cd799439031",
    "clientId": "507f1f77bcf86cd799439030",
    "updateType": "custom",
    "title": "Background Check Completed",
    "message": "Background check for John Smith has been completed",
    "priority": "high",
    "read": false,
    "createdBy": {
      "clientId": "507f1f77bcf86cd799439030",
      "name": "Jane Admin",
      "email": "jane@company.com",
      "system": false
    },
    "createdAt": "2025-10-19T14:00:00Z"
  },
  "message": "Update created successfully"
}
```

**Notes**:
- Creates update for authenticated organisation
- `createdBy` automatically populated from request
- Used for custom integration scenarios
- All fields optional except `updateType`, `title`, `message`

---

### 5. Publish Verification Update

Publish a verification status update (specialized verification endpoint).

**Endpoint**: `POST /api/v1/merchant/updates/verification`

**Request Body**:
```json
{
  "verificationType": "identity",
  "status": "cleared",
  "verificationResult": {
    "_id": "507f1f77bcf86cd799439011",
    "documentType": "passport",
    "documentNumber": "A1234567",
    "clearedAt": "2025-10-19T14:00:00Z"
  },
  "customer": {
    "name": "John Smith",
    "firstName": "John",
    "lastName": "Smith",
    "emailAddress": "john@example.com",
    "phoneNumber": "+1876xxxxxxx",
    "userId": "507f1f77bcf86cd799439050"
  },
  "priority": "normal"
}
```

**Request Parameters**:
- `verificationType` (string, required) - Type: `identity`, `address`, `income`, `employment`, `qualification`, `reference`, `company`, `background`
- `status` (string, required) - Verification status: `cleared`, `rejected`, `submitted`, etc.
- `verificationResult` (object, optional) - Verification result details
- `customer` (object, required) - Customer information
  - `name` (string) - Full name
  - `firstName` (string) - First name
  - `lastName` (string) - Last name
  - `emailAddress` (string) - Email
  - `phoneNumber` (string) - Phone
  - `userId` (string) - User ID
- `priority` (string, optional) - Priority level (default: `normal`)

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "_id": "507f1f77bcf86cd799439200",
    "updateType": "verification",
    "status": "cleared",
    "title": "Identity Verification Cleared",
    "message": "John Smith's identity verification has been approved",
    "verificationType": "identity",
    "customer": {...},
    "priority": "normal",
    "createdAt": "2025-10-19T14:00:00Z"
  },
  "message": "Verification update published successfully"
}
```

**Error Response** (400):
```json
{
  "success": false,
  "message": "verificationType, status, and customer are required"
}
```

**Auto-Generated Content**:
The system automatically generates:
- **Title** - Based on verification type and status
- **Message** - Formatted message with customer name and verification type
- **Category** - Set to `verification`
- **Tags** - Array with verification type and status
- **Action URL** - Link to verification report
- **Action Text** - "View Report" or similar

**Notes**:
- Automatically called by verification services when status changes
- Can be called manually for custom integration
- Uses merchant service helper function
- Creates standardised verification updates

---

### 6. Publish System Update

Publish a system-wide update or announcement.

**Endpoint**: `POST /api/v1/merchant/updates/system`

**Request Body**:
```json
{
  "title": "Scheduled Maintenance Notice",
  "message": "Platform will be offline for maintenance",
  "priority": "high",
  "category": "maintenance",
  "actionRequired": false,
  "actionUrl": "https://status.cleared.id",
  "actionText": "View Status Page"
}
```

**Request Parameters**:
- `title` (string, required) - Update title
- `message` (string, required) - Brief message
- `priority` (string, optional) - Priority level (default: `normal`)
- `category` (string, optional) - Category (default: `system`)
- `actionRequired` (boolean, optional) - Whether action required (default: false)
- `actionUrl` (string, optional) - URL for action button
- `actionText` (string, optional) - Text for action button

**Success Response** (200):
```json
{
  "success": true,
  "data": {
    "_id": "507f1f77bcf86cd799439201",
    "updateType": "system",
    "title": "Scheduled Maintenance Notice",
    "message": "Platform will be offline for maintenance",
    "priority": "high",
    "category": "maintenance",
    "createdAt": "2025-10-19T10:00:00Z"
  },
  "message": "System update published successfully"
}
```

**Error Response** (400):
```json
{
  "success": false,
  "message": "title and message are required"
}
```

**Common Categories**:
- `maintenance` - Maintenance notices
- `feature` - Feature announcements
- `security` - Security alerts
- `policy` - Policy/terms updates
- `service` - Service disruptions

**Notes**:
- Publishes to all organisations or specific organisation
- Used by platform administrators
- Can include action links for more information

---

## Update Object Structure

### Complete Update Object

```json
{
  "_id": "507f1f77bcf86cd799439200",
  "organisationId": "507f1f77bcf86cd799439031",
  "clientId": "507f1f77bcf86cd799439030",
  "updateType": "verification",
  "status": "cleared",
  "title": "Identity Verification Cleared",
  "message": "John Smith's identity verification has been approved",
  "description": "The identity verification for John Smith (passport A1234567) has been successfully verified and cleared.",
  "verificationType": "identity",
  "verificationId": "507f1f77bcf86cd799439011",
  "requestId": "507f1f77bcf86cd799439013",
  "userId": "507f1f77bcf86cd799439050",
  "customer": {
    "name": "John Smith",
    "firstName": "John",
    "lastName": "Smith",
    "emailAddress": "john@example.com",
    "phoneNumber": "+1876xxxxxxx",
    "userId": "507f1f77bcf86cd799439050"
  },
  "priority": "normal",
  "category": "verification",
  "tags": ["identity", "cleared"],
  "actionRequired": true,
  "actionUrl": "/portal/identity/report/507f1f77bcf86cd799439011",
  "actionText": "View Report",
  "metadata": {
    "documentType": "passport",
    "documentNumber": "A12****7",
    "clearedBy": "operations",
    "processingTime": "2.5 hours"
  },
  "read": false,
  "readAt": null,
  "createdBy": {
    "clientId": "507f1f77bcf86cd799439030",
    "name": "System",
    "email": "system@cleared.id",
    "system": true
  },
  "createdAt": "2025-10-19T14:00:00Z",
  "updatedAt": "2025-10-19T14:00:00Z"
}
```

### Field Descriptions

| Field | Type | Description |
|-------|------|-------------|
| `_id` | ObjectId | Unique update identifier |
| `organisationId` | ObjectId | Organisation receiving update |
| `updateType` | String | Type of update |
| `status` | String | Status (context-dependent) |
| `title` | String | Short update title |
| `message` | String | Brief message |
| `description` | String | Detailed description (optional) |
| `verificationType` | String | Verification type (if applicable) |
| `verificationId` | ObjectId | Related verification ID (optional) |
| `requestId` | ObjectId | Related request ID (optional) |
| `userId` | ObjectId | Related user ID (optional) |
| `customer` | Object | Customer information (optional) |
| `priority` | String | Priority level |
| `category` | String | Category for grouping |
| `tags` | Array | Tags for filtering |
| `actionRequired` | Boolean | Whether action required |
| `actionUrl` | String | URL for action button |
| `actionText` | String | Text for action button |
| `metadata` | Object | Custom metadata |
| `read` | Boolean | Whether update has been read |
| `readAt` | Date | When update was read |
| `createdBy` | Object | Who/what created the update |
| `createdAt` | Date | Creation timestamp |

## Integration Examples

### Polling for New Updates

```javascript
// Poll for unread updates every 30 seconds
setInterval(async () => {
  const response = await fetch('/api/v1/merchant/updates?unreadOnly=true&limit=10', {
    headers: {
      'Authorization': `Bearer ${apiKey}`
    }
  });
  
  const { data } = await response.json();
  
  if (data.updates.length > 0) {
    console.log(`${data.updates.length} new updates`);
    
    // Display notification to user
    data.updates.forEach(update => {
      showNotification(update.title, update.message, update.priority);
    });
  }
  
  // Update unread badge
  updateNotificationBadge(data.unreadCount);
}, 30000);
```

### Marking Updates as Read

```javascript
// When user views an update, mark it as read
async function viewUpdate(updateId) {
  // Display update to user
  displayUpdateDetails(updateId);
  
  // Mark as read
  await fetch('/api/v1/merchant/updates/mark-read', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${apiKey}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      updateIds: [updateId]
    })
  });
  
  // Refresh unread count
  updateUnreadCount();
}

// Bulk mark as read
async function markAllAsRead(updateIds) {
  await fetch('/api/v1/merchant/updates/mark-read', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${apiKey}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      updateIds: updateIds
    })
  });
}
```

### Filtering Updates

```javascript
// Get only high-priority verification updates
const response = await fetch(
  '/api/v1/merchant/updates?updateType=verification&priority=high&limit=20',
  {
    headers: { 'Authorization': `Bearer ${apiKey}` }
  }
);

// Search updates
const searchResponse = await fetch(
  '/api/v1/merchant/updates?search=john+smith&limit=50',
  {
    headers: { 'Authorization': `Bearer ${apiKey}` }
  }
);

// Get unread verification updates only
const unreadResponse = await fetch(
  '/api/v1/merchant/updates?updateType=verification&unreadOnly=true',
  {
    headers: { 'Authorization': `Bearer ${apiKey}` }
  }
);
```

## Common Use Cases

### Use Case 1: Notification Center

```javascript
// Build a notification center UI
class NotificationCenter {
  async loadUpdates(page = 1, filters = {}) {
    const response = await fetch(
      `/api/v1/merchant/updates?limit=20&skip=${(page-1)*20}&` +
      new URLSearchParams(filters),
      {
        headers: { 'Authorization': `Bearer ${apiKey}` }
      }
    );
    
    const { data } = await response.json();
    return data;
  }
  
  async markAsRead(updateIds) {
    await fetch('/api/v1/merchant/updates/mark-read', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${apiKey}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({ updateIds })
    });
  }
  
  async getUnreadCount() {
    const response = await fetch('/api/v1/merchant/updates/unread-count', {
      headers: { 'Authorization': `Bearer ${apiKey}` }
    });
    
    const { data } = await response.json();
    return data.unreadCount;
  }
}
```

### Use Case 2: Email Digest

```javascript
// Send daily email digest of unread updates
async function sendDailyDigest() {
  const response = await fetch(
    '/api/v1/merchant/updates?unreadOnly=true&limit=100',
    {
      headers: { 'Authorization': `Bearer ${apiKey}` }
    }
  );
  
  const { data } = await response.json();
  
  if (data.updates.length === 0) return;
  
  // Group by priority
  const urgent = data.updates.filter(u => u.priority === 'urgent');
  const high = data.updates.filter(u => u.priority === 'high');
  const normal = data.updates.filter(u => u.priority === 'normal');
  
  // Send email
  await sendEmail({
    to: adminEmail,
    subject: `Daily Update Digest - ${data.updates.length} new updates`,
    html: generateDigestHTML({ urgent, high, normal })
  });
}
```

### Use Case 3: Webhook Alternative

```javascript
// Use updates as alternative to webhooks
// Regularly poll for verification updates

async function processVerificationUpdates() {
  const response = await fetch(
    '/api/v1/merchant/updates?updateType=verification&unreadOnly=true',
    {
      headers: { 'Authorization': `Bearer ${apiKey}` }
    }
  );
  
  const { data } = await response.json();
  
  for (const update of data.updates) {
    if (update.status === 'cleared') {
      // Process cleared verification
      await processVerification(update.verificationId);
      
      // Mark as read
      await markAsRead([update._id]);
    }
  }
}

// Run every minute
setInterval(processVerificationUpdates, 60000);
```

## Best Practices

### 1. Update Management

- **Poll Regularly** - Check for new updates every 30-60 seconds
- **Mark as Read** - Always mark updates as read when viewed
- **Filter Appropriately** - Use filters to show relevant updates
- **Unread Badge** - Display unread count prominently

### 2. Priority Handling

- **Urgent Updates** - Show prominent alerts, send push notifications
- **High Priority** - Highlight in UI, email notifications
- **Normal Priority** - Standard display in notification list

### 3. Action Handling

- **Action Required** - Display prominently, prevent dismissal until acted upon
- **Action URLs** - Navigate directly to relevant page
- **Action Tracking** - Track which actions users complete

### 4. Performance

- **Pagination** - Load updates in batches (20-50 per page)
- **Caching** - Cache unread count, refresh every 30 seconds
- **Lazy Loading** - Load older updates only when scrolled
- **Selective Fields** - Request only needed fields

### 5. User Experience

- **Grouping** - Group by date or category
- **Filtering** - Allow filtering by type, priority, status
- **Search** - Provide search functionality
- **Mark All Read** - Provide bulk mark as read option
- **Clear Messaging** - Use clear, concise update messages

## Error Codes

| Status Code | Description |
|------------|-------------|
| 200 | Success |
| 400 | Bad Request (missing required fields) |
| 401 | Unauthorised (invalid/missing token) |
| 403 | Forbidden (insufficient permissions) |
| 500 | Internal Server Error |

## Rate Limits

- **List Updates**: 500 per hour per organisation
- **Mark as Read**: 1000 per hour per organisation
- **Get Unread Count**: 2000 per hour per organisation
- **Create Update**: 100 per hour per organisation

## Related Documentation

- [API Overview](./README.md)
- [Verification Requests API](./verification-requests.md)
- [Identity Verification API](./identity/identity-api.md)
- [Address Verification API](./address/address-api.md)

