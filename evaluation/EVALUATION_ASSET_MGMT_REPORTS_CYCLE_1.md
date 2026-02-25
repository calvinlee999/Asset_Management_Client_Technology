# Evaluation Cycle 1 — Asset Management Reports & Use Cases
## Self-Reinforcement Training: FinTech/Asset Management Executive Review
### Document Under Review: [ASSET_MANAGEMENT_REPORTS.md](../ASSET_MANAGEMENT_REPORTS.md)

**Evaluation Date:** February 2026
**Cycle:** 1 of 3
**Evaluator Panel:** Kathleen Hess McNamara (35%) | Stuart Mumley (35%) | Technical Architecture Board (30%)
**Cycle 1 Score: 8.57 / 10.00**

---

## Evaluator 1 — Kathleen Hess McNamara
### ED, Client Experience & AI | Former Macquarie AI Digital Agent Lead
**Weight: 35% | Raw Score: 8.60 / 10**

### Strengths Identified

**S1 — AI Governance Architecture is Directionally Correct**
The "LLM for language, deterministic SQL/Python for numbers" principle is exactly the right foundation. The Bedrock Guardrails groundedness ≥ 0.95 threshold is appropriate. The document correctly identifies that this must be infrastructure-enforced, not policy-enforced. This is the same discipline I applied at Macquarie — and the only one that survives a Series 7/63 accountability test.

**S2 — Seismic Governance Framing is Production-Ready**
The compliance expiry workflow (Draft → Compliance Review → Approved → Expiry + self-deactivation) reflects how this actually works in a regulated distribution environment. The point about Seismic becoming "ambient" through Agentforce surfacing — content appearing when needed without RM action — is the correct end state. Well framed.

**S3 — Client Reporting Modernization Data Lineage is Excellent**
The explicit Aladdin → MSK Kafka → Snowpipe → Bronze → Silver → Gold → Vermilion/Power BI → Portal chain with stated SLAs is precisely the level of rigor a client experience leader needs to trust that reporting will not regress. The T+2h end-of-month SLA is a real commitment.

**S4 — Panel Q&A is Substantive**
My three panel questions are answered with the depth I would expect from a Head of Client Technology. The Vermilion/Power BI long-term architecture answer — "we never consolidate the regulated composite engine and the analytics engine onto the same platform" — demonstrates genuine product and regulatory judgment.

### Gaps Identified

**P1-K-01 — NPS/CES Driver Analysis methodology is undefined [CRITICAL]**
The document commits to a "50% NPS/CES improvement" but does not specify: (a) the baseline measurement methodology, (b) the survey cadence (transactional vs. relational NPS), (c) the driver analysis model (key driver analysis regression, CHAID tree, or MaxDiff?), or (d) how NPS responses are captured in the technology stack (Salesforce survey, Qualtrics, Medallia?). Without a measurement methodology, the KPI is unenforceable. Fix before Cycle 2.

**P1-K-02 — Proactive Covenant Alert mechanism is described but not architecturally specified [CRITICAL]**
Use Case 6 mentions "life event alerts" and "proactive covenant alerts" — but the covenant monitoring architecture is not specified anywhere. What data source detects a covenant breach event? Which system triggers the RM alert? What is the alert latency SLA? For institutional clients, a missed covenant alert is a relationship-ending event. Needs concrete specs.

**P2-K-03 — Wealth portal mobile experience governance is missing [MAJOR]**
The Wealth Self-Service Portal (UC6) references React Native + AppSync but does not specify: (a) accessibility standards (WCAG 2.1 AA for institutional clients), (b) biometric authentication fallback if Face ID fails, (c) offline mode capability for advisors travelling internationally, (d) jurisdiction-specific content filtering on mobile vs. web. These are client experience details, not technical afterthoughts.

**P2-K-04 — AI pre-meeting brief content sources are not enumerated [MAJOR]**
UC2 and UC7 mention "AI pre-meeting brief" from Agentforce + Bedrock. What specific data sources does the brief pull from? Is it: last 3 CRM meeting notes, AUM delta in last 90 days, open Salesforce opportunities, Seismic content sent + opened, NCIE sentiment signals, recent fund performance vs. benchmark? The RMs will ask exactly this question. The answer must be explicit.

**P3-K-05 — GIPS composite ownership model is not described [MINOR]**
GIPS composites have strict inclusion criteria. Who owns the composite definition? Who approves changes to composite inclusion criteria in the platform? A governance owner must be assigned in the document.

---

## Evaluator 2 — Stuart Mumley
### Director, Client Platforms | Former Macquarie Salesforce FSC Lead
**Weight: 35% | Raw Score: 8.55 / 10**

### Strengths Identified

**S1 — Salesforce Platform Boundary Discipline is Correct**
The "Salesforce is the system of engagement, not record" principle is enforced consistently throughout the document. The zero-copy integration pattern (Salesforce reads Snowflake Gold via integration layer, never duplicates data) is the right architecture. The ADR governance model for platform boundary is exactly how I would want this managed.

**S2 — CRM Engagement Scorecard is Business-Ready**
The 5 metrics are the right ones: meeting coverage, CRM update compliance, content engagement rate, follow-up action close rate, NPS correlation. These are the metrics I would present to the Head of Distribution in a quarterly business review. The 90% CRM update compliance target via Agentforce one-tap post-meeting automation is realistic.

**S3 — Distribution Enablement Roadmap Sequencing is Sound**
Phase 2 correctly prioritises distribution enablement (Seismic + CRM + pipeline visibility) before client experience transformation (Phase 3). This is the right sequencing: internal RM tools first, external client portal second. Revenue-generating capabilities before cost-reduction ones.

**S4 — CRM Update Compliance Answer is Operationally Honest**
The panel Q&A answer about the 10% who still don't update CRM — using manager notification as behavioural pressure rather than blame, combined with making the automated record approval easier than manual entry — is operationally honest. I've managed this exact problem. That answer works.

### Gaps Identified

**P1-S-01 — Salesforce FSC data model for mandate pipeline is not specified [CRITICAL]**
UC1 references "Salesforce FSC Opportunity + custom `Mandate_Stage__c` field" but the full FSC data model for mandate management is not defined. Which FSC standard objects are used (Financial Account, Opportunity, Account Branch)? How is the institutional mandate hierarchy (Prospect → Account → Multiple Mandates) modelled in FSC? How does the `Mandate_Stage__c` picklist align to the BIAN Customer & Party Agreement service domain stages? This is a platform architecture decision that requires explicit specification.

**P1-S-02 — Seismic ↔ Salesforce integration pattern is underspecified [CRITICAL]**
The document references "Seismic LiveSend → email delivery with engagement analytics → written to `Content_Engagement__c`" but does not specify (a) whether this uses Seismic's native Salesforce connector or a custom API integration, (b) the data mapping from Seismic engagement events to Salesforce activity records, (c) who owns the Seismic API key governance, or (d) the sync frequency (real-time webhook vs. scheduled job). For a platform of this compliance sensitivity, the integration pattern must be explicit.

**P2-S-03 — Salesforce Experience Cloud persona design is not specified [MAJOR]**
UC4 mentions Salesforce Experience Cloud for the client portal but does not specify the Experience Cloud template (Build Your Own vs. Customer Service vs. Help Centre template), the Lightning Web Component architecture for report rendering, or how Cognito JWT claims map to Salesforce permission sets. These determine whether the portal can be built within the existing FSC licence or requires additional Experience Cloud add-ons.

**P2-S-04 — No Salesforce release management process defined [MAJOR]**
The document proposes significant Salesforce customization (custom fields, Agentforce actions, Experience Cloud pages, LWC components, Platform Events). There is no mention of: sandbox strategy (Full → Partial → Dev sandboxes), CI/CD pipeline for Salesforce metadata (Salesforce CLI + GitHub Actions), change set vs. scratch org model, or regression testing approach for Agentforce flow changes. For a 14-person team managing a regulated platform, this is critical.

**P3-S-05 — No Salesforce licence cost model included [MINOR]**
The document adds Experience Cloud (external portal), Marketing Cloud journey (welcome email), additional FSC custom objects, and Agentforce actions. No licence cost impact assessment is provided. For a $22M budget envelope, every additional Salesforce SKU requires explicit business case justification.

---

## Evaluator 3 — Technical Architecture Board
### Cloud Architecture & Security Review | Nomura Technology Governance
**Weight: 30% | Raw Score: 8.55 / 10**

### Strengths Identified

**S1 — CQRS Query Path Architecture is Correctly Abstracted**
The four-layer data product stack (Event Backbone → CQRS Query Path → Semantic / Business Logic → Intelligence Surface) is the right way to describe the architecture to a business and technical audience simultaneously. The reference back to the certified CQRS document is appropriate — this document correctly treats it as a dependency, not a duplicate.

**S2 — Saga Pattern Reference for Onboarding is Correct**
The MWAA Saga compensating transaction pattern for onboarding (UC3) is the correct approach for a multi-step distributed workflow where partial failures must trigger clean rollbacks. The reference to the NAPCE Saga implementation is appropriate — consistent architecture across platforms.

**S3 — S3 COMPLIANCE-mode WORM Archival is Complete**
The 7-year WORM retention specification for GIPS calculation inputs/outputs meets GIPS Standards s.4.A.1 data retention requirements and SEC Rule 17a-4 electronic records requirements. The specification is correct and complete.

### Gaps Identified

**P1-T-01 — Power BI DirectQuery latency under concurrent load is not benchmarked [CRITICAL]**
The document asserts "< 500ms p95 Power BI DirectQuery on Snowflake Gold" but provides no benchmarking methodology or load test specification. At peak quarterly reporting (300+ institutional clients accessing portfolios simultaneously), Snowflake compute cluster sizing must be validated. The document must specify: Snowflake warehouse size for DirectQuery workload, query concurrency limits, and load test methodology.

**P1-T-02 — Cognito JWT claim validation implementation is not specified [CRITICAL]**
The document states "every surface validates JWT claims encoding role, permitted composites, client access list." But the JWT claim schema is not defined: What claim names? What format (comma-separated composite IDs vs. JSON array)? How are claim updates propagated when a client's entitlements change mid-session? What is the token expiry window for portal sessions? This is a security architecture specification gap.

**P2-T-03 — Disaster Recovery and RTO/RPO for reporting platform are absent [MAJOR]**
The document specifies operational SLAs (T+2h report delivery, < 500ms query) but provides no DR specifications: (a) RTO (Recovery Time Objective) for Snowflake Gold table unavailability, (b) RPO (Recovery Point Objective) for Kafka → Snowpipe streaming failure, (c) failover region for the client portal, (d) GIPS report cold-start time if Vermilion rendering API is unavailable. For a platform with regulatory reporting obligations, RTO/RPO are mandatory.

**P2-T-04 — Snowflake Horizon RLS policy specification is missing [MAJOR]**
The document mentions "Snowflake Horizon Row-Level Security enforces composite access" but the policy specification is absent: (a) what is the RLS policy expression on `performance_attribution_gold`?, (b) how does the Cognito JWT claim map to the Snowflake session variable used by the RLS policy?, (c) is Dynamic Data Masking applied to any columns in the Gold tables (e.g., masking client LEI for support analysts but not for compliance)?

**P3-T-05 — AWS Cost model for new use cases is not estimated [MINOR]**
UCs 3, 5, 6, and 9 add Cognito user pools, additional AppSync resolvers, Amazon Batch Monte Carlo compute, and Kinesis Firehose for portal analytics. No incremental AWS cost estimate is provided beyond the "$700K/year baseline" inherited from CQRS. An order-of-magnitude cost estimate is needed.

---

## Weighted Score Calculation

| Evaluator | Raw Score | Weight | Weighted Score |
|---|---|---|---|
| Kathleen Hess McNamara | 8.60 | 35% | 3.010 |
| Stuart Mumley | 8.55 | 35% | 2.993 |
| Technical Architecture Board | 8.55 | 30% | 2.565 |
| **CYCLE 1 TOTAL** | | **100%** | **8.57 / 10** |

---

## Gap Resolution Plan — Cycle 1 → Cycle 2

### Priority 1 Gaps (Must Resolve Before Cycle 2)

| Gap ID | Description | Resolution Action |
|---|---|---|
| P1-K-01 | NPS/CES measurement methodology undefined | Add NPS methodology section: transactional NPS (post-interaction) + relational NPS (quarterly), Salesforce survey capture, key driver regression model |
| P1-K-02 | Covenant alert architecture unspecified | Add covenant monitoring architecture: Snowflake `covenant_monitoring_gold` table, MWAA daily check, Agentforce push notification to RM within 15 minutes |
| P1-S-01 | Salesforce FSC mandate data model unspecified | Add FSC data model: Account (institutional entity) → Financial Account (mandate) → Opportunity (pipeline) with `Mandate_Stage__c` picklist aligned to BIAN stages |
| P1-S-02 | Seismic ↔ Salesforce integration pattern underspecified | Add integration pattern: Seismic native Salesforce connector + webhook for real-time engagement events → `Content_Engagement__c` object mapping table |
| P1-T-01 | Power BI DirectQuery load benchmark absent | Add Snowflake warehouse sizing spec: XL warehouse for DirectQuery workload, 4-cluster auto-scaling, tested to 500 concurrent users |
| P1-T-02 | Cognito JWT claim schema undefined | Add JWT claim schema: `composite_ids` (array), `client_ids` (array), `role` (enum), `channel` (enum), 1-hour token expiry, Cognito Lambda trigger for mid-session entitlement refresh |

### Priority 2 Gaps (Resolve in Cycle 2, Verify in Cycle 3)

| Gap ID | Description | Resolution Action |
|---|---|---|
| P2-K-03 | Wealth portal mobile governance missing | Add mobile governance section: WCAG 2.1 AA, biometric fallback PIN, offline position snapshot (last 24h), jurisdiction content filter |
| P2-K-04 | AI pre-meeting brief sources not enumerated | Add explicit data source table for Agentforce brief: 6 named sources with CRM/Snowflake/Seismic lineage |
| P2-S-03 | Experience Cloud persona design absent | Add EC template selection (Build Your Own LWC) + permission set matrix |
| P2-S-04 | No Salesforce release management process | Add SF DevOps section: scratch org model, GitHub Actions CI/CD, deployment approval gate |
| P2-T-03 | DR/RTO/RPO absent | Add DR specification: RTO < 4h (portal), RPO < 15min (Kafka), multi-AZ Snowflake, Vermilion failover |
| P2-T-04 | Snowflake Horizon RLS policy missing | Add RLS policy expression for `performance_attribution_gold` + Dynamic Data Masking spec |

### Priority 3 Gaps (Minor — Address in Cycle 2)

| Gap ID | Description | Resolution Action |
|---|---|---|
| P3-K-05 | GIPS composite ownership governance | Add GIPS composite ownership table: Composite ID → Business Owner → Technical Steward → Review Frequency |
| P3-S-05 | Salesforce licence cost model missing | Add Salesforce licence impact table: EC add-on, Agentforce pricing, incremental FSC SKUs |
| P3-T-05 | AWS cost estimate for new UCs | Add incremental cost estimate table: Cognito, AppSync, Batch Monte Carlo, Kinesis Firehose |

---

## Cycle 1 Summary

**Cycle 1 Score: 8.57 / 10**

The document successfully establishes a production-anchored, executive-ready Asset Management Data-as-a-Product framework. The 10 use cases, 10 reports, and 4 strategic domains are coherent, business-grounded, and aligned to the Nomura technology ecosystem. The Lemons Table and Panel Q&A demonstrate genuine product and regulatory judgment.

The 15 identified gaps are concentrated in three categories: (1) measurement methodology specificity (NPS/CES), (2) Salesforce platform architecture detail (FSC data model, Seismic integration, Experience Cloud), and (3) infrastructure resilience specifications (DR/RTO/RPO, Cognito JWT schema, Snowflake RLS policy). None of these are architectural flaws — they are specification completeness gaps appropriate for a first cycle.

**Cycle 2 target: ≥ 9.30 / 10**, contingent on resolving all 6 Priority-1 gaps.

---

*Evaluated under Nomura Asset Management International Client Technology Platform governance standards. Self-reinforcement training simulating Kathleen Hess McNamara, Stuart Mumley, and the Technical Architecture Board review panel.*
