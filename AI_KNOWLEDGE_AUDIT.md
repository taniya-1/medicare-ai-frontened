# MediGuard — AI and Knowledge Audit Governance

## MVP boundary
No LLM-generated medical guidance is shown to users. The Library Q&A may retrieve or display only approved, versioned content items. If a question cannot be answered from approved items, say the library does not contain a reviewed answer and direct the user to a qualified professional or local emergency service as appropriate—without inventing instructions.

## Future audit pipeline
1. **Ingest metadata:** source organization, title, URL/reference, jurisdiction, publication/revision date, retrieval date, content type, and permitted-use notes.
2. **Validate:** schema, URL availability where feasible, required provenance, jurisdiction, dates, version, and source allow-list.
3. **Compare:** detect possible conflicts, stale versions, missing citations, unsupported claims, and duplicate/near-duplicate items.
4. **Flag, never decide:** AI or automated checks may produce internal flags with evidence and confidence; they must not select between conflicting clinical recommendations or publish content.
5. **Human review:** authorized reviewer checks source and content, records identity, decision, rationale, and review timestamp.
6. **Publish:** controlled workflow creates an immutable version and package manifest; only approved versions enter user-facing packages.
7. **Distribute:** client validates schema and trusted hash/signature and keeps the previous package if the update fails.
8. **Audit:** preserve version history and who/what approved each publication.

## Required item metadata
`source_organization`, `source_title`, `source_reference`, `publication_or_revision_date`, `retrieved_at`, `jurisdiction`, `locale`, `version`, `review_status`, `reviewer_id`, `reviewed_at`, `content_hash`.

## Review statuses
Suggested values: `draft`, `needs_review`, `approved`, `rejected`, `superseded`, `withdrawn`. Only `approved` and currently published versions are user-visible. Withdrawal/supersession must propagate in the next package update and be clearly handled offline.

## Prompt-injection and source safety
Treat source pages, uploaded documents, OCR text, and retrieved content as untrusted data—not instructions to the AI pipeline. Never allow source text to change system policy, invoke tools, expose secrets, or publish itself. Separate content from metadata and sanitize rendered HTML.

## Clinical-content policy
Use authoritative, jurisdiction-appropriate sources selected by a qualified reviewer. Do not fabricate sources or dates. Do not publish first-aid/CPR/medication-specific material until a human reviewer confirms the exact content, jurisdiction, and currency. Conflicts and uncertain evidence remain blocked for review.
