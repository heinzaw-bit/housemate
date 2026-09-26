# DATA_MODEL.md

Use PostgreSQL.

All IDs should be UUIDs.

Every household-scoped table must contain `household_id`.

Use `created_at`, `updated_at` consistently.

## users

- id
- auth_user_id
- display_name
- avatar_url nullable
- timezone
- locale
- created_at
- updated_at

## households

- id
- name
- default_currency
- timezone
- owner_user_id
- created_at
- updated_at
- archived_at nullable

## household_members

- id
- household_id
- user_id
- role: owner | admin | member
- status: invited | active | left | removed
- joined_at nullable
- left_at nullable
- created_at
- updated_at

Unique:
(household_id, user_id)

## household_invites

- id
- household_id
- invited_email nullable
- invite_code
- role
- expires_at
- created_by
- claimed_by nullable
- claimed_at nullable

## expenses

- id
- household_id
- title
- description nullable
- category
- total_amount_minor
- currency
- paid_by_member_id
- expense_date
- source_type: manual | bill | event | shopping | ai
- source_id nullable
- status: active | voided
- created_by_member_id
- created_at
- updated_at

Use integer minor units for normal currency storage where practical.

For calculations requiring intermediate fractions, use PostgreSQL numeric/decimal in calculation tables or deterministic allocation logic.

## expense_participants

- id
- expense_id
- member_id
- included boolean
- final_share_minor
- metadata_json

Unique:
(expense_id, member_id)

## expense_split_configs

- id
- expense_id
- method: equal | exact | percent | ratio | hybrid
- config_json
- calculation_version
- created_at

`config_json` preserves the original rule used to calculate shares.

## expense_adjustments

Used by hybrid splitting.

- id
- expense_id
- member_id nullable
- label
- amount_minor
- adjustment_type: individual_charge | shared_charge | credit | deduction
- created_at

## bills

- id
- expense_id
- household_id
- bill_type
- provider nullable
- billing_period_start nullable
- billing_period_end nullable
- due_date nullable
- bill_status: draft | due | partially_settled | settled | overdue
- recurring_rule_id nullable

## settlements

- id
- household_id
- from_member_id
- to_member_id
- amount_minor
- currency
- settlement_date
- method nullable
- note nullable
- proof_url nullable
- status: active | voided
- created_by_member_id
- created_at
- updated_at

## meal_events

- id
- household_id
- title
- meal_type
- starts_at
- response_deadline nullable
- created_by_member_id
- status
- created_at
- updated_at

## meal_responses

- id
- meal_event_id
- member_id
- response: join | not_join | maybe
- responded_at

Unique:
(meal_event_id, member_id)

## cooking_assignments

- id
- household_id
- meal_event_id nullable
- assignment_date
- role_name
- assigned_member_id
- status: assigned | completed | swapped | skipped
- notes nullable
- created_at
- updated_at

## shopping_lists

- id
- household_id
- title
- status
- created_at
- updated_at

MVP can default to one active household list.

## shopping_items

- id
- shopping_list_id
- title
- quantity nullable
- unit nullable
- priority: low | normal | high
- assigned_member_id nullable
- added_by_member_id
- status: needed | assigned | purchased | cancelled
- estimated_amount_minor nullable
- purchased_amount_minor nullable
- linked_expense_id nullable
- created_at
- updated_at

## chores

- id
- household_id
- title
- instructions nullable
- recurrence_rule_json nullable
- default_assignee_member_id nullable
- reminder_rule_json nullable
- active boolean
- created_by_member_id
- created_at
- updated_at

## chore_instances

- id
- chore_id
- due_at
- assigned_member_id
- status: pending | completed | skipped | swap_requested
- completed_at nullable
- completion_note nullable
- created_at
- updated_at

## chore_swap_requests

- id
- chore_instance_id
- requested_by_member_id
- target_member_id nullable
- status: open | accepted | rejected | cancelled
- created_at
- resolved_at nullable

## events

- id
- household_id
- title
- event_type
- starts_at
- ends_at nullable
- description nullable
- created_by_member_id
- created_at
- updated_at

## event_attendees

- id
- event_id
- member_id
- response: join | not_join | maybe | no_response
- responded_at nullable

## announcements

- id
- household_id
- title
- body
- priority: normal | important | urgent
- requires_acknowledgement boolean
- expires_at nullable
- published_by_member_id
- published_at
- created_at
- updated_at

## announcement_acknowledgements

- id
- announcement_id
- member_id
- acknowledged_at

## polls

- id
- household_id
- question
- allow_multiple boolean
- deadline nullable
- created_by_member_id
- created_at
- updated_at

## poll_options

- id
- poll_id
- label
- position

## poll_votes

- id
- poll_id
- poll_option_id
- member_id
- created_at

Enforce uniqueness appropriately for single-choice polls.

## notifications

- id
- household_id
- recipient_member_id
- category
- title
- body
- priority
- deep_link nullable
- source_type
- source_id nullable
- status: pending | sent | failed | read
- created_at
- sent_at nullable
- read_at nullable

## notification_preferences

- id
- member_id
- category
- in_app_enabled
- web_push_enabled
- line_enabled
- email_enabled
- sms_enabled
- quiet_hours_json nullable

## push_subscriptions

- id
- user_id
- endpoint
- p256dh
- auth
- user_agent nullable
- last_used_at
- created_at

## integrations

- id
- household_id
- provider: line | email | sms
- encrypted_config_json
- status
- created_at
- updated_at

## ai_conversations

- id
- household_id
- member_id
- created_at
- updated_at

## ai_messages

- id
- conversation_id
- role
- content
- structured_payload_json nullable
- created_at

## ai_action_drafts

- id
- household_id
- requested_by_member_id
- action_type
- parsed_payload_json
- validation_status
- confirmation_status: pending | confirmed | cancelled | expired
- execution_result_json nullable
- created_at
- confirmed_at nullable
- executed_at nullable

## audit_log

- id
- household_id
- actor_member_id nullable
- action_type
- entity_type
- entity_id
- before_json nullable
- after_json nullable
- source: user | ai_confirmed | system
- created_at

## Important constraints

1. Never derive household access from client-supplied household IDs alone.
2. Always verify membership server-side.
3. Financial rows should not be hard-deleted during normal usage.
4. Preserve split configuration used at calculation time.
5. Recalculating an edited expense must produce a new audit entry.
6. Settlement records must remain independent of expense records.
