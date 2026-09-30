# HouseMate project boundaries

These rules apply across this workspace and to the API, web, and future mobile projects. Keep them when the projects move to separate repositories. Follow additional instructions in a project's own `AGENTS.md` where applicable.

## Document ownership

- `SPEC.md` describes product features, user-visible behavior, release scope, and open product decisions. It may change as the product changes.
- `README.md` and each project's README explain the project to users and developers: what exists, setup, commands, and links.
- This file holds durable engineering boundaries and working rules. `CLAUDE.md` imports this file; do not duplicate rules there.
- `api/openapi.yaml` is the concrete API contract for implemented endpoints. Older root domain documents are supporting references; reconcile conflicts with the current SPEC and contract when changing behavior.

## Project ownership

- Keep `api`, `app`, and `mobile` independently buildable and releasable. Do not create runtime imports across project directories or rely on sibling file paths after separation.
- The API owns authentication verification, authorization, household and financial business rules, persistence, migrations, jobs, notifications, and provider integrations.
- Web and mobile own presentation, platform session handling, accessibility, and local UI state. They call the versioned API for authoritative household reads and writes; they do not write household records directly to the database.
- Publish API changes in `api/openapi.yaml` before clients rely on them. Coordinate incompatible changes with both clients and preserve compatibility during migration.

## Trust and data boundaries

- Keep database credentials, service-role keys, AI keys, and private provider credentials out of clients, `NEXT_PUBLIC_` variables, examples with real values, and version control.
- Verify the caller and enforce active household membership, role, and entity ownership on every protected API operation. A client-provided household ID is a selector, not authorization.
- Apply the same household boundary to realtime data, attachments, jobs, and notifications. Do not display sample or cached data as live household truth.
- Treat AI output as an untrusted draft. Validate it through normal API services and require explicit user confirmation before any mutation.

## Financial integrity

- The API is the authority for money calculations. Represent amounts as integer minor units with an explicit currency; use deterministic allocation and ensure shares total the original amount exactly.
- Return an API-calculated preview before a financial confirmation or material edit. Make confirmations idempotent and preserve a traceable history for edits, voids, and settlements.
- Keep recorded manual settlements distinct from actual payment processing.

## Working on the project

- Update the SPEC for product behavior changes, the OpenAPI contract for API changes, and the relevant README for setup or usage changes. Change this file only when a durable project boundary changes.
- Run the affected project's lint, typecheck, tests, and build as appropriate. Do not claim a feature works end to end when its API, web, or mobile piece is still pending.
