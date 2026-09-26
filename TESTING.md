# TESTING.md

## Goal

HouseMate should be safe to use with real household money.

Testing must focus especially on:
- allocation correctness,
- household isolation,
- permissions,
- idempotency,
- notification behavior,
- AI confirmation boundaries.

## Unit tests

### Money engine
Mandatory comprehensive coverage.

See `MONEY_ENGINE.md`.

### Utility
- currency formatting
- timezone resolution
- recurrence generation
- notification quiet hours
- date parsing helpers

## Integration tests

### Auth/household
- create household
- invite
- join
- removed member loses access
- member of A cannot access B

### Expense
- create
- preview
- confirm
- edit
- void
- balance updates

### Settlement
- record
- balance changes
- void

### Event → expense
- event attendees prefilled
- non-attendees excluded

### Shopping → expense
- purchased item creates correct draft

### AI
- draft created
- mutation not executed before confirmation
- confirmation executes exactly once
- cancelled draft cannot execute
- expired draft cannot execute
- ambiguous member requires clarification

## End-to-end tests

Use Playwright.

Critical flows:

1. Sign in → create household → invite/join.
2. Add equal expense.
3. Add hybrid electricity bill.
4. See balance.
5. Record settlement.
6. Create dinner poll and respond.
7. Create BBQ event.
8. Shopping item → purchased → expense.
9. Create chore → complete.
10. Publish announcement.
11. Create poll → vote.
12. Enable web push where test environment permits.
13. AI expense draft → preview → confirm.
14. AI announcement draft → cancel.

## Exact electricity test

Use 9 members.

Total:
210657 minor units.

Individual:
500
6000
12500
18000
2000

Additional:
20400

Check:
special total = 59400
remaining = 151257

Base shared allocation across 9 must sum exactly to 151257.

Final participant allocations plus any designated adjustment allocation must sum exactly to 210657.

## Security tests

- direct API access to foreign household
- modified household ID
- foreign entity ID
- foreign realtime channel
- unauthorized admin operation
- guessed invite token
- reused invite
- forged integration webhook
- AI tool call with foreign household data
- user tries to confirm another user's restricted draft

## Notification tests

- permission/subscription registration
- duplicate push prevented
- quiet hours
- category disabled
- urgent handling
- deep link
- invalid subscription cleanup
- retry behavior

## Accessibility tests

Automated:
- axe or equivalent

Manual:
- keyboard-only navigation
- screen reader basic flows
- 200% zoom
- mobile viewport
- reduced motion

## Performance tests

Monitor:
- dashboard query count
- N+1 queries
- household with 20 members
- 10k historical expenses
- 1k notifications

MVP does not need extreme scale, but avoid obviously unbounded queries.

## CI

Required on pull request:
- lint
- typecheck
- unit tests
- integration tests
- build

E2E:
run on main or appropriate PR pipeline.

Do not merge code that breaks money tests.
