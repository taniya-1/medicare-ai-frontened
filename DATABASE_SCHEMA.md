# MediGuard — Proposed Database Schema

This is a proposed first migration, not an assertion that tables already exist. Use UUID primary keys, UTC `timestamptz` for instants, and explicit timezone identifiers for local scheduling.

## Core tables

### `profiles`
- `id uuid primary key references auth.users(id) on delete cascade`
- `display_name text`
- `timezone text not null default 'Asia/Kolkata'`
- `locale text not null default 'en'`
- `created_at`, `updated_at timestamptz not null`
- No clinical profile or clinician-verification claim.

### `emergency_contacts`
- `id uuid primary key`
- `user_id uuid not null references auth.users(id)`
- `display_name text not null`
- `relationship_label text`
- `phone_e164 text not null`
- `allow_sms_link boolean not null default false`
- `created_at`, `updated_at timestamptz not null`
- Consent/escalation settings should be separate, explicit, and revocable.

### `medications`
- `id uuid primary key`
- `user_id uuid not null references auth.users(id)`
- `display_name text not null`
- `strength_text text` and `form_text text` (user-entered strings; do not parse into clinical recommendations)
- `entry_source text not null check (entry_source in ('manual','ocr_confirmed'))`
- `label text not null default 'User-entered'`
- `archived_at timestamptz`
- `created_at`, `updated_at timestamptz not null`
- Do not hard-delete when dose history exists.

### `medication_schedule_versions`
- `id uuid primary key`
- `medication_id uuid not null references medications(id)`
- `user_id uuid not null references auth.users(id)`
- `version integer not null`
- `schedule_payload jsonb not null` (validated schema; user-confirmed values only)
- `timezone text not null`
- `effective_from timestamptz not null`
- `effective_until timestamptz`
- `user_confirmed_at timestamptz not null`
- `created_at timestamptz not null`
- `unique(medication_id, version)`
- Append new versions instead of overwriting prior instructions.

### `dose_events`
- `id uuid primary key`
- `user_id uuid not null references auth.users(id)`
- `medication_id uuid not null references medications(id)`
- `schedule_version_id uuid references medication_schedule_versions(id)`
- `scheduled_for timestamptz not null`
- `timezone text not null`
- `status text not null check (status in ('scheduled','acknowledged','reported_taken','skipped','missed','cancelled'))`
- `snoozed_until timestamptz`
- `reported_taken_at timestamptz`
- `verification jsonb` nullable (experimental unverified observation only; no images)
- `created_at`, `updated_at timestamptz not null`
- Unique idempotency key/constraint based on schedule version and occurrence identity to prevent duplicate generation.

### `dose_event_history`
- `id uuid primary key`
- `dose_event_id uuid not null references dose_events(id)`
- `user_id uuid not null references auth.users(id)`
- `previous_status text`, `new_status text not null`
- `actor text not null check (actor in ('user','system'))`
- `reason text`
- `created_at timestamptz not null`
- Append-only from the application perspective; never expose update/delete to normal clients.

### `refill_events`
- `id uuid primary key`
- `user_id uuid not null references auth.users(id)`
- `medication_id uuid not null references medications(id)`
- `quantity_delta numeric`
- `event_type text not null check (event_type in ('stock_added','stock_adjusted','refill_reminder'))`
- `occurred_at timestamptz not null`
- `notes text`
- Avoid assuming dose quantity or calculating clinical changes.

## Later tables
- `caregiver_access_grants`: patient, caregiver identity, explicit scopes, expiry, accepted/revoked timestamps, audit metadata.
- `notification_attempts`: dose event, channel, attempt state, timestamps, non-sensitive failure code; never treat delivery/open as a dose action.
- `knowledge_sources`: source organization/title/reference/jurisdiction/retrieval date/trust metadata.
- `knowledge_item_versions`: content, source metadata, version, reviewer, status, review timestamps.
- `content_packages`: jurisdiction, locale, version, content hash, publication identity, status, created/published timestamps.

## RLS requirements
- Enable RLS on every user-owned table.
- Patient-owned rows: `user_id = auth.uid()` for select/insert/update; restrict deletes and prefer archive.
- Schedule versions must be owned by the same user as the medication; enforce via transaction/RPC/trigger and tests, not only UI.
- Dose events must reference medications owned by the same user.
- `dose_event_history` should be readable only by the owning user and insertable only through a controlled function/transaction; ordinary clients cannot update/delete history.
- No public access to patient data. Caregiver access is not implicit and must not be added until scoped grants are implemented and tested.
- Content package publication should be writable only by a dedicated authorized reviewer/service workflow, not normal authenticated users.
- Add indexes for `(user_id, scheduled_for)`, `(user_id, archived_at)`, and relevant foreign keys.

## Retention and deletion
Define a documented account deletion/export process before production. Preserve history when archiving medications. Deletion semantics and backups must be explained to users; legal requirements remain unverified.
