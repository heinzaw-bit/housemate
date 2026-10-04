# HouseMate project handoff

## Goal and current state

HouseMate coordinates shared household money and daily work. The first release uses one currency per household (USD or THB), records payments manually, and has no payment integration. This workspace contains independent `api/` (Node.js/TypeScript) and `app/` (Next.js) Git repositories, registered as submodules of the whole-project repository; `mobile/` (React Native/Expo) is planned but does not exist. Inspect root and child Git status before any Git operation. Commit and push child code first, then commit and push the parent pointers and shared documents.

The configured Supabase project is `avbroprkgyrkophcxvtm` (`https://avbroprkgyrkophcxvtm.supabase.co`). Local API and web environment files exist. Database migrations 001–016 and the additive action-create retry migration were applied to that project; the latter was verified on 2026-10-02. Local services were last verified on 2026-09-30 at `http://localhost:3001` (API, restarted with notification routes/worker, PID 38684) and `http://localhost:3000` (web, PID 9140); running processes are transient, so check them again.

## GitHub repositories — 2026-09-30

The user created three private repositories and authorized pushing them to **heinzaw-bit**. GitHub CLI's authenticated login was verified as heinzaw-bit. The actual umbrella name is **housemate**, not the earlier proposed house_mate. Remotes are `https://github.com/heinzaw-bit/housemate.git`, `https://github.com/heinzaw-bit/housemate-api.git`, and `https://github.com/heinzaw-bit/housemate-app.git`. API and app have independent commits on main and are registered as submodules with main as their tracking branch. Root Git history is preserved. Each child has engineering instructions and a README linking the shared product documents; neither needs a sibling to build.

Before publication, both projects passed lint, typecheck, tests (35 API and 50 web), and production builds. Staged files and existing root history were checked against private local environment values and common credential patterns; API `.env`, app `.env.local`, dependencies, build output, local run files, and Vercel metadata are ignored. Existing local credentials and database configuration were preserved. The user completed GitHub CLI's workflow permission authorization, and all three main branches were pushed successfully. The API initial commit is `a4269e2c79a37786a78ba4c536caad5bdcd9fa7a`; web initial is `d6d125334dd9801349dce2ebfd0ca1082c6c7c71`. Web CI's clean install found missing optional `@emnapi/core` and `@emnapi/runtime` lockfile entries; repair commit `38d961b34e2c3d6ebba52c6cd49d75d4d5d42bbd` adds only those entries, and local `npm ci --dry-run` passes. Both API and repaired web GitHub CI completed successfully, including clean installation, lint, typecheck, tests, and production builds. The whole-project pins the repaired web revision. GitHub publishing does not deploy either service.

## Architecture and files

| File or folder | Purpose |
| --- | --- |
| `AGENTS.md`; `CLAUDE.md` | Durable project boundaries; CLAUDE imports AGENTS. Also read `app/AGENTS.md` before web edits. |
| `SPEC.md` | Current product behavior, scope, and open decisions. Older root PRD/domain docs are supporting references. |
| `WEB_APP_TODO.md` | Prioritized web checklist and completion status. |
| `README.md`, `api/README.md`, `app/README.md` | Setup, use, and checks. |
| `api/openapi.yaml` | Concrete implemented HTTP contract; update before clients rely on API changes. |
| `api/src/application.ts`, `runtime.ts`, `app.ts`, `main.ts`, `auth.ts`, `config.ts`, `database.ts` | Injectable Fastify route factory, production wiring, Vercel/local entry points, auth/config, and serverless pool settings. |
| `api/src/{households,expenses,bills,money,meals,shopping,chores,events,announcements,polls,dashboard}.ts` | Authorized persistence and business rules. Communication routes/access helpers and financial calculation, edits, history, and void logic have separate modules. |
| `api/sql/001_*.sql`–`015_notification_trigger_fix.sql`, `api/scripts/` | Private PostgreSQL schema, migration runners, rollback-only database smoke checks. |
| `app/lib/api.ts`, `auth-context.tsx` | Typed API client and Supabase browser session handling. |
| `app/components/{expense,bill,balance,meal,shopping,chore,event,home-dashboard,house-workspace}.tsx` | Main user flows. `void-confirmation.tsx` contains the financial void confirmation UI. |

## Implemented

- Google and email-link sign-in, callback errors, session restoration, sign-out; profile name, household creation/selection/settings, member roles, single-use seven-day shareable invites. Two-account join and household isolation were browser checked earlier.
- Shared expenses with selected participants and equal/exact/percentage/ratio splits; hybrid bills with individual/shared charges and credits; API previews and idempotent confirmations; manual provider Paid/Unpaid bill status.
- Balances, source entries, suggestions, recorded manual settlements, edits, void previews/confirmations, and field-level financial history. Home shows live balances, pending bills, recent financial activity, next meal, shopping count, and chore summary.
- Meal checks, responses, cooking roles; shopping item lifecycle and optional purchase plus expense; one-off chores with assignment, due/overdue views, completion, skip reason, and accept/decline swap requests. `WEB_APP_TODO.md` marks these basic flows complete.
- Responsive shadcn/Base UI web interface, local PWA shell, and a separate development-only design preview. `mobile/` has no implementation.
- Home was reorganized around household-wide Needs attention (three earliest unpaid bills, next chore, next meal), compact personal/household summary cards, an expandable recent-activity preview, and quick actions. Expense and bill quick actions open their create forms through `&add=1`; Events and Announcements no longer occupy Home. This is a web layout change using existing dashboard fields, with no API contract change. The user chose existing household-wide data over member-specific API work.
- Money now opens on Balances with one shared header, personal confirmed balance, household unpaid-bill total, and Balances/Expenses/Bills navigation. Confirmed expense, bill, status, void, and settlement actions refresh the summary. Expense/bill record selection focuses details on small screens with a Back control. The bill form moves through Details, Split and adjustments, and API-calculated Review before confirmation. This is a web-only flow/layout change using existing API endpoints.
- House now opens on Chores with one shared header, an Add menu, and a personal next-actions preview. The new authorized `GET /v1/households/{id}/house-actions` endpoint returns up to three items each for unanswered meal checks, swap requests, assigned open chores, and assigned shopping. The web preview shows the first of each, supports a direct meal reply, and refreshes after relevant changes. Chores, Shopping, and Meals separate active work from history and focus record details on small screens with Back controls. Meal lists now support filtered, paginated reply/upcoming/past scopes. The API contract and SPEC document this behavior. The local API process was restarted after this endpoint change; authenticated browser acceptance was not completed because both computer-use helpers failed to initialize.
- Events and attendance are implemented through `api/src/events.ts`, migration `012_events.sql`, and `app/components/event-workspace.tsx`. Active members can create future events, browse upcoming/past, and change their own Going/Maybe/Not going response until start. Event creation is idempotent. “Add expense from event” opens a new expense draft, prefills the event title, and suggests currently active Going members as participants; the user may change all fields before the normal API preview/confirmation. No event-expense relationship is persisted, and later RSVP changes do not change confirmed expenses. The migration was applied. API unit tests and rollback-only `smoke:events` passed; web lint, typecheck, 25 tests, and build passed. The local API was restarted; `/health` returned 200, the unauthenticated events route returned 401, and `/house` and `/money` returned 200. Authenticated browser acceptance remains unverified.

## Section 10 completion — 2026-09-30

- Section 10 announcements and polls are implemented (2026-09-30). Any active member can review and publish targeted announcements with Normal/Important/Urgent priority, optional future expiry, and optional acknowledgement. Selected active recipients acknowledge only as themselves; retries keep the original timestamp. Polls support 2–10 distinct choices, single/multiple selection, targeted eligible voters, and a required future deadline in the household timezone. Saving atomically replaces the caller's selection; an empty selection withdraws it. Votes close at the deadline, and previously saved votes remain counted if a member leaves. Published content and audience are immutable in this first version. Selected recipients/voters, authors, and active owners/admins can read targeted records. Poll output exposes aggregate counts and the caller's own choices; there is no anonymous-ballot mode.
- Migration `013_announcements_polls.sql` is applied, with RLS on all six private tables and composite/unique vote constraints. `communication-access.ts`, `communication-routes.ts`, `announcements.ts`, and `polls.ts` implement the API; `announcement-workspace.tsx`, `poll-workspace.tsx`, shared `communication-parts.tsx`, and `use-house-records.ts` implement the web. House has six sections and Add actions, and personal next actions include pending acknowledgements and votes. New sections use paginated filters, mobile detail navigation, safe publication retry keys, and API refresh every 15 seconds while visible; unsaved poll choices survive result refresh. Deep links use `view=announcements&announcement=<id>` or `view=polls&poll=<id>`, and `add=1` opens their form. SPEC, OpenAPI, READMEs, and checklist are updated. API lint/typecheck/build and 32 tests passed; web lint/typecheck/build and 27 tests passed. `smoke:announcements-polls` and `smoke:house-actions` passed and rolled back their isolated fixtures. OpenAPI parsed and all local references resolved. `/health` returned 200, the unauthenticated polls route returned 401, and `/house` returned 200. Signed-in browser acceptance is unverified: `cua.getState()` succeeded, but binding the existing localhost IAB tab was rejected by the browser URL security policy (reported unsupported protocol). Do not circumvent that denial through another browser surface.

## Section 11 implementation — 2026-09-30

- Notifications are implemented in `api/src/notifications.ts`, `notification-routes.ts`, `notification-worker.ts`, `app/components/notification-bell.tsx`, `notification-workspace.tsx`, `app/lib/web-push.ts`, `app/app/notifications/page.tsx`, and `app/public/sw.js`. The bell opens a personal per-household All/Unread inbox with unread count, stable cursor pages, individual read, bounded Mark all read, and exact record links. All operations enforce the caller's active household membership and recipient ownership. PostgreSQL microsecond precision is retained in cursors/read boundaries to prevent skipped or duplicated pages. Web refresh runs roughly every 30 seconds while visible and on focus/reconnection; access failures clear retained data.
- Migration 014 adds recipient feed, per-member preferences, account push devices, and a durable push queue with private RLS tables and transactional source triggers. The first smoke check found a `title` local variable/source column collision; migration 015 replaces the trigger function with a distinct local name. Both are applied, and `migrate:notifications` applies 014 once and safely reapplies 015. Do not edit already-applied 014 to hide that correction or omit 015 in a new setup.
- Nine categories have separate In app/Web push controls per member per household. In-app defaults on; Shopping push defaults off. Existing feed history is retained when a category is muted. Optional quiet hours suggest 23:00–07:00 in the household timezone, initially disabled, and delay every push priority. Source actions create entries transactionally, with targeted communications/assignments/transfers scoped to relevant recipients. There is no activity backfill. The API runs a worker on startup/every 60 seconds for deterministic reminder phases; keep an API instance running. Detailed reminder windows are in SPEC.
- Web push includes explicit opt-in, permission-denial/unsupported guidance, existing-subscription refresh on signed-in visits, remote device disable, current-browser unsubscribe on sign-out, source/access/preference checks before sending, quiet deferral, leased SKIP LOCKED queue claims, retries, and 404/410 device disable. A device belongs to one account across households and cannot be claimed by another; ten active-device limit. Missing/disabled subscriptions require explicit Enable again (`refreshOnly` never re-enables them). New opt-in/re-enable does not replay old inbox or queued updates. Supported HTTPS push-provider endpoints and valid encryption keys are validated before persistence and delivery. Service workers store no JWT and only open same-origin supported record links. Push text is generic, without household names, amounts, or private message text. Topics/tags coalesce repeats; actual device delivery is not guaranteed.
- **User has no public contact email or HTTPS support URL.** VAPID keys were generated into ignored `api/.env` without printing values, but no fake/private contact was supplied. Live push remains disabled (`loadConfig().vapid` is null); the web explains setup is pending. Inbox, reminder generation, and preferences work independently. Once a real user-provided contact exists, set server-only `VAPID_SUBJECT`, restart API, then explicitly enable and check a real browser/device. Native mobile push, email, SMS, and LINE remain unimplemented.
- Verification: API lint/typecheck/build and 35 unit tests; web lint/typecheck/build and 30 unit tests passed. Rollback-only `smoke:notifications` passed recipient/privacy isolation, retry deduplication, cursor/read precision, preferences, reminders, device ownership, no backlog replay on opt-in/re-enable, quiet deferral, removal, provider 410, and RLS with **simulated transport**. Regression smoke checks passed for financial edits/voids, settlements, meals, shopping, chores/swaps, events, and announcements/polls. OpenAPI parsed and all 421 local references resolved. API health returned 200; unauthenticated notification/config endpoints returned 401; web `/notifications` returned 200. Actual provider/device delivery and signed-in visual acceptance are unverified; the browser tool's previous localhost denial must not be circumvented. SPEC, OpenAPI, READMEs, and checklist are updated.

## P2 first-release web quality implementation — 2026-09-30

- The user asked to implement the unchecked **Finish first-release quality** items and selected **both Android Chrome and iPhone Safari** when asked which platforms to prioritize. Implementation is complete; actual-device, signed-in rendered accessibility, and runtime performance acceptance remains explicitly unchecked in `WEB_APP_TODO.md` and documented in `app/QUALITY_CHECKS.md`. Do not claim those checks ran or bypass the prior localhost browser-access denial.
- `InstallApp`, `install-prompt.ts`, PNG/maskable/Apple icons, manifest identity/scope, Apple metadata, and safe-area spacing implement installation help on sign-in and Profile and retain an offered browser install prompt until explicit interaction. Installation does not enable push. The production-only worker caches only public `offline.html`, waits for an explicit update activation message, cleans only its own old caches, and falls back on offline/timed-out direct navigation. No private household/API cache or write queue was added.
- Root `AppStatus` shows offline/reconnection/update notices. `pending-work.ts` protects open money/house creation forms, shopping purchase, removal, dirty poll votes, and settings from update reloads and registers a browser leave warning. Update activation rechecks pending work, and another tab's activation does not automatically reload a draft. Drafts remain in memory; section changes, app termination, or overriding browser leave warnings can discard them. This is not crash recovery or offline editing.
- `lib/api.ts` now rejects locally offline requests, bounds requests to 20 seconds, rejects unreadable non-204 success replies, and shares only simultaneous identical GETs without independent abort signals. Nothing remains cached after the request. Financial confirmation UI uses `financial-confirmation.ts` for **Not yet confirmed** states, keeps the original review/key, and never automatically retries a write. Bill status and shopping purchases without an expense ask members to refresh the current record before trying again. An ambiguous removal confirmation can retry its original key after preview expiry, allowing the API to return an already-saved void.
- Accessibility changes include a skip link, native card headings, semantic filter selection state, darker small/muted/error text, stronger visible focus (including household selection), larger touch targets, wrapping/min-width rules, zoom-enabled viewport, and reduced-motion styles. Mobile navigation now stays available through 900px, closing the earlier 850–900px gap. Main routes have distinct document titles.
- Workspaces load with `next/dynamic`, inactive money/House sections load separately, bell reads request one entry plus unread count, and background reads pause when hidden/offline and resume on return. Expense/bill Load more tracks raw offsets and prevents concurrent requests; `appendRecords` uses a Set to deduplicate overlapping pages, also used by Notifications. Long history rows use content-visibility to reduce off-screen rendering work.
- Verification: web lint, typecheck, **50 tests in 13 files**, and production build pass. Tests cover an interrupted expense confirmation and identical-key retry, removal retry after expiry, explicit keyboard installation, update/draft races, public-only service-worker behavior, request timeout/read sharing, 1,000-record page merging, palette contrast, and structural axe checks on the main forms. DOM accessibility runs disable axe color contrast because jsdom lacks layout; palette checks and structural checks do not establish screen-reader/zoom/rendered conformance. No API source/contract/migration changed, so no API restart was required.
- `npm run measure:bundle` after the production build reports **961,396 raw / 287,797 gzip bytes** in 14 initial script assets for each of Home, Money, House, Notifications, and Profile. The pre-change baseline was **1,144,110 raw / 325,949 gzip bytes** in 9 script assets: about **16.0% raw / 11.7% gzip less**. This metric is only the signed-out/session-restoration HTML shell, excluding deferred authenticated workspaces; it is not a device runtime or Core Web Vitals result.
- HTTP checks returned 200 for the five main routes, manifest, service worker, public offline page, all four PNG assets, and API health. Production HTML contains unique titles, Apple icons, safe-area viewport settings, and no zoom restriction. The dev server remains at port 3000; full worker behavior requires a production build and phone acceptance needs a reachable HTTPS origin. No credentials were changed. Live push still awaits the user's real public contact.

## Decisions and constraints

- API owns authentication verification, active-household authorization, money calculations, persistence, migrations, and integrations so clients cannot bypass household or financial rules. Web/mobile call the versioned API; they do not write household records to Supabase directly. Never treat a supplied household ID as proof of access.
- Database tables are in private `housemate` schema with RLS enabled. Trusted API uses `DATABASE_URL`; the browser uses only the Supabase project URL and publishable key. Never expose database URL, service-role key, or private provider credentials in clients or version control.
- Financial amounts are integer minor units to avoid rounding drift. A household chooses USD **or** THB, without conversion. Unpaid bills do not affect member balances; Paid bills and confirmed expenses do. Manual settlements report transfers outside HouseMate.
- Expense/bill creator or owner/admin may edit or void. Material edits and voids require fresh API-calculated previews, revision checks, idempotency keys, explicit confirmation, and audit history so corrections remain safe and traceable. A voided record remains visible, is excluded from balances/pending totals, and cannot be edited or have its bill status changed. Previously recorded settlements remain. **No hard-delete endpoint exists for confirmed financial records.**
- Any active member can create a one-off chore. Assignee or owner/admin can complete/skip; owner/admin can reassign; assignee can request a swap and target can accept/decline. Recurrence policy is deliberately unresolved in `SPEC.md`.
- Update `SPEC.md` for product behavior, `api/openapi.yaml` for API changes, relevant READMEs for setup/use. Keep `api`, `app`, and future `mobile` independently buildable and releasable.

## Current issues and unfinished work

- A duplicate React key warning in expense Change history was reported and fixed in the web client. `FinancialHistory` now deduplicates overlapping history pages, uses an event-kind-plus-ID key, tracks the raw API offset separately, and prevents concurrent Load more requests. A focused pagination merge test was added. Web lint, typecheck, 24 tests, and build passed. The exact signed-in account screen was not browser checked. Earlier bill-void date rendering and the generic Meals/Shopping error were fixed. The latter was caused by an API process started before new routes existed; `api npm run dev` builds **once** and must be restarted after API code changes. The web client now gives a targeted stale-API message.
- A duplicate React key warning in expense Change history was reported and fixed in the web client. `FinancialHistory` now deduplicates overlapping history pages, uses an event-kind-plus-ID key, tracks the raw API offset separately, and prevents concurrent Load more requests. A focused pagination merge test was added. After the Money redesign, web lint, typecheck, 24 tests, and build passed; `/money` returned HTTP 200. The exact signed-in account screen was not browser checked; computer-use tooling failed to initialize. Earlier bill-void date rendering and the generic Meals/Shopping error were fixed. The latter was caused by an API process started before new routes existed; `api npm run dev` builds **once** and must be restarted after API code changes. The web client now gives a targeted stale-API message.
- The one-off chore flow passed API/database checks but has not had a reported two-account browser acceptance check. Do not claim that manual check happened.
- Chore recurrence remains unchecked pending schedule/cutoff rules. Sections 10, 11, and P2 web quality implementation are complete; live push enablement/device acceptance awaits a real contact as described above. P2 actual-phone, rendered accessibility, and runtime-performance acceptance remains pending. Mobile and later AI/search/LINE work remain unchecked in `WEB_APP_TODO.md`.
- The user accepted the creator-or-owner/admin permission rule and authorized a mistaken-record removal UX for bills and expenses. The web now labels the existing audited Void action **Remove mistaken bill/expense**, requires a reason before its preview, and labels the unconfirmed form action **Discard draft**. No hard-delete endpoint or API contract change was made. The existing Void API still accepts an optional reason for compatibility; the web requires it. Web lint, typecheck, tests, and build passed, and `/money` returned HTTP 200 after the change. Authenticated browser acceptance was not run.

## Run and verify

Requires Node.js 22+, npm, and configured Supabase. Keep credentials in ignored `api/.env` and `app/.env.local`; use each `.env.example` for variable names. API expects `DATABASE_URL`, `SUPABASE_URL`, `SUPABASE_PUBLISHABLE_KEY`, and `WEB_ORIGINS=http://localhost:3000`; web expects `NEXT_PUBLIC_API_URL=http://localhost:3001`, `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY`, and optional `NEXT_PUBLIC_GOOGLE_AUTH_ENABLED=true`. Auth redirects allow `http://localhost:3000`. Never copy actual secrets into this file or tool output.

```powershell
cd C:\Users\ADMIN\Desktop\house_mate\api
npm ci                 # only if dependencies are missing
npm run dev            # port 3001; restart after API edits
npm run lint
npm run typecheck
npm test
npm run build
npm run smoke:chores   # configured DB; fixture rolls back
npm run smoke:events   # configured DB; fixture rolls back
# Other targeted checks: smoke:expenses, smoke:bills, smoke:settlements,
# smoke:edits, smoke:voids, smoke:dashboard, smoke:meals, smoke:shopping
```

```powershell
cd C:\Users\ADMIN\Desktop\house_mate\app
npm ci                 # only if dependencies are missing
npm run dev            # port 3000
npm run lint
npm run typecheck
npm test
npm run build
npm run measure:bundle # after production build; shell bytes, not phone runtime
```

All API/web lint, typecheck, tests, and builds passed after the events slice on 2026-09-28. The 2026-09-30 announcements/polls verification is recorded above. Re-run checks for affected code after new edits. Verify API process version with an endpoint request if web says a route is missing.

## API and schema map

- `GET /v1/households/{id}/dashboard` returns an authorized household snapshot. `/v1/me`, `/v1/households`, `/members`, and `/invites` cover identity/onboarding.
- Expenses and bills expose list/detail, create preview/confirmation, edit preview/confirmation, void preview/confirmation, and history. Bills also have provider-status changes. `/balances`, balance entries, and `/settlements` cover money explanations/transfers.
- `/meals`, `/shopping-items`, `/chores`, `/events`, `/announcements`, and `/polls` have their own nested actions; see `api/openapi.yaml` for exact paths, bodies, and errors. API uses bearer tokens from Supabase Auth.
- Migrations 001–002 create household/member foundation; 003–007 add financial records, settlements, edits, and voids; 008 meals; 009–010 shopping/currency; 011 chores and swap requests; 012 events and attendance; 013 announcements/polls; 014 notifications/preferences/push queue; 015 corrects the notification trigger. Notification/read/preferences endpoints are household scoped, while push configuration and devices are account scoped. Migration runner scripts check whether their change was already applied (015 is safe to reapply); normal API traffic must not run migrations.

## Approaches tried and abandoned

- An earlier running API process returned 404 for newly added Meals/Shopping routes even though source built; restarting the API fixed it. Do not mistake this for a database or auth failure.
- `npx supabase migration new` was attempted for chores but failed with Windows `EPERM` creating `C:\Users\ADMIN\.supabase`. The project continued its existing numbered `api/sql/011_chores.sql` plus runner convention; it was applied and smoke tested.
- Older root documents assume a web-only backend or early AI MVP. Follow current `SPEC.md`, `AGENTS.md`, and `api/openapi.yaml` instead.

## Exact next steps

1. If the user asks for more work on mistaken-record removal, start from the completed web flow described above. Keep the creator-or-owner/admin permission unless the user changes that decision. Avoid hard-deleting confirmed financial records or settlements merely to hide an error.
2. Sections 11 and P2 implementation are complete. Live push requires a real contact the user currently lacks; do not repeatedly ask or invent one. When provided, configure it and verify a real device. P2 manual acceptance should follow `app/QUALITY_CHECKS.md` on Android Chrome and iPhone Safari once an accessible production origin/devices are available. For other authorized work, continue with the next eligible unchecked `WEB_APP_TODO.md` item, respecting open decisions (especially chore recurrence).

## START HERE

### Mobile UI refinements — production 2026-10-04

The user requested deployment. Web main **833f07d1e8da4ff09e7cc89e280e51c5bde20034** is committed and pushed to heinzaw-bit/housemate-app. It includes the local fixes below: matching 44px mobile header mascot, a single scrolling Board filter row, removal of View balance/Calculate a split answer-footer actions, active-tab + New beside the Board title, in-form cancellation/focus, and Refresh beside list update status. No API, migration or environment change was required. API remains **51b716dd82b2aca8811cc78b0f2bc5c4463ea231**.

DreamyDay production deployment **dpl_J1ywBBipoNMVTRPqfTEpnCugjAZa** (housemate-njfbgx0l0-dreamy-day.vercel.app) reached READY for this exact web commit with https://housemate-app-xi.vercel.app assigned. Build duration was approximately 35 seconds. Existing authenticated Vercel CLI and Git integration were used. Both pre-publication local main branches matched origin/main; staged web credential scans and diff checks passed.

Verification: web lint, production build including TypeScript, 27 Board/communications/draft/accessibility tests and 21 Assistant tests passed. All ten deployed route/health/manifest/service-worker probes passed. GitHub CI **37193082186** passed on the exact deployed commit, including clean installation, lint, typecheck, full tests and build. Sanitized deployment log checks found zero error-level or 5xx entries in the checked 15-minute window. No production household record was written. Actual Android/iPhone visual acceptance for these layout fixes remains pending. Earlier local publication-status statements below are superseded by this entry. Whole-project main pins the published app revision and current shared documents.


### Board create actions repositioned — local 2026-10-04

The user approved moving creation beside the Board title. HouseWorkspace now owns one active-tab action: + New on phones (up to 650px), New announcement/New event on desktop, with full accessible labels. Compact announcement/event workspaces no longer render the old toolbar below section tabs. Refresh is beside Updated status via RecordRefreshStatus; events now track last successful list refresh. Noncompact legacy communication toolbars remain available. Opening/reopening a form focuses and scrolls to it with reduced-motion support, retaining existing input. Compact forms include Cancel with existing saved-draft clearing behavior. Announcement review/publication and event submission, API authorization and endpoints are unchanged. SPEC, web README and quality checklist updated.

Verification: lint, all 27 tests across Board/communications/saved-draft forms/form accessibility, and production build including TypeScript passed. Two new Board regressions verify active-tab creation, repeated New preserving and focusing input, Cancel without writes, and refresh reads without creation. React review checked stable state, effect dependencies, lazy loading, accessible actions and no extra writes. Actual Android/iPhone visual acceptance remains pending. Local/uncommitted on main in D:/house_mate; preserve the previous mascot sizing, filter wrapping and Assistant footer fixes below. No commit/push/deployment requested for this change.


### Assistant money footer links removed — local 2026-10-04

The user requested removing the View balance link and Calculate a split action shown after Assistant output. AnswerCard now filters balance-view links from the footer sources and no longer renders Calculate a split for balance/bill answers. This applies to latest and earlier answers. Other source links, individual record links, API-calculated amounts and checked timestamps remain. Removed the unused moneyCreateHref import, householdId presentation prop and nextAction CSS. No API/contract change. SPEC, AI plan, web README and quality checklist updated to remove stale descriptions of the split action.

Verification: web lint, all 21 existing Assistant tests and production build including typecheck passed. Existing balance-answer assertions now check both requested links are absent. Work remains local/uncommitted on main in D:/house_mate alongside the mobile mascot sizing and Android Board filter fixes below. No deployment requested for this change.

### Android Board filter wrapping — local 2026-10-04

The user reported Expired wrapping below the announcement filters on Android. Compact CommunicationBrowser now displays To acknowledge for needs_ack, with an accessible label beginning with that visible wording and retaining the full acknowledgement meaning. A dedicated board-record-filters flex row never wraps; min-width: 0 and max-width: 100% contain overflow, and horizontal scrolling keeps filters available at narrow widths/enlarged text without widening the page. Four-pixel inset/scroll padding leaves room for focus indication. Compact announcement controls have 20px top padding below the Board section tabs. Noncompact legacy communication filters retain their wrapping layout. Filter values, data reads, publication/acknowledgement flows and permissions are unchanged. SPEC/web README updated. Preserve the earlier uncommitted 44px mobile mascot fix.

Verification: lint passed and all seven existing Board/communications tests passed. A final production build started after the accessible-name adjustment passed, including typecheck. These are structural/automated checks; actual Android visual acceptance remains pending. Work is local and uncommitted on main in D:/house_mate; no push/deployment requested for this correction.

### Mobile header mascot sizing — local 2026-10-04

The user reported the Android topbar mascot was smaller than the adjacent icons. The existing mobile-brand brand-icon rule at widths up to 850px used a 32px tile while header controls use 44px. Increased that tile to 44px square and set flex-shrink: 0; the existing 84% SVG scales with it. Change is scoped to the mobile header, with no desktop/sidebar/auth/Assistant mascot or API change. SPEC and web README describe the matching size. Web lint and production build (including typecheck) passed; no new tests were added for this reversible styling change. Actual Android visual acceptance remains pending. This is uncommitted on main in D:/house_mate after the previous approved release; no push or deployment was requested for this correction.

### Latest Board, Assistant and reload release — production 2026-10-04

The user explicitly requested deploying the latest changes. API main **51b716dd82b2aca8811cc78b0f2bc5c4463ea231** and web main **cab40c38b8806b4155263745260cee2abfbf5fed** fast-forward the codex/house-announcements-events work into main and are pushed to their private heinzaw-bit repositories. This includes the previously published feature-branch Board announcement/event layout and name, question-focused Assistant, and all subsequent Assistant/general-reply/context/notice/draft-removal and reload-entry changes below. Weather is removed, including a stale provider-schema enum found and corrected during release review. Child .gitattributes enforce LF and normalization-only files were excluded from staged semantic changes. Staged credential scans passed. No migration or environment/secret transfer was needed.

API was deployed before web. DreamyDay API deployment **dpl_G2YKryUEnSB9bAceJwA9wVUjZj4H** (housemate-qim7fwrrx-dreamy-day.vercel.app) and web deployment **dpl_3HmJMiuMkiEusMWMedPbTpmpBjAd** (housemate-nvx07vp1v-dreamy-day.vercel.app) reached READY for these exact commits. Canonical URLs remain https://housemate-api.vercel.app and https://housemate-app-xi.vercel.app. The web deployment aliases include the canonical URL. Connected Vercel MCP tools returned a DreamyDay scope 403; the existing Vercel CLI login was verified as heinzaw-bit with access to both projects and used to verify Git-triggered production deployments. No access setting was bypassed or changed.

Verification: API lint/typecheck/build and all 83 tests passed; after the provider-schema correction, lint/build and all 21 Assistant tests passed again. Web lint/typecheck/build passed in the immediately preceding entry fix; the full release suite passed **206 tests in 32 files** with two workers and a 15-second timeout. The initial concurrent API/web run had three existing five-second web timeouts; no test implementation was changed to hide them. All ten canonical web route/health/manifest/worker checks passed, root HTML serves Opening HouseMate with the loading status, the worker matches tested source, API health and database readiness returned 200, unauthenticated identity returned 401 and production-origin preflight returned 204. No production household records were created. API GitHub CI **37188152922** passed; web GitHub CI **37188224576** also passed on the exact published commit. Signed-in/device reload, Assistant and Board acceptance still requires actual Android/iPhone checks; route probes do not establish those flows.

The earlier local entries below describe implementation detail and are superseded by this publication status. Whole-project release pins the published child commits and current shared product documentation. Continue from main; preserve private local configuration and ignored operator files on D:.

### Home reload and installed-app entry fixed — local 2026-10-04

The user approved retaining the landing for new browser visitors while fixing the Android reload flash. HomeEntry now waits for auth checking and its browser entry detection before showing any landing content, using the mascot and Opening HouseMate loading status. Restored sessions open the dashboard directly. Signed-out standalone launches use the existing AppView sign-in flow, detected by display-mode: standalone on Android and navigator.standalone on iPhone. Ordinary signed-out browser visitors still see the public landing after checks. Pending invites, household links and auth callbacks/errors preserve their app flow. The display-mode listener is cleaned up on unmount. No manifest, service-worker, auth-provider or API change is needed. The root entry now requires JavaScript to finish its session/launch check; SPEC and web README reflect that.

Verification: web lint, typecheck and production build passed. All 32 focused tests passed (11 entry-routing tests plus 21 Assistant tests), covering session restoration without landing flash, signed-out visitors, Android/iPhone standalone sign-in, invites/callbacks and sign-in routing. Phone reload acceptance is still pending deployment. No commit, push or deployment requested; keep all prior uncommitted Assistant work.

### Assistant privacy UI removed — local 2026-10-04

The user requested removing both the “How answers work · Privacy” accordion and the Gemini sharing/privacy text below the composer. Both are removed from AssistantWorkspace, including the unused state, aria-describedby and CSS. The current page keeps the mascot, question input, three shortcuts and answers. The existing API privacyAcknowledged request field is still supplied for compatibility on explicit submission; no acknowledgement UI remains. Provider handling and household access rules are unchanged. This entry supersedes older documentation describing a visible sharing notice. Updated SPEC, AI plan, READMEs and quality checklist to match the UI.

Verification: web lint, typecheck (without incremental writes), all 21 existing Assistant tests and production build passed. Updated existing assertions to check both requested sections are absent. No commit, push or deployment was requested; preserve the prior uncommitted Assistant work below.

### HouseMate Assistant consistency; weather removed — local 2026-10-04

Latest user steering removes weather first and implements the six Assistant recommendations. Work remains in D:/house_mate on codex/house-announcements-events; no commit/push/merge/deployment requested. This supersedes the weather portion of the expanded-Assistant entry below. Removed the untracked Open-Meteo module/tests and their obsolete generated outputs, all city/weather schema fields, weather answer type and UI references. Weather requests get a local explanation and household choices; no city prompt, location access or weather provider call remains.

The chat/mascot remains the main action. Three shortcuts are Catch me up, My balance and Board updates. API Board combines caller-visible active announcements and upcoming events; Catch me up prioritizes urgent/important active notices, upcoming events and unpaid bills due within seven days. Recent pages and record previews remain bounded, with honest partial-list notices and normal household/audience authorization. Old chore/shopping/meal topics/direct links remain compatible but are not promoted. Greetings/about and Gemini instructions use concise, friendly HouseMate vocabulary. Household answers have short summary/context, explicit record actions and Checked times; general answers render escaped paragraphs/simple bullets with a subtle AI label and Answered time. Provider failures/limits keep household shortcuts available and provide Open Board/Open Money links. No assistant mutations or action editors were reintroduced.

The API contract adds board topic, optional response choices (up to three) and optional request context containing only a previous allowlisted household topic. Exact local follow-ups such as Today/What about today after Board/events read fresh authorized events in the household timezone. Context cannot contain messages, answers, member/household IDs or financial values; property-name validation rejects extra context keys. It grants no access and never goes to Gemini. General questions remain independent. Client context is derived from the latest household answer and clears with answers/account/household/reload. Clarification choices issue a normal explicit read; after clarification the input is selected for easy replacement. Assistant draft saving/resume/discard remain disabled; other form drafts are unchanged.

Verification: API lint/typecheck/build and all **83 tests (70 TypeScript + 13 operational)** passed. Web lint/typecheck/build passed; **202 tests in 32 files** passed with four concurrent workers. The first default-worker web run had one five-second timeout in the existing bill-history test; that suite passed alone, then the full rerun passed without modifying Money or its tests. Assistant-focused checks cover removed weather, combined Board/summary boundaries, expired notices, topic-only follow-ups, invalid/foreign context, no private model context, clarification choices, plain-text bullets and clear/reset isolation. OpenAPI parsed with **445 resolved local references**. One live non-private shared-space advice question returned a valid general reply under the revised Gemini schema, with no city field; no household records or credentials were printed/sent in that check. No database migration or new environment variable needed. Deploy API before web when authorized; signed-in phone/rendered acceptance remains pending. next-env.d.ts has no semantic diff.

Updated SPEC, AI plan, root/API/web READMEs and web quality checklist for current behavior. Preserve earlier uncommitted Assistant changes and the clone line-ending normalization/.gitattributes described below.

### Assistant saved-question recovery removed — local 2026-10-04

The user requested removing save-draft/discard behavior from the Assistant tab only. AssistantWorkspace no longer imports/calls useSavedDraft or displays SavedDraftNotice, no longer blocks submission on older saved questions, and no longer writes question drafts to browser storage. Unsent questions and answers remain in tab memory; leaving/reloading resets them. Old saved Assistant questions/action seeds are ignored (normal expiry/sign-out cleanup still applies). Existing Cancel question and unsent-work leave/update guards remain. Shared draft utilities, expense/bill and other record forms retain their current recovery behavior. No API changes were needed for this request; preserve the previous uncommitted expanded-Assistant API work below.

Updated current SPEC, AI plan, web README/quality checklist and existing recovery tests. Web lint, typecheck, **43 affected tests across Assistant, saved-draft forms and shared saved-draft utilities**, and production build passed. Tests verify no question persistence/recovery controls, ignored old action seeds and working expense/bill recovery. This remains local in D:/house_mate on codex/house-announcements-events, alongside the expanded read-only Assistant. No commit, push or deployment requested. Generated next-env.d.ts has no semantic diff.

### Expanded read-only Assistant — local 2026-10-04

The user requested ordinary greetings, app identity/help, weather and announcements instead of only fixed-topic replies, and chose **Ask for a city** over device location. Implemented in **D:/house_mate** on the existing **codex/house-announcements-events** root/API/app branches. The user reports deleting the original C: checkout; use D: explicitly for tools and local services. No commit, push, merge or deployment was requested for this update.

Local greetings/thanks, HouseMate explanations and common announcement questions need no Gemini configuration/quota. Announcement answers call the existing authorized repository (same targeted visibility rules), omit expired notices, show up to ten active records from a bounded recent page, disclose limits and preserve exact Board deep links. General non-sensitive questions now receive bounded Gemini replies rather than only classifications; a visible AI-answer label distinguishes these from authoritative household records. Gemini receives no household records/member lists/balances/prior replies. Existing private-input checks, explicit submission acknowledgement, one-request/no-retry behavior and server-only free-tier key configuration remain. Questions stay independent; no autonomous writes, draft editors or conversation context were added.

Current weather uses an explicitly typed city/country through fixed Open-Meteo geocoding/current-condition endpoints. Missing/ambiguous cities need a new complete question with a clearer city/country/region. No device location, forecast or severe-weather alerts. Results show modeled conditions, city-local data time and Open-Meteo/GeoNames attribution. Eight-second total weather deadline, bounded ten-minute public-city cache/coalescing and the existing private usage budget limit abuse. City weather spends one budget slot, including if Gemini classified it first; greetings/app explanations/local announcement reads spend none. Gemini output cap is now 800 tokens (12-second deadline unchanged). Availability adds chatEnabled; response topic enums add announcements/greeting/about/general/weather before the client relies on them. No new migration/key needed; existing migration 016 remains required. Runtime always wires its budget repository, including when Gemini is absent. Deploy API before web when publication is authorized.

Verification: API lint/typecheck/build and all **85 tests (72 TypeScript + 13 operational)** passed. Web lint/typecheck/build and all **199 tests in 32 files** passed. OpenAPI YAML parsed and **445 local references** resolved. A single benign live Gemini question about dry-rice storage returned a valid general reply; a live public **Bangkok, Thailand** weather check returned current attributed conditions. No household data was sent in those provider checks, no credentials printed and no database records changed. Automated checks cover access/revocation, provider parsing/privacy/failures, announcement scopes, weather ambiguity/cache/coalescing/stale data, new UI reply types and local-only availability. Signed-in browser/phone acceptance remains pending; these checks are not an end-to-end phone claim. Restart any old local API process before testing new endpoints/answers.

The fresh Windows clone used CRLF where Biome expects LF. Normalized source/test working files without semantic changes and added child .gitattributes enforcing LF for future checkouts. Git diff confirms semantic changes are limited to Assistant source/tests/contract and documentation, plus new weather module/tests; unchanged line-ending files may appear in status until Git refreshes attributes. Existing generated next-env.d.ts has no diff. SPEC, AI plan, API/web/root READMEs and web quality checklist reflect current behavior.

### GitHub publication and D: checkout — 2026-10-04

The user requested publishing the current project to GitHub and cloning it onto D:. The verified fresh checkout is **`D:/house_mate`**, including independent API/app submodules on **`codex/house-announcements-events`**. Web commit **7054c99aba57cf4a9b180cbc7e2c42b8a0b2dff4** contains the tested Board/question-only Assistant changes; API commit **ea7d59e822ce2eb557519a45fa01fd723b003b7a** records previously verified operational guidance. Initial whole-project publication **977c6677a4e1c7fc6cf3c18f37caa87fc74371f4** pins both. All branches were pushed to the three private heinzaw-bit repositories; credential scans passed. GitHub CI runs **37180074722** (web) and **37180078322** (API) succeeded on these exact revisions. Main and production were not merged or deployed by this migration.

Private environment files, TLS certificate, backup recovery key, local provider metadata, current encrypted archives, attachments and local operator helpers were copied directly between local folders, outside Git. Eight private configuration/key files have inheritance disabled and explicit access for the user's Windows identity and SYSTEM. All 1,681 copied files (about 155 MB) passed SHA-256 comparison; no configuration referenced the old checkout path. An older unreadable encrypted archive, `api/.run/backups/housemate-2026-09-30T17-46-40-087Z.hmbak`, remains only in the retained C: original. Do not delete that original without addressing its remaining archive and any other desired local history. Restore-drill databases, dependencies, build output and optional pgAdmin/StackBuilder packages were not migrated. Required PostgreSQL bin/lib/share tooling was verified with pg_dump 17.11.

Fresh `npm ci` and production builds passed in both projects on D:. All three new repositories were clean at verification. The read-only scheduler status command from `D:/house_mate/api` reported enabled, valid configuration and a fresh successful execution; the existing reminder-monitor automation now uses this directory, retaining its schedule and notification policy. The old C: checkout remains intact. This chat still belongs to its original C: workspace; open/add **D:/house_mate** as the Codex project before continuing development there. Previously dated C: paths below are historical references, not the new working directory.

### Question-only Assistant with loading inside input — local 2026-10-04

Latest user steering removes draft commands from Assistant and places the loading animation inside its chat/search input, with only answers below. AssistantWorkspace now calls only the question endpoint, ignores legacy action availability, and no longer imports preparation/editor/recovery APIs. Ask/Enter remains one step with a visible Gemini sharing notice and no consent checkbox. The mascot/dots/skeleton overlay replaces the visible textarea while busy but preserves its mounted value; Cancel/error/clarification retain text and restore focus. Reduced motion remains supported. Device recovery restores only unsent questions after explicit Resume; obsolete action seeds cannot reopen editors. Backend contracts/editor components and normal Board/Money creation remain intact for compatibility.

This is local on codex/house-announcements-events, alongside the Board changes below. No commit, push or deployment was requested for this update. Preserve unrelated local changes and generated next-env.d.ts. Web lint, typecheck, all **193 tests in 32 files**, and production build passed. Tests cover loading inside the input, answers outside, cancellation/late reads, no draft preparation or writes, explicit question recovery and rejection of obsolete action seeds. Retained communication editors still have direct confirmation/retry tests, alongside existing financial editor coverage. Generated next-env.d.ts was restored to its pre-build content. Actual signed-in Android/iPhone rendering remains pending.


### Assistant loading and one-step questions — local 2026-10-04

Verification: lint (123 files), typecheck, all **194 tests in 31 files** and production build passed. The Assistant/draft subset passed 37 tests, including visible-notice submission, loading/cancellation, Shift+Enter/Enter, unchanged decimals/context, retained clarification input and household/account isolation. The first full run exposed a one-second lazy-import wait in the Board test; its initial wait was adjusted to five seconds and the full rerun passed. Generated next-env.d.ts was restored to its prior local state. Actual phone animation/keyboard acceptance is still pending.

On the existing **codex/house-announcements-events** root/app branch, the user requested loading animation, removal of the consent control and easier asking/readable answers. The separate checkbox is removed. A visible Gemini sharing notice below the composer explains free-tier use/review and omitting private input; explicit Ask/Enter acknowledges it using the unchanged API field. Typing/restoring never sends. Ask/Enter submits, Shift + Enter adds a newline, and Ctrl/Command + Enter remains supported. The loading card shows a gently moving mascot, dots and placeholder lines, with an accessible status/Cancel and static reduced-motion behavior. Answers add topic headings and separate the exact first API sentence from context, preserving money decimals/newlines and escaped text. Failures/cancellation/clarification keep input; completed requests restore focus. Existing action editing/review/confirmation, idempotency, private-input guards, provider budgets and scoped late-response rejection remain unchanged. No API/provider/secret change, commit, push or deployment is included in this local request.


### House simplification on a separate branch — 2026-10-04

The user subsequently chose **Board** as the tab name. Navigation (desktop/mobile), the page heading, accessible section label, browser title and landing tour now use Board. The `/house` URL and internal section identifiers remain compatible with existing links. This rename remains local on the same feature branch.

Board now matches Money's shared page-header styling, with eyebrow “SHARED UPDATES, PLANS TOGETHER” and description “Share announcements, plan events, and see who’s joining.”

The user requested the preceding recommendation on a new branch. Root and app are on **codex/house-announcements-events**, created from the deployed main release; main and production have not been changed by this work. House now defaults to Announcements with only Announcements/Events navigation, no Your next actions panel or shared Add menu, one create action per tab, compact announcement previews and mobile event Back navigation. Home drops next chore/meal and household-work cards. Assistant default copy/shortcuts no longer promote those tools. Search groups old filters under Earlier household tools; notification settings reveals their switches on request while preserving all stored preferences. Existing direct links, records, API functionality and saved drafts remain compatible. Loaded announcement pages prioritise active urgent/important notices locally; no API sorting contract change or migration was made.

Changes are local for review; no commit, push or deployment was requested for this branch. Preserve unrelated generated web files, line-ending-only changes, API operational notes and attachments.

Verification: web lint (123 files), typecheck, all **193 tests in 31 files** and production build passed. After the final event deep-link preservation adjustment, the affected House/accessibility suite was rerun (**19 tests passed**) and lint/build passed again. The new House integration tests cover default navigation, loaded notice ordering, mobile detail/back, attendance, announcement/older chore links and past events outside the first page. Main remains at deployed app e1ce531; no API changes or shared migrations. Actual signed-in phone layout acceptance is pending.


### Saved drafts deployed; bill templates canceled — 2026-10-04

The user canceled bill templates and authorized committing saved drafts on the current branch, syncing origin/main, merging into main, pushing and deploying. Only the uncommitted template source, web controls, tests, contract and unapplied migration were removed; no shared database migration was applied. Existing local operational notes, generated web files and attachments were preserved. API source and production API are unchanged.

App **e1ce5315e3a18c9ff91f8d7dd61ce07bf5bbd622** was committed on codex/money-focused-records, merged with the fetched origin/main, fast-forwarded into main and pushed. The whole-project feature commit **5513abd** pins that release and was likewise merged/pushed to main. Root and app are now on main; the feature branches are retained. No unrelated API documentation or generated web files were committed.

Local lint/typecheck, **188 tests in 30 files** and production build passed. Staged credential checks passed. GitHub clean-install CI [37174173286](https://github.com/heinzaw-bit/housemate-app/actions/runs/37174173286) completed successfully for that exact web SHA. DreamyDay housemate-app production is **READY** at https://housemate-dmtfxce1n-dreamy-day.vercel.app; the canonical https://housemate-app-xi.vercel.app origin and all ten main route/PWA checks returned HTTP 200. Vercel's status on the exact commit also reports deployment completed. The MCP connector still reports missing dreamy-day scope; the existing signed-in CLI verifies deployment.

Drafts now recover on the same device after an explicit Resume choice. Seven-day account/household/context isolation, sign-out clearing, original interrupted-confirmation retry keys and fresh financial previews are implemented. Browser data clearing removes them; they do not sync across devices. Actual Android/iPhone refresh/full restart/offline-reconnection/interrupted confirmation/app-update acceptance remains pending. Do not implement or deploy bill templates unless the user requests them again.


### Automatically saved drafts — publication requested 2026-10-04

The user requested device-local automatic draft recovery. The user authorized committing saved drafts on codex/money-focused-records, merging origin/main and deploying on 2026-10-04. Web commit e1ce5315e3a18c9ff91f8d7dd61ce07bf5bbd622 is committed on that branch, merged into main and pushed to heinzaw-bit/housemate-app. The user canceled the uncommitted bill-template feature; its source, contract, migration and tests were removed without applying a shared database migration. The production release is READY at app e1ce531; the preceding production release was app 20aeed3. No API source, migration, contract, provider configuration or secret changed.

`app/lib/saved-drafts.ts`, `assistant-draft-storage.ts` and `components/saved-draft.tsx` implement versioned, bounded, seven-day browser-storage slots scoped to account, household and currency/timezone context. Workspace supplies scope only after authorized identity/membership reads; AuthProvider clears saved drafts on sign-out/account changes, and membership pruning removes departed households. New expense, bill, meal, shopping, chore, event, announcement and poll creation forms save their explicit editable fields and offer Resume/Discard. Assistant stores unsent input and a compact action seed; resuming retrieves current members and then restores edited action fields after that explicit choice. Tokens, API preview totals, API member/record snapshots, answers and Gemini consent are not persisted.

Before a create request, the slot checkpoints original fields plus the exact payload signature and idempotency key. An unresolved attempt keeps its original fields despite later typing and rejects a changed confirmation; an identical manually reviewed retry uses the original key across refresh. Definitive first-request rejection permits editing. Resumed financial drafts need a fresh API preview. Nothing writes automatically. Successful/cancelled drafts clear without being recreated on cleanup; storage failures are visible, invalid/expired contexts are discarded, and changed cross-tab entries are not silently overwritten. Local storage is not encrypted by HouseMate or synchronized between devices. Existing-record edits, purchases, transfers/removals, settings and votes retain their existing in-memory guards.

Verification: web lint/typecheck, all **188 tests in 30 files**, and production build passed. Added recovery tests cover explicit resume/no automatic writes, cancel/success cleanup, interrupted-confirmation fields/key retention, malformed/expired/wrong-context data, account/household isolation, blocked storage, pagehide and competing tabs, plus real Expense/Bill/Assistant form integration. SPEC, README, QUALITY_CHECKS, checklist and Assistant documentation are reconciled. Actual Android/iPhone refresh/full restart/reconnection/update acceptance remains pending. Existing generated `next-env.d.ts`, the content-unchanged member-settings test, API documentation and attachments are preserved outside this implementation.

Next: perform the pending phone acceptance checklist. Do not redo the already deployed Money controls or unrelated operational work.


### Inline Money controls merged and deployed — 2026-10-03 (latest request)

The user authorized merging the current branch into main and deploying it.
App `codex/money-focused-records` was fast-forwarded into main at
`20aeed3aeb40e16add08f9fc605f3d2b67cab1a4` and pushed to
`heinzaw-bit/housemate-app`. The matching whole-project branch was merged into
main at `1012536` and pushed to `heinzaw-bit/housemate`, preserving the app
submodule pointer and shared SPEC. No API code or configuration changed.

Web CI [37127947004](https://github.com/heinzaw-bit/housemate-app/actions/runs/37127947004)
passed clean install, lint, typecheck, all 164 tests in 28 files and production
build. Vercel production deployment `dpl_3Kg3rePeFT6TJBEEBdV3YmCkLsAz` reached
READY at `https://housemate-976df7hc7-dreamy-day.vercel.app`, with the canonical
`https://housemate-app-xi.vercel.app` alias attached. Home, Assistant, House,
Money, Profile, Notifications, sign-in, manifest, service worker and offline
page each returned HTTP 200. The Vercel connector reported missing DreamyDay
scope authorization; the existing signed-in CLI verified the deployment.
The exact deployment's preceding 15-minute log checks returned zero error-level
and 5xx entries; this is a bounded snapshot, not ongoing monitoring.

The corrected inline text controls and cleaner household selector are now
published; the superseded separate-panel design below is not the final UI.
Both working repos are on main; the feature branches are retained. Preserve
unrelated generated web files, API documentation and attachments. Actual
Android/iPhone visual acceptance remains pending; CI and HTTP checks do not
establish rendered device acceptance. Continue the remaining product and
operational checklist only when requested; draft recovery is a recommendation,
not implemented by this release.

### Money views corrected to inline text toggles — 2026-10-03

The user rejected separate View all views and asked for expansion inside the
components using labelled buttons instead of arrows. On the existing local
`codex/money-focused-records` branch, the original desktop list/detail cards and
mobile Back behavior are restored. Bill/expense history previews three records
and expands inline with View all bills/expenses, then Show less. Details use
Hide details/View details and remain mounted to preserve confirmation drafts;
selecting a record opens the detail. Why this balance and Recorded transfers
retain three-entry previews; Suggested transfers retains two. View all
sources/transfers/suggestions and Show less independently expand each card
inline, with pagination available while expanded and no chevron arrows.
The new side panels, separate full-list dialogs and Details/Activity tabs are
removed. Financial history keys are prefixed to avoid sibling collisions with
removal drafts. The household-selector cleanup remains on this branch.

SPEC and app README are reconciled with this correction. Web lint, typecheck,
all **164 tests in 28 files**, and production build passed. Seven branch flow
tests now verify inline keyboard controls, no dialogs, selection/detail state,
draft retention, pagination failure/retry and independent balance expansion.
This supersedes the focused-view design described below. Main and production
remain unchanged; no push, merge or deployment is authorized by this request.
Actual phone visual acceptance remains pending. Preserve unrelated generated
files, API documentation and attachments outside the branch commits.

The corrected app is saved locally in branch commit
`20aeed3aeb40e16add08f9fc605f3d2b67cab1a4`; the matching root branch records
that revision. This supersedes app branch commit `17c2e7b` below. Neither branch
has been pushed, and the deployed app still uses main `9d9d5cd`.

### Focused Money views — feature branch 2026-10-03

The user requested a new branch to implement the compact-preview recommendation.
Root and app are on `codex/money-focused-records`; API remains on main and has no
new source/contract changes. Bill/expense history shows three rows in existing
API date order, with View all opening the full paginated list. Clicking a record
opens a focus-trapped desktop side panel or mobile full-screen view; Back returns
to the originating list. Total, status and shares stay visible, while Details
contains metadata/adjustments and Activity lazily loads financial change history.
Edit closes the panel and focuses the existing preview form. Removal drafts
block Escape/outside/close and preserve their input; in-flight actions also block
closing. Open forms disable View all and prevent selecting other records.

Balances now uses a short Why this balance explanation and View breakdown for
the selected member's paginated sources (with failure retry), two suggestions
plus View all suggestions, and three compact recorded transfers plus View all
transfers with audit details and pagination. Existing API previews, confirmation
retry keys, role permissions and money calculations are unchanged. The prior
household-selector cleanup is retained on this branch. SPEC and app README are
updated. Web lint, typecheck, all **166 tests in 28 files**, and production build
passed. Nine new flow tests cover full-list navigation, page failure/retry,
records beyond the first 50, direct links, lazy activity, detail structural axe
checks, edit focus, draft-preserving Escape/cancel and balance focused views.
Rendered Android/iPhone acceptance is still pending; browser tooling has been
blocked previously. This branch is local; main and production are unchanged.
Unrelated generated files, API documentation and attachments remain outside
the feature commits. Do not push, merge or deploy this branch without the user's
next instruction.

App implementation is saved in local branch commit
`17c2e7bd616ba32f839166a39062d89d3d03d560`. App main remains
`9d9d5cd8c79e3df0b7f359e9768c7521772fd975`. The matching root branch records
this app revision and shared documents. No remote branch or deployment was
created. Switch back to the feature branch for local review before modifying it.

### Minimal household selector — local 2026-10-03

The user requested a cleaner household selector. The shared desktop/mobile
trigger now uses a single-line household name, 32px green initial badge and
16px chevron in a transparent 56px-minimum row, with themed hover/open feedback.
The uppercase caption, filled outer card and heavy type are removed. Focus,
disabled state, long-name ellipsis and reduced-motion behavior remain supported.
Menu/bottom-sheet actions and draft guards are unchanged. SPEC and app README
are updated. Web lint, typecheck, CSS formatting and all six existing household
menu tests passed. This change is local, uncommitted and not deployed; actual
phone visual acceptance remains pending.

### Money cards and profile avatar release — 2026-10-03

The user authorized committing and deploying the four local UI changes below:
theme-aware green profile avatar, removal of the Money currency badge, and
independent bill/expense history/detail dropdowns. App main is
`9d9d5cd8c79e3df0b7f359e9768c7521772fd975`. Local web lint, typecheck, all
**157 tests in 27 files**, and production build passed. Generated local files,
unrelated API documentation and attachments are excluded. No API release is
required. The earlier local status in the implementation notes below describes
the state before this deployment request; follow this release entry for the
current publication state. Phone visual acceptance remains pending.

Vercel production `dpl_9F37h7K7RgxSNKnAkzmd19FpBxbX` is READY and aliased to
`https://housemate-app-xi.vercel.app`. GitHub CI **37121479086** passed, including
clean installation, lint, typecheck, tests and production build. All ten
canonical web route/PWA HTTP checks returned 200. The user should finish open
drafts and accept the app update/reload to see the changes on their phone.

### Collapsible expense cards — local 2026-10-03

The user requested the same dropdown implementation for Expense history and
Expense details. Both now match the bill card heading buttons: independent
expand/collapse, initially expanded, chevron and accessible state/control links,
44px-minimum targets, and top-aligned desktop cards. Selecting an expense or
following a direct record link opens its details automatically. Hidden bodies
remain mounted to preserve loaded state and confirmation input; mobile Back
remains visible. SPEC and app README are updated. Web lint, typecheck and 16
existing financial-history/form quality tests passed. This is local, uncommitted
and not deployed; the bill toggles, avatar color and badge removal below are
also still local. Phone visual acceptance remains pending.

### Collapsible bill cards — local 2026-10-03

The user requested dropdown buttons in Bill history and Bill detail. Each card
now has an independent 44px-minimum keyboard-accessible heading button with a
chevron, `aria-expanded` and `aria-controls`. Both start expanded. Opening a
bill, including a direct record link, expands its detail. Collapsing hides the
body without unmounting it, preserving pagination, record and confirmation
state; desktop cards align at the top and the mobile Back control stays visible.
SPEC and app README are updated. Web lint, typecheck and 15 existing bill/form
quality tests passed. Actual-device visual acceptance remains pending. This is
local, uncommitted and not deployed, alongside the two UI changes below.

### Money header currency badge removed — local 2026-10-03

The user asked to remove the THB HOUSEHOLD badge. It is removed from the live
Money header and the design-preview Money header, including unused Badge imports.
Amounts still show currency. SPEC is updated; web lint and typecheck passed.
This is local and not deployed,
alongside the profile avatar color change below.

### Profile avatar theme color — local 2026-10-03

The user asked to replace the peach profile initial avatar with HouseMate's
theme colors. `app/components/member-settings.module.css` now uses the shared
`--accent` and `--accent-foreground` tokens: soft green/dark green in light mode,
and deep green/mint in dark mode. SPEC is updated. The affected CSS passes Biome.
This small presentation change is local, uncommitted and not deployed.

### Header/search and Android header fix — published 2026-10-03

The user authorized deployment. App main is now
`20f09817c798d8d8625d0d0bbe3cdacc6c0476af`, including matching header controls,
compact household search, the single search focus outline, search beside the
breadcrumb/mascot, plain notification/theme buttons, and the Android wrapping
fix below. Vercel production `dpl_5SrR4Kz4RM14WE37bzHFNijAa9U2` is READY and
serves `https://housemate-app-xi.vercel.app`. All ten canonical web route/PWA
HTTP checks returned 200. Local lint, production build (including TypeScript),
and all **157 tests in 27 files** passed. GitHub CI **37114836743** also passed
clean installation, lint, typecheck, tests and production build.
The discarded Money redesign is not
included; API code and deployment are unchanged. Only the app code, shared
product documentation and app submodule pointer are included in publication;
unrelated local API docs, generated files and attachments are preserved.

The user reported the Android logo and header controls wrapping onto two rows.
The old mobile flex-wrap rule is removed. Up to 900px the header now uses two
grid columns, with logo/search at the left and notification/theme/account at
the right. Up to 650px only the mascot is shown; all controls retain 44px touch
targets, 8px spacing, theme styling and safe-area gutters. This extends the
earlier local header/search work and preserves the restored Money layout.
Android visual acceptance remains separate from lint/build checks. Ask the user
to finish any open drafts, accept the app update/reload, and check the header on
the affected phone; do not claim an actual-device visual test was performed.

### Money redesign discarded — 2026-10-03

The user rejected the minimal Money redesign and requested deleting its branch.
Money components, API scope/filter changes, tests and documentation were reverted.
The earlier header/search changes and API monitoring documentation were preserved.
Root, app and API return to main; `codex/minimal-money-tabs` is removed locally.
The feature was never committed, pushed, merged or deployed.
Money-only tracked files match main again and feature-only components/tests were
removed. Web lint and production build, and API build passed after restoration.
Generated `next-env.d.ts` was restored to its prior local state.

### Search outline and header refinement — 2026-10-03 (latest local work)

The user requested removal of the layered search outline and no circles on
Notifications or Theme. Their subsequent screenshot clarified Search should
sit beside the left page breadcrumb, rather than at the far right. A shared
left group places Search after the breadcrumb on desktop and after the brand
on phones, while date/Notifications/Theme/Account stay on the right.
The search input now suppresses its
own focus decoration while its wrapper provides a single contiguous outline.
Header DOM/tab order is Search, Notifications, Theme, Account. Bell and theme
controls (including preview) have transparent backgrounds and no border, with
44px touch targets, hover feedback and keyboard focus indication retained.
These edits refine the two local updates below. Lint, typecheck and 22 relevant
search, notification-popup, account-menu and theme tests passed.
The subsequent Search placement adjustment passed affected-file lint and
typecheck; it did not change search behavior. Phone visual
acceptance remains pending. No commit or deployment was requested. Preserve
unrelated generated/test/API docs.

### Compact search panel — 2026-10-03 (latest local work)

The user approved the minimal search recommendation. The panel now has a Search
heading and small household label, prominent focused input, compact All records
selector and a one-character-only minimum-length hint. Initial empty search
omits the large illustration and session-storage/sorting explanation. Results
have category icons, titles, status, one-line previews and subtle dates at the
right (dates wrap below on narrow screens). The whole result remains a link.
Desktop height follows contents up to a cap. Phones use a safe-area-aware bottom
sheet with non-scrolling controls above results; visualViewport resize/scroll
updates its available height and keyboard offset while open.

Changed `app/components/household-search.tsx`, its CSS module and an existing
search test (conditional minimum-length help and clear-search focus/row reset).
The API search contract, debounce/IME, scope checks, request cancellation,
pagination, draft navigation protection and query privacy remain intact.
Lint/typecheck, all 157 tests and the production build passed. Actual
phone keyboard/visual acceptance remains pending. This and the preceding header
control update are local and not committed/deployed. Preserve pre-existing
generated/test changes and the API monitoring documentation edits.

### Matching header controls — 2026-10-03 (latest local work)

The user approved matching Search, Notifications, Theme and Account icons and
the phone spacing fix. All four now share `header-icon-button` styling: 44px
circles, 20px/2px Lucide outlines, theme-aware soft green backgrounds, 8px gaps,
focus rings and labels/tooltips. Search drops its visible text; Account replaces
the initial with UserRound and retains the initial/name/email in its menu. The
notification badge sits inside the corner. Preview shows a disabled matching
bell instead of an inert unlabelled span. Shared styles prevent shrinking and
keep 16px/safe-area phone gutters; up to 420px the mobile wordmark is hidden while
the existing mascot stays visible so all four controls fit on one row.

Changes are confined to web components/styles and setup/product documentation.
Search shortcuts/dialog behavior, theme persistence, inbox links/read counts,
account switching and sign-out are unchanged. Web lint, typecheck, all 157 tests
and production build passed. Real Android visual acceptance remains pending.
Not committed or deployed.
Preserve pre-existing `next-env.d.ts`, `tests/member-settings.test.tsx` and the
API monitoring doc edits.

### Minimal notification layout — 2026-10-03 (latest implementation)

The user approved the cleaner Notifications recommendation. Local web changes
replace large inbox cards with compact source-link rows, category icons, unread
dots, Important/Urgent badges and Today/Yesterday/Earlier grouping in the
household timezone. Inbox keeps All/Unread, bounded Mark all read, separate
accessible individual-read buttons and older cursor pages. The header has one
unread count and an icon refresh action; empty states reuse the existing mascot.

Settings now has Device notifications, Choose updates and Quiet hours. Device
management and the nine Inbox/Push category switches use native collapsible
details. Quiet-hour times appear only when enabled. The current device offers
Enable notifications or Turn off based on account-scoped local identity, the
server's enabled state and browser permission. Unknown/blocked setup remains
explicit. Household preferences still require Save changes; device refresh
preserves pending edits, leaving Settings asks before discarding them, and the
Inbox switch is disabled while saving/changing push devices. Theme variables,
phone layout rules, keyboard focus and reduced-motion styles are included.

Changed web files: `components/notification-workspace.tsx`, new
`components/notification-inbox.tsx`, `app/globals.css`, README and tests. The API,
OpenAPI contract, push sender/scheduler and service worker were not changed.
Five new tests cover timezone grouping/read/filter/pagination actions, explicit
saves/refresh preservation, leaving dirty settings/device disable, permission
and setup failures, and structural axe checks. All 157 web tests passed; lint
and typecheck passed, and the production build succeeded. Actual phone
visual acceptance remains pending. The user subsequently requested deployment.
Web main `3992528c4c9965bc9b13fec4c0cb31e62736fbd2` was committed/pushed after
staged credential checks passed. Clean GitHub CI `37095562648` succeeded; Vercel
production `housemate-ey91m3sre-dreamy-day.vercel.app` reached READY for that exact
commit. Canonical web Notifications, main routes, manifest, worker and offline
page returned HTTP 200. The production site is `https://housemate-app-xi.vercel.app`.
Preserve the pre-existing `next-env.d.ts` and `tests/member-settings.test.tsx`
changes, plus the previous uncommitted monitoring documentation.

### Monitoring receipt and iPhone timing — 2026-10-03 (latest report)

The user enabled GitHub Actions failure emails and confirmed checking the
cron-job.org failure/recovery test messages. Their attachment shows the isolated
test's HTTP 404 failure email for 08:10:36 Bangkok; recovery receipt is
user-reported. GitHub failure-email receipt itself has not been exercised.
The separate backup-key copy is still pending; the original file is
`api/.run/keys/backup.key` (32 bytes, existence/size checked without printing it).
Guide copying the exact file into a password manager encrypted attachment or
encrypted removable storage; do not regenerate, print or expose it. The user's
choice of protected destination is pending.

Android push continues to work. The user tested a friend's iPhone installed
through Safari and reports both the system notification and in-app entry arrived
after about 30 seconds. They initially associated arrival with reopening the app;
the closed-app test and notification-tap result remain unconfirmed. Focus,
Scheduled Summary and quiet-hours settings are not recorded. Read-only aggregate
production checks found two Apple deliveries accepted in about 45–48 seconds,
with no Apple delivery errors or outstanding queue entries. The scheduler is
enabled, configured correctly and healthy every minute (12:14:01 Bangkok,
533 ms). A 30-second delay is consistent with that schedule, rather than proof
of a platform bug. Provider acceptance does not establish display timing.
No push code/config changes were made. Ask for a closed-app 90-second test with
Focus/Scheduled Summary/quiet hours off; do not mark iPhone background/tap
acceptance complete until confirmed.

### Sentry owner alerts — 2026-10-03 (latest completed work)

The user saved `SENTRY_AUTH_TOKEN` in ignored `api/.env.monitoring` and explicitly
approved production alerts/test emails to the single active Sentry owner. The
US organization is `dreamyday`; existing project alerts were updated in place:
API workflow `6093012` / issue-stream detector `10494760`, web workflow `6093036`
/ detector `10494807`. Both are enabled, production-only, trigger on new issues
or regressions, filter error/fatal levels and email that owner directly. The
repeat notification interval is 30 minutes per issue. Readback verified the
configuration. Unrelated PR and unattached default rules were preserved.

Two synthetic non-private errors went through the actual API/web sanitizers;
both were accepted with HTTP 200 at 09:55:31–32 Bangkok. Sentry's `lastTriggered`
advanced for web at 09:55:54 and API at 09:55:56. The user confirmed both API
and web test emails arrived in their Sentry owner inbox. Test event IDs are API
`5e0da6ba584c453cba598a31458126a9`, web `10ba267ebda44fb290c1ceb3d5dec4f2`.
This verifies ingestion, alert triggering and owner email receipt. Actual browser
crashes and the resolve/recur regression path remain untested. The operator token remains
local, is not a runtime variable and was never printed or committed. No app
deployment, household writes or public crash endpoint were needed.

**Next:** resolve the two synthetic test issues in Sentry after reviewing them;
exercise regression alerts separately if desired. iPhone push acceptance,
the first scheduled backup, GitHub failure email receipt and separate recovery-key
preservation remain pending. Uptime email receipt and GitHub email settings were
subsequently confirmed above. Do not redo
the completed backup activation or resend these successful Sentry tests.

### Offsite backup activation — 2026-10-03

The user explicitly approved private backup setup/activation and confirmed only
Android has been push-tested; iPhone remains pending. Private DreamyDay store
`housemate-backups` is now connected only to API Development with prefix
`BACKUP_OFFSITE`; Vercel generated `BACKUP_OFFSITE_READ_WRITE_TOKEN` (not a
`_BLOB_` name). The connection POST returned empty success; metadata was checked
before continuing, and the connection write was not repeated. Its token, the
existing production database URL and retained recovery key were provisioned as
the three prepared private API GitHub Actions secrets. The verified activation
flag is true. No secret was printed, committed, stored in Blob or web variables.

Initial cloud run 37089252729 uploaded a verified private archive at 09:17:41
Bangkok, but its Linux restore failed. That downloaded archive restored locally
at 09:22:13 with all 66 tables/checks and recovery SQL passing. API main
`5d28806021422976fa15b08ae0062791bb677e26` disables system Unix sockets for the
isolated loopback cluster and adds static restore-stage diagnostics. It passed
lint/typecheck, 75 tests/build and clean CI 37089625330. Its Vercel deployment
`housemate-ckfzqc5ru-dreamy-day.vercel.app` reached READY; public health/readiness,
protected-route rejection and CORS passed.

Cloud rerun 37089625945 succeeded: captured 09:23:53, full downloaded-archive
restore passed 09:25:11 (66 tables, exact counts/digests, Auth references, RLS,
financial/history checks and recovery SQL). Private inventory is fresh with one
498,706-byte encrypted archive. Daily around 03:17 Bangkok is active; weekly
Saturday/manual restore checks are active. The first scheduled run, GitHub
failure email receipt, separate key preservation and managed Supabase/Auth
cutover remain unverified. No Storage object bytes exist/are backed up.

Sentry token access, alert configuration and owner test email receipt were
subsequently completed above.
The manual operator checklist in `api/OPERATIONS.md` explains GitHub failure
notifications, separate recovery-key storage, cron-job.org test emails around
08:10–08:13 Bangkok, Sentry setup and iPhone acceptance. Do not re-ask for backup
secret transfer approval or redo successful initial cloud verification.

### Notification tap navigation — 2026-10-03 (latest report)

**Latest acceptance:** after the deployed fix, the user confirmed notification
taps now work. Android Chrome delivery and opening are user-verified. A question
is pending about whether iPhone was also tested; do not infer that it passed.
The user now requests completion of the unfinished backup/monitoring work and
manual instructions. Backup-secret transfer/activation was subsequently approved
and completed above. Sentry owner alert configuration was also completed above.

The user confirms phone push delivery works, but tapping does not go straight to
the app; they identified Android Chrome. Exact post-tap behavior is still being
clarified and iPhone-specific acceptance is not recorded. The previous worker
selected the first same-origin window, awaited navigation before focus, and had
no fallback on rejection. The revised worker immediately calls `openWindow` to
let the browser select an installed app, focuses the returned client, corrects a
returned start-page URL, and falls back to live same-origin clients if opening
fails/returns null. It continues past inert/closed clients. Normal session and
API authorization remain unchanged; unsupported links still open the inbox.
Worker regression tests cover immediate opening, returned-window focus, root URL
correction, failed focus/opening, null results, inert/closed/foreign clients and
unsafe links. Actual Android/iPhone tap acceptance requires a production phone
retest after accepting the installed app's Reload update prompt with no drafts.

Web main local commit `b9c030abeae4bf7ef548e32a5df684f36dd792b9` passed
lint, typecheck, all 152 tests and production build. Publication scan and staged
diff checks passed. Automatic approval review initially rejected publication;
the user then explicitly requested deployment on 2026-10-03. The tested commit
was pushed to app main and Vercel production
`housemate-q7zd168a0-dreamy-day.vercel.app` reached READY. Canonical
`https://housemate-app-xi.vercel.app/sw.js` returns HTTP 200 and matches the
tested worker byte-for-byte after line-ending normalization. Web health,
House and standalone manifest checks passed on the canonical origin. GitHub
clean-build run `37088547498` succeeded (clean install, lint, typecheck, all 152
tests and build). Sanitized Vercel error/5xx checks returned zero entries in the
15-minute window. Actual Android/iPhone notification-tap retesting is pending;
instruct the user to open the installed app, finish drafts, accept Reload if
offered, then tap a fresh notification with the receiving app closed.

The push report itself did not authorize GitHub backup secrets; separate approval
was subsequently obtained and activation completed above. Sentry alert access
and actual owner email receipt remain pending.

### Backup automation and monitoring — 2026-10-03 (progress before activation)

The user requested push delivery acceptance, then automated offsite backups and
monitoring alert verification. They deferred phone checks because the phones are
unavailable and deferred Sentry alert-token setup, asking to continue work that
needs no manual intervention. Push remains enabled; real device receipt/deep
links and Sentry/email receipt must remain unchecked.

Web uptime job 8546967 was disabled with status 6 (response too large). The
landing page returned 200 with 66,264 bytes; HEAD also failed at the provider.
New web `GET /api/health` is a small 41-byte, no-store Next.js runtime check.
App main `eb87edc82e600722fe83dca502e806f786a092d7` passed lint/typecheck, all
142 tests and build; clean GitHub CI 37085290465 succeeded and Vercel production
deployment `dpl_4TpXjk9YAp67hhyUrvx1PzRRjvKX` reached READY at
`housemate-p6bvps6uo-dreamy-day.vercel.app`, with canonical web alias unchanged.
The existing web monitor was migrated in place to that endpoint and recorded
scheduled HTTP 200 at 08:20:03 Bangkok (747 ms). API readiness and reminder job
8546464 remained healthy. Do not replace an unrelated monitor or invoke the
notification worker as a test.

Isolated temporary alert job 8567732 used an unknown API path (404), then public
`/health` (200), with account-owner email settings and no credentials. Scheduled
failure at 08:10:36/08:11:02 and recovery at 08:12:17/08:13:02 were verified;
the job was deleted after recovery. Actual emails remain unverified. Repeatable
`uptime:alert-*` commands preserve sanitized local evidence and expire tests after
one hour using the provider's YYYYMMDDhhmmss local-time format.

A fresh encrypted production backup was captured at 08:15:47, 66 tables and
zero Storage objects. The existing recovery key had an empty Windows access
list. Owner-only Read access was restored using native ACL tooling for MSI\\ADMIN
on that single key; no inheritance/group access was enabled and no key rotated.
The complete isolated restore passed at 08:25:04 in 16 seconds: all counts and
digests, Auth references, private-table RLS, three API/financial smoke checks,
and recovery SQL against provisioned managed schemas. PostgreSQL extraction
needed explicit `--file=-`; schema-filtered SQL also needed namespace creation.
Those fixes apply to both drills and unpacking. Managed Supabase/Auth service
cutover is not claimed complete.

Private DreamyDay Vercel Blob store `housemate-backups`, ID
`store_tVVZomZrIaTQi403`, bom1, exists with no project connections. API offsite
scripts implement encrypted private uploads, download/decryption/checksum and
anonymous-denial verification before retention (7 daily/4 weekly/3 monthly,
Bangkok buckets, maximum 14 overlapping points, only owned strict paths). The
store budget is 256 MiB and an unattended archive is capped at 16 MiB; nonzero
Storage objects block validation until object bytes are backed up separately.
The API GitHub workflow is prepared for daily 03:17 Bangkok and initial/manual
plus Saturday isolated restore checks. It remains gated by repository variable
`BACKUP_AUTOMATION_ENABLED` until credentials and a successful initial cloud run
are verified. **No offsite archive or unattended cloud run is verified yet.**
API main `3af2ba05eb5f0efa934329e4870232164ce2b37b` is published. Clean API CI
37086450067 passed lint/typecheck, 75 tests (62 TypeScript plus 13 operations),
and build; the backup workflow is registered as 373608794 with zero runs.
The API deployment `dpl_2xzeSW9LRK1rc8CoMpAxNXHHuFQm` is READY at
`housemate-okmnr15oz-dreamy-day.vercel.app` with the canonical API alias unchanged.
Public health/readiness, protected-route rejection and CORS checks passed after
publication; the previously approved VAPID configuration remains in production.
Store read/write credentials are not exposed by the available Vercel connector;
browser initialization also failed. An explicit approval question is pending:
connect only the new backup store to API Development, read only its generated
server token, and provision the existing database URL/recovery key plus store
token as private API GitHub Actions secrets, then activate and verify an initial
offsite run. This avoids manual dashboard setup if the approved CLI path works.
Do not read/decrypt that new environment value or transfer secrets before this
approval, and do not reuse the earlier VAPID/Gemini consent. Keep the recovery key
separate from Blob and Git.
See `api/OPERATIONS.md` for activation and verification. Preserve the pre-existing
generated web type-file difference and member-settings line-ending change.

### Production web push enabled — 2026-10-02 (latest request)

The user reports completing rollout priorities 1 and 2 (joining/sign-in and core phone flows) with household members, and requested priority 3: enable live web push. This report does not establish every separate accessibility, performance or Assistant edge-case acceptance check. The user supplied a real email contact and explicitly approved transferring the existing VAPID private/public key pair and that contact to DreamyDay housemate-api production, then redeploying. Automatic approval review initially rejected the secret transfer; it proceeded only after that explicit consent. Contact and key values remain only in server configuration and ignored local environment files; do not copy them into tracked documents or outputs. Existing key pair was retained and cryptographically validated, not rotated.

Vercel production has all three VAPID variables as sensitive values. Ignored api/.env and api/.env.vercel were synchronized; the verified production configuration constructs a push sender. API source remains `cbddc597d185310171f987d35a31abebcf87dba7`. Redeployment `dpl_85RjGnNcDsu54tQxZzTCWuvpw2LV` reached READY at `https://housemate-5qslo01h4-dreamy-day.vercel.app`; canonical `https://housemate-api.vercel.app` was verified to resolve to it. Web stays at `4a55f76f50b23f2734d4c756e656305f1098537c`; no runtime web/API code, migration or scheduler configuration change was needed. Health/readiness, protected-route rejection and CORS checks passed. Scheduler job 8546464 remains healthy and enabled every minute; recent actual scheduled HTTP 200 runs continue after activation. Runtime error counts are checked separately; do not infer actual device delivery from a healthy scheduler or READY deployment.

Next: each member explicitly enables their own browser under Notifications → Settings → Refresh settings and devices → Enable push in this browser. iPhone requires a Home Screen web app on iOS 16.4+, opened from its icon. Category preferences/quiet hours remain personal; Shopping push defaults off. New opt-in does not replay old notifications. With the receiving app closed (without signing out) and quiet hours off, have another account publish a new announcement addressed to the receiving member, wait about two scheduler cycles and verify the generic notification and exact announcement deep link. The user was asked to report actual delivery/error; this check remains pending until evidence arrives. No household test messages were created or sent by Codex. Key/contact setup is complete; real Android/iPhone delivery, denial and deep-link acceptance must not be claimed complete prematurely.


### Search and additional Assistant drafts deployed — 2026-10-02 (latest request)

The user authorized deployment of both pending features. Production action-create migration was applied before publication using the ignored private production configuration. A read-only schema check verified four columns, two validated constraints, two unique partial indexes and retained RLS on both private tables. No household records, credentials or provider settings were changed for deployment acceptance.

API main `cbddc597d185310171f987d35a31abebcf87dba7` deployed first, then app main `4a55f76f50b23f2734d4c756e656305f1098537c`. Both exact-commit Vercel production deployments reached READY under DreamyDay. Canonical API `https://housemate-api.vercel.app` resolves to `housemate-j6vpolnk7-dreamy-day.vercel.app` (`dpl_GP9CcHE79tr7gHRNpF2ZAbnKZzCD`); canonical web `https://housemate-app-xi.vercel.app` resolves to `housemate-grf1btwro-dreamy-day.vercel.app` (`dpl_3dLavLMK4uUkcSCNvtmH6yFaivKp`). The Vercel connector lacked DreamyDay scope access, so the existing authorized Vercel CLI session was used for sanitized metadata and log counts. No new provider credentials were transferred.

GitHub API CI run [36909165068](https://github.com/heinzaw-bit/housemate-api/actions/runs/36909165068) and web CI run [36909395543](https://github.com/heinzaw-bit/housemate-app/actions/runs/36909395543) both completed successfully. API health and database readiness returned 200; unauthenticated search, assistant availability and action preparation returned 401; allowed web CORS and unlisted-origin rejection passed. Home (`/`), Assistant, House, Money, Profile, Notifications, sign-in, manifest, service worker and offline page returned 200. The Home route is `/`, not `/home`. Initial read-only log checks for each exact deployment returned zero error-level and 5xx entries in the preceding 15 minutes; this is a bounded snapshot, not ongoing monitoring.

Next: use the signed-in production app to check search permissions and editable shopping/chore/expense/bill draft confirmations, same-key interrupted retries and eligible notifications, then run Android Chrome/iPhone Safari acceptance from `app/QUALITY_CHECKS.md`. These authenticated/device checks remain pending; HTTP checks and DOM tests do not establish them. Preserve generated local `app/next-env.d.ts` development imports and the content-unchanged Profile test line-ending status outside commits. Production push was subsequently enabled as documented in the latest section above; device delivery acceptance remains separate.


### Assistant shopping, chores, expense and hybrid-bill drafts — 2026-10-02 (included in current release)

The user requested the combined Assistant follow-on item and explicitly chose **shopping, chores, expenses and bills** in this release. Existing authorized Q&A and meal/announcement drafts were retained. Additional local action parsing now returns unsaved typed drafts with visible original-request/detail guidance and dynamically unresolved required fields; no Gemini/budget calls or raw prompt persistence are added. New clients opt into new kinds via `allowedKinds`; omission preserves meal/announcement-only replies for older clients. Availability advertises `supportedDraftKinds`. Unsupported edits/deletes, negated or multiple commands are rejected locally. Ambiguous member names/dates/amounts are not guessed; only one unambiguous explicitly household-currency-marked positive total is proposed. Empty payer/participant/date fields require explicit input.

The retained focused Assistant chat dispatches new drafts to `app/components/assistant-record-draft.tsx`. Shopping supports optional quantity/unit/note/buyer/price and priority, one item per confirmation. Chores require a selected active assignee and exact future household-local due time. Expense supports equal/exact/percentage/ratio inputs; bill supports provider/period/due/status, selected shared base participants and explicit individual/shared charges/credits. Financial Review calls the existing API preview, displays its exact shares/base/adjustment breakdown and freezes the reviewed payload; Edit invalidates that preview. Confirm and save calls the normal authorized creation API. Review/Edit/Cancel never writes. Saving blocks repeat submissions; interrupted replies retain the exact payload/key, lock edit/cancel and require manual same-key retry even if a later retry is rejected. Definitive first-attempt validation rejection permits correction. Scope/access changes discard drafts and late replies; existing tab-memory leave/update/account-switch guards remain. Normal record links and actor/time/history/notification rules apply; no payment is processed.

Shopping/chore normal creation APIs now accept an optional UUID idempotency key. `api/src/create-request.ts` uses a caller/household/table-scoped transaction advisory lock, immutable normalized request hash and partial unique index; identical retries return the current saved record even after a later edit/completion, changed input conflicts, and legacy callers may omit the key. Existing creator/time/record ID plus immutable key/hash trace execution. New creation uses the same normal insert, so only an original insert triggers creation notifications. CLI-generated additive migration `api/supabase/migrations/20261001180100_action_create_idempotency.sql` adds nullable key/hash columns and scoped indexes to existing private shopping/chore tables; no new client grants/RLS policies are added. `npm run migrate:action-create` applies it once in a transaction with a bounded lock timeout and rejects partially applied columns. Production migration is applied, and this code is committed, pushed and deployed together with search; see the release evidence above.

Verification: API lint/typecheck/build and **70 tests (62 TypeScript + 8 operational)** pass. Web lint/typecheck, **142 tests in 26 files** and production build pass; focused Assistant suites have **33 passing tests**. New coverage checks all four expense split inputs, all four bill charge/credit kinds, API-only calculation before explicit save, visible unresolved details, retained edits/cancel without writes, preview failure, definitive rejection, safe same-key retry through a later rejection, single in-flight writes, unmounted reply discard, chat/local privacy and saved record links, optional route-key dispatch/auth/validation, parser ambiguity and old-client compatibility. Structural axe passed for the draft in its actual main landmark context; rendered contrast/device acceptance is not inferred.

`npm run smoke:assistant-create` passed against **empty isolated copies in a scratch schema**, applying the migration only to those copies and rolling back every scratch object. It checked scoped retries/conflicts, one creation side effect via an isolated test trigger, current state after edits, legacy callers, invalid/inactive/foreign membership, malformed keys and saved creator/time/hash evidence; production household/schema and real notification triggers were unchanged. This does not establish real notification delivery, parallel-request concurrency or signed-in/device publication. Existing database security advisors show expected private-table no-policy INFO and the pre-existing Auth password-protection warning; no new exposure. OpenAPI parses with **445 local references resolved**; diff checks pass. SPEC, AI plan, READMEs, checklist and quality acceptance document this release and its migration requirement. Manual signed-in/Android Chrome/iPhone Safari acceptance remains pending; previous browser failures/denial must not be bypassed. Preserve pre-existing generated app/next-env.d.ts development imports outside commits.

### Household record search — 2026-10-01 (included in current release)

The user requested the follow-on household search item. API and web implementation is complete and published with the 2026-10-02 release above. The new GET `/v1/households/{id}/search` contract was added before the web client; `api/src/search.ts` and `search-routes.ts` implement verified active-household membership, the existing announcement/poll creator/moderator/audience rules, literal case-insensitive substring matching and optional type filtering across expenses, bills, recorded transfers, meals, chores, shopping, events, announcements and polls. Saved historical/voided/closed records are included with state labels; drafts, history events and emails are excluded. Queries are trimmed to 2–120 characters, pages default to 20 (max 50), and a five-second database statement timeout returns a safe retryable 503. SQL values are bound and wildcard characters are escaped. Cursor scope includes verified caller, household, normalized query and type; ordering preserves PostgreSQL microseconds, type and UUID ties. Pagination is a live read, not a frozen snapshot. No migrations, credentials, dependencies, provider calls or household mutations were added.

`app/components/household-search.tsx` adds a branded global header dialog with Ctrl/Command + K, keyboard/focus/Escape behavior, IME-aware 300ms debounce, type selector, status/snippet/date results, normal authorized record links, empty/offline/error guidance and guarded Load more. Explicit abort signals and scope/version checks prevent delayed responses from restoring old results; household/account changes, query changes, closing and offline transitions clear data. Access errors on pagination clear retained results. Search terms stay out of browser URLs and storage. Existing unfinished drafts block opening results. CSS uses existing light/dark tokens, >=44px controls and small-screen safe-area bounds. API request URL logging is already disabled, and existing error-reporting sanitizers remove private request/breadcrumb details.

Verification: API lint/typecheck/build and **66 tests (58 TypeScript + 8 operational)** passed; web lint/typecheck, **126 tests in 25 files** and production build passed. OpenAPI parses with **443 local references resolved**; root/API/web diff checks pass. Nine new web tests cover shortcut/focus/structural accessibility, empty/errors/retry, literal queries/filters, whitespace re-querying, stale response discard, cursor merging and unique keys, pending drafts, scope remounts, offline/reconnection, revoked access and IME input. Rollback-only `npm run smoke:search` passed all nine types (including each filtered branch), targeted privacy and author/moderator access, inactive/foreign membership rejection, literal `%`/`_`, Unicode, saved states, cursor scope, microsecond ties without duplicate/skipped pages, and existing private-table RLS. Temporary fixtures were rolled back. Normalized an existing Profile test's line endings for lint; it has no content diff. React review checked bounded state, request/listener cleanup, scoped keys, keyboard semantics and absence of writes. Actual signed-in rendered and Android Chrome/iPhone Safari acceptance remain pending in `app/QUALITY_CHECKS.md`; do not bypass prior browser failures/denial or call DOM mocks real-device acceptance.

The API was released **before** web so the search endpoint exists. Preserve generated local `app/next-env.d.ts` dev imports outside commits. Production includes search and the additional Assistant drafts; no production household mutations were performed for deployment acceptance. SPEC, checklist, API/app READMEs and quality checks describe the new feature. Large-household substring performance should be measured with query plans before adding new search indexes.

### Profile settings hub — 2026-10-01 (previous published request)

The user approved implementing the Profile suggestions: compact identity header and three sections (My profile, Household, Preferences), saved summaries with explicit Edit/Save/Cancel, searchable timezone selection, simpler members/invitations and preference shortcuts. `app/components/member-settings.tsx` now uses existing Base UI controlled keyboard tabs and scoped `member-settings.module.css` with brand light/dark tokens, flexible phone layouts and >=44px controls. Email appears inside expandable account details; member rows show avatar/name/role without emails. The production and design preview share the layout; preview disables household API reads/writes and retains Exit preview.

Name/household forms require a meaningful change to save; Cancel restores saved values without writes, first fields receive edit focus and the Edit button receives focus after cancel/save. Failed saves retain input and in-flight saves block repeats/cancellation. Pending edits, role review and operations protect section changes, ordinary navigation links, household/account switching and existing reload/update prompts. Browser history/forced termination are not persistent draft recovery. Owner role changes prepare an inline exact member/role review, require Confirm, and restore focus to the role selector; cancel performs no write. All saves use existing authorized API calls and role/currency rules. Timezone search filters native select options from Intl supported values, retaining the selected timezone.

Invitations show active links with expiry, explicit Invite someone/Copy/Revoke actions, and optional used/revoked/expired history. Create/join/switch remain in the global household menu. `workspace.tsx` removes the old preview-only Profile and passes existing Google account switching into the shared Profile. The household menu's optional settings callback opens the Household panel when its existing anchor link is used from the same Profile route; anchor entry from other routes remains supported. Preferences has the shared appearance toggle, installation help/status and `/notifications?household=<id>&view=settings`; notification workspace recognizes that query and opens its existing Settings. Opening Preferences does not request push or create a subscription. No API, schema, provider, credential or dependency changes.

Verification: lint, typecheck, **117 tests in 24 files**, and production build pass. Eight new Profile tests cover keyboard/panel semantics and structural axe, collapsed account details, edit/cancel/confirmed save, protected normal navigation and sections, retained failures/in-flight controls, settings anchor/same-route request, timezone/currency permissions, member/admin restrictions, role review confirmation/cancel/focus, invitation history/create/revoke and preview isolation. React review checked bounded controlled state, unique component-local fields, cleaned-up navigation/hash listeners, normal API-only writes, focus restoration, no render-time mutations or additional fetch/dependency chains and responsive/theme styles. Signed-in rendering and Android Chrome/iPhone Safari acceptance remain pending; previous browser-tool failures/denial must not be bypassed. Tests use mocked APIs; no production household records were created.

Publication status: the user explicitly requested deployment. App main `f8609a0d4ff86e3623b6042bf86f476a13f29591` is committed and pushed. Vercel production `housemate-dp5qzo8h0-dreamy-day.vercel.app` is **READY** for that commit at https://housemate-app-xi.vercel.app. Canonical Home, Profile and `/notifications?view=settings` routes return **200**. A bounded one-hour error-log scan for this deployment returned **0 error records**; it does not prove signed-in/device acceptance. GitHub CI **36888482107** remains **queued waiting for a runner** at the last check; local lint/typecheck/117 tests/build and Vercel production build passed. Recheck this existing run before claiming remote CI passed; do not trigger duplicate deployment/jobs merely because it is queued. SPEC, app README and quality checklist describe the new behavior; the whole-project release commit pins the published app revision. Preserve the pre-existing generated local `app/next-env.d.ts` dev imports outside feature commits. API remains `8c145f5` and needs no release. No production household records were created during deployment checks.

### Household navigation menu; Home selector removed — 2026-10-01 (latest request)

The user asked to remove the household picker from the Home heading and apply the proposed functional menu to the sidebar card. `workspace.tsx` removes the old Home selector and wires shared `household-menu.tsx` into global navigation. Desktop gets one compact whole-card menu trigger with checked current household, other memberships, Create household, Join with invite and Household settings. When the sidebar is hidden at <=900px, a global header control opens the same actions as a modal bottom sheet; Home content no longer has a picker. CSS uses existing light/dark tokens, >=44px actions, dynamic viewport bounds, safe-area padding and reduced-motion support. Base UI provides menu arrow/typeahead semantics, sheet focus trapping, Escape/dismissal and focus restoration.

Actions use existing callbacks and API flows. An open draft blocks switching/create/join/settings navigation with an inline message and leaves the draft intact. Join locally validates a link/code, protects unfinished invitation input with the pending-work guard, then opens the existing invitation review before acceptance. No menu-open/selection/form-open action directly creates household records. Preview disables real create/join actions. Settings link to the selected household's existing Profile settings anchor with normal role permissions. Two related web state fixes retain existing memberships after creating another household and clear old entity/create/event-prefill query parameters when switching while preserving the current tab/view. No API, schema, secret or dependency change.

Verification: web lint/typecheck, **109 tests in 23 files**, and production build passed. Six new menu tests cover keyboard selection/check/dismissal/focus, one-household actions, draft-blocked context changes, invite validation/routing, mobile sheet semantics/structural axe and preview restrictions. Two new account-view tests check retained memberships after creation and removal of old record/prefill links during switching. React review checked unique IDs, controlled current-household state, callback-only writes through normal API flows, guarded navigation, focus/modal semantics and theme/responsive styles. Actual signed-in rendering and Android Chrome/iPhone Safari checks remain pending; existing browser-tool failures are documented and must not be bypassed. Preserve local generated `app/next-env.d.ts` development imports outside commits.

Publication status: after the initial automatic-review rejection, the user explicitly approved deployment. App main `251a1ee70844f36872df2753da9a38f22df88c6b` is pushed. GitHub CI **36872387909** passed clean install, lint, typecheck, all 109 tests and production build. Vercel production `housemate-dfa8tji5c-dreamy-day.vercel.app` is READY for that commit at the existing https://housemate-app-xi.vercel.app alias. Canonical Home, Profile/settings and Money routes return **200**; these establish deployment/routing, not signed-in rendering or phone acceptance. No production household records were created for verification. Root implementation/docs commit `c892034` pins the new app revision; the follow-up root commit records approved publication and these results. API is unchanged. Preserve the generated local app/next-env.d.ts dev import outside commits.

### Assistant meal-check and announcement drafts — 2026-10-01 (latest request)

The user asked to start the recommended action features: meal checks and announcements first, before expense/bill drafts. The existing focused chat/mascot layout remains. A supported English command such as “Plan dinner tomorrow at 7pm” or “Announce that dinner is ready” prepares an ephemeral draft, shows missing details, allows editing recipients/content, then requires Review and Confirm and publish. Review/Edit/Cancel perform no writes. Successful publication links to the normal House record. Financial drafting remains pending.

The additive `POST /v1/households/{id}/assistant/drafts` contract was added before client use; availability adds `actionDraftsEnabled`. API `assistant-drafts.ts` interprets commands locally to keep names, times and private messages out of unpaid Gemini. It performs no provider/budget calls, writes or draft persistence; no migration/secrets are added. Household membership is checked before and after preparing member data; output excludes emails and is no-store. Unclear/extra dates, bare hours, relative weeks and limited/named audiences require explicit editing. Meal response deadlines are always supplied by the user. Recognized unsupported, negated and multiple commands never fall back to Gemini; old question clients receive local clarification. General Q&A retains its Gemini consent/privacy guard.

Web `assistant-draft-review.tsx` shows an exact household/timezone/audience review and confirms through existing meal/announcement APIs, which enforce permissions, active recipients, time/content and idempotency and create normal preference-scoped notifications. Confirmations freeze their exact payload/key; an interrupted result stays locked for manual same-key retry, including when a later retry gets a validation rejection. Only a definitive first-attempt rejection allows editing/new key. Access/scope changes clear private drafts and reject late responses. Drafts protect update/account-switch reloads through the existing pending-work guard; reload/termination still loses tab-memory drafts. Expense actions still open Money's deterministic preview flow.

Verification: API lint/typecheck/build and **62 tests (54 TypeScript + 8 operational)** pass. Web lint/typecheck/build and **101 tests in 22 files** pass, including 17 Assistant tests covering local drafting without Gemini consent/setup, review without writes, targeted recipient selection, publication links, validation correction, same-payload/key retry after interruption, scope/access clearing and structural axe checks. OpenAPI parsed and all **438** local references resolved. React review checked no render-time writes, bounded state, explicit confirmation, single in-flight write, exact retry payload, private-response isolation, labels/focus, responsive/theme tokens and no new dependencies. Tests are mocked; signed-in actual publication/notifications and Android Chrome/iPhone Safari rendered acceptance remain pending. Existing browser-tool failures are documented; do not bypass them. Preserve generated local `app/next-env.d.ts` dev imports outside commits.

Publication status: API main `8c145f524333d94818a11800f7a5372650233435` was pushed and verified before web main `df3bf0890297d1637afff2bbe0a6cfbf7564c82f`. GitHub CI **36856097003** (API) and **36856226330** (web) passed clean install/lint/typecheck/tests/build. Vercel API production `housemate-cwurpposh-dreamy-day.vercel.app` and web production `housemate-f5wymz2q6-dreamy-day.vercel.app` are READY for those commits at the existing canonical aliases. API health returns **200**; the new draft route rejects an unauthenticated valid request with **401**; the canonical Assistant page returns **200**. These establish deployment/routing/access protection, not authenticated publication or browser rendering. No production household record was created as a deployment check. Availability gating preserves compatibility. Whole-project code/documentation commit `4ba55cf` pins these child revisions; deployment verification is recorded in the follow-up root commit.

### Sidebar removal, Suggested transfers and balanced attention cards — 2026-10-01 (latest request)

The user clarified that the entire bottom sidebar profile block should be removed, including its divider, avatar and name. The block and its unused CSS selectors are removed; top-bar account controls remain. Suggested transfers now previews two current suggestions in the API's existing calculation order, with its own accessible Show more/Show less button, chevron and unique controlled-region ID. Suggestions have no dates, so no chronology is invented; expansion performs no API reads or financial mutations. Existing source/recorded-transfer previews remain independent.

Home's Bills to watch no longer spans two columns/two rows. Bills, chore and meal cards now form a three-column equal-width/equal-height desktop row, naturally stacked on small screens. Flexible content and dedicated card footers align action links along the bottom; no fixed height or clipped content is introduced. Bills keep the three-record preview and all existing household record links and empty states. API/financial logic and Assistant layout are unchanged.

Web lint/typecheck, all **94 existing tests in 22 files**, and production build passed. React review checked independent suggestion expansion, unique controlled-region IDs, preserved API ordering, flexible card contents/footers, responsive wrapping and retained empty-state links. The user approved publishing the completed update after the automatic-review block. App main **cff1ef28b7593c70061c884c185891d34bd50e35** is pushed. GitHub CI **36850081487** passed clean install, lint, typecheck, tests and build. Vercel production `housemate-4ighdtjuv-dreamy-day.vercel.app` is READY for that commit at https://housemate-app-xi.vercel.app. Canonical Home and Money routes return **200**; these HTTP checks establish deployment/routing, not signed-in UI rendering. Keep generated local `app/next-env.d.ts` development imports outside commits. Signed-in rendering and phone acceptance remain pending as previously documented.


### Compact sidebar, Home and Money — 2026-10-01 (latest request)

The user asked to remove the displayed Gmail address in the sidebar profile, Home quick actions, and add expandable previews to Recorded transfers and Why this balance. The sidebar now shows the avatar/name without the email line. Home's entire Quick actions section and unused imports are removed; normal Money/House add flows remain available. The two Money cards initially show up to three newest records by the existing API source/transfer-date ordering. Separate accessible Show more/Show less buttons with chevrons and unique controlled-region IDs expand loaded history; existing 50-record pagination is shown only while expanded. Switching members resets the balance-source expansion. Recorded transfers are sorted immutably to retain API date ordering after adding a backdated transfer. API-calculated balances, financial confirmation/preview, record deep links and Assistant layout are unchanged.

Web lint/typecheck, all **94 existing tests in 22 files**, and production build passed. React review checked independent expansion state, member-reset behavior, unique disclosure IDs, keyboard button semantics and immutable date sorting without new fetches on expansion. The user explicitly approved pushing/deploying the completed app and whole-project commits after the initial automatic-review rejection. App main **922afce40b74e4078ac9f553f8c75f44c139004b** is pushed. GitHub CI **36848119296** passed clean install, lint, typecheck, tests and build. Vercel production `housemate-8pbv0ittl-dreamy-day.vercel.app` is READY for that commit at https://housemate-app-xi.vercel.app; canonical Home and Money routes return **200**. HTTP checks establish deployment/routing, not signed-in UI acceptance. Actual signed-in rendering and phone checks remain pending. Preserve the generated local `app/next-env.d.ts` dev import outside commits.


### Mascot brand and app icons — 2026-10-01 (latest request)

The user requested using the existing Assistant mascot as the in-app logo and installed app icon. `app/components/housemate-mascot.tsx` now shares the unchanged house-helper drawing between Brand and Assistant. All Brand consumers (landing, auth, onboarding, invitation and navigation) keep the HouseMate wordmark and receive the mascot. Assistant layout/styles are retained. `public/icon.svg` mirrors the character in fixed brand colors; `npm run icons:generate` uses the direct Sharp development dependency to export 192/512 PNGs, a padded opaque Android maskable icon and an opaque 180px Apple icon. Manifest and root metadata icon URLs use `?v=mascot-1`; the installed app identity, start URL and scope are unchanged. No API/provider/database changes.

Web lint/typecheck, all **94 tests in 22 files**, production build and clean-install lockfile dry run passed. Asset verification checked all PNG dimensions, opaque Apple/maskable backgrounds, matching SVG path geometry and existing versioned manifest assets. The maskable character occupies a maximum 188px radius inside the 204.8px safe circle. The normal and maskable exports were visually inspected. React review found no new hooks, fetches, client boundary or focusable decorative SVG. Web main **367449f402403a3b0c297bf93ab96d766f97cb1e** is committed/pushed. GitHub CI **36845790451** passed clean install/lint/typecheck/tests/build. Vercel production `housemate-bxs88caz2-dreamy-day.vercel.app` is READY for that commit at the existing https://housemate-app-xi.vercel.app alias. Production manifest and all five icon assets return 200 and match local files (SVG compared with normalized Git/Windows line endings); public landing HTML includes the smiling mascot and versioned browser/Apple icon metadata. These checks establish asset delivery, not browser/device rendering. Actual rendered/browser and phone installation acceptance remains pending as documented; do not claim OS icon refresh from asset checks. Preserve the generated local `app/next-env.d.ts` dev import outside commits.



### Assistant focused chat layout — 2026-10-01 (current layout)

The user rejected the card-heavy Assistant because it hid the main chat action, asked to keep the mascot, and prioritized questions, meal planning and money calculation. The web layout now puts one large chat box directly below the retained smiling house mascot and a short heading. The welcome hero, four-card grid, secondary-topic row, sticky composer, repeated answer mascot/badges and automatic answer scrolling are removed. Only three small text shortcuts follow the input: Catch me up (summary), Meal plans (meals) and My balance (balance). The API still accepts all seven quick topics and free-text intent classification; the web prioritizes the requested three areas.

The newest answer stays below the chat box. Up to five older answers are tucked into native collapsed history (six total in tab memory), with compact three-row previews, expansion, checked timestamps and authorized source links. Meal answers offer **Plan a meal**, linking to `/house?view=meals&household=<id>&add=1`; balance/bill answers offer **Calculate a split**, linking to the existing household-scoped expense creation form. These open the normal planning/calculation/review flows; no record is created by navigation and no AI-generated meal/financial draft capability was added. The API remains the financial authority with preview and explicit confirmation.

The Gemini privacy acknowledgement appears when input is typed or consent has already been accepted, and is still required before free-text submission. Full details are behind a small privacy summary; quick questions need no consent/provider request. Ctrl/Command + Enter, cancellation, rejected late responses, access-loss clearing, household/session reset and unsent-question protection remain. Clearing answers retains unfinished input and restores focus. The composer stays in normal page flow, preserving access above the standard phone navigation without covering answers. Colors use the existing light/dark tokens; no dependency, API contract, migration or provider configuration change was needed.

Verification: web lint/typecheck, **94 tests in 22 files** and production build passed, including **10 assistant tests** for consent/error/isolation behavior, latest/earlier-answer display, expansion without extra reads, keyboard cancellation, preserved input/focus and household-scoped meal/split links. SPEC, AI plan, web README and QUALITY_CHECKS describe this layout and supersede the historical UI entry below. Web main **afc05e58ffa9d84f9a49bf8e02c1aabb1e8762e7** is committed/pushed. GitHub CI **36842378707** passed all checks, including the full suite/build. Vercel production `housemate-h92g2y46p-dreamy-day.vercel.app` is READY for that commit; the canonical https://housemate-app-xi.vercel.app/assistant route returned **200**. HTTP establishes routing/deployment, not signed-in rendering. Preserve the generated local `app/next-env.d.ts` dev import outside commits. Browser preview is documented as unable to initialize; no repeated bypass attempt, rendered or real-phone acceptance was performed. Next manual acceptance is the signed-in screen on Android Chrome/iPhone Safari, keyboard/safe areas/zoom/dark mode, and the linked normal meal/expense forms.


### Assistant conversation redesign — 2026-10-01 (superseded layout)

The user asked for a clear, simple, cute Assistant tab aligned with HouseMate's brand, with an explanation before implementation. The layout was explained in commentary and implemented in `app/components/assistant-workspace.tsx` and its CSS module. It has a compact household header, a small smiling house mascot drawn as a decorative SVG, a friendly welcome, four main question cards (overview, personal balance, chores, meals), and smaller bills/shopping/events choices. Once a question starts, shortcuts become compact chips. The six recent answers run oldest-to-newest, with user bubbles, answer cards, household-scoped source links and checked timestamps. Three record rows appear initially, with explicit expansion/collapse that makes no extra API request. API list-limit notices remain intact.

A responsive bottom composer stays above phone navigation/safe areas and uses normal page flow on short screens or when privacy details are expanded. Ctrl/Command + Enter submits only with consent; plain Enter remains a newline and composing input is not sent. Progress shows a small house-helper and reduced-motion-aware dots; cancellation ignores late replies and retains unfinished input. Errors retain input and quick questions remain available. Clearing answers preserves the next question and restores focus to input (or the first shortcut if free text is unavailable). New answers scroll into view without moving focus. The Google data-use consent is still visible and required; fuller privacy information is expandable. Each question is independent. The existing API, Gemini configuration, budgets, authorization, mutation boundaries and migrations are unchanged; no additional provider calls or secret transfers were needed.

SPEC, AI plan, web README and QUALITY_CHECKS reflect the new experience. Web lint/typecheck, the full **93 tests in 22 files**, and production build passed; nine focused assistant tests cover prior isolation/consent/error behavior plus keyboard submission/cancellation, record expansion without extra reads, conversation order and clearing while preserving input/focus. A final availability-status formatting correction is included in web main **099596fe65339eb7fe30839d2efa7de847f6aae3** after redesign commit `129d4da`. Both are pushed. Vercel production deployment `housemate-9h34180pv-dreamy-day.vercel.app` is READY at the existing https://housemate-app-xi.vercel.app alias. Final GitHub CI **36840163033** passed all gates, and the canonical production `/assistant` route returned **200**. This HTTP check verifies deployment/routing, not signed-in rendering. Keep the existing local `app/next-env.d.ts` dev import outside commits.

Browser preview still failed to initialize (`failed to write kernel assets`, missing path). No rendered or signed-in phone check was performed. Do not bypass the documented browser denial or invent a new helper protocol. Next acceptance is the signed-in Assistant on Android Chrome and iPhone Safari: narrow layouts, keyboard/safe areas, expanded privacy details, zoom, dark mode, motion preferences and source navigation. Gemini integration is already active as described below; do not redo keys/migration setup. Shopping/chore and financial action drafts remain later features.


### Gemini free-tier integration — 2026-10-01 (current assistant setup)

The user requested Gemini's free tier, saved `GEMINI_API_KEY` privately in ignored `api/.env`, and explicitly approved uploading it to DreamyDay's production API environment and deploying. This supersedes the OpenAI setup in the historical phase-1 entry below. `GeminiAssistantProvider` uses fixed `gemini-3.1-flash-lite` with Google GenerateContent structured JSON, server-only `x-goog-api-key`, a 12-second timeout, 400 output-token cap, one attempt and no paid fallback. Strict parsing rejects blocked/incomplete/malformed/tool-like output. Provider 429 becomes sanitized `AI_PROVIDER_LIMIT_REACHED`; local budget limits remain unchanged. Migration 016 was already applied and was not rerun.

The private key and `GEMINI_FREE_TIER_CONFIRMED=true` are configured locally and on **housemate-api Production** only (key sensitive). The flag is the operator's acknowledgement of a Google Free Tier project with billing disabled; HouseMate cannot verify billing from a key. Keeping Google's project on Free Tier prevents paid use. No Gemini secret was added to the web project, public variables, Git, logs or chat. A benign live call, “How much do I owe?”, successfully classified as balance with null scope. No household data was sent for that check.

Google's unpaid-service terms allow product-improvement use/human review and restrict personal/confidential inputs. General typed questions now require a visible privacy acknowledgement and `privacyAcknowledged:true`. Before budget/provider calls, the API checks known active member names/emails, digits, links and credential markers; this is explicitly a limited guard, not a complete personal-information detector. Named-member balances remain in Money. Google sees only a general question, never member lists, household records, balances or previous answers. API services still produce all answers and amounts. Quick questions need no acknowledgement or provider request. Consent, unsent input and answers clear on household/session changes; cancellation and draft protection remain. Action drafts remain pending.

Verification: API lint/typecheck/build and **60 tests** (52 TypeScript + 8 operations) passed; web lint/typecheck/build and **90 tests in 22 files** passed, including six assistant tests. Final consent layout passed the focused tests and a rebuilt production app. OpenAPI parsed with **432 resolved local references**. API main **bb43d425e8f7ebec283984825aa56aefc810229b** and web main **3c366d88f92a05a020009ac394950a4f353bfd35** are pushed; GitHub CI **36837094271** and **36837177236** passed. Vercel production deployments `housemate-8aozjmys6-dreamy-day.vercel.app` (API) and `housemate-1shbulask-dreamy-day.vercel.app` (web) are READY. Canonical API health returned 200 and unauthenticated assistant config/questions returned 401. API/web READMEs, SPEC, AI plan and checklist reflect Gemini. The generated local `app/next-env.d.ts` retains the dev route import outside the feature commit.

Next acceptance: sign in at https://housemate-app-xi.vercel.app/assistant, acknowledge the notice, ask a general question and compare with the source page; test both target phones. Signed-in production/browser/device acceptance remains unverified; the live classifier check plus automated tests do not establish that acceptance. Browser tooling previously failed/denied localhost access; do not circumvent the denial. No new migration or local API restart was needed for production deployment; a local API process must be started/restarted before local live testing. Future shopping/chore and financial draft phases remain unchecked and require normal preview/explicit confirmation/idempotency/audit boundaries.

### Assistant phase 1 — household Q&A, 2026-10-01

The user asked to start the recommended assistant features in order. The first implemented slice is read-only household Q&A; shopping/chore drafts and financial drafts remain later phases. `api/src/assistant.ts`, `assistant-provider.ts`, and `assistant-budget.ts` add authorized quick topics and optional OpenAI classification, with `GET /v1/households/{id}/assistant` availability and `POST /assistant/questions`. Contract was added before the web client. All financial values and source links are generated by API services. The provider sees only the typed question, never household records/member lists/prior answers, and uses Responses structured output with `store: false`. No mutation tools/endpoints or conversation persistence exist.

`app/components/assistant-workspace.tsx` replaces the live placeholder with seven quick questions, optional free text, checked timestamps, household-scoped source links, bounded-list notices, six in-memory recent answers, cancellation, and unsent-question update protection. Account/household/session changes abort pending reads and clear previous answers. Duplicate exact display names, unsupported writes/date ranges/questions, and individual bill-payment queries need clarification. Bills still record provider payment status only. Manual core flows remain available after provider failure.

Migration `016_assistant_usage.sql` is applied. The private RLS table has no client grants or policies intentionally, consistent with API-only access. The Supabase advisor returned the expected no-policy INFO and an existing Auth password-protection warning; no new exposure was introduced. `npm run migrate:assistant` and rollback-only `npm run smoke:assistant` cover setup and enforce 20 model attempts per account per UTC hour and 200 globally per UTC day with a shared advisory lock. Failed/cancelled attempts may consume budget. Provider calls have a 12-second timeout, 400-token output cap, and no automatic retries.

API lint/typecheck/build and **58 tests** passed; web lint/typecheck/build and **89 tests in 22 files** passed. OpenAPI YAML parsed and **432 local references** resolved. Database smoke passed for RLS/client privileges, atomic counters, and both limits; test changes rolled back. Preserve the existing local generated `app/next-env.d.ts` development import outside commits. Signed-in rendered/phone acceptance and live provider verification remain unverified.

**Setup still needed:** no OpenAI key has been provided or uploaded. Optional server-only `OPENAI_API_KEY` and `OPENAI_MODEL` (default `gpt-4o-mini`) are documented in the API README and blank `.env.example`. Quick questions work without this key; free text remains disabled until configured/redeployed. An optional provider-choice question was sent; no answer received while building, so the integration prepares the recommended OpenAI provider. Never ask the user to paste a key in chat or put it in the web project. Next: configure/verify live free text, then implement shopping/chore drafts with editable unresolved fields and audited idempotent explicit confirmation, followed by expense/hybrid-bill drafts. The combined assistant P3 item stays unchecked until those drafts are complete.

Published API main `02ec985d52259892cca96e612edf802ac08ae369` and web main `aa04d8b7477124097940b48b54973dbb74c1ca32`. Both corresponding Vercel production deployments are READY (`housemate-l6fnq74kz-dreamy-day.vercel.app` API, `housemate-gvzuz0i8f-dreamy-day.vercel.app` web), with the existing public aliases unchanged. API GitHub CI run `36833541600` succeeded. Production `/health` returns 200; both assistant endpoints reject missing authentication with 401 and `Cache-Control: no-store`; the question endpoint allows the exact production web origin. No provider calls or secret transfers were made during these checks.

Web GitHub CI run `36833723378` also succeeded. The public `/assistant` route returns 200 with its distinct Assistant title. These HTTP checks establish deployment/routing, not signed-in or live-provider acceptance. Local API was not listening on port 3001 at the final check; use `npm run dev` from `api/` if local work resumes.

### Scroll reveal landing correction — 2026-10-01 (latest request)

The user clarified that visitors should **scroll down naturally**, seeing each section fade in, rather than use a section-by-section wizard. This supersedes the guided presentation below. All six sections now remain in normal document flow, with ordinary header anchors, no Next/Previous controls and no progress counter or intercepted scrolling/history. Desktop sections have generous viewport-based spacing; phone sections grow with their content. A single IntersectionObserver fades upcoming sections in over 600ms and unobserves each after its first reveal, so scrolling back never hides already-read content.

Content is fully visible in server HTML and without JavaScript. Only below-viewport content is prepared for fading after hydration. Keyboard focus reveals its section immediately, and reduced-motion preferences (including changes while open), missing IntersectionObserver or observer setup errors reveal everything. CSS also ensures focused/reduced-motion content stays visible without a transition. Observer/media/focus listeners are cleaned up on unmount. Signed-in dashboard, callback/invitation flows and API behavior are unchanged. README, SPEC and QUALITY_CHECKS now describe natural scrolling. Preserve the local generated `next-env.d.ts` development import outside publication. Actual rendered phone/motion acceptance remains pending because browser preview tooling has not initialized successfully.

Published app main **`e8e8dcb62eec4de2d4cad3db63caf431f1382f4f`**, including the scrolling implementation in `dffa564` and immediate-focus CSS refinement. GitHub CI **36820845778** passed clean install, lint, typecheck, all **84 tests in 21 files** and production build; local checks also passed before the CSS refinement, which passed lint and final CI. Tests cover scroll entry/one-time reveal, focus, changing reduced motion, unsupported/failed observer fallback, cleanup, preserved URL/history, CTAs, product-tour interaction and structural axe. Vercel production **`dpl_ARyieWXFnoWUKMNkv46g3TUi3CWs`** is READY at `https://housemate-app-xi.vercel.app`. Public HTTP returned 200 with six non-hidden sections, no pagination, and healthy CSS including fade/reduced-motion/immediate-focus rules. API main is unchanged.

### Guided landing sections — 2026-10-01 (superseded by scrolling)

The user requested breaking up the landing page to avoid a long scroll, with a fade effect for each section. `LandingSections` now presents six sections: Overview, Features, How it works, Tour, Privacy and FAQ. One section is visible at a time; header links and Previous/Next controls update the selected section and a 01/06 progress indicator. Sections fade in over 380ms, disabled for reduced motion. The header and pagination remain available on longer phone sections; content can grow rather than being clipped to a fixed height. The redundant problem block and large final panel were condensed; the final household CTA is alongside FAQ.

Hash links, reload and browser Back/Forward restore sections while retaining search parameters and Next.js history metadata. Navigation focuses the selected heading; the skip link focuses the current main content without resetting the section. Hidden panels are excluded from focus/accessibility navigation but stay mounted, preserving the product-tour choice and expanded FAQs. No autoplay or intercepted wheel/touch scrolling was added. Static content remains server-rendered and a noscript style reveals all public sections if JavaScript is disabled. Existing signed-in dashboard/authentication routing is unchanged. README, SPEC and QUALITY_CHECKS cover the behavior. No API changes or new dependency were needed.

Published as app main **`9791adddac3d19ed005a5afde9876c6035b59bce`**. GitHub CI **36819352631** passed clean install, lint, typecheck, all **81 tests in 21 files** and production build. Local production build/lint/typecheck and the final four landing interaction tests also pass. Vercel production **`dpl_57AZXKJQLikwUmhe8QK6rXb1JGtX`** is READY on `https://housemate-app-xi.vercel.app`; public HTTP returns 200 with six panels, five initially hidden, section links and pagination, and all referenced CSS returns 200 with fade/reduced-motion rules. Tests cover one visible section, keyboard/focus/progress, deep links/history, retained tour choices, CTA destinations and skip-link behavior, with structural axe checks on Overview and FAQ. Rendered transitions and Android Chrome/iPhone Safari layouts remain manual acceptance because browser preview tooling has not initialized successfully. Preserve the local generated `next-env.d.ts` development import outside publication.

### Public landing page — 2026-10-01

The user accepted the professional landing-page proposal and requested its implementation. Signed-out `/` now renders a public page headed **House life, in sync.**, with three feature groups, setup steps, an interactive Home/Money/House tour, a privacy overview, native expandable FAQs and sign-in/create-household calls to action. The cream/forest-green design uses the existing light/dark theme and responsive styles. Preview records are fictional, labelled **Example data**, and never read the household API. Footer privacy/use links point to factual on-page information; no binding legal terms, fabricated support address, testimonials, payment processing or unimplemented AI/LINE claims were added.

Published as app main **`1ddfde3ec7bd89979b0e65b6b6f09d9199be3941`**. GitHub CI **36812247622** passed; Vercel production **`dpl_75i33B3HMFsg9w3kSSW21GQnvkTM`** is READY with the stable `https://housemate-app-xi.vercel.app` alias. Public `/` and `/sign-in` returned HTTP 200; deployed HTML contains the hero, fictional-record labels, sign-in links and sign-in title. Production browser preview again failed to initialize (`failed to write kernel assets`, OS error 3), so this is HTTP/automated verification, not rendered acceptance. API main remains unchanged.

`app/components/home-entry.tsx` retains the existing authenticated root dashboard and invite/callback/error flow, including a stored pending invitation. `/sign-in` uses the existing auth screen and redirects restored signed-in members to `/`. Server-rendered landing children give signed-out visitors meaningful HTML, while authenticated workspaces stay lazy-loaded. The page/tour/previews and scoped `app/app/landing.css` introduce no dependency or API changes. SPEC, app README and QUALITY_CHECKS document the behavior.

Verification: web lint, typecheck, production build and **79 tests in 21 files** passed. Nine new tests cover public CTAs/anchors, structural axe checks, keyboard tour selection, FAQ focus/click expansion, restored sessions/sign-out, invitation/callback intent and sign-in redirection. jsdom does not establish rendered color contrast or native summary keyboard activation. Actual Android Chrome/iPhone Safari layouts, real OAuth callbacks and signed-in rendered acceptance remain pending in QUALITY_CHECKS. Preserve the original generated `app/next-env.d.ts` development routes import outside the commit. Follow-on P3 features remain unchecked; the user switched to this landing-page request before those implementation choices were resolved.

### Dark mode and Google account menu — 2026-10-01

Publication confirmed: web GitHub CI **36766449728** passed; Vercel production **`dpl_HhxzBzL5wpjcBNfYeseHGt37mymF`** is READY on `https://housemate-app-xi.vercel.app`. Stable-origin HTTP returned 200, and public assets include the theme bootstrap, account menu and `select_account` option. The parent main commit records the new web pointer and SPEC; API main remains unchanged by these features.

The user requested, in order, a dark mode toggle beside the profile icon, a clickable icon with Google account switching, then commits/pushes to main. These web features are implemented and published as app main **`c984146`**. `ThemeProvider` and a static pre-paint bootstrap apply the device preference until an explicit light/dark choice is saved in browser storage. The toggle is beside the profile control and on sign-in. Media/storage events synchronize system preferences and cross-tab overrides; blocked storage still allows the current-page toggle. Dark tokens and legacy surface/text overrides cover navigation, forms, cards and errors. No new dependency was added.

`AccountMenu` uses the existing Base UI menu for current name/email, Profile, Switch Google account and local Sign out, with keyboard controls, focus return and visible errors. Google switching requires the existing enabled provider/environment flag and sends `prompt=select_account` through Supabase OAuth, without signing out the current session before success. Open drafts block switching/sign-out, including a draft opened during asynchronous push cleanup. Before navigation it attempts the existing push-device cleanup, clears previous household/invite selection and returns to the approved root origin. Account-ID-keyed `AccountView` immediately unmounts previous household views/drafts on session changes, including cross-tab changes; stale responses cannot replace the new account. Callback cancellation errors remain visible even when the old session restores. This is one active browser session shared across tabs, not multiple stored Google sessions or identity linking. API/auth authorization and financial records are unchanged.

Verification: web lint, typecheck, production build and **70 tests in 19 files** passed. New coverage checks pre-paint theme boot, device/default/explicit persistence, cross-tab sync, blocked storage, keyboard/pointer menu interaction, disabled provider/error UI, Google chooser options without premature sign-out, draft races, immediate account isolation and late old-account reads. Palette checks include dark foreground/muted/error/primary pairs at 4.5:1. README, SPEC and `app/QUALITY_CHECKS.md` document behavior and remaining actual-provider/device acceptance. Production browser automation could not initialize (`failed to write kernel assets`, OS error 3); signed-in rendered acceptance and a real two-Google-account switch remain unverified. Do not bypass earlier browser-access denials. Preserve the original local generated `app/next-env.d.ts` development routes import, excluded from publication. The reliability/backup decisions below remain pending; this request did not grant restore-folder permission or authorize offsite transfer.

### Production reliability — 2026-10-01 (latest)

The user requested production error reporting, uptime checks and a tested backup/restore procedure. Sentry SDK 11.1.0 is deployed independently in API and web; errors are rebuilt from an allowlist that removes raw exception text, request/user/household data, tokens, breadcrumbs and local variables. Tracing, replay and log uploads are disabled. No private source-map build token was supplied, so uploads are disabled. Normal validation/permission errors are not reported. Existing confirmation/idempotency behavior remains intact. `GET /health/ready` is now documented in OpenAPI and runs a bounded read-only database probe, returning only status with no-store.

The user saved both write-only DSNs in ignored `api/.env.monitoring` (`SENTRY_API_DSN`, `SENTRY_WEB_DSN`). They were configured as production API `SENTRY_DSN`, web `SENTRY_DSN`, and public web `NEXT_PUBLIC_SENTRY_DSN` (Vercel CLI requires explicit `--type config` for that public variable). Both Sentry projects accepted synthetic redacted verification events with HTTP 200: API `7b5fe0e381d7480c80153884640f40bc`, web `b1554d6003314395acac0d34b1a96113`. The deployed browser assets include the web DSN/privacy filter. A real browser exception, owner issue/regression alert configuration and email delivery remain unverified. Do not print DSNs or introduce a public crash endpoint. The user-requested Sentry agent plugin installed successfully with `npx @sentry/agent-plugin install dreamyday#1bc2ed03ee --no-interactive --no-telemetry`; restart Codex to expose its tools, without restarting or redoing this work merely for installation.

Production API code commit `666675643ee188ee97a69a62f6892f691a4c5642` passed GitHub CI `36758894120` and deployed READY as `dpl_G9SXaQ1XApSZnMDPo4PxQqVREWmd`. A follow-up API operations evidence commit is included in the current submodule pointer. Web monitoring commit `9e908a6ecf122aa255cb1fc821e14ca437e385f5` deployed successfully but clean CI found missing optional WASM lock entries. Repair `f3a9bfcd2d025a51a7d4a636af0e0c65c642ca1b` adds only the two missing entries: CI `36760142771` passed and production `dpl_B9ZnziKAKvdaeoxsNeSbcGTQ8ZNk` is READY with stable alias `https://housemate-app-xi.vercel.app`. API stable origin remains `https://housemate-api.vercel.app`. Both projects passed affected lint/typecheck/tests/build locally and in clean CI (50 API tests, 58 web tests in 15 files). Child and parent credentials were checked before publication. No migration or paid upgrade occurred. Preserve the local generated web development type path difference.

Uptime jobs **8546966** (API `/health/ready`) and **8546967** (web `/`) are enabled every five minutes via the existing cron-job.org account. They attach no authentication, have a 30-second timeout, treat redirects as failures, store no response and enable owner-email failure/recovery/disablement/certificate alerts. Initial actual executions on 2026-10-01 at 01:35:02 Asia/Bangkok returned HTTP 200 in 208 ms / 434 ms. Alert settings are verified; delivery is not. `api/scripts/uptime.mjs` / `npm run uptime:plan|uptime:configure|uptime:status` provide repeatable setup/status. Status uses four management calls; the provider quota is 100/day, with the existing hourly reminder monitor already using about 48/day. Do not poll frequently or create duplicate monitors. Reminder job **8546464** and its existing Codex heartbeat are unchanged.

Backup tooling `api/scripts/backup.mjs` uses a strict-TLS, read-only exported snapshot and AES-256-GCM before saving, including 65 tables across HouseMate/Auth/Storage metadata. Supabase Free and PostgreSQL 17.6.1.166 were verified; Storage has zero objects. Official EDB portable PostgreSQL 17.11 tools are ignored under `api/.run/tools/pg17/pgsql/bin`. Real encrypted production backup: `api/.run/backups/housemate-2026-09-30T17-46-40-087Z.hmbak`. Private recovery key: `api/.run/keys/backup.key`. Never print, commit or transfer these without authorization. The actual isolated custom-archive restore completed at 2026-09-30T17:51:58.203Z in 17 seconds, all 65 counts/content digests, Auth references, private RLS and three dashboard/financial edit/void smoke checks passed. Evidence: `api/.run/restore-drills/bf06c38d-e4c9-4014-8132-6e31a5bea47d/report.json`; its server stopped. No production writes or restore occurred.

An additional phase exercising the generated recovery SQL locally was added after that passing drill and is **not yet verified**. Tightening Windows ACLs on key, backup and restore directories to ADMIN/SYSTEM/Administrators removed access required by the Codex restricted PostgreSQL process. The latest attempt (`ebb35b00-0985-48f5-b53a-28834515f735`) could not open its log; no server is running. Automatic approval review rejected recursively restoring inherited permissions because it could broaden access to private data. A pending user question asks approval to grant this computer's `MSI\CodexSandboxUsers` access **only to `api/.run/restore-drills`**, leaving keys/backups restricted. Do not bypass that rejection or grant access before approval. If approved, validate the exact absolute workspace path, grant only the approved group Modify with child inheritance on that directory, rerun the existing backup's drill, record phase-two evidence, verify the server stopped and remove exact temporary private cluster data after retaining sanitized evidence. If declined, retain the valid initial drill result and label recovery SQL untested.

`api/OPERATIONS.md` contains setup, monitoring, encrypted backup commands and a new managed Supabase recovery runbook. A local drill does not establish Supabase Auth/OAuth/email delivery or managed-project traffic cutover. Storage bytes, platform settings/provider/JWT/Vault keys, custom role passwords and Edge Functions need separate recovery if introduced. Offsite backup storage and unattended scheduling are not configured; recovery point is this manual backup. A separate pending optional question asks whether to choose an offsite destination later or use Google Drive. Do not upload private backups or keys until the user chooses/authorizes the destination. Continue pending recovery permission/offsite decisions and alert acceptance; do not redo deployed monitoring, initial backup, scheduler or prior product features.

### External reminder scheduler — configured and verified

The user authorized external reminder scheduling and failed-run monitoring, created a cron-job.org account/API key, and saved the key in ignored `api/.env.scheduler`. Automatic approval review initially rejected sending production CRON_SECRET to that service. The user then explicitly approved storing the existing secret from ignored `api/.env.vercel` as the private scheduler Authorization header and activating every-minute execution with owner-email alerts. Do not request this same approval again or display either key.

Job **8546464**, titled **HouseMate notifications**, is enabled at `https://housemate-api.vercel.app/internal/notifications/run`: GET, every minute, `Asia/Bangkok`, no expiry, 30-second timeout, redirects classified as failures. Read-back verified the exact saved authorization and settings. Notifications use only the account-email channel (mode 2, selected channel 0), on the first failure, recovery after failure, automatic disablement, and certificate expiry. Four initial actual scheduled executions returned HTTP 200 at 17:04:07, 17:05:02, 17:06:01 and 17:07:01 UTC on 2026-09-30 (duration 226–698 ms). Native email settings are verified; no synthetic production failure was introduced and actual email delivery is untested. Live web push remains disabled, so the worker currently creates in-app reminders.

`api/scripts/external-scheduler.mjs` and `npm run scheduler:plan|scheduler:configure|scheduler:status` provide repeatable setup and sanitized read-only monitoring (`scheduler:status -- --history` adds recent HTTP status/duration, no private headers/body). Configuration lists jobs before creating, stops on duplicate or partial inventory, tests the production endpoint, saves settings while disabled, verifies authorization, then enables and rechecks. Reruns reuse an existing URL-matching job. `.run/external-scheduler.json` contains only job metadata. The private key lives only in ignored `.env.scheduler`; CRON_SECRET remains in ignored `.env.vercel` and the configured cloud systems. No database credentials were transferred to the scheduler. Six focused tests cover duplicate/partial inventory, repeatable setup, incorrect saved credentials, interrupted create, redacted errors, stale/failed/disabled status. API lint/typecheck/build and all **46 tests** (40 existing + six scheduler tests) passed. API main `463b38a25bba08d3956fbf559b7eeddfe2f2307a` is pushed; GitHub CI `36749483874` passed and production deployment `dpl_uY3p167DNfCi7yiFpwDfr1EyGwtc` is READY. README and DEPLOYMENT explain setup/recovery and the management API's 100 requests/day default; do not poll it frequently.

The Codex heartbeat **housemate-reminder-monitor** is ACTIVE hourly in this chat. It runs the status command read-only, stays quiet when healthy/unchanged, and reports new failures/stale/disabled/configuration drift/inaccessible monitoring, plus recovery once. It is supplemental to native account-email alerts, which operate independently of Codex/the computer. Read existing automation before updating it; do not create duplicates. Do not invoke the worker during read-only monitoring or automatically change credentials/jobs.

Local disk exhaustion briefly blocked npm/Git writes. Only verified generated caches `.publication/npm-cache` and `api/.npm-cache` inside the workspace were removed; source, dependencies, private configuration and the app's generated local modification were retained. Around 1.1 GB was freed. Scheduler execution continued in the cloud during cleanup. Do not repeat cache cleanup or scheduler creation.

### Current verified deployment state — 2026-09-30

The popup feature is merged and the API/web are now deployed to HTTPS. Do not redo the merge or create duplicate Vercel projects. Use the CLI authenticated as **heinzaw-bit** in team **DreamyDay**, slug `dreamy-day`, ID `team_TdQ2vprmfRQjAQwhf4QulXK8` (Hobby). The connected-app Vercel tools still have the wrong team scope; CLI access works. The user requested `https://vercel.com/get-started.md`; its setup was followed with Vercel CLI 61.1.0 installed/authenticated and shared Codex MCP server `https://mcp.vercel.com` added with OAuth and tool approval set to prompt. This ongoing session does not yet expose that newly added server's tools. The user granted Vercel's GitHub app access to both private child repositories.

| Service | Project ID | Stable origin | Published main commit | Successful deployment |
| --- | --- | --- | --- | --- |
| API | `prj_73mCu0HTg2ylW0waEkCG7BnheHiB` | `https://housemate-api.vercel.app` | `463b38a25bba08d3956fbf559b7eeddfe2f2307a` | `dpl_uY3p167DNfCi7yiFpwDfr1EyGwtc` |
| Web | `prj_KXOTSWMF5NBOccE8cAI5hig7xuvs` | `https://housemate-app-xi.vercel.app` | `b35dc02e3032f95094199505bf9e70a94cc05dd3` | `dpl_2kKLyo1c2pUrRfphGs4rA2Utrd8X` |

Both projects are linked to the matching `heinzaw-bit` GitHub repositories, repository-root builds, and production `main`. Both use Node.js 24 and Mumbai (`bom1`). The API has Fluid compute and project `resourceConfig.functionDefaultTimeout=120`. Automatic approval review initially blocked transferring the private database/job credentials; the user then explicitly approved uploading existing DATABASE_URL and CRON_SECRET from ignored `api/.env.vercel` to DreamyDay's API production environment. Those two variables are sensitive and production-only; remaining API variables are SUPABASE_URL, SUPABASE_PUBLISHABLE_KEY and exact WEB_ORIGINS. Web variables are only NEXT_PUBLIC_API_URL, NEXT_PUBLIC_SUPABASE_URL, NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY and NEXT_PUBLIC_GOOGLE_AUTH_ENABLED. No private key is a client variable, no preview database credentials were configured, and live push remains disabled. Prepared `.env.vercel` now has the real web origin; local development settings were retained. Child `.vercel` metadata and CLI-generated OIDC tokens are ignored; duplicate CLI-added ignore rules were removed, preserving `.env.example` handling.

Three concrete deployment issues were resolved and published on API main: `functions["src/app.ts"]` failed Vercel's matching stage, so the 120-second limit moved to project settings; the builder required a direct Fastify import in the detected entry point, so the factory is passed through shared runtime/application wiring; top-level `await app.listen()` hung Vercel's intercepted listener during module loading, so startup now starts the listener without awaiting its promise and handles failures with a generic log. Routes, authorization and business rules retain their original shared implementation. API lint/typecheck/build and all 40 tests passed; latest GitHub CI `36734971012` passed. `api/DEPLOYMENT.md` documents these requirements. Do not restore the failed timeout pattern or awaited listener.

Verified live: web Home/Money/House/Notifications/Profile, manifest, service worker, offline fallback and declared icons return 200; manifest uses standalone mode/root scope. API health returns 200; unauthenticated identity/household/job reads return 401; preflight allows exactly the production web origin and does not allow an unlisted origin. One job request with the private Authorization header returned 200, exercising the deployed database/reminder worker. No migration/reset ran. Both deployments are READY with stable aliases assigned. This is not signed-in or actual-device acceptance.

Supabase Auth configuration cannot be edited with the exposed connector tools, and the browser tool failed to initialize. The user reported saving Site URL `https://housemate-app-xi.vercel.app` and exact root redirect URLs. Retain existing localhost redirect URLs; the user asked whether to remove them and was told to keep them for development. A pending question asks the user to open the deployed app, sign in and confirm existing household data loads. Incorporate that response; do not claim the signed-in flow passed until confirmed. The next independent deployment task is the selected external minute-level scheduler; no scheduler account/job exists yet. Live web push still awaits a real contact that the user currently lacks. Actual Android Chrome/iPhone Safari acceptance remains pending in `app/QUALITY_CHECKS.md`. Preserve the only local app source difference, generated `next-env.d.ts` development type paths.

### Earlier deployment preparation and resolved connection attempts

Latest request: the user authorized merging the popup feature and deploying API/web to HTTPS with origins and sign-in redirects configured. This supersedes the earlier preference to perform deployment manually. On 2026-09-30 the verified popup feature was fast-forward merged and pushed to both app and whole-project `main`; web main is `b35dc02e3032f95094199505bf9e70a94cc05dd3`, and the whole-project pins that revision. Feature GitHub CI passed. API stays at `d02be66a985fa2f84662bfbea0a8ef33567ab855`. The only remaining local app modification is Next.js-generated `next-env.d.ts` pointing to development type paths; it was preserved and is not part of the published code. Supabase connector confirmed project `avbroprkgyrkophcxvtm` is ACTIVE_HEALTHY. Plugin discovery found Vercel available but not installed/connected; a connection suggestion and request to reply "connected" were sent. No Vercel deployment, production domain, origin update, or Auth redirect change has happened yet. Continue this same deployment request once Vercel is connected, inspect actual project/team access and assigned domains, deploy the two independent GitHub repositories, then configure exact production origins and Supabase Auth Site URL/Redirect URLs while retaining localhost development redirects. Do not invent deployment URLs or request passwords/tokens in chat.

Connection update: the user replied "connected" and Vercel tools are now callable. `list_teams` returned no teams; `list_projects` with personal-account scope returned no projects. Available tools do not expose `get_git_deployment_context`, `create_git_project`, or environment-variable configuration. A generic `deploy_to_vercel({})` request was rejected by automatic approval review because target/environment were unspecified and it could publish the umbrella checkout; no deployment ran. Do not retry an ambiguous root deployment. The user was asked what their Vercel dashboard shows (account setup versus team/account name). Continue from that answer, verify the exact API and web targets and production configuration before publishing, and preserve the local generated file. The connector is confirmed connected; do not repeat the earlier plugin connection request.

Team access check: the user named **DreamyDay**. `list_projects({teamId:"DreamyDay"})` returned **403 Forbidden / Not authorized**, and a repeat team inventory is empty. Team access is unverified; the user needs to reconnect Vercel with access to the intended DreamyDay team and provide its actual slug if the display name differs. The Vercel deployment skill directs reauthentication on this error. No supported publish-button reconnection tool is exposed, so ask the user to reconnect in plugin settings; do not uninstall the plugin, request tokens in chat, or infer an account/plan restriction from this error. Both main branches are already merged/pushed and web main CI passed (`36727434902`); don't redo the merge. HTTPS deployment and production CORS/Auth configuration remain pending.

The user's latest request adds in-app popups on new branch `codex/in-app-notification-popups` in the web and whole-project repositories. Web changes use the existing authorized notification feed and current category preferences; no API contract, migration, or database change is needed. The bell establishes a quiet first-load baseline, reads cursor pages through a timestamp/ID boundary (including same-millisecond ties), and commits deduplication metadata only after a successful refresh. Unread new announcements, chore assignments/reassignments, and added shopping tasks show at most three popups with exact source links and dismissal; excess notifications remain in the inbox. Popups close after eight seconds with hover/focus/hidden-tab pauses and never take focus. Dismissal does not mark read. Account/household metadata lives only in tab memory; pending responses are ignored on unmount, and authorization failures clear visible popups. See `app/components/notification-popups.tsx`, `notification-bell.tsx`, `app/lib/notification-popups.ts`, and focused tests in `app/tests/notification-popups.test.tsx`. Web lint, typecheck, all **57 tests in 14 files**, and production build pass, including seven new tests covering preferences/read state, dismissal timing, deduplication/remounts, cursor ties/retry, household/account isolation, access loss, offline/hidden polling, and burst limits. Signed-in rendered/device acceptance remains unverified. Manual deployment remains pending.

The three-repository setup is implemented under **heinzaw-bit**: whole-project `housemate`, API `housemate-api`, and web `housemate-app`. API and app are Git submodules, with source changes committed and pushed in the child first and parent pointers committed afterward. Use `git clone --recurse-submodules https://github.com/heinzaw-bit/housemate.git` for a fresh complete checkout. The user wants to deploy manually and selected **Vercel Hobby + external scheduler**. The API code preparation is implemented: `src/app.ts` is Vercel's detected Fastify listener, `src/application.ts` retains the injectable route factory exactly, `src/runtime.ts` shares production wiring with local `src/main.ts`, and Vercel runs no background interval. `GET /internal/notifications/run` requires server-only CRON_SECRET, returns no household data, rejects member JWTs, and uses existing reminder keys/queue leases. Vercel requires the secret and explicit WEB_ORIGINS, caps each warm instance's database pool at one connection, and enforces certificate/hostname validation. An initial read-only TLS check found SELF_SIGNED_CERT_IN_CHAIN. The user then supplied `C:/Users/ADMIN/Downloads/prod-ca-2021.crt` and transaction-pooler host `aws-0-ap-south-1.pooler.supabase.com`. The unexpired public CA is bundled in `src/supabase-ca.ts` with standard trusted roots; `DATABASE_SSL_CA` can override these for future/custom hosts. A read-only query through port 6543 then verified certificate/hostname-validated connectivity and the existing reminder function. No validation was disabled and no database write/migration ran. Private production values were prepared in ignored `api/.env.vercel` using the existing project/password and generated CRON_SECRET. Replace its WEB_ORIGINS placeholder with the intended/actual deployed web origin; keep these values private and server-only. Local api/.env remains intact. `npm run cron:secret` creates/retains the local private secret without printing it. API lint/typecheck/build and all 40 tests passed; compiled Vercel-entry HTTP smoke passed without a database connection. OpenAPI parsed and all 423 local references resolved. Both runtime preparation and final CA commit d02be66a985fa2f84662bfbea0a8ef33567ab855 passed GitHub CI, including clean install, lint, typecheck, all 40 tests, and build. The Vercel configuration also validated against the current official schema. The whole-project submodule pointer pins that final API commit. Follow `api/DEPLOYMENT.md` for manual deployment and external scheduler setup, including its 30-second execution timeout and future push-duration monitoring. No Vercel deployment or external scheduler job has been created. The existing local service may still run the pre-change build; restart it before testing new routes on port 3001. Keep credentials server-only, set web/API origins, configure Supabase Auth redirects, and complete production/device acceptance after deployment.

Sections 10, 11, and P2 first-release web quality implementation are complete. Read the P2 section above for changes, 50 passing web tests, production bundle measurements, and limits. The user selected both Android Chrome and iPhone Safari; actual installation/offline/update, signed-in rendered accessibility/200% zoom/screen-reader, and mid-range phone runtime checks remain unchecked in the manual release acceptance list and `app/QUALITY_CHECKS.md`. Earlier browser access was denied; do not bypass it. The local dev app remains at port 3000 and API health returns 200 at 3001. Service-worker behavior runs only in production. Live push still awaits a real public contact the user currently lacks; do not ask again or invent one. Do not redo completed communications, notifications, financial correction, or P2 implementation. Review the latest request and continue from this state.
