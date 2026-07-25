# Identity verification workflow guide

## Added
- New public guide explaining the identity verification lifecycle stage by stage, and which webhooks clients should handle at each step
- Plain-language note on what **Initial Review** means (preliminary clearance / viable path vs likely rejection — not a final clear or reject)
- Links from the docs README and endpoints index

## Fixed
- Onboarding webhooks config: subscribe via the **Verification portal** (not Screening Portal)

## Removed
- (none)

## How to test (QA)
1. Open `identity/identity-verification-workflow.md` and confirm the Initial Review meaning note and stage list match product behaviour.
2. From README → Identity, open the new workflow page.
3. From endpoints webhook section, follow the new workflow link.
4. Confirm onboarding webhooks still say Verification portal for bindings.

## Technical notes
- Event names remain catalog camelCase (`initialReviewCompleted`, `dueDiligenceStarted`, `identityVerificationCleared`, `identityVerificationRejected`).
