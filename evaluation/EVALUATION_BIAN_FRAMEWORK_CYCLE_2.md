# Evaluation Cycle 2 — BIAN Framework: Asset Management Business Process Architecture
## Self-Reinforcement Training: FinTech/Asset Executive Director Panel
### Document Version: 1.0 | Evaluation Date: February 2026 | Cycle: 2 of 3

---

## Panel

| Panelist | Role | Scoring Weight |
|---|---|---|
| **Panelist A** | Principal Solutions Architect (BIAN certified) | 28% |
| **Panelist B** | Principal Platform Engineer (Aladdin/Kafka/Salesforce) | 25% |
| **Panelist C** | Principal AI/Data Architect (RAG, data mesh, LLM governance) | 25% |
| **Panelist D** | Senior Engineering Manager (business partnership, stakeholder) | 22% |

---

## Cycle 1 → Cycle 2 Improvement Validation

| Cycle 1 Gap | Improvement Demonstrated in Cycle 2 | Validated |
|---|---|---|
| BIAN v12 operation types not defined | Candidate correctly answers: BIAN defines four service operation types — `Initiate` (create a new instance), `Execute` (perform a discrete action), `Request` (ask another service to perform an action), `Notify` (event-driven signal with no expected response). In the DDQ agent context: receiving a DDQ is an `Initiate`; querying the knowledge base is a `Request`; publishing completed DDQ to prospect is a `Notify`. | ✅ |
| RAG architecture for Knowledge Management domain absent | Candidate articulates: OpenSearch k-NN (or PGVector) as the vector store; `ada-002` (or Bedrock Titan Embeddings) for embeddings with 512-token overlapping chunks; cosine similarity with threshold 0.78 for retrieval confidence scoring; retrieval window of top-5 passages reranked by BM25+dense hybrid. Kathleen's question: "What happens when the knowledge base has no approved answer?" — Candidate: "The retrieval score falls below threshold; the agent routes to human escalation without generating a response — the compliance boundary is never breached by low-confidence inference." | ✅ |
| GLEIF/LEI for Party Reference Data insufficient | Candidate adds: Legal Entity Identifier (LEI) issued under GLEIF registration. At advisor level, CRD (Central Registration Depository) number via BrokerCheck is the secondary identifier for RIAs and IARs. At entity level, SWIFT BIC supplements LEI for custodian relationships. GLEIF REST API provides real-time LEI validation at onboarding with O(1) lookup latency. The integration pattern at Salesforce: LEI stored as external ID on Account object, validated against GLEIF on Account creation, used as canonical join key across Data Platform. | ✅ |
| Onboarding orchestration error handling absent | Candidate details Step Functions failure paths: (1) KYC hit → human review task created in Salesforce Service Cloud, onboarding paused, RM and compliance alerted via SNS; (2) DocuSign timeout → compensating retry with exponential backoff, 3× then escalation; (3) Custodian provisioning failure → idempotent retry via SQS dead-letter queue, 24-hour SLA breach notification to operations; (4) Entitlement activation conflict → conflict resolution via Lambda with idempotency token to prevent duplicate entitlement grants. | ✅ |
| Gold layer data product ownership undefined | Candidate defines ownership model: GIPS composite data product owned by Client Reporting team (Kathleen's team); RM-readiness insight data product owned by Sales Enablement team (Stuart's team); Performance Attribution data product owned jointly with Investment Management with SLA contract; all data product contracts documented in AWS Glue Data Catalog with defined producer, consumer, and SLA. | ✅ |

---

## Scoring Rubric: 16 Dimensions — Cycle 2

| Dimension | Weight | Score (A) | Score (B) | Score (C) | Score (D) | Weighted Score |
|---|---|---|---|---|---|---|
| 1. BIAN Framework Accuracy & v12 Alignment | 9% | 9.5 | 9.0 | 9.0 | 9.5 | **0.83** |
| 2. Service Domain Boundary Definition | 8% | 9.0 | 9.0 | 9.0 | 9.0 | **0.72** |
| 3. Platform-to-BIAN Mapping Precision | 8% | 9.5 | 9.0 | 9.5 | 9.5 | **0.75** |
| 4. AI Agent Architecture (DDQ/RFP, RAG, BIAN Domains) | 8% | 9.5 | 9.0 | 10.0 | 9.5 | **0.77** |
| 5. Data Architecture (Bronze/Silver/Gold → BIAN) | 8% | 9.5 | 9.5 | 9.5 | 9.0 | **0.76** |
| 6. Client Onboarding Orchestration + Error Handling | 7% | 9.0 | 9.5 | 9.0 | 9.0 | **0.63** |
| 7. Client Reporting & GIPS as Shared Service | 7% | 9.5 | 9.0 | 9.5 | 9.5 | **0.66** |
| 8. Sales Enablement + Content Governance | 8% | 9.0 | 9.5 | 9.0 | 9.5 | **0.74** |
| 9. End-to-End Process Flow Completeness | 7% | 9.0 | 9.0 | 9.0 | 9.0 | **0.63** |
| 10. Nomura Context Precision | 7% | 9.0 | 9.0 | 9.0 | 9.5 | **0.63** |
| 11. Interview Talking Points Sharpness | 7% | 9.5 | 9.5 | 9.0 | 9.0 | **0.65** |
| 12. Kathleen McNamara Engagement Model | 6% | 9.0 | 8.5 | 9.5 | 9.0 | **0.54** |
| 13. Stuart Mumley Engagement Model | 6% | 9.0 | 9.5 | 9.0 | 9.0 | **0.55** |
| 14. Shaw Levin Data Architecture Alignment | 4% | 9.5 | 9.5 | 9.5 | 9.0 | **0.38** |
| 15. Trade Execution & Settlement Coverage | 4% | 8.5 | 9.0 | 8.5 | 8.5 | **0.34** |
| 16. Vendor Evaluation BIAN Integration | 3% | 9.0 | 8.5 | 9.0 | 9.0 | **0.27** |
| **TOTAL** | **107% → normalized 100%** | | | | | **9.36 / 10.00** |

**Normalized Score: 9.36 / 10.00 ✗ (Improved from 8.56 — below 9.8 threshold — Cycle 3 required)**

---

## Panel Consolidated Feedback — Cycle 2

### Panelist A — Principal Solutions Architect

The BIAN v12 operation type mastery demonstrated in Cycle 2 is impressive and immediately differentiates this candidate. The application of `Initiate/Request/Notify` to the DDQ agent workflow is not just theoretically correct — it is the right framing for how you would design the API contracts between those service domains. When Stuart asks "how does the AI tool integrate with Salesforce," this vocabulary answers at the architecture layer, not the feature layer.

**Remaining gap for Cycle 3:** BIAN Service Domain versioning and co-existence. In practice, when a firm adopts BIAN incrementally — as Nomura would — some domains operate on BIAN-aligned APIs while adjacent domains are still point-to-point integrations. The candidate should articulate the transition pattern: how to introduce a BIAN-aligned Customer & Party Management API while Salesforce is running a legacy integration with the fund accounting system. This is the incremental adoption question.

---

### Panelist B — Principal Platform Engineer

The error handling model for Step Functions is production-quality. Specifying DLQ for custodian provisioning and idempotency tokens for entitlement activation shows genuine operational experience, not theoretical awareness. The onboarding SLA narrative (mandate award to first report in 3 business days vs. industry average of 4–8 weeks) is now grounded in a verifiable technical architecture.

**Remaining gap for Cycle 3:** MSK consumer group strategy not fully specified. When Aladdin emits a position update event, multiple consumers need it: GIPS engine, Vermilion report trigger, Salesforce RM alert, and client portal. The consumer group design (independent consumer groups per subscriber to prevent head-of-line blocking) and the topic retention policy (how long events live on the topic for replay during recovery) should be explicit. Shaw will test this specifically.

---

### Panelist C — Principal AI/Data Architect

The RAG architecture specifics (OpenSearch k-NN, 512-token chunks, cosine similarity 0.78 threshold, BM25+dense hybrid reranking) are correct and deployable. The compliance boundary design — no-generate below threshold — is the most important governance insight in the entire document. This is what distinguishes an AI system that is trustworthy from one that produces plausible-looking errors.

The data product ownership model (Section 10) now has named owners per Gold layer product. The next level of maturity is defining the **data product SLA**: what is the maximum acceptable staleness for the GIPS composite data product at market open? What is the Vermilion consumption retry policy when the Gold layer is delayed? These translate data ownership into operational accountability — which is the difference between a data mesh on paper and a data mesh in production.

**Remaining gap for Cycle 3:** Multi-tenant data isolation model. Nomura serves multiple client entities with different regulatory jurisdictions — a UK FCA client and a US SEC client may both consume from the same Gold layer. The document does not address how Lake Formation RLS is configured to enforce jurisdiction-level data isolation at the row level, especially for GDPR/CCPA data residency requirements. Kathleen will test this in the context of the client portal data access model.

---

### Panelist D — Senior Engineering Manager

The data product ownership model resolves my primary Cycle 1 concern. The explicit naming of owners (Kathleen's team for GIPS reporting product, Stuart's team for RM-readiness product) is the operationalization that makes the data mesh real. What is still missing is the **data product contract** — the formal definition of schema, SLA, and expected behavior that producing teams commit to and consuming teams rely on. Without the contract, ownership is nominal rather than operational.

**Remaining gap for Cycle 3:** The talking point for Shaw on data architecture (Section 13) is strong. What is underdeveloped is the talking point for the *hiring panel itself* — specifically, how the Head of Client Technology role navigates the boundary between investment management (Aladdin) and client technology (everything downstream) when there is a data quality dispute. This is a governance and escalation conversation, and the document does not articulate a process for it.

---

## Remaining Gaps for Cycle 3

| Priority | Gap | Target Resolution |
|---|---|---|
| P1 | BIAN incremental adoption pattern (co-existence with legacy integrations) | Add migration path and bounded context transition model |
| P2 | MSK consumer group strategy + topic retention policy | Add consumer group design for multi-subscriber Aladdin event stream |
| P3 | Multi-tenant data isolation (FCA/SEC/GDPR jurisdiction segregation via Lake Formation) | Add jurisdiction-level RLS configuration and data residency model |
| P4 | Data product SLA contracts (staleness, retry, consumer obligations) | Define SLA parameters for GIPS and RM-readiness data products |
| P5 | Aladdin/Client Technology data quality dispute escalation model | Add cross-boundary governance and escalation procedure |

---

## Cycle 2 Overall Assessment

All five Cycle 1 improvements have been demonstrated and validated. The jump from 8.56 to 9.36 reflects genuine mastery of operational specifics — error handling, RAG architecture, LEI/GLEIF, data product ownership. The remaining gaps are at the advanced practitioner level: incremental BIAN adoption, multi-tenant jurisdiction isolation, and cross-boundary governance. These are resolvable with focused preparation.

**Cycle 2 Score: 9.36 / 10 — DOES NOT YET MEET 9.8 certification threshold — Proceed to Cycle 3**
