# HouseMate

HouseMate is an app for people sharing a household to coordinate expenses, bills, settlements, meals, shopping, chores, events, announcements, and reminders. It supports households using either USD or Thai baht. Payments are recorded manually; HouseMate does not move money.

## Current workspace

| Folder | What is here |
| --- | --- |
| [`api/`](api/README.md) | Node.js/TypeScript API, Supabase Auth, PostgreSQL migrations, and OpenAPI contract. Household, money, daily work, events, announcements, polls, personal notifications, preferences, and a web push queue are implemented. |
| [`app/`](app/README.md) | Next.js web app with sign-in, household workflows, money, daily work, events, announcements, polls, notification inbox/settings, and explicit browser push opt-in. |
| `mobile/` | Planned React Native/Expo app; it has not been created yet. |

The API and web app are independent Git repositories with their own packages, lockfiles, and CI checks. This whole-project repository includes them as Git submodules. Each folder's README has its own details and check commands.

## GitHub repositories

| Repository | Purpose |
| --- | --- |
| [heinzaw-bit/housemate](https://github.com/heinzaw-bit/housemate) | Shared product documents, engineering boundaries, handoff, and pinned API/app revisions. |
| [heinzaw-bit/housemate-api](https://github.com/heinzaw-bit/housemate-api) | Independently buildable and releasable API. |
| [heinzaw-bit/housemate-app](https://github.com/heinzaw-bit/housemate-app) | Independently buildable and releasable web app. |

Clone the whole project, including its pinned API and app versions:

```powershell
git clone --recurse-submodules https://github.com/heinzaw-bit/housemate.git
```

For an existing clone, run `git submodule update --init --recursive`. The repositories are private; authenticate as an account with access. Either child repository can also be cloned on its own, with no sibling runtime dependency.

### Current development checkout on D:

The current Board and question-only Assistant work is published on `codex/house-announcements-events` in the project, web and API repositories. Clone that branch to include the latest development work:

```powershell
git clone --branch codex/house-announcements-events --recurse-submodules https://github.com/heinzaw-bit/housemate.git D:\house_mate
cd D:\house_mate
git -C app switch codex/house-announcements-events
git -C api switch codex/house-announcements-events
```

GitHub contains source and shared documents. Private `.env` files, local TLS certificates, backup recovery keys, provider metadata and local attachments are excluded. Transfer needed configuration directly between your local checkouts and keep recovery keys protected; never commit them. Install dependencies independently with `npm ci` in `api` and `app`. Existing configured Supabase migrations do not need to be reapplied when moving this checkout. Keep the original directory until the fresh clone and local configuration are verified.

When changing API or web code, commit and push from that child repository first. Then commit the updated `api` or `app` submodule pointer and any shared documents in the whole-project repository and push it. `git submodule update --init --recursive` checks out the recorded versions, which may use a detached HEAD; switch the affected child to `main` before making new changes. Shared product documents remain here, and each child includes its own engineering instructions.

## Run locally

You need Node.js 22 or newer, npm, and a Supabase project. The API needs a PostgreSQL connection URL; the web app needs only the Supabase project URL and publishable key. Keep the database URL and any secret keys out of the web app.

1. In `api/`, run `npm ci`, copy `.env.example` to `.env`, and fill in `DATABASE_URL`, `SUPABASE_URL`, and `SUPABASE_PUBLISHABLE_KEY`. Set `WEB_ORIGINS=http://localhost:3000` for local web development. Apply `api/sql/001_household_foundation.sql` and then `api/sql/002_member_onboarding.sql` if they have not already been applied; run `npm run migrate:expenses`, `npm run migrate:bills`, `npm run migrate:settlements`, `npm run migrate:edits`, `npm run migrate:voids`, `npm run migrate:meals`, `npm run migrate:shopping`, `npm run migrate:shopping-currency`, `npm run migrate:chores`, `npm run migrate:events`, `npm run migrate:announcements-polls`, `npm run migrate:notifications`, and `npm run migrate:assistant` in order. See the [API setup guide](api/README.md).
2. In `app/`, run `npm ci`, copy `.env.example` to `.env.local`, and fill in `NEXT_PUBLIC_API_URL`, `NEXT_PUBLIC_SUPABASE_URL`, and `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY`. For local development, the API URL is `http://localhost:3001`. See the [web app setup guide](app/README.md).
3. Add `http://localhost:3000` to the Supabase Auth redirect allow list. In one terminal run `npm run dev` from `api/`; in another run `npm run dev` from `app/`.
4. Open [the web app](http://localhost:3000). The [design preview](http://localhost:3000/preview) works in development without sign-in and uses sample data. The [API health route](http://localhost:3001/health) checks that the API is running.

The design preview is for browsing the interface. Household creation and live data require configured Supabase Auth, the API, and the database migrations.

Assistant now answers household questions with source links and checked timestamps. Quick questions about balances, bills, chores, shopping, meals, events, and the household overview work without AI configuration. Optional general free text uses Gemini’s free tier with a private API key, operator billing-tier confirmation and a member privacy acknowledgement; see the [assistant setup guide](api/README.md#optional-natural-language-questions). Action drafting is the next phase and remains pending.

The bell opens the personal notification inbox and settings. Keep the API running for scheduled reminders. Optional live web push needs server-only VAPID keys and a real public contact email or HTTPS support URL; it is currently disabled because no contact is available. The inbox, reminders, and category/quiet-hours preferences work independently of push. See the [push setup guide](api/README.md#notifications-and-optional-web-push).

The web app includes Android/iPhone installation guidance, a public offline fallback, connection/update notices, interrupted financial-confirmation guidance, accessibility improvements, and deferred workspace loading. See [web release checks](app/QUALITY_CHECKS.md) for verification and the remaining actual-phone acceptance checks. PWA service-worker behavior runs in production, rather than the development server.

## Production and deployment

The API and web app are deployed as separate projects in Vercel's **DreamyDay** team on Hobby:

| Service | Production URL |
| --- | --- |
| Web app | [housemate-app-xi.vercel.app](https://housemate-app-xi.vercel.app) |
| API health | [housemate-api.vercel.app/health](https://housemate-api.vercel.app/health) |

Both projects build from their independent repository roots and deploy `main` automatically. The API runs in Mumbai (`bom1`) on Node.js 24 with Fluid compute and a project default function duration of 120 seconds. Production database/job credentials are server-only sensitive API variables. The web uses the API HTTPS origin; API CORS allows the exact web origin. The user confirmed saving the production Supabase Site URL and redirect URLs while retaining localhost for development.

HTTPS routes, install/offline assets, API authentication rejection, CORS, and the reminder worker's database access were verified. The external cron-job.org job **8546464** runs every minute in `Asia/Bangkok`; four initial scheduled executions returned HTTP 200 in under one second. Account-email alerts are enabled on failure, recovery, and automatic disablement. An hourly Codex follow-up checks for failed, stale, disabled, or changed configuration and stays quiet while healthy. Native scheduler email alerts run independently of this computer; actual email delivery has not been tested. See the [deployment guide](api/DEPLOYMENT.md#repeatable-setup-and-status-commands) for status commands and recovery. Signed-in household acceptance and actual Android/iPhone installation/offline/update checks remain pending. Live web push still needs a real public contact.

## Project documents

The [operations guide](api/OPERATIONS.md) covers Sentry integration, API readiness/web
uptime checks, encrypted backups, an isolated restore drill and new-project
recovery steps. Offsite backup storage and unattended scheduling need configuration.

- [Product SPEC](SPEC.md): planned features, behavior, release scope, and open product decisions.
- [Web app to-do](WEB_APP_TODO.md): prioritized checklist for the browser app.
- [Project boundaries](AGENTS.md): durable engineering and security rules for work across all three projects.
- [API contract](api/openapi.yaml): implemented HTTP endpoints and schemas.
- The other root Markdown files are supporting product and domain references. Some describe an earlier web-only architecture; follow the current SPEC and project boundaries when they differ.
