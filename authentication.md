# Authentication

## Overview

All requests to the SWF Cloud API must be authenticated using a secure API key, provided in the `Authorization` header using the **Bearer token format**. The API key is a loaded JWT token that contains your organisation's credentials and permissions.

This page explains how to obtain your API key, how to authenticate requests properly, and the security expectations for systems integrating with our cloud.

## Authentication Method

Each organisation integrating with the API must generate its own unique API key. This key must be included in every request using the following header format:

```
Authorization: Bearer YOUR_API_KEY
```

### Example Request

```bash
curl -X GET https://cleared.id/api/v1/merchant/identity/verifications \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json"
```

### Example in JavaScript

```javascript
const axios = require('axios');

const apiKey = process.env.SWF_API_KEY;

const response = await axios.get('https://cleared.id/api/v1/merchant/identity/verifications', {
  headers: {
    'Authorization': `Bearer ${apiKey}`,
    'Content-Type': 'application/json'
  }
});
```

### Example in Python

```python
import requests
import os

api_key = os.environ.get('SWF_API_KEY')

headers = {
    'Authorization': f'Bearer {api_key}',
    'Content-Type': 'application/json'
}

response = requests.get(
    'https://cleared.id/api/v1/merchant/identity/verifications',
    headers=headers
)
```

### Example in PHP

```php
<?php

$apiKey = getenv('SWF_API_KEY');

$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, 'https://cleared.id/api/v1/merchant/identity/verifications');
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HTTPHEADER, [
    'Authorization: Bearer ' . $apiKey,
    'Content-Type: application/json'
]);

$response = curl_exec($ch);
curl_close($ch);
?>
```

## Obtaining an API Key

To authenticate with the API, you must first obtain an API key from the Admin Portal.

### Steps to Generate an API Key

1. **Log into the Admin Portal**
   - Sandbox: [https://qa.cleared.id/admin](https://qa.cleared.id/admin)
   - Production: [https://cleared.id/admin](https://cleared.id/admin)

2. **Navigate to API Integrations**
   - Go to **Integrations** > **API Integrations** from the main menu

3. **Generate a New Key**
   - Click **Generate New Key**
   - Provide a name/description for the key (e.g., "Production Server", "Test Environment")
   - Select the appropriate environment (Sandbox or Production)
   - Click **Generate**

4. **Copy and Store the Key**
   - The API key will be displayed once
   - **Copy it immediately** – you won't be able to see it again
   - Store it securely in your environment variables or secrets manager

5. **Test the Key**
   - Make a test API call to verify the key works correctly
   - Verify you can authenticate successfully

### Multiple API Keys

You can generate multiple API keys for different purposes:

- **Separate keys per environment** (Sandbox vs Production)
- **Separate keys per integration** (Web app, mobile app, backend service)
- **Separate keys per developer** (for tracking and access control)
- **Separate keys per client** (if you're a reseller or agency)

Each key can be individually managed, tracked, and revoked if needed.

## API Key Structure

The API key is a JWT (JSON Web Token) that contains:

- **Organisation ID** – Your organisation's unique identifier
- **Client ID** – Your admin user identifier
- **Permissions** – Services and actions your key can access
- **Environment** – Sandbox or Production
- **Expiration** – Key expiry date (if set)
- **Issued At** – When the key was generated

**Example JWT payload (decoded):**
```json
{
  "organisationId": "org_123456789",
  "clientId": "client_123456789",
  "permissions": ["identity:read", "identity:write", "address:read"],
  "environment": "production",
  "iat": 1634567890,
  "exp": 1642343890
}
```

**Note:** You don't need to decode or manipulate the JWT. Simply include it in the Authorization header as-is.

## Authentication Errors

### 401 Unauthorized

You'll receive a `401 Unauthorized` response if:

- No API key is provided
- API key is invalid or malformed
- API key has expired
- API key has been revoked

**Error Response:**
```json
{
  "success": false,
  "error": "Unauthorized",
  "code": "INVALID_API_KEY",
  "message": "Invalid or expired API key"
}
```

**Solutions:**
1. Verify the API key is correct and hasn't been modified
2. Check that the key hasn't expired
3. Ensure you're using the correct environment (Sandbox vs Production)
4. Regenerate the API key if it's been compromised

### 403 Forbidden

You'll receive a `403 Forbidden` response if:

- Your API key lacks permissions for the requested action
- Your organisation's subscription doesn't include the requested service
- Your account has been suspended

**Error Response:**
```json
{
  "success": false,
  "error": "Forbidden",
  "code": "INSUFFICIENT_PERMISSIONS",
  "message": "Your API key does not have permission to access this resource"
}
```

**Solutions:**
1. Contact your administrator to request additional permissions
2. Upgrade your subscription to access the required services
3. Contact support if you believe this is an error

## Security Best Practices

To ensure safe use of your API key, follow these practices:

### 1. Never Expose Keys in Frontend Code

API keys must **only** be used in secure, server-side environments.

❌ **DON'T DO THIS:**
```javascript
// Never put API keys in frontend JavaScript
const apiKey = 'YOUR_API_KEY';
fetch('/api/endpoint', {
  headers: { 'Authorization': `Bearer ${apiKey}` }
});
```

✅ **DO THIS INSTEAD:**
```javascript
// Make requests to your backend, which securely stores the API key
fetch('/api/your-backend-endpoint', {
  // Your backend handles API key authentication
});
```

### 2. Store Keys Securely

Use encrypted secrets storage, environment variables, or a key management system.

**Environment Variables:**
```bash
# .env file (never commit to version control)
SWF_API_KEY=YOUR_API_KEY
```

**AWS Secrets Manager:**
```javascript
const AWS = require('aws-sdk');
const secretsManager = new AWS.SecretsManager();

const secret = await secretsManager.getSecretValue({
  SecretId: 'swf-cloud-api-key'
}).promise();

const apiKey = JSON.parse(secret.SecretString).apiKey;
```

**Azure Key Vault:**
```csharp
var client = new SecretClient(vaultUri, new DefaultAzureCredential());
KeyVaultSecret secret = await client.GetSecretAsync("swf-api-key");
string apiKey = secret.Value;
```

### 3. Do Not Log API Keys

Prevent accidental exposure by ensuring keys are not written to logs or error reports.

❌ **DON'T DO THIS:**
```javascript
console.log('API Key:', apiKey); // Never log the key
logger.error('Request failed', { apiKey }); // Don't include in error logs
```

✅ **DO THIS INSTEAD:**
```javascript
// Log without exposing the key
logger.info('Making API request to identity service');
logger.error('Request failed', { endpoint, statusCode });
```

### 4. Rotate Keys Periodically

We recommend rotating keys at least every **90 days** or immediately upon suspicion of compromise.

**Key Rotation Process:**
1. Generate a new API key in the Admin Portal
2. Update your application configuration with the new key
3. Test the new key in a staging environment
4. Deploy to production
5. Revoke the old key after confirming the new key works

### 5. Use Separate Keys per Integration

If multiple systems will interact with the API, use separate keys to track and manage access independently.

**Example Setup:**
- `prod-web-app` – Key for production web application
- `prod-mobile-app` – Key for production mobile application
- `prod-backend-service` – Key for backend microservices
- `sandbox-testing` – Key for sandbox/testing environment

**Benefits:**
- Track API usage per integration
- Revoke access to specific integrations without affecting others
- Debug issues by isolating integration-specific problems
- Apply different rate limits or permissions per key

### 6. Implement IP Whitelisting (Optional)

For enhanced security, you can restrict API key usage to specific IP addresses.

**To enable IP whitelisting:**
1. Go to Admin Portal > Integrations > API Integrations
2. Select your API key
3. Click **Configure IP Restrictions**
4. Add allowed IP addresses or CIDR ranges
5. Save changes

**Example:**
```
Allowed IPs:
- 203.0.113.10 (Production server)
- 203.0.113.11 (Backup server)
- 192.0.2.0/24 (Office network)
```

### 7. Monitor API Key Usage

Regularly review API key usage in the Admin Portal:

- Check request counts and patterns
- Review error rates
- Identify unusual activity
- Monitor for potential security issues

**To view usage:**
1. Go to Admin Portal > Integrations > API Integrations
2. Select your API key
3. View **Usage Statistics** and **Activity Log**

### 8. Revoke Compromised Keys Immediately

If you suspect a key has been compromised:

1. **Immediately revoke the key** in the Admin Portal
2. Generate a new key
3. Update your application configuration
4. Review API logs for unauthorized access
5. Contact support if you detect suspicious activity

## API Key Management

### Viewing Active Keys

To view all active API keys:

1. Log into the Admin Portal
2. Navigate to **Integrations** > **API Integrations**
3. View the list of all active keys with their:
   - Name/Description
   - Environment (Sandbox/Production)
   - Created date
   - Last used date
   - Status (Active/Revoked)

### Revoking a Key

To revoke an API key:

1. Go to **Integrations** > **API Integrations**
2. Find the key you want to revoke
3. Click **Revoke**
4. Confirm the action

**Note:** Revocation is immediate. Any requests using the revoked key will fail with a `401 Unauthorized` error.

### Key Expiration

API keys can be configured with an expiration date:

- **No Expiration** (default) – Key remains valid until manually revoked
- **Custom Expiration** – Set a specific expiration date
- **Auto-Expiration** – Keys expire after a set period (30, 60, 90, 180, 365 days)

**To set expiration:**
1. When generating a new key, select **Set Expiration**
2. Choose expiration period or custom date
3. You'll receive email notifications before expiration

## Testing Authentication

### Test Endpoint

Use this endpoint to verify your API key is working correctly:

**Endpoint:** `GET /api/v1/merchant/auth/verify`

**Request:**
```bash
curl -X GET https://cleared.id/api/v1/merchant/auth/verify \
  -H "Authorization: Bearer YOUR_API_KEY"
```

**Success Response (200):**
```json
{
  "success": true,
  "data": {
    "authenticated": true,
    "organisationId": "org_123456789",
    "organisationName": "Acme Corporation",
    "environment": "production",
    "permissions": ["identity:read", "identity:write", "address:read"],
    "keyCreatedAt": "2025-10-01T10:00:00Z",
    "keyExpiresAt": "2026-10-01T10:00:00Z"
  },
  "message": "API key is valid"
}
```

**Error Response (401):**
```json
{
  "success": false,
  "error": "Invalid API key",
  "code": "INVALID_API_KEY"
}
```

## Troubleshooting

### Common Issues

**Issue: "Invalid or expired API key"**
- Verify the API key is correctly copied (no extra spaces or characters)
- Check the key hasn't expired
- Ensure you're using the correct environment URL
- Try generating a new key

**Issue: "Insufficient permissions"**
- Check your organisation's subscription includes the required services
- Verify the API key has the necessary permissions
- Contact your administrator or support

**Issue: "Rate limit exceeded"**
- Wait for the rate limit window to reset
- Implement exponential backoff in your requests
- Consider upgrading your plan for higher limits

**Issue: "CORS errors in browser"**
- API keys should never be used from frontend/browser code
- Move authentication to your backend server
- Use your backend as a proxy to the SWF API

## Next Steps

Now that you understand authentication:

1. **[Register your organisation](./registration.md)** if you haven't already
2. **Generate your API key** from the Admin Portal
3. **Test authentication** using the verify endpoint
4. **Explore [Endpoints](./endpoints.md)** to begin integrating services

## Support

If you encounter authentication issues:

- **Check the [Status Page](https://status.cleared.id)** for system status
- **Review this documentation** for common solutions
- **Contact Support** at [support@cleared.id](mailto:support@cleared.id)
- **Access Admin Portal** for key management and logs

---

← [Introduction](./introduction.md) | [Registration & Onboarding](./registration.md) →

