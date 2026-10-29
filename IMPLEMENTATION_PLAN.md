# MediGuard — Implementation Plan

## Decisions locked
- Backend: Supabase Auth + Postgres/RLS + Edge Functions as needed.
- First milestone: core account + medication CRUD + dose history.
- Repository approach: docs first; implementation follows approved contracts.
- Existing React/Vite frontend is retained and migrated incrementally.

## Phase 0 — Documentation and baseline
- [x] Capture product safety requirements and scope.
- [x] Document architecture, schema, service contract, privacy/security, and knowledge governance.
- [ ] Verify actual default branch, scripts, current dependency versions, and complete route/type usage before implementation.
- [ ] Run existing build and typecheck; record baseline failures before changing code.
- [ ] Confirm `.env` is ignored and scan for accidentally committed secrets.

## Phase 1 — Domain types and data boundary
- Replace legacy dose vocabulary (`taken`, `not_confirmed`) with the canonical state model through explicit migrations/mappers; do not do a blind global replace.
- Separate camera observation from dose status.
- Define typed domain objects and service interfaces.
- Keep demo data isolated and visibly marked; avoid nondeterministic sample history.
- Preserve medication history through archive rather than cascading delete.

## Phase 2 — Supabase foundation
- Create project and local environment variables.
- Add SQL migrations for core tables, constraints, indexes, RLS policies, and history-writing transaction.
- Test RLS with two users and negative cross-account access cases.
- Add AuthContext and safe session/error handling.
- Never commit credentials or service-role keys.

## Phase 3 — Core milestone
- Medication list/create/edit/archive.
- User-confirmed schedule versions.
- Dose event generation with stable occurrence identity/idempotency.
- Dose event list and explicit acknowledge/report-taken/skip/snooze actions.
- Append-only history and self-reported adherence calculation.
- Loading, empty, error, offline, and permission states.
- Tests for transitions, timezone/DST boundaries, duplicate generation, and authorization.

## Phase 4 — Offline emergency library
- Define package JSON schema and reviewed content metadata.
- Build package manifest/hash validation and IndexedDB storage.
- Preserve last known-good package on update failure.
- Verify emergency contacts and call/message links work offline after initial load.
- Replace or disable unprovenanced demo guidance until reviewed.

## Phase 5 — Reminder reliability
- Document actual notification limitations per browser/platform.
- Add server-side missed marking only after grace period and transition policy are approved.
- Add idempotent notification attempts and rate limiting.
- Caregiver escalation remains out of scope until consent/grants are implemented.

## Phase 6 — Hardening and release review
- RLS/security tests, accessibility, mobile/responsive checks, offline tests, dependency/secret scan.
- Data export and account deletion behavior.
- Privacy notice and consent copy reviewed.
- Clinical content and local emergency configuration reviewed by qualified people.
- Do not claim regulatory compliance without a separate review.

## Definition of done for core milestone
- Authenticated users can manage only their own records.
- A medication can be archived without deleting historical doses.
- Schedules are explicitly user-confirmed and versioned.
- Dose states match the canonical enum and transitions are audited.
- Notification interactions never mark doses taken.
- Camera observations cannot change dose state.
- No real medical guidance is presented without approved provenance.
- Build/typecheck and relevant tests pass, or remaining failures are documented.
