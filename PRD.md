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
