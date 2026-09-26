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
