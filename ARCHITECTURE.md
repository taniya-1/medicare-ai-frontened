# MediGuard — Architecture

## Decision record
- **Frontend:** Existing React 18 + TypeScript + Vite app under `project/`; preserve existing UI and migrate incrementally.
- **Backend:** Supabase Auth + PostgreSQL + Row Level Security (RLS) + Edge Functions.
- **Offline store:** IndexedDB for approved content packages and minimal emergency contact data; service worker for app shell/static assets.
- **No separate Node/FastAPI server in the MVP.** Reconsider only if a demonstrated requirement cannot be met safely with Supabase.
- **AI:** No AI-generated clinical guidance in the MVP. A later audit workflow may flag source conflicts for human review.

## Logical components
1. **React UI:** routes and components; forms validate inputs but are not the security boundary.
2. **Domain services:** typed modules for medications, dose events, emergency contacts, and content packages. Avoid making `DataContext` a monolithic backend client.
3. **Supabase client:** browser-safe URL + anon/publishable key only; user JWT is managed by Supabase Auth.
4. **Postgres + RLS:** durable user-owned records; every table exposed to the client has explicit grants and policies.
5. **Edge Functions / scheduled job:** privileged or trusted workflows only, with strict authentication, authorization, idempotency, rate limits, and minimal logs.
6. **Offline emergency library:** previously downloaded, human-approved, versioned package; no network/LLM dependency for reading it.
7. **Content governance (later):** source registry → automated checks/flags → human review → authorized publication → signed/hashed package.

## Data flow
- User signs in → Supabase Auth session → client queries tables through RLS.
- User creates medication/schedule → validated service → RLS-protected rows.
- Scheduler materializes dose events idempotently for a bounded future horizon; local UI notifications are best-effort, not guaranteed background alarms.
- User explicitly reports taken/skipped → append audit event and update current state in a transaction/RPC or Edge Function where needed.
- Emergency package is fetched while online, integrity-checked, stored in IndexedDB, then rendered offline.
- Contact links open device dialer/messaging app; the app does not send or claim dispatch itself.

## Suggested frontend organization
```text
project/src/
  lib/supabase.ts
  services/
    medications.ts
    doseEvents.ts
    emergencyContacts.ts
    contentPackages.ts
  types/
    domain.ts
  context/
    AuthContext.tsx
    DataContext.tsx   # temporary UI adapter; gradually slim down
  features/
    medications/
    dose-history/
    emergency/
    library/
  hooks/
  components/
```

## Migration strategy
1. Add architecture and safety docs before changing runtime behavior.
2. Introduce canonical domain types and explicit mapping from legacy localStorage/demo types.
3. Add Supabase schema, RLS, and policies before wiring production reads/writes.
4. Add auth/profile and core medication CRUD.
5. Add immutable schedule versions and dose event history.
6. Add offline package manifest, integrity validation, and reviewed-content metadata.
7. Migrate screens feature-by-feature; remove mock data only when each screen has a verified replacement.
8. Add automated tests and a staging review before enabling reminders or user data migration.

## Operational constraints
- `VITE_SUPABASE_URL` and `VITE_SUPABASE_ANON_KEY` (or publishable key) are public client configuration, not secrets.
- Never put a Supabase service-role key in Vite variables or client bundles.
- Supabase Edge Functions must verify caller identity and authorization; service-role access bypasses RLS and must be narrowly scoped.
- Client-side timers and browser notifications cannot guarantee delivery while a browser/PWA is closed. Do not market them as guaranteed medication alarms.
- Package hashes detect accidental/tampered bytes only when compared to a trusted expected hash; they do not prove reviewer identity. Publication authorization and transport must be controlled.
