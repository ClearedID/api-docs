# Permissions catalog documentation

## Added

- **Permissions catalog** (`permissions.md`): published reference of Admin Portal / Control Centre / DevOps privilege ids with name, category, and description, grouped for browsing.
- **Spreadsheet export** (`permissions-catalog.csv`): same catalog as CSV (Name, Category, Description, Id, SubCategory, Registry) for sharing and offline review.
- Linked from the main API docs README under API Fundamentals.

## How to test (QA)

1. Open `permissions.md` in the api-docs repo (or published docs site after sync).
2. Spot-check a few known permissions (e.g. `full_access`, `view_organisation_team`, a `cc_*` id) against `swf-core-gateway/data/permissionRegistry.js` (and the Control Centre / DevOps registries).
3. Open `permissions-catalog.csv` in Excel or Google Sheets and confirm columns and row counts look complete.
