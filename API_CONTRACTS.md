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
