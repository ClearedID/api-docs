# Messaging (Merchant SMS API)

Cleared provides organisation-scoped **SMS messaging**. Merchants send texts using **approved templates only**. Message **classification** is `messageType` (`otp` | `notification` | `alert` | `reminder` | `other`) — not separate products or webhook types.

Delivery is operated by Cleared. Merchants do not choose a delivery route, supply delivery credentials, or see how Cleared routes messages.

**Important:** `accepted` means Cleared accepted the message for delivery. It does **not** mean the handset received it. Final status arrives asynchronously via webhooks and status polling.

**Base paths**

| Surface | Path prefix |
|---------|-------------|
| Merchant (recommended) | `/api/v1/merchant/sms/…` |
| Alternate consumer prefix | `/api/v1/{apiConsumer}/sms/…` |

### Not the QA sandbox inbox

QA / non-production environments also expose a **sandbox mock inbox** for inspecting messages Cleared would have sent when live SMS/email is off (`/api/v1/public/sandbox/sms`, `/api/v1/public/sandbox/emails`). That inbox is **not** this Merchant Messaging API.

## Authentication / scopes

Authenticate with a Merchant API key or portal session. Required scopes (API keys) / privileges (portal):

| Scope | Privilege | Purpose |
|-------|-----------|---------|
| `sms:messages:send` | `sms_messages_send` | Send / retry messages |
| `sms:messages:read` | `sms_messages_read` | List / get messages |
| `sms:templates:read` | `sms_templates_read` | List templates |
| `sms:templates:manage` | `sms_templates_manage` | Create / update / archive / delete templates |

Legacy scopes `sms:otp:send` / `sms:otp:read` map to the same message send/read privileges.

Organisation is always taken from the credential — never from the request body.

## Creating templates

Templates have a **message type**: `otp` | `notification` | `alert` | `reminder` | `other` (default `notification`). Type is set on create and **cannot be changed** later.

### Template review

Saving a template (create or content update) returns immediately with `status: pending_review`. Automatic review runs **asynchronously** and then sets `active`, `rejected`, or leaves `pending_review` when a person must follow up (for example message type `other`).

| `status` | Meaning |
|----------|---------|
| `pending_review` | Queued or in automatic review (or waiting on a compliance follow-up) |
| `active` | Approved — **only** `active` templates can be used to send |
| `rejected` | Rejected with `reviewReason` |
| `archived` | Archived (not sendable) |

Response fields include `reviewReason`, `reviewedAt`, `reviewSource` (`ai` | `rule` | `system`), plus tracking `reviewQueuedAt`, `reviewStartedAt`, and `reviewAttemptCount`.

`GET …/templates/{templateId}` re-checks pending templates: if review never started or has been pending too long, it triggers another review attempt. Clients that show a template under review should poll this GET every few seconds until status leaves `pending_review`.

Rejected templates and type-`other` submissions notify Cleared compliance operations (default `compliance-ops@cleared.id`).

```http
POST /api/v1/merchant/sms/templates
Authorization: Bearer YOUR_API_KEY
Content-Type: application/json
```

```json
{
  "name": "Login code",
  "messageType": "otp",
  "content": "Hi {{name}}, your Cleared verification code is {{otp}}."
}
```

Non-OTP example:

```json
{
  "name": "Appointment reminder",
  "messageType": "reminder",
  "content": "Hi {{name}}, your appointment is on {{date}} at {{time}}."
}
```

- **OTP** templates **must** include `{{otp}}`.
- **Other** types **must not** include `{{otp}}` (use placeholders such as `{{name}}` instead).
- Placeholder names match `[A-Za-z][A-Za-z0-9_]{0,49}`. Do not put `otp` inside `variables` when sending — for OTP sends, supply `otp` as its own field.

Edits create a new template version (content/name only — not message type). Historical SMS logs keep the version that was used. Prefer archive over delete.

```http
GET    /api/v1/merchant/sms/templates
GET    /api/v1/merchant/sms/templates/{templateId}
PUT    /api/v1/merchant/sms/templates/{templateId}
PATCH  /api/v1/merchant/sms/templates/{templateId}
POST   /api/v1/merchant/sms/templates/{templateId}/archive
DELETE /api/v1/merchant/sms/templates/{templateId}
```

## Sending a message

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

Notification / reminder send (no `otp` field):

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

`otp` is required only when the template `messageType` is `otp`. For other types, omit `otp` and pass any placeholders via `variables`.

Successful response is usually **202 Accepted**:

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

The rendered message after substitution must be **≤ 140 characters** or the API returns `SMS_RENDERED_MESSAGE_TOO_LONG`.

Legacy path `POST /api/v1/merchant/sms/otp` is accepted and behaves the same as `/sms/messages`.

## Retrying a message

```http
POST /api/v1/merchant/sms/messages
```

```json
{
  "requestId": "smsreq_01K6ABC123",
  "retry": true
}
```

Retry ladder (from previous transmission): 30s → 1 minute → 5 minutes. Maximum **4** transmissions. Request expires **1 hour** after creation. Cleared may use an alternate delivery route on retry when available.

Blocked retries return **429** with an `enforcement` object, e.g. `OTP_RETRY_TOO_SOON`, `OTP_RETRY_EXHAUSTED`, `OTP_REQUEST_EXPIRED`.

## Destination restrictions

Destinations are sanitised with Cleared’s standard phone sanitiser, then checked against `organisation.meta.allowedSmsAreaCodes` (telephone prefixes). If unset/empty, defaults are `1876` and `1658` (Jamaica). Merchants cannot override this per request.

## Listing and reading logs

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

Detail includes a merchant-safe attempt timeline and enforcement state. OTP values and internal delivery routing details are not returned.

Legacy list/get paths under `/api/v1/merchant/sms/otp` remain available.

## Delivery statuses

Normalized statuses: `created` / `pending` / `routing` / `queued` / `accepted` / `sent` / `delivered` / `undelivered` / `failed` / `rejected` / `expired` / `cancelled`.

Subscribe to public webhook event **`messageStatusUpdated`** with webhook event type **`Messaging`**. Payloads include `data.messageType` for classification. Payloads never include OTP codes or internal delivery routing details.

Legacy webhook type `SmsOtp` and event id `sms.otp.status.updated` may still appear on older subscriptions; new configs should use **`Messaging`** / **`messageStatusUpdated`**.

Example payload shape:

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

All paths above are under `/api/v1/merchant` (or `/api/v1/{apiConsumer}`).
