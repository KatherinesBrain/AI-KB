# LivingKB – Evidence & Citation Rules (v0.1)

> Goal: Avoid hallucinated sources, keep claims proportionate to evidence, and make it obvious when something is solid vs contested.  
> These rules apply to all TopicCards and EvidenceItems.

---

## 0. First-Principles Constraints

1. **Traceability** – Every KB statement must point back to a concrete document (`EvidenceItem` or `meta.sources`). No trace, no claim.
2. **Mechanical verifiability** – Encode rules in schemas or lint scripts wherever possible so violations fail fast without waiting for human reviewers.
3. **Predictable degradation** – When evidence is mixed or missing, soften language and keep `meta.review_status != "verified"` rather than silently implying certainty.
4. **Role clarity** – Each agent (Content, Evidence, Answer, etc.) must understand exactly what the rules permit or forbid; policy drift should be impossible.

Everything below operationalizes these invariants.

---

## 1. Core Principles

1. **No fabricated sources**
   - Never invent authors, journals, DOIs, years, or URLs. If you don’t have a real reference, you cannot make one up.

2. **Evidence-proportional claims**
   - Claim strength in summaries/slots must match the quality, quantity, and agreement level of the evidence.

3. **Honest uncertainty**
   - Use language like “proposed contributors”, “associated with”, “observational data suggests” when evidence is weak or contested.

4. **Description vs marketing**
   - Describe what was published/claimed, but clearly label marketing narratives rather than treating them as consensus.

---

## 2. Evidence Tiers

Every EvidenceItem implicitly falls into one of these tiers:

- **Tier 0 – Canonical records** (laws, census data, physical constants)
- **Tier 1 – Peer-reviewed syntheses** (meta-analyses, consensus guidelines)
- **Tier 2 – Peer-reviewed primary research** (RCTs, observational studies)
- **Tier 3 – Authoritative references** (textbooks, encyclopedias, WHO/NIH documents)
- **Tier 4 – Serious journalism/commentary** (NatGeo, major newspapers, Science-Based Medicine)
- **Tier 5 – Opinion / blogs / lived experience** (personal blogs, forums, anecdotal reports)

You don’t have to store the tier number yet, but know it mentally (or in `notes`) so consensus judgments stay grounded.

---

## 3. Minimum Evidence Requirements by Consensus Level

### `consensus_high`
- Use only when core claims rely on Tier 0–3 sources and multiple independent sources agree.
- Requires at least one Tier 1–3 source in `evidence` or `meta.sources`.

### `consensus_mixed`
- Use when evidence exists but methods/results are debated or critics raise non-trivial issues.
- Requires at least one “pro” source **and** one critique/skeptical source (Tier 2–5) if critiques are mentioned.
- Summaries must avoid causal certainty.

### `frontier_early`
- Use when evidence is new, small-N, or unreplicated.
- Requires at least one real study/report (Tier 2–4) plus explicit mention in `expert_notes`/`limitations` that findings are early-stage.

### `controversial`
- Use when disagreement is primarily ideological or value-driven.
- Separate empirical statements (“this happened”) from value judgments (“this is good/bad”).

---

## 4. Citation Rules

1. **No realistic-looking fakes** – Never create faux scholarly entries with plausible-sounding metadata. If you lack a source, describe the uncertainty instead.
2. **Allowed placeholders (explicit)** – When you intend to add a real source later, mark it with `type: "placeholder"` and empty metadata:
   ```json
   {
     "source_id": "blue_zones_critique_tbd",
     "type": "placeholder",
     "title": "TBD critique of Blue Zones data quality",
     "authors": [],
     "year": null,
     "venue": null,
     "url": null,
     "supports_claims": ["/slots/limitations"],
     "notes": "Placeholder: a real critique source must be added before this card can be marked verified."
   }
   ```
   Any card containing a placeholder must keep `meta.review_status != "verified"`.
3. **Schema-backed enforcement** – `schemas/evidence_item.schema.json` codifies the placeholder behavior (e.g., `url` forced to `null`, `notes` required). Keep policy + schema aligned so CI can reject violations automatically.
4. **URL rules** – If a source has a URL, it must be real and reachable before verification. Books/offline sources omit `url` but must include `venue`/`notes`.
5. **Support mapping** – Every `supports_claims` entry must reference a real JSON path (`/slots/...`, `/summary_variants/...`). Invalid paths are review failures.

---

## 5. Summary Variant Rules (Especially Kid-Friendly)

1. Kid-friendly text must never make stronger causal claims than `standard`.
2. Safe simplification is welcome (fewer clauses, friendlier words) but cannot flip correlation into causation.
3. For sensitive or mixed-consensus topics, automatically compare `kid_friendly` vs `standard` after edits; if the kid version uses stronger verbs (“causes”, “proves”), soften it.

---

## 6. Card Status & Production Rules

A TopicCard can power “trusted KB answers” only if:

- `meta.review_status == "verified"`.
- It contains **no** `type: "placeholder"` EvidenceItems.
- Consensus tag matches the cited tier mix.
- Kid-friendly summary is consistent with `standard`.
- `supports_claims` paths are valid.

If any check fails, the card stays `draft` or `flagged`, and AnswerAgent must either avoid using it or mark the answer as unverified/frontier.

---

## 7. Talking About Critiques Without Overfitting

- Summarize critiques in `limitations` / `expert_notes` (“Some demographers argue age records may contain clerical errors…”).
- Cite one or more real critique sources as examples, but don’t imply unanimity (“some researchers” vs “all researchers”).
- Avoid hinging the entire argument on a single paper when the critique is a broader pattern.

---

## 8. Example: Applying the Rules to `blue_zones`

- Consensus: `consensus_mixed` (✅) because tiers span NatGeo (Tier 4), Pes & Poulain (Tier 2/3), and critiques (Tier 4/5).
- Evidence: include at least one promoter, one demographic overview, one critique.
- Kid-friendly: “People there often eat mostly plants…” is fine; “live long *because* of plants” is not.
- Status: keep `review_status: "flagged"` while any placeholder/unknown critique citation remains.

---

## 9. Review Workflow Hook

1. Validate that sources obey the schema (no fake URLs, placeholders flagged).
2. Set `meta.review_status = "verified"` only if citations are real, consensus tag matches evidence, and summaries respect the rules above.
3. Record `meta.review_notes` describing which tiers were reviewed, why the consensus tag fits, and open questions.

---

## 10. Automation Hooks & Agent Responsibilities

- **Schema validation:** Run JSON Schema (TopicCard, EvidenceItem, QuestionTemplate, AnswerEnvelope) in CI; reject commits that violate placeholder or `supports_claims` rules.
- **Consensus-vs-tier lint:** Add a script that inspects cited tiers and ensures `consensus_level` is compatible (no `consensus_high` with only Tier 4/5 sources).
- **Kid-friendly regression test:** Automatically diff `kid_friendly` vs `standard` for mixed/frontier topics and fail when the kid text makes stronger claims.
- **Agent checklists:**
  - **ContentAgent** never commits a TopicCard without `supports_claims` on each EvidenceItem and at least implicit tier info in `notes`.
  - **EvidenceAgent** records tier reasoning in `meta.review_notes` and enforces status changes (verified vs flagged).
  - **AnswerAgent** only uses fully verified cards (no placeholders) in “This is what is known…” mode; otherwise it labels the output as unverified/frontier.

These automation hooks keep the evidence rules grounded in executable checks rather than hopeful prose.
