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
