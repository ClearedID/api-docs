# Schedule send (sendBy) and mute behaviour

## Added

- Docs for `sendBy` on signature and verification APIs.

## Fixed

- Docs now state **mute/quiet cannot be combined with sendBy** (400), and that muted invites are finalized without Cleared emailing.

## Removed

- (none)

## How to test (QA)

1. Review updated pages under document-signatures and initiate-verification.
2. Confirm mute + sendBy incompatibility is clear for integrators.
