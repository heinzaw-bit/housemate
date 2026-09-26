# HouseMate — Codex Build Specification

HouseMate is a web-first shared-household operating system for roommates.

The initial real-world use case is a group of university students living together who repeatedly need to coordinate:

- shared expenses,
- utilities,
- different per-person charges,
- groceries,
- cooking,
- meals,
- BBQ/hotpot events,
- shopping,
- chores,
- announcements,
- polls,
- reminders,
- and settlements.

The system should remove the need to combine Messenger/LINE chats, calculators, notes, spreadsheets, and repeated manual reminders.

## Product principle

**HouseMate should not require users to keep reopening the website just to know what is happening.**

The web application is the source of truth. Notifications and messaging channels deliver important actions to users.

Primary access:

- Responsive web app
- Installable PWA
- Web Push
- In-app activity feed
- LINE integration later in the MVP+ roadmap
- SMS only as an optional future fallback

## Core product areas

1. Household & members
2. Money
3. Bills
4. Balances & settlements
5. Meals & cooking
6. Shopping
7. Chores
8. Events
9. Announcements & polls
10. Notifications
11. AI assistant
12. Household summaries

## Source-of-truth rule

The AI never owns financial truth.

The AI:
- understands natural language,
- extracts structured intent,
- prepares drafts,
- asks for confirmation,
- explains calculations,
- creates summaries.

The deterministic business logic:
- performs calculations,
- enforces permissions,
- writes financial records,
- computes balances,
- applies settlement rules.

## Recommended implementation stack

### Web
- Next.js
- React
- TypeScript
- Tailwind CSS
- PWA support

### Backend / data
- PostgreSQL
- Supabase for database, auth, storage, realtime, and server-side functions where useful

### AI
- OpenAI API with structured outputs/tool calls
- AI actions must be validated by server-side application logic

### Messaging / notifications
- Web Push first
- Notification preference center
- LINE Messaging API after core household workflows are stable
- Email/SMS only if needed later

## Documents

Read these in this order before implementing:

1. `PRODUCT.md`
2. `PRD.md`
3. `UX_FLOWS.md`
4. `DATA_MODEL.md`
5. `MONEY_ENGINE.md`
6. `AI_ASSISTANT.md`
7. `NOTIFICATIONS_MESSAGING.md`
8. `ARCHITECTURE.md`
9. `SECURITY_PRIVACY.md`
10. `API_CONTRACTS.md`
11. `TESTING.md`
12. `IMPLEMENTATION_PLAN.md`
13. `CODEX_MASTER_PROMPT.md`
14. `FIGMA_MAKE_PROMPT.md`

## MVP boundary

The first usable version must solve real shared-house problems without requiring AI.

AI is added on top of reliable household workflows.

The minimum reliable product is:

Household → Members → Expenses → Bills → Flexible Split → Balances → Settlements → Meals → Shopping → Chores → Announcements/Polls → Notifications → AI Assistant.

Do not begin with banking integrations, smart-home devices, cameras, autonomous purchases, or complicated analytics.
# house_mate
