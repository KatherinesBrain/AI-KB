# LivingKB – Data Formats (v0.1)

> First principles: if an answer cannot be regenerated from structured data with provenance, it is out of scope for LivingKB. Everything described below is designed so that a deterministic script (or careful human) can rebuild the answer word-for-word.

---

## 1. Primitive Objects

1. **TopicCard** – Canonical record for a single entity (Person, Place, Concept/Thing, Event, optional Organization).
2. **EvidenceItem** – Traceable citation describing where a fact came from and how strong it is.
3. **QuestionTemplate** – Mapping of user intent patterns to TopicCard field paths and rendering styles.
4. **AnswerEnvelope** – Audit log of which fields were used, consensus level, and rendered output text.

These primitives are sufficient to reconstruct any KB-only answer.

---

## 2. Ontology Overview

| Entity Type | Purpose | Required Slots | Optional Slots |
|-------------|---------|----------------|----------------|
| `Person` | People in history, science, arts, etc. | `full_name`, `date_of_birth`, `place_of_birth`, `nationalities`, `occupations`, `summary_variants.*` | `date_of_death`, `roles_offices`, `notable_actions`, `awards`, `education`, `current_status`, `relations.*` |
| `Place` | Cities, regions, natural formations | `place_type`, `country`, `summary_variants.*`, `coordinates`, `notable_features` | `region`, `population_estimate`, `founded_or_incorporated`, `economic_roles`, `governance` |
| `Concept` | Ideas, technologies, theories | `definition`, `key_characteristics`, `summary_variants.*` | `applications`, `limitations`, `history`, `related_concepts` |
| `Event` | Historical or scientific events | `date`, `location`, `event_type`, `summary_variants.*` | `key_participants`, `outcomes`, `significance` |
| `Organization` (future) | Institutions, companies | `legal_name`, `founded`, `summary_variants.*` | `headquarters`, `key_people`, `mission`, `products` |

All TopicCards inherit the base schema and extend the `slots` object with type-specific keys.

---

## 3. TopicCard Schema (JSON)

See `schemas/topic_card.schema.json` for the formal definition. Highlights:

- `id`: lowercase snake_case unique identifier.
- `entity_type`: enum (`Person`, `Place`, `Concept`, `Event`, `Organization`).
- `labels`: array of surface forms; first label is canonical display.
- `summary_variants`: `short_answer`, `kid_friendly`, `standard`, `expert_notes`.
- `slots`: object; keys vary by entity type.
- `evidence`: array of `EvidenceItem` references.
- `consensus_level`: enum describing confidence.
- `epistemic_type`: how the knowledge was established.
- `relations`: list of typed edges to other TopicCards (`member_of`, `located_in`, etc.).
- `meta`: schema version, timestamps, reviewer IDs, and `sources` array.

Example card: `cards/barack_obama.json`.

---

## 4. EvidenceItem Schema

Defined in `schemas/evidence_item.schema.json`.

Fields:

- `source_id` – Stable key (e.g., `wp_barack_obama`, `doi_10_1038_xyz`).
- `type` – Enum (`reference_article`, `journal_article`, `book`, `primary_source`, `interview`).
- `title`, `authors`, `year`, `venue`.
- `url` – HTTP(S) reference or DOI link.
- `study_type` – Optional (RCT, observational, analysis, etc.).
- `supports_claims` – Array of JSON Pointers referencing the card fields.
- `notes` – Summary + caveats.

Evidence items can appear at the TopicCard level or inline per slot (future extension).

---

## 5. QuestionTemplate Schema

Formal version in `schemas/question_template.schema.json`.

Key fields:

- `id`
- `entity_type`
- `intent` (normalized key, e.g., `identity_overview`, `place_of_birth`, `definition`)
- `patterns` (literal phrases, regex-like fragments, or intent tags)
- `field_paths` (JSON Pointer strings or dot paths such as `slots.place_of_birth`)
- `style` (`short`, `kid_friendly`, `standard`, `expert`)
- `fallback_style` (if primary field missing)
- `explanation_style` (`plain`, `list`, `timeline`)
- `notes`

Templates are stored in `questions/{entity_type}.json`.

---

## 6. Answer Envelope

`schemas/answer_envelope.schema.json` documents the structure:

```json
{
  "entity_id": "barack_obama",
  "question_intent": "place_of_birth",
  "consensus_level": "consensus_high",
  "epistemic_type": "empirical_historical",
  "answer_fields_used": ["slots.place_of_birth"],
  "raw_answer": "Honolulu, Hawaii, United States",
  "rendered_answer": "Barack Obama was born in Honolulu, Hawaii, United States.",
  "notes_for_user": "High-consensus biographical data.",
  "frontier_notes": null
}
```

The AnswerAgent converts the envelope into the user-facing sentence prefixed with “This is what is known from the KB:”.

---

## 7. Consensus & Epistemic Enums

- `consensus_high` – widely agreed, low dispute.
- `consensus_mixed` – mostly agreed but with notable dissent.
- `frontier_early` – emerging research, low replication.
- `controversial` – significant disagreement or sociopolitical sensitivity.
- `historical_record` – archival fact with little room for dispute.

Epistemic types:

- `empirical_science`
- `empirical_historical`
- `expert_opinion`
- `lived_experience`
- `value_judgment`
- `mixed_empirical_hypothesis`
- `emerging_research`

---

## 8. Relations

Each relation object includes:

```json
{
  "type": "member_of",
  "target_id": "democratic_party",
  "description": "Obama served under the Democratic Party."
}
```

Types include `member_of`, `located_in`, `influenced_by`, `involves_person`, `related_concept`, `successor_of`, etc. Additional types can be added as ontology evolves.

---

## 9. File Naming & Versioning

- TopicCards: `cards/{id}.json`.
- QuestionTemplates: `questions/{entity_type}.json`.
- Schemas: `schemas/*.schema.json`.
- Tests: `tests/*.json`.
- Each JSON file includes `meta.schema_version` (currently `0.1`) plus timestamps in ISO 8601.

This structure keeps the repository grab-and-go: cloning it provides every fact, schema, and template required to rebuild the KB.

