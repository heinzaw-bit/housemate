# HouseMate web app to-do

Ordered from highest to lowest priority. This is a web app checklist, not the API or mobile implementation plan. Complete a live feature only when its endpoint is defined in `api/openapi.yaml` and the web app uses real API data. Keep the existing honest empty states until then.

Current foundation: household onboarding, money, meals, shopping, one-off chores, events, announcements, polls, personal notifications, preferences, and first-release web quality improvements are implemented. Production push configuration is enabled as of 2026-10-02; each device still needs explicit opt-in. The user reports joining/sign-in and core phone flows tested with household members; specific push, accessibility and other acceptance checks remain separate.

## P0 — Make the core household usable

### 1. Complete sign-in and member onboarding

- [x] Verify email-link sign-in reaches the household screen, survives a reload, and returns to sign-in after sign-out in the configured Supabase project.
- [x] Handle expired-link callbacks and email rate-limit failures with clear messages; verify callback behavior in the browser and error mapping in tests.
- [x] Verify Google sign-in reaches the household screen with the configured Supabase project.
- [x] Verify a Google session survives a browser reload and that sign-out returns to the sign-in screen.
- [x] Let a signed-in user join through a single-use shareable link; handle invalid, expired, revoked, already-used, and already-member invites.
- [x] Show household members and their owner/admin/member roles, with API-enforced invite and role-management controls.
- [x] Add display-name editing and household name/timezone settings; lock currency after the first confirmed financial record.
- [x] Complete a live two-account browser check: join the same household, then verify another household cannot be opened by selection or `?household=<id>` link. API isolation tests pass.

### 2. Create and view shared expenses

- [x] Build an API-backed expense list and detail view with payer, participants, date, amount, currency, and each person's share.
- [x] Build the add-expense flow: enter details, choose participants and split method, show the API-calculated breakdown, then confirm. Never show a draft as final before the API confirms it.
- [x] Support equal, exact amount, percentage, ratio, and selected-participant inputs. Show validation errors such as shares that do not add up to the total.
- [x] Format USD and THB consistently with two decimals and a clear currency label; use the household currency throughout its money screens.
- [x] Preserve entered values after a failed save and offer a safe retry without creating a duplicate expense.

### 3. Add bills and the hybrid split flow

- [x] Add bill type/provider, billing period, due date, and payment status to the money flow.
- [x] Build hybrid inputs for individual charges, shared adjustments, credits/deductions, and shared participants. Ask what an ambiguous adjustment means instead of guessing.
- [x] Show an understandable preview of the shared remainder and final per-person shares before confirmation.
- [x] Use the nine-member electricity example in `MONEY_ENGINE.md` as a web flow acceptance case; displayed shares must match the API and total the bill exactly.

### 4. Explain balances and record manual settlements

- [x] Show each member's net balance, what they owe, what is owed to them, and the expenses/bills behind those amounts. Unpaid bills remain pending until marked Paid.
- [x] Show settlement suggestions as suggestions, distinct from recorded transfers.
- [x] Let an authorized member record and confirm a manual settlement with sender, recipient, amount, date, and optional note or method; show the updated API balance.
- [x] Label settlements clearly as money transferred outside HouseMate. Do not imply that HouseMate processed or verified a payment.

### 5. Correct financial records safely

- [x] Add expense and bill edit flows. Require a fresh API preview when payer, amount, participants, or split rules change. The API ties each preview to the current record revision and exact draft.
- [x] Add mistaken-record removal actions with a confirmation step and explain their effect on balances. An API void preview shows each changed member balance or pending-bill total; the web flow requires a reason and a second confirmation, keeps the voided record visible, and excludes it from active totals. Unconfirmed entries can be discarded.
- [x] Show audit/history details for creation, material edits, voids, and settlements so members can understand what changed and who did it. Expense and bill details show actors, times, and changed fields with old and new values; reported transfers remain in Balances.
- [x] Check permissions in the UI and verify the API rejects unauthorized edits, voids, and foreign-household IDs. Edit and Void controls use API permission flags; rollback-only database checks cover unauthorized preview and confirmation and foreign-household record IDs.

## P1 — Make daily coordination useful

### 6. Populate the home dashboard

- [x] Replace available dashboard placeholders with API-backed personal balance, pending bills, and recent financial activity. The next upcoming meal, open shopping count, and chore summary are live. Home omits Events and Announcements until their feature slices exist.
- [x] Put pending bills first, ordered by earliest due date with overdue/due-soon labels; link summaries and activity to their Money tab or exact record. Show loading, empty, last-updated/stale, error, and retry states; refresh on focus/reconnection and while visible. Live mode has no sample records.

### 7. Add meals and cooking plans

- [x] Create meal checks with type, time, and response deadline; let members answer join, not joining, or maybe and see counts.
- [x] Show who is cooking and any assigned cooking roles.

### 8. Add the shared shopping list

- [x] Add, edit, assign, cancel, and mark items purchased, including quantity and optional price.
- [x] Offer “mark purchased and add expense” through the normal expense preview and confirmation flow.

### 9. Add basic chores

- [x] Show assigned, upcoming, and overdue chores; let members create, complete, skip with a reason, and request a swap as permissions allow.
- [ ] Add recurrence controls only after the first-release recurrence policy is settled in the SPEC and supported by the API.

### 10. Add events, announcements, and polls

- [x] Create events and collect attendance. Use attendees as optional default participants when starting a related expense.
- [x] Publish announcements with priority and optional acknowledgement, then show acknowledgement state. Includes selected recipients, optional expiry, review before publication, and personal action links.
- [x] Create single- or multiple-choice polls, collect votes, show results and deadlines, and prevent duplicate or unauthorized votes. Includes selected voters, atomic vote replacement/withdrawal, deadline enforcement, and API result refresh. API and rollback-only database checks passed; signed-in browser acceptance remains unverified because the browser tool rejected access to the existing localhost tab.

### 11. Deliver notifications without requiring repeated visits

- [x] Simplify Inbox into grouped compact rows and Settings into device notifications, expandable category choices and quiet hours, retaining explicit saves and read actions (deployed 2026-10-03; phone visual acceptance pending).
- [x] Add an in-app notification/activity feed with read state and links to related records. Includes unread bell, All/Unread, cursor pages, individual/bounded bulk read, transactional activity, and deduplicated scheduled reminders.
- [x] Add category preferences and quiet-hours controls through the authorized API. Preferences are personal per household; quiet hours delay all push priorities in the household timezone.
- [x] Add web push opt-in, denial guidance, existing-subscription refresh, device removal, and safe deep links with generic lock-screen text. Durable delivery, retry, quiet deferral, and expired-device handling passed simulated-transport database checks.
- [x] Enable production web push using the user-provided contact and explicitly approved server-only VAPID configuration; API redeployment reached READY on 2026-10-02 and scheduled worker runs remain healthy.
- [ ] Confirm real Android/iPhone push delivery with the app closed, denied-permission guidance, and an authorized notification deep link. User opt-in and delivery confirmation are pending; simulated tests and deployment checks do not establish actual delivery.

## P2 — Finish first-release quality

- [x] Implement PWA installation help/icons for Android Chrome and iPhone Safari, public offline navigation, connection notices, explicit updates that respect open drafts, and clear “not yet confirmed” financial states with original-key retries. Automated worker and interrupted-confirmation checks pass.
- [x] Implement keyboard/focus/label/contrast/reflow/reduced-motion improvements across completed flows. Structural form checks, palette calculations, and a keyboard install interaction pass; browser zoom remains enabled. Actual rendered and assistive-technology acceptance is listed below.
- [x] Defer inactive workspaces, share simultaneous identical reads, pause hidden/offline polling, and keep long histories paginated with unique records and protected Load more requests. Production HTML bundle measurement records the shell savings; actual mid-range phone runtime measurements remain below.

### Manual release acceptance — still required

- [ ] On both Android Chrome and iPhone Safari, install from a production HTTPS origin; check standalone launch, icons/safe areas, offline direct navigation, reconnection, interrupted confirmation, and app updates with open drafts.
- [ ] Complete signed-in keyboard and screen-reader checks, visible focus, rendered contrast, 200% zoom/320px reflow, reduced motion, and mobile layouts across all completed flows.
- [ ] Measure cold/warm loads, workspace switching, interactions, and long-history scrolling on actual mid-range phones. Record hardware/browser/network and resolve concrete remaining bottlenecks.

Use `app/QUALITY_CHECKS.md` for steps and evidence. These manual checks have not been claimed complete; access to the existing localhost browser tab was denied earlier, and no alternate browser access was used to bypass it.

## P3 — Follow-on features after the core release

- [x] Add search across household records once the core modules and API search contract exist. Global dialog, nine saved record types, literal text/type filters, authorized API cursor pages and record links implemented; signed-in/device rendering remains a manual acceptance check.
- [x] Add an assistant for authorized read-only questions and action drafts. Show unresolved fields and require an explicit edit/confirm/cancel step before any write. Six draft kinds implemented and deployed; signed-in/device acceptance remains pending below.
  - [x] Phase 1: authorized household Q&A with quick topics, optional server-side natural-language classification, deterministic answers, source links, clarification, bounded lists, and private usage limits.
  - [x] Integrate Gemini free-tier classification with server-only configuration, privacy acknowledgement, limited private-input checks and a benign live-provider check.
  - [x] Configure the approved private Gemini key and free-tier confirmation on the production API; deploy API and web.
  - [ ] Verify signed-in production free-text questions on both target phones.
  - [x] First action release: local meal-check and announcement drafts, missing-detail guidance, editable recipients/content, exact review, explicit normal-API publication, scope isolation and same-key interrupted-confirmation retry.
  - [ ] Verify signed-in meal/announcement draft publication and actual notifications on Android Chrome and iPhone Safari.
  - [x] Phase 2: shopping/chore drafts with unresolved fields, edit/confirm/cancel, creator/time/key/hash trace and durable idempotent execution.
  - [x] Phase 3: expense/hybrid-bill drafts using existing deterministic previews and confirmations. All expense split methods and explicit bill charge/credit types are supported.
  - [x] Apply the additive action-create migration and release API before web (2026-10-02); both exact-commit deployments reached READY and both CI runs passed.
  - [ ] Verify signed-in shopping/chore/money draft confirmations and notifications on both target phones.
- [x] Add natural-language expense and hybrid-bill drafting only after the manual money flows are reliable; use the API's deterministic preview for final amounts. Conservative local commands propose editable fields; ambiguous amounts/participants/adjustments require explicit input, and all shares come from the API.
- [ ] Add richer recurrence, digests, receipt handling, and reports as those product choices and API endpoints are defined.
- [ ] Prepare strings for localization; English is first, with Thai and Burmese possible later.
- [ ] Add LINE account-linking and messaging controls if LINE integration is released. Email and SMS notification controls can follow if needed.

## Household rollout operations — 2026-10-03

- [x] Activate the production web push sender with the approved existing key pair.
- [x] Verify Android Chrome phone push delivery and notification-tap app opening (user confirmed after the 2026-10-03 fix).
- [ ] Verify iPhone Home Screen closed-app delivery and exact notification links (Safari installation and delivery reported with about 30-second delay; closed-app/tap acceptance pending).
- [x] Repair web uptime monitoring using a small production runtime health endpoint; scheduled recovery verified.
- [x] Run an isolated owner-alert failure/recovery test and remove its temporary job.
- [x] Configure production-only API/web Sentry owner alerts and verify synthetic new-error alert triggering (2026-10-03).
- [x] Confirm actual owner email receipt for both Sentry project test alerts (user confirmed 2026-10-03).
- [x] Confirm uptime failure/recovery test emails (user reported both; failure screenshot provided 2026-10-03).
- [ ] Separately exercise Sentry regression alerts.
- [x] Create a fresh encrypted backup and verify the complete 66-table local restore, financial API checks and recovery SQL.
- [x] Implement private offsite backup storage/scripts and the guarded daily/weekly GitHub workflow.
- [x] Approve/provision private GitHub backup secrets, enable the schedule and verify an initial downloaded offsite restore (cloud run 37089625945, 66 tables, 2026-10-03).
- [x] Enable owner GitHub Actions failure emails (user confirmed 2026-10-03).
- [ ] Confirm the first daily scheduled backup and actual GitHub Actions failure-email receipt.
- [ ] Preserve a separate protected recovery-key copy.

## Device-local draft recovery — 2026-10-03

- [x] Automatically save new expense, bill, meal, shopping, chore, event, announcement and poll drafts, plus Assistant input/action editors; offer explicit Resume/Discard with seven-day expiry and per-account/household isolation.
- [x] Preserve interrupted create payload signatures and retry keys, require fresh API money previews, clear cancelled/completed drafts and handle corrupt/blocked storage and competing tabs without automatic writes.
- [x] Deploy draft recovery from app main e1ce531; Vercel production reached READY and all ten web route/PWA checks returned 200.
- [ ] Verify refresh/full restart, reconnection, interrupted confirmation and app-update behavior on Android Chrome and iPhone Safari. Existing-record edits, purchases, transfers/removals, settings and votes remain outside this recovery slice.

## Completion rule for every item

Use the relevant API contract, show loading/empty/error/success states, respect household and role boundaries, and test the user flow. For financial actions, verify the exact API result and retry behavior. Run the web app's lint, typecheck, tests, and build for each completed slice.

Sources: `SPEC.md` (current product scope), `AGENTS.md` (project boundaries), `app/README.md` and `api/openapi.yaml` (implemented state), plus `PRD.md`, `UX_FLOWS.md`, `MONEY_ENGINE.md`, `NOTIFICATIONS_MESSAGING.md`, `AI_ASSISTANT.md`, `TESTING.md`, and `IMPLEMENTATION_PLAN.md` for supporting detail. Where older files assume a web-only backend or put AI in the MVP, follow the current SPEC and project boundaries.
