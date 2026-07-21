# Document expiration docs: optional, validated when provided

## Changed

- Document Templates and Envelope Templates API docs now state clearly that `configuration.expiration` is optional everywhere, and describe the new 400 responses returned when a malformed expiration object is provided (wrong `type`, missing/invalid `fixedDate`, non-positive `relativeDays`). (Behaviour implemented in swf-cloudsign-api and swf-core-gateway.)
- Documented when `expiresAt` is calculated: at queueing for `sendImmediately`, at document send for drafts, and now also at envelope send for envelope documents.
- The envelope template instantiate example now uses the real `configuration` field for overrides instead of the unimplemented `customConfiguration`.

## How to test (QA)

- Documentation-only change: review the two updated pages for accuracy against the API behaviour above.
