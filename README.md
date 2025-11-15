# LivingKB

LivingKB is a first-principles attempt to build a small, typed, evidence-aware knowledge base that large language models can trust. Instead of relying on emergent reasoning, we decompose the problem into concrete building blocks:

1. **Stable ground truth objects** – TopicCards capture entities as structured data with provenance.
2. **Deterministic question mappers** – QuestionTemplates map natural-language intents to fields.
3. **Answer envelopes** – The AnswerAgent assembles “This is what is known…” responses directly from stored data.

By starting from these fundamentals (LLMs hallucinate → we must separate storage, interpretation, and narration), we keep the surface area small and the results auditable.

## Repository Layout

- `outline.md` – Project vision, scope, and roadmap.
- `todo.md` – Execution tracker aligned to the outline.
- `AGENTS.md` – Role definitions for human/AI contributors.
- `data_formats.md` – TopicCard, EvidenceItem, and QuestionTemplate schemas plus ontology notes.
- `answering_spec.md` – How to go from question → template → rendered answer.
- `schemas/` – JSON Schemas for validation.
- `cards/` – Hand-curated TopicCards (v0.1 slice).
- `questions/` – QuestionTemplate sets per entity type.
- `tests/` – Manual question sets for validation passes.

## First-Principles Design Highlights

- **Separation of concerns** – Storage (TopicCards) and narration (AnswerAgent) stay independent so that updating facts never requires prompt-tuning.
- **Explicit epistemics** – Every card is tagged with consensus and evidence, forcing clarity about what is known vs. frontier.
- **Deterministic matching** – Intents are mapped to fields using simple pattern sets before any generative step, constraining the LLM to copy data it can cite.
- **Composable agents** – Each responsibility (ontology, content, evidence, templates, answers) can be handled by a human or model without tight coupling.

## Getting Started

1. Inspect `schemas/topic_card.schema.json` to understand required fields.
2. Review `cards/*.json` for concrete examples across Person, Place, Concept, and Event types.
3. Use `questions/*.json` with the answering spec to simulate KB-only responses.
4. Add or edit cards/templates, then re-run validation (future automation TODO).

## Next Steps

- Finish the open review tasks in `todo.md`.
- Automate schema validation for cards and templates.
- Explore lightweight ingestion helpers (semi-automatic infobox parsing).

The entire repo is optimized for transparent auditing: everything needed to reconstruct an answer is a checked-in file.

