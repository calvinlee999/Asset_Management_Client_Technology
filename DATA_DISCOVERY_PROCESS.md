# Data Discovery Process & Multi-Modal Distribution Mesh Architecture

**Document Version**: 1.0  
**Classification**: Technical Architecture Guide  
**Last Updated**: February 18, 2026  
**Target Audience**: Data Engineers, Principal Architects, FinTech Leaders

---

## Executive Summary

Data discovery is the foundation of any modern data analytics and distribution system. Before architecting and deploying a data analytics platform, organizations must answer critical questions about **what data exists**, **who owns it**, **where it's located**, **how it should be transformed**, and **who needs access and in what modality**.

For Nomura's Asset Management Client Technology platform, data discovery involves understanding:
- **24+ proprietary data sources** (Aladdin, Bloomberg, FactSet, internal risk engines)
- **4-tier multi-modal distribution** allowing clients to consume via REST API, Kafka, web dashboards, or AWS Data Exchange
- **Real-time latency requirements** ranging from <100ms (portfolio trades) to daily batches (regulatory reporting)
- **70+ institutional clients** with varying data consumption preferences and compliance needs

This document provides a comprehensive framework for data discovery aligned with Nomura's architecture vision.

---

## Part 1: Foundational Data Discovery Framework

### 1.1 What is Data Discovery?

**Definition**: Data discovery is the process of finding, understanding, and cataloging relevant data sources within an organization and the relationships between them.

**Strategic Value**:
- ✅ Identifies hidden data assets that can drive revenue
- ✅ Reduces time-to-insight by mapping data relationships upfront
- ✅ Enables compliance and governance by understanding data lineage
- ✅ Optimizes infrastructure spend by eliminating duplicate data pipelines
- ✅ Accelerates onboarding of new data consumers (70% faster time-to-value)

**Why It Matters for Asset Management**:
- Institutional clients operate on split-second decision timelines; stale data costs money
- Regulatory requirements (GIPS, FINRA, MiFID) demand audit trails and data lineage
- Vendor lock-in risk: Aladdin, Bloomberg, FactSet each own customer relationships; data mesh equalizes access
- Cost explosion: Petabyte-scale data ingestion without discovery leads to redundant pipelines and wasted infrastructure

---

## Part 2: Five-Step Data Discovery Framework

### Step 1: Define Business Value

**Objective**: Conduct discovery workshops with stakeholders to understand business goals and identify potential data sources.

#### 1.1 Business Value Alignment for Asset Management

**Example Discovery Workshop Questions**:

| Category | Question | Asset Management Context |
|----------|----------|-------------------------|
| **Revenue Opportunities** | How would getting insight into data provide value to the business? | **Premium Client Analytics**: Charge institutional clients $50K-$200K/year for proprietary portfolio attribution and risk analytics. Current: Nomura builds custom reports (manual, 2-week turnaround). Target: Self-serve due/day (automated, 1-hour turnaround) → 5x faster, higher attach rate. |
| | Are you looking to create a new revenue stream from your data? | **SaaS Licensing Model**: Tier 4 onboarding + Tier 5 AI analytics = $2M ARR by 2027. Current model: services revenue with declining margins. New model: SaaS with 70% gross margin. |
| | What are the challenges with your current approach and tool? | **Current: Manual Data Delivery** → Push-based reporting; 70% of client onboarding time (12 weeks) spent setting up entitlements; ERR for non-real-time data (Bloomberg reports 5-15min delay acceptable, but Aladdin portfolio changes require <100ms). **Challenge**: Decoupled systems (Aladdin ≠ Bloomberg ≠ Salesforce) create reconciliation overhead. |
| | Would your business benefit from managing fraud detection, predictive maintenance, and root-cause analysis? | **Fraud Detection**: Monitor institutional client trading patterns for market manipulation (SEC requirement). **Predictive Maintenance**: Proactively fix portfolio calc errors before clients notice. **RCA**: When a client reports "My NAV dropped 2% overnight," root-cause analysis takes 6 hours; should be <30 min. |
| | How are you continually innovating on behalf of your customers? | **Innovation Roadmap**: Tier 5 AI-powered analytics ("Ask Nomura" chatbot) → Suitability analysis from 3 days → 3 minutes. Tier 6 multi-asset consolidation → Unified queries across equities, fixed income, derivatives. |

#### 1.2 Business Value Output: Data Impact Map

| Data Asset | Current State | Future State (12-18 months) | Revenue Impact | Risk Mitigation |
|------------|---------------|---------------------------|-----------------|-----------------|
| **Aladdin Portfolio Data** | Real-time CDC (3 MSK brokers, 10K events/sec) | Multi-region active-active failover; SLA 99.99% | +$825K from Tier 4 (validated, Citadel + AQR clients) | Single vendor risk: Aladdin represents 95% of real-time data; mitigation: Kafka circuit breaker + S3 cache |
| **Bloomberg Market Data** | $5M annual subscription; push reports 1-2x/day | ADX zero-copy model; clients query on-demand; ADX subscription cost $0 (Bloomberg pays Nomura tier arrangement) | +$200K from increased client engagement (lower friction = more queries) | Data freshness: 15-min lag in ADX vs. <1sec from CDC; clients accept tradeoff for zero-copy |
| **FactSet Fundamentals** | Batch REST API pulls every 4 hours | Real-time fallback via ADX; enrich portfolio with ESG + alternative factors monthly | +$150K from compliance reporting automation | FactSet API dependency: Implement circuit breaker + 24hr local cache |
| **Internal Risk Models** | Siloed in Python notebooks; non-reusable | Productionized as Lambda functions; available via API; row-level security via Lake Formation | +$500K from internal suitability analysis efficiency (3 days → 3 hours) | Model drift: Implement continuous retraining pipeline; quarterly audit |
| **Sales/Client Metadata** | Salesforce silos; manual reconciliation | Synchronized identity mesh (Cognito) + Lake Formation RBAC | +$300K from reduced manual data provisioning overhead | Client segmentation: Implement governance layer to prevent over-sharing |

---

### Step 2: Identify Your Data Consumers

**Objective**: Interview stakeholders to understand who needs data, in what modality, and at what latency.

#### 2.1 Data Consumer Personas for Nomura Asset Management

| Consumer Persona | Role | Data Needs | Consumption Model | Latency SLA | Tools | Peak Concurrency |
|-----------------|------|-----------|-------------------|-------------|-------|-----------------|
| **Portfolio Manager** | Internal decision-maker | Current NAV, Greeks, concentration risk, liquidity analysis | **Real-time dashboard** (React/Amplify) + API for programmatic access | <1 second (intraday decision support) | Nomura web portal; Excel via API | ~200 concurrent in market hours |
| **Institutional Client (Asset Allocator)** | External, 100+ portfolio managers | Multi-fund performance attribution; peer comparison | **REST API** (JSON) + periodic batch download (Parquet) | 1 hour (daily rebalancing) | Their own BI tools (Tableau, PowerBI) | ~50 concurrent per client |
| **Risk Officer (Compliance)** | Internal regulator | VaR, concentration, GIPS performance, FINRA CAT transactions | **Batch report** (PDF + CSV download); audit trail immutable log | End-of-day (overnight batch; 10 PM UTC delivery deadline) | Salesforce Service Cloud; Lake Formation SQL editor | &lt;10 users, infrequent |
| **Suitability Analyst** | Sales support team | Fund matching scores, performance comparisons, fee waiver requests | **REST API** (structured queries); optional AI chatbot ("Show me tech-heavy funds with <12% vol") | 15 minutes (client meetings pre-prepared) | Salesforce Sales Cloud; Nomura internal chatbot | ~50 concurrent during business hours |
| **Data Scientist** | Internal, model development | Raw event logs; full Aladdin CDC stream; market microstructure; alternative data | **Kafka topics** (schema-governed) + S3 Parquet files (historical, immutable) | Near-real-time (Kafka lags <5sec); historical: on-demand | Jupyter notebooks; Databricks; custom Python | ~5-10 simultaneous model training jobs |
| **Regulatory Auditor** | External, FINRA/SEC validator | Complete transaction log with audit trail; data provenance; change history | **Data Export API** (immutable, signed) + read-only Athena access; 7-year retention | On-demand (audit queries 1-2x/quarter) | Secure FTP; Athena CLI; AWS S3 signed URLs | Bursty: 1-2 users per audit (2-3 audits/year) |

#### 2.2 Consumption Model Deep Dive

For each consumer, define the **interaction pattern**, **infrastructure**, and **latency budget**:

```
┌─────────────────────────────────────────────────────────────────┐
│         NOMURA MULTI-MODAL DATA CONSUMPTION ARCHITECTURE        │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  CONSUMER: Portfolio Manager (Internal)                         │
│  ├─ Modality 1: Web Dashboard (React/Amplify)                  │
│  │  └─ Infrastructure: CloudFront → API Gateway → Lambda       │
│  │  └─ Latency: <1 sec (p95)                                   │
│  │  └─ Authentication: Cognito SSO (OIDC)                      │
│  │  └─ Data Source: Athena (real-time views) → in-memory cache │
│  │                                                               │
│  ├─ Modality 2: REST API (Programmatic)                        │
│  │  └─ Endpoint: POST /v2/portfolios/{id}/analytics            │
│  │  └─ Response: JSON + caching: ETag-based                    │
│  │  └─ Rate limit: 1000 req/min per user (burst 5000)         │
│  │  └─ Circuit breaker: If >100ms latency, return cached       │
│  │                                                               │
│  CONSUMER: Institutional Client (External)                      │
│  ├─ Modality 1: REST API (Query model)                         │
│  │  └─ Endpoint: GET /v1/funds/{id}/performance                │
│  │  └─ Response format: JSON; accept gzip compression          │
│  │  └─ Rate limit: 100 req/min (tier-based pricing)           │
│  │  └─ Authentication: OAuth2 (client credentials)             │
│  │  └─ Latency SLA: 1 hour (cached every 1 hour)              │
│  │                                                               │
│  ├─ Modality 2: Batch Download                                 │
│  │  └─ Frequency: Daily 10 PM UTC (after market close)        │
│  │  └─ Format: Parquet (schema-versioned)                      │
│  │  └─ Transport: S3 signed URL (24hr expiry) + SFTP           │
│  │  └─ Size: ~500MB per client per day                         │
│  │                                                               │
│  CONSUMER: Data Scientist (Internal)                            │
│  ├─ Modality 1: Kafka Topics (Streaming)                       │
│  │  └─ Topics: aladdin.trades, aladdin.positions, market.ticks │
│  │  └─ Consumer group: data-science-team                       │
│  │  └─ Latency: <5 sec lag behind source                      │
│  │  └─ Retention: 30 days (configurable per topic)            │
│  │                                                               │
│  ├─ Modality 2: S3 Data Lake (Batch)                           │
│  │  └─ Location: s3://nomura-datalake/gold/analytics/          │
│  │  └─ Format: Parquet (1-day partitions)                      │
│  │  └─ Access: IAM role-based (via Jupyter notebook kernel)    │
│  │  └─ Query engine: Athena (SQL) or Spark (PySpark)           │
│  │                                                               │
│  └─ Modality 3: Databricks Workspace                            │
│     └─ Pre-loaded: 60 days Aladdin + 365 days Bloomberg       │
│     └─ Compute: Auto-scaling clusters (0-20 workers)           │
│     └─ Lineage tracking: Delta Lake format (.DELTA protocol)   │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
```

---

### Step 3: Identify Your Data Sources

**Objective**: Research internal and external data origins, categorizing by type, source, and ingest mode.

#### 3.1 Data Source Inventory for Nomura Asset Management

| Category | Data Source | Type | Ingest Mode | Volume | Latency | Owner | Criticality |
|----------|-------------|------|------------|--------|---------|-------|------------|
| **Portfolio State** | Aladdin (BlackRock) | Semi-structured (JSON events) | Streaming (CDC Kafka) | 10K events/sec (market hours) | <100ms | Nomura Trading Team | **CRITICAL** |
| | Aladdin Position History | Structured (time-series) | Batch (daily 8 PM) | 50M records/day | 24 hours | Nomura Risk Team | HIGH |
| **Market Data** | Bloomberg Terminal Data | Semi-structured (tick data) | Streaming + Batch | 500K ticks/sec; daily files | <1 sec / 15 min | Bloomberg | CRITICAL |
| | FactSet Fundamentals | Structured (SQL) | Batch API (4hr intervals) | 50K securities × 200 metrics | 4 hours | FactSet Vendor | HIGH |
| | CME Futures (Tier 6 roadmap) | Structured (trade log) | Streaming | 10K contracts/sec | <1 sec | CME Data | MEDIUM |
| **Client Data** | Salesforce CRM | Structured (relational) | CDC Lambda (5 min intervals) | 2K accounts; 50K contacts | 5-10 min | Sales Team | HIGH |
| | Cognito Identity | Structured (user profiles) | Event-based (near real-time) | 5K users; 50K sessions/day | <1 sec | IT Security | CRITICAL |
| **Internal Models** | Risk Models (VaR, Greeks) | Semi-structured (model outputs) | Batch Lambda (hourly) | 5 GB/hour | Hourly | Risk Team | HIGH |
| | Performance Attribution | Structured (calculated) | Batch Spark (daily) | 100 GB/day | 24 hours | PM Office | HIGH |
| **Compliance** | Trading Transactions | Structured (audit log) | Streaming + Batch | 100K trades/day; append-only | <5 min / daily | Operations | CRITICAL |
| | Client Access Logs | Structured (event log) | Streaming | 10K events/min | <1 sec | IT Security | MEDIUM |

#### 3.2 Data Ingest Patterns by Modality

```yaml
Streaming (Real-time):
  - Aladdin CDC via Kafka: 10K events/sec
  - Bloomberg terminal ticks: 500K ticks/sec
  - Client portal access logs: 10K events/min
  - Infrastructure: AWS MSK (3 brokers); AsyncAPI schema governance
  - Consumer: Data scientists, portfolio managers (dashboards), risk engines

Micro-batch (5-15 minute intervals):
  - Salesforce CDC: Every 5 minutes (AppendLog extraction)
  - FactSet API polls: Every 4 hours (API rate limits)
  - Risk model updates: Every 1 hour (nightly batch)
  - Infrastructure: AWS Step Functions + Lambda; event-driven architecture
  - Consumer: Suitability analysts (near-real-time fund matching)

Batch (Daily/Weekly/Monthly):
  - Aladdin position history: Daily 8 PM (end-of-day after settlement)
  - Compliance exports: Daily 10 PM (regulatory deadline)
  - Performance reports: Daily 11 PM (due to clients by 7 AM next day)
  - Infrastructure: AWS Glue; scheduled via EventBridge
  - Consumer: External clients; regulatory bodies; internal analysts

Archive (Immutable long-term storage):
  - Audit log (7-year retention): S3 Glacier for years 5+
  - Client consent audit trail: DynamoDB + S3 backup
  - Infrastructure: S3 lifecycle policies (Standard → Intelligent-Tiering → Glacier)
```

---

### Step 4: Define Storage, Catalog, and Access Needs

**Objective**: Determine optimal data stores and catalog strategies for governance.

#### 4.1 Data Storage Architecture (Medallion Pattern)

```
┌──────────────────────────────────────────────────────────────────┐
│               NOMURA MEDALLION DATA ARCHITECTURE                  │
├──────────────────────────────────────────────────────────────────┤
│                                                                    │
│  BRONZE LAYER (Raw Ingestion)                                    │
│  ├─ Purpose: Store raw, ingested data (immutable)               │
│  ├─ Storage: S3 Bronze bucket; partitioned by date + source    │
│  ├─ Format: JSON (streaming), Parquet (batch)                  │
│  ├─ Examples:                                                    │
│  │   s3://nomura-bronze/aladdin/trades/yyyy-mm-dd/...         │
│  │   s3://nomura-bronze/bloomberg/ticks/yyyy-mm-dd/...        │
│  │   s3://nomura-bronze/salesforce/accounts/yyyy-mm-dd/...    │
│  ├─ Retention: 90 days (roll to Glacier for longer)           │
│  ├─ Access: Data engineers only (via IAM); pipeline automation │
│  └─ Storage cost: ~$0.001 per GB (standard S3)                │
│                                                                    │
│  SILVER LAYER (Cleaned & Deduplicated)                          │
│  ├─ Purpose: Curated, trusted data (business-ready)            │
│  ├─ Storage: S3 Silver bucket; schema-versioned (Delta Lake)  │
│  ├─ Format: Parquet + Delta transaction log                    │
│  ├─ Transformations applied:                                   │
│  │   - Deduplication (e.g., duplicate Aladdin trade events)   │
│  │   - Type casting (string → decimal for prices)             │
│  │   - Null handling (default values for missing fields)      │
│  │   - Fact/dimension separation (normalize referential data)  │
│  ├─ Examples:                                                   │
│  │   s3://nomura-silver/trades/yyyy-mm-dd/                    │
│  │   s3://nomura-silver/positions/yyyy-mm-dd/                 │
│  │   s3://nomura-silver/market_prices/yyyy-mm-dd/             │
│  ├─ Retention: 2 years (compliance requirement: FINRA CAT)    │
│  ├─ Access: Data engineers + analysts (Athena queries OK)     │
│  └─ Storage cost: ~$0.003 per GB (Intelligent-Tiering)        │
│                                                                    │
│  GOLD LAYER (Business Intelligence)                             │
│  ├─ Purpose: Fully aggregated, optimized for analytics        │
│  ├─ Storage: S3 Gold (analytics) + Redshift (operational)    │
│  ├─ Format: Parquet (aggregates); Redshift (materialized views)│
│  ├─ Examples:                                                   │
│  │   Daily NAV aggregates (by fund x security)                │
│  │   Performance YTD summaries (by fund x asset class)        │
│  │   Risk metrics dashboards (VaR, Greeks by portfolio)       │
│  │   Client-visible reports (curated by entitlements)         │
│  ├─ Retention: 5 years (archival for historical analysis)    │
│  ├─ Access: Portfolio managers, clients (via APIs/dashboards)│
│  └─ Compute cost: ~$3 per query (Athena) or ~$0.50/compute-unit/hr (Redshift) │
│                                                                    │
│  AUDIT LAYER (Compliance & Lineage)                             │
│  ├─ Purpose: Immutable audit trail (who accessed what, when)  │
│  ├─ Storage: S3 Audit bucket + DynamoDB (for fast lookups)   │
│  ├─ Events tracked:                                            │
│  │   - Data access (via CloudTrail + S3 access logs)         │
│  │   - Transformation history (via Delta log)                │
│  │   - Client entitlement changes (via Cognito events)       │
│  │   - Regulatory requests (via manual logging)              │
│  ├─ Retention: 7 years (SEC/FINRA requirement)               │
│  └─ Access: Compliance team + external auditors (read-only)  │
│                                                                    │
└──────────────────────────────────────────────────────────────────┘
```

#### 4.2 Data Catalog & Governance

| Tool/Component | Purpose | Implementation |
|----------------|---------|-----------------|
| **AWS Glue Data Catalog** | Central metadata registry for all data assets | Crawlers auto-discover schemas from Bronze/Silver; Data Quality rules check completeness |
| **AWS Lake Formation** | Row-level security (RBAC) and column masking | Fact tables: FactSet financial data → masked for junior analysts; Dimension: Client data → row filter by client_id |
| **Collibra Data Governance** | Business ownership; data lineage visualization; stewardship | Aladdin trades → traced through Silver deduplication → to Gold NAV aggregates → to client-facing reports |
| **AWS DataBrew** | Data profiling and quality assessment | Profile: Check for null rates, duplicates, anomalies; Alert if NAV prices > 10% change overnight |
| **Delta Lake (via Apache Iceberg)** | ACID transactions + time-travel (audit trail) | Query historical Gold layer at any timestamp; reconstruct NAV as of 2026-01-15 close; enable rollback if transformation bug discovered |

---

### Step 5: Define Data Processing Requirements

**Objective**: Determine transformation, enrichment, and aggregation logic.

#### 5.1 Data Processing Pipeline Architecture

```
┌────────────────────────────────────────────────────────────────────┐
│          NOMURA DATA PROCESSING PIPELINE ARCHITECTURE              │
├────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  INGESTION LAYER                                                   │
│  ├─ Aladdin CDC → Kinesis Data Streams (real-time)               │
│  ├─ Bloomberg ticks → MSK Kafka topic (real-time)                │
│  ├─ Salesforce accounts → EventBridge Lambda (5-min lambda)      │
│  └─ FactSet API → Lambda (4-hour scheduled)                      │
│                                                                     │
│         ↓ (Bronze layer S3 ingestion)                             │
│                                                                     │
│  TRANSFORMATION LAYER (Step 1: Deduplicate & Type-Cast)          │
│  ├─ Tool: AWS Glue (Spark job: 30 min SLA)                       │
│  ├─ Trigger: Scheduled daily at 7 AM UTC (after market close)   │
│  ├─ Logic:                                                        │
│  │   1. Load Bronze s3://bronze/aladdin/trades/2026-02-18/*     │
│  │   2. Deduplicate on (trade_id, timestamp) with row_number()  │
│  │   3. Type-cast: price (string) → decimal(18,8)               │
│  │   4. Null handling: null portfolio → 'UNKNOWN'               │
│  │   5. Save to s3://silver/trades/2026-02-18/ (Parquet)       │
│  │   6. Update Glue Catalog schema version                       │
│  ├─ Success metric: 99.9% data completeness (no dropped events) │
│  └─ Failure handling: CloudWatch alert → manual review           │
│                                                                     │
│         ↓ (Silver layer S3 storage)                               │
│                                                                     │
│  TRANSFORMATION LAYER (Step 2: Enrichment & Aggregation)         │
│  ├─ Tool: Amazon EMR (Spark cluster: 5 min SLA)                 │
│  ├─ Trigger: Daily at 8 AM UTC                                  │
│  ├─ Logic:                                                        │
│  │   1. Load Silver trades + positions + prices                 │
│  │   2. Join: trades ⟕  market_prices on (security_id, date)   │
│  │   3. Enrich: Aladdin positions ⟕  FactSet fundamentals      │
│  │   4. Calculate metrics: NAV = ∑(quantity × price), Greek     │
│  │   5. Aggregate: By (fund_id, date) → Gold layer             │
│  │   6. Publish Kafka topic: analytics.daily_nav                │
│  ├─ Cluster: 5 x m5.xlarge (auto-scales to 20 for large joins) │
│  └─ Output: ~1 GB Gold aggregates + Kafka broadcast            │
│                                                                     │
│         ↓ (Gold layer S3 + Kafka publication)                    │
│                                                                     │
│  MATERIALIZATION LAYER (Real-Time Cache)                         │
│  ├─ Platform: Amazon Elasticache (Redis cluster)                │
│  ├─ Cache invalidation: On every Kafka analytics.daily_nav event│
│  ├─ TTL: 4 hours (data freshness = 1 hour + cache margin)      │
│  ├─ Use cases:                                                   │
│  │   - Portfolio dashboard: Fast lookups (p95 <50ms)            │
│  │   - Client API: Rate-limited queries hit cache first         │
│  │   - Mobile apps: Precalculated aggregates (zero latency)     │
│  └─ Fallback: If cache miss, query Athena (SLA upgrade: 5 sec) │
│                                                                     │
│  API LAYER (Data Consumption)                                     │
│  ├─ REST API: API Gateway → Lambda (query Athena + cache)       │
│  ├─ Kafka consumers: Data scientists subscribe to real-time     │
│  ├─ Dashboard: CloudFront → React frontend → Lambda resolver    │
│  ├─ Batch exports: S3 pre-signed URLs (24 hr expiry)           │
│  └─ Rate limiting: 1000 req/min per user (tier-based SLA)      │
│                                                                     │
└────────────────────────────────────────────────────────────────────┘
```

#### 5.2 Data Quality & Observability

| Dimension | Metric | Target SLA | Tool |
|-----------|--------|------------|------|
| **Completeness** | % of expected records in daily load | >99.5% | Glue data quality rules + CloudWatch |
| **Accuracy** | Price deltas vs. Bloomberg reference feed | <0.01% variance | Athena test queries + Datadog PagerDuty |
| **Timeliness** | Aladdin trade ingestion latency | <5 min from trade execution | Kafka lag monitoring |
| **Consistency** | Client NAV calculated via API ≈ batch report | 100% match | Daily reconciliation job (alerts if mismatch) |
| **Schema Validity** | All fields match Glue catalog (type, nullability) | 100% | Glue crawlers validate on ingest |

---

## Part 3: Multi-Modal Data Distribution Mesh Architecture

### 3.1 Core Architecture Pattern: 4-Tier Distribution Model

The multi-modal distribution mesh evolves from "push-based" reporting to a client-driven "pull" model where each consumer chooses their preferred integration channel.

#### 3.1.1 Tier 1: Digital Dashboards (Web UI)

**Use Case**: Portfolio managers need real-time visibility into AUM, performance, risk metrics.

**Architecture**:
```
Client Browser (React)
    ↓
CloudFront CDN (5 edge locations globally)
    ↓
API Gateway (rate limiting: 1000 req/min per user)
    ↓
Lambda resolver (200ms timeout threshold)
    ├─ Check Elasticache for aggregates (typical: cache hit rate 80%)
    ├─ If miss: query Athena (p95: 500ms; retry with fallback)
    └─ Return JSON response with ETag (browser caching enabled)
    ↓
React component renders (D3.js charts; WebSocket for real-time updates)
```

**Latency SLA**: <1 second (p95)  
**Throughput**: 200 concurrent users (peak: 500 during market open)  
**Authentication**: Cognito SSO (OIDC); session tokens (1-hour TTL)  
**Data Freshness**: 15-minute cache invalidation (after each Kafka analytics event)

#### 3.1.2 Tier 2: REST APIs (Programmatic Access)

**Use Case**: Institutional clients query fund performance, benchmarks, fund-fund correlations via their own BI tools.

**Architecture**:
```
Client REST Client (curl, Python requests, Postman)
    ↓
API Gateway (OAuth2 client credentials; JWT validation)
    ↓
Lambda (100ms timeout → fallback to cached response if exceeded)
    ├─ Query parameter validation (e.g., fund_id must be in client's entitlements)
    ├─ Row-level security applied via Lake Formation
    ├─ Result set limited to 10,000 rows (pagination required for larger sets)
    └─ Response: JSON + CSV export option
    ↓
Client downloads/processes (gzip compression enables 80% reduction in payload size)
```

**Latency SLA**: <5 seconds for single fund query; <30 seconds for 100-fund batch  
**Rate Limiting**: 100 req/min (tier 1); 1000 req/min (premium tier with $50K/yr contract)  
**Authentication**: OAuth2 client credentials (service-to-service)  
**Response Format**: JSON (default); XML, CSV available via Accept header

#### 3.1.3 Tier 3: Async Kafka Streams (Real-Time Events)

**Use Case**: Data scientists subscribe to live portfolio events; compliance teams consume immutable transaction audit logs.

**Architecture**:
```
Kafka Broker (AWS MSK, 3 brokers across 3 AZs)
    ├─ Topic: aladdin.trades (10 partitions, retention: 30 days)
    ├─ Topic: aladdin.positions (5 partitions, retention: 90 days)
    ├─ Topic: analytics.daily_nav (partitioned by fund_id)
    └─ Topic: audit.client_access (append-only, 7-year retention → Glacier)
    ↓
Consumer Groups (Fan-out to multiple subscribers):
    ├─ Consumer: Data Science Lab (real-time model training; lag <5sec)
    ├─ Consumer: Risk Dashboard (live Greeks calculation; lag <10sec)
    ├─ Consumer: Compliance Audit Log (backup to S3; lag <1min)
    └─ Consumer: External Partners (via AWS EventBridge → SFTP delivery)
    ↓
Consumer Applications:
    ├─ Databricks Structured Streaming (PySpark; micro-batches every 30sec)
    ├─ Lambda functions (serverless; trigger on Kafka RecordBatch event)
    └─ Custom Java consumers (Apache KafkaConsumer API)
```

**Latency SLA**: <5 seconds for data scientists; <30 seconds for external partners  
**Throughput**: 10K events/sec (Aladdin trades); auto-scaling brokers if load exceeds 15K/sec  
**Retention**: Multi-tier (30d hot, 90d warm, 7yr cold via S3 Glacier)  
**Schema Governance**: AsyncAPI specifications; schema registry (Confluent Schema Registry)

#### 3.1.4 Tier 4: Batch Downloads (SFTP/S3)

**Use Case**: Institutional clients receive daily reconciliation files (fund holdings, performance feeds).

**Architecture**:
```
Scheduled Glue Job (EventBridge trigger daily at 10 PM UTC)
    ↓
Glue Script (Python Spark):
    1. Query Gold layer S3: s3://nomura-gold/daily_nav/2026-02-18/*
    2. Filter by client entitlements (Lake Formation row-level security)
    3. Format: Parquet → CSV/Excel for non-technical clients
    4. Compress: gzip (reduces 500MB → 50MB)
    5. Sign URL: S3 presigned URL generation (24-hour expiry)
    6. Deliver: Email presigned link OR SFTP push (OpenSSL encrypted channel)
    ↓
Client Downloads:
    ├─ Via HTTP: Click link in email → S3 direct download (CloudFront cache)
    ├─ Via SFTP: Automated daily pull from SFTP server (sftp.nomura.com:2222)
    └─ Via API: Trigger on-demand export via REST POST /exports/{file_id}
```

**Latency SLA**: Daily 11 PM UTC delivery; 99.5% on-time SLA  
**File Format**: Parquet (schema versioning); CSV (human-readable); Excel (pivot-friendly)  
**Size**: 50-500 MB per client per day (compression: 90% reduction)  
**Security**: PGP encryption; SFTP TLS 1.2; presigned URLs (signed by AWS SigV4)

---

### 3.2 Multi-Modal Choreography: Real-Time vs. Batch Trade-Offs

| Scenario | Modality | Latency | Infrastructure | Cost | Client Experience |
|----------|----------|---------|----------------|------|-------------------|
| **Portfolio Manager (Internal) - "What is my current NAV?"** | Tier 1 (Dashboard) | <1 sec | Elasticache + Athena | $500/mo | Real-time widgets on screen |
| | Fallback (API) | <5 sec | Lambda + S3 | $800/mo | Manual REST query for validation |
| **Suitability Analyst - "Match funds for this client"** | Tier 2 (REST API) | <5 sec | Lambda + Lake Formation | $200/mo | Salesforce workflow embedded |
| | Fallback (Batch) | <1 hour | Glue + S3 presigned URL | $50/mo | Weekly batch file for reference |
| **Data Scientist - "Stream live trades for backtesting"** | Tier 3 (Kafka) | <5 sec | MSK (3 brokers) | $2000/mo | Real-time model refinement ✅ |
| | Fallback (Batch) | <24 hr | Glue + Glacier | $100/mo | Daily file for historical analysis |
| **External Client - "Send me daily reconciliation"** | Tier 4 (Batch) | <1 min | Glue + SFTP | $300/mo | Automated email + SFTP delivery ✅ |
| | Fallback (API) | <5 sec | Lambda + Lake Formation | $200/mo | On-demand manual query via portal |

---

## Part 4: Real-Time Latency Requirements by Use Case

### 4.1 Latency Taxonomy for Asset Management

| Use Case | Primary Consumer | Latency Requirement | Rationale | Consequences of Breach |
|----------|-----------------|---------------------|-----------|------------------------|
| **Portfolio Rebalancing** | Fund Manager | <100ms (intraday decisions) | Market microstructure: prices move 0.1% in 100ms; stale data = execution slippage | $50K-$500K loss per missed rebalance opportunity |
| **Risk Monitoring (Greeks)** | Risk Officer | <1 sec (intra-hour monitoring) | Volatility spikes (e.g., market open) persist for minutes; early detection = hedge positioning | 2% portfolio loss if Greeks lag by >5 min |
| **Client Suitability** | Suitability Analyst | <15 min (pre-meeting prep) | Client calls scheduled; analyst needs analysis 30 min before; 15 min margin for manual verification | Call delay; poor positioning; customer churn |
| **Compliance Reporting** | Compliance Officer | <24 hr (end-of-day) | FINRA CAT reporting deadline: midnight UTC; no regulators check intraday | $10K+ daily fine per SEC/FINRA rule violation |
| **Performance Attribution** | Portfolio Manager | <1 hr (post-market) | After market close, PM wants to understand daily P&L before next day trading; attribution takes 30 min compute | Delayed strategy adjustments; next-day trades based on stale data |
| **Batch Client Reports** | Institutional Clients | <4 hr (delivered by 7 AM next calendar day) | Clients in APAC wake up 7 AM UTC (9 PM prior evening in New York); need reports for 9 AM meeting | Client complaints; competitive disadvantage (Bloomberg reports available 4 AM UTC) |
| **AI Analytics Chatbot** | Suitability Analyst | <3 min (query execution) | "Show me tech-heavy funds with <12% vol" must execute within time analyst waits | Customer frustration; requests sent to human analysts; operational overhead |
| **Live Client Dashboard** | Institutional Client | <5 sec (web interaction) | Client opens portal to check fund performance; modern web UX expects <3sec response | Portal abandonment; calls support team; operational cost |
| **Kafka Stream (Model Training)** | Data Scientist | <5 sec (stream lag) | Databricks job processes stream micro-batches every 30sec; if lag >5sec, model feedback loop breaks | Model staleness; backtests outperform production (data snooping bias) |

### 4.2 Latency Budget Allocation (Critical Path Analysis)

**Example: Portfolio Manager Real-Time Dashboard Update**

```
Total Latency Budget: <1 second (p95)

┌─────────────────────────────────────────────────────────────┐
│ Component                    │ Target   │ Allocation │ Risk  │
├──────────────────────────────┼──────────┼────────────┼───────┤
│ Kafka → Analytics topic      │ <10ms    │ 1%         │ LOW   │
│ (Aladdin → MSK publication)  │          │            │       │
│                              │          │            │       │
│ Spark aggregation (EMR)      │ <100ms   │ 10%        │ MED   │
│ (Gold layer computation)     │          │            │       │
│                              │          │            │       │
│ Elasticache cache load       │ <50ms    │ 5%         │ LOW   │
│ (Redis GET operation)        │          │            │       │
│                              │          │            │       │
│ Network: Client → Lambda     │ <100ms   │ 10%        │ MED   │
│ (CloudFront + API Gateway)   │          │            │       │
│                              │          │            │       │
│ Lambda cold start (if needed)│ <200ms   │ 20%        │ HIGH  │
│ (container init + handler)   │          │            │       │
│                              │          │            │       │
│ Athena query (if cache miss) │ <500ms   │ 50%        │ HIGH  │
│ (S3 scan + SQL execution)    │          │            │       │
│                              │          │            │       │
│ Browser rendering (React)    │ <50ms    │ 5%         │ LOW   │
│ (WebSocket frame paint)      │          │            │       │
│                              │          │            │       │
│ TOTAL (typical path)         │ <250ms   │ ✓ MEETS SLA│       │
│ WORST CASE (all fallback)    │ <900ms   │ P95 target │       │
└─────────────────────────────────────────────────────────────┘

Optimization Tactics:
1. Keep cache hit rate >85% (Elasticache TTL: 15 min refresh cycles)
2. Pre-warm Lambda containers (reserved concurrency: 50 instances)
3. Use CloudFront edge caching (vary by user_id + fund_id)
4. Partition Athena queries by date + fund_id (partition pruning)
5. Implement circuit breaker: if Athena >300ms, return stale cache (p95 <1sec guaranteed)
```

---

## Part 5: Data Discovery Governance Checklist

### 5.1 Pre-Architecture Approval Checklist

Before deploying any data pipeline, answer these discovery questions:

- [ ] **Business Value**: Have we quantified revenue uplift or cost savings? (e.g., +$825K from Tier 4 onboarding)
- [ ] **Consumer Identification**: Have we interviewed all stakeholder personas? (portfolio managers, analysts, compliance, external clients)
- [ ] **Source Inventory**: Have we cataloged all 24+ data sources with ownership, latency, volume, format documented?
- [ ] **SLA Definition**: Have we defined latency, throughput, and availability SLAs per consumer? (e.g., <1 sec dashboard, <24hr batch reports)
- [ ] **Storage Strategy**: Have we selected medallion (Bronze/Silver/Gold) partitioning schemes that enable pruning and cost optimization?
- [ ] **Access Control**: Have we implemented row-level security (Lake Formation), column masking (PII), and audit trails?
- [ ] **Test Coverage**: Have we load-tested to peak concurrency (200 concurrent portfolio managers; 10K Kafka events/sec)?
- [ ] **Disaster Recovery**: Have we documented RTO <4hr, RPO <1hr backup/restore procedures?
- [ ] **Compliance**: Have we mapped data lineage to GIPS/FINRA/GDPR requirements and 7-year retention needs?

---

## Conclusion: Data Discovery as Continuous Practice

Data discovery is not a one-time workshop; it's a **continuous feedback loop**. As markets shift, clients evolve, and Nomura's product roadmap advances:

1. **Monthly reviews**: Assess emerging data sources (e.g., alternative data providers)
2. **Quarterly planning**: Refresh consumer needs (e.g., AI analytics demand vs. traditional dashboards)
3. **Annual roadmapping**: Expand multi-modal distribution (e.g., Tier 5-8 expansion to blockchain, real-time regulatory reporting)

By embedding data discovery into your architecture governance, you transform from a **services company** (custom reports, 2-week turnaround) to a **product company** (self-serve analytics, 1-hour turnaround, $2M SaaS ARR).

---

**Document Owner**: Principal Data Architect | Nomura Asset Management  
**Last Updated**: February 18, 2026  
**Next Review**: May 18, 2026
