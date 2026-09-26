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
