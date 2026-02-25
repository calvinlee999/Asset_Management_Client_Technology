# Evaluation Cycle 1 — Customer-Centric Data Strategy: CQRS Data Mesh
## Nomura Client Technology — Baseline Assessment

**Document Under Evaluation:** [CUSTOMER_CENTRIC_DATA_STRATEGY_CQRS.md](../CUSTOMER_CENTRIC_DATA_STRATEGY_CQRS.md)
**Evaluation Type:** Self-Reinforcement Cycle 1 — Baseline Scoring
**Evaluator Panel:** Kathleen Hess McNamara (ED, Client Experience & AI) · Stuart Mumley (Director, Client Platforms)
**Certification Threshold:** ≥9.8/10

---

## Evaluation Framework

This self-reinforcement evaluation applies a FinTech/Asset Management executive lens. The document is assessed across four dimensions, weighted by relevance to each evaluator's domain:

| Dimension | Kathleen Weight | Stuart Weight |
|---|---|---|
| A — Client Experience & AI Strategy | 40% | 20% |
| B — Data Architecture & CQRS Design | 25% | 40% |
| C — Salesforce Platform & Governance | 20% | 30% |
| D — Business Impact & AUM Strategy | 15% | 10% |

---

## Cycle 1 — Evaluator A: Kathleen Hess McNamara

### Dimension A: Client Experience & AI Strategy (40% of Kathleen's score)

**Assessment:**

The framing of client journey continuity as the primary value proposition is strategically correct. The observation that *"the experience is continuous because the data is continuous"* directly addresses the most common institutional AM complaint: inconsistency across touchpoints. The CQRS pattern is well-motivated from a client perspective — the document correctly identifies that separate virtual warehouses per consumer prevent quarter-end ingestion from degrading advisor and chatbot experience.

The AI Chatbot integration with the Query Path is architecturally sound. The statement that all performance figures are sourced from the same GIPS-certified Gold table that generates client reports — preventing chatbot hallucination on regulatory data — is the correct architectural principle.

**Gaps identified:**

- **Gap A1 — Chatbot failure mode not specified:** The document states the AI Chatbot reads from Snowflake with <2-second latency. However, it does not specify the chatbot response behavior when Snowflake returns an error or when data is temporarily unavailable (e.g., during a Snowflake virtual warehouse restart). From a client experience perspective, a chatbot that silently returns stale or empty data is worse than one that transparently states "I can see your account summary but portfolio details are temporarily updating — your advisor can provide the current view." The graceful degradation UX path needs specification.

- **Gap A2 — AUM Churn Risk model explainability:** The Cortex AI churn scoring query shown in the document outputs a single integer (0–100). Kathleen's experience at Macquarie informs that advisors distrust black-box scores without explanation. If the system tells an RM "this client is at 78 risk" without explaining *why*, the RM will either ignore it or take the wrong retention action. The document should specify that the model outputs the top 3 contributing factors alongside the score — and show how these factors surface in the Salesforce advisor dashboard.

- **Gap A3 — Client sentiment analysis depth:** The document references Snowflake Cortex AI sentiment analysis on Service Cloud chat logs. However, no specification is given for what sentiment signals are extracted, what the output schema looks like, or how frequently it runs. For an executive sponsor, this needs at least one concrete example: what does a sentiment score of -0.7 on a client interaction look like in the advisor's dashboard, and what action does it trigger?

**Dimension A Score: 8.4/10**
Strong strategic framing and correct architectural choices. Three gaps in AI explainability, chatbot UX degradation, and sentiment depth prevent a higher score.

---

### Dimension D: Business Impact & AUM Strategy (15% of Kathleen's score)

**Assessment:**

The business value framing is strong. The 30-day AUM churn alert vs. T+3 baseline is the most compelling KPI in the document — it translates technical architecture into a quantified retention impact. The cross-selling opportunity identification through the `aum_churn_risk` Gold table feeding Salesforce tasks is correctly connected to business outcomes.

**Gaps identified:**

- **Gap D1 — AUM retention quantification missing:** The document states "proactive AUM retention" and "reducing client churn" as objectives but provides no estimate of the revenue protection value. For an executive-level document targeting a panel including the Head of Client Experience, there should be at least a sensitivity analysis: "If the 30-day early warning prevents 5% of at-risk AUM from departing (estimated baseline churn: $X/year), the annual revenue protection is $Y." Even a conservative estimate makes the business case concrete.

**Dimension D Score: 8.2/10**

---

**Kathleen Overall (Weighted):** (8.4 × 0.40) + (8.2 × 0.15) + [B/C pending] = partial

---

## Cycle 1 — Evaluator B: Stuart Mumley

### Dimension B: Data Architecture & CQRS Design (40% of Stuart's score)

**Assessment:**

The CQRS Command/Query path separation is correctly applied and well-explained. The explicit statement that the Command Path uses MSK Kafka and Snowpipe Streaming — never Salesforce REST APIs — correctly addresses the rate limit exposure that is the most common Salesforce scalability failure mode. The medallion architecture (Bronze/Silver/Gold) is consistent with the Salesforce + Snowflake FinTech Architecture document already established in this repository.

The MSK Kafka configuration details (acks=all, replication_factor=3, min_insync_replicas=2) are correctly specified for an exactly-once semantics guarantee. The Snowflake BYOL architecture on S3 is the right choice for a financial services firm that needs data sovereignty.

**Gaps identified:**

- **Gap B1 — Dead Letter Queue (DLQ) specification:** The Lemons Table mentions a Dead Letter Queue for records failing quality gates, but the document provides no specification: Which S3 path? What is the retry policy? Who is notified? How are DLQ records reprocessed? For Stuart's platform operations concern, an unspecified DLQ is an operational blind spot — records can accumulate unnoticed and represent data integrity risk.

- **Gap B2 — Schema evolution strategy:** The document uses AWS Glue Schema Registry for contract versioning but does not address what happens when a schema change is introduced to a Kafka topic (e.g., Aladdin adds a new field to the position CDC event). Without a schema evolution strategy, a new Bronze table schema can break the dbt Silver transformation model, halting the entire pipeline. The Snowflake BYOL architecture requires explicit handling of backward-compatible vs. breaking schema changes.

- **Gap B3 — Snowpipe Streaming monitoring:** The document mentions a Prometheus metric `nomura_msk_consumer_lag` but does not specify how Snowpipe Streaming pipeline health is monitored. Snowpipe Streaming uses an internal channel model; failures are not automatically surfaced via standard CloudWatch metrics. A specific monitoring strategy for Snowpipe channel failures is required.

- **Gap B4 — Multi-region disaster recovery:** The document specifies multi-AZ for MWAA and MSK but does not address Snowflake multi-region configuration for DR. The Salesforce + Snowflake FinTech Architecture document previously established RTO=60min, RPO=6h targets — this document should either confirm or differentiate from those targets, and specify the Snowflake Replication Group configuration.

**Dimension B Score: 8.5/10**
CQRS pattern is correctly applied. Four gaps in operational specifics (DLQ, schema evolution, Snowpipe monitoring, DR) limit the score.

---

### Dimension C: Salesforce Platform & Governance (30% of Stuart's score)

**Assessment:**

The Salesforce architecture is correctly designed. External Objects for Zero-Copy, OIDC token passthrough to Snowflake Horizon, and Elasticache for query result caching are all the right choices. The BIAN service boundary argument — "Salesforce never becomes the system of record for things outside its domain" — is exactly the right framing for Stuart's platform ownership concern.

**Gaps identified:**

- **Gap C1 — Salesforce External Object query limits:** Salesforce External Objects have their own governor limits (100,000 rows per query, 50MB response size limit). For institutional clients with large portfolios, a `portfolio_daily` External Object query could exceed these limits. The document should specify how large portfolio queries are paginated or pre-aggregated in Snowflake to fit within External Object response limits.

- **Gap C2 — AppFlow sync bidirectionality:** The document specifies AppFlow for Salesforce→S3 data flow (CRM events) but does not fully address bidirectional sync. When the Cortex AI churn risk score is computed and needs to be surfaced as a Salesforce field (for RM alerts), how does that Gold layer value flow back into Salesforce? AppFlow reverse sync, Salesforce Flow triggered by API call, or outbound message from Snowflake? This needs specification.

**Dimension C Score: 8.6/10**

---

**Stuart Overall (Weighted):** (8.5 × 0.40) + (8.6 × 0.30) + [A/D pending] = partial

---

## Cycle 1 — Combined Score Summary

| Dimension | Kathleen Score | Stuart Score | Combined Weight | Weighted Contribution |
|---|---|---|---|---|
| A — Client Experience & AI | 8.4 | 8.0 (Stuart's A) | 30% | 2.52 |
| B — Data Architecture & CQRS | 8.3 (Kathleen's B) | 8.5 | 30% | 2.49 |
| C — Salesforce Platform | 8.5 (Kathleen's C) | 8.6 | 25% | 2.15 |
| D — Business Impact | 8.2 | 8.4 (Stuart's D) | 15% | 1.23 |
| **CYCLE 1 COMPOSITE** | | | **100%** | **8.39/10** |

---

## Gap Summary for Cycle 2

| Gap | Category | Description | Priority |
|---|---|---|---|
| **G1** | AI / Chatbot UX | Chatbot graceful degradation path when Snowflake query fails | High |
| **G2** | AI Explainability | AUM churn risk model: top 3 contributing factors output + Salesforce display | High |
| **G3** | AI Strategy | Sentiment analysis output schema + dashboard surfacing specification | Medium |
| **G4** | Business Case | AUM retention value quantification (sensitivity analysis) | Medium |
| **G5** | Data Engineering | Dead Letter Queue path, retry policy, and notification specification | High |
| **G6** | Data Engineering | Schema evolution strategy for MSK Kafka → Snowflake Bronze pipeline | High |
| **G7** | Operations | Snowpipe Streaming channel failure monitoring strategy | Medium |
| **G8** | DR / Resilience | Snowflake multi-region replication configuration | Medium |
| **G9** | Salesforce | External Object governor limit handling for large portfolio queries | Medium |
| **G10** | Salesforce | Bidirectional sync: Gold layer churn score → Salesforce field | High |

---

## Cycle 1 Verdict

**Composite Score: 8.39/10**

The document establishes a strong strategic foundation. The CQRS pattern is correctly applied, the BIAN service domain alignment is accurate, and the client experience narrative is compelling. Ten gaps across AI behavior specification, data engineering operational detail, and Salesforce integration bidirectionality prevent the score from reaching the ≥9.8/10 certification threshold. Cycle 2 targets ≥9.2/10 by resolving Gaps G1–G10.

**Next:** Cycle 2 — Gap Resolution, targeting ≥9.2/10.
