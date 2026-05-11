---
applyTo:
  - "**/*.sql"
  - "**/Migrations/**"
  - "**/migrations/**"
---

# Schema Migration Documentation Sync

When reviewing changes that include schema migrations:

**AP-06 🟡 Schema change without doc sync.** Lion's documentation governance is weak and schema drift is hard to trace back. Flag PRs that:

- Contain `ALTER TABLE` / `ADD COLUMN` / `DROP COLUMN` in `*.sql` migration files
- Contain EF migration files in `Migrations/*.cs`
- But have no corresponding update to the schema documentation in the same PR

Exception: pure index adjustments (no column structure change), or the PR description explicitly references a documentation PR.

Phrasing example: "🟡 Should-fix — This PR includes column-level schema changes but doesn't appear to update the schema docs. Lion schema drift is hard to trace later. Consider updating the schema doc in this PR, or referencing the doc PR link in the description."

Additionally, for any new column / dropped column, suggest the reviewer cross-check whether downstream consumers (ExAPI, search ES index, OSIP) need coordinated changes — but do not look those up yourself.
