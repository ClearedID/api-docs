# Cleared® API Documentation

## Overview

Cleared® is a **digital verification and signature platform** that provides trusted solutions for businesses, consumers, and organisations.

Our services include **identity, address, income, employment, qualification, background, and company verification**, as well as **digital document signing**, **onboarding workflows**, and **compliance management**.

This repository hosts the official **Cleared API documentation** in Markdown format. It is intended for developers and integrators who wish to embed Cleared's verification and trust services into their own applications, portals, or workflows.

---

## 🚀 Getting Started

New to the Cleared API? Follow these steps:

1. **[Introduction](./introduction.md)** – Understand what the API offers and how it works
2. **[Authentication](./authentication.md)** – Learn how to authenticate your requests  
3. **[Registration & Onboarding](./registration.md)** – Register your organisation and get API access
4. **[Endpoints](./endpoints.md)** – Explore the API structure and common patterns

---

## 📚 Core Documentation

### API Fundamentals

| Document | Description |
|----------|-------------|
| **[Introduction](./introduction.md)** | Welcome to the Cleared API - overview and key concepts |
| **[Authentication](./authentication.md)** | API key generation, JWT tokens, and security best practices |
| **[Registration](./registration.md)** | Complete registration and onboarding walkthrough |
| **[Endpoints](./endpoints.md)** | Base URLs, request/response formats, pagination, and filtering |

---

## 🔐 Verification Services

### Identity Verification

Verify customer identities with government-issued documents and biometric facial matching.

| Document | Description |
|----------|-------------|
| **[Identity API Overview](./identity/identity-api.md)** | Service introduction, features, and workflow |
| **[Identity Endpoints](./identity/identity-endpoints.md)** | 35 endpoints for ID verification, TRN validation, face auth, and account recovery |

**Key Features**: Passport/ID/driver's licence verification, liveness detection, biometric matching, duplicate detection, TRN verification

---

### Address Verification

Validate residential and business addresses with proof of address documents.

| Document | Description |
|----------|-------------|
| **[Address API Overview](./address/address-api.md)** | Service introduction, features, and workflow |
| **[Address Endpoints](./address/address-endpoints.md)** | 13 endpoints for individual and company address verification |

**Key Features**: Proof of address validation, GPS verification, company address checks, due diligence requests

---

### Verification Requests

Create and manage verification requests sent to customers.

| Document | Description |
|----------|-------------|
| **[Initiate Verification](./initiate-verification.md)** | Complete guide to creating verification requests |
| **[Manage Requests](./verification-requests-management.md)** | List, view, withdraw, and extend verification requests |

**Key Features**: Multi-service requests, automatic customer lookup, expiration management, in-person mode, request bundling

---

### Verification Links

Create shareable verification links for customer self-service verification.

| Document | Description |
|----------|-------------|
| **[Verification Links API](./verification-links/verification-links-api.md)** | Service introduction and key features |
| **[Verification Links Endpoints](./verification-links/verification-links-endpoints.md)** | 7 endpoints for link management and tracking |

**Key Features**: Shareable URLs, custom branding, webhook integration, submission tracking

---

### Background Checks

*Documentation coming soon*

**Services**: Criminal records, credit history, employment verification, reference checks

---

### Income & Employment Verification

*Documentation coming soon*

**Services**: Income validation, employment history, salary verification

---

### Qualifications & References

*Documentation coming soon*

**Services**: Education verification, professional certifications, reference checks

---

### Company Verification

*Documentation coming soon*

**Services**: Business registration, TRN validation, corporate identity checks

---

## 📝 Digital Signatures

Secure document signing with legal validity and tamper-proof sealing.

### Document Signatures

| Document | Description |
|----------|-------------|
| **[Signatures Overview](./document-signatures/README.md)** | Introduction to digital signature services |
| **[Merchant Documents](./document-signatures/merchant-signature-documents.md)** | 24 endpoints for document management and tracking |
| **[Envelopes](./document-signatures/envelopes.md)** | 16 endpoints for multi-document signing packages |
| **[Document Templates](./document-signatures/document-templates.md)** | 9 endpoints for reusable document templates |
| **[Envelope Templates](./document-signatures/envelope-templates.md)** | 7 endpoints for reusable envelope packages |

**Total Endpoints**: 56 endpoints

**Key Features**: PDF signing, certificate-based signatures, multi-signer workflows, template system, audit trails

---

## 🎯 Workflow Management

### Onboarding Pages

Create branded onboarding pages combining lead capture, verifications, and forms.

| Document | Description |
|----------|-------------|
| **[Onboarding Pages API](./onboarding-pages.md)** | Complete guide with 12 endpoints |

**Key Features**: Visual page builder, lead capture forms, custom fields, verification workflows, submission tracking, logo management

**Use Cases**: Loan applications, rental applications, employee onboarding, tenant screening

---

### Merchant Updates

Real-time notification system for verification events and system announcements.

| Document | Description |
|----------|-------------|
| **[Merchant Updates API](./merchant-updates.md)** | Complete guide with 6 endpoints |

**Key Features**: Real-time updates, unread tracking, priority levels, filtering, webhook alternative

**Update Types**: Verification updates, system announcements, request notifications

---

## 🌍 Environments

### Sandbox (Testing)
- **Base URL**: `https://cleared.id/api/v1/merchant`
- **Admin Portal**: `https://qa.cleared.id/portal`
- **Purpose**: Integration testing with test data
- **Cost**: Free (unlimited testing)

### Production (Live)
- **Base URL**: `https://cleared.id/api/v1/merchant`
- **Admin Portal**: `https://cleared.id/portal`
- **Purpose**: Live operations with real data
- **Cost**: Credit-based pricing

---

## 📖 Quick Examples

### Authentication
```bash
# Verify API key
curl -X GET https://cleared.id/api/v1/merchant/auth/verify \
  -H "Authorization: Bearer YOUR_API_KEY"
```

### Initiate Identity Verification
```bash
curl -X POST https://cleared.id/api/v1/merchant/identity/verification/initiate \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "John Smith",
    "emailAddress": "john@example.com",
    "verificationRequests": [
      {
        "type": "identity",
        "required": true
      }
    ]
  }'
```

### Create Verification Link
```bash
curl -X POST https://cleared.id/api/v1/merchant/verification-links/create \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "linkTitle": "Customer Verification",
    "verificationType": "identity",
    "destination": "https://yoursite.com/complete"
  }'
```

### Create Onboarding Page
```bash
curl -X POST https://cleared.id/api/v1/merchant/screening/onboarding/pages/create \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -F "pageTitle=Loan Application" \
  -F "requiredVerifications=[\"identity\",\"address\",\"income\"]"
```

### Sign Document
```bash
curl -X POST https://cleared.id/api/v1/merchant/signatures/documents/create \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -F "document=@contract.pdf" \
  -F "signers=[{\"name\":\"John Smith\",\"email\":\"john@example.com\"}]"
```

---

## 📊 Complete Documentation Summary

### By Category

| Category | Files | Endpoints | Description |
|----------|-------|-----------|-------------|
| **Core API** | 4 files | - | Authentication, endpoints, registration |
| **Identity Verification** | 2 files | 35 endpoints | ID verification, biometrics, TRN |
| **Address Verification** | 2 files | 13 endpoints | Address validation, GPS checks |
| **Verification Requests** | 2 files | 5 endpoints | Create and manage requests |
| **Verification Links** | 2 files | 7 endpoints | Shareable verification URLs |
| **Digital Signatures** | 5 files | 56 endpoints | Document and envelope signing |
| **Onboarding Pages** | 1 file | 12 endpoints | Custom onboarding workflows |
| **Merchant Updates** | 1 file | 6 endpoints | Real-time notifications |

**Total**: 23 files, 151 endpoints documented

---

## 🗂️ Documentation Structure

```
api-docs/
├── README.md                              # This file
├── introduction.md                        # Getting started
├── authentication.md                      # API keys and security
├── registration.md                        # Account registration
├── endpoints.md                           # API structure
│
├── initiate-verification.md              # Create verification requests
├── verification-requests-management.md   # Manage requests
├── merchant-updates.md                   # Notification system
├── onboarding-pages.md                   # Onboarding workflows
│
├── identity/
│   ├── identity-api.md                   # Identity service overview
│   └── identity-endpoints.md             # Identity endpoints (35)
│
├── address/
│   ├── address-api.md                    # Address service overview
│   └── address-endpoints.md              # Address endpoints (13)
│
├── verification-links/
│   ├── verification-links-api.md         # Links overview
│   └── verification-links-endpoints.md   # Links endpoints (7)
│
├── document-signatures/
│   ├── README.md                         # Signatures overview
│   ├── merchant-signature-documents.md   # Merchant endpoints (24)
│   ├── envelopes.md                      # Envelopes (16)
│   ├── document-templates.md             # Doc templates (9)
│   └── envelope-templates.md             # Envelope templates (7)
│
├── criminal-records/
│   └── introduction.md                   # Coming soon
│
├── income/
│   └── introduction.md                   # Coming soon
│
└── references/
    └── introduction.md                   # Coming soon
```

---

## 💡 Common Use Cases

### Use Case 1: Employee Onboarding
1. Create verification request ([Initiate Verification](./initiate-verification.md))
2. Customer completes identity verification
3. Receive updates ([Merchant Updates](./merchant-updates.md))
4. Retrieve verification results

### Use Case 2: Loan Application
1. Create onboarding page ([Onboarding Pages](./onboarding-pages.md))
2. Customer fills lead capture form
3. Customer completes identity + address + income verification
4. Review submission and results

### Use Case 3: Tenant Screening
1. Create verification link ([Verification Links](./verification-links/verification-links-api.md))
2. Share link with potential tenant
3. Tenant completes identity + address + reference checks
4. Receive webhook notification
5. Review results and approve/deny

### Use Case 4: Contract Signing
1. Create document ([Document Signatures](./document-signatures/merchant-signature-documents.md))
2. Add signers with email addresses
3. Send for signature
4. Track signing progress
5. Download signed document with certificate

### Use Case 5: KYC Compliance
1. Initiate identity + address verification
2. Monitor request status
3. Receive clearance notification
4. Store verification results for compliance

### Use Case 6: ABC Company LLC Loan Application Workflow
Follow this end-to-end workflow for ABC Company LLC onboarding (IDV via onboarding pages + redirect) and contract signing (document templates, eSeal, and outbound webhooks): [ABC Company LLC – Cleared Workflow Documentation](./client-loan-application-workflow.md)

---

## 🛠️ Developer Support

### Documentation
- **[API Overview](./introduction.md)** – Start here for an introduction
- **[Authentication Guide](./authentication.md)** – Security and API keys
- **[Endpoint Reference](./endpoints.md)** – API structure and patterns
- **Service Guides** – Detailed documentation for each service

### Technical Support
- **Email**: [support@cleared.id](mailto:support@cleared.id)
- **Admin Portal**: [https://cleared.id/admin](https://cleared.id/admin)
- **Status Page**: [https://status.cleared.id](https://status.cleared.id)
- **Website**: [https://cleared.id](https://cleared.id)

### Resources
- API Changelog (coming soon)
- SDK Libraries (coming soon)
- Postman Collection (coming soon)
- Integration Examples (coming soon)
- Developer Forum (coming soon)

---

## 🔒 Security & Compliance

### Authentication
- Bearer token authentication
- JWT-based API keys
- Key rotation support
- IP whitelisting available

### Data Protection
- GDPR compliant
- CCPA compliant
- POPIA compliant
- SOC 2 Type II certified (in progress)
- ISO 27001 certified (in progress)

### Best Practices
- Always use HTTPS
- Never expose API keys in client-side code
- Implement webhook signature verification
- Store customer data securely
- Follow data retention policies

---

## 📋 API Features

### General Features
- ✅ RESTful API design
- ✅ JSON request/response format
- ✅ Comprehensive error messages
- ✅ Pagination support
- ✅ Filtering and search
- ✅ Webhook notifications
- ✅ Audit logging
- ✅ Rate limiting
- ✅ Sandbox environment

### Verification Features
- ✅ Multi-service requests
- ✅ Automatic customer lookup
- ✅ User auto-creation
- ✅ Email/SMS notifications
- ✅ Custom expiration times
- ✅ In-person verification mode
- ✅ Batch processing support
- ✅ Real-time status updates

### Workflow Features
- ✅ Custom onboarding pages
- ✅ Lead capture forms
- ✅ Shareable verification links
- ✅ Custom branding
- ✅ Multi-step workflows
- ✅ Template system
- ✅ Submission tracking

### Signature Features
- ✅ PDF document signing
- ✅ Certificate-based signatures
- ✅ Multi-signer workflows
- ✅ Sequential/parallel signing
- ✅ Template system
- ✅ Audit trails
- ✅ Legal validity

---

## 🆕 What's New

### Recently Added
- ✨ **Onboarding Pages API** - Create complete onboarding workflows
- ✨ **Merchant Updates API** - Real-time notification system
- ✨ **Initiate Verification API** - Streamlined request creation
- ✨ **Verification Links API** - Shareable verification URLs

### Coming Soon
- 🔜 Background Checks API
- 🔜 Income Verification API
- 🔜 Reference Checks API
- 🔜 Webhook Management UI
- 🔜 GraphQL API
- 🔜 SDK Libraries (JavaScript, Python, PHP)

---

## 🤝 Contributing

This documentation is maintained by the Cleared team. If you find errors or have suggestions:

1. **Report Issues**: Contact [support@cleared.id](mailto:support@cleared.id)
2. **Request Features**: Submit via your Admin Portal
3. **Provide Feedback**: Share during integration process
4. **Join Beta**: Request early access to new features

---

## 📜 Terms & Compliance

By using the Cleared API, you agree to:
- Comply with all applicable data protection regulations (GDPR, CCPA, POPIA)
- Use the API only for legitimate business purposes
- Protect user data and privacy
- Maintain secure storage of API credentials
- Follow responsible disclosure for security issues

**Full Terms**: [https://cleared.id/terms](https://cleared.id/terms)  
**Privacy Policy**: [https://cleared.id/privacy](https://cleared.id/privacy)  
**SLA**: [https://cleared.id/sla](https://cleared.id/sla)

---

## 📞 Getting Help

### Before You Ask
1. Check this documentation
2. Review the specific service guide
3. Search our knowledge base
4. Check the status page for incidents

### Support Channels
- **Technical Support**: [support@cleared.id](mailto:support@cleared.id)
- **Sales**: [sales@cleared.id](mailto:sales@cleared.id)
- **Billing**: [billing@cleared.id](mailto:billing@cleared.id)
- **Security**: [security@cleared.id](mailto:security@cleared.id)

### Response Times
- **Critical Issues**: < 1 hour
- **High Priority**: < 4 hours
- **Normal Priority**: < 24 hours
- **Low Priority**: < 48 hours

---

## 🎓 Training & Certification

### Developer Training
- API Integration Bootcamp (coming soon)
- Best Practices Workshop (coming soon)
- Security & Compliance Training (coming soon)

### Certification Program
- Cleared Certified Developer (coming soon)
- Cleared Certified Architect (coming soon)

---

**Ready to get started?** Begin with [Introduction](./introduction.md) →

---

*Last updated: October 19, 2025*  
*Version: 2.0*  
*© 2025 Cleared®. All rights reserved.*
