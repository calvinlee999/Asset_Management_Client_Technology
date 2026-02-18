# Asset Management Client Technology - Architecture Overview

## Executive Summary

This repository documents the comprehensive technology architecture for an Assessment Management FinTech company's **Client Technology and Integration** domain. It represents a strategic shift from traditional push-based reporting to a multi-modal, data-driven distribution mesh that enables institutional clients (hedge funds, pension funds, asset managers) to consume proprietary insights through their preferred integration channels.

**Mission**: Enable 60% reduction in client onboarding time for data delivery while maintaining 100% automated entitlement synchronization across platforms.

---

## 1. Strategic Context

### Business Problem
Asset management firms face increasing pressure to:
- **Accelerate client adoption** of proprietary data and insights
- **Reduce operational overhead** in manual data delivery and entitlement management
- **Enable omnichannel access** (Web, API, Streaming, Marketplace)
- **Ensure security and scalability** across thousands of institutional clients

### Role & Mandate
As the senior engineering leader for Client Technology and Integration, you are accountable for:
- **End-to-end delivery** (requirements → architecture → development → release → support)
- **Strategic roadmap development** aligned with KPIs and business partner needs
- **Architecture governance** ensuring patterns, security, run costs, and UX excellence
- **Team leadership** of principal engineers, architects, and software engineering managers
- **Digitalization & automation** initiatives with business partners (Sales, Marketing, Distribution)
- **Vendor consolidation** through unified data mesh (Aladdin, Bloomberg, FactSet)

### Reference Documentation

**For Infrastructure & Cloud Platform Architecture**, see [CLOUD_PLATFORM_ARCHITECTURE.md](CLOUD_PLATFORM_ARCHITECTURE.md) for Nomura's "New Cloud" Platform with Terraform IaC, GitOps (FluxCD), LTGM observability stack, chaos engineering, and 12-month execution roadmap.

**For Advanced AWS Data Exchange Marketplace Tier**, see [AWS_DATA_EXCHANGE_MARKETPLACE_TIER.md](AWS_DATA_EXCHANGE_MARKETPLACE_TIER.md) for Levin-era zero-ETL patterns, multi-region disaster recovery, and production deployment procedures.

---

## 1.5 The Integrated Investment Data Mesh

### Business Context: Modern Asset Management Vendor Consolidation

Modern firms face siloed system complexity:
- **Aladdin (BlackRock)**: Real-time portfolio system; every trade must be captured <100ms
- **Bloomberg & FactSet**: Market data subscriptions; can tolerate 5-15min delay
- **Internal Systems**: Salesforce, risk engines, compliance (fragmented)

**Problem**: Manual reconciliation between Aladdin (internal state) + Bloomberg (market prices) + risk engines (exposure) creates delays and errors.

**Solution**: Unified data mesh via AWS where Aladdin CDC feeds real-time events, Bloomberg/FactSet publish via ADX zero-copy, and internal systems query unified layer.

---

## 1.6 Strategic Talking Points for Shaw Levin

Given Shaw's background as AWS Data Exchange architect + FinOps/SRE engineer, focus on these three "high-bar" dimensions:

### A. FinOps Challenge: Cost as First-Class Citizen

**Talking Point**:
> "With petabyte-scale data ingest from Aladdin, Bloomberg, and FactSet, 'Run Cost' can't be afterthought. I've implemented AWS Cost Categories: Aladdin CDC ($45K/mo), Bloomberg ADX ($5K/mo zero-egress), internal storage ($1K/day). Total: $290K/year. Compare to legacy push model: $2M/year (egress charges for 1000 clients). **85% cost reduction from architecture choice, not just AWS discounts.**"

**Numbers to memorize**:
- Aladdin CDC: 3 MSK brokers, 10K events/sec during market hours
- Bloomberg: $5M annual subscription → now $5K/mo storage (clients pay query compute)
- Internal storage: 50TB medallion lake at $1K/day AWS storage charges
- Total: $290K/year = $24K/month operational expense

### B. SRE & Resilience: Chaos Engineering for Vendor Dependencies

**Talking Point**:
> "Integrating three external vendors creates complex failure modes. If Aladdin CDC lags (which happens!), portfolio analytics degrade. If Bloomberg ADX subscription fails, clients can't query market data. I've implemented resilience patterns: (1) Circuit Breaker for Kafka producer—if Aladdin CDC API exceeds 30sec latency, we buffer events locally and retry; (2) Bulkhead isolation—Aladdin consumer runs on dedicated thread pool, never starving REST API handlers; (3) Automatic failover—if ADX becomes unavailable, switch to Athena cached copy (15min old but better than 500 error). **Result: Service survives any single vendor outage with acceptable degradation.**"

**Resilience Scenarios to Prepare**:
1. Aladdin CDC lags >30sec → circuit breaker engages → buffer locally → retry with exponential backoff
2. Bloomberg ADX endpoint returns 503 → fallback to Athena cache (via Lambda retry) → notify clients "data 15min delayed"
3. FactSet Redshift cluster unavailable → queries rerouted to backup read replica in us-west-2 → RTO <5min

### C. Data-as-a-Product: Internal Zero-Copy Sharing

**Talking Point**:
> "You built AWS Data Exchange—you know the value of zero-copy data sharing. I want to apply that internally: treat our proprietary alpha (portfolio analytics, risk models) as a 'Data Product' for internal teams. Using AWS Lake Formation row-level security, Risk/Sales/Trading engineers get instant read-only access. No data copies, no security reviews, no API latency. Within 24 hours of joining, a Sales engineer can self-serve: 'Show me funds with >15% tech exposure AND <12% volatility' without touching our APIs. **Business impact: Suitability analysis time drops from 3 days (manual data pulls) to 3 hours (self-service query).**"

**Internal Data Products to Demonstrate**:
1. Portfolio Analytics: Current NAV, performance YTD, Greeks by asset class
2. Risk Dashboard: VaR, concentration risk, liquidity analysis
3. Compliance Reports: GIPS performance, FINRA CAT transaction log
4. Sales Enablement: Fund match scores, performance comparisons, fee models

---

## 1.7 Proactive Actions: Risk Mitigation & Strategic KPIs

### How We "Squeeze the Lemon": Convert Risks into Opportunities

| Risk Area | Proactive Mitigation | Implementation | Strategic KPI | Evidence of Success |
|-----------|--------|-----------------|-------------|----------------------|
| **Vendor Lock-In (Aladdin + ADX combo)** | Export to open formats quarterly; 90-day contract exit clause | Quarterly Glue jobs export Parquet + Iceberg; contract specifies data portability | Can migrate to competitor in 90 days | Successful dry-run migration to S3 + Snowflake (no code changes needed) |
| **Data Silos** | Centralized Identity Mesh reconciling Aladdin/Bloomberg/Salesforce IDs | Lambda job syncs portfolio IDs across systems nightly; DynamoDB lookup table | Single SQL query joins Aladdin + Bloomberg + internal data | Sales query: "All funds with >$100M AUM AND Bloomberg subscribe" completes in <1sec |
| **Security Gaps from external integrations** | Zero-Trust architecture for all vendor connections | PrivateLink for Aladdin (no internet), IAM roles for Bloomberg ADX (no creds passed), encrypted channels | SOC2 Type II audit score: 98%+ | Zero breaches; zero credential leaks in logs; quarterly penetration test passes |
| **Vendor API Outages** | Multi-source fallbacks with graceful degradation | Circuit breaker: if CDC lag >30sec, buffer locally; if ADX fails, query Athena snapshot; if FactSet 503, retry | RTO <4 hours; RPO <1 hour | Incident: ADX down 2hrs → clients got cached data from 2hrs ago → service kept running |
| **Cost Explosion from scaled data ingestion** | FinOps cost categories segregating by vendor + auto-alerts | Separate AWS tags for Aladdin MSK, Bloomberg storage, internal ETL; CloudWatch alert if >10% month-over-month increase | Cost per GB ingested <$0.001 | Running $290K/year vs. $350K/year budget (20% underspend) |
| **Performance Degradation under load** | Bulkhead thread pool isolation + circuit breaker | Aladdin consumer: dedicated 20-thread pool (never starves API); API thread pool: separate 100-thread pool | API p95 latency <500ms even during peak Aladdin ingestion; p99 <1s | Load test: 10K Aladdin events/sec ingesting simultaneously with 5K API requests/sec → API stays <300ms |

---

## 1.8 Salesforce Ecosystem & Strategic Integration Blueprint

### The CRM-to-Insights Integration Imperative

For the Head of Client Technology role at Nomura, Salesforce is **not just a CRM**; it is the **Engagement Orchestration Layer** of the distribution engine. To align with the AWS-first strategy and the high-value needs of Asset Management (AM), the architecture must shift from "Salesforce as a silo" to "Salesforce as a headless consumer of the Data Mesh."

### A. Domain Deep Dive: Sales Cloud vs. Service Cloud

#### Sales Cloud (The Growth Engine)
**Focus**: Institutional Sales & Distribution  
**Key AM Capability**: Institutional Pipeline Management (complex, long-cycle deals with sovereign wealth funds or pension funds)

**Advanced Features**:
- **Einstein Relationship Insights**: AI-driven mapping of connections between consultants, portfolio managers, and institutional leads
- **Account Teams & Sharing Models**: Critical for managing cross-border distribution (Japan vs. US teams collaborating on global clients)
- **Activity Capture**: Auto-logging emails/meetings ensures high data fidelity for sales leadership

#### Service Cloud (The Retention Engine)
**Focus**: Client Onboarding, Investor Relations, and Reporting Support  
**Key AM Capability**: Entitlement Management (authorizing institutional clients for specific private market data or premium research)

**Advanced Features**:
- **Case Management & Omni-Channel**: Routing complex queries ("Why did the NAV for Fund X dip?") to the right SME
- **Experience Cloud Integration**: Powering the secure client portal where institutional investors download monthly/quarterly reports

### B. Integration Blueprint: Salesforce in the AWS Ecosystem

**Data Layer—The "Zero-Copy" Strategy**:
- Use Salesforce Data Cloud (Data 360) to connect AWS S3 Gold Layer
- Salesforce "reads" live investment data (NAV, Attribution) directly via Athena connector
- No data duplication into Salesforce = cost-efficient + always fresh

**Process Layer—Event-Driven Orchestration**:
```
Salesforce Contract Signed (Sales Cloud)
  ↓
  → Outbound Message via EventBridge
  → AWS Step Functions workflow triggered
  → Validate client record
  → Lookup client AWS Account ID
  → Call AWS ADX API to grant dataset access
  → Update Salesforce contract status
  ↓
  SLA: <2 minutes from signature → ADX activation
```

**Enablement Layer—Content & Intelligence**:
- Seismic embedded in Salesforce: RM views lead → latest approved collateral surfaces automatically
- PowerBI embedded via LWC: RM opens account → sees client portfolio analytics without context switch
- Alerts: Client downloads report → Service Cloud case created for proactive follow-up

### C. Strategic "Proactive Actions" (Mitigating Lemons)

| Challenge | Mitigation | Strategic Value |
|-----------|---|---|
| **Data Fragmentation** | Data Cloud as \"Identity Mesh\" to unify IDs across AWS, Salesforce, Seismic | SSOT (Single Source of Truth) |
| **User Experience Friction** | Salesforce Lightning Console + Amazon Connect integration | Reduces screen-switching; RM calls handled from console |
| **Compliance Risk** | Salesforce Shield + AWS Lake Formation RLS | SOC2/FINRA audit compliance at field level |
| **Legacy Integration Debt** | MuleSoft on AWS bridges legacy banking systems | Shortens data integration from 6→2 months |

### D. Interview Talking Point

> \"At Nomura, Salesforce isn't just a CRM. It's the Engagement Orchestration Layer of our distribution engine. When an RM opens a client account, they see real-time portfolio analytics embedded directly in Salesforce—no context switching. Combined with Seismic for content delivery and Amazon Connect for omni-channel support, we've transformed the CRM into a **real-time insights engine**. Result: Deal velocity 2x faster, sales cycles compress from 90→45 days.\"

### E. Logical Architecture: The \"Three-Layer\" Integration (Identity Mesh)

**Layer 1: Presentation** - Power BI/QuickSight embedded directly into Salesforce Account pages via Lightning Web Components (LWC)

**Layer 2: Data** - Salesforce Data Cloud queries S3 Gold Layer via Athena (zero-copy integration); ensures RMs see same data as clients

**Layer 3: Identity** - OIDC/IAM federation; JWT tokens carry client_id metadata; row-level security enforced at Athena query level (Client A never sees Client B's data)

---

## 2. Logical Architecture: Nomura Front-to-Back Stack

Map each industry tool to its optimal AWS integration method:

| Domain | Industry Tool | Business Purpose | AWS Component | Integration Pattern | Latency SLA | Cost Model |
|--------|---|---|---|---|---|---|
| **Front Office** | Bloomberg | Real-time market pricing, volatility, news | AWS Data Exchange + Redshift Data Share | Zero-ETL (no copy) | 5-15 min (market hours) | Subscription included in ADX |
| **Front Office** | FactSet | Financial analytics, comp companies, research | AWS Data Exchange + Lake Formation RLS | Zero-copy SQL queries | On-demand | Subscription to FactSet |
| **Middle Office** | BlackRock Aladdin | Portfolio manager system of record; real-time trading | Amazon MSK (Kafka) + AppFlow CDC | Real-time Change Data Capture stream | <100ms (per trade) | Aladdin license + MSK storage |
| **Risk & Performance** | GIPS Compliance Suite | Performance reporting engine | AWS Step Functions + Vermillion API | Orchestrated report generation | End-of-day batch | Internal SLA |
| **Risk & Performance** | Custom Risk Engines | VaR, Greeks, PnL calculations | Lambda + Athena | Event-driven triggers on portfolio update | <5 min | Lambda + Athena storage |
| **Digital/Sales** | Salesforce | CRM, entitlements, contract management | AWS AppSync (GraphQL) + Lambda | Real-time sync via Outbound Messages | <1 second | Salesforce license |
| **Compliance** | Regulatory Repositories | CAT, CFR, audit trails | S3 Object Lock + CloudTrail | Immutable storage with versioning | Archived (7yr retention) | S3 storage ($23/TB/year) |

**Key Insight**: Each vendor has distinct latency + throughput + cost requirements. We don't force them through a single pipeline; instead, we match integration pattern to vendor SLA:
- **Aladdin**: Real-time CDC (must capture every trade <100ms)
- **Bloomberg/FactSet**: Zero-copy ADX (high cost if replicated; better to let client query)
- **GIPS**: Scheduled batch (end-of-day is sufficient for compliance reports)
- **Salesforce**: Event-driven (minimal latency for entitlement provisioning)

---

## 2.1 Multi-Modal Data Distribution Mesh Architecture

The core architectural pattern evolves from a "push-based" reporting model to a **4-Tier Distribution Mesh**, allowing clients to consume data via their preferred channel.

```
┌─────────────────────────────────────────────────────────────┐
│           NOMURA ASSET MANAGEMENT DATA SERVICES              │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌─────────────┐  ┌──────────────┐  ┌────────────┐          │
│  │   Digital   │  │    SaaS      │  │   Async    │          │
│  │ Dashboards  │  │   REST APIs  │  │   Kafka    │          │
│  └──────┬──────┘  └──────┬───────┘  └────┬───────┘          │
│         │                │               │                   │
│         └────────────────┼───────────────┴──────┐            │
│                          │                      │            │
│                          ▼                      ▼            │
│              ┌─────────────────────────────────────┐         │
│              │   Core Data Processing Layer        │         │
│              │   (Medallion Architecture)          │         │
│              │   Bronze → Silver → Gold            │         │
│              └──────────────┬──────────────────────┘         │
│                             │                                 │
│          ┌──────────────────┼──────────────────────┐         │
│          ▼                  ▼                      ▼         │
│   ┌────────────┐    ┌──────────────┐     ┌────────────┐   │
│   │ AWS Data   │    │ Data Product │     │ Audit &    │   │
│   │ Exchange   │    │ Marketplace  │     │ Compliance │   │
│   │ (ADX)      │    │              │     │ Layer      │   │
│   └────────────┘    └──────────────┘     └────────────┘   │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### 2.1 Tier 1: Digital Layer (Web/Mobile/Dashboards) — Comprehensive Architecture

**Purpose**: Real-time visualization and interactive analytics for Portfolio Managers and Relationship Managers with seamless Salesforce integration

**Technology Stack**:
- **Frontend**: React + TypeScript on AWS Amplify
- **Visualization**: PowerBI for self-service analytics + Amazon QuickSight for internal FinOps
- **Backend Compute**: Amazon Athena + S3 (Medallion Architecture)
- **Real-time Data**: QuickSight with SPICE caching for sub-second loads
- **Authentication**: AWS Cognito + SAML + Salesforce SSO
- **Embedding**: Lightning Web Components (LWC) for Salesforce UI integration
- **API**: GraphQL + REST via API Gateway

**Key Characteristics**:
- Low-latency queries (<5 sec for complex aggregations; <1 sec for cached)
- Multi-tenancy with RBAC enforced at Athena query level
- Real-time performance attribution via Kafka → materialized views
- Customizable dashboards per client (Power BI parameterized reports)
- Context-aware filtering (Account_ID from Salesforce → auto-filtered dashboard)

**Sample Data Products**:
- **Internal Operational Dashboards** (Amazon QuickSight):
  - NAV & Performance Analytics (real-time portfolio updates)
  - Risk Dashboard (VaR, Sharpe ratio, concentration risk)
  - FinOps Dashboard (AWS spend by domain: Aladdin vs. Bloomberg vs. internal)
  - SRE Observability (EKS cluster utilization, data pipeline latency, error rates)

- **Client-Facing Dashboards** (Power BI Embedded):
  - Current NAV + 1-/3-/5-year performance vs. benchmarks
  - Attribution breakdown by fund manager / strategy / geography
  - Exposure analysis (equity %, fixed income %, alternatives %)
  - Fee transparency and cost-benefit analysis
  - Risk simulations (\"What if\" portfolio stress tests)

**Architecture: The \"Three-Layer\" Integration (Identity Mesh)**

#### Layer 1: Presentation (The \"Single Pane of Glass\")
**Pattern**: LWC Embedding in Salesforce  
Power BI/QuickSight dashboards embedded directly into Salesforce Account or Opportunity pages

```typescript
// Lightning Web Component (Salesforce)
import { LightningElement, api } from 'lwc';
export default class PowerBIEmbedded extends LightningElement {
  @api recordId; // Account_ID from Salesforce context
  embedUrl;
  connectedCallback() {
    // Pass Account_ID to PowerBI dashboard as filter
    this.embedUrl = `https://app.powerbi.com/view?r=eyJrIjoiYWJ...` +
      `&config=${btoa(JSON.stringify({
        \"filterPaneEnabled\": false,
        \"filters\": [{
          \"name\": \"Account_ID\",
          \"values\": [this.recordId]
        }]
      }))}`; // Client sees only their holdings
  }
}
```

When RM opens \"BlackRock\" account, dashboard auto-filters to show only BlackRock holdings, AUM, and performance.

#### Layer 2: Data (The \"Zero-Copy\" Mesh)
**Pattern**: Salesforce Data Cloud + AWS S3/Athena  
Salesforce \"reads\" live investment data directly from AWS S3 Gold Layer (no data duplication)

```sql
SELECT portfolio_id, nav_current, aum_total, perf_ytd, perf_1yr
FROM s3://nomura-data-lake/gold/portfolio_analytics/
WHERE profile_id = @account_context
  AND date = CURRENT_DATE
```

Both Salesforce and Power BI query same S3 → single source of truth, <1 min latency

#### Layer 3: Identity (The Secure Bridge)
**Pattern**: OIDC + Row-Level Security (RLS)  
JWT token carries user context; Athena enforces RLS at query level

```
RM logs into Salesforce via OKTA
  ↓
  Opens BlackRock account page
  ↓
  PowerBI LWC requests embed token from Lambda
  ↓
  Lambda validates: User_ID → Account ownership
  ↓
  JWT generated with claims {user_id, account_id, role}
  ↓
  PowerBI applies RLS: Filter to account_id in JWT
  ↓
  RM cannot see competitor data (APAC RM cannot see EMEA client data)
```

**Strategic Use Cases**

| Use Case | Workflow | Implementation | Outcome |
|----------|----------|---|---|
| **RM Alpha Discovery** | Identify under-allocated clients | PowerBI in SFDC finds clients with low fund exposure; auto-create task | 5 new prospects/day with hyper-personalized pitches |
| **Investor Portal Sync** | Client downloads report; RM notified | Client portal → EventBridge → SFDC case | Proactive outreach within 2 hours; NPS +5 |
| **FinOps Visibility** | Shaw Levin sees AWS spend | QuickSight feeds EKS metrics + Cost Explorer | 20% cost optimization monthly |
| **Distribution Analytics** | Measure pitch-to-close velocity | SFDC Opportunity + Seismic engagement in QuickSight | Sales cycle 90→45 days |

**Proactive Actions (The \"Lemons\" Table)**

| Lemon | Mitigation | Implementation | Impact |
|-------|-----------|---|---|
| **Data Latency** | Use SPICE or DirectQuery | SPICE 15-min refresh; DirectQuery for <1min freshness | <1 sec dashboard load (cached) vs. 5+ sec (uncached) |
| **High Licensing Costs** | Session-based pricing + embed | QuickSight readers $2/user/mo; Power BI embedded | 60% reduction in BI spend |
| **Fragmented UX** | Custom branding/themes | Nomura colors, logos, fonts in embed layer | User adoption +25%; NPS +10 |
| **Manual Provisioning** | AWS CloudFormation + Power BI APIs | terraform deploys dashboards | New dashboard live in <2 hours (vs. 3-5 days) |

---

### 2.2 Tier 2: SaaS Tier (REST APIs)

**Purpose**: Programmatic access for client developers integrating with their own systems

**Technology Stack**:
- **API Gateway**: AWS API Gateway (REST + GraphQL)
- **Compute**: AWS Lambda (serverless) with auto-scaling
- **Authentication**: OIDC/OAuth 2.0 + IAM roles
- **Rate Limiting**: Token bucket algorithm with per-client quotas
- **Caching**: Amazon ElastiCache (Redis) for frequently accessed endpoints

**Key Characteristics**:
- RESTful and GraphQL endpoints
- Request-Response synchronous pattern
- Secure credential management via AWS Secrets Manager
- Webhook support for event notifications
- API versioning and deprecation management

**Sample Endpoints**:
```
GET  /v2/funds/{fundId}/nav
GET  /v2/funds/{fundId}/performance?period=YTD
POST /v2/reports/generate
GET  /v2/entitlements/me
```

**SLA**: 99.99% uptime, p95 latency <200ms

---

### 2.3 Tier 3: Async Tier (Apache Kafka / Streaming)

**Purpose**: High-frequency institutional trading and real-time risk engines

**Technology Stack**:
- **Message Broker**: Amazon Managed Streaming for Apache Kafka (MSK)
- **Producers**: Custom ingestion services (Java/Python)
- **Consumers**: Client applications or data warehouses
- **Schema Registry**: Confluent Schema Registry
- **Monitoring**: Prometheus + Grafana

**Key Characteristics**:
- Topics act as domain contracts (immutable schemas)
- Event sourcing pattern for audit trail
- Multi-partition distribution for high throughput
- Consumer group management with offset commits
- Exactly-once semantics for critical flows

**Sample Kafka Topics**:
```
asset-management.nav.intraday          // Streaming NAV updates
asset-management.trades.execution      // Trade confirmations
asset-management.risk.alerts           // Risk threshold breaches
asset-management.performance.metrics   // Real-time performance metrics
```

**Throughput**: 100K+ events/second per topic

---

### 2.4 Tier 4: Marketplace Tier (AWS Data Exchange)

**Purpose**: Zero-copy data distribution for institutional data scientists and quant shops

**Technology Stack**:
- **Distribution Platform**: AWS Data Exchange (ADX)
- **Storage Backend**: S3 + Redshift
- **Billing Engine**: AWS ADX native billing
- **Entitlements**: Salesforce + AWS IAM sync
- **Data Catalog**: AWS Glue Data Catalog

**Key Characteristics**:
- Fully serverless, no infrastructure management
- Automatic billing and subscriber management
- Data products discoverable via AWS Marketplace
- Subscribers query data in their own AWS environment
- "Zero-copy" = no data egress charges for Nomura

**Data Products**:
```
Nomura ESG Insights                // Quarterly ESG scores
Nomura Private Credit Performance  // NAV and cash flow analysis
Nomura Hedge Fund Benchmarks       // Cross-fund performance rankings
Nomura Macro Signals               // Leading indicators
```

**Business Model**: Subscription-based with usage-based pricing

---

## 3. Comparison: AWS Data Exchange vs. Snowflake Data Exchange

Both enable zero-copy data sharing, but serve different architectural contexts:

### Quick Comparison Matrix

| Aspect | AWS Data Exchange | Snowflake Data Exchange |
|--------|-------------------|------------------------|
| **Architecture** | Metadata-only sharing via Redshift/S3/Glue | Live data sharing within Snowflake ecosystem |
| **Data Movement** | No duplication; consumers query provider's storage | No duplication; consumers query within Snowflake cloud |
| **Platform Lock-In** | AWS-only (S3, Redshift, Lambda, Athena) | Snowflake-only (but works on AWS/Azure/GCP clouds) |
| **Ideal Customer** | Enterprises with AWS data lake | Enterprises using Snowflake as primary warehouse |
| **Cost Model** | Providers: pay storage; Consumers: pay compute | Providers: pay storage; Consumers: pay warehouse compute |
| **Multi-Cloud** | AWS only | Seamless across AWS/Azure/GCP |
| **Setup Complexity** | Moderate (IAM roles, Redshift sharing, Lake Formation) | Simple (built-in ADX UI; share creation is click-and-share) |
| **Time-to-Launch** | 2 weeks (product creation) | 6+ weeks (licensing, data migration) |
| **Vendor Partner Ecosystem** | Bloomberg, FactSet, Yahoo Finance on ADX | Fewer fintech vendors compared to ADX |
| **Governance** | AWS Lake Formation RLS + IAM + resource policies | Snowflake role-based access + secure shares + tagging |
| **Use Case: Nomura** | ✅ **CHOSEN** - Existing S3/Redshift | ❌ Would require wholesale migration ($5M+) |

### Nomura's Decision: AWS Data Exchange

**Why ADX**:
1. **Existing investment**: S3 + Redshift + Glue already deployed; ADX adds ~5% effort
2. **Vendor distribution**: Bloomberg & FactSet already publish equity / derivatives data to AWS via ADX
3. **Cost advantage**: $290K/year vs. $600K+/year if we switched to Snowflake licensing model
4. **Technical alignment**: ADX subscribers (clients) pay only for compute (no separate warehouse licenses) → simpler finance model for Nomura
5. **Aladdin integration**: MSK Kafka + Redshift is native AWS; Snowflake would force different ingestion architecture

**Migration cost to Snowflake**:
- Data lake rip-and-replace: $3-5M
- Workflow rewriting: 6+ months engineering
- Aladdin CDC adapter: must rebuild for Snowflake native connectors
- Learning curve: Snowflake SQL quirks, Snowpark, Iceberg compatibility
- Result: Not worth it when AWS Data Exchange already works

**When Snowflake might be compelling**:
- If Nomura already heavily invested in Snowflake (they're not)
- If we needed seamless multi-cloud (we're AWS-first strategy)
- If we needed simpler governance (Lake Formation RLS is comparable)

---

## 4. Core Integration Patterns

### 3.1 Entitlement Automation: Salesforce ↔ AWS Data Exchange

**Problem**: Manual ticket process delays data access provisioning by 2-5 days

**Solution**: Event-driven entitlement sync pipeline

```
┌──────────────────────────────────────────────────────────┐
│ Sales Executive closes deal in Salesforce Sales Cloud    │
│ (Contract = "Nomura ESG Insights" dataset)               │
└────────────────────────┬─────────────────────────────────┘
                         │
                         ▼
        ┌────────────────────────────────────┐
        │ Salesforce Change Data Capture     │
        │ (Outbound Messages)                │
        └────────────────┬───────────────────┘
                         │
                         ▼
        ┌────────────────────────────────────┐
        │ AWS Event Bridge                   │
        │ (Route to Step Functions)          │
        └────────────────┬───────────────────┘
                         │
                         ▼
        ┌────────────────────────────────────┐
        │ AWS Step Functions Orchestration   │
        │ 1. Validate client record          │
        │ 2. Lookup client AWS Account ID    │
        │ 3. Call ADX API to grant access    │
        │ 4. Send confirmation to Salesforce │
        └────────────────┬───────────────────┘
                         │
                         ▼
        ┌────────────────────────────────────┐
        │ Client AWS Account now has access  │
        │ to ESG Insights dataset            │
        │ (Zero manual intervention)         │
        └────────────────────────────────────┘
```

**Implementation Details**:
- **Latency**: <2 minutes from Sales Cloud click to active in ADX
- **Compliance**: Audit trail in CloudTrail + Salesforce
- **Rollback**: Automatic revocation upon contract termination
- **Error Handling**: Dead letter queue for manual intervention

---

### 3.2 Data Integrity & Quality Framework

#### The "Lemons" Problem
When data quality issues arise (stale NAV, missing performance data), clients experience friction and lose confidence.

#### Mitigation Strategy

| Issue | Mitigation | Implementation |
|-------|-----------|-----------------|
| **Data Staleness** | Real-time freshness monitoring | CloudWatch alarms if NAV not updated by 5PM |
| **Missing Values** | Null/NaN detection before publishing | AWS Glue data validation jobs |
| **Schema Drift** | Strict schema evolution governance | Confluent Schema Registry with BACKWARD compatibility |
| **PII Exposure** | Automated PII scanning | AWS Macie on all data products before ADX publish |
| **Cross-Fund Contamination** | Multi-tenancy isolation tests | Row-level security (RLS) in Athena + Redshift |

---

### 3.3 Run Cost Optimization

**Key Innovation: Shift from Nomura Pull → Client Push**

**Traditional Model** (Expensive):
- Nomura egresses data to thousands of clients (expensive AWS Data Transfer)
- Nomura pays for compute to transform + deliver
- Result: $500K+/year for large client bases

**New Model** (Cost-Efficient):
- Nomura publishes data once to S3 (Medallion Gold Layer)
- AWS ADX handles multi-tenancy at storage layer
- Clients query via their own compute (they pay for it)
- Result: Nomura pays only for master dataset storage (~$50K/year)

**Cost KPI**: 85% reduction in data delivery run costs

---

## 5. Technology Stack & Choices

### 4.1 Compute & Data

| Layer | Service | Reasoning |
|-------|---------|-----------|
| **Storage** | S3 + Data Lake | Unlimited scalability, cost-effective |
| **Data Warehouse** | Redshift Spectrum | Federated queries across S3 + structured data |
| **Real-Time Analytics** | Amazon Athena | Serverless SQL, automatic query optimization |
| **Streaming** | Amazon MSK | Managed Kafka, 99.99% SLA |
| **Data Catalog** | AWS Glue | Automated schema discovery, lineage tracking |
| **API Runtime** | Spring Boot 3.x on ECS/Lambda | Java ecosystem; Spring Cloud integration |
| **Service Mesh** | AWS App Mesh (optional) | Traffic management, observability |

### 4.2 Integration & APIs

| Layer | Technology | Standard |
|-------|-----------|----------|
| **API Protocol** | REST + GraphQL | Industry standard for integrations |
| **API Framework** | Spring Boot WebFlux | Reactive, non-blocking for high throughput |
| **Authentication** | OIDC / OAuth 2.0 + Spring Security | Zero-trust security model |
| **Event Streaming** | Kafka + CloudEvents | CNCF standard for events |
| **Kafka Clients** | Spring Cloud Stream + Kafka Streams | Java/JVM ecosystem |
| **Data Format** | Parquet + JSON | Columnar storage for analytics; JSON for APIs |
| **Observability** | Micrometer + Prometheus + CloudWatch | Standard JVM metrics collection |

### 4.3 Security & Compliance

| Domain | Implementation |
|--------|-----------------|
| **Identity & Access** | AWS IAM + SAML + MFA + Spring Security |
| **Data Encryption** | TLS 1.3 (transit), AES-256 (at rest) |
| **Secrets Management** | AWS Secrets Manager + Spring Cloud Config |
| **Audit Logging** | CloudTrail + ClamAV scanning + Spring Audit |
| **Compliance** | SOC 2 Type II, FINRA CAT reporting |
| **API Rate Limiting** | Token bucket + Spring Cloud Gateway | Prevent abuse and cascading failures |

### 4.4 Cost Estimation (Monthly Baseline)

**Assumptions**: 1000 API clients, 100K NAV updates/day, ADX: 50 subscribers

| Service | Volume | Unit Cost | Monthly Cost |
|---------|--------|-----------|-------------|
| **S3 (Data Lake)** | 50 TB | $0.023/GB | $1,150 |
| **Redshift** | 1 cluster (Ra3, 2 nodes) | $2.96/node/hr | $4,300 |
| **Lambda (APIs)** | 500M requests/mo | $0.20/1M | $100 |
| **MSK (Kafka)** | 3 brokers, 2.8TB/mo | $0.28/broker/hr + storage | $1,800 |
| **Athena (Queries)** | 5K queries/mo | $5/TB scanned | $250 |
| **AWS Data Exchange** | 50 subscribers | $0-100% share of margin | $5,000 |
| **NAT Gateway** | 1 TB/mo egress | $0.045/GB | $45 |
| **CloudWatch** | Logs + metrics | ~50GB ingest | $500 |
| **Development/Testing** | - | ~30% of production | $4,000 |
| **TOTAL** | - | - | **~$17,145/month (~$205K/year)** |

**Savings vs. Legacy (push-based model)**: $450K/year (69% reduction)

---

## 6. KPIs & Success Metrics

### 5.1 Operational KPIs

| KPI | Target | Current | Status |
|-----|--------|---------|--------|
| **Client Onboarding Time** | <24 hours | 3-5 days | 🎯 In Progress |
| **Entitlement Sync Automation** | 100% | 30% (manual) | 🎯 In Progress |
| **Data Freshness** | <5 min latency | 4 hours (batch) | 🎯 In Progress |
| **API Success Rate** | 99.99% | 99.2% | ⚠️ Needs Improvement |
| **Time to Market (new data product)** | <1 week | 4 weeks | 🎯 In Progress |

### 5.2 Business Impact KPIs

| KPI | Target | Benefit |
|-----|--------|---------|
| **Run Cost Reduction** | 85% | $450K annual savings |
| **Client Friction Score** | <2/10 | Improved NPS by 15 points |
| **Data Adoption Rate** | 80%+ | Increased sticky revenue (APIs/ADX) |
| **Marketplace Revenue** | $2M annual | New revenue stream |

---

## 7. Roadmap & Phasing

### Vendor Consolidation Timeline: From Legacy to Unified Mesh

#### Phase 1 (Q1 2026): Aladdin CDC Foundation (Months 1-3)
- ✅ Digital Layer: PowerBI dashboards (internal + first 5 pilot clients)
- ✅ SaaS Tier: REST APIs for NAV + Performance endpoints
- ⏳ Aladdin CDC: PrivateLink connection + Kafka ingest pipeline (CDC consumer pattern)
- ⏳ Real-time entitlement sync: Salesforce → Step Functions → granting portfolio access

**Aladdin-Specific Milestones**:
- Month 1: PrivateLink tunnel to Aladdin established; CDC events flowing
- Month 2: Kafka consumer stability; <100ms end-to-end latency verified
- Month 3: Gold layer portfolio snapshots queryable via Athena

**Team Hires for Phase 1**:
- Principal Architect (lead vision)
- Aladdin Integration Lead ($240K, L4) — owns CDC pipeline
- 2 Backend Engineers (MSK + Kafka patterns)
- DevOps Engineer ($180K, datacenter + monitoring)

**Business Impact**: 60% faster client onboarding for Aladdin-sourced data; real-time portfolio analytics

**Deliverable**: 50% of client base consuming Aladdin data via APIs; CDC latency <100ms

---

#### Phase 2 (Q2 2026): Bloomberg & FactSet Zero-Copy (Months 4-6)
- ✅ AWS Data Exchange product launch
- ✅ Redshift Data Share for FactSet analytics
- ✅ Automated entitlement sync (100% automation)
- ✅ GraphQL API layer (for complex queries spanning Aladdin + Bloomberg)

**Bloomberg/FactSet-Specific Milestones**:
- Month 4: ADX product listing + first 5 Bloomberg subscribers live
- Month 5: Redshift Data Share + RLS policies enforced
- Month 6: GraphQL queries joining Aladdin + Bloomberg data

**Team Hires for Phase 2**:
- Bloomberg Integration Lead ($240K, L4) — owns ADX product lifecycle
- FactSet Integration Lead ($240K, L4) — owns Redshift Data Share governance
- Data Quality Engineer ($200K) — validates cross-domain data consistency
- Security Engineer ($220K) — Lake Formation RLS policies + audit logging

**Business Impact**: 90% egress cost reduction (vs. legacy push model); zero-copy enables subscribers to scale without Nomura infrastructure cost

**Deliverable**: $1M ADX subscription revenue; <24hr client onboarding; 35% of clients now on zero-copy model

---

#### Phase 3 (Q3 2026): Intelligence & GIPS Automation (Months 7-9)
- ⏳ GIPS reporting automation (Step Functions → Vermillion)
- ⏳ AI-powered anomaly detection (Amazon Bedrock + Llama models for outlier flagging)
- ⏳ Predictive entitlement provisioning (ML: predict which clients will want which data)
- ⏳ Advanced analytics (consumption patterns, churn prediction, data affinity)

**GIPS/Compliance-Specific Milestones**:
- Month 7: GIPS reports auto-generated from Aladdin + Bloomberg + internal cost basis
- Month 8: Anomaly detection live (trade price vs. Bloomberg: flag >2% discrepancies)
- Month 9: Churn prediction model active (flag at-risk clients; trigger proactive outreach)

**Team Hires for Phase 3**:
- Analytics Lead ($200K) — owns GIPS automation
- ML Engineer ($210K) — owns anomaly detection models
- Solutions Architect ($220K) — customer success + feature requests
- QA Lead ($190K) — comprehensive testing

**Business Impact**: 100% GIPS compliance automation (was manual); 5-hour GIPS generation (was 2 days); anomaly detection catches data quality issues before clients see them

**Deliverable**: 80% data adoption; 85% cost reduction achieved; 5 new data products launched

---

### Vendor Timeline Heatmap

| Vendor | Phase 1 (M1-M3) | Phase 2 (M4-M6) | Phase 3 (M7-M9) | Status |
|--------|-----------------|-----------------|-----------------|--------|
| **Aladdin** | ✅ CDC live, <100ms latency | Enrichment + Redshift join | ML anomaly detection | Critical path |
| **Bloomberg** | Planning | ✅ ADX product live | FactSet + Bloomberg join | ADX dependent |
| **FactSet** | Research | ✅ Redshift Data Share | Analytics + GIPS join | Phase 2 follow-on |
| **Internal Systems** | Foundation (APIs, Kafka) | RLS + multi-tenancy | Governance + SSOT | Continuous |

---

## 8. Engineering Team Structure

### Domain-Based Organization

Instead of traditional "API team" + "Data team," we organize around **data domains**:

```
Senior Engineering Leader (You)
│
├── Aladdin Domain Lead (L4, Principal Engineer - $240K)
│   ├── CDC Pipeline Engineer ($180K)
│   ├── Kafka Specialist ($170K)
│   └── Performance Testing Engineer ($160K)
│   Owns: Real-time portfolio data, <100ms SLA, trade audit trail
│
├── Bloomberg/FactSet Domain Lead (L4, Principal Engineer - $240K)
│   ├── ADX Product Engineer ($200K)
│   ├── Redshift/Data Sharing Engineer ($190K)
│   └── Data Quality Engineer ($200K)
│   Owns: Zero-copy market data, subscriber lifecycle, governance
│
├── Internal Data Mesh Lead (L4, Principal Architect - $250K)
│   ├── Salesforce Integration Engineer ($170K)
│   ├── Identity Mesh Engineer ($180K)
│   └── Multi-Tenancy Engineer ($170K)
│   Owns: RBAC, RLS, entitlement lifecycle, internal data products
│
├── Platform & Reliability (L3, Principal SRE - $220K)
│   ├── Kubernetes + Monitoring Engineer ($180K)
│   ├── Disaster Recovery Engineer ($180K)
│   └── Security + Compliance Engineer ($220K)
│   Owns: Resilience, failover, incident response, SOC2 compliance
│
└── Engineering Manager (L3, $200K)
    ├── 4 Backend Engineers (Kafka, Spring Boot, L2-L3)
    ├── 3 Frontend Engineers (React, Dashboards, L2-L3)
    └── Admin & Hiring

**Total: 24 FTE, ~$3.8M annual opex (incl. overhead)**
```

---

### Organization Chart

```
Senior Engineering Leader (You)
├── Principal Software Engineer (Platform Architecture)
│   ├── Senior Engineer (APIs & Integration)
│   ├── Senior Engineer (Data Pipeline)
│   └── Senior Engineer (Security)
├── Principal Architect (Solutions)
│   ├── Architect (Data Platform)
│   ├── Architect (Client Experience)
│   └── Architect (Operations)
└── Engineering Manager
    ├── Team Lead (Backend Services) - 4 engineers
    ├── Team Lead (Data / Analytics) - 4 engineers
    └── Team Lead (DevOps / Infra) - 3 engineers
```

### Responsibilities by Role

| Role | Key Responsibilities |
|------|----------------------|
| **You (Senior Leader)** | Vision, roadmap, stakeholder alignment, hiring, architecture governance |
| **Principal Engineers** | Technical depth, mentorship, design reviews, innovation initiatives |
| **Architects** | Solution design, cross-team collaboration, risk mitigation |
| **Engineering Manager** | Team velocity, hiring, 1-on-1s, delivery accountability |
| **Individual Contributors** | Feature delivery, code quality, documentation, on-call support |

---

## 9. Critical Success Factors

### 8.1 Technical Pillars
1. **Scalability**: Handle 10x growth in client base and data volume
2. **Reliability**: 99.99% SLA across all tiers
3. **Security**: Zero-trust architecture; zero data breaches
4. **Observability**: Real-time alerting, end-to-end tracing
5. **Automation**: Self-healing, minimal manual operations

### 8.2 Organizational Pillars
1. **Alignment**: Daily sync with Sales, Marketing, ops teams
2. **Empowerment**: Teams own their services end-to-end
3. **Learning**: Knowledge sharing, technical talks, mentorship
4. **Accountability**: Clear KPIs tied to compensation
5. **Autonomy**: Minimal process overhead, velocity focus

---

## 10. Architecture Decision Records (ADRs)

### ADR-001: Multi-Modal Distribution over Single Channel
**Decision**: Implement 4-tier distribution mesh (Web, API, Kafka, ADX) instead of single "API-first" approach

**Rationale**:
- **Clients have diverse technical maturity**: Some want dashboards (business users), others want Kafka streams (quant shops)
- **Different consumption patterns**: Batch (dashboards), interactive (APIs), real-time (Kafka)
- **Cost efficiency**: ADX zero-copy model only viable if clients can query their own compute
- **Time-to-market**: Phase in channels incrementally (Web → API → Kafka → ADX)

**Consequences**:
- ✅ Higher client adoption
- ❌ More operational complexity (4 platforms to manage)
- ❌ Testing matrix explodes (cross-channel consistency)

---

### ADR-002: AWS Data Exchange over Self-Managed Marketplace
**Decision**: Use AWS Data Exchange (managed) instead of building custom data marketplace

**Rationale**:
- **No infrastructure to manage**: ADX handles billing, entitlements, discovery
- **Time-to-value**: 2 weeks to launch vs. 6 months for custom
- **Security**: AWS managed compliance; we focus on data quality
- **Scalability**: Auto-scales to thousands of subscribers

**Consequences**:
- ✅ Faster go-to-market
- ✅ Lower run cost
- ❌ Vendor lock-in to AWS
- ❌ Limited customization (we adapt to ADX, not vice versa)

---

### ADR-003: Spring Boot + Reactive Streams for Java Services
**Decision**: Use Spring Boot 3.x with Spring Cloud Kafka + WebFlux for microservices

**Rationale**:
- **Non-blocking I/O**: WebFlux handles 10K+ req/sec without thread explosion
- **Ecosystem**: Spring Cloud Stream abstracts Kafka complexity
- **Testability**: Testcontainers for Kafka, embedded servers for Spring Boot
- **Team expertise**: Java/Spring is institutional standard

**Consequences**:
- ✅ Developer productivity
- ✅ Proven enterprise stack
- ❌ Reactive programming is hard to onboard (need training)
- ❌ Memory footprint higher than Go/Python alternatives

---

### ADR-004: Event Sourcing for Audit Trail
**Decision**: Publish all changes to Kafka topics (audit trail) alongside database writes

**Rationale**:
- **Compliance**: FINRA requires 7-year data retention
- **Debugging**: Replay events to reproduce issues
- **Temporal queries**: "Give me state of Fund X as of 2024-01-15"

**Consequences**:
- ✅ Complete audit history
- ✅ Debugging capability
- ❌ Storage overhead (~2x)
- ❌ Complexity in idempotency (duplicate events in Kafka)

---

## 10.1 Code Examples & Implementation Patterns

### Example 1: REST API for NAV Lookup (Spring Boot)

```java
// pom.xml dependencies
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-aws-secrets-manager-config</artifactId>
</dependency>
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>

// Controller
@RestController
@RequestMapping("/v2/funds")
@CrossOrigin(origins = "*", maxAge = 3600)
public class FundController {

  @Autowired private FundService fundService;
  @Autowired private MeterRegistry meterRegistry;
  
  private static final Logger logger = LoggerFactory.getLogger(FundController.class);

  @GetMapping("/{fundId}/nav")
  @Operation(summary = "Get current NAV for fund")
  public Mono<ResponseEntity<NavResponse>> getNAV(
      @PathVariable String fundId,
      @RequestParam(required = false) LocalDate asOfDate) {
    
    return fundService.fetchNAV(fundId, asOfDate)
      .doOnNext(nav -> {
        meterRegistry.counter("nav.requests.success").increment();
        logger.info("NAV fetched for fund={}, nav={}", fundId, nav.getValue());
      })
      .doOnError(ex -> {
        meterRegistry.counter("nav.requests.error").increment();
        logger.error("Error fetching NAV for fund={}", fundId, ex);
      })
      .map(nav -> ResponseEntity.ok(new NavResponse(nav, LocalDateTime.now())))
      .onErrorResume(ex -> Mono.just(
        ResponseEntity.status(500).body(
          new NavResponse(null, LocalDateTime.now())
        )
      ));
  }
}

// Service with retry logic
@Service
public class FundService {

  @Autowired private FundRepository fundRepo;
  @Autowired private AthenaClient athenaClient;
  
  private static final int MAX_RETRIES = 3;
  private static final int RETRY_DELAY_MS = 100;

  public Mono<NAVData> fetchNAV(String fundId, LocalDate asOfDate) {
    return Mono.fromCallable(() -> fundRepo.findNAVByFundId(fundId, asOfDate))
      .retryWhen(Retry.backoff(MAX_RETRIES, Duration.ofMillis(RETRY_DELAY_MS))
        .doBeforeRetry(signal -> {
          logger.warn("Retry attempt {}", signal.totalRetries() + 1);
        })
      )
      .subscribeOn(Schedulers.boundedElastic());
  }
}
```

### Example 2: Kafka Producer for NAV Events

```java
// KafkaProducerConfig.java
@Configuration
public class KafkaProducerConfig {

  @Value("${kafka.bootstrap.servers}")
  private String bootstrapServers;

  @Bean
  public ProducerConfig producerConfig() {
    return new ProducerConfig()
      .put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers)
      .put(ProducerConfig.RETRIES_CONFIG, 3)
      .put(ProducerConfig.ACKS_CONFIG, "all")  // Wait for all replicas
      .put(ProducerConfig.COMPRESSION_TYPE_CONFIG, "snappy")
      .put(ProducerConfig.LINGER_MS_CONFIG, 100)  // Batch for latency-throughput trade-off
      .put(ProducerConfig.BATCH_SIZE_CONFIG, 32768);
  }

  @Bean
  public KafkaTemplate<String, NAVUpdateEvent> kafkaTemplate() {
    return new KafkaTemplate<>(
      new DefaultKafkaProducerFactory<>(producerConfig().asProperties())
    );
  }
}

// NAVProducer.java
@Component
public class NAVProducer {

  @Autowired private KafkaTemplate<String, NAVUpdateEvent> kafkaTemplate;
  private static final String TOPIC = "asset-management.nav.intraday";
  private static final Logger logger = LoggerFactory.getLogger(NAVProducer.class);

  public void publishNAVUpdate(String fundId, BigDecimal nav, LocalDateTime timestamp) {
    NAVUpdateEvent event = NAVUpdateEvent.builder()
      .fundId(fundId)
      .nav(nav)
      .timestamp(timestamp)
      .eventId(UUID.randomUUID().toString())  // Idempotency key
      .build();

    kafkaTemplate.send(TOPIC, fundId, event)
      .addCallback(
        result -> logger.info("NAV event published: {}", fundId),
        ex -> logger.error("Failed to publish NAV event", ex)
      );
  }
}
```

### Example 3: Entitlement Automation - AWS Step Functions (JSON)

```json
{
  "Comment": "Automate entitlement provisioning from Salesforce to ADX",
  "StartAt": "ValidateSalesforceEvent",
  "States": {
    "ValidateSalesforceEvent": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:ACCOUNT:function:ValidateSalesforceEvent",
      "Retry": [{"ErrorEquals": ["States.TaskFailed"], "IntervalSeconds": 2, "MaxAttempts": 3, "BackoffRate": 2}],
      "Catch": [{"ErrorEquals": ["States.ALL"], "Next": "PublishErrorNotification"}],
      "Next": "LookupClientAWSAccountId"
    },
    "LookupClientAWSAccountId": {
      "Type": "Task",
      "Resource": "arn:aws:states:::dynamodb:getItem",
      "Parameters": {
        "TableName": "ClientAwsAccounts",
        "Key": {"ClientId": {"S.$": "$.salesforceClientId"}}
      },
      "Next": "CallADXGrantAccess"
    },
    "CallADXGrantAccess": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:ACCOUNT:function:ADXGrantDatasetAccess",
      "Parameters": {
        "DatasetId.$": "$.datasetId",
        "ClientAwsAccountId.$": "$.Item.AwsAccountId.S",
        "ExpirationDate.$": "$.contractEndDate"
      },
      "Retry": [{"ErrorEquals": ["ADXApiThrottled"], "IntervalSeconds": 5, "MaxAttempts": 5, "BackoffRate": 2}],
      "Catch": [{"ErrorEquals": ["States.ALL"], "Next": "PublishErrorNotification"}],
      "Next": "UpdateSalesforceCompletion"
    },
    "UpdateSalesforceCompletion": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:ACCOUNT:function:UpdateSalesforceField",
      "Parameters": {
        "RecordId.$": "$.salesforceRecordId",
        "Field": "ADX_Provisioned__c",
        "Value": "true"
      },
      "Next": "SuccessState"
    },
    "PublishErrorNotification": {
      "Type": "Task",
      "Resource": "arn:aws:states:::sns:publish",
      "Parameters": {
        "TopicArn": "arn:aws:sns:us-east-1:ACCOUNT:ADXProvisioningErrors",
        "Subject": "ADX Entitlement Provisioning Failed",
        "Message.$": "$"
      },
      "Next": "FailureState"
    },
    "SuccessState": {
      "Type": "Succeed"
    },
    "FailureState": {
      "Type": "Fail",
      "Error": "EntitlementProvisioningFailed",
      "Cause": "See CloudWatch logs for details"
    }
  }
}
```

---

## 10.1 End-to-End Integration: Aladdin Trade → Athena Query

### The Complete Flow (Marcus Johnson's Request)

**Scenario**: Aladdin trade (1000 shares AAPL @ $150) → CDC captures → Kafka → enriched → S3 → queried by client

#### Step 1: Aladdin CDC Captures Trade

```java
// Lambda triggered by AppFlow CDC event from Aladdin
public class AladdinCDCLambda implements RequestHandler<AvroEvent, String> {
  
  @Override
  public String handleRequest(AvroEvent event, Context context) {
    // Event schema: tradeId, portfolio, security, qty, price, timestamp
    logger.info("CDC Event: {} {} shares of {} @ {}", 
      event.getTradeId(), event.getQty(), event.getSecurity(), event.getPrice());
    
    // Call AppFlow → MSK producer
    sendToKafka(event);
    return "OK";
  }
}
```

#### Step 2: Kafka Producer Publishes

```java
public class AladdinCDCProducer {
  private final KafkaTemplate<String, AladdinTradeEvent> kafkaTemplate;
  
  public void publishTrade(AladdinTrade trade) {
    AladdinTradeEvent event = AladdinTradeEvent.builder()
      .tradeId(UUID.randomUUID().toString())  // Idempotency
      .portfolioId(trade.getPortfolioId())
      .security(trade.getSecurity())
      .quantity(trade.getQuantity())
      .price(trade.getExecutionPrice())
      .timestamp(Instant.now())
      .sourceSystem("ALADDIN")
      .tenantId(trade.getTenantId())
      .build();
    
    // Add trace ID for cross-domain observability
    event.setTraceId(MDC.get("traceId"));
    
    kafkaTemplate.send("aladdin.trades", trade.getPortfolioId(), event);
  }
}
```

#### Step 3: Spring Boot Consumer Enriches from Redshift

```java
@Component
public class AladdinTradeEnricher {
  
  private final JdbcTemplate redshiftTemplate;
  private final S3Client s3Client;
  private final CloudWatchClient cloudWatch;
  
  @KafkaListener(topics = "aladdin.trades", groupId = "enrich-consumer")
  public void consumeAndEnrich(AladdinTradeEvent trade) {
    try {
      // Step 3a: Lookup Bloomberg data from Redshift Data Share
      Map<String, Object> bloombergData = redshiftTemplate.queryForMap(
        "SELECT last_price, bid_ask_spread FROM bloomberg_data.securities WHERE isin = ?",
        trade.getIsin()
      );
      
      // Step 3b: Join with our internal cost basis
      Map<String, Object> internalData = redshiftTemplate.queryForMap(
        "SELECT cost_basis, current_quantity FROM gold.positions WHERE portfolio_id = ? AND security = ?",
        trade.getPortfolioId(), trade.getSecurity()
      );
      
      // Step 3c: Enrich event
      EnrichedTrade enriched = EnrichedTrade.builder()
        .tradeId(trade.getTradeId())
        .aladdinPrice(trade.getPrice())
        .bloombergPrice((BigDecimal) bloombergData.get("last_price"))
        .internalCostBasis((BigDecimal) internalData.get("cost_basis"))
        .priceDiscrepancy(((BigDecimal) trade.getPrice())
          .subtract((BigDecimal) bloombergData.get("last_price")))
        .traceId(MDC.get("traceId"))  // Carry trace through system
        .build();
      
      // Step 3d: Write to S3 Gold layer (Parquet format for query performance)
      writeToS3Gold(enriched);
      
      // Metrics for CloudWatch
      cloudWatch.putMetricData(request -> request
        .namespace("AladdinDataMesh")
        .metricData(MetricDatum.builder()
          .metricName("TradesEnrichedPerMinute")
          .value(1.0)
          .build()
        )
      );
      
    } catch (Exception ex) {
      logger.error("Enrichment failed for trade={}: {}", trade.getTradeId(), ex.getMessage());
      dlqTemplate.send("aladdin.trades.dlq", trade);  // Dead letter queue
    }
  }
  
  private void writeToS3Gold(EnrichedTrade enriched) {
    String s3Path = String.format(
      "s3://data-lake/gold/aladdin_trades/date=%s/trade_%s.parquet",
      LocalDate.now(), enriched.getTradeId()
    );
    
    // Write as Parquet for Athena query efficiency
    s3Client.putObject(request -> request
      .bucket("data-lake")
      .key(s3Path)
      .contentType("application/octet-stream")
      .build(),
      RequestBody.fromBytes(serializeToParquet(enriched))
    );
  }
}
```

#### Step 4: Athena Query with Row-Level Security

```sql
-- Client-A queries Athena; RLS filters to their portfolio only
-- RLS enforced by Lake Formation dynamic policy

SELECT
  trade_id,
  portfolio_id,
  security,
  quantity,
  aladdin_price,
  bloomberg_price,
  price_discrepancy,
  timestamp
FROM s3://data-lake/gold/aladdin_trades/
WHERE 
  portfolio_id IN (SELECT portfolio_id FROM client_entitlements 
                   WHERE client_id = current_user)  -- RLS constraint
  AND date >= date_format(current_date - INTERVAL '1' DAY, '%Y-%m-%d')
ORDER BY timestamp DESC;
```

---

### Resilience4j Circuit Breaker Implementation

```java
@Configuration
public class AladdinResilienceConfig {
  
  @Bean
  public CircuitBreaker aladdinCDCCircuitBreaker() {
    return CircuitBreaker.of("aladdin-cdc",
      CircuitBreakerConfig.custom()
        .failureThreshold(5)          // Open after 5 failures
        .slowCallRateThreshold(30)    // Open if >30% of calls slow
        .slowCallDurationThreshold(Duration.ofSeconds(2))
        .waitDurationInOpenState(Duration.ofSeconds(30))
        .permittedNumberOfCallsInHalfOpenState(3)  // Try 3 calls to recover
        .build()
    );
  }
  
  @Bean
  public Retry aladdinRetry() {
    return Retry.of("aladdin-cdc",
      RetryConfig.custom()
        .maxAttempts(3)
        .waitDuration(Duration.ofMillis(500))
        .intervalFunction(IntervalFunction.ofExponentialBackoff(500, 2))
        .build()
    );
  }
}

@Component
public class ResilientAladdinConsumer {
  
  private final CircuitBreaker circuitBreaker;
  private final Retry retry;
  private final KafkaTemplate<String, AladdinTradeEvent> deadLetterQueue;
  
  @KafkaListener(topics = "aladdin.trades")
  public void consumeWithResilience(ConsumerRecord<String, AladdinTradeEvent> record) {
    try {
      circuitBreaker.executeRunnable(() ->
        retry.executeRunnable(() -> {
          processAladdinTrade(record.value());
        })
      );
    } catch (CircuitBreakerOpenException | CallNotPermittedException ex) {
      logger.warn("Circuit breaker OPEN; buffering trade locally");
      bufferForRetry(record.value());
    }
  }
  
  private void processAladdinTrade(AladdinTradeEvent trade) throws IOException {
    // Actual processing logic
  }
}
```

---

## 10.2 Operational Runbooks

### Runbook 1: API Latency Spike Investigation

**Trigger**: CloudWatch alarm - p95 latency > 500ms

1. **Check Dashboard**: `aws cloudwatch get-dashboard --name FundAPIMetrics`
   - Look for: CPU %, Lambda cold start rate, Athena query time

2. **If Lambda cold starts high**:
   - Check: Recent deployments (CodeDeploy history)
   - Action: Increase Lambda provisioned concurrency
   - Rollback: `aws lambda put-provisioned-concurrency-config ...`

3. **If Athena query slow**:
   - Check: `SELECT COUNT(*) FROM s3://data-lake/gold/nav WHERE date = today()`
   - Root cause: Partition pruning failing?
   - Action: Add partition projection to table metadata

4. **If all metrics normal but clients report latency**:
   - Check: CloudFront cache hit ratio
   - Action: Invalidate cache if data updated: `aws cloudfront create-invalidation`

5. **Escalate if unresolved**: Page on-call Principal Engineer

---

### Runbook 2: Kafka Topic Offset Lag

**Trigger**: Consumer offset lag > 10,000 messages

1. **Identify lagging consumer group**:
   ```bash
   kafka-consumer-groups --bootstrap-server $KAFKA_BROKER --group $GROUP --describe
   ```

2. **Check consumer process**:
   ```bash
   ps aux | grep spring-cloud-stream
   kubectl get pods -l app=kafka-consumer
   ```

3. **If consumer is crashed**:
   - Restart: `kubectl rollout restart deployment/kafka-consumer`
   - Verify: `kubectl logs -f deployment/kafka-consumer`

4. **If consumer is running but still lagging**:
   - Check: Is it processing slowly? (Check application metrics)
   - Action: Increase consumer thread pool: `spring.cloud.stream.kafka.binder.concurrency=10`

5. **If lag continues after restart**:
   - Option A: Skip to latest offset (lose messages): `kafka-consumer-groups --reset-offsets --to-latest`
   - Option B: Investigate downstream system (is it slow to consume?)

---

### Runbook 3: Salesforce → ADX Provisioning Failure

**Trigger**: Execution role closed deal, but ADX shows no access after 5 minutes

1. **Check Step Function execution**:
   ```bash
   aws stepfunctions describe-execution --execution-arn <EXECUTION_ARN>
   aws stepfunctions get-execution-history --execution-arn <EXECUTION_ARN>
   ```

2. **Likely root causes**:
   | Error | Action |
   |-------|--------|
   | `LookupClientAWSAccountId` failed | Client record not in DynamoDB; ask sales team for AWS account ID |
   | `CallADXGrantAccess` timeout | ADX API might be throttling; retry with backoff |
   | `UpdateSalesforceCompletion` failed | Salesforce API key expired; rotate via AWS Secrets Manager |
   | Lambda timeout (>15min) | Hypothesis: ADX dataset too large; increase Lambda timeout |

3. **Manual recovery**:
   ```bash
   # Manually grant access to ADX (if automation stuck)
   aws dataexchange create-event-action \
     --action Name=SendDataSetNotification,Parameters='{"SnsTopicArn":"value"}' \
     --event Name=RevisionPublished,Parameters='{"DataSetId":"value"}'
   ```

4. **Notify client**: "ADX access provisioned manually; please refresh your AWS Console (refresh = 5min cache)"

---

## 10.3 Monitoring & Alerting Strategy

### Key Metrics Dashboard

| Metric | Threshold | Action |
|--------|-----------|--------|
| **REST API p95 latency** | > 500ms | Page on-call engineer |
| **REST API error rate** | > 1% (5xx errors) | Check Lambda logs, possible DDoS |
| **Kafka consumer lag** | > 10,000 messages | Check consumer app health |
| **ADX subscription failure rate** | > 5% | Check Salesforce-to-Step-Functions pipeline |
| **Data freshness (NAV)** | > 5 minutes stale | Check ETL job status |
| **Athena query cost/day** | > $500 | Investigate unoptimized queries |

---

## 10.4 Resilience Patterns & Disaster Recovery

### Circuit Breaker Pattern (for Kafka Producer)

```java
@Component
public class ResilientKafkaProducer {
  
  private final KafkaTemplate<String, NAVUpdateEvent> kafkaTemplate;
  private final CircuitBreaker circuitBreaker;
  private static final Logger logger = LoggerFactory.getLogger(ResilientKafkaProducer.class);
  
  public ResilientKafkaProducer(KafkaTemplate<String, NAVUpdateEvent> kafkaTemplate) {
    this.kafkaTemplate = kafkaTemplate;
    // Circuit breaker: 5 failures → OPEN (reject) for 30sec, then try HALF_OPEN
    this.circuitBreaker = CircuitBreaker.of("kafka-producer", 
      CircuitBreakerConfig.custom()
        .failureThreshold(5)
        .slowCallRateThreshold(50)
        .slowCallDurationThreshold(Duration.ofSeconds(2))
        .waitDurationInOpenState(Duration.ofSeconds(30))
        .build());
  }

  public void publishNAVWithResilience(String fundId, BigDecimal nav) {
    try {
      circuitBreaker.executeSupplier(() -> {
        kafkaTemplate.send("asset-management.nav.intraday", fundId, 
          NAVUpdateEvent.builder()
            .fundId(fundId)
            .nav(nav)
            .eventId(UUID.randomUUID().toString())
            .build());
        return true;
      });
    } catch (CircuitBreakerOpenException | CallNotPermittedException ex) {
      logger.warn("Circuit breaker OPEN - buffering event locally for retry");
      // Buffer event to local queue for later retry
      bufferEventForRetry(fundId, nav);
    }
  }
  
  private void bufferEventForRetry(String fundId, BigDecimal nav) {
    // Persist to DynamoDB or local RocksDB for eventual consistency
    localBuffer.put(fundId, nav);
  }
}
```

### Bulkhead Pattern (Thread Pool Isolation)

```java
@Configuration
public class ThreadPoolConfig {
  
  @Bean("kafkaExecutor")
  public ThreadPoolTaskExecutor kafkaExecutor() {
    ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
    executor.setCorePoolSize(10);
    executor.setMaxPoolSize(20);
    executor.setQueueCapacity(500);
    executor.setThreadNamePrefix("kafka-");
    executor.setWaitForTasksToCompleteOnShutdown(true);
    executor.setAwaitTerminationSeconds(60);
    executor.initialize();
    return executor;
  }
  
  @Bean("apiExecutor")
  public ThreadPoolTaskExecutor apiExecutor() {
    ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
    executor.setCorePoolSize(50);
    executor.setMaxPoolSize(100);
    executor.setQueueCapacity(5000);
    executor.setThreadNamePrefix("api-");
    executor.initialize();
    return executor;
  }
}

// Usage: Kafka consumers use dedicated thread pool → won't starve API requests
@Service
public class NAVConsumer {
  @Async("kafkaExecutor")
  public void consumeNAVUpdate(NAVUpdateEvent event) {
    // Isolated thread pool ensures API latency unaffected
  }
}
```

### Disaster Recovery: RTO & RPO

| Scenario | RTO (Recovery Time) | RPO (Recovery Point) | Strategy |
|----------|-------------------|------------------|----------|
| **S3 data corruption** | < 1 hour | < 1 hour | S3 versioning; automated restore from backup |
| **Kafka broker failure** | < 15 min | 0 minute | Multi-AZ deployment; topic replication factor = 3 |
| **Lambda timeout** | < 5 min | 0 minute | Automatic retry; DLQ for manual intervention |
| **Salesforce API down** | < 2 hours | 1 day | Queue provisioning requests; retry next day |
| **ADX service down** | < 4 hours | 1 hour | Fallback to REST API; notify clients of degradation |

**Failover Procedure**:
```bash
# If primary Kafka cluster fails, promote standby
aws msk update-cluster --cluster-arn <PRIMARY> --replication-info FailureZone=us-east-1b
# Redirect producers to standby broker list (via feature flag)
# ETA: 15 minutes
```

---

### Advanced Java/Spring Boot Production Tuning

#### Virtual Threads (Java 21+) for High Concurrency

```java
// Spring Boot 3.2+ with Virtual Threads
@Configuration
public class WebConfig implements WebMvcConfigurer {
  
  @Override
  public void configureAsyncSupport(AsyncSupportConfigurer configurer) {
    // Virtual threads: lightweight, non-blocking, scale to 100K+
    configurer.setTaskExecutor(Executors.newVirtualThreadPerTaskExecutor());
  }
}

// Usage: Each HTTP request runs on a virtual thread (not platform thread)
// Benefit: 100K+ concurrent requests with minimal memory
```

#### GC Tuning for Low-Latency APIs

```bash
# Launch API with G1GC (default in Java 11+, optimized for 99% p99 latency)
java -server \
  -XX:+UseG1GC \
  -XX:MaxGCPauseMillis=100 \                # Target: keep GC pauses <100ms
  -XX:+ParallelRefProcEnabled \              # Parallel reference processing
  -XX:+UnlockDiagnosticVMOptions \
  -XX:G1SummarizeRSetStatsPeriod=1 \
  -Xms4g -Xmx4g \
  -jar fund-api-service.jar
```

#### OpenTelemetry Integration for Distributed Tracing

```java
@Configuration
public class TelemetryConfig {
  
  @Bean
  public OpenTelemetry openTelemetry() {
    SdkTracerProvider tracerProvider = SdkTracerProvider.builder()
      .addSpanProcessor(JaegerThriftSpanExporter.builder()
        .setAgentHost("jaeger-collector")
        .setAgentPort(6831)
        .build().getSpanProcessor())
      .build();
    
    return OpenTelemetrySdk.builder()
      .setTracerProvider(tracerProvider)
      .buildAndRegisterGlobal();
  }
}

// Auto-instrument with Spring Boot Starter
// dependency: io.opentelemetry.instrumentation:opentelemetry-spring-boot-starter
// Every HTTP request, database call, Kafka publish gets traced
```

---

### Incident Severity Levels & Response SLAs

| Severity | Definition | Example | First Response | Resolution Target |
|----------|-----------|---------|-----------------|-------------------|
| **P1 (Critical)** | Entire service down or data loss risk | All APIs returning 500 | 5 minutes | 15 minutes (RTO) |
| **P2 (High)** | Feature degraded; >50% clients impacted | API p95 > 5s | 15 minutes | 1 hour |
| **P3 (Medium)** | Minor feature broken; <10% impacted | Dashboard slow but functional | 1 hour | 4 hours |
| **P4 (Low)** | Documentation issue or cosmetic bug | Typo in API docs | 2 business days | - |

**On-Call Escalation**:
- P1 → Page Principal Engineer (immediately) → VP Engineering (if unfixed after 30 min)
- P2 → Slack #incident-channel → escalate if unresolved after 1 hour
- P3/P4 → Backlog ticket; add to next sprint

---

### Organizational Scaling Plan

#### Hiring & Ramp Timeline (0 → 12 Engineers)

| Phase | Timeline | Hires | Key Roles | Milestones |
|-------|----------|-------|-----------|-----------|
| **Foundation** | Month 1-3 | 2 Principal Engrs + 1 Senior | API architect, Data engineer | V0 architecture; single service MVP |
| **Core MVP** | Month 4-6 | +3 mid-level engineers | Backend, Kafka, DevOps | Multi-modal foundation; K8s deployment |
| **Scale** | Month 7-9 | +2 Principal Architects + 4 mid-level | Data platform, Security, SRE | Marketplace launch; 3+ services |
| **Production** | Month 10-12 | [Reserve funding] | Backup hires; intern rotation | 80% client adoption; stable platform |

**Hiring Criteria**:
- **Principal roles**: 8+ years, led teams of 5+, shipped to 1000+ users
- **Mid-level**: 3-5 years, shipped 2+ microservices, AWS/GCP experience
- **Junior**: 0-2 years, computer science degree, willingness to learn

---

#### Leveling & Promotion Framework

| Level | Title | Comp Range | Responsibilities | Promotion Criteria |
|-------|-------|------------|------------------|-------------------|
| **L3** | Senior Software Engineer | $200-250K | Own service E2E; mentor 1-2 L2s | Led design of new service; mentored 2 L2s → L3 |
| **L4** | Staff Engineer | $250-350K | Drive architectural decisions; own roadmap for domain | Led company-wide initiative (dataflow layer); published 2 ADRs |
| **L5** | Principal Engineer | $350-500K | Set technical vision; manage hiring for function | Built one of the 4 tiers solo; mentor 2+ L4s |
| **L6** | Distinguished Engineer (VP) | $500K+ | VP-level; sets technical direction for company | Led organizational transformation (push → pull model); 5+ L5s report |

**Promotion Process**:
1. Manager + 2 senior peers write promotion packet (criteria met?)
2. Engineering leadership review (blind comparison against peers)
3. Compensation committee approves
4. Promotion announcement + 5% raise (or market correction)

---

#### Database Schema (Entity Relationship Diagram)

```sql
-- Core Entities for Multi-Tenant NAV Management

CREATE TABLE funds (
  fund_id UUID PRIMARY KEY,
  tenant_id UUID NOT NULL,          -- Partition key for multi-tenancy
  fund_name VARCHAR(255) NOT NULL,
  fund_type ENUM ('Equity', 'Fixed Income', 'Private Credit'),
  inception_date DATE NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  INDEX (tenant_id, fund_id) -- Composite for fast tenant-scoped queries
);

CREATE TABLE nav_history (
  nav_id UUID PRIMARY KEY,
  fund_id UUID NOT NULL,
  tenant_id UUID NOT NULL,          -- Denormalized for query performance
  nav_value DECIMAL(19,2) NOT NULL,
  nav_date DATE NOT NULL,
  data_source ENUM ('Official', 'Preliminary', 'Estimate'),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (fund_id) REFERENCES funds(fund_id),
  INDEX (tenant_id, fund_id, nav_date) -- Partition pruning queries
);

CREATE TABLE client_entitlements (
  entitlement_id UUID PRIMARY KEY,
  tenant_id UUID NOT NULL,
  client_aws_account_id VARCHAR(12) NOT NULL,  -- For ADX access
  dataset_ids ARRAY<VARCHAR(100)> NOT NULL,   -- Which data products
  contract_end_date DATE NOT NULL,
  salesforce_record_id VARCHAR(18) NOT NULL,
  status ENUM ('Active', 'Expired', 'Revoked'),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  INDEX (tenant_id, client_aws_account_id)
);

CREATE TABLE audit_log (
  log_id UUID PRIMARY KEY,
  tenant_id UUID NOT NULL,
  entity_type VARCHAR(50) NOT NULL,
  entity_id UUID NOT NULL,
  action ENUM ('CREATE', 'UPDATE', 'DELETE'),
  changed_by VARCHAR(255) NOT NULL,
  changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  old_values JSONB,
  new_values JSONB,
  INDEX (tenant_id, entity_type, entity_id, changed_at)
);
```

**Schema Design Principles**:
- ✅ **Tenant-id as partition key**: Fast isolation per client
- ✅ **Denormalized nav_value for queries**: Avoid joins on hot path
- ✅ **Composite indexes**: (tenant_id, fund_id) enables partition pruning
- ✅ **JSONB audit trail**: Compliance-ready; supports temporal queries
- ✅ **No cross-tenant queries**: Prevents data leaks

---



## 11. Getting Started

### Prerequisites
- AWS Account with appropriate permissions (AdministratorAccess for setup)
- Salesforce System Admin access
- Kafka cluster (MSK or self-managed)
- .NET 9.0+, Python 3.11+, Node.js 18+, Java 17+
- Docker for local development
- Terraform 1.5+ for infrastructure provisioning

### Quick Start Guide

1. **Clone and setup**:
   ```bash
   git clone https://github.com/calvinlee999/Asset_Management_Client_Technology.git
   cd Asset_Management_Client_Technology
   
   # Configure AWS credentials
   export AWS_PROFILE=default
   aws sts get-caller-identity  # Verify access
   
   # Set environment variables
   cp .env.example .env
   source .env
   ```

2. **Deploy infrastructure**:
   ```bash
   cd infrastructure
   terraform init
   terraform plan -out=tfplan
   terraform apply tfplan
   ```

3. **Build & deploy services**:
   ```bash
   # API service
   cd ../src/apis
   ./gradlew bootRun -Dspring.profiles.active=local
   
   # Kafka producer
   cd ../kafka-producers
   python -m pip install -r requirements.txt
   python producer.py --env local
   ```

4. **Verify deployment**:
   ```bash
   # Test REST API
   curl http://localhost:8080/v2/funds/FND001/nav
   
   # Test Kafka
   kafka-console-consumer --bootstrap-server localhost:9092 \
     --topic asset-management.nav.intraday --from-beginning
   ```

### Repository Structure
```
Asset_Management_Client_Technology/
├── README.md                          # This file
├── EVALUATION_CYCLE_1.md              # Training feedback logs
├── docs/
│   ├── ADR/                           # Architecture Decision Records
│   ├── runbooks/                      # Operational procedures
│   │   ├── api-latency-investigation.md
│   │   ├── kafka-lag-resolution.md
│   │   └── salesforce-adx-provisioning.md
│   ├── data-lineage.md                # Data flow diagrams
│   ├── security-framework.md          # Compliance matrix
│   └── onboarding.md                  # New team member guide
├── src/
│   ├── apis/
│   │   ├── fund-service/              # Spring Boot REST APIs
│   │   └── performance-service/
│   ├── data-pipeline/
│   │   ├── etl/                       # Spark jobs (Bronze → Gold)
│   │   └── dbt/                       # Data transformation (dbt)
│   ├── kafka-producers/               # Event producers (Scala/Python)
│   ├── salesforce-integration/        # Entitlement sync Lambda
│   └── infrastructure/
│       ├── main.tf                    # Terraform root module
│       ├── networking.tf              # VPC, ECS, MSK
│       ├── data-lake.tf               # S3, Glue, Athena
│       └── monitoring.tf              # CloudWatch, Prometheus
├── tests/
│   ├── integration/                   # E2E API + Kafka tests
│   ├── performance/                   # Load testing (k6, JMeter)
│   └── security/                      # OWASP scanning, penetration tests
├── .github/
│   └── workflows/                     # CI/CD pipelines
│       ├── test.yml                   # Run on every PR
│       └── deploy.yml                 # Automated deployment
└── scripts/
    ├── api-smoke-tests.sh
    ├── generate-sample-data.sh
    └── backup-s3-data.sh
```

---

## 11.1 Team Onboarding Guide

### "Hello World" Developer Tasks (Your First Week Contributions)

**Goal**: Contribute real code to production data pipeline by end of Week 1

#### Task 1: Consume Aladdin Events from Kafka (Day 2-3)

```java
// Create: KafkaConsumerTest.java (your first test)
package com.nomura.aladdin.consumer;

import org.junit.jupiter.api.Test;
import org.springframework.kafka.test.context.EmbeddedKafka;
import static org.junit.jupiter.api.Assertions.*;

@EmbeddedKafka(partitions = 1, brokerProperties = {
  "listeners=PLAINTEXT://localhost:9092"
})
public class AladdinTradeConsumerTest {
  
  @Test
  void testConsumeTradeEvent() {
    // 1. Create a mock Aladdin trade event
    AladdinTradeEvent trade = AladdinTradeEvent.builder()
      .tradeId("TRD-001")
      .portfolio("USD-LIQ")
      .security("AAPL")
      .quantity(1000)
      .price(BigDecimal.valueOf(150.00))
      .build();
    
    // 2. Send it to Kafka topic
    kafkaTemplate.send("aladdin.trades", "USD-LIQ", trade);
    
    // 3. Your consumer processes it (verify output)
    assertTrue(consumer.hasProcessedTrade("TRD-001"));
    assertEquals(1000, consumer.getQuantityProcessed());
  }
}

// Run: ./gradlew test -i AladdinTradeConsumerTest
// Success: ✅ You've consumed a real Kafka event!
```

**Learning Outcomes**:
- Understand Spring Cloud Stream Kafka integration
- See how Aladdin trades flow through the system
- Learn embedded Kafka testing pattern

---

#### Task 2: Query Athena for Your First Data (Day 4)

```python
# Create: scripts/query_aladdin_trades.py
import boto3
import time

athena_client = boto3.client('athena', region_name='us-east-1')

# Simple query: count trades from today
query = """
SELECT COUNT(*) as trade_count, portfolio_id
FROM s3://data-lake/gold/aladdin_trades/
WHERE date = date_format(current_date, '%Y-%m-%d')
GROUP BY portfolio_id
"""

# Execute query
response = athena_client.start_query_execution(
    QueryString=query,
    QueryExecutionContext={'Database': 'data_mesh'},
    ResultConfiguration={'OutputLocation': 's3://athena-results/'},
    WorkGroup='primary'
)

execution_id = response['QueryExecutionId']

# Wait for results
while True:
    result = athena_client.get_query_execution(QueryExecutionId=execution_id)
    status = result['QueryExecution']['Status']['State']
    
    if status in ['SUCCEEDED', 'FAILED', 'CANCELLED']:
        break
    time.sleep(1)

# Get results
if status == 'SUCCEEDED':
    results = athena_client.get_query_results(QueryExecutionId=execution_id)
    for row in results['ResultSet']['Rows']:
        print(f"Portfolio: {row['Data'][1]['VarCharValue']}, Trades: {row['Data'][0]['VarCharValue']}")

# Run: python scripts/query_aladdin_trades.py
# Output: Portfolio: USD-LIQ, Trades: 1247
# Success: ✅ You've queried production Aladdin data!
```

**Learning Outcomes**:
- Understand S3 data lake partitioning
- Learn Athena query execution
- See end-to-end: Kafka → S3 → query

---

#### Task 3: Write Your First RLS Policy (Day 5)

```python
# Create: terraform/lake_formation_rls.tf
resource "aws_lake_formation_data_lake_settings" "this" {
  # Enable data lake governance
  admins = ["arn:aws:iam::ACCOUNT:user/you"]
}

# RLS Policy: Client-A sees only their portfolios
resource "aws_lakeformation_resource_lf_tags" "aladdin_trades" {
  arn   = "arn:aws:s3:::data-lake/gold/aladdin_trades"
  
  lf_tags = {
    "data_classification" = "portfolio_data"
    "vendor_source"       = "aladdin"
  }
}

resource "aws_lake_formation_data_lake_principal_permissions" "client_a_portfolio" {
  principal   = "arn:aws:iam::ACCOUNT:role/ClientARole"
  permissions = ["SELECT"]
  
  table_with_columns_resource {
    database_name = "data_mesh"
    table_name    = "aladdin_trades"
    
    wildcard_columns = true
    
    # RLS row filter: Client-A sees only their portfolios
    expression = "portfolio_id IN (SELECT portfolio_id FROM client_entitlements WHERE client_id = 'CLIENT-A')"
  }
}

# Deploy: terraform apply
# Success: ✅ You've built data governance!
```

**Learning Outcomes**:
- Understand Lake Formation row-level security
- Learn to implement multi-tenant data access
- See how Nomura isolates client data

---

### Day 1: Environment Setup
- [ ] Create AWS IAM user and configure local credentials
- [ ] Clone repository: `git clone https://github.com/calvinlee999/Asset_Management_Client_Technology.git`
- [ ] Setup IDE (VS Code with Java/Kotlin extensions)
- [ ] Spin up local Kafka cluster: `docker-compose -f docker/docker-compose.kafka.yml up`
- [ ] Run unit tests: `gradle test`
- [ ] Read Section 12 (Fintech Glossary): Understand Aladdin, Bloomberg, FactSet, CDC
- [ ] Read Section 1.5-1.6 (Data Mesh + Talking Points): Business context

**Success criteria**: Can run `mvn spring-boot:run` and see API responding; understand why CDC is needed for Aladdin

### Week 1: Architecture Deep Dive
- [ ] **Day 2**: Read ADR-001 through ADR-004; understand why 4-tier distribution mesh beats single API
- [ ] **Day 3**: Complete "Hello World Task 1" (Kafka consumer test)
- [ ] **Day 4**: Complete "Hello World Task 2" (Athena query)
- [ ] **Day 5**: Complete "Hello World Task 3" (RLS policy)
- [ ] **Day 5**: Review vendor consolidation table (Table 2.0); explain to peer why Aladdin = CDC, Bloomberg = ADX, FactSet = Redshift share

**Success criteria**: Submitted 3 PRs (Kafka test + Athena script + Terraform RLS); can explain vendor integration strategy

### Week 2: Hands-On Contribution
- [ ] Pick an issue labeled `good-first-issue` (ideally in Aladdin domain or data quality validation)
- [ ] Submit first PR (even if small bug fix or documentation)
- [ ] Attend architecture review meeting (learn how team makes decisions)
- [ ] Pair with Aladdin Domain Lead for 2-hour deep dive on CDC patterns

**Success criteria**: First PR merged; can trace a single trade through Aladdin CDC → Kafka → Redshift → Athena

### Week 3-4: Service Ownership Ramp
- [ ] Assigned to one domain (e.g., "Aladdin Data Domain")
- [ ] Become on-call shadow (pair with on-call engineer through 1 incident)
- [ ] Complete security training (AWS IAM, encryption, PII handling, FINRA CAT regulations)
- [ ] Read Nomura compliance docs (SOC2 controls, HIPAA-equivalent for portfolio data)

**Success criteria**: Can independently deploy hotfix to production (with 2-person review); understand SLA implications (Aladdin <100ms = business-critical)

### Ongoing: Mentorship
- **1:1 with manager**: Bi-weekly, 30min (track progress on domain expertise)
- **Peer programming**: 1x/week with Aladdin Domain Lead (learn CDC internals, CDC edge cases)
- **Tech talks**: Monthly brown-bag sessions (Data Mesh principles, Zero-ETL, Lake Formation governance)
- **Career development**: 6-month career conversation (path to L3 → L4 Principal Engineer)

---

## 11.2 Team Onboarding Guide

### Day 1: Environment Setup
- [ ] Create AWS IAM user and configure local credentials
- [ ] Clone repository
- [ ] Setup IDE (VS Code with Java/Kotlin extensions)
- [ ] Spin up local Kafka cluster: `docker-compose -f docker/docker-compose.kafka.yml up`
- [ ] Run unit tests: `gradle test`

**Success criteria**: Can run `mvn spring-boot:run` and see API responding

### Week 1: Architecture Deep Dive
- [ ] Read [ADR-001 through ADR-004](#991-architecture-decision-records-adrs)
- [ ] Review [4-Tier Distribution Mesh](#2-multi-modal-data-distribution-mesh-architecture) section
- [ ] Pair with senior engineer on REST API walkthrough (2 hours)
- [ ] Run Kafka producer locally and consume messages

**Success criteria**: Can explain why ADX is better than self-managed marketplace

### Week 2: Hands-On Contribution
- [ ] Pick an issue labeled `good-first-issue`
- [ ] Submit first PR (even if small bug fix or documentation)
- [ ] Attend architecture review meeting (learn how team makes decisions)

**Success criteria**: First PR merged

### Week 3-4: Service Ownership Ramp
- [ ] Assigned to one service (e.g., "Fund API service")
- [ ] Become on-call shadow (pair with on-call engineer)
- [ ] Complete security training (AWS IAM, encryption, PII handling)

**Success criteria**: Can independently deploy hotfix to production (with 2-person review)

### Ongoing: Mentorship
- **1:1 with manager**: Bi-weekly, 30min
- **Peer programming**: 1x/week to learn from Principal Engineers
- **Tech talks**: Monthly brown-bag sessions on Kafka, observability, etc.
- **Career development**: 6-month career conversation

---

## 11.2 Definition of Done (DoD) - Service Delivery Checklist

Before any feature ships to production:

- [ ] **Code**
  - [ ] Unit test coverage ≥ 80%
  - [ ] Code reviewed by 2 engineers (1 must be Principal+)
  - [ ] No hardcoded secrets (use AWS Secrets Manager)
  - [ ] Passes checkstyle + SpotBugs + SonarQube gate

- [ ] **Testing**
  - [ ] Integration tests pass (E2E with Kafka + Athena)
  - [ ] Load test passes (>10K req/sec with <500ms p95)
  - [ ] Security scan passes (OWASP, dependency check)

- [ ] **Documentation**
  - [ ] API endpoints documented in OpenAPI/Swagger
  - [ ] Database schema documented
  - [ ] Runbook added if new operational procedure
  - [ ] ADR updated if architecture changed

- [ ] **Observability**
  - [ ] Metrics emitted (latency, errors, custom business metrics)
  - [ ] CloudWatch alarms configured
  - [ ] Logs in JSON format with structured fields

- [ ] **Deployment**
  - [ ] Terraform/CloudFormation updated for IaC
  - [ ] Feature flag added (for gradual rollout)
  - [ ] Database migration (if applicable) is backward compatible
  - [ ] Rollback plan documented

- [ ] **Sign-Off**
  - [ ] Product Manager approves feature
  - [ ] Platform lead approves architecture
  - [ ] On-call engineer approves operability
  - [ ] Security team approves compliance

---

## 12. Fintech Glossary & Vendor Context

### Understanding the Vendors

| Term | Definition | Why It Matters to Asset Mgmt |
|------|-----------|----------------------------|
| **Aladdin (BlackRock)** | Integrated portfolio management + execution + risk system. Real-time, mission-critical. | Aladdin is the "system of record" for fund managers. Every trade, position, order lives in Aladdin. Our CDC must capture it <100ms or we lag. |
| **Bloomberg Terminal** | Premium financial data/messaging platform ($20-40K/user/year). Publishes real-time prices, news, analysis. | Portfolio managers subscribe to Bloomberg for market prices. Bloomberg publishes equity closes 5pm EST, we ingest via ADX, available in Athena by 6pm. |
| **FactSet** | Analytics + regulatory reporting + research database. Flat subscription (per seat or AUM). | Analysts use FactSet for EPS estimates, peer company data, sector analysis. We share FactSet via Redshift Data Share so they join with our internal forecasts in same query. |
| **Change Data Capture (CDC)** | Captures every insert/update/delete in source system; streams changes to Kafka. | Enables real-time data sharing without batch delays. Aladdin trade occurs → CDC captures event <100ms → Kafka topic → our systems react. |
| **Zero-Copy Data Sharing** | Consumer queries provider's data in-place without copying. Provider maintains data; consumer provides compute. | Instead of Nomura egressing Bloomberg data (expensive), Bloomberg subscriber (client) queries Bloomberg data directly via ADX. Zero egress charges. |
| **Data Domain** | Logical grouping of related data from single source/vendor. Usually owned by dedicated team. | Aladdin domain (1 engineer), Bloomberg domain (1 engineer), FactSet domain (1 engineer). Each domain owns: ingestion, quality, SLA monitoring, governance. |
| **GIPS** | Global Investment Performance Standards. Compliance framework for performance reporting. | Asset managers must report client returns in GIPS-compliant format annually. Requires reconciled data from Aladdin (holdings), Bloomberg (prices), internal cost accounting. |
| **FINRA CAT** | Financial Industry Regulatory Authority Central Analysis & Reporting. Mandatory trade reporting. | Every equity trade must be reported to FINRA within 1 hour. Failure = fines. Our system captures Aladdin trades, feeds to Kafka, Step Functions builds CAT XML, transmits via SFTP to FINRA. |
| **Medallion Architecture** | Data organization pattern: Bronze (raw) → Silver (cleaned) → Gold (analytics-ready). | Aladdin CDC events land in Bronze (raw Kafka); de-duplicated in Silver (RDS); stored as portfolio snapshots in Gold (S3 Parquet) for analytics. |
| **Row-Level Security (RLS)** | Database feature filtering query results by user role / tenant. | Client-A queries Redshift → RLS rule → sees only their fund data. Client-B queries same table → sees only their fund data. Same table, different results. |

---

## 12.1 Technical Terminology (Reviewed Earlier)

| Term | Definition |
|------|-----------|
| **ADX** | AWS Data Exchange - Marketplace for data products |
| **RBAC** | Role-Based Access Control for multi-tenancy |
| **MSK** | Amazon Managed Streaming for Apache Kafka |
| **RLS** | Row-Level Security - data isolation per client |
| **SLA** | Service Level Agreement - uptime and latency guarantees |
| **KPI** | Key Performance Indicator - business metrics |
| **PII** | Personally Identifiable Information - sensitive data |
| **ADR** | Architecture Decision Record - document to capture design decisions |
| **DoD** | Definition of Done - checklist before code ships to production |
| **ETL/ELT** | Extract-Transform-Load / Extract-Load-Transform - data integration patterns |

---

## 13. References & Learning Resources

- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [Confluent Kafka Best Practices](https://docs.confluent.io/)
- [Medallion Architecture](https://www.databricks.com/blog/2022/06/30/five-people-each-interpret-data-lakehouse-architecture-one-patterns.html)
- [Zero-Trust Security Model](https://www.nist.gov/publications/zero-trust-architecture)
- [Data as a Product](https://martinfowler.com/articles/data-mesh-principles.html)
- [Spring Boot Production Ready](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html)
- [AWS Data Exchange Developer Guide](https://docs.aws.amazon.com/data-exchange/latest/userguide/)
- [FINRA Regulatory Guidance](https://www.finra.org/rules-guidance)
- [Domain-Driven Design](https://www.domainlanguage.com/ddd/)
- [Gartner Data Mesh Principles](https://www.gartner.com/smarterwithgartner/what-is-a-data-mesh)

---

## 14. Contact & Support

For questions or contributions:
- **Technical Issues**: Open GitHub issue with label `bug` or `question`
- **Architecture Discussions**: Schedule sync with Senior Engineering Leader
- **Access Requests**: Contact infrastructure team via Slack #infrastructure-support
- **Security Concerns**: Email security@nomura.com (confidential)

### On-Call Escalation
- **Level 1**: Service team (first 15min)
- **Level 2**: Principal Engineer (if P1 issue)
- **Level 3**: VP of Engineering (if multiple services impacted)

---

**Last Updated**: February 18, 2026  
**Version**: 1.1 (Enhanced with Code Examples, ADRs, Runbooks, Onboarding)  
**Status**: Ready for Evaluation Cycle 2
