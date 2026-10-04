# AI_ASSISTANT.md

## Purpose

The AI assistant reduces friction in household coordination.

It must not become the source of financial truth.

## Current implementation — local update 2026-10-04

The current local web Assistant is question-only. Action commands, preparation and editor/publication flows are removed from this tab; use Board or Money for creation. Questions stay in tab memory only, without automatic saving, Resume or Discard draft controls. Older saved questions/action seeds are ignored and cannot reopen an editor. The backend draft contract and existing editor components remain for compatibility; the earlier deployed release still has action drafting until this branch is deployed.

Read-only conversation includes local greetings/app explanations, visible announcement reads, combined Board updates, Catch me up (important announcements/events/due bills), and Gemini general-information replies. Weather lookup has been removed. The three shortcuts are Catch me up, My balance and Board updates. Answers use concise HouseMate language, record actions and checked timestamps; general replies use plain paragraphs/bullets and a subtle AI label with an Answered timestamp. Ambiguous questions offer read-only choices. Short household follow-ups retain only the previous allowlisted topic in tab memory, re-read authorized current records and never send previous prompts/answers to Gemini. Clearing answers, reloading or changing account/household clears context. Saved drafts remain disabled in Assistant.

Household Q&A remains supported. The web centers on one large chat box under the house mascot, with three small shortcuts for Catch me up, My balance and Board updates. The latest answer appears below the input; earlier answers stay in collapsed history (six total in memory). Existing record/source links remain accessible, except the View balance footer link. Calculate a split is removed from answer footers. Ask/Enter submits directly, without a sharing notice or privacy accordion. Shift + Enter adds a newline. An accessible animated mascot/dots loading indicator appears inside the input area, temporarily replacing the visible question while preserving its text. Cancel stays in the composer; only answers appear below it. Reduced-motion settings show a static indicator. Answers separate the main API sentence from supporting context; clarification keeps input editable. General questions are independent; supported household follow-ups use only the previous topic in this tab. The API owns classification validation and answers; the web prioritizes three quick questions and optional free text; the API still supports all nine household topic selectors, including announcements and Board. Gemini (`gemini-3.1-flash-lite`) classifies household questions and writes bounded general-information replies using a strict allowlisted schema. It receives no household records or prior conversation. Deterministic API services supply all financial values. Household answers are timestamped snapshots with household-scoped source links and bounded-list notices where needed. General answers have a separate AI label and Answered timestamp, without claiming verified household facts. Named-member balances remain in Money to keep personal information out of unpaid Gemini requests.

Quick topics require no provider key. General model questions require server-only `GEMINI_API_KEY`, `GEMINI_FREE_TIER_CONFIRMED=true` after checking the Google project is Free Tier with billing disabled, and migration 016 for durable private counters. The confirmation flag does not verify Google billing. One bounded attempt uses a 12-second timeout and 800 output-token cap, with no retries or paid fallback, 20 attempts per account per UTC hour and 200 globally per UTC day. Local greetings, app explanations and common household reads/follow-ups need no provider quota. HouseMate saves no prompts/history. Google's unpaid tier may use/review input. The web retains the existing privacyAcknowledged request field for API compatibility but has no acknowledgement UI. The API blocks recognized active-member names/emails, digits, links and credential markers before spending. This limited guard cannot detect all private details; typed questions must be general. See [Google's terms](https://ai.google.dev/gemini-api/terms) and the [API setup guide](api/README.md#optional-natural-language-questions).

**Earlier first action release (retained API, hidden in current web):** local meal-check and announcement drafts now accompany Q&A. `POST /v1/households/{id}/assistant/drafts` interprets common English commands without sending private action text or household data to Gemini. The web shows unresolved details and an editable draft, then an exact household/timezone/audience review. Explicit confirmation calls the existing idempotent meal or announcement API, preserving authorization, creator/time records and normal notifications. Interrupted publication retains the same locked payload/key for a manual retry. Editable drafts now have seven-day account/household-scoped device-local recovery (deployed on 2026-10-04), while answers and Gemini consent remain in memory. Sign-out/account changes clear saved drafts; household changes isolate them and timezone/currency changes invalidate their context. Update/account-switch guards remain; no separate AI execution audit is implemented. Financial creation drafts now use the same authoritative preview and confirmation services as Money.

**Earlier additional action drafts (retained API, hidden in current web):** shopping, chores, expenses and hybrid bills are deployed to production as of 2026-10-02, after the additive creation migration and API-before-web release. Signed-in/device acceptance remains pending. Commands stay in HouseMate; ambiguous assignees, payers, participants, dates and amounts require explicit editing. Expense shares and hybrid-bill charges/credits use normal API previews, with Edit/Cancel before explicit confirmation. Shopping/chore creation adds scoped durable keys/hash traces; the existing actor/time record identifies execution. Interrupted confirmations retain the exact payload/key without automatic retry or changed re-confirmation. Older clients opt into new draft kinds explicitly. The original action release added no raw prompt/draft persistence or autonomous writes. The local 2026-10-03 recovery addition stores unsent input and explicit editable fields on the device, plus interrupted-create retry metadata; it does not store answers, member snapshots or previews, synchronize drafts to the API, or send autonomous writes. The broader design below is a future plan, not a list of shipped endpoints. Current product behavior is specified in `SPEC.md` and implemented endpoints in `api/openapi.yaml`.

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
- GET_UNPAID_PROVIDER_BILLS (individual member bill payment status is not recorded)
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
- ask whether “7” means 07:00 or 19:00, then show the absolute date/time
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
