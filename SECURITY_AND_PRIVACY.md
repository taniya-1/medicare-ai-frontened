# MediGuard — Security and Privacy Baseline

## Threat model
Sensitive assets include medication names/schedules, dose history, emergency contacts, account identifiers, access tokens, and optional caregiver relationships. Threats include cross-account access, leaked client secrets, overbroad caregiver access, logs containing health data, compromised dependencies, malicious package updates, and accidental loss of offline emergency information.

## Required controls
1. Enable RLS on every user-owned table and test both allowed and denied cross-user access.
2. Keep only Supabase URL and anon/publishable key in the browser. Never expose service-role keys or other secrets in `VITE_*`, source control, logs, or browser storage.
3. Validate all input server-side where security matters. Client-side validation improves UX but is not authorization.
4. Use least privilege for Edge Functions and publication workflows; authenticate caller and authorize the requested record/scope.
5. Avoid logging request bodies or medical data. Redact authorization headers, tokens, phone numbers, and user-provided notes.
6. Require explicit, revocable consent before any future caregiver sharing or external AI/provider processing.
7. Do not upload camera frames. Camera analysis is optional, on-device, and returns only a separately stored unverified observation.
8. Do not persist OCR source images unless a separately reviewed requirement is added; user must confirm extracted text.
9. Treat all medication entries as “User-entered.” No clinician verification claim in MVP.
10. Keep approved emergency content available offline; validate package schema/hash and preserve last known-good package on failed updates.
11. Apply dependency updates, secret scanning, and lockfile review. Verify `.env` is ignored and no secrets have been committed.
12. Implement account export/deletion and explain retention before production use.

## Authentication and session
- Use Supabase Auth-supported session handling; do not build a custom token store.
- Sign-out should clear authenticated application state while leaving only the minimal offline emergency package/contact data that the user knowingly opted to retain.
- Avoid caching authenticated API responses in the service worker cache.
- Do not place private data in URL query strings.

## Logging and analytics
- Log operation identifiers, coarse outcomes, and non-sensitive error codes only.
- Never log PHI, medication schedules, access tokens, service keys, raw OCR text, camera frames, or emergency contact details.
- Analytics must be optional and must not contain health data by default.

## Release gate
Before any public/real-user deployment, review RLS policies, test account isolation, inspect build output for secrets, test offline access, test export/deletion, review third-party data flows, and obtain appropriate privacy/legal review. Compliance status is currently unverified; do not claim regulatory compliance.
