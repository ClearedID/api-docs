# Onboarding require email / phone (Advanced Setup)

Published page/app flags `requireEmail`, `requirePhone`, and `forceUrlEmailAddress` are enforced on the gateway customer auth and onboarding session APIs (not UI-only).

## Rules

- **requireEmail only** — `/auth/start` and `/auth/verify` accept email OTP only.
- **requirePhone only** — phone OTP only.
- **both** — first contact may be either; completing registration (or returning-user login) requires a verified second contact via `/auth/second-credential/start` + OTP on `/auth/register` or `/auth/additional-credential/complete`.
- **forceUrlEmailAddress** — phone start rejected; if `lockedEmailAddress` is sent, it must match the OTP email.
- **Phone attach** — a phone that was not the first-credential OTP must be verified; on success the number is **migrated** off any other user (`migratePhoneNumber` + `meta.userTransfers`).
- **Email attach** — if the verified second email belongs to another **active** user, registration/completion fails closed (no email transfer).
- **Session gate** — `POST /api/v1/onboarding/:pageId/session` returns `403` with `needsAdditionalCredential` when the authenticated user lacks a required contact.

Clients: `ClearedCustomerLoginForm` (core-ui) plus onboarding runtimes in `cleared-web.app` and `cleared-onboarding-pages-app`.
