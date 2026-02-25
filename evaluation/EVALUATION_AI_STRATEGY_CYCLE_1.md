# Evaluation Cycle 1 — Customer-Centric AI Strategy
## Five Business Cases on the CQRS Data Mesh

**Document evaluated:** `CUSTOMER_CENTRIC_AI_STRATEGY.md`
**Evaluation date:** February 2026
**Cycle purpose:** Baseline quality assessment — identify gaps before panel presentation

---

## Evaluator Framework

| Evaluator | Role | Weight | Focus Dimensions |
|---|---|---|---|
| **Kathleen Hess McNamara** | ED, Client Experience & AI | 35% | Client journey coherence, AI reliability & HITL design, NPS/CSAT/CES framework, AUM growth narrative |
| **Stuart Mumley** | Director, Client Platforms | 35% | Salesforce BIAN boundaries, API architecture, platform maintainability, MWAA orchestration |
| **Independent Technical Reviewer** | Architecture SME | 30% | AWS service correctness, CQRS read path integrity, security architecture, FinOps |

---

## Evaluator A: Kathleen Hess McNamara
### ED, Client Experience & AI — Weight: 35%

#### Dimension 1: Client Journey Coherence (0–10)
**Score: 8.2**

**Strengths:**
- The five-case taxonomy maps customer lifecycle stages cleanly: Sales (A1), Service (A2), Advisory (A3), Proposals (B), Product Enhancement (C)
- HITL gate is universally applied — no AI case produces client-facing output without human approval; this is architecturally correct for a regulated environment
- The NCIE feedback flywheel connecting chatbot fallbacks → product backlog is a sophisticated closed-loop design that demonstrates genuine understanding of continuous improvement architecture

**Gaps identified:**
1. **Personalization case (A3) lacks specificity on life-event triggers.** The document states "life events detected" but does not enumerate which events trigger which recommendations. Retirement timeline < 5 years and liquidity events are mentioned in the architecture diagram but the narrative section is sparse. A panel question on "how does the system know when a client's life circumstances change?" has no crisp answer.
2. **Cross-case journey stitching is implicit but not diagrammed.** The document describes the flywheel (A1 fallbacks → NCIE → NAIM KB → A1 improved) in Section 5 but there is no single diagram showing the complete cross-case data flow in the context of the client lifecycle. Kathleen would want to see this visually framed as "the client journey across all surfaces."
3. **NPS/CSAT/CES composite improvement mechanism lacks quantified contribution per case.** The 50% target is stated but the attribution (what % comes from NAIM alone vs. NCIE product fixes vs. personalized reports) is not disaggregated. This makes the target hard to defend if challenged on any individual case's contribution.

#### Dimension 2: AI Reliability & HITL Design (0–10)
**Score: 8.9**

**Strengths:**
- Bedrock Guardrails specification per case is detailed — groundedness thresholds, denied topics, PII controls are all present
- The GIPS-SageMaker boundary (Bedrock monitors; SageMaker calculates) is correctly stated and clearly positioned as an architecture principle, not an implementation note
- Three-layer Lemons Table with mitigation architecture for each risk; not just risk identification

**Gaps identified:**
1. **Confidence threshold rationale is not explained.** The DDQ Agent uses 0.85 confidence threshold; the NAIM Bedrock guardrail uses 0.70 groundedness threshold; the chatbot uses 0.75 for DDQ response. Why are these thresholds different? A panel question on threshold calibration — "how did you arrive at 0.85 for DDQ vs. 0.70 for NAIM?" — has no answer in the document.
2. **HITL gate timeout policy is absent.** What happens if an RM does not approve a Next Best Action for 72 hours? Does it expire? Does it convert to a follow-up task? The HITL gate is correctly defined as a governance control, but the operational policy around stale approvals is undefined.

#### Dimension 3: AUM Growth Narrative (0–10)
**Score: 8.4**

**Strengths:**
- Cross-sell ROI is quantified and compelling: $118.8M → $1.03B pipeline (8.7×) from the AI Chatbot source document
- 30% cross-sell conversion improvement is tied to a specific mechanism (churn risk signal → Seismic pitch retrieval → HITL approval)

**Gaps identified:**
1. **The personalization case (A3) has no AUM impact projection.** The personalized portfolio section articulates the mechanism but stops short of an AUM retention or wallet share estimate. If tax-loss harvesting conversations add 50bps to client retention probability, what is the AUM at risk per year? This quantification gap means Case A3 is the weakest case for AUM growth narrative.
2. **The total AI investment ROI across all five cases is not consolidated.** Individual FinOps estimates are scattered across the document. A single "Total investment vs. total expected return" table would strengthen the executive summary considerably.

**Kathleen Subtotal: (8.2 + 8.9 + 8.4) / 3 = 8.50**

---

## Evaluator B: Stuart Mumley
### Director, Client Platforms — Weight: 35%

#### Dimension 1: Salesforce BIAN Boundaries (0–10)
**Score: 8.6**

**Strengths:**
- BIAN service domain assignment for each case is accurate and clearly stated: Sales & Marketing (A1), Client Servicing (A2), Portfolio Management / Wealth Advisory (A3), Product Management / Compliance (B), Product Design / Business Insight (C)
- The "one write surface, one read surface, one HITL gate" architectural principle is clean and correctly applied across all five cases
- Cross-case schema versioning via EventBridge event bus with breaking-change notification policy is noted in the Lemons Table — this demonstrates platform maturity awareness

**Gaps identified:**
1. **Seismic DAM integration boundary with Salesforce is underspecified.** Case A1 states that Seismic pitch decks are retrieved and "attached" to the Salesforce Next Best Action but doesn't clarify: is this a Seismic API call returning a link (recommended) or a file embedded in Salesforce (not recommended for storage hygiene)? Stuart would want to know the precise integration pattern.
2. **Case A3 Salesforce write path is vague.** The personalized quarterly report is delivered to the "Client Portal" but the Salesforce touch — "a report delivery task + suitability note → Salesforce" in the architecture diagram — is not described with enough precision for a platform architect. What SF object? Which fields? What triggers the Task?
3. **Lightning Web Component architecture for NAIM is referenced (Section 3B) but not shown in the unified architecture.** The Section 6 data flow diagram shows all five cases reading from CQRS and writing to Salesforce, but the LWC Co-Pilot layer (Case A2) is not represented. Stuart would want to see the Salesforce Platform Event → API Gateway boundary explicitly diagrammed.

#### Dimension 2: Platform Maintainability & MWAA Orchestration (0–10)
**Score: 8.3**

**Strengths:**
- MWAA DAG trigger specifications for all four NAPCE DAGs are clearly defined (two event-driven, two scheduled)
- Stateless AWS architecture principle is correctly stated: Lambda routes, MWAA orchestrates, no custom application servers
- Saga compensation chain pseudocode direction (reverse order from failed step) is architecturally sound

**Gaps identified:**
1. **MWAA worker scaling configuration is not specified.** The FinOps answer in Section 10 mentions "MWAA m5.xlarge workers" but the concurrency limit (max concurrent DAG runs per type) and auto-scaling triggers are not defined. Stuart would want to see max_active_runs per DAG type documented to validate the 100-proposal/month FinOps estimate.
2. **DAG dependency management across cases is absent.** NAPCE DAGs consume Snowflake Gold data. If the CQRS `gold_refresh` DAG has not completed for the current trading day, should `napce_rfp_proposal` block? This inter-DAG dependency governance is not addressed.
3. **NAIM EventBridge trigger reliability SLA gap.** The Lemons Table (Section 7) mentions "EventBridge weekly trigger fails silently" as a risk with a CloudWatch mitigation, but the document does not specify the monitoring runbook SLA or escalation path beyond "alert on-call team."

#### Dimension 3: API Architecture (0–10)
**Score: 8.1**

**Strengths:**
- CQRS architecture as the Salesforce API throttling solution is correctly positioned: all five cases read from Snowflake, only final outputs write to Salesforce
- Salesforce Bulk API recommendation for high-volume opportunity updates is correctly cited
- Cognito OIDC → Lambda → Snowflake Horizon RLS chain is architecturally sound

**Gaps identified:**
1. **REST API rate limit calculation for concurrent AI scenarios is absent.** The document states Salesforce API throttling is mitigated by Bulk API + exponential backoff, but no API consumption estimate per business case per day is provided. Stuart would want a table showing estimated daily API calls per case to validate the "API consumption is dramatically reduced" claim.
2. **Seismic DAM API rate limits are not addressed.** If 30 RMs simultaneously trigger cross-sell co-pilot actions, Seismic API calls increase proportionally. No rate limit boundary or circuit-breaker is defined for the Seismic API layer.

**Stuart Subtotal: (8.6 + 8.3 + 8.1) / 3 = 8.33**

---

## Evaluator C: Independent Technical Reviewer
### Architecture SME — Weight: 30%

#### Dimension 1: AWS Service Correctness (0–10)
**Score: 8.7**

**Strengths:**
- Bedrock Claude 3.5 Sonnet is correctly named as the primary LLM for generative tasks; SageMaker JumpStart is correctly used for fine-tuned domain-specific tasks — the routing logic between them is architecturally justified
- SageMaker GIPS Engine separation (deterministic calculations only; Bedrock monitors compliance) is architecturally correct and demonstrates understanding of LLM limitations for regulated calculations
- Amazon Textract DocumentAnalysis with TABLES + FORMS feature types is correctly specified for complex DDQ PDF extraction

**Gaps identified:**
1. **Snowflake Cortex AI model specification is incomplete.** Section 3C uses `llama3-70b` via `COMPLETE()` but does not specify whether this is Snowflake Cortex COMPLETE (native) or an external Bedrock model via Snowflake External Function. The distinction matters for data egress, cost, and latency — Cortex COMPLETE keeps data in Snowflake, External Function routes to Bedrock. The architecture diagram should clarify.
2. **MSK Kafka configuration for the CQRS write path references parameters (acks=all, replication_factor=3) from the CQRS source document but the AI strategy document does not discuss how AI write-back events (Next Best Actions, Case updates) are handled.** Are AI outputs published to MSK Kafka topics and consumed by Snowpipe into a separate Gold table? Or do they write directly to Salesforce via API? The write-path architecture for AI outputs is ambiguous.

#### Dimension 2: Security Architecture (0–10)
**Score: 8.8**

**Strengths:**
- IAM identity mesh for NAPCE (Bedrock Agent assumes least-privilege IAM role per invocation, scoped to requesting RM's entitlements) is correctly specified
- Zero-Trust chain (Cognito OIDC → Lambda → Snowflake Horizon RLS + Dynamic Data Masking) is correct and consistent with the CQRS foundation document
- Defense-in-depth for PII: Comprehend masking at ingestion + Bedrock Guardrails PII redaction at output is double-layered correctly

**Gaps identified:**
1. **IAM role per-case isolation is not diagrammed.** All five cases use `NOMURA_AI_WH` but the IAM roles for each are not specified. Does the cross-sell JumpStart endpoint use a different IAM role than the NAPCE DDQ Agent? Role boundary isolation documentation is absent.
2. **S3 Object Lock configuration details for NAPCE audit trail are missing.** The document states "immutable audit log (CloudTrail-compliant)" but does not specify Object Lock retention period or compliance mode (GOVERNANCE vs. COMPLIANCE mode — the latter cannot be overridden by root account). For a GIPS audit trail, COMPLIANCE mode is likely required.

#### Dimension 3: FinOps (0–10)
**Score: 8.5**

**Strengths:**
- Per-case FinOps estimates are provided where available; combined total (<$7K/month) is credible given the serverless-first architecture
- NAPCE break-even framing ("pays for itself on first institutional mandate assisted") is compelling and quantitatively defensible
- SageMaker JumpStart auto-scale to zero during off-hours is correctly specified for cost optimization

**Gaps identified:**
1. **Snowflake Cortex AI cost for personalization (Case A3) is not estimated.** COMPLETE() calls on Snowflake are billed per token; at 10,000 clients × quarterly reports, the token volume is significant. This is the largest cost unknown in the FinOps section.
2. **NAIM OpenSearch Serverless OCU scaling is underspecified.** 2 OCUs minimum is stated but the peak concurrent query estimate (how many agents simultaneously using NAIM during morning support shift?) is not provided. OCU auto-scaling could significantly exceed the $350/month baseline estimate.

**Technical Reviewer Subtotal: (8.7 + 8.8 + 8.5) / 3 = 8.67**

---

## Cycle 1 Composite Score

| Evaluator | Raw Score | Weight | Weighted Score |
|---|---|---|---|
| Kathleen Hess McNamara | 8.50 | 35% | 2.975 |
| Stuart Mumley | 8.33 | 35% | 2.916 |
| Independent Technical | 8.67 | 30% | 2.601 |
| **CYCLE 1 TOTAL** | | **100%** | **8.49 / 10** |

---

## Cycle 1 Gap Register — Priority List for Cycle 2

The following gaps must be resolved in the main document before Cycle 2 evaluation:

| Priority | Gap | Section | Evaluator |
|---|---|---|---|
| P1 | Add quantified life-event trigger taxonomy for Case A3 (retirement triggers, liquidity events) | Section 3C | Kathleen |
| P1 | Add consolidated Total AI Investment vs. AUM Return table in Section 1 or Section 8 | Section 8 | Kathleen |
| P1 | Seismic DAM API integration pattern clarification (link vs. file attachment) | Section 3A | Stuart |
| P1 | Snowflake Cortex AI model specification (Cortex COMPLETE vs. External Function) | Section 3C | Technical |
| P2 | HITL gate timeout / expiry policy for unapproved AI recommendations | Section 6 | Kathleen |
| P2 | Confidence threshold calibration rationale (0.85 vs. 0.75 vs. 0.70) | Section 7 | Kathleen |
| P2 | NPS/CSAT/CES 50% target disaggregated by contributing case | Section 8 | Kathleen |
| P2 | Salesforce write-path object specification for Case A3 (which SF object, fields, triggers) | Section 3C | Stuart |
| P2 | MWAA max_active_runs and auto-scaling configuration per DAG type | Section 4 | Stuart |
| P2 | Inter-DAG dependency governance (gold_refresh → napce_rfp_proposal blocking) | Section 4 | Stuart |
| P3 | Seismic API rate limit boundary and circuit-breaker | Section 3A | Stuart |
| P3 | S3 Object Lock retention period and COMPLIANCE vs. GOVERNANCE mode for NAPCE | Section 4 | Technical |
| P3 | Snowflake Cortex AI cost estimate for personalization at 10K client scale | Section 8 | Technical |
| P3 | NAIM OpenSearch Serverless peak OCU demand estimate | Section 3B | Technical |
| P3 | Case A3 AUM retention / wallet share quantification | Section 3C | Kathleen |

---

## Cycle 1 Verdict

**Score: 8.49 / 10**

The document demonstrates strong architectural depth across all five cases, correct application of the BIAN taxonomy, and well-specified risk mitigation. The primary gaps are in specificity of integration boundaries, quantification of Case A3's AUM impact, and FinOps completeness for two cases. These are resolvable gaps — the architecture is sound; the documentation needs additional precision.

**Proceed to Cycle 2 with P1 and P2 gaps resolved.**

---

*Evaluation prepared as part of the Customer-Centric AI Strategy self-reinforcement training loop.*
*Target final score: > 9.8 / 10 by Cycle 3.*
