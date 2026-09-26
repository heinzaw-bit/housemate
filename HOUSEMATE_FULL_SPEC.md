# HouseMate — Combined Build Specification


---

# FILE: README.md

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


---

# FILE: PRODUCT.md

# PRODUCT.md

## Product name

Working name: **HouseMate**

The name is temporary and can be changed later.

## Product statement

HouseMate is an AI-assisted shared-living platform that helps roommates coordinate household money, meals, groceries, chores, events, announcements, and everyday decisions from one place.

## Problem

People living together usually coordinate through a mixture of:

- group chat,
- calculator,
- notes,
- spreadsheets,
- bank-transfer screenshots,
- memory,
- verbal agreements,
- repeated reminders.

This becomes difficult when:

- one person pays for everyone,
- only some people participated,
- different people owe different amounts,
- a bill contains shared and individual components,
- previous credits must be deducted,
- a shared meal has changing attendance,
- the buyer and the participants are different people,
- several expenses should be netted instead of settled individually,
- people forget chores, payments, or food coordination.

## Real-world example

Electricity bill:

- Total bill: ฿2,106.57
- Individual/special amounts:
  - Ethan: ฿5
  - Henry: ฿60
  - Donut: ฿125
  - Johnny: ฿180
  - Rio: ฿20
- Additional adjustment: ฿204
- Special/adjustment total: ฿594
- Shared remainder: ฿1,512.57
- Shared across 9 people: ≈ ฿168.06 each

The system must support this kind of calculation without requiring manual calculator work.

## Product goals

### G1 — Financial clarity
Every member can understand:
- what they owe,
- why they owe it,
- who paid,
- whether they already paid,
- which expenses contribute to their balance.

### G2 — Household coordination
The system should help coordinate:
- meals,
- cooking,
- shopping,
- chores,
- events,
- announcements,
- quick decisions.

### G3 — Low-friction interaction
Users should not need to continuously reopen the website.

HouseMate should reach them through:
- web push,
- in-app activity,
- messaging integrations.

### G4 — Trust
No hidden financial decisions.

Every financial calculation must be deterministic, inspectable, and reproducible.

### G5 — AI usefulness
AI should reduce typing and coordination overhead, not replace core business logic.

## Non-goals for MVP

- bank account linking,
- automatic money transfer,
- crypto,
- loans,
- credit scoring,
- landlord accounting software,
- smart-home device control,
- CCTV/computer vision,
- automatic grocery purchasing,
- fully autonomous AI actions without confirmation.

## Target users

### Primary
University students or young adults living together in one shared house/apartment.

### Secondary
- friends sharing accommodation,
- co-living residents,
- small shared apartments,
- temporary group housing.

## Product pillars

### 1. Money
- expenses
- bills
- flexible splits
- balances
- settlements
- payment status
- recurring expenses
- attachments/receipts

### 2. Meals
- who is eating
- who cooks
- meal events
- BBQ/hotpot attendance
- meal-related spending

### 3. Shopping
- shared list
- assigned buyer
- status
- conversion to expense
- optional inventory state

### 4. Chores
- recurring tasks
- assignments
- swaps
- completion
- reminders

### 5. Communication
- announcements
- polls
- reminders
- household activity feed
- important updates

### 6. AI assistant
- natural-language household commands
- expense parsing
- bill parsing
- announcement drafting
- poll creation
- summaries
- explanations

## Product principles

1. One source of truth.
2. AI proposes; business logic decides.
3. Financial actions are auditable.
4. Every action should explain itself.
5. Minimize repeated data entry.
6. Do not overload users with notifications.
7. Mobile-first responsive UX even though the app is web-first.
8. Support shared-house workflows, not just accounting.
9. Users must be able to correct mistakes.
10. Keep MVP focused on one household before expanding to multiple advanced organizations.


---

# FILE: PRD.md

# PRD.md

## 1. Overview

HouseMate is a responsive web application and PWA for managing a shared household.

The first deployment should support a real household of friends living together.

## 2. Success criteria

The MVP is successful when household members can:

- join one household,
- see other members,
- record shared spending,
- create utility bills,
- use multiple split strategies,
- understand their current net balance,
- record settlements,
- coordinate meals,
- maintain a shopping list,
- manage basic chores,
- create announcements and polls,
- receive web notifications,
- use AI to prepare supported actions,
- confirm AI-generated actions before they affect household records.

## 3. Roles

### Owner
- creates household,
- manages household settings,
- invites/removes members,
- grants admin rights,
- archives household.

### Admin
- manages members and household configuration,
- creates/edit household-level recurring rules,
- moderates announcements where required.

### Member
- participates in expenses,
- creates normal household entries,
- votes/responds,
- records own payments,
- completes chores.

Do not create excessive role complexity in MVP.

## 4. Authentication

MVP:
- Google sign-in
- email magic link or email/password as fallback

Every user has:
- user profile,
- display name,
- optional avatar,
- preferred timezone,
- notification preferences.

## 5. Household onboarding

Flow:

1. Welcome
2. Create household or join with invite link/code
3. Household name
4. Default currency
5. Add/invite members
6. Choose optional starter modules:
   - Money
   - Meals
   - Shopping
   - Chores
7. Notification permission prompt only after the user understands the value
8. Dashboard

## 6. Dashboard

Mobile-first dashboard.

Show:
- current balance,
- amounts owed to user,
- amounts user owes,
- unpaid/pending bills,
- tonight meal status,
- current shopping count,
- chores due,
- upcoming event,
- latest important announcement,
- AI input.

Avoid showing too many cards.

## 7. Money requirements

### 7.1 Expense
Fields:
- title
- category
- total amount
- currency
- payer
- participants
- split method
- line items or adjustments if needed
- date
- notes
- receipt attachment optional
- status

### 7.2 Split methods
Required:
- equal
- exact amount
- percentage
- ratio/shares
- selected participants
- hybrid split

### 7.3 Hybrid split
Must support:
- bill total,
- explicit individual charges,
- explicit shared charges/adjustments,
- remaining amount shared among selected members.

### 7.4 Bill
Bill is an expense with additional semantics:
- provider/type,
- billing period,
- due date,
- recurring possibility,
- payment status.

### 7.5 Settlement
Record money transferred between household members.

Fields:
- from member
- to member
- amount
- date
- method optional
- note
- proof optional

Settlement reduces balances but does not delete expenses.

### 7.6 Balance
For each member calculate:
- total paid on behalf of household,
- total allocated share,
- settlements sent,
- settlements received,
- net balance.

Positive net:
household owes the member.

Negative net:
member owes others.

### 7.7 Simplified settlement suggestions
Later MVP:
calculate a minimized set of payments that can settle balances.

Do not alter original accounting records.

## 8. Meals

### 8.1 Meal check
Create:
- meal type,
- date/time,
- question,
- deadline,
- participants.

Responses:
- joining
- not joining
- maybe

### 8.2 Cooking responsibility
Assign:
- rice
- main dish
- other custom role

### 8.3 Meal event
BBQ/hotpot/etc:
- title
- date/time
- attendees
- shopping list
- linked expenses

When an expense is created from an event, attendees can automatically become split participants.

## 9. Shopping

List item:
- title
- quantity optional
- unit optional
- added by
- priority
- assigned buyer optional
- status
- estimated price optional
- purchased price optional
- linked expense optional

Statuses:
- needed
- assigned
- purchased
- cancelled

Allow “mark purchased and add expense”.

## 10. Chores

Chore:
- title
- instructions
- assignee
- due date/time
- recurrence
- completion status
- reminder
- optional rotation rule

MVP recurrence:
- daily
- weekly
- selected weekdays
- monthly

Support:
- complete
- skip with reason
- request swap

## 11. Announcements

Announcement:
- title
- body
- author
- priority
- publish time
- optional acknowledgement requirement
- optional expiry

Priorities:
- normal
- important
- urgent

## 12. Polls

Poll:
- question
- choices
- single or multiple choice
- deadline
- eligible voters
- anonymous optional later

Use cases:
- BBQ vs hotpot
- cleaning time
- who is eating
- event timing
- household purchase decision

## 13. Notifications

Required channels:
- in-app activity feed
- web push

Future:
- LINE
- email
- SMS fallback

Notification categories:
- money
- meals
- chores
- shopping
- announcements
- polls
- events
- AI summaries

User can independently disable categories and non-critical reminders.

## 14. AI assistant

AI input must support examples such as:

- “Electricity is 2106.57. Ethan 5, Henry 60, Donut 125, Johnny 180, me 20 special usage and another adjustment 204. Split the remainder across 9.”
- “Tell everyone BBQ tomorrow at 7 and ask who is joining.”
- “Add eggs and drinking water to shopping.”
- “Who hasn’t paid electricity?”
- “Who is cooking tonight?”
- “Draft an announcement that landlord is coming tomorrow around 2.”
- “Summarize what is happening in the house tonight.”

AI produces structured preview.

User confirms before:
- creating financial records,
- sending announcements,
- creating polls,
- scheduling reminders,
- modifying chores.

Read-only questions may be answered immediately.

## 15. Search

Search across:
- expenses,
- bills,
- settlements,
- announcements,
- events,
- shopping,
- chores.

## 16. Activity log

Track important household actions:
- expense created/edited/deleted,
- settlement recorded,
- bill status changed,
- announcement published,
- member joined/removed,
- AI-created action confirmed.

## 17. Accessibility

- keyboard navigation
- visible focus
- semantic labels
- screen-reader friendly
- sufficient contrast
- no color-only meaning
- responsive text
- reduced motion preference

## 18. Internationalization

Design strings for localization.

Initial:
- English

Later:
- Burmese
- Thai

Currency should be household-configurable.

## 19. Performance

Targets:
- usable on mid-range mobile devices,
- dashboard should load quickly on normal mobile internet,
- optimistic UI only where safe,
- financial writes require server confirmation.

## 20. Offline behavior

PWA shell may load offline.

Do not allow unsafe financial writes to appear confirmed while offline.

Optional:
queue non-financial drafts locally.

## 21. Auditability

Every money item must preserve:
- creator,
- timestamps,
- original amount,
- allocation details,
- edits,
- settlement links.

Soft-delete financial records or preserve audit history rather than silently removing history.


---

# FILE: UX_FLOWS.md

# UX_FLOWS.md

## Design direction

Web-first, mobile-first, app-like.

Primary navigation on mobile:
- Home
- Money
- House
- Assistant
- Profile

“House” opens:
- Meals
- Shopping
- Chores
- Events
- Announcements
- Polls

Desktop:
left sidebar plus content area.

## 1. First-run flow

Welcome
→ Sign in
→ Create / Join household
→ Household basics
→ Invite members
→ Select modules
→ Dashboard

Do not ask for notification permission on the first screen.

Ask after demonstrating a reason:
“Get reminded when bills are due or dinner polls need your answer.”

## 2. Add expense flow

Money
→ Add
→ Expense

Step 1:
- title
- amount
- payer

Step 2:
- participants
- split method

Step 3:
- breakdown preview

Step 4:
- confirm

Preview must show each person’s final share before save.

## 3. Hybrid bill flow

Money
→ Add bill
→ amount
→ payer
→ billing period
→ individual charges
→ shared adjustments
→ shared participants
→ preview

Example:

Total: 2106.57
Individual:
Ethan 5
Henry 60
Donut 125
Johnny 180
Rio 20
Other adjustment 204

Special total: 594
Remaining: 1512.57
Shared among 9
Base share: 168.063333...

Final displayed amounts use currency rounding rules.
Internal allocation must preserve exact balancing.

## 4. Settlement flow

Money
→ Balances
→ member
→ Settle

Show:
- recommended amount
- custom amount
- payment method optional
- note
- proof optional

Confirmation:
“You paid Henry ฿420.”

## 5. Meal check flow

House
→ Meals
→ New meal check

Choose:
- breakfast/lunch/dinner/custom
- date/time
- response deadline

Publish.

Member response:
- Join
- Not joining
- Maybe

Organizer sees live count.

## 6. BBQ flow

House
→ Events
→ Create
→ BBQ

Set date/time
→ invite household members
→ collect attendance
→ create shopping list
→ purchased items become expenses
→ event attendees prefill split participants.

## 7. Shopping flow

House
→ Shopping
→ Add item

Quick add:
“Eggs”

Detailed:
- quantity
- priority
- assign to
- note

Purchased:
→ enter actual price
→ optionally create expense
→ choose payer
→ choose participants
→ confirm.

## 8. Chore flow

House
→ Chores
→ New chore

Set:
- title
- assignee
- recurrence
- reminder

Assignee can:
- complete
- skip
- request swap

## 9. Announcement flow

House
→ Announcements
→ New

Title
Body
Priority
Recipients
Acknowledgement toggle

Preview
→ Publish

## 10. Poll flow

House
→ Polls
→ New

Question
Choices
Deadline
Recipients
Allow multiple choices

Publish
→ responses update in real time.

## 11. AI command flow

Assistant

User:
“Tell everyone BBQ tomorrow at 7 and ask who’s joining.”

AI returns:

Intent: Create event + attendance poll

Preview:
BBQ
Tomorrow
7:00 PM
Recipients: Household

Actions:
- Edit
- Confirm
- Cancel

Only after Confirm:
create records and send notifications.

## 12. AI read-only flow

User:
“Who hasn’t paid electricity?”

AI:
- finds matching bill,
- returns unpaid/pending members,
- links to bill.

No confirmation required because no mutation occurs.

## 13. Notification deep links

Every notification should take user to relevant page.

Examples:
Bill reminder
→ `/money/bills/:id`

Meal poll
→ `/house/meals/:id`

Chore
→ `/house/chores/:id`

Announcement
→ `/house/announcements/:id`

## 14. Error UX

Financial save failure:
- never show fake success,
- preserve form values,
- show retry.

AI parse uncertainty:
- ask for clarification or show unresolved fields.

Example:
“I found two people named Alex. Which Alex did you mean?”

## 15. Empty states

Money:
“No expenses yet. Add your first shared expense.”

Shopping:
“Nothing needed right now.”

Meals:
“No meal check for tonight.”

Chores:
“No chores due today.”

Avoid fake sample records after onboarding.


---

# FILE: DATA_MODEL.md

# DATA_MODEL.md

Use PostgreSQL.

All IDs should be UUIDs.

Every household-scoped table must contain `household_id`.

Use `created_at`, `updated_at` consistently.

## users

- id
- auth_user_id
- display_name
- avatar_url nullable
- timezone
- locale
- created_at
- updated_at

## households

- id
- name
- default_currency
- timezone
- owner_user_id
- created_at
- updated_at
- archived_at nullable

## household_members

- id
- household_id
- user_id
- role: owner | admin | member
- status: invited | active | left | removed
- joined_at nullable
- left_at nullable
- created_at
- updated_at

Unique:
(household_id, user_id)

## household_invites

- id
- household_id
- invited_email nullable
- invite_code
- role
- expires_at
- created_by
- claimed_by nullable
- claimed_at nullable

## expenses

- id
- household_id
- title
- description nullable
- category
- total_amount_minor
- currency
- paid_by_member_id
- expense_date
- source_type: manual | bill | event | shopping | ai
- source_id nullable
- status: active | voided
- created_by_member_id
- created_at
- updated_at

Use integer minor units for normal currency storage where practical.

For calculations requiring intermediate fractions, use PostgreSQL numeric/decimal in calculation tables or deterministic allocation logic.

## expense_participants

- id
- expense_id
- member_id
- included boolean
- final_share_minor
- metadata_json

Unique:
(expense_id, member_id)

## expense_split_configs

- id
- expense_id
- method: equal | exact | percent | ratio | hybrid
- config_json
- calculation_version
- created_at

`config_json` preserves the original rule used to calculate shares.

## expense_adjustments

Used by hybrid splitting.

- id
- expense_id
- member_id nullable
- label
- amount_minor
- adjustment_type: individual_charge | shared_charge | credit | deduction
- created_at

## bills

- id
- expense_id
- household_id
- bill_type
- provider nullable
- billing_period_start nullable
- billing_period_end nullable
- due_date nullable
- bill_status: draft | due | partially_settled | settled | overdue
- recurring_rule_id nullable

## settlements

- id
- household_id
- from_member_id
- to_member_id
- amount_minor
- currency
- settlement_date
- method nullable
- note nullable
- proof_url nullable
- status: active | voided
- created_by_member_id
- created_at
- updated_at

## meal_events

- id
- household_id
- title
- meal_type
- starts_at
- response_deadline nullable
- created_by_member_id
- status
- created_at
- updated_at

## meal_responses

- id
- meal_event_id
- member_id
- response: join | not_join | maybe
- responded_at

Unique:
(meal_event_id, member_id)

## cooking_assignments

- id
- household_id
- meal_event_id nullable
- assignment_date
- role_name
- assigned_member_id
- status: assigned | completed | swapped | skipped
- notes nullable
- created_at
- updated_at

## shopping_lists

- id
- household_id
- title
- status
- created_at
- updated_at

MVP can default to one active household list.

## shopping_items

- id
- shopping_list_id
- title
- quantity nullable
- unit nullable
- priority: low | normal | high
- assigned_member_id nullable
- added_by_member_id
- status: needed | assigned | purchased | cancelled
- estimated_amount_minor nullable
- purchased_amount_minor nullable
- linked_expense_id nullable
- created_at
- updated_at

## chores

- id
- household_id
- title
- instructions nullable
- recurrence_rule_json nullable
- default_assignee_member_id nullable
- reminder_rule_json nullable
- active boolean
- created_by_member_id
- created_at
- updated_at

## chore_instances

- id
- chore_id
- due_at
- assigned_member_id
- status: pending | completed | skipped | swap_requested
- completed_at nullable
- completion_note nullable
- created_at
- updated_at

## chore_swap_requests

- id
- chore_instance_id
- requested_by_member_id
- target_member_id nullable
- status: open | accepted | rejected | cancelled
- created_at
- resolved_at nullable

## events

- id
- household_id
- title
- event_type
- starts_at
- ends_at nullable
- description nullable
- created_by_member_id
- created_at
- updated_at

## event_attendees

- id
- event_id
- member_id
- response: join | not_join | maybe | no_response
- responded_at nullable

## announcements

- id
- household_id
- title
- body
- priority: normal | important | urgent
- requires_acknowledgement boolean
- expires_at nullable
- published_by_member_id
- published_at
- created_at
- updated_at

## announcement_acknowledgements

- id
- announcement_id
- member_id
- acknowledged_at

## polls

- id
- household_id
- question
- allow_multiple boolean
- deadline nullable
- created_by_member_id
- created_at
- updated_at

## poll_options

- id
- poll_id
- label
- position

## poll_votes

- id
- poll_id
- poll_option_id
- member_id
- created_at

Enforce uniqueness appropriately for single-choice polls.

## notifications

- id
- household_id
- recipient_member_id
- category
- title
- body
- priority
- deep_link nullable
- source_type
- source_id nullable
- status: pending | sent | failed | read
- created_at
- sent_at nullable
- read_at nullable

## notification_preferences

- id
- member_id
- category
- in_app_enabled
- web_push_enabled
- line_enabled
- email_enabled
- sms_enabled
- quiet_hours_json nullable

## push_subscriptions

- id
- user_id
- endpoint
- p256dh
- auth
- user_agent nullable
- last_used_at
- created_at

## integrations

- id
- household_id
- provider: line | email | sms
- encrypted_config_json
- status
- created_at
- updated_at

## ai_conversations

- id
- household_id
- member_id
- created_at
- updated_at

## ai_messages

- id
- conversation_id
- role
- content
- structured_payload_json nullable
- created_at

## ai_action_drafts

- id
- household_id
- requested_by_member_id
- action_type
- parsed_payload_json
- validation_status
- confirmation_status: pending | confirmed | cancelled | expired
- execution_result_json nullable
- created_at
- confirmed_at nullable
- executed_at nullable

## audit_log

- id
- household_id
- actor_member_id nullable
- action_type
- entity_type
- entity_id
- before_json nullable
- after_json nullable
- source: user | ai_confirmed | system
- created_at

## Important constraints

1. Never derive household access from client-supplied household IDs alone.
2. Always verify membership server-side.
3. Financial rows should not be hard-deleted during normal usage.
4. Preserve split configuration used at calculation time.
5. Recalculating an edited expense must produce a new audit entry.
6. Settlement records must remain independent of expense records.


---

# FILE: MONEY_ENGINE.md

# MONEY_ENGINE.md

## Purpose

The money engine is the most trust-sensitive part of HouseMate.

It must be deterministic, testable, auditable, and independent from the LLM.

## Currency representation

Store currency amounts as integer minor units where possible.

Example:
฿2106.57 → 210657 satang.

Never use JavaScript floating-point arithmetic for final money allocation.

Use integer arithmetic or a decimal library.

## Core accounting equation

For a member:

`net = amount_paid_for_household - allocated_share - settlements_sent?`

Recommended clearer ledger approach:

For each expense:
- payer receives credit equal to total paid,
- each participant receives debit equal to allocated share.

For each settlement:
- sender receives credit for amount sent,
- receiver receives debit for amount received.

Then sum ledger entries per member.

Choose one sign convention and document it in code.

## Equal split

Example:
Total = 1000
Participants = 3

Ideal:
333.333...

Actual minor-unit allocation must total exactly 1000.

Use deterministic remainder distribution.

Example:
334
333
333

Remainder recipient selection must be deterministic:
- stable sorted participant IDs, or
- configured ordering.

Do not randomly assign rounding differences.

## Exact split

User enters each member share.

Validation:
sum(exact shares) == total.

If not:
block confirmation and show difference.

## Percentage split

Validation:
percent sum == 100%.

Use decimal-safe calculation.

Allocate rounding remainder deterministically.

## Ratio split

Example:
A: 1
B: 2
C: 1

Total ratio = 4.

Allocate total proportionally.

## Selected participants

Selected participants are simply the participant set used by the split method.

Non-selected household members receive zero share.

## Hybrid split

Required because this is a key real-world use case.

Definitions:

- `bill_total`
- `individual_charges`
- `shared_adjustments`
- `credits/deductions`
- `shared_participants`

One supported model:

1. Start with bill total.
2. Reserve explicit individual charges.
3. Reserve shared adjustments if they must be deducted before base sharing.
4. Apply credits/deductions according to explicit rule.
5. Compute remaining shared amount.
6. Split shared amount across selected shared participants.
7. Add each person's explicit individual charge.
8. Apply person-specific credits/deductions.
9. Validate final allocated total equals bill total.

## Electricity acceptance scenario

Input:

Total:
2106.57

Individual:
Ethan 5
Henry 60
Donut 125
Johnny 180
Rio 20

Other adjustment:
204

Special/adjustment total:
390 + 204 = 594

Shared remainder:
2106.57 - 594 = 1512.57

Shared participants:
9

Base share:
1512.57 / 9 = 168.063333...

Displayed approximate:
168.06

However, final stored member shares must still sum exactly to 2106.57 after allocation and rounding.

If the 204 is not associated with a specific person, the UI must ask what it represents:
- shared amount already handled separately,
- individual charge,
- deduction,
- credit,
- excluded/non-household amount.

The system must not guess accounting semantics silently.

## Expense edit rules

Editing:
- title only → no recalculation.
- amount → requires recalculation.
- participants → requires recalculation.
- split config → requires recalculation.
- payer → changes ledger but not shares.

Always show new preview before saving meaningful financial changes.

## Voiding

Do not hard-delete a finalized expense.

Void it:
- preserve original,
- exclude from active balances,
- write audit event.

## Settlement suggestion algorithm

Input:
all member net balances.

Goal:
reduce number of transfers while preserving totals.

Algorithm:
- separate creditors and debtors,
- greedily match largest debtor and creditor,
- create suggested transfers,
- continue until all near zero.

Suggestions are not actual settlements until users record them.

## Required money tests

- equal split exact division
- equal split remainder
- exact split
- invalid exact split
- percentage split
- percentage remainder
- ratio split
- selected participants
- hybrid split
- negative/credit cases
- payer included
- payer excluded
- settlement effect
- voided expense
- edited expense
- 1 participant
- large amount
- zero amount rejection
- currency precision
- duplicated confirmation/idempotency


---

# FILE: AI_ASSISTANT.md

# AI_ASSISTANT.md

## Purpose

The AI assistant reduces friction in household coordination.

It must not become the source of financial truth.

## AI responsibilities

Allowed:
- parse natural language,
- identify intent,
- map member names,
- extract money amounts,
- extract dates/times,
- prepare expense drafts,
- prepare event drafts,
- prepare announcements,
- prepare polls,
- prepare reminders,
- summarize household activity,
- answer read-only household questions,
- explain deterministic calculations.

Not allowed without confirmation:
- finalize expense,
- edit financial records,
- publish announcement,
- send household-wide message,
- record settlement,
- modify chore ownership,
- create recurring reminders,
- remove member,
- change household settings.

## Intent types

Recommended initial intents:

### Read-only
- GET_MY_BALANCE
- GET_MEMBER_BALANCE
- GET_UNPAID_BILL_MEMBERS
- GET_TODAY_MEALS
- GET_TODAY_COOKING
- GET_DUE_CHORES
- GET_SHOPPING_LIST
- GET_UPCOMING_EVENTS
- SUMMARIZE_HOUSE

### Draft actions
- DRAFT_EXPENSE
- DRAFT_BILL
- DRAFT_SETTLEMENT
- DRAFT_MEAL_CHECK
- DRAFT_EVENT
- DRAFT_SHOPPING_ITEMS
- DRAFT_CHORE
- DRAFT_ANNOUNCEMENT
- DRAFT_POLL
- DRAFT_REMINDER

## Structured output

The model should return JSON-like structured output validated by a schema.

Example:

```json
{
  "intent": "DRAFT_EVENT",
  "confidence": 0.94,
  "requires_confirmation": true,
  "payload": {
    "title": "BBQ",
    "starts_at": "2026-09-27T19:00:00+07:00",
    "attendance_poll": true,
    "participants": "ALL_ACTIVE_MEMBERS"
  },
  "unresolved_fields": []
}
```

## Confirmation model

AI mutation flow:

User message
→ intent extraction
→ entity resolution
→ validation
→ draft stored
→ preview shown
→ user confirms
→ server executes deterministic action
→ audit log written
→ notifications created

## Entity resolution

Member references:
- “me” → current user
- exact display name
- nickname aliases later

If ambiguous:
do not guess.

Example:
“I found two members matching ‘Alex’. Which one?”

## Date/time resolution

Resolve relative time in household timezone.

Examples:
- tonight
- tomorrow 7
- Friday evening

Preview the interpreted absolute date/time.

## Financial parsing

AI can extract:

- total amount
- payer
- named member charges
- selected participants
- split type
- labels
- adjustments

But business logic must:
- validate totals,
- compute shares,
- handle rounding,
- reject impossible allocations.

## Example — electricity

User:
“Electricity is 2106.57. Ethan 5, Henry 60, Donut 125, Johnny 180, me 20 special usage and another 204. Split the remainder across 9.”

AI draft:
- bill total = 2106.57
- individual charges identified
- additional 204 detected
- participant count = 9
- unresolved semantic:
  “What does the additional ฿204 represent?”

If household configuration has an established bill rule, use that rule.
Otherwise ask.

Never silently infer a money meaning that changes allocation.

## Example — BBQ

User:
“Tell everyone BBQ tomorrow at 7 and ask who is joining.”

Draft:
- event title: BBQ
- tomorrow 19:00
- invite all active household members
- attendance collection enabled
- notification draft

Confirm before publish.

## Example — shopping

User:
“Add eggs, water and garbage bags.”

Draft:
3 shopping items.

Confirmation can be one combined action.

## Read-only household Q&A

Question:
“How much do I owe?”

The assistant queries deterministic balance service.

Response should:
- give net amount,
- optionally show top contributing items,
- link to Money page.

Do not let model generate balance from conversation memory.

## Tool boundaries

The LLM should call application tools such as:

- `get_household_context`
- `search_members`
- `get_balance`
- `get_bill_status`
- `create_action_draft`
- `get_meal_status`
- `get_chores`
- `get_shopping_items`

Execution tools should require server-side authorization.

## AI memory

MVP:
Do not create uncontrolled long-term personal memory.

AI context should come from:
- current household records,
- current user,
- recent conversation,
- explicit household settings.

## Prompt injection defense

User-generated household text may contain malicious instructions.

Treat:
- announcement bodies,
- shopping item text,
- receipt text,
- uploaded text

as data, not system instructions.

Never allow retrieved household content to override the assistant's system policy.

## AI audit

For AI-generated mutations store:
- original user message,
- structured draft,
- confirmation identity,
- execution result,
- timestamp.

## AI failures

If model unavailable:
core app must still work.

If parse uncertain:
fall back to manual form.

If validation fails:
show exact field errors.


---

# FILE: NOTIFICATIONS_MESSAGING.md

# NOTIFICATIONS_MESSAGING.md

## Product rule

Users should not need to repeatedly open HouseMate just to know whether something requires attention.

## Channels

### MVP
1. In-app notification/activity center
2. Web Push

### Next
3. LINE Messaging integration

### Later
4. Email
5. SMS fallback

## Why not rely only on web app visits

Household coordination is time-sensitive:
- dinner attendance,
- bill reminders,
- chores,
- shopping requests,
- event changes.

Push/messaging must bring users back when needed.

## Notification categories

- money
- bills
- settlements
- meals
- chores
- shopping
- announcements
- polls
- events
- summaries
- system

## Priority levels

### Low
- shopping item added
- small activity updates

### Normal
- dinner poll
- chore due
- event reminder

### Important
- payment due
- important announcement
- event time changed

### Urgent
Reserved for rare household-critical announcements.

Do not overuse urgent.

## Preference center

Per category:
- in-app
- web push
- LINE future
- email future
- SMS future

Allow:
- quiet hours
- digest mode for low-priority activity

## Web Push

Requirements:
- service worker
- push subscription storage
- permission UX
- VAPID or platform-supported push mechanism
- deep link
- subscription refresh/removal handling

Permission strategy:
Do not request browser permission immediately on first visit.

First explain:
“Get notified when bills are due, meals need your response, or chores are assigned.”

Then ask.

## Notification payload

- id
- category
- priority
- title
- short body
- deep link
- entity type
- entity id
- created time

Do not put sensitive financial details in notification body unless product settings explicitly allow it.

Default:
“You have a new electricity payment request.”
rather than displaying full balance on lock screen.

## Deep links

Examples:

- `/money/bills/:id`
- `/money/balances`
- `/house/meals/:id`
- `/house/chores/:id`
- `/house/events/:id`
- `/house/polls/:id`

## Digesting

Low-priority updates may be bundled.

Example:

House update:
- 6 eating tonight
- eggs and water added
- 2 chores due
- BBQ poll closes tonight

## LINE integration

Goal:
allow users to interact with HouseMate without opening the web app for simple actions.

Capabilities to design for:
- household bot
- send household announcements
- quick responses
- attendance polls
- payment reminders
- deep links back to web app
- natural-language commands

## LINE security

Never trust messaging-platform user identity automatically.

Create an explicit account linking flow:
Web account
→ “Connect LINE”
→ secure one-time token
→ messaging identity linked to member.

## LINE group design

Do not assume every bot message in a group can safely mutate household state.

For financial actions:
- parse
- create draft
- send confirmation link or require authorized confirmation.

## SMS

Use only later if required.

Potential use:
- critical fallback
- verification
- user who cannot use push/messaging integration

Avoid using SMS for normal noisy household coordination.

## Notification idempotency

Every scheduled notification must have a deterministic or stored idempotency key.

Avoid duplicate notifications after retries.

## Scheduling

Store schedule in database.

Worker/cron processes:
- due bills
- meal response deadlines
- chores
- event reminders
- summary delivery

## Quiet hours

Default example:
23:00–07:00

Urgent notifications may bypass only if explicitly allowed by user preference.

## Reminder policy

Do not nag.

Examples:
- bill: due soon + due day + overdue
- meal poll: initial + one deadline reminder
- chore: due reminder + one overdue reminder

Avoid repeated hourly reminders.

## Delivery failure

Record:
- sent
- failed
- invalid subscription
- retry count

Invalid push subscriptions should be disabled/removed safely.


---

# FILE: ARCHITECTURE.md

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


---

# FILE: SECURITY_PRIVACY.md

# SECURITY_PRIVACY.md

## Threat model

HouseMate stores:
- household membership,
- expenses,
- payment status,
- household routines,
- messages/announcements,
- optional receipts.

Treat this as private household data.

## Core security requirements

1. Authentication required for household data.
2. Server-side authorization on every household-scoped action.
3. Row-level security where supported.
4. Secrets only on server.
5. Webhook signatures verified.
6. Integration tokens encrypted at rest.
7. Sensitive data minimized.
8. Financial operations audited.
9. AI actions validated.
10. Secure account-linking for messaging integrations.

## Household isolation

A user in Household A must never access Household B through:
- guessed IDs,
- altered URLs,
- API requests,
- realtime subscriptions,
- search,
- AI queries.

Add explicit automated tests.

## Supabase RLS

Use RLS for household-scoped data.

Policy concept:
current authenticated user must have an active membership in row.household_id.

Service role bypass should be restricted to trusted server functions.

## Financial authorization

MVP:
normal members may create expenses.

Editing:
- creator, admin, or allowed participant according to policy.

Voiding:
- creator/admin.

Settlement:
sender should confirm their own outgoing settlement, or household policy must define alternatives.

## Attachments

Receipts:
- private bucket
- signed URLs
- file size limit
- file type allowlist
- malware scanning if product expands

## AI privacy

Do not send unnecessary household history to the LLM.

Use minimum context needed for the request.

For example:
“Who owes electricity?”
requires bill and allocation data,
not complete shopping/chore history.

## AI prompt injection

Retrieved user content is untrusted.

Never allow:
- announcement text,
- shopping text,
- notes,
- receipts

to redefine AI rules or tool permissions.

## AI confirmation

Mutation tools should require:
- authenticated user,
- active household member,
- valid draft,
- authorized action,
- unexpired confirmation,
- idempotency key.

## Messaging integration

Verify provider webhook signature.

Map external sender identity only through verified account link.

Do not allow unknown group participants to execute financial commands.

## Push privacy

Lock-screen notifications can expose data.

Default notification copy should be privacy-conscious.

Example:
“Electricity bill needs your attention.”

Detailed amount appears after app opens.

## Audit log

Must include:
- actor
- action
- entity
- timestamp
- before/after for important edits
- whether source was direct user or AI-confirmed

## Account removal

Define:
- leave household
- remove household member
- delete personal account
- archive household

Financial history involving departed members should remain referentially valid.

Use historical display names if necessary.

## Rate limits

Apply to:
- AI requests
- invite creation
- announcement publishing
- push test requests
- messaging webhooks
- auth endpoints

## CSRF/XSS

Use framework protections.

Never render household-provided HTML unsanitized.

Prefer plain text / sanitized rich text.

## Security tests

- horizontal household access
- role access
- RLS
- invite abuse
- expired invite
- forged LINE webhook
- duplicated webhook
- AI draft reuse
- financial action without confirmation
- attachment authorization
- push subscription ownership


---

# FILE: API_CONTRACTS.md

# API_CONTRACTS.md

The exact route style can be REST, server actions, or RPC, but contracts should remain explicit.

## Household

### Create household
Input:
- name
- currency
- timezone

Output:
- household
- owner membership

### Invite member
Input:
- household_id
- email optional
- role

Output:
- invite link/code

### Join household
Input:
- invite token

Output:
- membership

## Expenses

### Create expense draft/manual
Input:
- title
- amount
- currency
- payer
- participants
- split_config
- date

Server:
- validates membership
- calculates shares
- returns preview

### Confirm expense
Input:
- preview/draft id
- idempotency key

Output:
- expense
- allocations
- balance changes

## Bills

Create:
- base expense data
- bill metadata

Query:
- current status
- participant shares
- settlement/payment state

## Settlements

Create:
- from
- to
- amount
- date

Return:
- settlement
- updated balances

## Meals

Create meal event.

Respond:
- join
- not_join
- maybe

Return updated counts.

## Shopping

Create item.

Assign buyer.

Mark purchased.

Optional:
create linked expense draft.

## Chores

Create chore.

Generate/return instances.

Complete instance.

Request swap.

Accept/reject swap.

## Announcements

Create draft.

Publish.

Acknowledge.

## Polls

Create poll.

Vote.

Get results.

## Notifications

List notifications.

Mark read.

Register push subscription.

Delete push subscription.

Update preferences.

## AI

### POST /assistant/messages

Input:
```json
{
  "householdId": "...",
  "message": "Tell everyone BBQ tomorrow at 7"
}
```

Output can be:

Read-only answer:
```json
{
  "type": "answer",
  "message": "...",
  "links": []
}
```

or action draft:
```json
{
  "type": "action_draft",
  "draftId": "...",
  "intent": "DRAFT_EVENT",
  "preview": {},
  "unresolvedFields": []
}
```

### POST /assistant/actions/:draftId/confirm

Server:
- authenticates
- verifies household
- verifies authorization
- checks unexpired draft
- uses idempotency
- executes app service
- creates audit event

### POST /assistant/actions/:draftId/cancel

Marks cancelled.

## Error envelope

Recommended:

```json
{
  "error": {
    "code": "INVALID_SPLIT_TOTAL",
    "message": "Exact shares must equal the expense total.",
    "details": {}
  }
}
```

Use stable error codes.

## Pagination

For history:
cursor-based pagination.

## Versioning

Internal API can evolve with app.

External integration/webhook contracts should be versioned explicitly.


---

# FILE: TESTING.md

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


---

# FILE: IMPLEMENTATION_PLAN.md

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


---

# FILE: CODEX_MASTER_PROMPT.md

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


---

# FILE: FIGMA_MAKE_PROMPT.md

# FIGMA_MAKE_PROMPT.md

Copy the prompt below into Figma Make AI.

---

Design a complete mobile-first responsive web application concept for **HouseMate**, an AI-assisted shared-household operating system for roommates.

This is NOT just a bill-splitting app.

HouseMate helps university students and roommates manage:

- shared expenses,
- utility bills,
- balances,
- settlements,
- meals,
- cooking,
- groceries,
- chores,
- BBQ/hotpot events,
- announcements,
- polls,
- reminders,
- and an AI household assistant.

## Product personality

The interface should feel:

- trustworthy,
- friendly,
- modern,
- youthful,
- calm,
- practical,
- collaborative,
- not childish,
- not corporate banking,
- not like a generic admin dashboard.

Avoid excessive gradients, glassmorphism, neon effects, and over-decorated cards.

Use a clean system with soft surfaces, clear typography, rounded but not cartoonish components, strong spacing, and excellent mobile usability.

## Platform

Design as:

- responsive web app,
- mobile-first,
- installable PWA,
- desktop layout as a responsive extension,
- optimized for frequent use from phone browsers.

## Mobile navigation

Bottom navigation:

1. Home
2. Money
3. House
4. Assistant
5. Profile

“House” contains:
- Meals
- Shopping
- Chores
- Events
- Announcements
- Polls

Desktop should use a left sidebar.

## Important design principle

Users should be able to understand what needs attention in less than 5 seconds.

Do not make the home page a wall of cards.

Prioritize the most actionable information.

## Screen 1 — Home dashboard

Design a mobile home dashboard for a household named:

**Our House**

Show:

### Header
- household switcher/name
- notification bell
- small avatar

### Personal money summary
Example:
Your balance: **+฿730**
Subtitle:
“You should receive ฿730”

Also support negative state:
“You owe ฿486.06”

Button:
“View breakdown”

### Tonight
Example:
Dinner tonight
6 joining
Rice: Ethan
Cooking: Henry

Actions:
“Respond”
“View”

### Tasks needing attention
Examples:
- Electricity due tomorrow
- BBQ poll closes tonight
- Trash chore due today

### Shopping
“5 items needed”
Top items:
Eggs, Water, Garbage Bags

### Upcoming
“BBQ — Saturday, 7:00 PM”

### Assistant entry
Compact AI input:
“Ask HouseMate…”

Example suggestions:
- How much do I owe?
- Who is eating tonight?
- Add eggs to shopping

## Screen 2 — Money overview

Show:

- net balance,
- “you owe” / “you are owed” breakdown,
- recent expenses,
- current bills,
- settlements,
- “Add expense” primary button.

Tabs:
- Overview
- Expenses
- Bills
- Settlements

Use clear positive/negative financial treatment without relying only on color.

## Screen 3 — Add expense

Mobile multi-step form.

Fields:
- title
- total amount
- payer
- participants
- split method

Split options:
- Equal
- Exact
- Percentage
- Ratio
- Hybrid

Show live preview before confirm.

## Screen 4 — Hybrid electricity bill

Design a detailed bill builder using this scenario:

Total:
฿2,106.57

Individual charges:
- Ethan ฿5
- Henry ฿60
- Donut ฿125
- Johnny ฿180
- Rio ฿20

Other adjustment:
฿204

Shared remainder:
฿1,512.57

Shared members:
9

Approx base:
฿168.06/person

The UI must make the calculation understandable.

Show a final member-by-member preview.

Include:
“Why this amount?” interaction.

## Screen 5 — Balances

List household members with:

- owes you
- you owe
- settled

Example:
Henry owes you ฿420
Ethan owes you ฿210
Johnny owes you ฿100

Add:
“Settle up”

## Screen 6 — Meal check

Title:
Dinner Tonight

Question:
“Are you eating rice tonight?”

Responses:
- Joining
- Not joining
- Maybe

Show participant avatars and totals.

Show:
Rice: Ethan
Main dish: Henry

## Screen 7 — BBQ event

Title:
BBQ Night

Saturday
7:00 PM

Attendance:
7 joining
2 not joining

Sections:
- Attendees
- Shopping list
- Expenses
- Notes

Primary action:
“Add purchase”

Make event expenses automatically feel connected to attendees.

## Screen 8 — Shopping

Shared shopping list.

Items:
- Eggs
- Drinking water
- Cooking oil
- Pork
- Garbage bags

Each row may show:
- quantity
- priority
- assigned buyer
- status

Actions:
- Add item
- Assign
- Mark purchased

When purchased, offer:
“Add as shared expense”

## Screen 9 — Chores

Show:

Today:
- Trash — Rio
- Kitchen — Ethan
- Water refill — Donut

States:
- due
- completed
- swap requested

Actions:
- Complete
- Request swap

Also create a weekly chore schedule concept.

## Screen 10 — Announcements

House announcements feed.

Examples:

Important:
“Landlord visiting tomorrow around 2 PM.”

Normal:
“Internet maintenance Friday.”

Allow:
- acknowledgement indicator
- priority
- publish time

## Screen 11 — Polls

Example:

“Saturday dinner?”

Options:
- BBQ
- Hotpot
- Order food

Show live results and deadline.

## Screen 12 — AI Assistant

This should feel like an action assistant, not a generic ChatGPT clone.

Conversation example:

User:
“Tell everyone BBQ tomorrow at 7 and ask who’s joining.”

HouseMate:
“I can create a BBQ event for tomorrow at 7:00 PM and invite all 9 active household members.”

Show a structured preview card:

Event
BBQ
Tomorrow • 7:00 PM
Invite: All members
Attendance: Enabled

Buttons:
- Edit
- Confirm
- Cancel

Another example:

User:
“How much do I owe?”

Assistant:
“You currently owe ฿486.06.”

Breakdown:
Electricity ฿228.06
Groceries ฿138
BBQ ฿120

Add deep-link actions.

## Screen 13 — Notifications

Notification center grouped by:

- Today
- Earlier

Examples:
- Electricity due tomorrow
- Ethan responded to dinner
- Your trash chore is due
- BBQ poll closes tonight

Add filter chips.

## Screen 14 — Notification preferences

Categories:
- Money
- Meals
- Chores
- Shopping
- Announcements
- Events
- Polls

Channels:
- In-app
- Web Push
- LINE (show as “coming soon” if desired)

Quiet hours:
11 PM – 7 AM

## Screen 15 — Household settings

Show:
- household name
- currency
- timezone
- members
- roles
- invite
- notification defaults
- integrations

## Responsive desktop concept

Create desktop versions of:
- Home
- Money
- AI Assistant

Use:
- left navigation sidebar,
- central main content,
- optional right contextual panel.

Do not simply stretch mobile cards across a large screen.

## Visual system

Create:
- typography scale
- spacing scale
- buttons
- inputs
- cards
- badges
- tabs
- segmented controls
- avatar groups
- amount formatting
- empty states
- loading states
- error states
- confirmation dialogs
- toast/snackbar
- bottom sheets for mobile

## Accessibility

Use:
- strong contrast,
- visible focus states,
- 44px+ touch targets,
- semantic hierarchy,
- no color-only status indication,
- legible financial amounts.

## Content style

Friendly and concise.

Examples:
“You owe ฿486.06”
“6 people are eating tonight”
“3 items still need to be bought”
“Electricity is due tomorrow”
“HouseMate needs your confirmation”

Avoid robotic text.

## Output

Generate:

1. Design system / style direction
2. Mobile screen set
3. Desktop screen examples
4. Main user flow connections
5. Important interaction states
6. Consistent reusable components

The result should feel polished enough to use as the visual direction for a real implementation.

---

End prompt.
