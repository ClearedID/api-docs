# Initial Review and Due Diligence webhook docs

## Added

- Docs for receiving `initialReviewCompleted` when Initial Review finishes after ID submit (including automated high-confidence pass)
- Docs for the Due Diligence stage on the same webhook (`caseStatus` / `nextCaseStatus` / identity status = `due_diligence`)
- Client lifecycle overview and recommended consumption steps

## Fixed

- Loan workflow and endpoints webhook sections now describe Initial Review and Due Diligence for integrators

## Removed

- None

## How to test (QA)

1. Open the published or local API docs → Onboarding (IDV) webhooks.
2. Confirm automated Initial Review (`image_check_auto_ir`) and Due Diligence sections with example JSON.
3. Confirm README and client loan workflow link to the updated guide.
4. Optionally: subscribe a test URL to `initialReviewCompleted`, complete Initial Review (manual or automated), and move a case to Due Diligence — payloads match the doc examples.
