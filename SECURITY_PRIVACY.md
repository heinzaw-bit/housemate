# SECURITY_PRIVACY.md

## Threat model

HouseMate stores:
- household membership,
- expenses,
- payment status,
- household routines,
- messages/announcements,
- optional receipts.

Treat this as private household data.

## Core security requirements

1. Authentication required for household data.
2. Server-side authorization on every household-scoped action.
3. Row-level security where supported.
4. Secrets only on server.
5. Webhook signatures verified.
6. Integration tokens encrypted at rest.
7. Sensitive data minimized.
8. Financial operations audited.
9. AI actions validated.
10. Secure account-linking for messaging integrations.

## Household isolation

A user in Household A must never access Household B through:
- guessed IDs,
- altered URLs,
- API requests,
- realtime subscriptions,
- search,
- AI queries.

Add explicit automated tests.

## Supabase RLS

Use RLS for household-scoped data.

Policy concept:
current authenticated user must have an active membership in row.household_id.

Service role bypass should be restricted to trusted server functions.

## Financial authorization

MVP:
normal members may create expenses.

Editing:
- creator, admin, or allowed participant according to policy.

Voiding:
- creator/admin.

Settlement:
sender should confirm their own outgoing settlement, or household policy must define alternatives.

## Attachments

Receipts:
- private bucket
- signed URLs
- file size limit
- file type allowlist
- malware scanning if product expands

## AI privacy

Do not send unnecessary household history to the LLM.

Use minimum context needed for the request.

For example:
“Who owes electricity?”
requires bill and allocation data,
not complete shopping/chore history.

## AI prompt injection

Retrieved user content is untrusted.

Never allow:
- announcement text,
- shopping text,
- notes,
- receipts

to redefine AI rules or tool permissions.

## AI confirmation

Mutation tools should require:
- authenticated user,
- active household member,
- valid draft,
- authorized action,
- unexpired confirmation,
- idempotency key.

## Messaging integration

Verify provider webhook signature.

Map external sender identity only through verified account link.

Do not allow unknown group participants to execute financial commands.

## Push privacy

Lock-screen notifications can expose data.

Default notification copy should be privacy-conscious.

Example:
“Electricity bill needs your attention.”

Detailed amount appears after app opens.

## Audit log

Must include:
- actor
- action
- entity
- timestamp
- before/after for important edits
- whether source was direct user or AI-confirmed

## Account removal

Define:
- leave household
- remove household member
- delete personal account
- archive household

Financial history involving departed members should remain referentially valid.

Use historical display names if necessary.

## Rate limits

Apply to:
- AI requests
- invite creation
- announcement publishing
- push test requests
- messaging webhooks
- auth endpoints

## CSRF/XSS

Use framework protections.

Never render household-provided HTML unsanitized.

Prefer plain text / sanitized rich text.

## Security tests

- horizontal household access
- role access
- RLS
- invite abuse
- expired invite
- forged LINE webhook
- duplicated webhook
- AI draft reuse
- financial action without confirmation
- attachment authorization
- push subscription ownership
