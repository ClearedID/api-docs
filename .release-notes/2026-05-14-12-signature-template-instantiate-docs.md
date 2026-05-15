# Release notes — signature template instantiate (docs)

## Added

- **Workflow guide:** Envelope template instantiate documents **`configuration`** / **`useDigitalSignature`** at the root and per **`documentAssignment`**.
- **Document templates API doc:** Top-level **`useDigitalSignature`** and how it merges with **`configuration`**.
- **Envelope templates API doc:** Note pointing to the real **`documentAssignments`** contract and document-level config overrides.

## Fixed

- (n/a)

## Removed

- (n/a)

## How to test (QA)

- Open **`client-loan-application-workflow.md`**, **`document-signatures/document-templates.md`**, and **`document-signatures/envelope-templates.md`** and confirm the new bullets match the **swf-core-gateway** behaviour for template and envelope-template instantiate.
