# LivingKB – TODO (v0.1)

> Focus: Build a first-principles KB slice with auditable schemas, cards, and answering rules.

---

## 0. Meta / Project Setup

- [x] Confirm project name (LivingKB) and first-principles charter.
- [x] Decide pilot domain (clean, general-knowledge mix across people, places, concepts, one event).
- [x] Create root-level repo structure:
  - README, outline, todo, AGENTS, data_formats, answering_spec
  - `schemas/`, `cards/`, `questions/`, `tests/`
- [x] Choose canonical storage format (JSON + Markdown docs).

---

## 1. Ontology & Schema Design

- [x] Finalize entity types for v0.1:
  - [x] Person
  - [x] Place
  - [x] Concept/Thing
  - [x] Event
  - [ ] Organization (optional, planned for v0.2)
- [x] Define required + optional slots per type (`data_formats.md`).
- [x] Publish schemas:
  - [x] `schemas/topic_card.schema.json`
  - [x] `schemas/question_template.schema.json`
  - [x] `schemas/evidence_item.schema.json`
  - [x] `schemas/answer_envelope.schema.json`
- [x] Document consensus/epistemic enums and relation types.

---

## 2. Data Formats & Storage

- [x] Confirm file naming (`cards/{id}.json`, `questions/{entity_type}.json`).
- [x] Include versioning metadata (`meta.schema_version`, timestamps).
- [x] Provide worked examples for each entity type (see `cards/`).
- [x] Provide QuestionTemplate sets per type (see `questions/`).
- [x] Describe serialization + validation path in `data_formats.md`.

---

## 3. Prototype KB Content (Hand-Curated)

- [x] Select 20 v0.1 cards:
  - 7 People, 6 Places, 6 Concepts, 1 Event.
- [x] Fill summary variants + slots for every card.
- [x] Attach evidence metadata + consensus labels.
- [ ] Perform manual sanity review / fact-check pass (needs second editor).

---

## 4. Question / Answer Layer

- [x] Design QuestionTemplate coverage per entity type.
- [x] Capture intent → field mapping rules.
- [x] Write answering spec with “This is what is known…” prefix.
- [x] Define fallback patterns for missing fields/cards.

---

## 5. Evaluation & Testing

- [x] Draft manual question set (`tests/question_set.json`).
- [ ] Run KB-vs-raw LLM comparison notes.
- [ ] Record identified schema/template/summary gaps after review.

---

## 6. Stretch / Future Tasks (v0.2+)

- [ ] Add Organization entity type + templates.
- [ ] Automate schema validation pipeline.
- [ ] Prototype semi-automatic ingestion (infobox → TopicCard).
- [ ] Explore answer ranking when multiple entities match.
- [ ] Model graph relations more richly (who/what/where/when edges).
- [ ] Design lightweight UI for browsing cards and templates.
- [ ] Document alignment considerations for sensitive topics.

