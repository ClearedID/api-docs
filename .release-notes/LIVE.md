# Initial Review Triage

## Added
- Initial Review Triage queue and detail in Control Centre (claim, start, advance, reject, assign)
- Due Diligence as a first-class identity status, with client webhook and ops notifications
- Structured Initial Review levels and reason codes
- Resubmission expiry job that can auto-reject when triage is enabled
- Mobile-friendly evidence review (swipe, pinch zoom, compare, including archived media)

## Fixed
- Org kill-switch now respected when a submission enters the triage queue
- Due Diligence cases can only leave via allowed clear/reject paths
- Reject stamp retries when identity reject succeeds but triage stamp races

## Removed
- None

## How to test (QA)
1. Enable `INITIAL_REVIEW_TRIAGE_ENABLED` in a non-prod environment and grant `cc_initial_review_triage_*` privileges.
2. Submit an identity verification; confirm it appears in Control Centre → Initial Review.
3. On a phone: claim, start, review evidence (swipe/pinch/compare), Advance with a level and next status (including Due Diligence and resubmission).
4. Confirm client `initialReviewCompleted` webhook and ops Teams notification include elapsed minutes.
5. Reject a case from triage; confirm existing rejection webhook (not IR completed).
6. As supervisor: Assign from queue/detail; Release from detail on mobile.
7. Confirm flag-off returns feature disabled and legacy mark-complete still works.

## Technical notes
- Repos: swf-shared-library, swf-core-gateway, swf-identity-api.ms, swf-control-centre-api.ms, swf-control-centre-app, swf-machine-runner.ms, api-docs
- Feature flag default remains off
- Host cron required for resub expiry: see machine-runner CRON_JOBS / control-centre-api docs/initial-review-triage-rollout.md
