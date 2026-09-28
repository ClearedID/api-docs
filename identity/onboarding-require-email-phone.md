# Onboarding require email / phone (Advanced Setup)

Published page/app flags `requireEmail`, `requirePhone`, and `forceUrlEmailAddress` control which contacts the customer auth and onboarding session APIs accept.

## Rules

- **requireEmail only** — `/auth/start` and `/auth/verify` accept email OTP only.
- **requirePhone only** — phone OTP only.
- **both** — first contact may be either; completing registration (or returning-user login) requires a verified second contact via `/auth/second-credential/start` + OTP on `/auth/register` or `/auth/additional-credential/complete`.
- **forceUrlEmailAddress** — phone start rejected; if `lockedEmailAddress` is sent, it must match the OTP email.
- **Phone attach** — a phone that was not the first-credential OTP must be verified before it is attached to the account.
- **Email attach** — if the verified second email belongs to another **active** user, registration/completion fails (no email transfer).
- **Session gate** — `POST /api/v1/onboarding/:pageId/session` returns `403` with `needsAdditionalCredential` when the authenticated user lacks a required contact.
