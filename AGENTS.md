# LivingKB – AGENTS

> Conceptual agents/roles that could be automated (LLMs, scripts) or done by humans.  
> Goal: keep responsibilities clear and composable.

---

## 1. CoordinatorAgent

**Goal:** Orchestrate work between the other agents and keep the project aligned with the design docs.

**Responsibilities:**

- Track high-level tasks from `todo.md`.
- Decide which agent should handle each subtask.
- Maintain overall consistency across schemas, cards, and templates.

**Inputs:**

- `outline.md`
- `todo.md`
- Project configuration (paths, schemas).

**Outputs:**

- Updated TODO status.
- Requests/instructions for other agents.
- Periodic status summaries.

---

## 2. OntologyAgent

**Goal:** Design and maintain the **entity types**, their fields, and the overall ontology.

**Responsibilities:**

- Manage the list of entity types (Person, Place, Concept, Event, Organization).
- Define and update required/optional fields (`slots`) for each type.
- Ensure TopicCard and QuestionTemplate schemas remain consistent with the ontology.
- Propose changes when gaps appear (e.g., new fields needed).

**Inputs:**

- Existing schemas (TopicCard, QuestionTemplate).
- Feedback from AnswerAgent and ContentAgent about missing fields.

**Outputs:**

- Updated ontology definitions.
- Updated schema snippets for `data_formats.md`.

---

## 3. ContentAgent (TopicCard Builder)

**Goal:** Create and maintain **TopicCards** for specific entities.

**Responsibilities:**

- Given a target entity (e.g., “Barack Obama”), fill out a TopicCard:
  - Entity type and basic slots.
  - `summary_variants` (short, kid_friendly, standard, expert_notes).
  - Initial `consensus_level` and `epistemic_type`.
- Attach sources and citations at least at a basic level.
- Flag anything uncertain for EvidenceAgent.

**Inputs:**

- Ontology definitions.
- Source materials (Wikipedia, reference docs, curated links).

**Outputs:**

- TopicCard documents (e.g., `cards/{id}.json` or equivalent).
- Notes for EvidenceAgent when evidence is unclear or mixed.

---

## 4. EvidenceAgent

**Goal:** Evaluate and tag the **evidence strength** and **consensus level** for TopicCards and key claims.

**Responsibilities:**

- Review TopicCards and their cited sources.
- Assign or refine:
  - `consensus_level` (e.g. `consensus_high`, `frontier_early`, etc.).
  - `epistemic_type` (empirical_science, expert_opinion, lived_experience, value_judgment).
- Optionally structure **EvidenceItems**:
  - Study type (meta-analysis, RCT, observational, etc.).
  - Publication year and venue.
- Flag controversial or sensitive topics for extra review.

**Inputs:**

- TopicCards with preliminary evidence metadata.
- External references (papers, guidelines, etc.).

**Outputs:**

- Updated evidence metadata in TopicCards.
- Guidance for AnswerAgent on how strongly to present claims.

---

## 5. TemplateAgent (QuestionTemplate Designer)

**Goal:** Define and maintain **QuestionTemplates** for each entity type.

**Responsibilities:**

- For each entity type, define:
  - Canonical questions (who/what/when/where/why/how).
  - The mapping from each question to TopicCard fields.
- Add alternative phrasings and intents for natural language mapping.
- Refine templates when AnswerAgent encounters missing or awkward answers.

**Inputs:**

- Ontology definitions (fields per entity type).
- Example user questions.
- Feedback from AnswerAgent about coverage gaps.

**Outputs:**

- QuestionTemplate sets (e.g., `questions/Person.json`).
- Documentation for how to map intents → templates.

---

## 6. AnswerAgent (KB Answerer)

**Goal:** Turn **user questions** into **KB-backed answers** using TopicCards and QuestionTemplates.

**Responsibilities:**

- Detect:
  - Target entity (or set of candidate entities).
  - Question intent (e.g. “when born”, “where located”, “what is it”).
- Select the appropriate QuestionTemplate for the entity type.
- Build an answer using TopicCard fields and summary variants.
- Enforce the “**This is what is known from the KB**” pattern in KB-only mode.
- Optionally:
  - Add frontier/uncertainty sections (using EvidenceAgent metadata).
  - Perform extra reasoning if allowed by mode.

**Inputs:**

- User question.
- TopicCards.
- QuestionTemplates.
- Evidence metadata (consensus level, etc.).

**Outputs:**

- Final answer text.
- Internal logs: which entity, template, and fields were used.

---

## 7. FrontierScoutAgent (Optional / Future)

**Goal:** Discover **new or changing knowledge** that might update TopicCards or create new ones.

**Responsibilities:**

- Scan sources (e.g., news, new papers, updated Wikipedia entries).
- Detect:
  - New entities that meet inclusion criteria.
  - New evidence that changes consensus level.
- Suggest updates for ContentAgent and EvidenceAgent.

**Inputs:**

- External feeds (web, APIs, curated lists).
- Existing TopicCards and evidence metadata.

**Outputs:**

- Proposals for new TopicCards.
- Proposed updates to existing TopicCards.
- Flags where consensus may be shifting.

---

## 8. SafetyAgent (Alignment / Guardrails)

**Goal:** Ensure the KB and answers are aligned with safety, ethics, and user well-being goals.

**Responsibilities:**

- Review:
  - Sensitive topics (health, identity, politics, etc.).
  - How consensus vs frontier information is presented.
- Enforce:
  - Clear labeling of uncertainty and frontier knowledge.
  - Guardrails on how advice is phrased (especially health/legal).
- Coordinate with EvidenceAgent when “peer-reviewed” sources are historically biased or harmful.

**Inputs:**

- TopicCards and QuestionTemplates for sensitive domains.
- EvidenceAgent assessments.
- Safety policy guidelines.

**Outputs:**

- Recommendations for phrasing and disclaimers.
- Block/soften certain answer patterns if needed.
- Notes for future policy updates.

