# Permissions catalog wording

## Fixed
- ID validation queue “Validate” permission description no longer mentions CBS; it describes manual validation and retry only

## Added
- None

## Removed
- “CBS/” from the Validate ID queue items permission description in the permissions catalog

## How to test (QA)
1. Open the permissions catalog CSV (or regenerated permissions docs from it).
2. Find `cc_id_validation_queue_validate` — description should say “Run manual validation…” with no “CBS”.
