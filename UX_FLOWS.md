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
