# Release notes — envelope quietMode, invitationLinks, public signer API docs

## Added

- **Public Signer API** doc: load document for signing, submit signature, envelope auto-advance, and signed-link redirect/completion behaviour.
- **Quiet mode** documentation across send/instantiate flows: suppress Cleared signer emails; use `invitationLinks` from API responses for client-owned delivery.
- **Envelope invitation behaviour**: one email per signer (document list + single start link); signer auto-advance through envelope documents.
- **Envelope context endpoint** (`GET .../documents/:documentId/envelope-context?signerId=`) documented with current response shape.
- **Resend/remind** under quiet mode: `409 QUIET_MODE` and invitation link in resend response.
- **Changelog v2.1** on Document Signatures overview.

## Fixed

- Envelope-context and send/instantiate response examples aligned with current gateway behaviour.
- Client loan application workflow updated for envelope invitations and quiet mode.

## Removed

- (n/a)

## How to test (QA)

- Open **document-signatures/public-signer-api.md** and confirm envelope auto-advance and `redirectTo` / `envelopeComplete` match signer app behaviour in QA.
- Send a document with `quietMode: false` and confirm docs describe `invitationLinks` in the send response.
- Send an envelope and confirm docs state one invitation per signer (not one per document).
- Call resend on a quiet-mode document and confirm docs match `409` + `invitationLink`.
- Review **client-loan-application-workflow.md** envelope section for LOS integration accuracy.
