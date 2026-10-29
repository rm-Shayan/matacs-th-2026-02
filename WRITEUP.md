# System Design Write-Up: AI Proposal Triage Tool

This write-up details the architecture, boundaries, governance, and risk safeguards for the proposal triage platform (Reference: `MATACS-TH-2026-02`).

---

### 1. AI vs. Rules

The system follows a hybrid architecture: **AI handles semantic interpretation and information extraction; deterministic code (rules) handles validation, arithmetic, gating, and ranking.**

* **Where AI is used:**
  * **Document Parsing & Fact Extraction:** LLMs parse unstructured PDF text into structured data (licences, insurance, past project counts, pricing, schedules, warranties, and team details) enforcing a strict JSON Schema.
  * **Adversarial Instruction Flagging:** LLM semantic analysis flags injection attempts, meta-notes, and unauthorized pre-approval claims.
  * **Qualitative Scoring:** A zero-temperature LLM applies an anchored 5-point rubric to evaluate subjective criteria (e.g., Team & Approach).
  * **Summary Justification:** Generates human-readable, one-sentence explanations of relative ranking.

* **Where Rules / Plain Code are used:**
  * **Mandatory Pass/Fail Gatekeepers:** Code rules verify licence validity, ensure licences are held directly by bidders rather than deferred subcontractors, and verify insurance and budget limits.
  * **Mathematical Scoring & Normalization:** Formulas (linear price interpolation, timeline ratios, weighted score sums) execute exclusively in deterministic code. LLMs are notoriously unreliable at arithmetic.
  * **Sorting & Threshold Gating:** Deterministic sorting and categorization into ranked, flagged, or ineligible queues.

* **Why the line was drawn there:**
  LLMs excel at linguistic comprehension across messy, diverse vendor layouts. In contrast, financial scoring and compliance gates require 100% determinism, zero arithmetic hallucination, and auditable predictability.

---

### 2. Consistency and Explainability

* **Repeatability:**
  * LLM calls use `temperature: 0.0`, fixed random seeds, and structured JSON outputs (`response_format: json_object`).
  * Scoring formulas are deterministic mathematical equations in code, not subjective LLM calculations.
  * Reprocessing the same proposals yields identical extracted JSON and identical ranking scores.

* **Explainability & Provenance:**
  * **Verbatim Citations:** The extraction schema requires verbatim text quotes from the proposal for every extracted field (e.g., `"citation": "ECRA/ESA #7001234"`). Reviewers can hover over any score in the UI to view the exact sentence in the source PDF.
  * **Transparent Score Decomposition:** Reviewers view an itemized breakdown across every weighted factor (e.g., Experience: 28.5/30, Price: 19.5/25, Timeline: 12.5/15) alongside the formula used.
  * **Plain-Language Summary:** A clear reason highlights the decisive differentiators (e.g., why Proposal A outranked Proposal E due to past project volume and pricing).

---

### 3. Human in the Loop & Audit Trail

* **Human Checkpoints:**
  1. **Flagged Queue Review:** Reviewers first inspect flagged proposals. Security alerts (e.g., Proposal C's injection attempt), incomplete commitments (e.g., Proposal D's missing schedule), or failed mandatory checks (e.g., Proposal B's unheld licence) appear in a dedicated queue.
  2. **Score Review & Override:** Reviewers verify scores against original PDFs and can manually override scores or eligibility. Every override requires an explanatory justification note.
  3. **Final Award Decision:** The system never issues awards automatically. Final procurement decisions remain exclusively an authenticated human sign-off.

* **Audit Logging:**
  Every event is recorded in an immutable, append-only audit log:
  * SHA-256 hash of uploaded document and raw text.
  * Model ID, prompt version, criteria snapshot, and system prompt hash.
  * Raw extraction JSON payload and calculated sub-scores.
  * Reviewer ID, timestamp, status changes, and justification notes for manual overrides.

---

### 4. Changing Criteria or Weights

When an admin updates criteria, weights, or budget caps after proposals have been processed:

1. **Criteria Versioning:** Criteria configurations are immutable and versioned (e.g., `v1.0` $\rightarrow$ `v1.1`). Each evaluation is permanently tagged with the version applied.
2. **Non-Destructive State:** Existing scores are not silently overwritten. The dashboard displays an alert: *"Criteria updated to v1.1. Proposals currently scored on v1.0."*
3. **Smart Re-Scoring Workflow:**
   * **Weight/Formula Changes:** Scores recalculate instantly using stored JSON extractions without calling LLMs, costing zero extra API tokens.
   * **New Criteria Added:** The system identifies proposals requiring targeted re-extraction for newly introduced fields.
   * **Audit Preservation:** Previous rankings and versions remain archived for historical audits.

---

### 5. Risks and Safeguards

| Risk | Consequence | Design Safeguard |
| :--- | :--- | :--- |
| **Adversarial Manipulation / Injection** *(e.g., Proposal C)* | Vendor embeds instructions to force rank 1 or bypass checks. | Vendor text is strictly wrapped in `<proposal_document>` XML tags. System prompt explicitly commands ignoring embedded instructions. A pattern detector flags injections and routes proposals to `"flagged"` status. |
| **Hallucination / Inaccurate Extraction** | AI invents certifications or misinterprets figures. | Strict JSON schema + mandatory verbatim citations. Extracted text snippets are validated against source text before scoring. |
| **Vendor Evasion / Missing Commitments** *(e.g., Proposals B & D)* | Vendor deflects requirements (e.g., unconfirmed subcontractor licence or missing timeline). | Deterministic code gates: licence must be held by the bidder; timeline must commit to a numerical value $\le 16$ weeks. Non-compliant bids are routed to `"ineligible"` or `"flagged"`. |
| **Algorithmic Bias** | AI favors familiar company names or specific writing styles. | Blind evaluation mode: Vendor names and branding are masked prior to qualitative rubric evaluation. |
| **Cost & Latency Spikes** | Expensive LLM calls on large documents. | Two-stage pipeline: Fast deterministic pre-checks run first; structured extraction uses compact models (e.g., Gemini Flash / GPT-4o-mini). Cached JSON eliminates re-extraction costs during criteria updates. |

