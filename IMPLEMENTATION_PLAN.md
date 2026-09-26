# IMPLEMENTATION_PLAN.md

## Development rule

Do not ask Codex to build the entire product in one pass.

Build vertical slices.

After every phase:
- run lint
- run typecheck
- run tests
- update docs if architecture changed
- commit

## Phase 0 — Repository foundation

Deliver:
- Next.js + TypeScript
- Tailwind
- environment validation
- lint
- formatting
- test framework
- Playwright
- Supabase config
- CI
- docs folder

Acceptance:
clean install and build.

## Phase 1 — Auth & household

Deliver:
- sign-in
- user profile
- create household
- invite
- join
- member list
- roles
- RLS
- household switch architecture even if MVP uses one household mostly

Acceptance:
two test users join same household.
foreign household access denied.

## Phase 2 — Money engine

Deliver pure deterministic module first.

- equal
- exact
- percent
- ratio
- selected
- hybrid
- ledger
- balances
- settlement suggestions

No UI dependency.

Acceptance:
all money tests passing.

## Phase 3 — Money UI

Deliver:
- expense list
- add expense
- preview
- bill
- hybrid bill
- balances
- settlement
- history
- edit/void

Acceptance:
real electricity scenario can be entered and explained.

## Phase 4 — Home dashboard

Deliver:
- balance summary
- due bill card
- activity summary
- empty states
- mobile navigation

## Phase 5 — Meals & events

Deliver:
- meal checks
- responses
- cooking assignments
- BBQ/event
- attendance

Connect event attendees to expense participant defaults.

## Phase 6 — Shopping

Deliver:
- shared list
- quick add
- assignment
- purchased status
- convert purchase to expense

## Phase 7 — Chores

Deliver:
- chores
- recurrence
- instances
- complete
- swap request
- reminders model

## Phase 8 — Announcements & polls

Deliver:
- announcements
- priority
- acknowledgement
- polls
- realtime responses

## Phase 9 — Notification center + Web Push

Deliver:
- notification data model
- in-app center
- preferences
- web push
- deep links
- scheduled reminders
- quiet hours

Acceptance:
users do not need to manually revisit app to notice key actions.

## Phase 10 — AI assistant foundation

Deliver:
- chat UI
- intent schemas
- read-only queries
- action drafts
- confirmation UI
- audit records

Initial supported commands:
- balances
- unpaid bills
- shopping add
- event draft
- announcement draft
- poll draft
- expense draft

## Phase 11 — AI money parsing

Deliver:
- natural-language expense/bill parser
- member resolution
- ambiguity handling
- deterministic validation
- hybrid bill support

Acceptance:
electricity example produces valid draft.

## Phase 12 — LINE integration

Only after core workflows are stable.

Deliver:
- secure LINE account linking
- webhook verification
- inbound commands
- outbound notification adapter
- attendance quick actions/deep links
- financial actions remain confirmation-gated

## Phase 13 — Polish

- PWA install experience
- accessibility
- localization architecture
- performance
- mobile responsiveness
- error recovery
- audit/history UX
- observability

## Backlog after MVP

- recurring utilities
- inventory
- receipt OCR
- payment proof workflow
- monthly reports
- household digest
- email
- SMS fallback
- multiple households
- richer LINE interactions
- native wrapper only if web/PWA limitations justify it
