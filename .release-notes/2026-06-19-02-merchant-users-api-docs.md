# Merchant Users API documentation

## Added

- **Merchant Users API** guide (`merchant-users.md`): documents `POST /api/v1/merchant/users/create` — authentication, single and batch payloads, opaque success response, and example `curl`.
- Linked from the main API docs README under Verification Requests.

## How to test (QA)

1. Open `merchant-users.md` in the api-docs repo (or published docs site after sync).
2. Confirm endpoint path, field names (`emailAddress`, not `email`), and Bearer API key auth match QA gateway behaviour.
3. Follow the example `curl` against QA with a valid API key and confirm the doc matches the live response shape.
