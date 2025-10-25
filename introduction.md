# Introduction

## Welcome to the Cleared® API

Welcome to the **Cleared API**, a secure and modern REST API that enables your platform to seamlessly integrate screening and verification services provided by **Cleared®**.

Our API is built for scale, security, and flexibility, and allows you to perform checks, request verifications, retrieve results, and manage interactions with your verified subjects from your own platform.

## About Cleared®

Cleared® is a **digital signature and verification platform** that provides trusted solutions for businesses, consumers, and organisations. Our services include:

- **Identity Verification** – AI-powered ID checks with biometric matching
- **Address Verification** – Validate residential and business addresses
- **Income & Employment Verification** – Confirm salary history and employment records
- **Qualifications & References** – Verify academic and professional references
- **Background Checks** – Criminal record and risk-based screening
- **Company Verification** – Corporate identity, TRN validation, and compliance checks
- **Digital Signatures** – Secure signing with certificates and tamper-proof sealing

## What You'll Need

To consume the API, you must complete a short onboarding process and have the following:

1. **Registered Organisation** on [cleared.id](https://cleared.id)
2. **Administrator (client) account** with access to the Admin Portal
3. **Generated API key** (Bearer token format) for authentication

All requests must be authenticated with your API key using this header:

```
Authorization: Bearer YOUR_API_KEY
```

## What the Documentation Covers

This API documentation provides full, production-ready detail on:

### Core Concepts
- **[Authentication](./authentication.md)** – How to authenticate using your API key
- **[Registration & Onboarding](./registration.md)** – How to register and onboard your organisation
- **[Endpoints](./endpoints.md)** – Full breakdown of all available routes, grouped by service type

### API Usage
- **Request/Response Structures** – Exact payloads and schemas for all actions
- **Statuses & Results** – Understand result types, statuses, and report formats
- **Certificates** – How to generate and retrieve signed verification certificates
- **Error Handling** – Understand how the API communicates failures
- **Environment Isolation** – Clear guidance on sandbox vs production usage

## API Architecture

The Cleared® API follows RESTful principles and uses standard HTTP methods:

- **GET** – Retrieve resources or data
- **POST** – Create new resources or initiate actions
- **PUT/PATCH** – Update existing resources
- **DELETE** – Remove resources

### Response Format

All API responses use JSON format and follow a consistent structure:

**Success Response:**
```json
{
  "success": true,
  "data": {
    // Response data
  },
  "message": "Operation completed successfully"
}
```

**Error Response:**
```json
{
  "success": false,
  "error": "Error message",
  "code": "ERROR_CODE",
  "details": "Additional error details"
}
```

### HTTP Status Codes

The API uses standard HTTP status codes:

| Status Code | Description |
|------------|-------------|
| 200 | OK – Request successful |
| 201 | Created – Resource created successfully |
| 400 | Bad Request – Invalid request parameters |
| 401 | Unauthorized – Invalid or missing API key |
| 403 | Forbidden – Insufficient permissions |
| 404 | Not Found – Resource not found |
| 429 | Too Many Requests – Rate limit exceeded |
| 500 | Internal Server Error – Server error |

## Environments

The Cleared® API provides two separate environments:

### Sandbox (UAT/Testing)
Use this environment for integration testing and QA with test accounts and dummy data.

- **Base URL:** `https://qa.cleared.id/api/v1/merchant`
- **Admin Portal:** `https://qa.cleared.id/portal`
- **Purpose:** Testing, development, and integration
- **Data:** Test data only, no real personal information

### Production (Live)
Use this environment for live production data and real verifications.

- **Base URL:** `https://cleared.id/api/v1/merchant`
- **Admin Portal:** `https://cleared.id/portal`
- **Purpose:** Live operations with real data
- **Data:** Real personal and business information

**Important:** Always test thoroughly in the Sandbox environment before moving to Production.

## Rate Limiting

To ensure fair usage and system stability, the API implements rate limiting:

- **Sandbox:** 100 requests per minute per API key
- **Production:** 1000 requests per minute per API key

Rate limit information is included in response headers:
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 987
X-RateLimit-Reset: 1634567890
```

If you exceed the rate limit, you'll receive a `429 Too Many Requests` response. Wait until the reset time before making additional requests.

## API Versioning

The current API version is **v1**. The version is included in the base URL path:

```
https://cleared.id/api/v1/merchant
```

We maintain backward compatibility within major versions. When breaking changes are introduced, a new major version will be released with advance notice.

## Security

The Cleared® API implements multiple layers of security:

- **TLS/HTTPS:** All API communication uses TLS 1.2 or higher
- **API Key Authentication:** Bearer token authentication for all requests
- **IP Whitelisting:** Optional IP restrictions for enhanced security
- **Data Encryption:** All sensitive data encrypted at rest and in transit
- **Audit Logging:** Complete audit trail of all API actions

## Getting Started

Follow these steps to get started with the API:

### Step 1: Register Your Organisation
Begin by registering at [cleared.id](https://cleared.id) and completing the onboarding process. See [Registration & Onboarding](./registration.md) for detailed instructions.

### Step 2: Generate API Key
Once your organisation is set up, generate your API key from the Admin Portal. See [Authentication](./authentication.md) for details.

### Step 3: Test in Sandbox
Use the Sandbox environment to test your integration with dummy data. This allows you to validate your implementation without affecting production data.

### Step 4: Integrate Services
Explore the available verification services and integrate the ones you need:
- Identity verification
- Address verification
- Income verification
- Employment verification
- Background checks
- Company verification
- Digital signatures

### Step 5: Go Live
After thorough testing in Sandbox, switch to the Production environment and begin processing real verifications.

## Next Steps

- **New to the API?** Start with [Authentication](./authentication.md) and [Registration](./registration.md)
- **Ready to integrate?** Explore [Endpoints](./endpoints.md) and specific service documentation
- **Need help?** Contact our support team at [support@cleared.id](mailto:support@cleared.id)

## Support and Resources

### Developer Support
- **Email:** [support@cleared.id](mailto:support@cleared.id)
- **Documentation:** [https://docs.cleared.id](https://docs.cleared.id)
- **Status Page:** [https://status.cleared.id](https://status.cleared.id)

### Resources
- [Cleared Website](https://cleared.id)
- [API Portal](https://cleared.id/portal) (Admin access required)
- [API Changelog](./changelog.md) (Coming soon)
- [SDK Libraries](./sdks.md) (Coming soon)

### Community
- Developer Forum (Coming soon)
- Integration Examples (Coming soon)
- Best Practices Guide (Coming soon)

## Terms and Compliance

By using the Cleared® API, you agree to:

- Comply with all applicable data protection regulations (GDPR, CCPA, etc.)
- Use the API only for legitimate business purposes
- Protect user data and privacy
- Follow our API usage guidelines and best practices
- Maintain secure storage of API keys and credentials

For full terms of service, visit [https://cleared.id/terms](https://cleared.id/terms).

---

**Ready to get started?** Continue to [Authentication](./authentication.md) →

