# LivingKB – Project Outline (v0.1, first-principles draft)

> Working title: **LivingKB**  
> Goal: A structured, living knowledge base that LLMs can reliably query and copy from, instead of hallucinating facts.  
> Method: Derive every component from three first-principles constraints—(1) language models hallucinate when forced to invent facts, (2) structured data is easier to audit than prose, (3) provenance and consensus labels must travel with every answer.

---

## 0. First-Principles Decomposition

1. **Problem definition** – LLMs intermingle retrieval, reasoning, and style, which makes it impossible to guarantee factual correctness.
2. **Invariant requirements** – Facts must be separable from reasoning, transparent in provenance, and typed so that downstream systems can reuse them.
3. **Minimal sufficient system** – Therefore we need:
   - Typed TopicCards (storage)
   - Deterministic QuestionTemplates (intent → field mapping)
   - An AnswerAgent protocol (rendering)
4. **Measurable success** – If an answer can be recomputed offline from checked-in data, we achieved KB-first behavior.

Everything else (agents, ingestion helpers, ranking) is an optimization layered on top of this minimal core.

---

## 1. Vision

Build a **typed, evidence-aware knowledge base** that:

- Stores facts as **structured entities** (Person, Place, Concept, Event, etc.).
- Associates each entity with a **canonical set of questions** appropriate for its type.
- Provides **ready-to-use answer fragments** (short, kid-friendly, standard, expert) that an LLM can paste or lightly rephrase.
- Clearly separates:
  - **Consensus knowledge** (evidence-backed)
  - **Frontier / speculative knowledge**
  - **Opinion / value-judgment content**

The long-term goal is to make a mode where an AI can say:

> “**This is what is known from the KB:** …  
> If you want more reasoning, I can extrapolate beyond this.”

---

## 2. Scope & Non-goals

### 2.1 Initial Scope (v0.1)

- Design the **ontology** (entity types + required fields).
- Define the **TopicCard** format and **QuestionTemplate** format.
- Manually create a small, high-quality KB slice:
  - ~20–50 entities across:
    - People
    - Places
    - Concepts/Things
    - Events (optional for v0.1)
- Create a simple answering spec:
  - Given: entity + natural language question
  - Return: “This is what is known…” answer built from TopicCard fields.

### 2.2 Non-goals (for v0.1)

- No full internet-scale ingestion.
- No automatic real-time crawling.
- No complex UI; focus on **data structures** and **answer patterns**.
- No attempt to encode *all* controversial/nuanced domains. Start with “clean” topics.

---

## 3. Core Concepts

### 3.1 Entity Types

At minimum:

- **Person**
- **Place**
- **Concept/Thing** (tech, ideas, objects)
- **Event**
- **Organization** (optional in v0.1)

Each type has:

- A set of **required fields** (slots).
- A set of **default questions** (Who/What/When/Where/Why/How variants).

### 3.2 TopicCard

A **TopicCard** is the canonical record for one topic (one entity or concept).

High-level fields:

- `id` (stable identifier)
- `entity_type` (Person, Place, Concept, Event, Organization)
- `labels` (names, aliases, keywords)
- `summary_variants`:
  - `short_answer`
  - `kid_friendly`
  - `standard`
  - `expert_notes`
- `slots` (type-specific structured fields, e.g. `date_of_birth`, `location`, etc.)
- `evidence` (list of EvidenceItems)
- `consensus_level` (e.g. `high`, `medium`, `low`, `frontier`, `controversial`)
- `relations` (links to other TopicCards)
- `meta` (last_reviewed, sources, editors)

### 3.3 QuestionTemplate

A **QuestionTemplate** describes how to answer a class of questions for a given entity type.

Example:

- For `Person`:
  - “Who is {name}?” → use `summary_variants.short_answer`
  - “Where was {name} born?” → use `slots.place_of_birth`
  - “When was {name} born?” → use `slots.date_of_birth`

Templates include:

- `entity_type`
- `pattern` (NL pattern or intent)
- `field_paths` to use in the answer
- `style` (short / kid_friendly / standard / expert)
- Optional: `explanation_style` (plain, list, timeline)

### 3.4 Evidence & Consensus Labels

Each TopicCard and, ideally, each key claim stores:

- Evidence items (citations, study metadata, guidelines).
- A `consensus_level`, e.g.:
  - `consensus_high`
  - `consensus_mixed`
  - `frontier_early`
  - `controversial`
- Optionally an `epistemic_type`:
  - `empirical_science`, `expert_opinion`, `lived_experience`, `value_judgment`

LLMs can use this metadata to:

- Prefer consensus-backed facts for “what is known.”
- Clearly mark frontier or speculative content.

---

## 4. System Components

### 4.1 Ontology & Schema

- Define entity types and their **required fields**.
- Define **TopicCard** and **QuestionTemplate** schemas.
- Document consensus/evidence labels.

### 4.2 Content Ingestion (v0.x-level)

- v0.1: manual entry using the schema.
- v0.2+: semi-automatic ingestion from sources like:
  - Wikipedia
  - Reference summaries
  - Curated docs

### 4.3 Evidence Analysis

- v0.1: human judgment to assign `consensus_level`.
- v0.2+: automated helpers:
  - Parse citations
  - Classify study types
  - Suggest consensus labels for human review.

### 4.4 Question/Answer Engine

- Map user questions to:
  - An entity (TopicCard)
  - A QuestionTemplate (by entity_type + intent)
- Build answers primarily by:
  - Plugging TopicCard fields into templates.
- Enforce a “**KB-first, hallucination-minimized**” mode:
  - Only reason beyond the KB when explicitly allowed.

### 4.5 LLM Integration Modes

Planned modes:

1. **KB-only mode**  
   - Answers must come from TopicCards.

2. **KB+Reasoning mode**  
   - “This is what is known from the KB…”  
   - + optional extra reasoning, clearly marked as extrapolation.

3. **Free-form mode**  
   - Normal LLM behavior (not a v0.1 priority, but a reference point).

---

## 5. MVP (v0.1) Definition

For v0.1, “done” means:

- Ontology defined for:
  - Person, Place, Concept/Thing (Event optional).
- TopicCard schema stable and documented.
- QuestionTemplate schema stable and documented.
- At least **20 TopicCards** fully filled in and reviewed.
- A written answering spec defining:
  - “This is what is known” answer pattern.
  - How to select summary variants and fields.
- A small test set of questions and expected answers.

---

## 6. v0.2+ Roadmap (High-level)

- Expand entity coverage and number of TopicCards.
- Add `Event` and `Organization` as fully supported types.
- Add basic **graph relationships** (who did what, where, when).
- Design and implement a basic **ranking strategy** when multiple candidates/answers exist.
- Explore **semi-automatic ingestion** from Wikipedia and other sources.
- Attach **evidence strength scoring** for scientific/medical topics.
- Add user/person-specific TopicCards with privacy controls (long-term idea).

---

## 7. Risks / Open Questions

- How opinionated should consensus labels be on controversial topics?
- How to handle entities that evolve quickly (e.g. AI models, political roles)?
- How much manual curation is sustainable before automation is required?
- How will KB updates propagate to downstream LLM behavior (cache invalidation, versioning)?
- How to represent **lived experience** and **value judgments** without presenting them as empirical “facts”?
