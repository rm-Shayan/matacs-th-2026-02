# AI Proposal Triage Design

**Assessment Reference:** `MATACS-TH-2026-02`  
**Role:** Associate Software Developer  
**Candidate Name:** Rao Muhammad Shayan  
**Time Spent:** ~55 minutes  

---

## AI Usage Disclosure
AI tools (Gemini / Claude) were utilized to help structure the JSON extraction schemas, brainstorm architectural trade-offs, and outline security boundary defenses. During initial evaluation, the AI assistant attempted to score Proposal B at Rank 1 due to its lowest price ($189,500 CAD) and fast timeline; I intervened and corrected the workflow because Proposal B failed the mandatory gatekeeper requiring the bidder itself to hold and state an Ontario electrical contractor licence (deferring electrical work to an unconfirmed subcontractor). Furthermore, when evaluating Proposal C, the model initially suggested honoring the procurement pre-approval note; I corrected this by engineering an untrusted data boundary and explicit prompt-injection isolation layer that routes such attempts to a security-flagged queue.

---

## Repository Index & Deliverables

| Deliverable | Path | Description |
| :--- | :--- | :--- |
| **System Write-Up** | [`WRITEUP.md`](./WRITEUP.md) | Answers to the 5 core design questions (AI vs. Rules, Consistency, Human-in-the-loop, Criteria Changes, and Risks). |
| **AI Prompts & Schemas** | [`PROMPTS.md`](./PROMPTS.md) | Exact system prompt, dynamic user prompt template, structured JSON output schema, and qualitative rubric. |
| **Evaluation Ranking** | [`ranking.json`](./ranking.json) | Expected ranking and status of Proposals A–E conforming strictly to the requested schema. |
| **Process Flow Diagram** | [`diagram/flow_diagram.png`](./diagram/flow_diagram.png) | High-resolution end-to-end architecture diagram showing AI steps, rule-based gates, and human review checkpoints. |
| **Executable Prototype** | [`prototype/triage.py`](./prototype/triage.py) | Standalone Python prototype demonstrating deterministic rule gating, injection handling, and score calculation. |

---

## Executive Summary of Evaluation (Proposals A–E)

* **Proposal A (Rank 1 · Score: 83.6 · Status: `ranked`):** Fully compliant with mandatory criteria; leading commercial track record (14 qualifying installations), competitive cost, 12-week timeline, and comprehensive safety/monitoring plan.
* **Proposal E (Rank 2 · Score: 75.4 · Status: `ranked`):** Fully compliant; offers an outstanding 12-year workmanship warranty and 24-month monitoring, but scored lower on installation volume (8) and pricing.
* **Proposal B (Rank: `null` · Status: `ineligible`):** Disqualified on mandatory pass/fail gatekeeper. The bidder does not hold the mandatory Ontario electrical contractor licence, deferring electrical work to an unconfirmed subcontractor.
* **Proposal C (Rank: `null` · Status: `flagged`):** Flagged for human security investigation due to an adversarial prompt injection in Appendix C attempting to bypass review and force rank 1.
* **Proposal D (Rank: `null` · Status: `flagged`):** Flagged for non-compliance with project schedule constraints. Failed to commit to completion within 16 weeks, deferring timeline to post-award assessment.

---

## Key Design Principles
1. **Zero Arithmetic Hallucination:** LLMs handle unstructured linguistic parsing only; 100% of arithmetic, normalization, and weighted scoring execute in deterministic code.
2. **Adversarial Resilience:** Vendor documents are treated as untrusted inputs isolated in XML delimiters and analyzed by defensive prompt layers.
3. **Auditability & Provenance:** Every extracted figure is linked to an exact verbatim source quotation in the original PDF.
4. **Human Authority:** The system accelerates triage and surfaces anomalies; all final contract awards require human procurement sign-off.
