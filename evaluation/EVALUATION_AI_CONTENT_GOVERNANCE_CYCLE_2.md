# Evaluation Cycle 2 — AI Content Governance Enhancement
## Documents Under Review
- `CUSTOMER_CENTRIC_DATA_STRATEGY_CQRS.md`
- `CUSTOMER_CENTRIC_AI_STRATEGY.md`
- `BUSINESS_PARTNERSHIP_README.md`

**Evaluation Date:** February 2026
**Evaluation Cycle:** 2 of 3
**Evaluator Panel:** Kathleen Hess McNamara (35%) · Stuart Mumley (35%) · Technical Architecture Board (30%)
**Previous Cycle Score:** 8.51/10.00
**Target:** All three evaluators individually ≥ 9.75; weighted composite ≥ 9.80

---

## Cycle 1 Remediation Verification

| Gap ID | Action Taken | Status |
|---|---|---|
| GAP-K1 | Added 90% KPI connection: "5 Pillars not overhead — mechanism that makes 90% turnaround defensible" | ✅ RESOLVED |
| GAP-K2 | Added executive summary box before Pillar 1 technical detail in CQRS 5.3 | ✅ RESOLVED |
| GAP-K3 | Added "Current State at Nomura" anchoring row in both CQRS and AI Strategy Three Layers tables | ✅ RESOLVED |
| GAP-S1 | CQRS already uses `knowledge_base_correction_dag` (snake_case); AI Strategy standardized to match | ✅ RESOLVED |
| GAP-S2 | Added `napce_kb_expiry_purge` Lambda name and `napce_kb_expiry_dlq` DLQ to AI Strategy RAG Mesh table | ✅ RESOLVED |
| GAP-S3 | Added 72-hour timeout and escalation path to BUSINESS_PARTNERSHIP_README Pillar 4 | ✅ RESOLVED |
| GAP-T1 | Added Redis KEYS pattern `kb_query:*:{chunk_id}*` to CQRS Pillar 3 cache invalidation | ✅ RESOLVED |
| GAP-T2 | Added Lambda DLQ `napce_kb_expiry_dlq` + CloudWatch alarm to CQRS Pillar 1 | ✅ RESOLVED |
| GAP-T3 | Updated QLDB references from "blockchain-verified" to "SHA-256 cryptographic digest chain; independently verifiable" in all three documents | ✅ RESOLVED |

---

## Evaluator Scores — Cycle 2

### Kathleen Hess McNamara (ED, Client Experience & AI) — Weight: 35%

**Raw Score: 9.35 / 10.00**

#### Improvements Since Cycle 1

**GAP-K1 Resolution — 90% KPI Connection is Now Explicit and Compelling**
The addition of the critical business case connection paragraph in BUSINESS_PARTNERSHIP_README Section 6 is strong: *"A 90% faster DDQ that contains a stale policy citation is worse than a slow DDQ — it combines speed with error at scale."* This framing precisely captures the governance risk that Kathleen has lived in production. The 5 Pillars are now positioned as the enabler of the 90% KPI claim, not a separate compliance requirement.

**GAP-K2 Resolution — Executive Summary Box is Scannable and Executive-Grade**
The five-pillar executive summary in CQRS Section 5.3 — one sentence per pillar in a blockquote box — allows Kathleen to grasp the governance architecture in 30 seconds before reading the technical detail. This is the correct information hierarchy for a mixed-audience document.

**GAP-K3 Resolution — Three Layers Current State Anchoring is Precise**
Both the CQRS and AI Strategy Three Layers tables now explicitly state: "Kathleen operationalized Layer 1 at Macquarie. Stuart has built Layer 2. Layer 3 requires the 5 Pillars to be operational." This sequencing establishes layered governance maturity in terms Kathleen will immediately recognize as coming from operational experience.

#### Residual Gaps — Cycle 2

**GAP-K4: BUSINESS_PARTNERSHIP_README "What to Say in the Room" quote could reference ISO/IEC 42001 by name**
The quote is now substantially stronger — it references "Current Answers perimeter," "Ownership layer," and "Audit trail" explicitly. However, the compliance framework references (ISO/IEC 42001, NIST AI RMF) are in the table below the quote but not named in the spoken language of the prepared statement. Kathleen will be in a room with professionals who will recognize these frameworks by name. Citing them in the spoken context, not just the reference table, adds a layer of credibility.

*Remediation required: Add one sentence to the "What to Say in the Room" prepared statement referencing ISO/IEC 42001 as the standard the governance architecture is designed to satisfy.*

---

### Stuart Mumley (Director, Client Platforms) — Weight: 35%

**Raw Score: 9.45 / 10.00**

#### Improvements Since Cycle 1

**GAP-S1+S2 Resolution — Operational Consistency Across Documents**
The standardized `knowledge_base_correction_dag` naming and the addition of `napce_kb_expiry_purge` Lambda function and `napce_kb_expiry_dlq` DLQ make the architecture across all three documents operationally coherent. Stuart's team can read the same function name in CQRS, AI Strategy, and Business Partnership README and understand they are referring to the same infrastructure component.

**GAP-S3 Resolution — 72-Hour Timeout Policy is Now Operational**
The addition of the escalation path in BUSINESS_PARTNERSHIP_README Pillar 4 — "72 hours → senior reviewer; 96 hours → Head of Client Technology CloudWatch alarm" — transforms the HITL description from a process aspiration into an operational policy with defined SLAs. Stuart will recognize this as production-ready specification language.

**AI Strategy NAPCE Section — RAG Mesh Table is Architecturally Complete**
The enhanced RAG Mesh comparison table with `napce_kb_expiry_purge` Lambda and DLQ specification provides the architectural detail needed for a platform team to implement Pillar 1 as specified. The note that the DLQ "ensures no silent expiry failures" is the correct quality-of-service framing.

#### Residual Gaps — Cycle 2

**GAP-S4: CQRS Section 5.3 Pillar 2 Metadata-as-Code block could show the AWS Glue ingestion enforcement**
The metadata-as-code JSON block is clear and shows the chunk structure. However, the CQRS document does not specify *how* the metadata is enforced at ingestion — specifically that the AWS Glue job validates the presence of `owner_entra_id`, `owner_salesforce_id`, and `status` fields before vectorization. A missing-field validation at ingestion (dead-letter queue for non-compliant objects) is the correct engineering control. Without it, a document could enter the knowledge base without an owner.

*Remediation required: Add AWS Glue ingestion validation note to CQRS Pillar 2: "AWS Glue job validates mandatory metadata fields at ingestion. Objects missing `owner_entra_id`, `status`, or `review_due` are routed to S3 DLQ `napce_kb_metadata_dlq` — never vectorized into OpenSearch."*

---

### Technical Architecture Board — Weight: 30%

**Raw Score: 9.40 / 10.00**

#### Improvements Since Cycle 1

**GAP-T1 Resolution — Cache Invalidation Precision is Production-Grade**
The Redis KEYS pattern `kb_query:*:{chunk_id}*` is the correct scope for precise cache invalidation. Specifying the KEYS pattern prevents over-invalidation (which would cause unnecessary cache misses) and under-invalidation (which would serve stale cached responses after a knowledge base correction). The comment "no global cache flush" explicitly rules out the naive but incorrect approach.

**GAP-T2 Resolution — Lambda DLQ Architecture is Complete**
The addition of `napce_kb_expiry_dlq` SQS dead-letter queue and `CloudWatch alarm: DLQ MessageCount > 0 → PagerDuty` closes the governance gap identified in Cycle 1. The explicit "ensures no silent expiry failures" language demonstrates that the architecture was designed with operational failure modes in mind, not just the happy path.

**GAP-T3 Resolution — QLDB Technical Accuracy Improved**
Updating from "blockchain-verified" to "SHA-256 cryptographic digest chain; independently verifiable without distributed consensus" is technically accurate and appropriate for a regulated financial services architecture document. It correctly distinguishes QLDB from distributed blockchain implementations while preserving the key property (cryptographic verification).

#### Residual Gaps — Cycle 2

**GAP-T4: AI Strategy Pillar 5 Audit Trail section does not specify Snowflake Immutable Table DDL pattern**
The Pillar 5 Three-Payload Architecture is strong, but the AI Strategy document does not specify how Snowflake Immutable Tables enforce append-only behavior. The DDL pattern (`ALTER TABLE SET DATA_RETENTION_TIME_IN_DAYS = 0; CLUSTER BY (rfp_id, payload_type)`) and the enforcement mechanism (Snowflake cannot `UPDATE` or `DELETE` from an immutable table without a governance override) would strengthen the architecture's credibility with the Technical Architecture Board.

*Remediation required: Add Snowflake Immutable Table specification note: `audit_trail` table with `INSERT-only` privilege grants; `DELETE` and `UPDATE` privileges revoked from all roles. Enforced at the Snowflake RBAC layer, not application layer.*

---

## Cycle 2 Composite Score

| Evaluator | Raw Score | Weight | Weighted Score |
|---|---|---|---|
| Kathleen Hess McNamara | 9.35 | 35% | 3.27 |
| Stuart Mumley | 9.45 | 35% | 3.31 |
| Technical Architecture Board | 9.40 | 30% | 2.82 |
| **COMPOSITE** | | **100%** | **9.40 / 10.00** |

**Cycle 2 Result: 9.40/10.00 — DOES NOT YET MEET CERTIFICATION THRESHOLD (≥9.80)**
**Progress from Cycle 1: +0.89 points**

---

## Remediation Plan for Cycle 3

| Gap ID | Evaluator | Priority | Required Action |
|---|---|---|---|
| GAP-K4 | Kathleen | HIGH | Add ISO/IEC 42001 by name to "What to Say in the Room" prepared statement |
| GAP-S4 | Stuart | HIGH | Add AWS Glue ingestion validation note to CQRS Pillar 2 with DLQ spec |
| GAP-T4 | TAB | HIGH | Add Snowflake Immutable Table INSERT-only privilege grant pattern to AI Strategy Pillar 5 |

---

## Status

**Cycle 1: COMPLETE — 8.51/10.00**
**Cycle 2: COMPLETE — 9.40/10.00**
**Cycle 3: IN PROGRESS**
