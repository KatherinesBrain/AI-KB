# LivingKB – Answering Spec (KB-first mode)

> First principles reminder: the AnswerAgent is not allowed to invent content. Every token must trace back to a TopicCard field plus template metadata.

---

## 1. Inputs

1. **User question** – Free-form natural language.
2. **TopicCard** – Identified via entity resolution (exact label, alias, or search).
3. **QuestionTemplate** – Selected by matching question intent and entity type.

---

## 2. Selection Pipeline

1. **Entity resolution**
   - Normalize the question (lowercase, remove punctuation).
   - Search labels/aliases for matches.
   - If multiple candidates, prefer higher consensus_level or ask clarifying question.
2. **Intent detection**
   - Compare question tokens against `patterns` for the entity type (see `questions/*.json`).
   - Use deterministic mapping before any LLM invocation.
3. **Template binding**
   - Load the template’s `field_paths`.
   - Check if the TopicCard contains the required fields.
   - If missing, use `fallback_style` or return a “field unavailable” notice.

---

## 3. Rendering Rules

1. Begin every answer with:  
   `This is what is known from the KB:`
2. Insert the `raw_answer` assembled from the template. For example:
   - `summary_variants.short_answer` for identity questions.
   - `slots.place_of_birth` for birthplaces.
3. Append epistemic notes when useful:
   - `Consensus: consensus_high (empirical_historical).`
   - `Frontier notes: ...` (only when `consensus_level` is not high).
4. Never mix KB facts with extrapolated reasoning unless the mode explicitly allows it (v0.1 focuses on KB-only).

---

## 4. Missing Data Patterns

- **Field missing:**  
  `This is what is known from the KB: We do not yet store {intent} for {entity}.`
- **Card missing:**  
  `This is what is known from the KB: We do not have a TopicCard for that entity yet.`
- **Multiple matches:**  
  `The KB has multiple entities with that name. Please specify {list of candidates}.`

---

## 5. Example Envelope → Answer

Envelope:

```json
{
  "entity_id": "photosynthesis",
  "question_intent": "definition",
  "consensus_level": "consensus_high",
  "answer_fields_used": ["slots.definition"],
  "raw_answer": "Photosynthesis is the process by which plants, algae, and some bacteria convert light energy into chemical energy stored in sugars.",
  "rendered_answer": "Photosynthesis is the process by which plants, algae, and some bacteria convert light energy into chemical energy stored in sugars.",
  "notes_for_user": "High consensus biology process.",
  "frontier_notes": null
}
```

Rendered response:

```
This is what is known from the KB:
Photosynthesis is the process by which plants, algae, and some bacteria convert light energy into chemical energy stored in sugars.
Consensus: consensus_high (empirical_science).
```

---

## 6. Future Modes (Preview)

- **KB + Reasoning:** After the KB answer, add `If you want extrapolation…` and clearly mark the speculative part.
- **Frontier highlight:** If `consensus_level` is `frontier_early`, automatically include `Frontier notes`.
- **List answers:** For templates with `explanation_style = list`, bulletize each fact while still referencing TopicCard fields.

For v0.1 we stay in KB-only mode; this spec is sufficient to generate audit-friendly answers.

