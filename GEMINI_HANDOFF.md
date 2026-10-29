# MediGuard — Implementation Handoff

Use this file as the implementation prompt for a coding agent.

## Repository facts from the initial audit
- Frontend lives in `project/`.
- React 18 + TypeScript + Vite 7; React Router, Tailwind, Lucide, Recharts, jsPDF.
- Supabase JS dependency is installed, but inspected `DataContext.tsx` is localStorage/demo-data based.
- `src/types/index.ts` uses legacy dose statuses including `taken` and `not_confirmed`, which conflict with the product spec.
- `deleteMedication` currently removes associated dose records; change to archive semantics.
- `demoData.ts` contains sample profile/medication/contact/article content; keep demo content clearly marked and do not treat it as verified medical content.
- `src/lib/offline.ts` and `public/sw.js` provide basic offline groundwork, not versioned reviewed content packages.
- Tesseract.js is present for client-side OCR. Camera monitoring is a prototype and must remain separate from dose state.
- These are findings from key-file inspection, not a claim that every file or test has been audited.

## Instructions
1. Read `docs/PRODUCT_SPEC.md`, `docs/ARCHITECTURE.md`, `docs/DATABASE_SCHEMA.md`, `docs/API_CONTRACT.md`, and `docs/SECURITY_AND_PRIVACY.md` before editing.
2. Inspect the actual repository, package scripts, current routes, all usages of dose statuses, and all medication deletion paths.
3. Work on a feature branch. Do not overwrite unrelated user work.
4. Start by reporting the current build/typecheck/test baseline; do not silently fix unrelated errors.
5. Implement the core milestone in small reviewable changes: Supabase setup, schema/RLS, auth, medication CRUD/archive, schedule versions, dose events/history.
6. Never put service-role credentials in the frontend. Do not claim RLS exists until SQL policies and tests have been added.
7. Keep all medical content out of generated AI answers. No clinical advice, prescription inference, or schedule inference.
8. Every OCR-derived schedule requires explicit user confirmation. Camera output is an unverified observation, never ingestion confirmation.
9. Notification open/dismiss never changes dose state. Only explicit user actions do.
10. Do not delete history when archiving medications. Use append-only dose event history.
11. Do not implement caregiver sharing, escalation, facility claims, or AI medical Q&A in this milestone.
12. After each step, run relevant checks and report files changed, commands run, results, assumptions, and remaining risks.

## Expected first deliverables
- Domain type/migration proposal.
- SQL migration with core tables, constraints, RLS policies, and test plan.
- `src/lib/supabase.ts` using public environment variables only.
- Service modules separated from `DataContext`.
- Auth + medication CRUD + archive + dose history, integrated incrementally.
- Tests for cross-user access, dose transitions, idempotency, and history preservation.
