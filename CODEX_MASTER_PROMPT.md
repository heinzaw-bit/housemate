# CODEX_MASTER_PROMPT.md

Copy this prompt into Codex after placing all specification files in the repository.

---

You are building a production-quality web application named **HouseMate**.

Before writing code, read all specification files in `docs/` in this exact order:

1. README.md
2. PRODUCT.md
3. PRD.md
4. UX_FLOWS.md
5. DATA_MODEL.md
6. MONEY_ENGINE.md
7. AI_ASSISTANT.md
8. NOTIFICATIONS_MESSAGING.md
9. ARCHITECTURE.md
10. SECURITY_PRIVACY.md
11. API_CONTRACTS.md
12. TESTING.md
13. IMPLEMENTATION_PLAN.md

Treat these files as the product source of truth.

## Product

HouseMate is a web-first, mobile-first shared-household operating system for roommates.

It combines:

- household expenses,
- utility bills,
- flexible splitting,
- balances,
- settlements,
- meals,
- cooking,
- shopping,
- chores,
- events,
- announcements,
- polls,
- notifications,
- and an AI household assistant.

The app must be suitable for real shared-house use.

## Critical architecture rules

1. AI must NEVER be the source of financial truth.
2. All money calculations must be deterministic and tested.
3. Use integer minor units or a safe decimal strategy. Never depend on raw JS floating point for final money allocations.
4. Every household-scoped server action must verify active membership.
5. Use database RLS where appropriate.
6. AI mutation requests must become drafts and require user confirmation.
7. Do not let AI directly write financial records.
8. Financial records must be auditable.
9. Core application must work even if AI is unavailable.
10. External messaging integrations must go through adapters rather than being mixed into domain logic.
11. Do not hardcode the provided sample household members into product code.
12. Avoid oversized components and monolithic service files.
13. Add tests with every implementation phase.

## Technology

Preferred:
- Next.js
- React
- TypeScript
- Tailwind CSS
- PostgreSQL/Supabase
- Zod
- Vitest or equivalent
- Playwright
- PWA
- Web Push

Use current stable packages compatible with the project.

Do not add unnecessary dependencies.

## Coding style

- strict TypeScript
- no `any` unless justified
- domain logic separate from UI
- reusable typed schemas
- server-side validation
- accessible components
- mobile-first responsive design
- clear error handling
- no silent failures
- no fake success UI for failed writes

## Required money engine

Implement this before building complex financial UI:

- equal split
- exact split
- percentage split
- ratio split
- selected participants
- hybrid split
- balance ledger
- settlement suggestions
- deterministic rounding

The total allocated shares must always equal the original expense total exactly.

## Mandatory acceptance scenario

The application must be able to model this real bill:

Total bill:
฿2,106.57

Individual/special amounts:
- Ethan ฿5
- Henry ฿60
- Donut ฿125
- Johnny ฿180
- Rio ฿20

Additional adjustment:
฿204

Special/adjustment total:
฿594

Remaining shared amount:
฿1,512.57

Shared across:
9 people

The app must:
- represent the rule clearly,
- calculate it deterministically,
- show preview before save,
- explain each member's final share,
- preserve exact total after rounding.

If the meaning of the ฿204 adjustment is ambiguous, the product should require the user to classify it rather than guess.

## AI requirements

Initial AI intents should include:

Read-only:
- current balance
- member balance
- unpaid bill members
- today's meals
- cooking assignment
- due chores
- shopping list
- upcoming events
- house summary

Draft actions:
- expense
- bill
- settlement
- meal check
- event
- shopping items
- chore
- announcement
- poll
- reminder

AI draft workflow:

user message
→ structured parse
→ schema validation
→ entity resolution
→ authorization
→ draft preview
→ user confirmation
→ deterministic application action
→ audit log

## Notification requirements

Implement:
- in-app notification center
- preference center
- web push
- deep links
- quiet hours
- idempotent scheduling

Do not request push permission immediately on first visit.

## Workflow

Follow `IMPLEMENTATION_PLAN.md`.

For each phase:

1. inspect current repository,
2. explain a short implementation plan,
3. implement only that phase,
4. add/update tests,
5. run lint,
6. run typecheck,
7. run relevant tests,
8. run build if appropriate,
9. report files changed,
10. report validation output,
11. list remaining known limitations.

Do not continue into later phases unless explicitly requested.

## First task

Start with **Phase 0 — Repository foundation** only.

If the repository is already initialized, audit it against Phase 0 and make only the changes required to satisfy the specification.

Do not begin Phase 1 automatically.

---

End prompt.
