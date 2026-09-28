# Messaging (Merchant SMS API)

Use the Merchant SMS API to create message templates, send SMS, retry sends, and read delivery logs for your organisation.

Classify each message with `messageType`: `otp` | `notification` | `alert` | `reminder` | `other`. Use the same field when filtering logs and handling webhooks.

## Base paths

Merchant SMS API paths use the prefix `/api/v1/merchant/sms/…`.

Related QA endpoints for inspecting sandbox SMS/email when live delivery is off:

- `GET /api/v1/public/sandbox/sms`
- `GET /api/v1/public/sandbox/emails`

Those paths are separate from the Merchant SMS API documented below.

## Authentication and scopes

Authenticate with a Merchant API key or portal session. Required scopes (API keys) / privileges (portal):

| Scope | Privilege | Purpose |
|-------|-----------|---------|
| `sms:messages:send` | `sms_messages_send` | Send / retry messages |
| `sms:messages:read` | `sms_messages_read` | List / get messages |
| `sms:templates:read` | `sms_templates_read` | List templates |
| `sms:templates:manage` | `sms_templates_manage` | Create / update / archive / delete templates |

Legacy scopes `sms:otp:send` / `sms:otp:read` map to the same message send/read privileges.

Organisation is resolved from the credential. Omit organisation id from the request body.

## Create and manage templates

Templates require a **message type**: `otp` | `notification` | `alert` | `reminder` | `other` (default `notification`). Set `messageType` on create; updates may change name/content only — changing type returns `SMS_TEMPLATE_TYPE_IMMUTABLE`.

### Template review statuses

Create or content update responds immediately with `status: pending_review`. Review continues asynchronously and then sets `active`, `rejected`, or keeps `pending_review` when a follow-up is required (for example message type `other`).

| `status` | Meaning |
|----------|---------|
| `pending_review` | Review in progress or awaiting follow-up |
| `active` | Approved — required before send |
| `rejected` | Rejected; see `reviewReason` |
| `archived` | Archived — not sendable |

Response fields include `reviewReason`, `reviewedAt`, `reviewSource` (`ai` | `rule` | `system`), and review timestamps used while status is `pending_review`.

`GET …/templates/{templateId}` re-checks pending templates. If review has not started or has been pending too long, it queues another review attempt. Poll this GET every few seconds while status is `pending_review`.

Rejected templates and type-`other` submissions notify `compliance-ops@cleared.id`.

### Create template

```http
POST /api/v1/merchant/sms/templates
Authorization: Bearer YOUR_API_KEY
Content-Type: application/json
```

OTP example:

```json
{
  "name": "Login code",
  "messageType": "otp",
  "content": "Hi {{name}}, your Cleared verification code is {{otp}}."
}
```

Reminder example:

```json
{
  "name": "Appointment reminder",
  "messageType": "reminder",
  "content": "Hi {{name}}, your appointment is on {{date}} at {{time}}."
}
```

Rules:

- OTP templates must include `{{otp}}`.
- Non-OTP templates must not include `{{otp}}` (use placeholders such as `{{name}}`).
- Placeholder names match `[A-Za-z][A-Za-z0-9_]{0,49}`.
- When sending OTP, pass `otp` as its own request field — do not put `otp` inside `variables`.

Content or name edits create a new template version. Historical SMS logs keep the version used at send time. Prefer archive over delete.

### Template endpoints

```http
GET    /api/v1/merchant/sms/templates
GET    /api/v1/merchant/sms/templates/{templateId}
PUT    /api/v1/merchant/sms/templates/{templateId}
PATCH  /api/v1/merchant/sms/templates/{templateId}
POST   /api/v1/merchant/sms/templates/{templateId}/archive
DELETE /api/v1/merchant/sms/templates/{templateId}
```

## Send a message

```http
POST /api/v1/merchant/sms/messages
Authorization: Bearer YOUR_API_KEY
Content-Type: application/json
Idempotency-Key: optional-unique-key
```

OTP send:

```json
{
  "templateId": "smstpl_abc123",
  "to": "+1 (876) 555-0198",
  "otp": "482913",
  "variables": { "name": "John" },
  "retry": false,
  "clientReference": "login_483992"
}
```

Notification / reminder send (omit `otp`):

```json
{
  "templateId": "smstpl_reminder01",
  "to": "+1 (876) 555-0198",
  "variables": {
    "name": "John",
    "date": "27 Sep",
    "time": "2:00 pm"
  },
  "clientReference": "appt_9921"
}
```

`otp` is required only when the template `messageType` is `otp`. For other types, omit `otp` and pass placeholders in `variables`.

Typical success response is **202 Accepted**:

```json
{
  "requestId": "smsreq_01K6ABC123",
  "attemptId": "smsatt_01K6DEF456",
  "status": "accepted",
  "messageType": "otp",
  "enforcement": {
    "attemptNumber": 1,
    "maxAttempts": 4,
    "remainingRetries": 3,
    "canRetryNow": false,
    "retryAvailableAt": "2026-09-26T16:00:30Z",
    "retryAfterSeconds": 30,
    "expiresAt": "2026-09-26T17:00:00Z"
  }
}
```

`status: "accepted"` means the send request was accepted. Track later outcomes with:

1. Webhook `messageStatusUpdated`, and/or
2. `GET /api/v1/merchant/sms/messages/{requestId}`

Terminal statuses include `delivered`, `undelivered`, `failed`, `rejected`, `expired`, and `cancelled` (see [Delivery statuses](#delivery-statuses)).

The rendered body after placeholder substitution must be **≤ 140 characters**, or the API returns `SMS_RENDERED_MESSAGE_TOO_LONG`.

Also available: `POST /api/v1/merchant/sms/otp` — same behaviour as `POST /api/v1/merchant/sms/messages`.

## Retry a message

```http
POST /api/v1/merchant/sms/messages
```

```json
{
  "requestId": "smsreq_01K6ABC123",
  "retry": true
}
```

Retry cooldown from the previous transmission: 30 seconds → 1 minute → 5 minutes. Maximum **4** transmissions per request. Requests expire **1 hour** after creation.

Blocked retries return **429** with an `enforcement` object and codes such as `OTP_RETRY_TOO_SOON`, `OTP_RETRY_EXHAUSTED`, or `OTP_REQUEST_EXPIRED`.

## Destination restrictions

Phone numbers are normalised before send. Your organisation may restrict destinations to specific country/area prefixes (defaults: `1876` and `1658`). Destinations outside the allowed set return `SMS_DESTINATION_NOT_ALLOWED`.

## List and read message logs

```http
GET /api/v1/merchant/sms/messages?status=&messageType=&from=&to=&areaCode=&search=&sort=&order=&page=&limit=
GET /api/v1/merchant/sms/messages/{requestId}
GET /api/v1/merchant/sms/messages/summary?from=&to=
```

| Query | Purpose |
|-------|---------|
| `status` | Filter by delivery status. Comma-separated for multiple (e.g. `delivered,failed`) |
| `messageType` | Filter by type. Comma-separated for multiple (e.g. `otp,reminder`) |
| `from` / `to` | ISO date range on `requestedAt` |
| `areaCode` | Destination telephone prefix |
| `search` | Free-text search (destination / reference) |
| `sort` / `order` | Sort field and `asc` \| `desc` |
| `page` / `limit` | Pagination |

`GET …/messages/{requestId}` returns the request, attempt timeline, and `enforcement` state for retries. The `otp` value from send is write-only and does not appear on list or detail responses.

Also available: list/get under `/api/v1/merchant/sms/otp`.

## Delivery statuses

Normalised statuses: `created` | `pending` | `routing` | `queued` | `accepted` | `sent` | `delivered` | `undelivered` | `failed` | `rejected` | `expired` | `cancelled`.

### Webhooks

Subscribe to event **`messageStatusUpdated`** with webhook event type **`Messaging`**. Payloads include `data.messageType` and status fields (see example). The `otp` value is not part of the webhook payload.

Also available: older subscriptions may use webhook type `SmsOtp` and event id `sms.otp.status.updated`. New configs should use **`Messaging`** / **`messageStatusUpdated`**.

Example:

```json
{
  "event": "messageStatusUpdated",
  "sourceType": "Messaging",
  "data": {
    "requestId": "smsreq_01K6ABC123",
    "attemptId": "smsatt_01K6DEF456",
    "status": "delivered",
    "previousStatus": "sent",
    "messageType": "otp",
    "destinationAreaCode": "1876",
    "occurredAt": "2026-09-26T16:02:11Z"
  }
}
```

## Error codes (selected)

| Code | Meaning |
|------|---------|
| `OTP_RETRY_TOO_SOON` | Cooldown not elapsed |
| `OTP_RETRY_EXHAUSTED` | Four transmissions used |
| `OTP_REQUEST_EXPIRED` | Past one-hour lifespan |
| `SMS_DESTINATION_NOT_ALLOWED` | Prefix not allowed |
| `SMS_RATE_LIMIT_EXCEEDED` | Velocity / circuit limit |
| `SMS_ORGANISATION_DISABLED` | Org SMS messaging turned off |
| `SMS_DELIVERY_UNAVAILABLE` | Messaging temporarily unable to send or retry |
| `SMS_RENDERED_MESSAGE_TOO_LONG` | Rendered body > 140 chars |
| `SMS_TEMPLATE_MISSING_OTP` | OTP template lacks `{{otp}}` |
| `SMS_TEMPLATE_OTP_NOT_ALLOWED` | Non-OTP template includes `{{otp}}` |
| `SMS_TEMPLATE_TYPE_IMMUTABLE` | Cannot change `messageType` after create |
| `SMS_OTP_REQUIRED` | `otp` field required for OTP message type |
| `SMS_MESSAGE_NOT_FOUND` | Unknown `requestId` for this organisation |

## Idempotency

Send the same `Idempotency-Key` for the same organisation to avoid duplicate SMS on HTTP retries. This is separate from intentional `retry: true` resends.

## Endpoint summary

| Method | Path | Scope |
|--------|------|-------|
| `POST` | `/sms/templates` | `sms:templates:manage` |
| `GET` | `/sms/templates` | `sms:templates:read` |
| `GET` | `/sms/templates/{templateId}` | `sms:templates:read` |
| `PUT` / `PATCH` | `/sms/templates/{templateId}` | `sms:templates:manage` |
| `POST` | `/sms/templates/{templateId}/archive` | `sms:templates:manage` |
| `DELETE` | `/sms/templates/{templateId}` | `sms:templates:manage` |
| `POST` | `/sms/messages` | `sms:messages:send` |
| `GET` | `/sms/messages` | `sms:messages:read` |
| `GET` | `/sms/messages/summary` | `sms:messages:read` |
| `GET` | `/sms/messages/{requestId}` | `sms:messages:read` |

All paths above are under `/api/v1/merchant`.
