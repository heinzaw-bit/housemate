# HouseMate product specification

Status: evolving product scope  
Last updated: 2026-10-01

## Production reliability

Production integrates optional Sentry reporting for API and browser/server errors,
with household/request data, tokens and raw exception text removed before sending.
Operators configure separate project DSNs and issue alerts. HTTP uptime checks
cover the public web page and bounded read-only API database readiness, with owner
failure/recovery alerts. Financial writes retain their existing confirmation and
idempotency behavior during incidents. Encrypted logical backups include household,
Auth and Storage metadata; restore drills validate isolated data/API recovery.
Offsite retention and unattended backups require operator setup. A local drill
does not establish Supabase Auth delivery or production cutover acceptance.

## Product goal

HouseMate helps people sharing a home coordinate money and daily life in one place. Members should be able to see what they owe, what needs attention, and what changed without reconstructing decisions from chat messages or spreadsheets.

This document describes intended features and user-visible behavior. It can change as the product is tested. `AGENTS.md` holds the stable project boundaries, while `README.md` explains the current implementation and how to run it. The older product and domain documents in this folder provide detail; this SPEC takes precedence when their release scope or product choices conflict.

## First usable release

### Brand identity

- The signed-in navigation sidebar shows the member's avatar and display name without the account email.

- The Assistant's smiling house mascot is HouseMate's shared logo across navigation, sign-in and the public landing page, alongside the existing wordmark. The same character appears in browser, Android and iPhone app icons, using forest green, a pale background and a gold sparkle. The Assistant's chat-focused layout is retained.

### Household and members

- Sign in with Google or an email link, edit a display name, and create or join a household through a single-use shareable invitation link that expires after seven days.
- A household has an owner, optional admins, and members. Members see only households they belong to; role-appropriate controls are clear in the interface.
- Owners and admins may create and revoke invitations and edit household name and timezone. Only the owner may promote or demote admins; the owner role cannot be changed in this flow.
- Choose the household's currency and timezone at creation. The owner may change currency before the first confirmed financial record or saved shopping price; afterward it is read-only. Show household settings and membership status.
- Show a dashboard with balances, pending bills, meals, shopping, chores, events, and important recent activity as each feature becomes available.

### Home dashboard

- Put a household-wide Needs attention section first with up to three earliest Unpaid bills, the next open chore, and the next meal check. Bills remain ordered by due date. Overdue means before today in the household timezone; due soon means today through seven days ahead, inclusive. Do not imply a household item is assigned to the signed-in member without member-specific data.
- Follow with compact summaries of the signed-in member's confirmed net balance, the household's total Unpaid bills, and open shopping and chore counts. Amounts use the household currency; voided bills are excluded from pending totals. Show all pending bills in totals even though only three appear in Needs attention. Unpaid bills affect balances only when marked Paid.
- Show the three most recent financial events initially, with an option to expand to the ten returned by the API. Recent activity includes creation, edits, voids, provider-status changes, and reported manual transfers, newest first. Each entry opens the specific expense, bill, or transfer; record history shows prior values. Home has no quick-action button group; creation actions are available in their Money and House sections.
- Show loading, genuine empty results, errors with retry, and a last-updated time. Refresh on return to Home, window focus, reconnection, and roughly every minute while visible. Keep the last successful snapshot during temporary errors, visibly marked as possibly out of date; discard it on an access failure or household change.
- Meals, shopping, and chores use live household data. Events and announcements are not yet part of the Home summary. Do not present invented counts or sample records as live data.

### Money, bills, and settlements

- Money opens on Balances and uses one shared header with the signed-in member's confirmed balance and the household's unpaid-bill total. Expenses and Bills remain separate sections with section-specific add actions. Their record lists and details sit side by side on desktop; on small screens, selecting a record focuses its detail with a Back control to return to the list. Do not present household unpaid bills as the signed-in member's personal debt.
- Record an expense or bill with a payer, amount, date, description, participants, and split method. Preview each person's share before confirmation.
- Support equal, exact amount, percentage, ratio, selected-participant, and hybrid splits. Hybrid bills include individual charges, shared adjustments, credits or deductions, and a shared remainder. Ask the user to classify ambiguous adjustments.
- In the first bill flow, the shared remainder is split equally among selected active members. Each extra amount must be classified as a charge or credit for one member or a selected group. If an amount is outside the household, enter a household-only bill total before preview. The preview shows the base amount, every adjustment, and final shares that exactly total the household bill. An unassigned or ambiguous amount cannot be confirmed.
- The bill form presents Details, Split and adjustments, and Review in sequence. Details must be valid before moving to the split step; charges and credits are added only when needed. Review displays the API-calculated allocation and is the only step that can confirm the bill. Returning to edit a field invalidates the old preview.
- A bill has a type, provider, billing period, due date, and one manually recorded **Unpaid/Paid** status for payment to the provider. Any active member may update that status; HouseMate records the change but does not process a payment. Per-member repayments belong to the later settlement flow.
- Show each member's net balance and the confirmed expenses, Paid bills, and recorded transfers behind it. Why this balance and Recorded transfers each initially show the three newest records by their source/transfer date; a Show more/Show less button expands or collapses the loaded list, with existing pagination available while expanded. Changing the selected member resets the source preview. A positive balance is owed to that member; a negative balance is what they owe. Unpaid bills appear as pending and affect balances only when marked Paid.
- Show settlement suggestions as informational amounts, separate from the history of transfers members have recorded. Suggestions do not change balances.
- The record creator or a household owner/admin may edit or void a confirmed expense or bill. The web detail shows a "Remove mistaken expense/bill" action for this correction. The web flow requires a reason, a fresh API preview of the balance or pending-bill effect, and explicit confirmation. Editing payer, amount, participants, or split rules also requires a fresh API-calculated preview. A stale record or expired preview requires another preview. A voided record remains visible with actor, time, and reason, but cannot be edited or have its bill status changed. Its shares no longer affect balances; voided Unpaid bills leave pending totals. Recorded settlements remain unchanged. Store before and after versions of edits and voids for history. An unconfirmed web form may be discarded without creating a financial record.
- Every active household member may read a financial record's history on its expense or bill detail. Show creation, edits, bill provider-status changes, and voids with who acted and when. An edit shows changed fields with old and new values, including split inputs and final shares. Keep reported transfers and their recorder, transfer date, and recorded time in Balances. Only the creator or a household owner/admin sees edit and void actions; the API enforces those permissions and household isolation for every request.
- Let either the sender or recipient record a manual settlement after money has moved outside HouseMate. They review an API-calculated balance preview, then confirm the sender, recipient, amount, transfer date, and optional note or method. A custom amount is allowed with a warning if it exceeds the suggested amount. Show who recorded the transfer and when, and refresh balances after confirmation. A recorded settlement is a member's report, not proof that a payment provider completed or verified a transfer.

### Shared household life

- House uses one shared header and an Add menu for chores, shopping items, meal checks, events, announcements, and polls. An API-backed personal action preview surfaces the signed-in member's next unanswered meal check, pending chore swap, assigned open chore, assigned shopping item, targeted unexpired announcement needing acknowledgement, and eligible open poll without a saved vote. It never presents household-wide work as personally assigned. A meal reply can be sent from the preview; other cards open the exact record. Refresh the preview after changes and on return to the tab, and mark a failed refresh as possibly out of date. The preview is intentionally limited; each section remains the place to browse all available records. Lists use filtered API pages so older records remain reachable.
- Open House on Chores, with Chores, Shopping, Meals, Events, Announcements, and Polls navigation. Chores separates swap requests, personal assignments, overdue and upcoming work, and history. Shopping separates items to buy from purchased and cancelled history. Meals separates checks needing the member's reply, upcoming checks, and past checks; active response deadlines appear in the list. Events separates upcoming and past plans. On small screens, selecting a record focuses its detail and offers a Back control.
- Any active household member can publish a meal check with a type (breakfast, lunch, dinner, or other), question, meal time, response deadline, and selected active participants. Times are entered in the household timezone; the deadline must be in the future and before the meal. Each selected member can answer joining, not joining, or maybe, and change their own answer until the deadline. Responses lock at the deadline. Show each member's answer and counts for every state, including awaiting an answer.
- Any active member can assign short custom cooking roles, such as Cook or Helper, to selected active participants. Show the cooking plan on the meal detail. The home dashboard shows the next upcoming meal check and links to it.
- Any active household member may add and edit an open shopping item, assign an active member as buyer, cancel an open item, or mark it purchased. An item can record a name, optional quantity and unit, priority, note, estimated price, and actual purchased price. Unassigned open items are Needed; assigned open items are Assigned. Purchased and Cancelled items remain visible and cannot be edited or changed to another status in this release. A stale item revision must be refreshed before a change.
- Marking an item Purchased can optionally create a shared expense. The member enters the actual price, chooses the payer and split participants, selects a split method, reviews the normal API-calculated expense preview, and explicitly confirms. The member who marks it purchased is the default payer, but can choose another active member. Purchase and expense confirmation happen in one API transaction and the item links to the expense. A retry with the same confirmation key returns the same expense. Marking Purchased without an expense does not change balances. Prices use integer minor units with an explicit item currency; the first saved shopping price locks the household currency so old prices cannot change meaning. The home dashboard shows a live count of Needed and Assigned shopping items.
- Any active member may create a one-off chore with a title, optional note, due date and time in the household timezone, and one active assignee. Members see chores assigned to them, upcoming chores, overdue chores, and completed or skipped history. The assignee or an owner/admin can complete or skip an open chore; skipping requires a reason. An owner/admin can directly reassign an open chore.
- The assignee may request a swap with another active member. The requested member may accept or decline; accepting reassigns the chore. Only one request may be pending for a chore. Completing, skipping, or direct reassignment cancels a pending request. Stale revisions must be refreshed before a change. Home shows live open and overdue counts and the next due chore.
- Any active household member may create a future event with a title, optional details and location, and a start and optional end time in the household timezone. Members can browse upcoming and past events and see every current active member's attendance as Going, Maybe, Not going, or Awaiting. Each member may set or change only their own response until the event starts; responses then close. Event creation is safe to retry with the same key, and all reads and writes require active membership in the selected household.
- From an event, a member may start an expense draft with the event title and current Going members suggested as participants. If nobody is Going, the draft requires the member to select participants. The suggestion remains editable; payer, amount, date, split, and participants all go through the normal API-calculated expense preview and explicit confirmation. This first version does not persist an event-expense link or automatically change confirmed expenses when attendance changes.
- Any active member may publish an announcement with a title, message, Normal/Important/Urgent priority, selected active recipients, optional acknowledgement request, and optional future expiry in the household timezone. Review the draft before publishing. Publication is safe to retry; content and recipients stay fixed after publication. Selected active recipients, the author, and active owners/admins can read targeted records. Members acknowledge only as themselves; repeated acknowledgement requests retain the original timestamp. New acknowledgements close at expiry. Show recipient acknowledgement state and counts; departed recipients retain their recorded acknowledgement but do not count as actively awaiting. Browse All, Needs your acknowledgement, and Expired views.
- Any active member may publish a poll with a question, 2–10 distinct choices, single or multiple selection, selected eligible active voters, and a required future deadline in the household timezone. Review before publishing; question, choices, eligibility, and deadline stay fixed afterward. Selected active voters, the author, and active owners/admins may read aggregate results and their own saved selection. Only eligible active members may vote as themselves before the deadline. Saving replaces that member's entire selection atomically; retries never add another vote, and an empty selection withdraws it. Reject duplicate choices and option IDs belonging to another poll. Previous votes remain counted if a member leaves. Show option counts, distinct respondents, eligibility counts, and the deadline; multiple-choice percentages use respondent count and may sum beyond 100%. Browse Open, Needs your vote, and Closed views. Results refresh through the API about every 15 seconds while the section is visible and on return; preserve unsaved choices during refresh. This release has aggregate visible results rather than an anonymous-ballot mode.
- Deliver relevant activity and reminders through the personal notification feed described below. Mobile push remains planned for the mobile client.

### Notifications and reminders

- While the signed-in web app is open, new unread announcements, chore assignments/reassignments, and added shopping tasks show a brief popup from the selected household's authorized recipient feed, respecting the member's current In app category preferences. The first feed load establishes a quiet baseline; older entries are not replayed. The existing visible/online refresh detects updates roughly every 30 seconds and on return. Show at most three popups, with overflow available in the inbox. Each has View and Dismiss controls, closes after eight seconds, and pauses dismissal during hover, keyboard focus, or a hidden tab. It never takes focus automatically. Dismissal does not mark a notification read. Popups sit at the top-right on desktop and above phone navigation with safe-area spacing. Account/household-scoped timestamp and ID metadata in tab memory prevent duplicates across route changes; feed content and tokens are not persisted for this feature. Switching account/household discards visible popups and pending responses; access failures clear them. Quiet hours continue to apply to push delivery only. These in-app popups require neither browser notification permission nor live push setup.
- The header bell shows the selected household's unread count and opens Notifications. Browse All or Unread, load older entries with stable cursor pagination, mark individual entries read, or mark all entries through the newest displayed item read. Links open the exact related money or House record, whose API checks access again. Read state belongs to the recipient; another member cannot change it. Temporary refresh errors mark the retained view as possibly out of date; access failures clear it. Refresh while visible roughly every 30 seconds and on focus or reconnection.
- New expense/bill creations, revisions and voids, manual transfers, selected meal invitations and cooking roles, chore assignments/status and swap requests, shopping changes, events, targeted announcements, and eligible polls create recipient entries in the same transaction as the action. Active household membership is required to read the feed. Targeted communications are sent only to selected recipients/voters, transfers to their two participants, and assignments to the relevant members. Existing activity is not backfilled; unresolved deadlines can produce reminders after the worker starts.
- The API generates reminders about once per minute while running on a persistent host, or through a configured minute-level scheduler on serverless hosts. Vercel Hobby uses an external scheduler calling a secret-protected API job; it does not run a persistent background timer. Reminder timing depends on successful scheduler runs, so production requires scheduler setup and monitoring. Bills notify share participants within two household calendar days of due date, on the due date, and once overdue, using only the nearest phase at each check. Unanswered meals and unvoted polls get one reminder within an hour of closure; open assigned chores get one within a day and one overdue; upcoming events get one within a day for Going, Maybe, or unanswered members; expiring announcements get one within a day for recipients still needing acknowledgement. Deterministic keys prevent repeated reminders for the same phase. Completed, answered, paid, expired, or otherwise irrelevant items are rechecked before push delivery.
- Each member chooses In app and Web push independently for Expenses, Bills, Settlements, Meals, Chores, Shopping, Events, Announcements, and Polls in each household. All in-app categories are initially on; Shopping push is initially off. Muting affects future feed entries and leaves existing history available. Push requires an enabled browser device and is also checked against current preferences before sending.
- Optional quiet hours use the household timezone, support overnight windows, and delay every push priority until the window ends. They do not hide the inbox. Defaults suggest 23:00–07:00 with quiet hours initially disabled. A delayed reminder is skipped if it has expired, been answered, or been read.
- Web push is explicit opt-in after an explanation of its value and privacy. Do not open a permission request automatically. Show instructions for browser denial or unsupported platforms. Devices belong to the account across households, with up to ten active devices; members can disable them, and signing out removes the current browser subscription. Refresh an existing subscription on a signed-in visit; missing subscriptions require explicit Enable again, and remote-disabled devices are never silently re-enabled. Expired provider subscriptions are disabled; transient errors have bounded retries.
- Lock-screen title and body are generic: no financial amounts, household names, or private message text. Push links stay on the app origin and require authentication and authorized household access. Newly enabled devices receive only subsequent updates. Provider acceptance can be ambiguous during retries; notification tags coalesce duplicate deliveries, but device delivery is not guaranteed.
- Live web push requires a real server contact email or HTTPS support URL, server-only VAPID private key, and a supported secure browser context. Until configured, the inbox, reminder generation, and preferences work and the web screen explains why push is unavailable. Production requires HTTPS; some mobile browsers require an installed PWA. Native mobile push, email, SMS, and LINE delivery are not implemented in this slice.

### Web and mobile experience

- Provide the essential household workflows in both the responsive web app and the iOS/Android mobile app. Layout and platform controls may differ, but confirmed financial results must match.
- Clearly distinguish loading, empty, error, draft, pending, and confirmed states. A financial action is final only after the API confirms it.
- Make common flows usable with accessible labels, keyboard support on web, and meaningful notification deep links.

### Appearance and account menu

Signed-out visitors at the root see a public landing page with shared-money,
daily-coordination and household-updates features, setup steps, an interactive
product tour, FAQ and a factual privacy overview. Product illustrations use
explicitly labelled fictional records. Get started/Sign in open `/sign-in`;
restored members continue to their dashboard. Invite/household deep links and
authentication callbacks bypass marketing and preserve the normal access flow.
Landing content supports light/dark mode, keyboard controls and phone layouts.
The public page is divided into Overview, Features, How it works, Tour, Privacy
and FAQ along the natural vertical scroll. Fade each upcoming section in as it
enters view, then keep it visible. Header anchors jump directly to sections;
there are no Next/Previous controls, progress counters or forced scroll snapping.
Reduced motion, unavailable animation APIs and keyboard focus keep content
visible. All sections remain in the document/accessibility flow, grow on small
screens and are readable without JavaScript.

Web provides a light/dark toggle beside the header profile icon and on sign-in.
Initially it follows the device preference, then remembers an explicit choice
locally across visits/tabs. The profile icon opens the current account, Profile,
Switch Google account and Sign out. Google switching prompts for account selection
using the configured provider and retains the current session until completion.
Members must finish or discard open drafts before switching or signing out.
Account changes replace household UI state and request the new account's authorized
data. One browser has one active session shared across tabs; switching does not
merge users, memberships or financial records.

### First-release web quality

- Prioritize Android Chrome and iPhone Safari. Offer installation help on sign-in and Profile, an explicit browser install action when available, home-screen icons, and safe-area spacing. Installation and notification permission are separate choices.
- Cache only a public offline navigation screen. Never cache private household/API responses in the service worker or queue household writes. Existing displayed data may be stale offline; show a connection notice. Reconnection refreshes reads on return and never resubmits a financial action automatically.
- A disconnected, timed-out, or unreadable financial confirmation reply is **Not yet confirmed**: the server may already have saved it. Retain the current draft, preview, and confirmation key for an explicit retry. A request rejected locally because the browser is offline is clearly identified as not sent. Actions without a confirmation key, including bill payment-status changes and shopping purchases without an expense, require checking the current record before retrying. API requests have a 20-second timeout.
- Offer an explicit reload when a service-worker update is ready. Block that reload while a creation/edit/review form, unsaved poll vote, or settings change is open; recheck pending work when activation finishes. Register a browser leave warning while those drafts are open. Drafts remain in memory: switching sections, closing/terminating the app, or overriding the browser warning can still discard them. An update activated in another tab does not force this tab to reload.
- Provide a skip-to-content link, semantic headings, labels and selected-filter state, visible keyboard focus, readable text colors, larger touch targets, and reduced-motion styles. Allow browser zoom, wrap long content, and keep navigation available across mobile breakpoints. Structural accessibility and palette checks supplement manual screen-reader, keyboard, 200% zoom, and phone acceptance checks; automated results alone are not accessibility certification.
- Load inactive workspaces separately, coalesce simultaneous identical API reads, and pause background polling while hidden or offline. Expense/bill and notification histories remain API paginated; overlapping pages cannot duplicate rendered records. Measure initial production JavaScript separately from authenticated workspace loading and actual phone performance.

## Currency behavior

- Support **USD** and **Thai baht (THB)**. Each household chooses one currency, either USD or THB, and all its financial records and totals use that currency.
- Display amounts with two decimal places and a clear currency label. Store and calculate cents or satang precisely so every split allocates the full original amount.
- Do not combine USD and THB in one balance or convert between them in the first release.
- Once a household has financial activity, changing its currency is unsupported in the first release; historical amounts must not silently change meaning.
- The electricity example in `MONEY_ENGINE.md` is a product acceptance scenario for hybrid bills.

## Follow-on features

### Assistant phase 1 — household questions

- Assistant centers on a large chat box directly below the retained house mascot and a short heading. Only three small shortcuts follow it: household overview, meal plans and personal balance. The latest answer stays below the input; up to five earlier answers are available in collapsed history, with six total held in tab memory. Record previews show three rows with explicit expansion/collapse, source links and checked times.
- The composer uses normal page flow, with no sticky overlay or automatic answer scrolling. Ctrl/Command + Enter submits only with consent; Enter alone inserts a newline. Cancellation and failed requests preserve unsent input. Clearing answers preserves the unfinished question and restores focus. The privacy acknowledgement appears when the member types a question or has already accepted; full privacy details are expandable.
- Meal answers offer “Plan a meal,” opening the existing household-scoped meal creation form. Balance/bill answers offer “Calculate a split,” opening the existing expense form with deterministic API previews and explicit confirmation. These actions do not create records directly from chat; AI-generated meal or financial drafts remain later features.
- Active members can ask quick questions about their confirmed net balance, unpaid provider bills, household overview, open chores, shopping, upcoming meals/cooking roles, and upcoming events. Answers come from authorized API services, include source links scoped to the selected household, show when they were checked in the household timezone, and disclose bounded lists. Ask again to refresh; earlier answers are snapshots.
- Optional general free-text questions use server-only Gemini (`gemini-3.1-flash-lite`) to classify one supported question. Only that question is sent; household records, balances, member lists, credentials and previous answers remain in HouseMate. The API validates the classification and produces the answer itself. Today uses the household timezone. Named-member balances are available in Money; unsupported questions receive clarification.
- Gemini unpaid-service terms allow product-improvement use and human review and restrict personal/confidential inputs. Before a typed question, the member must acknowledge the notice and confirm it contains no personal or confidential information. The API checks known active-member names/emails, digits, links and credential markers before budget/provider calls. This is a limited guard, not a complete personal-information detector. Quick topics need no acknowledgement and never contact Google.
- Bills have provider Paid/Unpaid status, not per-member bill payment status. The assistant does not infer member payment status or calculate financial results from conversation text.
- Quick questions work without AI credentials and after provider failures or limits. Free text is limited to 1,000 characters, one provider attempt with a 12-second timeout and 400 output tokens, 20 attempts per account per UTC hour and 200 globally per UTC day. Failed attempts consume a slot. Usage counters are private and expire; HouseMate does not persist prompts or conversation history. There are no automatic retries or paid fallbacks. API configuration requires a private key and explicit operator confirmation of a Google Free Tier project with billing disabled; the app cannot verify billing from a key.
- The web keeps up to six answers in memory, clears them on account/household/session changes, supports cancellation without automatic retry, and protects unsent questions from app-update reloads. Record text is rendered as text. Membership is checked before provider use, by each data service, and before returning an answer.
- Shopping/chore drafts and expense/bill drafts are subsequent phases, still pending. Any future mutation needs an editable validated draft, unresolved-field guidance, explicit confirmation/cancellation, idempotent execution, and an audit record. No assistant mutation endpoint exists in phase 1.

### Other follow-on features

- More advanced recurring bills and chores, digests, receipt handling, richer reports, and LINE messaging can follow the core workflows.
- Email or SMS reminders can be considered later if the household needs them.

## Outside current scope

- **No payment integration yet:** no card charging, bank linking, in-app money transfer, payment intents, or automatic transfer verification.
- No automatic currency conversion or exchange-rate service.
- No autonomous AI financial action or purchase.

## Product acceptance examples

1. Two members can join the same household and see their own household activity; another household's private data is unavailable to them.
2. A USD household and a THB household can each create an expense, view exact shares, and record a manual settlement without currency mixing or lost cents or satang.
3. Equal and hybrid splits allocate the full amount exactly once, including the electricity scenario.
4. Editing or voiding an expense leaves an understandable record of what changed; a suggested settlement is never presented as a completed transfer.
5. Web and mobile show the same confirmed balance and bill state after an API update.
6. A notification opens the relevant item and respects the member's preferences.
7. When AI is released, an AI draft changes no household record until a member reviews and confirms it.

## Open product decisions

- Which chores and bills need recurrence after the first one-off release, and at what schedule and cutoff rules?
- When should native mobile push and optional email, SMS, or LINE delivery be added beyond the implemented web inbox and push controls?
