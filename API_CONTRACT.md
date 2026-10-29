# MediGuard — API and Service Contract

## Approach
Use the Supabase JS client for ordinary authenticated CRUD protected by RLS. Use database functions or Edge Functions only where a transaction, trusted scheduling, publication, or privileged operation is needed. Do not create a generic service-role proxy.

## Client services (planned)
- `medications.listActive()`
- `medications.create(input)` — user-entered fields only; never infer schedule.
- `medications.update(id, patch)`
- `medications.archive(id)` — preserve history.
- `medications.createScheduleVersion(id, confirmedSchedule)`
- `doseEvents.list(range)`
- `doseEvents.acknowledge(id)`
- `doseEvents.reportTaken(id, source='manual')`
- `doseEvents.skip(id, reason?)`
- `doseEvents.snooze(id, until)`
- `emergencyContacts.list/create/update/delete`
- `contentPackages.getManifest(jurisdiction, locale)`
- `contentPackages.download(version)` — verify expected hash before storing.

## Dose transition contract
Canonical states: `scheduled`, `acknowledged`, `reported_taken`, `skipped`, `missed`, `cancelled`. Snooze is a separate timestamp. Notification open/dismiss events do not transition a dose.

Suggested allowed transitions:
- `scheduled → acknowledged | reported_taken | skipped | missed | cancelled`
- `acknowledged → reported_taken | skipped | missed | cancelled`
- `missed → reported_taken` only if the product explicitly permits late self-report and the history captures it; otherwise require a correction workflow.
- `reported_taken`, `skipped`, `cancelled` are terminal in the MVP; corrections should be explicit, audited events rather than silent overwrites.
- `missed` is set by a trusted server process only after a documented grace rule. Do not infer that a person did or did not ingest medication.

Every transition writes `dose_event_history` atomically with the current state update. Retry-safe calls use idempotency keys or compare-and-set semantics.

## Error envelope
Return stable, non-clinical error codes such as:
- `UNAUTHENTICATED`
- `FORBIDDEN`
- `VALIDATION_ERROR`
- `NOT_FOUND`
- `CONFLICT`
- `RATE_LIMITED`
- `TEMPORARY_UNAVAILABLE`

Do not include tokens, full medical payloads, or internal secrets in errors/logs.

## Reminders
- Browser/in-app notifications are best-effort and platform-dependent.
- A notification attempt is not proof of delivery, opening, acknowledgement, or ingestion.
- Missed marking is server-side, idempotent, and based on an explicitly documented threshold; do not send clinical advice in escalation messages.
- Caregiver notifications are out of MVP; do not send to a contact without a future accepted grant and explicit scope/consent.

## Offline emergency package
Manifest fields: `package_id`, `jurisdiction`, `locale`, `version`, `published_at`, `content_hash`, `review_status`, `items[]`.
Each item requires: `item_id`, `title`, `body`, `source_organization`, `source_title`, `source_reference`, `publication_or_revision_date`, `retrieved_at`, `jurisdiction`, `version`, `review_status`, `reviewer_id`.
Only packages marked published/approved by the controlled publisher are installable. Validate schema and hash before replacing the last known-good package. Never let an update failure erase the working offline package.

## Emergency contact actions
`tel:` and `sms:` links open the corresponding device application. UI must state that the user still needs to place/send the call/message. Never report that a call was placed or help dispatched.
