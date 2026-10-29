# MediGuard — Product Specification

**Status:** Draft v1  
**Primary jurisdiction:** India (`IN`)  
**Default locale:** `en`  
**Default timezone:** `Asia/Kolkata`  
**Product class:** Medication-adherence and offline emergency-information helper; not a clinical system.

## Purpose
Help a person follow their own prescribed medication schedule, keep a self-reported dose history, access reviewed emergency-information packages offline, and optionally share bounded missed-dose alerts with an explicitly authorized caregiver.

## Non-goals
MediGuard is not a clinician, pharmacist, diagnostic tool, emergency dispatcher, medication recommender, or proof of ingestion. No clinician-verification workflow is part of the MVP.

## Safety requirements
- **SAFE-1 — No clinical decisions:** Never diagnose, prescribe, recommend a medication, or tell a user to start, stop, or change medication or dosage.
- **SAFE-2 — Schedule provenance:** Schedules come from user entry or OCR followed by user confirmation. The system never infers a schedule. Ambiguous instructions remain as entered and are flagged to ask a clinician/pharmacist.
- **SAFE-3 — Explicit dose action:** Opening or dismissing a notification never changes dose status. Only an explicit user action can report a dose taken, skipped, or otherwise acknowledged.
- **SAFE-4 — Canonical dose states:** `scheduled`, `acknowledged`, `reported_taken`, `skipped`, `missed`, `cancelled`. Snooze is `snoozed_until`, not a status. Camera output is a separate `verification` observation.
- **SAFE-5 — User-entered label:** User-entered medication information must be labeled “User-entered”; MVP must not claim clinician verification.
- **SAFE-6 — Camera limits:** Camera analysis is optional and experimental. Store an unverified observation and confidence only; never claim ingestion confirmed. Keep frames on-device and do not store them. Manual reporting always remains available.
- **SAFE-7 — Bounded escalation:** Escalation changes who is informed, not medical instructions. It is opt-in, bounded, visible, and revocable.
- **SAFE-8 — Approved content only:** User-facing emergency guidance comes only from published, human-approved, versioned, sourced packages. AI output is never directly shown as guidance.
- **SAFE-9 — Content provenance:** Every published item records source organization, title, URL/reference, publication/revision date, retrieval date, jurisdiction, version, review status, and reviewer.
- **SAFE-10 — Conflict handling:** Conflicting, outdated, or unsupported content is routed to human review. The system never chooses a clinical recommendation.
- **SAFE-11 — Offline access:** After the app and package have loaded once, the emergency library, emergency contacts card, and call/message links must work offline without login, LLM, or paid API access.
- **SAFE-12 — No dispatch claims:** `tel:` and `sms:` links merely open the device app. MediGuard must never claim that help was dispatched.
- **SAFE-13 — Facility uncertainty:** Facility information is approximate/unverified unless supplied by a trusted provider. Never invent opening hours, capacity, or travel time.
- **SAFE-14 — Privacy:** Medical data is sensitive. Obtain consent before caregiver sharing or external AI/provider processing. Never log PHI, access tokens, or secrets.

## MVP scope
1. Account/profile and timezone.
2. Emergency contacts.
3. Medication create/edit/archive; schedule versions; stock and refill events.
4. Dose events and self-reported dose history.
5. Reminder UI and in-app/browser notifications while supported; server-side missed marking only after explicit, documented rules.
6. Baseline offline emergency library built from reviewed content packages.
7. Client-side Tesseract OCR that requires user review and confirmation.
8. Experimental camera motion-difference prototype, isolated from dose status.
9. Client-side PDF report generated from authorized API data.

## Later / out of MVP
- Caregiver invitations and scoped grants; web push and bounded escalation.
- Versioned IndexedDB package updates.
- Document upload management.
- Facility-provider integration/manual location fallback.
- Knowledge-base audit pipeline and reviewer publication console.
- Server OCR, semantic search, smart pillbox, computer-vision classifier.

## Existing UI reconciliation
- “AI Health Advisor” becomes Library Q&A over approved content only; no generative clinical answers in MVP.
- Labs/Visits remain clearly labeled demo content until persistence and provenance exist.
- Inventory/refill lives on medication and `refill_events`.
- Reports use jsPDF in the client, populated from API responses.
- Hospital directory stays clearly marked “Sample” until a real provider is integrated.

## Metrics
Reported adherence for a period is:
`reported_taken / (reported_taken + skipped + missed)` across resolved dose events in that period. Label it **self-reported**. Exclude `scheduled`, `acknowledged`, and `cancelled` from the denominator.

## Terminology
A **dose event** is one scheduled occurrence at a local date/time, stored with a timezone-aware timestamp. A **content package** is a versioned JSON bundle of approved item versions for a jurisdiction and locale, identified by a cryptographic hash. A hash alone does not establish reviewer identity or authorization.
