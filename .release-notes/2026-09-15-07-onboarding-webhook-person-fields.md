# Onboarding webhook person fields and related docs

## Added
- Documented identity `taxNumber`, `idNumber`, and `dateOfBirth` on VerificationStatus webhook rows and on `identityVerificationCleared` / `initialReviewCompleted` `eventContext`
- Documented `identityVerificationStarted` / submitted in the onboarding event catalog and how Started pins to the flow request for `urlParameters`
- Cleared-stage examples in the identity verification workflow guide now show the person fields

## Fixed
- Endpoints overview webhook section no longer lists outdated dotted event names; it points to the live camelCase onboarding catalog and signing docs

## Removed
- Nothing

## How to test (QA)
1. Open `onboarding/onboarding-webhooks.md` — confirm identity row table and cleared / IR deep dives include tax / ID / DOB rules.
2. Open `identity/identity-verification-workflow.md` Stage 2 and Stage 4 examples — person fields present.
3. Open `client-loan-application-workflow.md` webhook body bullet — mentions the three fields and Started.
4. Open `endpoints.md` Webhooks — camelCase events, link to onboarding guide.
