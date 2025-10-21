# Registration & Onboarding

## Overview

Before you can begin using the Cleared API, your organisation must be fully registered, verified, and onboarded into the ecosystem at [cleared.id/portal](https://cleared/portal). This process ensures that your organisation has access to the Admin Portal, can generate API credentials, and is authorised to initiate verification requests via the API.

This page provides a complete walkthrough of the onboarding process, from creating your initial account to generating your first API key.

## Onboarding Process Overview

Our onboarding process is straightforward and involves **2 key steps**:

1. **Registering yourself as a client** (administrator)
2. **Setting up your organisation** (company information)

After this setup is complete, your organisation will be able to generate API keys and begin integrating with the API.

**Estimated time:** 10-15 minutes

## Step 1: Registering Yourself as a Client

To begin onboarding, you must first create a client account (administrator account).

### Creating Your Account

1. **Visit the Website**
   - Go to [cleared.id/portal](https://cleared.id/portal)
   - Click **"Log In"** from the top-right menu

2. **Enter Your Email**
   - Provide your work email address
   - Click **"Continue"**

3. **Verify Your Email**
   - Check your email inbox for a 6-digit verification code
   - The email will arrive from `support@cleared.id`
   - Enter the 6-digit code on the verification page
   - The code expires after 10 minutes

4. **Complete Your Profile**
   - Enter your full name
   - Set your preferred display name
   - Accept the Terms of Service and Privacy Policy
   - Click **"Complete Registration"**

### Account Verification

Your account will be created immediately, and you'll be taken through the organisation setup process.

**Account Features:**
- ✅ Access to the Admin Portal
- ✅ Ability to create and manage organisations
- ✅ Multi-factor authentication (optional but recommended)
- ✅ Session management and security settings

### Email Verification Tips

**Didn't receive the email?**
- Check your spam/junk folder
- Ensure you entered the correct email address
- Wait a few minutes (emails can sometimes be delayed)
- Click **"Resend Code"** to receive a new verification email

**Code expired?**
- Request a new code by clicking **"Resend Code"**
- Complete verification within 10 minutes of receiving the new code

## Step 2: Setting Up Your Organisation

After creating your account, you'll be guided through the organisation setup process.

### Organisation Information

You'll need to provide the following information:

#### Basic Information
- **Organisation Name** – Your company's legal name
- **Trading Name** (optional) – Your company's trading/brand name
- **Registration Number** – Company registration number (e.g., TRN, Companies House number)
- **Tax ID** – Tax identification number (if applicable)

#### Contact Information
- **Business Address** – Registered business address
- **Phone Number** – Primary business contact number
- **Email Address** – Business email for communications

#### Industry & Purpose
- **Industry Type** – Select your business industry
- **Use Case** – Describe how you'll use Cleared's services
- **Expected Volume** – Estimated monthly verification volume

#### Company Documents (May be required)
- Certificate of Incorporation
- Proof of Business Address
- Director/Officer Identification
- Business License (if applicable)

### Organisation Verification

Once you submit your organisation information:

1. **Initial Review** (1-2 business hours)
   - Our team reviews your application
   - We may request additional documentation

2. **Verification** (1-2 business days)
   - We verify your company details
   - Background checks on the organisation
   - Compliance review

3. **Approval**
   - You'll receive an email notification once approved
   - Your organisation status will change to **"Active"**
   - You can now generate API keys and begin integration

### Organisation Status

Your organisation can have the following statuses:

| Status | Description |
|--------|-------------|
| **Pending** | Application submitted, awaiting review |
| **Under Review** | Team is reviewing your application |
| **Pending Documents** | Additional documents required |
| **Active** | Approved and ready to use |
| **Suspended** | Temporarily suspended (contact support) |
| **Rejected** | Application rejected (reason provided) |

## Step 3: Accessing the Admin Portal

After your organisation is registered and approved, you can access the Admin Portal to manage your integration.

### Admin Portal URLs

**Sandbox Environment (Testing):**
```
https://qa.cleared.id/portal
```
Use this portal to:
- Test your integration with dummy data
- Generate sandbox API keys
- Explore features without affecting production

**Production Environment (Live):**
```
https://cleared.id/portal
```
Use this portal to:
- Manage live verifications
- Generate production API keys
- Monitor real-time activity and results

### First Login

1. **Navigate to the Admin Portal**
   - Click the environment-appropriate URL above
   - Or click **"Admin Portal"** from the main website

2. **Log In**
   - Enter the email you registered with
   - Receive and enter the 6-digit code
   - You'll be logged into the Admin Portal

3. **Dashboard Overview**
   - View your organisation details
   - See verification statistics
   - Access integration settings
   - Generate API keys

## Step 4: Generating Your First API Key

Once your organisation is active, you can generate your API keys.

### Generate API Key

1. **Navigate to API Integrations**
   - From the Admin Portal dashboard
   - Click **Integrations** in the left sidebar
   - Select **API Integrations**

2. **Create New Key**
   - Click **"Generate New Key"**
   - Provide a descriptive name (e.g., "Production Server", "Development")
   - Select the environment (Sandbox or Production)
   - Optionally set permissions and expiration
   - Click **"Generate"**

3. **Copy and Store Your Key**
   - The API key will be displayed **once**
   - Copy it immediately and store it securely
   - You won't be able to see it again
   - If lost, you'll need to generate a new key

4. **Test Your Key**
   - Use the verify endpoint to confirm it works:
   ```bash
   curl -X GET https://cleared.id/api/v1/merchant/auth/verify \
     -H "Authorization: Bearer YOUR_API_KEY"
   ```

### API Key Best Practices

- ✅ Generate separate keys for Sandbox and Production
- ✅ Use descriptive names for each key
- ✅ Store keys securely in environment variables
- ✅ Never commit keys to version control
- ✅ Rotate keys every 90 days
- ✅ Revoke unused or compromised keys immediately

See [Authentication](./authentication.md) for detailed security guidance.

## Admin Portal Features

### Dashboard

The Admin Portal dashboard provides:

- **Overview Statistics** – Verification counts, success rates, pending requests
- **Recent Activity** – Latest verifications and actions
- **Quick Actions** – Create verification, generate API key, view reports
- **Notifications** – Important updates and alerts

### Integrations

Manage your API integrations:

- **API Keys** – Generate, view, and revoke API keys
- **Webhooks** – Configure webhook endpoints for real-time notifications
- **IP Whitelisting** – Restrict API access to specific IP addresses
- **Usage Analytics** – Monitor API usage and performance

### Verification Management

Manage verifications across all services:

- **Identity Verifications** – View and manage ID checks
- **Address Verifications** – Track address validation requests
- **Employment & Income** – Monitor employment verification status
- **Background Checks** – View screening results
- **Company Verifications** – Manage corporate checks

### Reports & Analytics

Access comprehensive reporting:

- **Usage Reports** – API calls, verification volumes, trends
- **Billing Reports** – Credit usage, cost breakdowns, invoices
- **Performance Metrics** – Success rates, processing times, SLA compliance
- **Export Data** – Download reports in CSV, PDF, or Excel format

### Settings

Configure your organisation:

- **Organisation Details** – Update company information
- **Team Members** – Add/remove administrators
- **Billing** – Manage payment methods and invoices
- **Notifications** – Configure email alerts and preferences
- **Security** – Enable 2FA, view audit logs, manage sessions

## Multi-User Access

You can invite team members to access the Admin Portal.

### Adding Team Members

1. **Navigate to Settings**
   - Click **Settings** in the sidebar
   - Select **Team Members**

2. **Invite User**
   - Click **"Invite Team Member"**
   - Enter their email address
   - Assign role and permissions
   - Click **"Send Invitation"**

3. **They Receive Email**
   - They'll receive an invitation email
   - They follow the registration process
   - They gain access based on assigned permissions

### User Roles

| Role | Permissions |
|------|-------------|
| **Owner** | Full access, billing, team management |
| **Administrator** | Full access except billing |
| **Developer** | API keys, integrations, view verifications |
| **Analyst** | View-only access, reports, analytics |
| **Support** | View verifications, respond to queries |

## Sandbox vs Production

### Sandbox Environment

**Purpose:** Safe testing and development

**Characteristics:**
- ✅ Free to use (no credit charges)
- ✅ Test data and dummy results
- ✅ No real personal information processed
- ✅ Same API structure as production
- ✅ Unlimited test requests

**Use Cases:**
- Initial API integration
- Testing edge cases and error handling
- Developer training
- Pre-production validation

### Production Environment

**Purpose:** Live operations with real data

**Characteristics:**
- 💳 Credit-based pricing (per verification)
- 🔒 Real personal and business data
- 📊 SLA guarantees
- 🛡️ Full compliance and security measures
- ⚡ Production-level performance

**Use Cases:**
- Live customer verifications
- Production applications
- Real-time screening and checks
- Compliance-critical operations

### Switching Between Environments

To switch environments:

1. **Different Base URLs**
   - Sandbox: `https://qa.cleared.id/api/v1/merchant`
   - Production: `https://cleared.id/api/v1/merchant`

2. **Different API Keys**
   - Generate separate keys for each environment
   - Never use production keys in sandbox (or vice versa)

3. **Different Admin Portals**
   - Manage each environment independently
   - Separate dashboards, settings, and reports

## Billing & Credits

### Credit System

The SWF Cloud API uses a credit-based system:

- Each verification type costs a specific number of credits
- Credits are deducted upon verification initiation
- Failed verifications may be refunded (case by case)
- Purchase credits in advance or set up auto-top-up

### Credit Pricing

Typical credit costs (may vary by service):

| Service | Cost (Credits) |
|---------|----------------|
| Identity Verification | 5 credits |
| Address Verification | 3 credits |
| Employment Verification | 8 credits |
| Income Verification | 10 credits |
| Background Check | 15 credits |
| Company Verification | 7 credits |
| Digital Signature (regular) | 1 credit |
| Digital Signature (certified) | 5 credits |

### Purchasing Credits

1. **Navigate to Billing**
   - Admin Portal > Settings > Billing

2. **Purchase Credits**
   - Select a credit package
   - Enter payment details
   - Confirm purchase

3. **Auto Top-Up (Optional)**
   - Set minimum balance threshold
   - Automatically purchase credits when balance is low
   - Never run out of credits

### Billing Reports

Access detailed billing information:

- Credit usage history
- Cost breakdowns by service
- Invoice history
- Payment methods
- Tax documents

## Compliance & Security

### Data Protection

We comply with:

- **GDPR** – European data protection regulation
- **CCPA** – California privacy law
- **DPA 2018** – UK Data Protection Act
- **POPIA** – South African data protection

### Security Certifications

- **ISO 27001** – Information security management
- **SOC 2 Type II** – Security and compliance auditing
- **PCI DSS** – Payment card security (for payment verifications)

### Your Responsibilities

As an API client, you must:

- ✅ Obtain proper consent from users before verification
- ✅ Securely store API keys and credentials
- ✅ Comply with applicable data protection laws
- ✅ Use verification data only for stated purposes
- ✅ Implement proper data retention and deletion policies

## Support During Onboarding

### Getting Help

If you need assistance during onboarding:

**Email Support:**
- [support@cleared.id](mailto:support@cleared.id)
- Response time: 2-4 business hours

**Live Chat:**
- Available in the Admin Portal
- Monday-Friday, 9am-5pm GMT

**Documentation:**
- [docs.cleared.id](https://docs.cleared.id)
- Comprehensive guides and tutorials

**Status Updates:**
- Check application status in Admin Portal
- Email notifications for status changes

### Common Onboarding Issues

**Issue: "Organisation pending review for several days"**
- Contact support with your organisation details
- We may need additional documentation

**Issue: "Cannot generate API keys"**
- Ensure organisation status is "Active"
- Check you have appropriate admin permissions
- Contact support if issue persists

**Issue: "API key not working"**
- Verify you're using the correct environment URL
- Check key was copied correctly (no extra spaces)
- Use the auth/verify endpoint to test

## Next Steps

After completing registration and onboarding:

1. ✅ **Organisation registered and approved**
2. ✅ **API key generated**
3. ✅ **Authentication tested**

**Continue to:**
- **[Authentication](./authentication.md)** – Detailed authentication guide
- **[Endpoints](./endpoints.md)** – Explore available API endpoints
- **Service-specific documentation** – Learn about each verification type

**Ready to integrate?**
- Start with sandbox environment
- Test your integration thoroughly
- Switch to production when ready

---

← [Authentication](./authentication.md) | [Endpoints](./endpoints.md) →

