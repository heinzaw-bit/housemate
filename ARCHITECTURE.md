# ARCHITECTURE.md

## High-level architecture

```text
Browser / PWA
    |
    v
Next.js Web Application
    |
    +---- Auth
    |
    +---- Server/API layer
             |
             +---- Household service
             +---- Money service
             +---- Meal service
             +---- Shopping service
             +---- Chore service
             +---- Event service
             +---- Notification service
             +---- AI orchestration service
             |
             v
         PostgreSQL / Supabase
             |
             +---- Realtime updates
             +---- Storage
             +---- Scheduled jobs / server functions
```

External:
- OpenAI API
- Web Push
- LINE API later

## Recommended repository

```text
housemate/
  app/
  components/
  features/
    auth/
    household/
    money/
    meals/
    shopping/
    chores/
    events/
    announcements/
    polls/
    assistant/
    notifications/
  lib/
    db/
    auth/
    money/
    ai/
    notifications/
    permissions/
    validation/
  server/
    services/
    repositories/
    jobs/
  supabase/
    migrations/
    seed.sql
  tests/
    unit/
    integration/
    e2e/
  docs/
```

If using Next.js App Router, keep domain logic out of React components.

## Layering rule

UI
→ application service
→ domain/business logic
→ repository/database

AI must call application services, not directly write database rows.

## Money domain

Create a pure calculation module.

Example:

```text
lib/money/
  money.ts
  split-equal.ts
  split-exact.ts
  split-percent.ts
  split-ratio.ts
  split-hybrid.ts
  ledger.ts
  settlement-suggestions.ts
```

Functions should be testable without database or UI.

## Validation

Use shared schemas, e.g. Zod.

Validate:
- route input,
- AI structured output,
- form data,
- integration webhook payloads.

## Auth

Every server operation derives current user from authenticated session.

Then verify:
- household membership,
- required role,
- entity belongs to household.

Never rely on hidden UI controls for authorization.

## Realtime

Use realtime for:
- poll vote counts,
- meal responses,
- shopping updates,
- notifications,
- household dashboard refresh.

Do not require realtime for correctness.

## Background jobs

Needed for:
- scheduled notifications,
- overdue bill checks,
- chore instance generation,
- recurring bill generation later,
- digest summaries,
- expired AI draft cleanup.

## AI orchestration

```text
User message
→ policy + intent prompt
→ structured output
→ schema validation
→ entity resolution
→ application validation
→ action draft
→ preview
→ confirmation
→ execution
```

Read-only:
```text
User message
→ intent
→ authorized query tool
→ deterministic data
→ response generation
```

## Integration adapter pattern

```text
NotificationAdapter
  - WebPushAdapter
  - LineAdapter
  - EmailAdapter
  - SmsAdapter
```

The notification service should not contain provider-specific logic.

## Environment configuration

Examples:

```text
NEXT_PUBLIC_APP_URL=
SUPABASE_URL=
SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=
OPENAI_API_KEY=
WEB_PUSH_PUBLIC_KEY=
WEB_PUSH_PRIVATE_KEY=
LINE_CHANNEL_SECRET=
LINE_CHANNEL_ACCESS_TOKEN=
```

Never expose server secrets to client bundles.

## Observability

Track:
- API errors
- failed notifications
- failed jobs
- AI parse failures
- calculation validation failures
- webhook failures

Do not log raw secrets or private message contents unnecessarily.

## Idempotency

Required for:
- expense creation from AI confirmation
- settlement confirmation
- LINE webhook handling
- notification jobs
- recurring jobs

## Migrations

Database changes must use migration files.

Do not manually modify production schema outside migrations.

## Deployment

Possible initial deployment:
- Vercel for Next.js
- Supabase for backend/data

Validate platform limits before relying on long-running background work.

Use a dedicated scheduler/worker strategy if required.
