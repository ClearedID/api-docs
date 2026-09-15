# Permissions catalog

This reference lists the privilege identifiers used for **roles and access control** in the Cleared Admin Portal (and related surfaces). Assign these permissions to roles so team members can view or change specific areas of the product.

**Source of truth:** registries in `swf-core-gateway` (`permissionRegistry.js`, `controlCentrePermissionRegistry.js`, `devopsPermissionRegistry.js`).

**Spreadsheet:** a machine-readable export of the same catalog is available as [permissions-catalog.csv](./permissions-catalog.csv) (Name, Category, Description, Id, SubCategory, Registry).

## How permissions work

- Each permission has a stable **Id** (used in APIs and role configuration) and a human-readable **Name**.
- Permissions are grouped by **category** (and optional subcategory) in the Roles UI.
- Organisation owners are treated as having full access: the `full_access` permission bypasses privilege checks.
- Control Centre permissions (`cc_*`) are only available when Control Centre permissions are enabled for the organisation.
- DevOps permissions apply to infrastructure / DevOps control-plane operators.

---

## Portal

_228 permissions_

### Admin

#### Billing

| Name | Id | Description |
|------|-----|-------------|
| View billing | `view_billing` | Access the billing area to see plans, usage, payment methods, and credit balance. |
| Add payment method | `add_payment_method` | Add a new card or payment method for the organisation’s billing. |
| Set primary payment method | `set_primary_payment_method` | Choose which payment method is used by default for charges and renewals. |
| Delete payment method | `delete_payment_method` | Remove a saved payment method so it can no longer be charged. |
| Purchase credit bundle | `purchase_credit_bundle` | Buy a pack of credits (e.g. for verifications or screening) when using pay-as-you-go. |
| Switch billing mode | `switch_billing_mode` | Change the organisation between subscription billing and pay-as-you-go (or vice versa). |

#### Departments

| Name | Id | Description |
|------|-----|-------------|
| View departments | `view_organisation_departments` | List the organisation’s departments used when assigning team members. |
| Manage departments | `manage_organisation_departments` | Create, edit, or remove organisation departments. |

#### Integrations

| Name | Id | Description |
|------|-----|-------------|
| View integrations | `view_integrations` | Access the integrations screen to see API keys, webhook URLs, and allowed domains (read-only without other integration permissions). |
| Generate API key | `generate_api_key` | Create a new API key so external systems can call Cleared APIs on behalf of the organisation. |
| Update webhook endpoint | `update_webhook_endpoint` | Change the URL where Cleared sends webhook events (e.g. verification completed). |
| Update landing page | `update_landing_page` | Change the URL or page where candidates are sent to start a verification (e.g. your own hosted flow). |
| Add integration domain | `add_integration_domain` | Allow another domain (e.g. your app’s origin) to use the organisation’s integrations or embed verification. |
| Remove integration domain | `remove_integration_domain` | Revoke a previously allowed domain so it can no longer use the organisation’s integrations. |

#### Notifications

| Name | Id | Description |
|------|-----|-------------|
| Manage notification groups | `manage_notification_groups` | Define and edit groups that control who receives which system or product notifications. |

#### Organisations

| Name | Id | Description |
|------|-----|-------------|
| View organisation details | `view_organisation_details` | See the organisation’s name, branding, and high-level settings (e.g. enterprise status) without editing. |
| Request enterprise access | `update_organisation_enterprise_access` | Submit or trigger a request for enterprise-level features or terms for the organisation. |
| Update organisation details | `update_organisation_details` | Change the organisation’s name, branding, or other settings stored at org level. |
| View organisation documents | `view_organisation_documents` | Access documents or files stored under the organisation (e.g. contracts, certificates). |
| Create organisation documents | `create_organisation_documents` | Upload or create new documents that belong to the organisation. |
| Create organisation | `create_organisation` | Create a new organisation so a new team or brand can use the portal under its own billing and settings. |
| Upload organisation logo | `upload_organisation_logo` | Upload or replace the logo shown in the portal and on shared links for this organisation. |
| Delete organisation logo | `delete_organisation_logo` | Remove the organisation logo so the default or no logo is shown. |

#### Roles

| Name | Id | Description |
|------|-----|-------------|
| View Role Management | `view_role_management` | Access the Roles & permissions screen to see which roles exist and what permissions they carry. |
| Manage Roles and Permissions | `manage_roles_and_permissions` | Create or edit roles, assign permission sets to roles, and remove roles; required to change what team members can do. |

#### Subscriptions

| Name | Id | Description |
|------|-----|-------------|
| View all subscriptions | `view_subscriptions` | List every plan and add-on the organisation is subscribed to; needed for billing and capacity decisions. |
| View subscription details | `view_subscription_details` | Open one subscription to see plan name, status, renewal date, and linked usage or limits. |
| Start a subscription | `create_subscription` | Enrol the organisation in a new plan or add-on (e.g. identity, screening) so the team can use that feature. |
| Cancel a subscription | `delete_subscription` | End an existing plan or add-on so it stops renewing and the feature is no longer available. |

#### Team

| Name | Id | Description |
|------|-----|-------------|
| View all team members | `view_organisation_team` | List everyone in the organisation’s team, their roles, and invite status. |
| Remove team members | `delete_organisation_team_member` | Remove a user from the organisation so they lose access to the portal and APIs for this org. |
| Invite team members | `update_organisation_team_invitations` | Send email invitations so new users can join the organisation’s team. |
| Update team member | `update_organisation_team_member` | Change a team member’s assigned role(s), permission overrides, or department. |

#### Webhooks

| Name | Id | Description |
|------|-----|-------------|
| View webhook configs | `view_webhook_configs` | List and view central webhook endpoint configs (onboarding, documents, etc.). |
| Manage webhook configs | `manage_webhook_configs` | Create, edit, and delete central webhook endpoint configs. |
| View webhook event logs | `view_webhook_event_logs` | View unified event logs for all webhook types (onboarding, documents) with filters. |
| Rerun webhook events | `rerun_webhook_events` | Queue failed or skipped webhook events for another delivery attempt (up to the organisation additional-rerun limit). |

### Cleared Station

#### Audit

| Name | Id | Description |
|------|-----|-------------|
| View station audit log | `station.audit.view` | View organisation-wide station audit activity. |

#### Capture

| Name | Id | Description |
|------|-----|-------------|
| Use capture ID tool | `station.captureId.use` | Start ID capture from merchant mode on a station. |
| Use capture form tool | `station.captureForm.use` | Capture physical forms at a station. |
| Review captured form | `station.captureForm.review` | Review and approve OCR-extracted form data. |
| View form capture templates | `station.captureForm.templates.view` | List form capture templates for stations. |
| Create form capture template | `station.captureForm.templates.create` | Create a new OCR form capture template. |
| Update form capture template | `station.captureForm.templates.update` | Edit form capture template fields and samples. |
| Disable form capture template | `station.captureForm.templates.disable` | Deactivate a form capture template. |

#### Devices

| Name | Id | Description |
|------|-----|-------------|
| View station devices | `station.devices.view` | List tablets and browsers bound to stations. |
| Set up station device | `station.devices.setup` | Register or pair a device with a station. |
| Revoke station device | `station.devices.revoke` | Revoke a device token and disconnect it from a station. |

#### Profiles

| Name | Id | Description |
|------|-----|-------------|
| View station profiles | `station.profiles.view` | See profiles onboarded through stations. |
| View sensitive station profile data | `station.profiles.viewSensitive` | View sensitive profile fields from station onboarding. |
| Search station profiles | `station.profiles.search` | Search profiles by name, contact, or station session. |
| Export station profiles | `station.profiles.export` | Export station profile data for reporting. |

#### Reports

| Name | Id | Description |
|------|-----|-------------|
| View station reports | `station.reports.view` | View station operational reports. |
| Export station reports | `station.reports.export` | Export station session and onboarding reports. |

#### Runtime

| Name | Id | Description |
|------|-----|-------------|
| Access station runtime | `station.runtime.access` | Use the kiosk runtime or start foundation session APIs. |
| Use customer mode | `station.runtime.customerMode.use` | Run customer self-service mode on a station. |
| Use merchant mode | `station.runtime.merchantMode.use` | Run staff-assisted merchant mode on a station. |
| Bypass staff PIN | `station.runtime.staffPin.bypass` | Bypass staff PIN unlock when already authenticated with permission. |

#### Sessions

| Name | Id | Description |
|------|-----|-------------|
| View active station sessions | `station.sessions.view` | List active and recent station sessions. |
| View station session details | `station.sessions.viewDetail` | Open a station session to see customer, workflow, and timeline. |
| Force reset station session | `station.sessions.forceReset` | Force close an active station session and release the station lock. |
| Cancel station session | `station.sessions.cancel` | Cancel an in-progress station session. |
| View station session audit | `station.sessions.audit.view` | See the event timeline for a station session. |

#### Settings

| Name | Id | Description |
|------|-----|-------------|
| View station settings | `station.settings.view` | View Cleared Station organisation settings. |
| Update station settings | `station.settings.update` | Change Cleared Station organisation settings. |

#### Stations

| Name | Id | Description |
|------|-----|-------------|
| View stations | `station.stations.view` | List configured kiosk stations for the organisation. |
| Create station | `station.stations.create` | Create a new kiosk station configuration. |
| Update station | `station.stations.update` | Edit station settings, modes, and policies. |
| Disable station | `station.stations.disable` | Disable a station so it cannot accept new sessions. |
| Delete station | `station.stations.delete` | Permanently delete or archive a station configuration. |

#### Surface

| Name | Id | Description |
|------|-----|-------------|
| Access Cleared Station surface | `station.surface.access` | Enter the Cleared Station operational area in the portal. |
| View station dashboard | `station.dashboard.view` | See the station overview dashboard with live station status. |

#### Workflows

| Name | Id | Description |
|------|-----|-------------|
| View station workflows | `station.workflows.view` | See onboarding pages exposed to stations. |
| Expose workflow to station | `station.workflows.expose` | Expose an onboarding page as a station workflow. |
| Assign workflow to station | `station.workflows.assign` | Assign or reorder workflows on a specific station. |
| Override station workflow settings | `station.workflows.overrideSettings` | Set kiosk-specific overrides for an exposed workflow. |

### Dashboard

#### Overview

| Name | Id | Description |
|------|-----|-------------|
| View dashboard | `view_dashboard` | Access the main portal dashboard (stats, quick actions, recent activity). |

#### Reports

| Name | Id | Description |
|------|-----|-------------|
| View operations report | `view_operations_report` | View organisation operations metrics (signed applications, ID verification outcomes, daily blockers) and export CSV for invoice checks. |

### Digital IDs

#### Audit

| Name | Id | Description |
|------|-----|-------------|
| View digital ID audit | `view_digital_id_audit` | View the audit log of digital ID events (issuance, presentation, revocation) for compliance. |
| Export digital ID audit | `export_digital_id_audit` | Export the digital ID audit log (e.g. CSV) for external reporting or archiving. |

#### List

| Name | Id | Description |
|------|-----|-------------|
| View all digital IDs | `view_digital_ids` | List digital ID credentials (e.g. mobile-based IDs) that have been issued or linked to the organisation. |

### Documents

#### Documents

| Name | Id | Description |
|------|-----|-------------|
| View all documents | `view_documents_list` | List all documents (e.g. contracts, forms) that the organisation has created or received for signing. |
| Create document | `create_document` | Create a new document (upload or from template) so it can be sent for signing. |
| View document | `view_document` | Open a single document to read its contents and see signing status. |
| Download document | `download_document` | Download a copy of a document file (e.g. PDF) to your device. |
| Reuse document | `reuse_document` | Use an existing document as the basis for a new one (e.g. same content, new signers). |
| Cancel document | `cancel_document` | Cancel a document so signers can no longer sign and it is marked cancelled. |
| Remind document signers | `remind_document_signers` | Send a reminder email to signers who have not yet signed. |
| Create template from document | `create_template_from_document` | Save a completed or draft document as a reusable template for future sends. |
| Update document | `update_document` | Edit a document’s content or settings before or after sending. |
| Send document | `send_document` | Send a document to signers so they receive the signing link and can complete it. |
| Delete document | `delete_document` | Permanently remove a document from the organisation’s document list. |
| Add document to envelope | `add_document_to_envelope` | Add one or more documents to an envelope so they can be sent together for signing. |
| Publish document version | `publish_document_version` | Publish staged document changes so they become the live version used when sending for signing. |
| Restore document version | `restore_document_version` | Restore a previous document version as the new pending version (for review and re-publish). |

#### Envelope Templates

| Name | Id | Description |
|------|-----|-------------|
| View all envelope templates | `view_envelope_templates_list` | List all saved envelope templates (document sets + signer roles) the organisation can reuse. |
| Create envelope template | `create_envelope_template` | Create a new envelope template so the team can send the same document set repeatedly. |
| Duplicate envelope template | `duplicate_envelope_template` | Copy an existing envelope template to create a variant. |
| Delete envelope template | `delete_envelope_template` | Remove an envelope template so it can no longer be used. |
| Update envelope template | `update_envelope_template` | Edit an envelope template’s documents or signer configuration. |
| Use envelope template | `use_envelope_template` | Create a new envelope from a template (pre-filled document set and roles). |

#### Envelopes

| Name | Id | Description |
|------|-----|-------------|
| Create envelope | `create_envelope` | Create a new envelope to group documents and send them in one signing flow. |
| View all envelopes | `view_envelopes_list` | List all envelopes (groups of documents sent together) and their status. |
| View envelope | `view_envelope` | Open one envelope to see its documents and signing progress. |
| Send envelope | `send_envelope` | Send an envelope to signers so they receive links to sign all documents in it. |
| Cancel envelope | `cancel_envelope` | Cancel an envelope so no further signing can occur. |
| Remove document from envelope | `remove_document_from_envelope` | Remove a document from an envelope before sending (or to correct the set). |
| Download envelope documents | `download_envelope_documents` | Download the documents contained in an envelope as files. |

#### Templates

| Name | Id | Description |
|------|-----|-------------|
| View all document templates | `view_document_templates_list` | List all saved document templates the organisation can use to create new documents. |
| Create document template | `create_document_template` | Create a new template (e.g. standard contract) so the team can send documents from it repeatedly. |
| Duplicate document template | `duplicate_document_template` | Copy an existing template to create a variant or backup. |
| Delete document template | `delete_document_template` | Remove a template so it can no longer be used to create documents. |
| Use document template | `use_document_template` | Start a new document from a template (pre-filled content and fields). |
| Publish document template version | `publish_document_template_version` | Publish staged template changes so they become the live version used for new documents and signing. |
| Restore document template version | `restore_document_template_version` | Restore a previous template version as the new pending version (for review and re-publish). |

### Identity Verification

#### Batch

| Name | Id | Description |
|------|-----|-------------|
| View all batch verification requests | `view_batch_verification_requests` | List all batch verification jobs (e.g. CSV uploads) and their progress or outcome. |
| Upload batch verification request | `upload_batch_verification_request` | Upload a file (e.g. CSV) to start a batch of identity or address verifications. |
| Submit batch verification request | `submit_batch_verification_request` | Submit or start a batch job so the system processes multiple verification requests from the uploaded data. |

#### Requests

| Name | Id | Description |
|------|-----|-------------|
| View all ID verification requests | `view_identity_verification_requests_list` | List every ID verification request (e.g. link sent to a candidate) and their status. |
| View ID verification request details | `view_identity_verification_request_details` | Open one request to see expiry, link, and whether it was completed or withdrawn. |
| View verification request workflow | `view_verification_request_workflow` | See the Workflow timeline on a verification request (ops actions such as waiting on TRN or re-upload). |
| Extend verification request | `extend_verification_request` | Give a candidate more time to complete a verification by extending the request expiry. |
| Withdraw verification request | `withdraw_verification_request` | Cancel an active request so the link no longer works and the candidate cannot submit. |

#### Templates

| Name | Id | Description |
|------|-----|-------------|
| Create verification request template | `create_verification_request_template` | Define a reusable template (e.g. which checks to request) so you can send the same type of request repeatedly. |
| Update verification request template | `update_verification_request_template` | Change the checks or settings of an existing verification request template. |
| Delete verification request template | `delete_verification_request_template` | Remove a template so it can no longer be used to create new requests. |

#### Verify

| Name | Id | Description |
|------|-----|-------------|
| Verify reference number | `verify_reference_number` | Look up a verification request by its reference number (e.g. to check status without logging in). |

### Onboarding Apps

#### AI Builder

| Name | Id | Description |
|------|-----|-------------|
| AI generate onboarding app | `ai_generate_onboarding_app` | Use AI to generate or enhance onboarding app drafts (pages, forms, journey, rules). |

#### Apps

| Name | Id | Description |
|------|-----|-------------|
| View onboarding apps list | `view_onboarding_apps_list` | List onboarding apps the organisation has created. |
| Create onboarding app | `create_onboarding_app` | Create a new onboarding app with the default legacy template. |
| Update onboarding app | `update_onboarding_app` | Edit draft onboarding app settings and content. |
| Publish onboarding app | `publish_onboarding_app` | Publish staged onboarding app changes to the public /apps/{slug} runtime. |
| Delete onboarding app | `delete_onboarding_app` | Permanently delete an onboarding app. |

#### Conversion

| Name | Id | Description |
|------|-----|-------------|
| Convert legacy page to app | `convert_onboarding_page_to_app` | Convert a legacy onboarding page into a draft onboarding app and rollback before publish. |

#### Flow

| Name | Id | Description |
|------|-----|-------------|
| Manage onboarding app flow | `manage_onboarding_app_flow` | Edit the visual journey flow for an onboarding app. |

#### Forms

| Name | Id | Description |
|------|-----|-------------|
| Manage onboarding app forms | `manage_onboarding_app_forms` | Create, edit, and delete forms in an onboarding app library. |

#### Rules

| Name | Id | Description |
|------|-----|-------------|
| Manage onboarding app rules | `manage_onboarding_app_rules` | Configure journey rules, visibility, entry conditions, and webhooks for an onboarding app. |

### Onboarding Pages

#### Analytics

| Name | Id | Description |
|------|-----|-------------|
| View onboarding analytics | `view_onboarding_analytics` | See funnel, trends, and lead-capture field breakdowns for an onboarding page. |

#### Custom Fields

| Name | Id | Description |
|------|-----|-------------|
| Manage onboarding custom fields | `manage_onboarding_custom_fields` | Add, edit, or remove custom fields on an onboarding page so you collect the data you need. |

#### Entries

| Name | Id | Description |
|------|-----|-------------|
| View onboarding entries | `view_onboarding_entries` | See the list of submissions or entries (who completed the page and when). |

#### Event Logs

| Name | Id | Description |
|------|-----|-------------|
| View onboarding event logs | `view_onboarding_event_logs` | See the audit trail of events for an onboarding page (e.g. opened, submitted, bounced). |

#### Invitations

| Name | Id | Description |
|------|-----|-------------|
| Send onboarding invitations | `send_onboarding_invitations` | Send invite links or emails so candidates can access and complete an onboarding page. |

#### Lead Capture

| Name | Id | Description |
|------|-----|-------------|
| AI generate lead capture | `ai_generate_lead_capture` | Use AI-assisted tools to generate or suggest lead-capture form content (e.g. questions, fields) for onboarding pages. |

#### Pages

| Name | Id | Description |
|------|-----|-------------|
| View all onboarding pages | `view_onboarding_pages_list` | List all onboarding or lead-capture pages the organisation has created. |
| Create onboarding page | `create_onboarding_page` | Create a new onboarding page so candidates can submit info or complete a flow via a shared link. |
| Delete onboarding page | `delete_onboarding_page` | Permanently remove an onboarding page and (depending on config) its collected data. |
| Update onboarding page | `update_onboarding_page` | Edit an onboarding page’s fields, layout, or settings. |
| Publish onboarding page | `publish_onboarding_page` | Publish staged onboarding page changes so the public link shows the updated content. |
| Restore onboarding page version | `restore_onboarding_page` | Restore a previously published version into staging for review and re-publish. |

### Onboarding Surface

#### Surface

| Name | Id | Description |
|------|-----|-------------|
| Access onboarding workspace | `onboarding_surface_access` | Open the dedicated Onboarding workspace at /onboarding to manage legacy pages and (future) onboarding apps. |

### Profiles

#### Details

| Name | Id | Description |
|------|-----|-------------|
| View profile details | `view_profile_details` | Open a profile page; which tabs and evidence you see still depend on separate tab and evidence permissions. |
| View profile Identity tab | `view_profile_identity_tab` | See the Identity tab on a profile: document type, status, and result summary only (no ID images unless you also have view identity evidence). |
| View profile Address tab | `view_profile_address_tab` | See the Address tab: verified address and result summary only (no document or photo files unless you also have view address evidence). |
| View profile References tab | `view_profile_references_tab` | See the References tab: character or employer references and their status or response. |
| View profile Employment tab | `view_profile_employment_tab` | See the Employment tab: employment history and verification outcomes. |
| View profile Qualifications tab | `view_profile_education_tab` | See the Qualifications tab: qualification and credential verification outcomes (degrees, diplomas, certifications). |
| View profile Income tab | `view_profile_income_tab` | See the Income tab: income verification results (e.g. employer, amount, date range). |
| Export income evidence profile | `export_profile_income_evidence` | Download JSON, CSV, or PDF exports of the income evidence v2 profile when the full report is visible. |
| View profile Criminal Records tab | `view_profile_criminal_tab` | See the Criminal Records tab: background check results and any disclosures. |
| View profile Tenancy tab | `view_profile_tenancy_tab` | See the Tenancy history tab: past tenancies and landlord references. |
| View profile Screening tab | `view_profile_screening_tab` | See the Screening tab: risk screening cases and results linked to this profile. |
| Manage profile screening monitoring | `manage_profile_screening_monitoring` | Configure recurring background screening monitoring on a profile. |
| View profile Documents tab | `view_profile_documents_tab` | See documents this person has signed or is listed on as a signing party, with status and view link when complete. |

#### Evidence

| Name | Id | Description |
|------|-----|-------------|
| View identity evidence | `view_identity_evidence` | View or request watermarked ID document images and selfie/media on a profile; needed in addition to the Identity tab to see actual ID photos. |
| View address evidence | `view_address_evidence` | View address proof documents and residence photos on a profile; needed in addition to the Address tab to open or download those files. |

#### Filters

| Name | Id | Description |
|------|-----|-------------|
| Manage table filters | `manage_table_filters` | Save, edit, and delete saved filters on the profiles table (e.g. “Cleared only”, “Pending”) for yourself or your team. |

#### List

| Name | Id | Description |
|------|-----|-------------|
| View all profiles | `view_profiles` | Access the profiles list and search or filter people who have completed verifications. |

### Sales Pipe

#### Campaigns

| Name | Id | Description |
|------|-----|-------------|
| View campaigns | `campaigns_read` | List and open campaigns, enrolments, sends, and outcomes. |
| Create and edit campaigns | `campaigns_write` | Create drafts and edit campaign configuration. |
| Activate campaigns | `campaigns_activate` | Activate campaigns so Machine-Runner can enrol and send. |
| Pause campaigns | `campaigns_pause` | Pause active campaigns. |
| Archive campaigns | `campaigns_archive` | Archive campaigns. |
| View campaign enrolments | `campaigns_enrolments_read` | View enrolment lists and timelines. |
| Manage enrolments | `campaigns_enrolments_write` | Pause, resume, exit, or manually enrol organisations. |
| View scheduled sends | `campaigns_sends_read` | View planned and completed sends for campaigns. |
| View send outcomes | `campaigns_outcomes_read` | View delivery outcomes and attempts. |
| Browse Mailgun templates | `campaigns_mailgun_read` | List cached Mailgun templates for message configuration. |
| Refresh Mailgun templates | `campaigns_mailgun_refresh` | Force refresh of Mailgun template cache. |
| View email templates | `email_templates_read` | List and open org-scoped Sales Pipe email layouts. |
| Edit email templates | `email_templates_write` | Create, update, delete Sales Pipe email layouts and upload images. |
| View sender identities | `sender_identities_read` | List and open org sender identities (from address, header/footer). |
| Edit sender identities | `sender_identities_write` | Create, update, and delete sender identities and set default. |

### Screening

#### Cases

| Name | Id | Description |
|------|-----|-------------|
| Archive screening case | `archive_screening_case` | Archive or unarchive a screening case so it is hidden from or restored to active lists. |
| Create screening case | `create_screening_case` | Start a new screening case and attach people or companies to run checks. |
| Run screening search | `run_screening_search` | Execute or re-run searches (e.g. sanctions, PEP) within a screening case. |
| Contact screening candidate | `contact_screening_candidate` | Send messages or requests to a candidate linked to a screening case. |
| Request screening verification | `request_screening_verification` | Request that a candidate provide or confirm information for the screening case. |

#### Companies

| Name | Id | Description |
|------|-----|-------------|
| View all companies | `view_companies` | List companies that have been screened or are linked to screening cases. |
| View company details | `view_company` | Open one company record to see name, registration, and linked screening results. |
| Create company | `create_company` | Add a new company to the screening database so it can be screened. |
| Edit company | `edit_company` | Change a company’s name, registration number, or other details. |
| Delete company | `delete_company` | Remove a company record from the screening area. |
| Business discovery | `business_discovery` | Search external sources to discover and match companies before or after screening. |
| Manage company screening monitoring | `manage_company_screening_monitoring` | Configure recurring background screening monitoring on a company screening result. |
| Add company | `add_company` | Add a company to the screening company list so it can be screened or linked to cases. |
| Create company screening result | `create_company_screening_result` | Record or create a screening result (e.g. manual outcome) for a company. |
| Search companies | `search_companies` | Search the screening company list by name, registration, or other criteria. |
| Initiate sanctions search | `initiate_sanctions_search` | Run a sanctions list check on a company to see if it appears on restricted lists. |
| Initiate reputation search | `initiate_reputation_search` | Run a reputation or adverse media search on a company. |
| Initiate legal search | `initiate_legal_search` | Run a legal or court-record search on a company. |
| Request financial verification | `request_financial_verification` | Request that financial or credit information be verified for a company. |
| Request address verification | `request_address_verification` | Request that the company’s address be verified (e.g. proof of business address). |

#### Dashboard

| Name | Id | Description |
|------|-----|-------------|
| View screening dashboard | `view_screening_dashboard` | Access the screening overview (counts, recent cases, quick actions). |

#### People

| Name | Id | Description |
|------|-----|-------------|
| View all people | `view_people` | List people that have been screened or are linked to screening cases. |
| View person details | `view_person` | Open one person record to see name, identifiers, and linked screening results. |
| Edit person | `edit_person` | Change a person’s name, DOB, or other details in the screening database. |
| Delete person | `delete_person` | Remove a person record from the screening area (e.g. for data minimisation). |
| Create person | `create_person` | Add a new person to the screening database so they can be screened. |

#### Requests

| Name | Id | Description |
|------|-----|-------------|
| Submit screening request | `submit_screening_request` | Run a new screening (e.g. sanctions, PEP, adverse media) for a person or company. |

#### Results

| Name | Id | Description |
|------|-----|-------------|
| View all screening results | `view_screening_results_list` | List all risk screening cases and their outcomes (e.g. sanctions, PEP, adverse media). |
| View screening result | `view_screening_result` | Open one screening case to see hits, sources, and risk flags in detail. |

#### Templates

| Name | Id | Description |
|------|-----|-------------|
| Create screening template | `create_screening_template` | Define a reusable set of screening checks (e.g. sanctions + PEP) to run on cases. |
| Update screening template | `update_screening_template` | Change the checks or settings of an existing screening template. |
| Archive screening template | `archive_screening_template` | Archive a template so it can no longer be applied to new cases. |

### Settings

#### Account

| Name | Id | Description |
|------|-----|-------------|
| View account settings | `view_account_settings` | Access the account screen to see your email, name, and security-related settings. |
| Request email change | `request_email_change` | Start the process to change the email address used to log in (triggers verification email). |
| Verify email change | `verify_email_change` | Complete an email change by confirming the new address via the link sent to that address. |

#### Notifications

| Name | Id | Description |
|------|-----|-------------|
| View in-app notifications | `view_in_app_notifications` | Use the notifications bell and drawer in the portal to read and dismiss in-app alerts. |

#### Preferences

| Name | Id | Description |
|------|-----|-------------|
| View preferences | `view_preferences` | Access the preferences screen to see language, currency, and display options (read-only without update permission). |
| Update preferences | `update_preferences` | Change your language, currency, date format, or other personal display preferences. |

#### Security

| Name | Id | Description |
|------|-----|-------------|
| View Security settings | `view_security_settings` | Access Security for password, MFA, and trusted devices. |
| Manage security credentials | `manage_security_credentials` | Change password, configure SMS MFA and authenticator apps. |

### Support

#### Help

| Name | Id | Description |
|------|-----|-------------|
| View support | `view_support` | Access the help and support area (docs, contact options); not used for access control as support is available to all. |

### System

#### Access

| Name | Id | Description |
|------|-----|-------------|
| Full Access | `full_access` | Bypasses all permission checks; granted automatically to organisation owners. |

### Verification Links

#### List

| Name | Id | Description |
|------|-----|-------------|
| View all verification links | `view_verification_links_list` | List all shareable verification links (e.g. one-time or reusable) and their status. |

#### Manage

| Name | Id | Description |
|------|-----|-------------|
| Create verification link | `create_verification_link` | Generate a new verification link to share with a candidate (e.g. embed in your app or email). |
| Update verification link | `update_verification_link` | Edit a verification link’s expiry, type, or other settings. |
| Delete verification link | `delete_verification_link` | Revoke or delete a verification link so it can no longer be used. |

---

## Control Centre

_35 permissions_

### Control Centre

#### Case

| Name | Id | Description |
|------|-----|-------------|
| Open case | `cc_case_open` | View a case record, snapshots, and linked request metadata. |
| View closed cases | `cc_case_view_closed` | Include closed cases in queue search and confirm viewing a closed case in the workspace. |
| Update case | `cc_case_update` | Edit case fields, assign, change stage, soft-drop, record workspace open, link verification results, and income evidence v2 ops actions (`POST .../cases/:id/income-evidence-v2/...` aggregate, document, integrity routes). |
| View screening hits | `cc_case_screening_view` | View screening hits attached to a case. |
| Rerun screening | `cc_case_screening_rerun` | Trigger a screening rerun for a case. |
| View case verification requests | `cc_case_verification_requests_view` | View verification requests shown on a case. |
| View case comments | `cc_case_comments_view` | Read comments and threads on a case. |
| Post case comments | `cc_case_comments_post` | Post comments on a case. |
| View case activity | `cc_case_activity_view` | View activity and audit entries for a case. |
| View contact messages | `cc_case_contact_messages_view` | View contact-centre chats and messages linked to a case. |
| Send contact messages | `cc_case_contact_messages_send` | Send messages in contact-centre chats for a case. |
| View trust score | `cc_case_trust_score_view` | View or compute trust-score previews for a case. |

#### CourtHits

| Name | Id | Description |
|------|-----|-------------|
| View CourtHits | `cc_courthits_view` | Access the CourtHits area when available. |

#### Dashboard

| Name | Id | Description |
|------|-----|-------------|
| View operations dashboard | `cc_dashboard_view` | View operations dashboard metrics, attention signals, and performance trends. |

#### ID Validation

| Name | Id | Description |
|------|-----|-------------|
| View ID validation queue | `cc_id_validation_queue_view` | View the ID validation queue (pending, in progress, flagged, failed). |
| Validate ID queue items | `cc_id_validation_queue_validate` | Run CBS/manual validation and retry on ID validation queue items. |
| Decide ID validation / deferred clear | `cc_id_validation_queue_decide` | Mark queue items valid/invalid (fraud reject) and request deferred clearance. |

#### Initial Review

| Name | Id | Description |
|------|-----|-------------|
| View Initial Review Triage | `cc_initial_review_triage_view` | View the Initial Review Triage queue and case detail. |
| Claim Initial Review cases | `cc_initial_review_triage_claim` | Claim, release, and start Initial Review triage on assigned identity results. |
| Assign Initial Review cases | `cc_initial_review_triage_assign` | Assign or reassign Initial Review triage to agents. |
| Decide Initial Review | `cc_initial_review_triage_decide` | Advance or reject identity results from Initial Review Triage. |

#### People

| Name | Id | Description |
|------|-----|-------------|
| View people | `cc_people_view` | Search and list people in the Control Centre. |

#### Queue

| Name | Id | Description |
|------|-----|-------------|
| View queue | `cc_queue_view` | See the case queue and list cases for the organisation. |
| Manage queue | `cc_queue_manage` | Create cases from results, open workspace transitions, and other queue management actions. |

#### Requests

| Name | Id | Description |
|------|-----|-------------|
| View requests | `cc_requests_view` | See verification and related requests lists. |
| Manage requests | `cc_requests_manage` | Create or update requests, reminders, contact logs, and related actions. |

#### Supervisor

| Name | Id | Description |
|------|-----|-------------|
| View pipeline review | `cc_pipeline_review_view` | View global identity pipeline status, clearance speed stats, and case lookup for ops supervisors. |

#### Tasks

| Name | Id | Description |
|------|-----|-------------|
| View tasks | `cc_tasks_view` | See the tasks / actions inbox. |
| Manage tasks | `cc_tasks_manage` | Create and update tasks and actions. |

#### Team

| Name | Id | Description |
|------|-----|-------------|
| Search assignees | `cc_assignees_search` | Search organisation assignees for case assignment. |

#### Tools

| Name | Id | Description |
|------|-----|-------------|
| Use ID Generator | `cc_tools_id_generator` | Use the ID generator tool (templates, generate). |
| Use Face Match | `cc_tools_face_match` | Use the face-match tool. |

#### Verification

| Name | Id | Description |
|------|-----|-------------|
| Read verification data (proxy) | `cc_verification_proxy_read` | Read reports and data via Control Centre proxy routes (GET). |
| Write verification actions (proxy) | `cc_verification_proxy_write` | Submit decisions and mutations via Control Centre proxy routes (POST/PATCH). |
| View journey maps | `cc_verification_journeys_view` | Browse verification results and journey map timelines across services. |

---

## DevOps

_3 permissions_

### DevOps

#### Actions

| Name | Id | Description |
|------|-----|-------------|
| Run DevOps actions | `run_devops_actions` | Execute deployment actions, pipelines, and coordinated deploy sessions. |

#### Code

| Name | Id | Description |
|------|-----|-------------|
| View DevOps projects | `view_devops_projects` | View Code projects, repositories, and related DevOps workspace content. |

#### Infrastructure

| Name | Id | Description |
|------|-----|-------------|
| Manage infrastructure control plane | `manage_infrastructure` | Promote blue/green slots, stop/start slots, and force-off draining slots from the control plane. |

