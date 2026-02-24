# Evaluation Cycle 1 — BIAN Framework: Asset Management Business Process Architecture
## Self-Reinforcement Training: FinTech/Asset Executive Director Panel
### Document Version: 1.0 | Evaluation Date: February 2026

---

## Evaluation Panel

| Panelist | Role | Background | Scoring Weight |
|---|---|---|---|
| **Panelist A** | Principal Solutions Architect | 18 years asset management technology; BIAN certified practitioner; former State Street and Fidelity architecture lead | 28% |
| **Panelist B** | Principal Platform Engineer | 15 years capital markets engineering; Aladdin integrations, Salesforce FSC, Kafka CDC pipelines | 25% |
| **Panelist C** | Principal AI/Data Architect | 12 years financial services data mesh, RAG architectures, LLM integration in regulated environments | 25% |
| **Panelist D** | Senior Engineering Manager | 10 years managing client technology programs at asset managers; business partnership, stakeholder communication | 22% |

---

## Scoring Rubric: 16 Dimensions

| Dimension | Weight | Score (A) | Score (B) | Score (C) | Score (D) | Weighted Score |
|---|---|---|---|---|---|---|
| 1. BIAN Framework Accuracy & v12 Alignment | 9% | 8.5 | 8.0 | 9.0 | 8.5 | **0.76** |
| 2. Service Domain Boundary Definition | 8% | 8.0 | 8.5 | 7.5 | 8.0 | **0.65** |
| 3. Platform-to-BIAN Mapping Precision | 8% | 9.0 | 8.5 | 9.0 | 9.0 | **0.71** |
| 4. AI Agent Architecture (DDQ/RFP, BIAN Domains) | 8% | 8.5 | 8.5 | 9.5 | 9.0 | **0.71** |
| 5. Data Architecture (Bronze/Silver/Gold → BIAN) | 8% | 9.0 | 9.5 | 9.0 | 8.5 | **0.73** |
| 6. Client Onboarding Orchestration Model | 7% | 8.0 | 8.5 | 7.5 | 8.0 | **0.57** |
| 7. Client Reporting & GIPS as Shared Service | 7% | 9.0 | 8.0 | 8.5 | 9.0 | **0.61** |
| 8. Sales Enablement + Content Governance | 8% | 8.0 | 8.5 | 8.5 | 9.0 | **0.68** |
| 9. End-to-End Process Flow Completeness | 7% | 8.5 | 8.0 | 8.0 | 8.5 | **0.59** |
| 10. Nomura Context Precision | 7% | 8.0 | 8.0 | 8.0 | 8.5 | **0.56** |
| 11. Interview Talking Points Sharpness | 7% | 8.5 | 9.0 | 8.5 | 8.0 | **0.60** |
| 12. Kathleen McNamara Engagement Model | 6% | 8.0 | 7.5 | 9.0 | 8.5 | **0.50** |
| 13. Stuart Mumley Engagement Model | 6% | 8.5 | 9.0 | 8.0 | 8.0 | **0.51** |
| 14. Shaw Levin Data Architecture Alignment | 4% | 8.5 | 9.0 | 9.0 | 8.5 | **0.34** |
| 15. Trade Execution & Settlement Coverage | 4% | 7.0 | 8.0 | 7.5 | 7.5 | **0.30** |
| 16. Vendor Evaluation BIAN Integration | 3% | 8.0 | 7.5 | 8.0 | 8.5 | **0.24** |
| **TOTAL** | **107% → normalized 100%** | | | | | **8.56 / 10.00** |

**Normalized Score: 8.56 / 10.00 ✗ (Below 9.8 threshold — improvements required)**

---

## Panel Consolidated Feedback

### Panelist A — Principal Solutions Architect

**Strengths:**
- Service landscape ASCII diagram is the strongest element. The three-tier mapping (Front Office / Middle Office / Infrastructure) is BIAN-correct and Nomura-specific. A hiring committee would recognize this as practitioner-level framing.
- Platform-to-BIAN mapping table (Section 12) is interview-ready. Column structure (Platform → BIAN Domain → Behaviors → Integration Boundary) is precisely the format a CISO or CTO uses for vendor governance conversations.
- GIPS-as-shared-service insight (Section 8) demonstrates architectural reasoning beyond most candidate responses. This is the kind of point that changes the panel's impression of a candidate's depth.

**Gaps requiring improvement (Cycle 1 → Cycle 2):**

1. **BIAN Service Operation types not defined.** BIAN v12 defines four service operation types: `initiate`, `execute`, `request`, `notify`. The document uses qualifiers correctly but does not name the operation type classification. An ED candidate at BIAN-adopting institutions (State Street, Standard Chartered) would reference operation types when discussing API design.

2. **Party Reference Data (Section 10) — LEI specificity insufficient.** The document mentions LEI as the canonical identifier but does not reference the GLEIF registry, LEI issuance by endorsed organizations (including Nomura's own LEI), or the operational practice of validating LEI at onboarding via GLEIF API. Stuart specifically will test on party data governance at advisor level (RIA, IAR, CRD number as secondary identifier).

3. **Flow 3 (AI DDQ Agent) — confidence scoring mechanism undefined.** The flow describes "confidence scoring for each answer" but does not specify the mechanism: retrieval similarity score from vector store (cosine distance threshold), LLM self-grading prompt, or ensemble approach. Kathleen will probe on this in an AI governance discussion.

4. **KYC shared service reuse (Section 5) — counterparty KYC missing.** The onboarding architecture addresses issuer-side KYC but does not reference FCA/SEC beneficial ownership requirements under AML5D or FinCEN CDD. For Kathleen's Macquarie credit derivatives background, the counterparty verification step in trading relationships is as important as client KYC.

5. **Section 6 (Investment Portfolio Management) data boundary** — the phrase "the Head of Client Technology owns the integration layer and everything downstream" is correct but needs quantified SLA commitments: what is the contractual SLA for Aladdin-to-Gold tier latency for client portal vs. GIPS cutoff?

---

### Panelist B — Principal Platform Engineer

**Strengths:**
- The CDC-via-Kafka data flow in Section 10 is technically accurate and matches Aladdin's supported outbound integration pattern (Aladdin Data Cloud / Aladdin Data Access). Using "CDC via Kafka" without specifying the Aladdin connector type (REST Polling vs. native CDC) is a minor gap but the architecture pattern is right.
- Step Functions onboarding orchestration in Section 5 is correctly framed as an orchestration layer, with the right separation between orchestration logic and provisioning APIs. The explicit 5-step workflow with parallel-capable design (Steps 1–3 can run in parallel) is an ED-level architectural insight.
- Salesforce FSC integration boundary table (Section 3) — clearly separating what CRM stores vs. reads vs. never owns is exactly the right governance framing for a conversation with Stuart.

**Gaps:**

1. **MSK (Managed Streaming for Kafka) vs. standard Kafka not differentiated.** In the context of Nomura's AWS presence, MSK is the deployment platform. The document should specify MSK topic naming conventions, partition strategy for Aladdin event streams, and the consumer group architecture for multiple downstream subscribers (GIPS, Vermilion, portal).

2. **ADX zero-copy mechanism needs elaboration.** Section 10 correctly references ADX zero-copy but does not specify the revision-based subscription model, S3 bucket location constraints (same region), and the Lake Formation integration for access revocation when entitlements change.

3. **Step Functions orchestration error handling absent.** The onboarding flow does not specify failure states: what happens when KYC screening returns a hit? When DocuSign times out? When custodian account provisioning fails? A production-ready orchestration design must specify compensating transactions and human escalation paths.

---

### Panelist C — Principal AI/Data Architect

**Strengths:**
- Section 9 (Content Governance as Distinct BIAN Service) is the most strategically important insight in the document. The observation that DDQ agent and Seismic consume from the same governance service — and that managing them separately creates regulatory drift — is exactly the kind of architectural point that differentiates a technology strategist from a platform administrator.
- Flow 3 (AI DDQ Agent) correctly models the agent as spanning multiple BIAN domains (Document Management, Knowledge Management, AI-Assisted Processing, Compliance Review) rather than being a monolithic tool. This is the right framing for Kathleen's AI governance questions.
- Party Reference Data section in Section 10 — the identity mesh problem framing (four records for one client) is exactly the right business narrative for a data architecture conversation with Shaw.

**Gaps:**

1. **RAG architecture for DDQ agent missing.** Knowledge Management BIAN domain references "approved knowledge base" but does not specify the vector store (Pinecone, PGVector, OpenSearch k-NN), embedding model, chunking strategy, or retrieval window. Kathleen has built RAG pipelines at Macquarie and will probe on this.

2. **Gold layer data product ownership undefined.** The Gold layer correctly feeds multiple consumers but does not define who owns each data product: who owns the GIPS composite data product? Who owns the RM-readiness insight product? Data product ownership is the governance operationalization of the data mesh, and Shaw will specifically test on this.

3. **Section 6 (Portfolio Management boundary)** misses an important nuance: the Aladdin Data Cloud product differentiates between Order Management API, Portfolio Accounting API, and Risk Analytics API. These have different latency SLAs and different access control models. The boundary description should reference which Aladdin API surface the data integration layer connects to.

---

### Panelist D — Senior Engineering Manager

**Strengths:**
- Interview talking points (Section 13) are conversational and deployable. The talking point for Shaw on data architecture is technically dense in the right way — it uses BIAN language without being academic.
- The three-channel structure (institutional / advisor-wealth / end client) is consistently threaded through the process flows. A hiring committee testing business empathy will see that this candidate understands the revenue model, not just the platform.
- What to distinguish from: most ED candidates in this interview will give a generic "Salesforce is our CRM" answer. The system-of-engagement vs. system-of-record distinction in Section 3 immediately positions this candidate at a different level.

**Gaps:**

1. **Operating model for BIAN service ownership missing.** Who owns the Content Governance service? Who owns the GIPS composite service? Without naming the organizational owner (a named team or role), BIAN service domains are abstract architecture rather than operable services. This is the SEM's lens — Kathleen will test on who is accountable for the DDQ knowledge base quality.

2. **Change management narrative for adopting BIAN vocabulary absent.** When introducing BIAN vocabulary to existing teams (Stuart's Salesforce admins, reporting team), there is a change management challenge. The document does not address how to migrate existing team vocabulary to BIAN service domain language — which is an ED execution question, not just an architecture question.

---

## Improvement Priorities for Cycle 2

| Priority | Gap | Target Resolution |
|---|---|---|
| P1 | BIAN v12 operation types (initiate/execute/request/notify) | Add operation type column to behavior qualifier tables |
| P2 | RAG architecture for DDQ/RFP Knowledge Management domain | Add vector store, embedding model, retrieval specifics to Flow 3 |
| P3 | GLEIF/LEI specificity for Party Reference Data | Add GLEIF API integration, CRD secondary identifier for advisor channel |
| P4 | Onboarding error handling (compensating transactions) | Add failure state handling to Step Functions flow in Section 5 |
| P5 | Data product ownership model (Gold layer) | Name owner for each Gold layer data product; reference data mesh contracts |

---

## Cycle 1 Overall Assessment

The BIAN framework document demonstrates strong architectural breadth across all eight service domains. The platform-to-BIAN mapping table and GIPS-as-shared-service insight are interview-differentiating. The primary weaknesses are in operational specificity (operation types, error handling, data product ownership) and AI architecture depth (RAG mechanism, confidence scoring). These are addressable gaps, not structural deficiencies.

**Cycle 1 Score: 8.56 / 10 — DOES NOT MEET 9.8 certification threshold — Proceed to Cycle 2**
