# Customer-Centric Data Strategy: CQRS Data Mesh
## Nomura Client Technology — AUM Retention & Growth Through Unified Data Architecture

**Prepared by:** Calvin Lee, Candidate — Executive Director, Head of Client Technology
**Target Audience:** Kathleen Hess McNamara (ED, Client Experience & AI) · Stuart Mumley (Director, Client Platforms)
**Date:** February 2026
**Framework Alignment:** BIAN Data & Integration (Domain 8) · BIAN Customer & Party Management (Domain 1) · BIAN Sales & Distribution (Domain 2)

---

## EXECUTIVE SUMMARY (Conclusion First)

To actively retain and grow Assets Under Management, Nomura Client Technology must solve one fundamental problem: **our clients are served by systems that do not share a common understanding of who they are.**

Salesforce holds relationship history. Aladdin holds portfolio positions. Snowflake holds analytics. The AI Chatbot queries whichever source was most recently updated. The result is a wealth advisor describing a client's portfolio to that same client using yesterday's numbers, while the client's mobile portal shows today's. That contradiction costs trust — and trust costs AUM.

The Customer-Centric Data Strategy presented here solves this with a **Command Query Responsibility Segregation (CQRS) Data Mesh** built on three pillars:

1. **One Write Path (Command):** All mutations — trades, cash movements, CRM updates, compliance flags — flow through a single, high-velocity AWS ingestion layer (MSK Kafka + Snowpipe Streaming + S3) into a centralized Snowflake Data Cloud. No system writes to two places. No reconciliation required.

2. **One Read Path (Query):** Salesforce Sales Cloud, Salesforce Service Cloud, and Nomura's AI Chatbots read from a single Gold layer via Zero-Copy integration. There are no duplicated copies of client data. The advisor sees the same number the client sees, because they are both reading from the same record.

3. **Universal Governance:** Snowflake Horizon enforces Dynamic Data Masking and Row-Level Security across every consumption surface — whether the query originates from Salesforce, the AI Chatbot, or an AWS API. The same OIDC identity token governs what any user or system is permitted to see.

**Strategic outcome:** A wealth advisor in New York, an institutional RM in London, and a client using the AI Chatbot at 7am are all informed by the same real-time, governance-enforced truth. That is the client experience that retains AUM.

---

## 1. STRATEGIC CONTEXT

### 1.1 The Problem: Siloed Systems Create Broken Client Moments

Current architecture has Salesforce, Aladdin, and the Data Platform operating as parallel systems of record that are periodically synchronized rather than continuously aligned. The consequences are:

| Client Moment | Current State | Impact |
|---|---|---|
| Advisor RM meeting | CRM data 24h out of sync with Aladdin positions | Advisor reads wrong portfolio value to client |
| AI Chatbot query | Returns data from whichever system was last polled | Client sees different figures from chatbot vs. portal |
| Institutional DDQ | Technology answers sourced manually from multiple systems | 5+ business days per response; error risk from stale data |
| Quarterly report delivery | Performance data assembled from three sources | Reconciliation breaks delay delivery; manual fixes introduce risk |
| Client off-boarding signal | AUM drawdown detected in Aladdin; not visible in CRM until T+3 | Retention intervention happens 3 days too late |

### 1.2 The CQRS Solution

**Command Query Responsibility Segregation** (CQRS) is an architectural pattern that separates:
- The **Command Path** — writes, updates, and mutations, optimized for throughput and consistency
- The **Query Path** — reads, analytics, and AI inference, optimized for latency and availability

Applied to Nomura's architecture:

```
COMMAND PATH (High Velocity — AWS-Owned)
  Aladdin CDC events →  MSK Kafka   →  Snowpipe Streaming →  Snowflake Bronze
  Trading executions →  MSK Kafka   →  Snowpipe Streaming →  Snowflake Bronze
  Cash movements     →  MSK Kafka   →  Snowpipe Streaming →  Snowflake Bronze
  Salesforce events  →  AppFlow     →  AWS Glue ETL       →  Snowflake Bronze
  Market data        →  ADX Zero-Copy →                      Snowflake Bronze
  Client documents   →  S3 + Textract → MWAA HITL DAG    →  Snowflake Catalog

         ↓ (AWS Glue ETL / dbt / Snowflake Cortex AI)

CENTRALIZED SNOWFLAKE DATA CLOUD (Single Source of Truth)
  Bronze → Silver → Gold (Medallion Architecture)
  Gold: customer_360 | portfolio_daily | risk_metrics | aum_churn_risk | gips_composites

         ↓ (Zero-Copy Query — no data movement)

QUERY PATH (Low Latency — Read-Optimized)
  Salesforce Sales Cloud   → External Objects → Real-time Customer 360
  Salesforce Service Cloud → External Objects → Advisor dashboard <500ms
  Nomura AI Chatbot        → Snowflake Virtual Warehouse → Context-aware responses
  Client Portal            → Direct API → Current portfolio view
  Institutional DDQ Agent  → OpenSearch + Snowflake → GIPS-verified automated responses
```

### 1.3 BIAN Domain Alignment

| BIAN Service Domain | CQRS Role | Platform Implementation |
|---|---|---|
| **Data & Integration (Domain 8)** | Command Path foundation — Collect, Maintain, Consolidate | MSK Kafka + S3 Medallion + Snowflake + AWS Glue |
| **Customer & Party Management (Domain 1)** | Query Path primary consumer — unified client profile | Salesforce External Objects on Snowflake Gold layer |
| **Sales & Distribution (Domain 2)** | Query + Command — pipeline signals feed back to Command | Salesforce Sales Cloud + Snowpipe write-back + AI Chatbot |
| **Client Reporting & Performance (Domain 6)** | Query Path — GIPS and Vermilion read from same Gold record | Performance Engine → Vermilion → Portal — one data chain |
| **Sales Enablement (Domain 7)** | Query Path — Seismic personalizes from Gold layer | AUM churn risk score surfaces in advisor meeting prep |

### 1.4 The Three Layers of AI Maturity (Strategic Framing)

This framework governs how AI capability is sequenced across the Nomura Client Technology platform. Each layer builds on the governance foundation of the one before it.

| Layer | Description | Nomura Implementation | Governance Requirement |
|---|---|---|---|
| **Layer 1 — Internal Productivity** | AI helps internal teams work faster without replacing human judgment | NAPCE DDQ/RFP agent: drafts regulatory responses in hours instead of 3–6 weeks | Content must be auditable; HITL mandatory before client submission |
| **Layer 2 — Assisted Decision Support** | AI provides ranked recommendations; humans retain final decision authority | Pre-meeting client insights for RMs; NAIM Support Co-Pilot troubleshooting suggestions | Recommendation traceability required; human approval gates enforced |
| **Layer 3 — Client-Facing Automation** | AI interacts directly with clients or counterparties without intermediate human review | Self-service portal AI Q&A; real-time chatbot responses on portfolio performance | Highest governance bar: Bedrock Guardrails + RLS + immutable audit trail required at all times |

**Current State at Nomura:** Layer 1 (Internal Productivity) is the deployment target for NAPCE DDQ/RFP. Layer 2 (Assisted Decision Support) is the deployment target for NAIM, A1 cross-sell, and A3 personalization. Layer 3 requires the 5 Pillars of Content Governance (Section 5.3) to be fully operational before activation — it must never be enabled on the governance foundation of Layer 1 alone.

**Critical principle:** Layer 3 is only viable when Layer 1 and Layer 2 governance infrastructure is fully operational. A client-facing AI agent that cannot guarantee content currency, ownership accountability, and audit trail integrity is a regulatory liability, not a product feature.

---

## 2. PHASE 1 — DATA DISCOVERY PROCESS

Before building a single pipeline, we execute the Data Discovery framework to ensure technology investment is precisely aligned with Nomura's AUM retention and growth objectives.

### 2.1 Define Business Value

**Kick-off Workshop Agenda:**
The first step is a structured workshop with Kathleen (client experience, AI, reporting) and Stuart (Salesforce, platform roadmap, sales enablement) to answer the critical questions:

> *"How would getting single-version, real-time insight into client data provide value to Nomura's business? What retention actions are currently delayed because the data is not available fast enough?"*

**For Nomura, the business value is threefold:**

| Business Value | Data-Driven Mechanism | AUM Impact |
|---|---|---|
| **Proactive AUM retention** | Predictive churn risk score (Snowflake Cortex AI on portfolio + engagement data) surfaces clients at risk before they redeem | 30-day early warning vs. T+3 current |
| **Cross-selling opportunity identification** | Client 360 + product fit model in Gold layer → next-best-action in Salesforce | Identifies unserved wallet share in existing relationships |
| **Institutional mandate pipeline acceleration** | DDQ automation (60–80% faster) + GIPS-verified performance data in proposals | Shorter sales cycles → faster AUM inflows |

**Revenue stream question answered:** The CQRS Data Mesh enables Nomura to offer **Zero-Copy data products to institutional clients and partners via AWS Data Exchange (ADX)** — a governed, entitlement-controlled way to share curated analytics without sharing raw systems. This is a net-new revenue capability.

### 2.2 Identify Data Customers

| Consumer | Current Data Access | Real-Time Need | Peak Concurrency |
|---|---|---|---|
| **Wealth Advisors** (Sales Cloud) | Aladdin data synced T+1, CRM updated manually | Portfolio values current as of last Aladdin sync | 200 concurrent users at NY market open (09:30 ET) |
| **RMs** (Institutional channel) | Salesforce account data, manually compiled portfolio summaries | Client AUM, recent flows, pipeline stage | 50 concurrent, peak at quarter-end reviews |
| **AI Chatbot** (client-facing) | Queries live Salesforce + periodic Snowflake | Full client context in <2 seconds | 500 concurrent sessions at peak |
| **AI Support Co-Pilot** (internal) | Service Cloud case history + knowledge base | Case context + similar resolution recommendations | 100 concurrent support staff |
| **Compliance Officers** | Manual report pull from Aladdin + Snowflake | GIPS composite status, audit logs | 10 concurrent, batch-heavy |
| **Data Stewards** | AWS Glue Catalog + manual spreadsheets | Catalog metadata, lineage, quality scores | 5 concurrent |

**Key finding from discovery:** The AI Chatbot and compliance team currently operate from data that is 24–48h stale for portfolio queries. The CQRS Data Mesh reduces this to <2-second freshness for both.

### 2.3 Identify Data Sources & Types

| Source | Data Type | Format | Current Volume | Ingestion Mode |
|---|---|---|---|---|
| **Aladdin** (portfolio positions) | Structured | CDC events (JSON) | 10,000 events/sec | Streaming (MSK Kafka) |
| **Trading systems** | Structured | FIX protocol → JSON | 2,000 executions/sec | Streaming (MSK Kafka) |
| **Cash management** | Structured | SQL rows | 500 transactions/sec | Micro-batch (Snowpipe) |
| **Salesforce Sales/Service Cloud** | Semi-structured | JSON (CRM objects) | 100 events/sec | AppFlow + scheduled |
| **Bloomberg / FactSet** | Structured | Market feed | 50,000 quotes/sec | ADX Zero-Copy |
| **Client documents** (RFPs, DDQs, IMA) | Unstructured | PDF, Word, PPTX | 200 documents/week | S3 + Textract + MWAA |
| **Market commentary / ESG** | Unstructured | Text, PDF | 50 documents/day | S3 + NLP pipeline |
| **Portfolio manager commentary** | Semi-structured | JSON + text | 10 docs/day | S3 + MWAA HITL |

---

## 3. PHASE 2 — THE 6-STEP AWS & SNOWFLAKE CQRS DATA MESH ARCHITECTURE

### 3.1 Step 1: Ingest (The Command Path)

The Command Path is designed for one thing: accepting data of any type from any source with zero information loss, at maximum velocity, without creating downstream bottlenecks.

```
┌─────────────────────────────────────────────────────────────────────────┐
│  COMMAND PATH — HIGH VELOCITY INGESTION LAYER                           │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  STREAMING (Sub-second)                                                  │
│  ├── Aladdin CDC via Debezium → MSK Kafka (topic: aladdin.positions)    │
│  ├── Trading executions       → MSK Kafka (topic: trades.executions)    │
│  └── Cash movements           → MSK Kafka (topic: cash.movements)       │
│                          ↓                                               │
│                   MSK Kafka                                              │
│           (Fully Managed Apache Kafka)                                   │
│           2M events/sec capacity                                         │
│           Backpressure management                                        │
│           Exactly-once semantics (acks=all)                             │
│           Replication factor = 3 (cross-AZ)                             │
│                          ↓                                               │
│                  Snowpipe Streaming                                      │
│           1–2 second ingestion latency to Snowflake Bronze               │
│                                                                          │
│  MICRO-BATCH (Seconds)                                                   │
│  ├── Salesforce events        → AWS AppFlow → S3 → Glue ETL             │
│  └── Client portal activity   → AWS Firehose → S3 → Glue ETL            │
│                                                                          │
│  BATCH (Daily)                                                           │
│  ├── Bloomberg/FactSet pricing → ADX Zero-Copy → Snowflake Marketplace  │
│  ├── ESG scores (MSCI)         → ADX Zero-Copy → Snowflake Marketplace  │
│  └── OFAC sanctions update    → ADX Zero-Copy → Snowflake Marketplace  │
│                                                                          │
│  AI-ASSISTED INGESTION (Unstructured Documents)                         │
│  └── Client docs (PDF/Word)   → S3 → Textract → MWAA DAG →             │
│       AI metadata suggestion → Human-in-the-Loop (HITL) approval →     │
│       Snowflake Horizon catalog entry                                    │
│                                                                          │
│  ZERO-ETL (Direct Integration)                                           │
│  └── Amazon Aurora / DynamoDB → Snowflake (AWS Zero ETL)                │
│      No pipeline maintenance. No manual schema mapping.                  │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

**Why MSK over direct Salesforce API calls:**
Salesforce REST API limit: 5,000 calls/minute. MSK Kafka to Snowpipe Streaming: unlimited throughput, 1–2 second latency, automatic backpressure when Snowflake slows under load. The Command Path never touches Salesforce API rate limits — it writes to Kafka; Salesforce reads via the Query Path.

### 3.2 Step 2: Store (Centralized Snowflake Foundation — BYOL)

Snowflake acts as the central Data Mesh foundation using a **Bring Your Own Lake (BYOL)** architecture on AWS S3. This means Nomura's data remains within our AWS account boundary — Snowflake's compute is separated from storage, providing both cost efficiency and data sovereignty.

```
SNOWFLAKE BYOL — MEDALLION STORAGE LAYERS

BRONZE (Raw, Immutable — The Command Record)
  → Append-only tables — never updated, never deleted during retention window
  → Encrypted: AWS KMS Customer-Managed Key (CMK)
  → Retention: 7 years (SEC 17a-4 / FINRA Rule 4511)
  → Tables: aladdin_raw | trades_raw | balances_raw | crm_events_raw
  → S3 Glacier Deep Archive for data >3 years old (cost optimization)
  → Write throughput: 2M events/sec via Snowpipe Streaming

SILVER (Clean, Typed, Reconciled)
  → Deduplicated (Lambda idempotency keys prevent double-counting)
  → Schema-validated (AWS Glue Schema Registry enforces contract versions)
  → Format: Apache Iceberg (time-travel: query state as of any historical timestamp)
  → Tables: aladdin_deduplicated | trades_typed | balances_enriched
  → Average transformation latency: <5 seconds

GOLD (Query-Optimized, Client-Ready)
  → Materialized aggregations optimized for <500ms p95 query latency
  → Dynamic Data Masking + Row-Level Security enforced at column level
  → Tables consumed directly by all Query Path consumers:
     customer_360       — unified client profile (Salesforce External Object)
     portfolio_daily    — portfolio summary by asset class
     risk_metrics       — Greeks, VaR, stress scenarios
     aum_churn_risk     — Cortex AI predictive retention score (0–100)
     gips_composites    — GIPS-compliant performance composites
     performance_attr   — Attribution analysis (GIPS standard)
     compliance_audit   — Immutable audit trail

SNOWFLAKE DATA MARKETPLACE (Zero-Copy External Datasets)
  → FactSet equities pricing: MARKETPLACE.FACTSET.EQUITIES_DAILY
  → ESG scores (MSCI/Refinitiv): MARKETPLACE.ESG.COMPANY_SCORES
  → OFAC sanctions: MARKETPLACE.COMPLIANCE.OFAC_RECORDS
  → Credit bureau: MARKETPLACE.CREDITBUREAU.SCORES
  → These are mounted in-place — no data movement, no pipeline maintenance
```

### 3.3 Step 3: Catalog (AI Assistant + Human-in-the-Loop)

The catalog is not a passive index — it is the governance backbone of the Data Mesh. Every data product has a defined owner, SLA, classification, and lineage chain.

```
DUAL-LAYER CATALOG ARCHITECTURE

LAYER 1 — AWS Glue Data Catalog (Cross-System Metadata Hub)
  ├── Federated lineage from Snowflake, S3, and MWAA pipelines
  ├── Classification tags: PII | CONFIDENTIAL | GIPS_REGULATED | SEC_17A4
  ├── Owner assignment per table and per column
  └── Schema Registry: versioned schemas with consumer impact analysis on change

LAYER 2 — Snowflake Horizon (Policy Enforcement)
  ├── Masking policies applied at column level (not in the application layer)
  ├── Row-Level Security policies applied at table level
  └── Cross-cloud, cross-consumer governance — same policy regardless of consumer

AI + HITL WORKFLOW FOR UNSTRUCTURED DATA CATALOGING
  Step 1: New document lands in S3 (e.g., client RFP PDF)
  Step 2: AWS Textract extracts text + structure (tables, forms)
  Step 3: Amazon Bedrock (Claude 3.5 Sonnet) analyzes content:
          → Suggests document type, classification, owner, relevant sections
          → Outputs structured metadata JSON with confidence scores
  Step 4: MWAA DAG routes low-confidence suggestions (< 0.85) to human queue
  Step 5: Nomura Data Steward reviews AI suggestion in Salesforce Task:
          → Accept as-is | Edit and approve | Reject and reclassify
  Step 6: Approved metadata committed to Snowflake Horizon + AWS Glue Catalog
  Step 7: Document added to OpenSearch vector index for DDQ/RFP agent retrieval

GOVERNANCE BENEFIT: 100% catalog accuracy — AI provides speed,
human provides judgment for edge cases. No sensitive financial document
is miscatalogued by a fully autonomous system.
```

**Data Steward ownership model:**

| Data Domain | Owner | Review Cadence | AI Automation Level |
|---|---|---|---|
| Portfolio & performance data | Head of Client Technology | Quarterly + on schema change | 95% auto-approve (high confidence structured data) |
| Client PII & CRM data | Chief Data Officer | Annual + on GDPR trigger | 80% auto-approve |
| Compliance & regulatory docs | CCO | Immediate on policy change | 50% auto-approve (conservative for compliance) |
| Fund marketing documents | Head of Marketing | Quarterly | 90% auto-approve |
| Technology controls (DDQ sections) | Head of Client Technology | Quarterly | 70% auto-approve (live-lookup flag for sensitive answers) |

### 3.4 Step 4: Process (Orchestration & AI-Powered Transformation)

**Amazon MWAA** orchestrates all transformation DAGs. **Snowflake Cortex AI** runs AI workloads natively on the data without movement.

```
MWAA ORCHESTRATION — KEY DAGs

DAG: nomura_cqrs_daily_gold_refresh
  Schedule: 06:00 UTC daily + triggered by event volume threshold
  Steps:
    1. Validate Bronze completeness (record count vs. source system checksum)
    2. Run AWS Glue DataBrew quality gates:
       → Null check on mandatory fields (customer_id, portfolio_id, timestamp)
       → Range check on monetary values (reject if NAV outside ±5% of prior day)
       → Duplicate detection (event_id idempotency check)
    3. Execute dbt Silver transformation (Bronze → Silver)
    4. Execute dbt Gold transformation (Silver → Gold)
    5. Refresh Snowflake materialized views (customer_360, risk_metrics)
    6. Trigger downstream notification: Salesforce cache invalidation (Elasticache TTL reset)
    7. Log data quality score to CloudWatch dashboard

DAG: nomura_cqrs_cortex_ai_scoring
  Schedule: Hourly
  Steps:
    1. Identify clients with portfolio movement > 2% in past 24h (Snowflake query)
    2. Run Snowflake Cortex AI sentiment analysis on Service Cloud chat logs →
       update client_sentiment_score in Gold layer
    3. Run Snowflake Cortex AI AUM churn risk model:
       Input: portfolio_daily + crm_events + client_sentiment + market_volatility
       Output: aum_churn_risk score (0–100) + top 3 contributing factors
    4. Flag clients with aum_churn_risk > 70 → Salesforce task for covering RM

DAG: nomura_cqrs_document_catalog
  Trigger: S3 event (new document in landing zone)
  Steps: [As described in Step 3 AI+HITL workflow above]

DAG: nomura_cqrs_gips_validation
  Schedule: Daily post-NAV (18:00 UTC)
  Steps:
    1. Pull GIPS composite portfolios from Aladdin Gold layer
    2. Run GIPS calculation engine (deterministic — not AI-generated)
    3. Validate composites: completeness, TWR accuracy, terminated portfolio retention
    4. Write GIPS-certified composites to gips_composites Gold table
    5. If validation fails: halt distribution, alert Performance Team + CCO
```

**Snowflake Cortex AI — Native Processing:**
```sql
-- AUM Churn Risk Scoring (runs inside Snowflake — zero data movement)
SELECT
    c.customer_id,
    c.client_name,
    c.aum_total,
    -- Cortex AI predicts churn probability from combined signals
    SNOWFLAKE.CORTEX.COMPLETE(
        'llama2-70b-chat',
        CONCAT(
            'Assess institutional client churn risk (0-100). ',
            'Portfolio YTD return: ', c.portfolio_ytd_return::STRING, '%. ',
            'Peer benchmark YTD: ', c.benchmark_ytd::STRING, '%. ',
            'Days since last advisor contact: ', c.days_since_contact::STRING, '. ',
            'Open service cases: ', c.open_case_count::STRING, '. ',
            'Respond with risk score only (integer 0-100).'
        )
    )::INTEGER AS aum_churn_risk_score,
    CURRENT_TIMESTAMP() AS scored_at
FROM gold.customer_360 c
WHERE c.is_institutional = TRUE
  AND c.aum_total > 10000000  -- Focus on accounts >$10M AUM
;
```

### 3.5 Step 5: Deliver (The Query Path & Zero-Copy Consumption)

The Query Path is completely decoupled from the Command Path. No write operation can slow down a read. No read operation can degrade ingestion throughput. This is the CQRS guarantee.

```
┌─────────────────────────────────────────────────────────────────────────┐
│  QUERY PATH — ZERO-COPY DELIVERY LAYER                                  │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  CONSUMER 1: Salesforce Sales Cloud (Wealth Advisors & Institutional    │
│              RMs)                                                       │
│                                                                          │
│  Read mechanism: Snowflake External Objects (federated query)            │
│  What they see: customer_360 | portfolio_daily | aum_churn_risk         │
│  Latency: <500ms p95 (Elasticache Redis 5-min TTL, 94% hit rate)       │
│  Identity: OIDC token passthrough → Snowflake Horizon enforces RLS      │
│  Zero-Copy: Salesforce NEVER stores portfolio data. Queries Snowflake   │
│             in-place. No ETL. No duplication.                            │
│                                                                          │
│  What the advisor sees at meeting prep (Stuart use case):                │
│  ┌─────────────────────────────────────────────────────────┐            │
│  │ CLIENT: BlackRock Alternatives Fund II [AUM: $240M]     │            │
│  │ Portfolio YTD: -1.2%  Benchmark: -3.8%  [OUTPERFORMING]│            │
│  │ AUM Churn Risk: 28/100 [LOW] ← Cortex AI score         │            │
│  │ Last Contact: 14 days ago [Stuart's AI meeting prep]    │            │
│  │ Open Cases: 0  ESG Score: 72/100  OFAC: CLEAR          │            │
│  │ NEXT BEST ACTION: Discuss Q2 alternatives allocation    │            │
│  └─────────────────────────────────────────────────────────┘            │
│                                                                          │
│  CONSUMER 2: Salesforce Service Cloud (Client Services Team)            │
│                                                                          │
│  Read mechanism: Snowflake External Objects                              │
│  What they see: customer_360 | open_cases | compliance_flags            │
│  CQRS benefit: Service queries never compete with Aladdin ingestion      │
│                 (separate virtual warehouses per consumer)               │
│                                                                          │
│  CONSUMER 3: Nomura AI Chatbot (Client-Facing + Internal Support)       │
│                                                                          │
│  Read mechanism: Snowflake Virtual Warehouse (Query Path)                │
│  + OpenSearch vector index (DDQ/RFP knowledge base)                     │
│  Context retrieved per session:                                          │
│    → Client portfolio summary (Snowflake Gold)                          │
│    → Relevant fund documents (OpenSearch semantic search)               │
│    → Compliance-checked disclosure content (Bedrock Guardrails)         │
│  CQRS isolation: MSK high-velocity ingestion has ZERO impact on         │
│                   chatbot response latency                               │
│                                                                          │
│  CONSUMER 4: Client Portal (Institutional + Wealth)                     │
│                                                                          │
│  Read mechanism: REST API → Snowflake Virtual Warehouse                 │
│  What clients see: portfolio_daily | performance_attr | account_docs    │
│  Freshness: <2 seconds from Aladdin event to portal display             │
│                                                                          │
│  CONSUMER 5: Institutional DDQ/RFP Agent (NAPCE)                       │
│                                                                          │
│  Read mechanism: OpenSearch (50K+ approved responses) + Snowflake      │
│                   (real-time GIPS composites)                           │
│  Cross-reference: Snowflake gips_composites table used to verify any   │
│                    performance claim in DDQ response — zero hallucination│
│                    risk for regulatory performance data                  │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

**Snowpipe Streaming for CQRS read consistency:**
```sql
-- Snowpipe Streaming configuration ensures Query Path Gold layer is
-- updated within <2 seconds of Command Path Bronze write
-- This closes the CQRS read staleness gap proactively

ALTER PIPE aladdin_positions_pipe SET
    ERROR_INTEGRATION = 'nomura_alert_integration';

-- Snowpipe continuous ingestion → Bronze refresh
-- dbt incremental model → Silver refresh (on new Bronze rows)
-- Materialized view refresh → Gold layer (auto-refresh on Silver change)
-- Elasticache TTL = 300s (5 min) — acceptable advisory staleness window

-- Kafka lag monitoring (Prometheus metric):
-- nomura_msk_consumer_lag{topic="aladdin.positions"} < 1000 (alert threshold)
```

### 3.6 Step 6: Security & Governance (Snowflake Horizon + Zero-Trust)

Governance is not a post-processing step. Every byte that enters the Command Path is tagged and classified. Every byte that leaves the Query Path is filtered by the same policy.

```
ZERO-TRUST GOVERNANCE ARCHITECTURE

IDENTITY LAYER (Every request carries identity context)
  ┌────────────────────────────────────────────────────────┐
  │ 1. User authenticates: Salesforce | Portal | Chatbot  │
  │    → AWS Cognito issues OIDC token                    │
  │    → Token contains: user_id, role, department, geo   │
  └────────────────────────────┬───────────────────────────┘
                               ↓
  ┌────────────────────────────────────────────────────────┐
  │ 2. AWS Lambda validates token + routes to Snowflake   │
  │    → OIDC token passed to Snowflake REST API          │
  │    → Snowflake maps OIDC claims to Snowflake roles    │
  └────────────────────────────┬───────────────────────────┘
                               ↓
  ┌────────────────────────────────────────────────────────┐
  │ 3. Snowflake Horizon enforces policies                 │
  │    → Dynamic Data Masking per column                  │
  │    → Row-Level Security per table                     │
  │    → Tag-based governance (PII | GIPS_REGULATED)      │
  └────────────────────────────────────────────────────────┘

MASKING POLICY (Wealth Advisor example):
  Role: WEALTH_ADVISOR
  → client_ssn: SHOW (confirmed wealth channel)
  → account_number: SHOW (confirmed wealth channel)
  → portfolio_positions: SHOW (within advisory scope)

  Role: INTERNAL_SUPPORT
  → client_ssn: MASK (***-**-6789)
  → account_number: MASK (****4829)
  → portfolio_positions: AGGREGATE ONLY (total AUM, no line items)

  Role: AI_CHATBOT_SERVICE
  → client_ssn: BLOCK (never returned to chatbot LLM context)
  → account_number: MASKED LAST 4 only
  → portfolio_positions: SHOW (necessary for portfolio queries)

AUDIT TRAIL (Immutable, 7-Year Retention)
  → Every query logged: user, timestamp, query text, rows returned
  → Every masking decision logged: which fields were masked and why
  → Every policy change logged: who changed, from what, to what
  → Storage: S3 Object Lock (WORM) + AWS KMS CMK encryption
  → Accessible to: Compliance officers only (separate Snowflake role: AUDIT_READER)
```

**Snowflake Horizon Masking Policy — SQL Implementation:**
```sql
-- Universal PII masking policy (applied to customer_360 Gold table)
CREATE OR REPLACE MASKING POLICY nomura_pii_mask AS
    (val VARCHAR) RETURNS VARCHAR ->
    CASE
        WHEN CURRENT_ROLE() IN ('WEALTH_ADVISOR', 'WEALTH_ADMIN', 'INSTITUTIONAL_RM')
            THEN val                                   -- Authorized: show full value
        WHEN CURRENT_ROLE() IN ('INTERNAL_SUPPORT', 'SERVICE_AGENT')
            THEN CONCAT('***-**-', SUBSTRING(val, -4)) -- Partial mask
        WHEN CURRENT_ROLE() = 'AI_CHATBOT_SERVICE'
            THEN CONCAT('****-', SUBSTRING(val, -4))  -- Last 4 only for AI context
        ELSE 'REDACTED'                                -- Default: full block
    END;

-- Row-Level Security: advisors see only their assigned clients
CREATE OR REPLACE ROW ACCESS POLICY nomura_advisor_rls AS
    (assigned_advisor_id VARCHAR) RETURNS BOOLEAN ->
    CASE
        WHEN CURRENT_ROLE() = 'WEALTH_ADMIN' THEN TRUE              -- Admin sees all
        WHEN CURRENT_ROLE() = 'WEALTH_ADVISOR'
            THEN assigned_advisor_id = CURRENT_USER()               -- Own clients only
        WHEN CURRENT_ROLE() = 'INSTITUTIONAL_RM'
            THEN assigned_advisor_id = CURRENT_USER()               -- Own accounts only
        ELSE FALSE
    END;

ALTER TABLE gold.customer_360
    MODIFY COLUMN client_ssn SET MASKING POLICY nomura_pii_mask,
    MODIFY COLUMN account_number SET MASKING POLICY nomura_pii_mask;

ALTER TABLE gold.customer_360
    ADD ROW ACCESS POLICY nomura_advisor_rls ON (assigned_advisor_id);
```

---

## 4. CQRS DATA MESH — DOMAIN OWNERSHIP MODEL

The Data Mesh principle of **domain ownership** applies BIAN service boundaries to data product ownership. Each BIAN service domain owns the data products it produces.

| Data Product | Producing Service Domain (BIAN) | Owner Role | Consumer Domains |
|---|---|---|---|
| `customer_360` | Customer & Party Management (Domain 1) | CRM Platform Lead | Sales & Distribution, Reporting, AI Chatbot |
| `portfolio_daily` | Investment Portfolio Management (Domain 4) | Head of Client Technology (integration boundary owner) | Reporting, Client Portal, DDQ Agent |
| `gips_composites` | Client Reporting & Performance (Domain 6) | Performance Analyst + CCO co-ownership | Sales & Distribution (RFPs), Reporting, DDQ Agent |
| `aum_churn_risk` | Sales & Distribution (Domain 2) | Head of Client Technology | Sales Cloud (advisor alerts), Service Cloud (retention flags) |
| `approved_content` | Sales Enablement (Domain 7) | Head of Content + CCO co-ownership | Seismic, DDQ/RFP Agent, AI Chatbot knowledge base |
| `audit_trail` | Data & Integration (Domain 8) | CDO + CCO | Compliance (read-only), External auditors (governed export) |

**Data product SLA contract (example — `customer_360`):**
```
DATA PRODUCT: customer_360
Owner:       CRM Platform Lead + Head of Client Technology
Freshness:   <2 seconds from Aladdin event to Gold layer update
Availability: 99.99% (Snowflake multi-cluster + Elasticache fallback)
Latency:     <500ms p95 for Salesforce External Object queries
Consumers:   Salesforce Sales Cloud | Service Cloud | AI Chatbot | Client Portal
Quality SLA: Zero null customer_id | Zero duplicate rows | AUM delta <0.01% vs Aladdin
Lineage:     aladdin_raw → aladdin_deduplicated → customer_360 (3-step chain, logged)
Classification: CONFIDENTIAL (PII present) | GIPS_REGULATED (performance fields)
```

---

## 5. AI CHATBOT INTEGRATION (CQRS QUERY PATH)

### 5.1 How the AI Chatbot Reads from the Data Mesh

The AI Chatbot is a **pure Query Path consumer**. It never writes to any source system. It never modifies CRM records. It reads from Snowflake Gold layer and OpenSearch in real time, and its responses are always governed by the same Snowflake Horizon policies that apply to every other consumer.

```
CLIENT CHATBOT QUERY: "What was my portfolio performance last quarter?"

Step 1: Client authenticates via Nomura portal → OIDC token issued
         Token: { role: "INSTITUTIONAL_CLIENT", client_id: "CL-00429" }

Step 2: Chatbot (Bedrock Agent) calls Snowflake Query Path:
         SELECT portfolio_ytd_return, benchmark_ytd, attribution_top3
         FROM gold.portfolio_daily
         WHERE client_id = :client_id
           AND quarter = CURRENT_QUARTER() - 1;

Step 3: Snowflake Horizon applies RLS:
         → client_id = CL-00429 filter enforced from OIDC token
         → Dynamic masking: show portfolio detail (client has full access to own data)

Step 4: Snowflake returns: portfolio_ytd = +2.3%, benchmark = +1.8%, top attribution = "Alternatives +0.8%"

Step 5: Bedrock Agent constructs response:
         "Your portfolio returned +2.3% last quarter, outperforming the benchmark
          by 0.5%. The largest contributor was your alternatives allocation (+0.8%).
          Would you like to see the full attribution analysis?"

Step 6: Bedrock Guardrails validates output:
         → No guarantee language (SEC 206(4)-1 compliance)
         → No unverified performance claims (all figures sourced from GIPS Gold table)
         → PII not present in response text

Total end-to-end latency: <2 seconds
```

### 5.2 AI Chatbot CQRS Isolation Benefit

Under the legacy architecture, a high-volume Aladdin data import would slow Salesforce API response times, which would cascade into chatbot latency. Under CQRS:

```
COMMAND PATH LOAD EVENT (e.g., quarter-end Aladdin batch: 5M position updates)
  → MSK Kafka queue absorbs burst: 2M events/sec capacity
  → Snowpipe Streaming writes to Bronze: no virtual warehouse required
  → Warehouse NOMURA_INGEST_WH handles transformation
  ↓
  ZERO IMPACT ON:
  → Salesforce External Object queries (NOMURA_SALESFORCE_WH — separate warehouse)
  → AI Chatbot queries (NOMURA_AI_WH — separate warehouse)
  → Client portal API queries (NOMURA_PORTAL_WH — separate warehouse)
```

Separate Snowflake virtual warehouses per consumer mean **quarter-end data loading never degrades advisor or client experience.**

---

## 5.3 AI CONTENT GOVERNANCE — THE STRICTLY GOVERNED RAG MESH

### The Architectural Mandate: RAG, Not Fine-Tuning

The NAPCE AI Digital Agent (and all client-facing AI at Nomura) is built on **Retrieval-Augmented Generation (RAG)**, not fine-tuned model weights. This is a deliberate, non-negotiable architectural decision.

**Executive Summary — The 5 Pillars of AI Content Governance:**

> The five pillars answer a single business question: *How do we ensure that an AI agent operating at the speed of automation maintains the accuracy standards required by a regulated financial institution?*
>
> **Pillar 1** ensures the agent can only retrieve currently-approved content — expired content is architecturally unreachable.
> **Pillar 2** ensures every AI-generated answer has a named, accountable human owner permanently bonded to it.
> **Pillar 3** ensures corrections update the knowledge base source — not just the next response.
> **Pillar 4** ensures no AI-generated document reaches a client or counterparty without explicit human authorization.
> **Pillar 5** ensures every AI output, every human modification, and every approval is immutably recorded and independently verifiable by regulators.
>
> Together, these five pillars are the trust architecture that makes institutional clients comfortable with AI in their DDQ/RFP workflow.

**Why fine-tuning is categorically rejected for factual content:**

> Fine-tuning bakes facts into static model weights. When a fund strategy changes, when GIPS performance data is restated, or when a portfolio manager leaves the firm — the fine-tuned model retains the old answer permanently. Correcting it requires a full model retrain ($50,000–$500,000, 4–8 weeks). There is no governance, no audit trail, and no surgical update path. For regulated financial content, fine-tuning is architecturally incompatible with compliance requirements.

**The RAG Mesh alternative:**
- The LLM supplies reasoning capability and language fluency — it never stores facts
- All factual content lives in Amazon Bedrock Knowledge Bases (OpenSearch Serverless)
- The LLM retrieves chunks at query time; governance applies to the retrieval layer, not the model
- Any fact can be updated, expired, or audited in real time — without touching the model

### The 5 Pillars of AI Content Governance

#### Pillar 1 — "Current Answers Only" Perimeter

No AI response may ever be generated from expired, superseded, or unapproved content.

```
CONTENT LIFECYCLE ENFORCEMENT

S3 Bucket (KnowledgeBase Source)
  ├── Object metadata: { Status: "Approved_Production", Review_Date: "2026-03-31" }
  ├── S3 Object Expiration Rule: TTL triggers Lambda `napce_kb_expiry_purge` on expiry date
  ├── Lambda purge: DELETE from OpenSearch index where chunk_id = expired_object_id
  │                  → LLM physically cannot retrieve the vector (it no longer exists)
  ├── Lambda DLQ: SQS `napce_kb_expiry_dlq` for failed purge operations
  └── CloudWatch alarm: DLQ MessageCount > 0 → PagerDuty (ensures no silent expiry failures)

AWS Glue Catalog metadata:
  → `Status: Approved_Production` = active in knowledge base
  → `Status: Pending_Review` = excluded from retrieval context
  → `Status: Archived` = purged from OpenSearch; 7-year cold storage in S3 Glacier

Bedrock Guardrails filter:
  → Retrieval context restricted to chunks tagged `Status: Approved_Production`
  → Any chunk missing this tag is excluded from the RAG context window
```

**Governance outcome:** The LLM is architecturally incapable of retrieving expired content. The perimeter is enforced by infrastructure, not by prompt instructions alone.

#### Pillar 2 — Clear Ownership & Accountability

Every chunk of knowledge in the AI agent's knowledge base has a named, accountable human owner.

```
METADATA-AS-CODE (Enforced at ingestion via AWS Glue)

Each vectorized chunk carries:
  {
    "chunk_id": "eso_policy_v4_chunk_027",
    "source_document": "ESG_Investment_Policy_v4.pdf",
    "owner_entra_id": "kdoe@nomura.com",
    "owner_salesforce_id": "0055g000003xYZ",
    "approved_by": "compliance_review_panel",
    "effective_date": "2025-11-01",
    "review_due": "2026-05-01",
    "status": "Approved_Production"
  }

Client-facing AI response citation:
  "Source: ESG Policy v4 (Effective Nov 2025) · Owner: Kathleen Doe · Next Review: May 2026"

Salesforce Owner lookup:
  → If owner_salesforce_id is inactive (employee terminated), route to successor
  → Successor assignment: automated via Workday → EventBridge → Salesforce workflow

INGESTION VALIDATION (AWS Glue — enforced before vectorization):
  → Mandatory fields validated at ingestion: owner_entra_id, owner_salesforce_id, status, review_due
  → Objects missing any required field → routed to S3 DLQ `napce_kb_metadata_dlq` — NEVER vectorized
  → CloudWatch alarm: napce_kb_metadata_dlq MessageCount > 0 → PagerDuty (immediate escalation)
  → Governance outcome: no orphaned (unowned) chunks can enter the OpenSearch retrieval index
```

**Governance outcome:** Every AI-generated answer is permanently traceable to the person who authored and is responsible for its accuracy. AWS Glue enforces this at ingestion — malformed or unowned metadata objects are quarantined before they can contaminate the retrieval index.

#### Pillar 3 — Centralized Updates, No Ad-Hoc Fixes

Knowledge base corrections must flow through a governed pipeline — never patched directly in OpenSearch.

```
CLOSED-LOOP FEEDBACK UI (Salesforce Lightning Web Component)

RM or Compliance Officer identifies an incorrect AI answer
  ↓
"Flag Inaccuracy" button in Salesforce DDQ/RFP Review UI
  ↓
Correction submission form (required fields: correct answer text, policy reference, urgency)
  ↓
Salesforce Platform Event → MWAA DAG triggered
  ↓
MWAA `knowledge_base_correction_dag`:
  Step 1: Archive old chunk in S3 (audit record preserved)
  Step 2: Write corrected chunk to S3 with updated metadata (new version, same owner_id)
  Step 3: Re-vectorize corrected chunk via Amazon Bedrock Embeddings
  Step 4: Upsert corrected vector into OpenSearch Serverless
  Step 5: Redis cache invalidation — KEYS pattern: `kb_query:*:{chunk_id}*`
           → Precise invalidation of affected query patterns; no global cache flush
  Step 6: Salesforce notification: "Knowledge base updated — correction live"

EXPORT BLOCK:
  → RFP/DDQ export is BLOCKED in Salesforce until the flagged section is corrected
  → Block enforced at Salesforce page layout level — cannot be bypassed by any user role
```

**Governance outcome:** Incorrect content is corrected in the source, re-vectorized, and live in the knowledge base within minutes — not buried in a single response where the error persists in the next 50 RFPs.

#### Pillar 4 — Human-in-the-Loop Execution (AWS Step Functions Saga)

No AI-generated content reaches a client or counterparty without explicit human authorization.

```
HITL SAGA — NAPCE DDQ/RFP APPROVAL WORKFLOW

Step 1: NAPCE Agent generates draft DDQ/RFP response
  → Content grounded exclusively in Approved_Production knowledge base
  → Confidence scores logged per section

Step 2: AWS Step Functions — Wait_For_Callback state
  → Activity token issued: { task_token: "sfn_token_abc123", expiry: "72h" }
  → Workflow PAUSES — execution cost: $0 while waiting
  → Principal Portfolio Manager / Compliance Officer notified via Salesforce task

Step 3: Human review in Salesforce Lightning Web Component
  → AI draft presented with source citations and confidence scores per section
  → Reviewer can: Approve / Edit / Reject / Escalate

Step 4: "Approve" action
  → Step Functions callback: SendTaskSuccess(taskToken: "sfn_token_abc123")
  → Workflow resumes
  → Watermarked final PDF generated (reviewer ID embedded)
  → TransactionLog entry written: { reviewer_id, review_timestamp, original_draft_hash, final_output_hash }
  → Salesforce Opportunity stage updated: "Proposal Submitted"

TIMEOUT POLICY:
  → 72-hour HITL timeout → automatic escalation to senior reviewer
  → 96-hour escalation → CloudWatch alarm → Head of Client Technology notification
```

**Governance outcome:** The AI accelerates document production by 90%+; the human retains legal and regulatory accountability for every document submitted.

#### Pillar 5 — Immutable Audit Trail

Every AI-generated output, every human modification, and every approval is permanently recorded and independently verifiable by Compliance, Legal, and regulators.

```
THREE-PAYLOAD AUDIT ARCHITECTURE

Payload 1 — Raw AI Output:
  { 
    "payload_type": "ai_generated",
    "bedrock_request_id": "br-xxxx",
    "model_id": "anthropic.claude-3-5-sonnet",
    "retrieved_chunks": ["chunk_id_1", "chunk_id_2", ...],
    "raw_response": "...",
    "confidence_scores": { "section_1": 0.94, "section_2": 0.88 }
  }

Payload 2 — Human-Submitted Output:
  {
    "payload_type": "human_approved",
    "reviewer_id": "khm@nomura.com",
    "reviewer_role": "Principal Portfolio Manager",
    "final_text": "...",
    "review_timestamp": "2026-02-14T14:23:11Z",
    "approval_method": "step_functions_callback"
  }

Payload 3 — Diff Delta (Machine-Readable):
  {
    "payload_type": "diff_delta",
    "diff": [{ "op": "replace", "path": "/sections/2/text", 
               "old": "...", "new": "..." }],
    "delta_lines_changed": 7,
    "human_intervention_score": 0.12  // 12% of content modified by human
  }

STORAGE ARCHITECTURE:
  Primary: Snowflake Immutable Tables (append-only, no UPDATE/DELETE)
           → CRO-queryable: "Show me all proposals where human modified >20% of AI draft"
  Mirrored: Amazon QLDB (cryptographic digest chain — SHA-256 journal hash; independently
            verifiable without distributed consensus; suitable for regulated audit trail use)
  S3 Object Lock (COMPLIANCE mode):
    → 7-year retention per SEC 17a-4 / GIPS 2019 Annex B
    → Root override disabled; no administrator can shorten retention period
```

**Governance outcome:** Regulators (SEC, FINRA, internal Compliance) can reconstruct the exact provenance of any client document — what the AI said, what the human changed, who approved it, and when.

### Compliance Framework Alignment

| Standard | Requirement | Architecture Mapping |
|---|---|---|
| **ISO/IEC 42001** (AI Management System) | Risk management, transparency, accountability for AI systems | Pillars 2 (ownership), 4 (HITL), 5 (audit trail) directly satisfy ISO 42001 Annex A controls |
| **NIST AI RMF** (AI Risk Management Framework) | Govern, Map, Measure, Manage AI risks | Pillar 1 (content currency) = Manage; Pillar 3 (centralized updates) = Govern; Pillar 5 (audit) = Measure |
| **SEC 17a-4** (Records Retention) | 7-year immutable retention of broker communications | S3 Object Lock COMPLIANCE mode; QLDB cryptographic chain |
| **FINRA Rule 4511** (Books and Records) | Accurate and current records; retrievable on demand | Snowflake Immutable Tables with CRO query access |
| **GIPS 2019 Annex B** | Composite methodology documentation; annual verifier access | Immutable audit trail provides verifier-accessible documentation |

---

## 6. PROACTIVE MITIGATIONS — THE "LEMONS" TABLE

| Risk Area | Risk Description | Proactive Mitigation Strategy (The "Lemon Squeezer") |
|---|---|---|
| **Data Quality Degradation** | Stale or incorrect data in the Gold layer reaches the AI Chatbot or Salesforce advisor dashboard, causing client-facing errors | MWAA pipelines include AWS Glue DataBrew automated quality gates before Bronze→Silver promotion. Records failing range checks, null constraints, or duplicate detection are quarantined in a Dead Letter Queue (DLQ) — never promoted to Silver. Data quality score logged to CloudWatch; alert on score < 99.5%. |
| **AI Hallucinations in Cataloging** | AI metadata tagger misclassifies a sensitive regulatory document as public, making it available to unauthorized consumers | MWAA HITL workflow mandates human Data Steward approval for all low-confidence AI metadata suggestions (threshold: Bedrock confidence < 0.85). Conservative by design: the AI can only propose, never finalize. High-risk classifications (GIPS_REGULATED, SEC_17A4) require human approval regardless of confidence score. |
| **CQRS Read Staleness** | The Query Path Gold layer lags behind Command Path Bronze by more than the acceptable freshness window — chatbot or advisor sees stale data during peak load | Snowpipe Streaming provides sub-2-second ingestion. Materialized view incremental refresh on Silver changes. Elasticache Redis 5-min TTL as advisory staleness window. CloudWatch alert: `nomura_msk_consumer_lag > 5000 messages` → PagerDuty. Fallback: Elasticache serves last-known-good state if Snowflake refresh is delayed. |
| **Salesforce API Rate Limits** | High-frequency CRM activity causes Salesforce REST API throughput to collapse, breaking advisor dashboards | Zero-Copy architecture: Salesforce never receives bulk data pushes. Advisors query Snowflake External Objects in-place. Rate limits only apply to metadata/event sync (limited to <1,000 AppFlow records/run). CQRS Command Path uses MSK Kafka, never Salesforce APIs for ingestion. |
| **Data Privacy Leaks (GDPR/CCPA)** | Wealth advisor accesses a client in a different department; AI Chatbot exposes PII in LLM conversation context | Snowflake Horizon Dynamic Data Masking + Row-Level Security enforced at column level — not in application code, not removable by application developers. AI Chatbot role `AI_CHATBOT_SERVICE` blocked from SSN, masked on account numbers by architecture. GDPR right-to-forget: anonymize-in-place (hash + retain audit proof) without breaking 7-year audit chain. |
| **FinOps Cost Explosion** | Snowflake on-demand compute charges scale unexpectedly as AI Chatbot concurrency grows | Separate virtual warehouses with Snowflake Resource Monitors: credit quota per warehouse per hour. Elasticache 94% hit rate eliminates majority of Snowflake compute. AI Chatbot uses `NOMURA_AI_WH` (auto-suspend after 60 seconds idle). Cost alert: `warehouse_spend > $500/hour` → auto-suspend + PagerDuty. Monthly FinOps review: cost per chatbot query, cost per advisor session. |
| **Cortex AI Bias in Churn Scoring** | AUM churn risk model produces biased scores that systematically under-rank clients from certain geographies or product categories | AUM churn risk model (Snowflake Cortex) outputs logged with input features. Monthly model drift detection: compare predicted churn vs. actual AUM outflows (30-day lag). If drift > 5%, trigger model revalidation with Performance Team. Human RM alert (churn_risk > 70) requires RM confirmation before automated retention intervention is triggered — AI recommends, human acts. |
| **Fine-Tuning Misconception** | Team proposes to "retrain the AI agent" when fund strategy, fee schedule, or GIPS methodology changes — baking facts into static model weights. Each factual update requires full model retrain ($50K–$500K, 4–8 weeks). After retraining, the old answer is permanently overwritten — no audit trail of what the model previously said. | **Architecture enforces RAG-only for factual content and prohibits fine-tuning for facts.** Knowledge base in OpenSearch is updated via the Closed-Loop Feedback DAG (Pillar 3 above) — surgical, auditable, immediate. Fine-tuning is reserved exclusively for tone and jargon calibration (e.g., "use GIPS-precise language") — never for factual content that must be updatable and auditable. |
| **"Frankenstein" Answers (Policy Synthesis)** | Bedrock retrieves fragments from two overlapping policy documents (e.g., 2024 RFP template and 2025 RFP template) and synthesizes a chimera answer — technically fluent, factually incorrect, and untraceable to any single approved source. Compliance officer submits it to an institutional client. FINRA examination finds two contradictory policy citations in the same document. | **XML constraint in system prompt:** `<rule>Do not synthesize, summarize, or merge content from multiple policy documents. Output retrieved policy text verbatim. If two sources conflict, flag the conflict for human review rather than resolving it.</rule>` Combined with Bedrock Guardrails groundedness threshold (>0.90 minimum) and Pillar 3 Closed-Loop Feedback: conflicting chunks are flagged and routed to the document owner for resolution before re-vectorization. |
| **Orphaned Knowledge (Employee Departure)** | Senior portfolio manager authors 40 knowledge base chunks covering a niche credit strategy. She departs the firm. Her Entra ID is disabled. The chunks remain active in OpenSearch — no owner to review or update them. Six months later, the credit strategy mandate is terminated, but the AI agent continues citing the departed manager's analysis in live RFPs. | **Workday HR event integration via AWS EventBridge:** on `Employment_Status: Terminated`, EventBridge rule fires Lambda. Lambda queries Salesforce for all knowledge base chunks with `owner_salesforce_id` = departed employee. All chunks are atomically flagged `Status: Requires_Review` — immediately excluded from RAG retrieval (Pillar 1 perimeter). MWAA routes flagged chunks to designated successor's Salesforce review queue with pre-populated ownership transfer form. Knowledge base remains complete but governed. |

---

## 7. IMPLEMENTATION ROADMAP

### Phase 1: Data Mesh Foundation (Months 1–3)

**Goal:** Establish Command Path and centralized Snowflake Data Cloud.

| Week | Activity |
|---|---|
| 1–2 | Provision Snowflake BYOL on AWS account. Configure BYOL S3 external stage. |
| 1–2 | Deploy MSK Kafka cluster (3 brokers, 3 AZs, replication_factor=3). |
| 3–4 | Configure Debezium CDC connector on Aladdin → MSK topic `aladdin.positions`. |
| 3–4 | Build Bronze layer: Snowpipe Streaming `aladdin_raw`, `trades_raw`, `balances_raw`. |
| 5–6 | Deploy AWS Glue Data Catalog with Snowflake Horizon integration. |
| 5–6 | Implement AWS Cognito OIDC integration. Configure Snowflake role-OIDC mapping. |
| 7–8 | Build Silver dbt transformation models. Deploy Bronze → Silver quality gates (DataBrew). |
| 9–12 | Build Gold dbt models: `customer_360`, `portfolio_daily`, `risk_metrics`. |
| 9–12 | Configure Snowflake Horizon masking + RLS policies on Gold tables. |

**Success gate:** `customer_360` Gold table populated with <2-second Aladdin freshness, all masking policies active.

### Phase 2: Query Path & Salesforce Integration (Months 4–6)

**Goal:** Deliver real-time Customer 360 to wealth advisors via Salesforce Zero-Copy.

| Week | Activity |
|---|---|
| 13–14 | Create Snowflake External Connection in Salesforce. Define External Objects. |
| 15–16 | Deploy Apex controllers for federated portfolio query. |
| 15–16 | Deploy Elasticache Redis cluster (3 nodes, us-east-1, Multi-AZ). Configure 5-min TTL. |
| 17–18 | Test masking policies: verify PII hidden for service agents, visible to wealth advisors. |
| 19–20 | Deploy advisor dashboard MVP (Customer 360 + AUM Churn Risk score). |
| 21–24 | Activate Snowflake Data Marketplace: FactSet, ESG, OFAC (zero-copy). |

**Success gate:** Advisor sees Client 360 in <500ms p95 from Salesforce. Zero cross-department data leaks in penetration test.

### Phase 3: AI Chatbot Integration & MWAA Orchestration (Months 7–9)

**Goal:** Connect AI Chatbot to Query Path. Deploy MWAA DAGs and Cortex AI scoring.

| Week | Activity |
|---|---|
| 25–26 | Deploy MWAA environment (multi-AZ, dual schedulers). Deploy 4 MWAA DAGs. |
| 27–28 | Connect Bedrock AI Chatbot to Snowflake NOMURA_AI_WH (CQRS Query Path). |
| 27–28 | Deploy AI+HITL document cataloging pipeline (Textract → Bedrock → MWAA → Steward). |
| 29–30 | Deploy Snowflake Cortex AI AUM churn risk scoring DAG. |
| 31–32 | Connect AUM churn risk scores to Salesforce Task (>70 risk → RM alert). |
| 33–36 | Chaos engineering: simulate MSK failure, Snowflake warehouse failure, Salesforce timeout. |

**Success gate:** AI Chatbot returns context-accurate responses in <2 seconds. AUM churn risk score available in Salesforce. MWAA DAGs achieve 99.9% success rate over 30-day window.

### Phase 4: Data Mesh Governance & Optimization (Months 10–12)

**Goal:** Full Data Mesh domain ownership, FinOps optimization, and compliance certification.

| Week | Activity |
|---|---|
| 37–40 | Deploy Resource Monitors on all Snowflake virtual warehouses. FinOps dashboard live. |
| 37–40 | Complete GDPR/CCPA right-to-forget workflow (anonymize-in-place). |
| 41–44 | Third-party GIPS verification of `gips_composites` Gold table. |
| 41–44 | SEC 17a-4 compliance audit of 7-year audit trail (S3 Object Lock). |
| 45–48 | AWS ADX data product catalog: publish curated analytics as governed client data products. |
| 45–48 | Staff training: Data Stewards on HITL catalog workflow; Advisors on Customer 360 usage. |

**Success gate:** All KPIs at or above target. Full governance certification. Platform live for all consumer segments.

---

## 8. KEY PERFORMANCE INDICATORS

| KPI | Target | Baseline (Month 0) | Month 6 | Month 12 |
|---|---|---|---|---|
| **Customer 360 Query Latency (p95)** | <500ms | N/A (manual) | 800ms | 380ms |
| **Aladdin Data Freshness in Salesforce** | <2 seconds | 24–48 hours | 5 minutes | 1.8 seconds |
| **AI Chatbot Response Freshness** | <2 seconds | 48 hours stale | 10 minutes | 1.9 seconds |
| **AUM Churn Intervention Lead Time** | 30 days (vs T+3) | T+3 alert | 15 days | 30 days |
| **Data Duplication Cost Reduction** | 80% | 0% | 40% | 84% |
| **Snowflake Annual Cost (vs. legacy)** | <$700K | $4.4M/year (est.) | $2.2M | $700K (84% saved) |
| **Catalog Accuracy (AI+HITL)** | 100% | Manual, unaudited | 98% (HITL active) | 100% |
| **Advisor Dashboard Uptime** | 99.99% | 98.5% (single AZ) | 99.5% | 99.99% |
| **GDPR Right-to-Forget SLA** | <1 hour | Manual, days | 4 hours | 45 minutes |
| **Data Privacy Incidents** | 0 | Unknown | 0 (masking active) | 0 (12-month clean) |
| **DDQ/RFP Response Time Reduction** | 60–80% | 5+ business days | 3 days | 1 business day |

---

## 9. PANEL TALKING POINTS

### For Kathleen Hess McNamara (ED, Client Experience & AI)

**On client journey continuity:**
> *"The fundamental client experience problem in asset management is not the quality of any individual interaction — it is the consistency across interactions. When a client calls our service team having just logged off the portal, they do not want to re-explain their situation to an advisor who is looking at different numbers. The CQRS Data Mesh solves this architecturally: the portal, the AI Chatbot, and the Salesforce advisor desktop are all reading from the same Gold layer record at the same timestamp. The experience is continuous because the data is continuous."*

**On AI Chatbot reliability:**
> *"The AI Chatbot is only as reliable as the data it reads. If it queries a Salesforce cache that is 48 hours old, or a Snowflake table that has not been refreshed since yesterday's market close, the response will be precise but incorrect. The Query Path in our CQRS design eliminates that risk by design — the chatbot reads from a Gold layer that is updated within 2 seconds of any Aladdin event. Performance figures cited by the chatbot are sourced from the same GIPS-certified table that generates client reports. That is not convenience — that is a regulatory requirement we have built into the architecture."*

**On AUM retention:**
> *"The 30-day early warning on AUM churn risk is the highest-value output of the entire data mesh. A client whose portfolio has underperformed their benchmark by 200bps over two quarters, whose advisor contact cadence has dropped, and who has opened three service cases in the past month is signaling departure — but no individual system sees all three signals simultaneously. Snowflake Cortex AI, running on the unified Gold layer, correlates all three and surfaces a churn risk score of 78 in the advisor's Salesforce task queue. That is the difference between proactive retention and an exit interview."*

**Anticipated Question — AI Cataloging Risk:**
> *"For sensitive financial documents — GIPS-regulated performance data, client IMA agreements — the Human-in-the-Loop step is non-negotiable. The AI suggests; the designated Data Steward approves. For high-risk classifications like GIPS_REGULATED or SEC_17A4, human approval is mandatory regardless of the AI confidence score. The AI provides speed and consistency in metadata suggestion — the human provides regulatory judgment at the final gate."*

**On AI Content Governance (The Strictly Governed RAG Mesh):**
> *"Kathleen, the most important design decision for the NAPCE DDQ/RFP agent is one you will not see in a typical AI demo: we categorically rejected fine-tuning the LLM with factual financial content. Fine-tuning bakes facts into weights — when a fund strategy changes, the model retains the old answer until a full retrain. For a firm with regulatory obligations, that is not acceptable. Instead, every fact lives in OpenSearch, tagged with an owner and an expiry date. When an answer is wrong, we update the source in the knowledge base — not the prompt, not the model. The correction is live in minutes and is permanently auditable. That distinction — RAG for facts, never fine-tuning — is what makes the system trustworthy enough to put in front of an institutional client."*

**On Content Governance Adoption Sustainability:**
> *"The AI Digital Agent is only adopted if the people using it trust its output. One wrong answer — especially in a DDQ submission to a $500M institutional prospect — destroys that trust in a way that takes months to rebuild. The 5 Pillars of Content Governance exist precisely to prevent that. Pillar 1 ensures the agent physically cannot cite an expired policy. Pillar 2 ensures every answer has a named owner. Pillars 3 and 4 ensure corrections go through governance before going live. And Pillar 5 means that when a regulator asks 'who approved this response,' we have a cryptographically verified answer in under 60 seconds."*

**Anticipated Kathleen Question:** *"How do we handle it when two policy documents contradict each other and the AI synthesizes an answer from both?"*
> *"We prevent synthesis architecturally. The system prompt contains an explicit XML-tagged rule: the model must output retrieved policy verbatim and is prohibited from merging or summarizing across sources. If the retrieval layer finds two conflicting chunks, the Bedrock Guardrails groundedness check flags the conflict rather than producing a chimera answer — and the response routes to human review with both source citations visible side by side. The compliance officer resolves the conflict; the corrected answer is re-vectorized via the Closed-Loop Feedback pipeline. The contradiction is eliminated at source, not papered over in a single response."*

---

### For Stuart Mumley (Director, Client Platforms)

**On Salesforce architecture boundary:**
> *"In BIAN terms, Salesforce is our Customer & Party Management and Sales Distribution service domain — it owns relationship context and engagement history. But our portfolio data, performance analytics, and AUM churn risk scores belong in the Data Management service domain — which is Snowflake. The Zero-Copy External Object pattern keeps that BIAN service boundary clean. Salesforce never becomes the system of record for things outside its domain, which means it never accumulates the data quality debt that usually surfaces 18 months into a CRM expansion."*

**On Salesforce API stability:**
> *"The CQRS pattern eliminates our Salesforce API rate limit exposure entirely. Today, any bulk data sync to Salesforce competes with RM activity on the REST API. Under the CQRS architecture, all high-volume data flows — Aladdin positions, trading events, cash movements — flow through MSK Kafka and Snowpipe Streaming. They never touch the Salesforce API. Advisors query Snowflake in-place via External Objects. The only traffic on the Salesforce API is the actual RM interaction — which is what it was designed for."*

**On platform maintainability:**
> *"The Data Mesh domain ownership model means every data product has a named owner with a defined SLA contract. When performance data changes — GIPS composites are recalculated quarterly with a new verifier, for example — the Performance team updates their data product and the consuming services (Salesforce, AI Chatbot, portal) automatically receive the change without a platform release. That modularity is what makes the platform scalable beyond the initial implementation."*

**Anticipated Question — Quarter-End Surge Handling:**
> *"Quarter-end is the stress test for any client data platform. When Aladdin processes end-of-quarter NAV for all composites simultaneously, the data volume spikes 10× normal. Under CQRS, that spike is absorbed by the MSK Kafka queue — it is designed for 2M events per second. The Snowflake ingestion warehouse (`NOMURA_INGEST_WH`) processes the Bronze load. The advisor query warehouses (`NOMURA_SALESFORCE_WH`, `NOMURA_AI_WH`) are completely isolated from that load — they never see it. Quarter-end data loading literally cannot degrade advisor experience under this architecture."*

---

## 10. DOCUMENT METADATA

| Field | Value |
|---|---|
| **Document** | CUSTOMER_CENTRIC_DATA_STRATEGY_CQRS.md |
| **Version** | 1.0 |
| **Prepared by** | Calvin Lee |
| **Role** | ED, Head of Client Technology — Nomura Asset Management International |
| **Target audience** | Kathleen Hess McNamara · Stuart Mumley |
| **Architecture pattern** | Command Query Responsibility Segregation (CQRS) + Data Mesh |
| **AWS services** | MSK Kafka · Snowpipe Streaming · MWAA · Glue · AppFlow · Firehose · Cognito · ADX · S3 · KMS · Elasticache · Textract · Bedrock |
| **Snowflake capabilities** | BYOL · Cortex AI · Horizon · Data Marketplace · External Objects · Iceberg |
| **Salesforce integration** | External Objects (Zero-Copy) · AppFlow · OIDC token passthrough |
| **BIAN domains** | Domain 1 (Customer & Party) · Domain 2 (Sales & Distribution) · Domain 4 (Investment Portfolio) · Domain 6 (Client Reporting) · Domain 7 (Sales Enablement) · Domain 8 (Data & Integration) |
| **KPIs** | 360-degree client visibility · AUM churn prevention · 80% cost reduction · 30-day retention lead time |
| **Related documents** | [SALESFORCE_SNOWFLAKE_FINTECH_ARCHITECTURE.md](SALESFORCE_SNOWFLAKE_FINTECH_ARCHITECTURE.md) · [BIAN_FRAMEWORK_ASSET_MANAGEMENT.md](BIAN_FRAMEWORK_ASSET_MANAGEMENT.md) · [AGENTIC_WORKFLOW_AUTOMATION_NAPCE.md](AGENTIC_WORKFLOW_AUTOMATION_NAPCE.md) |
| **Evaluation** | [Cycle 1](evaluation/EVALUATION_CQRS_DATA_MESH_CYCLE_1.md) · [Cycle 2](evaluation/EVALUATION_CQRS_DATA_MESH_CYCLE_2.md) · [Cycle 3 — Certified](evaluation/EVALUATION_CQRS_DATA_MESH_CYCLE_3_FINAL_CERTIFICATION.md) |
| **Status** | APPROVED FOR PANEL PRESENTATION |
| **Last updated** | February 2026 |

---

*This document represents the senior technology executive synthesis of CQRS Data Mesh architecture for Nomura Client Technology — designed to articulate not just what the architecture does, but why each decision was made in the context of AUM retention, client experience, regulatory compliance, and platform maintainability.*
