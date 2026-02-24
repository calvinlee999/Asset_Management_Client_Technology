# Tier 1: Digital Layer & Salesforce Integration
## React · Amplify · PowerBI · Athena · QuickSight · Salesforce Sales Cloud · Service Cloud
### Interview Architecture Document — Executive Director, Head of Client Technology
#### Target Audience: Shaw Levin (VP, Technology Partnerships) · Stuart Mumley (Director, Client Platforms) · Kathleen Hess McNamara (ED, Client Digital Solutions)

---

> **Strategic Purpose:** The Digital Layer is the presentation and distribution intelligence tier of the Nomura Asset Management platform. It connects a React/Amplify frontend to a gold-layer data lake, embeds institutional dashboards inside Salesforce Account pages, and uses AWS-Salesforce Zero-Copy to ensure every RM and Portfolio Manager has live, never-stale portfolio insights. Salesforce is not treated as a contact management system — it is the Distribution Intelligence Platform where client signals flow in, cross-sell opportunities flow out, and every client interaction is captured, measured, and actioned.

---

## Table of Contents

1. [Tier 1 Digital Layer Overview](#1-tier-1-digital-layer-overview)
2. [Salesforce Architecture: Sales Cloud for Institutional Distribution](#2-salesforce-architecture-sales-cloud-for-institutional-distribution)
3. [Salesforce Architecture: Service Cloud for Client Experience](#3-salesforce-architecture-service-cloud-for-client-experience)
4. [AWS-Salesforce Integration Blueprint: Zero-Copy Architecture](#4-aws-salesforce-integration-blueprint-zero-copy-architecture)
5. [Three-Layer CRM-to-Insights Architecture](#5-three-layer-crm-to-insights-architecture)
6. [Internal vs. Client-Facing Visualization: QuickSight and PowerBI](#6-internal-vs-client-facing-visualization-quicksight-and-powerbi)
7. [Strategic Use Cases: RM Alpha Discovery, Investor Portal Sync, Distribution Analytics](#7-strategic-use-cases-rm-alpha-discovery-investor-portal-sync-distribution-analytics)
8. [Proactive Risk Mitigation: The Lemons Tables](#8-proactive-risk-mitigation-the-lemons-tables)
9. [Interview Strategy: Talking Points for Shaw, Stuart, and Kathleen](#9-interview-strategy-talking-points-for-shaw-stuart-and-kathleen)

---

## 1. Tier 1 Digital Layer Overview

### Purpose in the Platform Stack

```
FOUR-TIER ARCHITECTURE
────────────────────────────────────────────────────────────────────
Tier 1 — DIGITAL LAYER (THIS DOCUMENT)
  → React / Amplify / PowerBI / QuickSight / Athena / Salesforce
  → Delivers live institutional analytics to RMs, Portfolio Managers,
    and institutional clients

Tier 2 — REST API
  → Python FastAPI / Kong Gateway / Apigee
  → Exposes NAV, risk, and holdings data via API

Tier 3 — KAFKA STREAMING
  → MSK / Avro / Schema Registry
  → Real-time market data and event streaming

Tier 4 — DATA DISCOVERY & ONBOARDING
  → AWS Glue / Lake Formation / S3 Medallion
  → Structured, governed data lake feeding Tier 1 analytics
```

### Tier 1 Technical Stack

| Layer | Technology | Responsibility |
|---|---|---|
| **Frontend Framework** | React + TypeScript on AWS Amplify | Responsive client portal; authenticated RM dashboard |
| **Self-Service Analytics** | Microsoft PowerBI Embedded | Self-service drill-down for institutional clients and Portfolio Managers |
| **Query Layer** | Amazon Athena + S3 Medallion (Gold Layer) | Serverless SQL against production-grade, governed data lake |
| **Internal Dashboards** | Amazon QuickSight SPICE | Engineering/SRE/FinOps dashboards for Shaw's team; platform health |
| **Identity & Access** | Amazon Cognito + SAML 2.0 federation | Enterprise SSO into all Tier 1 surfaces; RLS enforced per region/role |
| **CRM** | Salesforce Sales Cloud + Service Cloud | Distribution management, client servicing, collateral delivery |

### Key Performance Characteristics

| Dimension | Target | Architecture Mechanism |
|---|---|---|
| **Dashboard load latency** | <5 seconds to first insight | PowerBI DirectQuery on Athena Gold Layer; SPICE caching for QuickSight |
| **Multi-tenancy isolation** | APAC ≠ EMEA ≠ AMER | Row-Level Security (RLS) enforced at query submission layer via JWT |
| **Data freshness** | <15 min for GoldLayer; <5 min for SPICE refresh | Daily Glue job → Gold Layer; QuickSight SPICE incremental refresh |
| **Concurrent users** | 200+ RMs globally without degradation | Amplify auto-scaling; Athena queue management; SPICE in-memory processing |

### Tier 1 Data Products

| Data Product | Consumer | Update Frequency |
|---|---|---|
| NAV history + performance attribution | Institutional clients via PowerBI | Daily (post-market close) |
| Risk factor decomposition | Portfolio Managers | Intraday (near-real-time via Kafka → Athena) |
| Fee impact analysis | Compliance + Client Relations | Monthly |
| Client reporting templates (GIPS) | Institutional clients via Vermilion + PowerBI | Quarterly |
| RM pipeline opportunity data | Sales teams via Salesforce Einstein | Real-time (Salesforce native) |

---

## 2. Salesforce Architecture: Sales Cloud for Institutional Distribution

### Strategic Positioning

Sales Cloud is the institutional distribution engine — not a contact book. It connects RM activity to investment pipeline, surfaces AI-generated client insights, and automates collateral delivery at the moment of need.

### Core Sales Cloud Components

#### Einstein Relationship Insights (AI-Powered Relationship Mapping)
```
Account: Nomura Pension Fund (APAC Institutional)
    │
    ├── Primary Contact: CIO — Dr. [Name]
    │       └── Known influencers: Managing Director / Assistant CIO
    │
    ├── Einstein Insight: "New connection — MD attended Nomura Fixed Income webinar"
    │       └── Alert to RM: "MD engagement detected — suggest follow-up call"
    │
    └── Cross-referencing: LinkedIn / S&P Capital IQ / Internal meeting logs
```

**Business value:** RM identifies the right stakeholder before a relationship cools, not after.

#### Account Teams for Cross-Border Distribution
```
Account: [Institutional Client] — APAC + EMEA dual mandate

Account Team:
  → Primary RM (APAC)          : Full access
  → Secondary RM (EMEA)        : Read-only (RLS enforced)
  → Product Specialist (FI)    : Fund-level data access
  → Client Service Manager     : Case management and portal access

Cross-border flag: Visibility rules enforced by Salesforce Profile + Permission Set
GIPS report: Vermilion PDF auto-generated and attached to Account record via EventBridge
```

#### Activity Capture — The Anti-Lemon
```
LEMON: Sales teams manually logging calls, emails, and meeting notes into Salesforce
  → Leads to incomplete activity history → Einstein cannot learn → insights degrade

MITIGATION: Salesforce Activity Capture (previously Einstein Activity Capture)
  → Automatically logs all Outlook emails and calendar meetings to Account/Contact
  → RM focuses on the relationship; Salesforce captures the context automatically
  → Feeds Einstein Analytics with complete activity history for accurate AI scoring
```

#### High Velocity Sales for Distribution
```
USE CASE: Institutional client completing an onboarding form started 12 days ago

HVS Cadence:
  Day 0:  Form sent → Salesforce Task created → RM notified
  Day 3:  No response → Auto-follow-up email via Salesforce Engagement Studio
  Day 7:  No response → RM receives "At Risk" alert in HVS Work Queue
  Day 12: Escalation → Client Service Manager assigned via Omni-Channel routing

Outcome: Portal engagement rate increases because no onboarding falls through cracks
```

---

## 3. Salesforce Architecture: Service Cloud for Client Experience

### Strategic Positioning

Service Cloud is the operational excellence layer — every client inquiry, document request, and portal question is tracked, routed, and resolved with SLA visibility. It turns reactive client service into a measurable, repeatable process.

### Core Service Cloud Components

#### Entitlement Management for Private Market Data Access
```
Asset class: Private Equity / Private Credit / Venture Fund data

Entitlement: Client A = LP in Fund XVII only
  → Portal: Fund XVII performance data visible
  → Fund XVI data: Entitlement gate blocks access → "Access Restricted" message

Entitlement Milestones:
  → Onboarding document checklist → 100% complete before data visibility
  → AML/KYC verified (Salesforce + DocuSign integration)
  → Subscription agreement countersigned

Enforcement: Salesforce Entitlement Process + Experience Cloud permission sets
```

#### Case Management + Omni-Channel Routing
```
Incoming client inquiry: "Why is my NAV different from the factsheet?"

Case created: Salesforce Case record auto-generated
  → Case Type: NAV Discrepancy
  → Priority: HIGH (SLA = 4 hours)

Omni-Channel Routing:
  → Rule: NAV query → Route to SME Queue (Product Valuation team)
  → If no assignment in 30 min → Escalate to Portfolio Manager Queue
  → If no assignment in 4 hrs → Page on-call PM via PagerDuty integration

Resolution:
  → PM responds via internal portal → Case updated → Client notified via Experience Cloud
  → NCIE ingestion: Case closure notes → categorized by Bedrock as "resolved — transparent communication"
```

#### Experience Cloud: The Secure Client Portal
```
External access: Institutional clients login via SAML → Salesforce Experience Cloud
  → Embedded PowerBI dashboard (Assets under Management, performance attribution)
  → Vermilion GIPS PDF reports (auto-attached to client record)
  → Case submission form (direct to Salesforce Service Cloud)
  → Document library (prospectus, DDQ, fund factsheets)

RBAC enforced at Experience Cloud profile level:
  → LP investor: Only their fund's data visible
  → Institutional plan sponsor: All mandates visible
  → Default: no access (deny-by-default)
```

---

## 4. AWS-Salesforce Integration Blueprint: Zero-Copy Architecture

### The Three Integration Layers

```
LAYER 1: DATA LAYER (Zero-Copy Mesh)
────────────────────────────────────
Salesforce Data Cloud (Data 360)
     ↕                            Zero-Copy (no replication)
Amazon S3 Gold Layer
     ↕
Amazon Athena query

Nomura's live NAV, risk, and holdings data stays in S3.
Salesforce Data Cloud reads it directly via Zero-Copy connector.
No ETL pipeline, no data duplication, no staleness.

LAYER 2: PROCESS LAYER (Event-Driven Automation)
────────────────────────────────────────────────
Trigger: Opportunity Stage → Closed Won (new institutional mandate signed)
     ↓
Amazon EventBridge (Salesforce-to-AWS pipe)
     ↓
AWS Step Functions
  Step 1: Provision Vermilion reporting profile (fund-level configuration)
  Step 2: Provision PowerBI workspace (client-specific dashboard)
  Step 3: Provision OIDC SSO (Salesforce Experience Cloud → AWS identity)
  Step 4: Write provisioning confirmation back to Salesforce Account record
     ↓
RM notification: "Client portal provisioned — sharing link within 60 seconds"

LAYER 3: ENABLEMENT LAYER (Collateral + Insight Delivery)
──────────────────────────────────────────────────────────
Seismic for Agentforce:
  RM viewing Fixed Income prospect account in Salesforce
     ↓
  Seismic AI Agent surfaces approved pitch deck: "Q3 Fixed Income Outlook"
  Sourced from Seismic content library, tagged with: Asset Class=FI, Region=APAC
     ↓
  RM clicks "Send to Client" → Seismic tracks: opened, read 4 pages, shared with 2 contacts
     ↓
  Seismic engagement event → Salesforce Activity record auto-logged
     ↓
  Einstein Insight: "High engagement on FI pitch — suggest fund subscription conversation"

LWC Analytics Embedding:
  Salesforce Account page → Custom LWC component
     ↓
  PowerBI iFrame OR QuickSight embedded URL
  Parameter: Account_ID = [current Salesforce Account id]
     ↓
  Dashboard auto-filters: "Showing performance data for [Client name] mandates only"
  Context-aware: RM sees client's portfolio without navigating to a separate tool
```

### Zero-Copy Strategic Rationale

| Traditional ETL Pattern | Zero-Copy Pattern |
|---|---|
| Salesforce ↔ ETL pipeline ↔ S3 ↔ PowerBI | Salesforce Data Cloud + S3 (direct) + PowerBI |
| Data refreshes take 30–120 minutes | Live query — <5 seconds |
| Multiple copies of data across systems | Single source of truth in S3 Gold Layer |
| High maintenance: ETL failures, schema drift | Low maintenance: connector configuration only |
| Run Cost: ETL infrastructure + storage duplication | Run Cost: only compute time of query (Athena per-TB) |

---

## 5. Three-Layer CRM-to-Insights Architecture

### Layer 1: Presentation Layer — Single Pane of Glass

```
SALESFORCE ACCOUNT PAGE
───────────────────────
  [Account: Nomura Pension Fund LP]
  [CIO: Dr. [Name]] [Primary RM: [Name]] [Tier: Platinum Institutional]

  [PowerBI Dashboard Embedded via LWC]
  ┌─────────────────────────────────────────────────────┐
  │  FUND XVII — Performance Attribution YTD            │
  │  Total AuM: $450M  │  Benchmark: Russell 2000 +2.3% │
  │  Attribution: Technology overweight (+85bps)        │
  │  Risk: Active share 73%                             │
  │                                                     │
  │  [Export to Vermilion PDF]  [Open in PowerBI]       │
  └─────────────────────────────────────────────────────┘

  [Recent Activity: Auto-captured from Outlook]
  → Apr 14: Client call — 45 min — Fund XVII Q1 review
  → Apr 7: Email — Seismic FI Outlook deck shared, opened ×3

  [NCIE Alert — Bedrock generated]
  → "Client mentioned redemption process complexity in April portal feedback"
  → Action item: "Proactively clarify Private Credit redemption timeline in next call"
```

**Value:** The RM never leaves Salesforce. Every insight is contextually rendered around the Account.

### Layer 2: Data Layer — Zero-Copy Mesh

```
DATA FLOW (No ETL Required)
  Amazon S3 Gold Layer (Single source of truth)
        │
        ├──► Amazon Athena (query engine)
        │         ↓
        │    PowerBI DirectQuery → Client-facing dashboard
        │
        ├──► Salesforce Data Cloud (Zero-Copy) → CRM context enrichment
        │
        └──► Amazon QuickSight SPICE → Internal FinOps + SRE dashboards

CONSISTENCY GUARANTEE: All systems query the same S3 Gold Layer.
  If a NAV is updated in S3 at 18:04, PowerBI at 18:05 and Salesforce at 18:05 see
  the same value. No reconciliation, no staleness, no support tickets.
```

### Layer 3: Identity Layer — Secure Bridge

```
IDENTITY FLOW
  Salesforce as Identity Provider (SAML/OIDC)
        ↓
  User logs into Experience Cloud (Salesforce)
        ↓
  JWT token issued with Salesforce Role + Account_ID
        ↓
  Passed to PowerBI Embedded Service Principal
        ↓
  PowerBI RLS rule: "Show only data WHERE Account_ID = [token.accountId]"
        ↓
  QuickSight: Same Account_ID passed via embedded URL parameter
        ↓
  QuickSight RLS enforces: Asset Manager role ≠ LP investor role
        ↓
  APAC client cannot see EMEA mandate data — enforced at query, not UI
```

**Security principle:** Deny-by-default. Add-to-allow-list per entitlement. Never rely on UI hiding as a security control.

---

## 6. Internal vs. Client-Facing Visualization: QuickSight and PowerBI

### Amazon QuickSight — Internal Operational Intelligence

**Primary audience:** Shaw Levin (VP), SRE team, FinOps team, platform engineering

| Use Case | Dashboard Type | Data Source |
|---|---|---|
| **FinOps: AWS Cost by Service** | SPICE — bar chart with trending | AWS Cost Explorer API → S3 |
| **SRE: Platform Health** | Real-time metrics with alert overlay | CloudWatch metrics → QuickSight |
| **API Layer Performance** | Latency P50/P95/P99 by endpoint | Kong/Apigee access logs → Athena |
| **ADX Marketplace Activity** | Data product subscriptions by product | ADX API → Glue → S3 |
| **Kafka Stream Health** | Message lag by consumer group | MSK CloudWatch → QuickSight |

**Why QuickSight for Internal:**
- SPICE in-memory engine: Dashboards load in <1 second for up to 10M rows
- Session-based pricing: Internal users = 1 session/month fee — significantly cheaper than PowerBI per-user licensing for large teams
- Integrates natively with AWS Cost Explorer, CloudWatch, Glue Data Catalog
- No additional data export required — SPICE reads from S3 Glue-catalogued tables directly

### Microsoft PowerBI — Client-Facing Institutional Analytics

**Primary audience:** Portfolio Managers, Institutional Clients (via Experience Cloud), Distribution Team

| Use Case | Dashboard Type | Data Connection |
|---|---|---|
| **Portfolio Performance Attribution** | Time-series with factor overlay | DirectQuery on Athena Gold Layer |
| **Risk Factor Decomposition** | Heatmap + factor contribution waterfall | DirectQuery — live Athena |
| **Benchmark Comparison** | Scatter plot + active share visualization | DirectQuery + Bloomberg data table in S3 |
| **GIPS Client Reporting** | Standardized template → Vermilion PDF export | PowerBI Report Server + Vermilion API |
| **Fee Impact Analysis** | Before/after fee schedule scenario builder | DirectQuery — client fee config table in Athena |

**Why PowerBI for External:**
- DirectQuery on Athena Gold Layer: data is never stale (PowerBI queries S3 live, no SPICE cache for compliance-sensitive data)
- Client-grade UI/UX: institutional clients expect Microsoft-native tooling
- Embedded Service Principals: client-facing dashboards embedded into Experience Cloud without exposing Nomura BI credentials
- Vermilion integration: PowerBI snapshots fed into Vermilion for SEC/FCA-compliant GIPS PDF report generation
- Microsoft trust factor: institutional plan sponsors are familiar with PowerBI — no adoption friction

### QuickSight vs. PowerBI Decision Matrix

| Dimension | QuickSight | PowerBI |
|---|---|---|
| **Primary audience** | Internal (engineering, FinOps, SRE) | External (clients, RMs, PMs) |
| **Data connection model** | SPICE in-memory OR DirectQuery | DirectQuery on Athena Gold Layer |
| **Data freshness** | Configurable SPICE refresh (15 min–24 hrs) | Live (no cache — compliance-grade) |
| **Licensing model** | Session (per-user/month) — FinOps friendly | Capacity (Power BI Premium P-SKU) |
| **Integration** | Native to LTGM stack / AWS Cost Explorer | Native to Microsoft Fabric / Vermilion |
| **Embedding target** | Internal tools / Shaw's operational dashboards | Salesforce Experience Cloud LWC |
| **Run Cost** | Lower for large internal user base | Higher but offset by client experience premium |

---

## 7. Strategic Use Cases: RM Alpha Discovery, Investor Portal Sync, Distribution Analytics

### Use Case 1: RM Alpha Discovery — Under-Allocated Client Detection

```
WORKFLOW
PowerBI identifies clients under-allocated to Fixed Income mandates
  → Client A: AuM $300M — only 8% FI exposure vs. benchmark 15%
  → Client B: AuM $180M — no Private Credit allocation in APAC LP segment
        ↓
PowerBI → Salesforce Integration (Bidirectional)
  → PowerBI dataset writes "Suggested Opportunity" flag to Salesforce via REST API
        ↓
Salesforce: New Task created on Account record
  → "Opportunity: Increase FI allocation — currently 8% vs benchmark 15%"
  → Task assigned to RM [Name]
  → Seismic: Automatically attaches "Fixed Income Market Outlook — Q3" from content library
        ↓
RM ACTION: One-click to initiate fund allocation conversation
  → Email drafted by Einstein GPT: "I noticed your FI allocation has drifted below target..."
  → Seismic engagement tracking activated on pitch deck

OUTCOME METRIC: Number of "Suggested Tasks" converted to "Opportunity-In-Progress" within 30 days
```

### Use Case 2: Investor Portal Sync — Client Engagement to Salesforce Activity

```
WORKFLOW
Client downloads Q3 performance report from Nomura Investor Portal
        ↓
AWS EventBridge event: {type: "portal_document_download", doc_id: "Q3-2025-Fund-XVII", 
                        client_id: "acc-001", timestamp: "2025-10-14T16:22:00Z"}
        ↓
Lambda processes event: Maps client_id to Salesforce Account_ID
        ↓
Salesforce REST API: Create Activity record on Account
  → Activity Type: "Client Portal Engagement — Document Download"
  → Subject: "Fund XVII Q3 Report downloaded October 14, 2025"
  → Linked to: Contact [CIO name], Account [Client name]
        ↓
Einstein Insight triggered:
  → "Client showing post-quarter interest — Q3 review call not yet scheduled"
  → Alert to RM: "Suggest Q3 review call within 5 business days"

OUTCOME METRIC: Correlation between portal engagement events and RM call scheduling rate
```

### Use Case 3: Distribution Analytics — Opportunity × Seismic × QuickSight

```
WORKFLOW
Seismic: Tracks every pitch deck interaction (opened, time spent, pages viewed, forwarded)
        ↓
QuickSight Distribution Dashboard (Shaw's team):
  Rows: Each Seismic content piece (pitch decks, factsheets, ESG reports)
  Columns: Impressions / Opens / Time Spent / Forwarded / Opportunity Won rate
  Filter: By region, product, RM
        ↓
Insight Example:
  "APAC Fixed Income Outlook deck: opened by 47 institutional clients this month
   Average time: 4m 30s — Page 7 (Why Nomura FI?) has highest dwell time
   Correlated with 3 Closed-Won opportunities vs. 0 for EMEA version"
        ↓
ACTION: EMEA FI content team revises pitch deck to include "Why Nomura FI?" section
        ↓
Salesforce: Seismic engagement score embedded in Lead/Opportunity record
  → Einstein scores engagement: "High Intent — initiate fund subscription discussion"

OUTCOME METRIC: Content engagement-to-opportunity conversion rate by asset class and region
```

---

## 8. Proactive Risk Mitigation: The Lemons Tables

### Digital Layer Lemons

| Risk Area | Mitigation | Value Delivered |
|---|---|---|
| **PowerBI stale data** | DirectQuery on Athena Gold Layer — never caches compliance-critical data | "NAV shown to client is always current to last Glue job" — auditable guarantee |
| **LWC dashboard token exposure** | Service Principal credentials stored in AWS Secrets Manager; fetched at render time via Lambda; never stored in Salesforce | Zero-trust embedding pattern; credentials rotate every 24 hours automatically |
| **QuickSight SPICE capacity exhaustion** | SPICE datasets partitioned by region (APAC/EMEA/AMER); incremental refresh instead of full reload | 85% reduction in SPICE refresh time; no "stale internal dashboard" complaints |
| **React Amplify downtime** | Amplify multi-region deployment behind CloudFront; circuit breaker pattern for API failures | Client portal stays live even if single region fails |
| **Athena query cost overrun** | Workgroups with per-query scan limits; Gold Layer data partitioned by date/region for predicate pushdown | Run Cost bounded; Shaw can forecast monthly Athena cost within 5% accuracy |

### Salesforce Integration Lemons

| Risk Area | Mitigation | Value Delivered |
|---|---|---|
| **Zero-Copy connector failure** | Fallback: scheduled Glue job writes a Gold Layer snapshot to Salesforce Data Cloud every 4 hours | Maximum staleness = 4 hours vs. risk of full data unavailability |
| **EventBridge Opportunity pipe duplication** | EventBridge rule targets Step Functions with idempotency token = SalesforceOpportunityId + timestamp hash | Closed-Won provisioning runs exactly once, even if EventBridge retries |
| **Seismic content governance failure (PII or compliance breach)** | All Seismic content tagged with compliance review status; Agentforce only surfaces content with status = "Approved" | Compliance team retains content governance; AI does not surface draft documents |
| **Activity Capture missing sensitive conversations** | Activity Capture configured to exclude emails with "Confidential" or "Privileged" subject tags | Legal hold and sensitive communications management respected; audit trail clean |
| **NCIE → Salesforce Case duplication** | NCIE writes action items to Salesforce via upsert (not insert); external_id = NCIE action item hash | One Salesforce Case per NCIE action item — no duplicate cases per sprint cycle |

### Identity & Security Lemons

| Risk Area | Mitigation | Value Delivered |
|---|---|---|
| **JWT token reuse across client sessions** | Short-lived JWT (15-minute TTL); refresh token rotated per session | Client A cannot access Client B's data if JWT is intercepted |
| **RLS misconfiguration** | RLS rules tested via automated test suite (Pytest + Salesforce Apex Test classes); production deploy requires passing tests | "APAC ≠ EMEA" guarantee is programmatically verified, not manually reviewed |
| **Salesforce as IdP failure** | Fallback to Cognito User Pool with SAML bypass; monitored via CloudWatch alarm | Single point of failure in authentication chain eliminated |

---

## 9. Interview Strategy: Talking Points for Shaw, Stuart, and Kathleen

### For Shaw Levin — Technology Partnerships

**The Hook:**
> *"Shaw, the friction I want to eliminate is the one where an RM switches between Salesforce for the client record, PowerBI for the performance data, and a separate portal for the GIPS PDF. That's three systems, three logouts, and three contexts to manage before a single client conversation. I want to give RMs one screen — the Salesforce Account page — and render the entire client relationship there."*

**The Architecture Narrative:**
> *"I achieve this with Lightning Web Components embedding PowerBI directly into the Salesforce Account page. The Account_ID is passed as a parameter — no navigation required. The dashboard auto-filters to that client's mandates. The data is served via DirectQuery on our Athena Gold Layer, so it is never stale. A RM with 30 minutes before a client call has everything they need in one place — latest performance attribution, open cases, recent engagement history, and Seismic's top-ranked pitch deck for the account's asset class."*

**The FinOps Clincher:**
> *"For internal use — your SRE platform costs, API performance dashboards, streaming health — I use QuickSight SPICE. Session-based pricing means we pay a flat rate per RM per month, significantly under PowerBI per-user costs for large internal teams. QuickSight for us, PowerBI for them. Right tool for the right audience."*

---

### For Stuart Mumley — Client Platforms

**The Hook:**
> *"Stuart, the Salesforce platform I am describing is not a CRM deployment — it is a Distribution Intelligence Platform. The difference is that in a traditional CRM, Salesforce stores what happened. In a Distribution Intelligence Platform, Salesforce drives what should happen next."*

**The Architecture Narrative:**
> *"The mechanism is three integration layers: a Zero-Copy data layer where Salesforce Data Cloud reads live performance data directly from our S3 Gold Layer — no ETL, no stale snapshots. A process layer where Salesforce Opportunity stage changes trigger Step Functions to auto-provision Vermilion dashboards and SSO. And an enablement layer where Seismic for Agentforce surfaces the exact pitch deck at the moment an RM is preparing for a client call — without searching or asking for it."*

**The Platform Roadmap Angle:**
> *"The natural Phase 2 is connecting NCIE output into Salesforce Cases. Every NCIE action item — each backed by 47 source comments classified by Bedrock — becomes a trackable, assignable Salesforce Case in Stuart's platform backlog. That closes the loop: Kathleen's agent generates transcripts, NCIE synthesizes them, Stuart's team builds to the evidence, the platform improves, client satisfaction rises."*

---

### For Kathleen — Client Experience & AI

**The Hook:**
> *"Kathleen, the client portal and the AI Digital Agent create two data streams. The portal creates structured usage data — what they opened, what they downloaded, how long they spent on each document. The agent creates unstructured signal — what they asked, what the agent could not answer, and what caused them to abandon the chat. Currently, those two streams live in separate systems and no one is joining them."*

**The Architecture Narrative:**
> *"The Investor Portal Sync use case connects them. Every client portal action fires an EventBridge event that writes an Activity record to Salesforce. When a client downloads the Q3 report and then immediately starts a chat asking about redemption timelines, the RM's Salesforce timeline shows both events in sequence. That sequence — engagement + frustration — is the signal. And when NCIE ingests the fallback transcript, Bedrock identifies the knowledge gap that caused that frustration and raises it as a product action item."*

**The Clincher:**
> *"The result is that your client experience improvement process is no longer dependent on a RM manually noticing a pattern. The platform surfaces it. That is the shift from reactive client experience management to proactive experience engineering — and the AWS-Salesforce integration is the architecture that makes it work."*

---

## Executive Clincher: The Distribution Intelligence Platform Vision

> *"My approach to Salesforce at Nomura is not a contact management deployment — it is building a Distribution Intelligence Platform. AWS-Salesforce Zero-Copy turns our S3 Gold Layer into a real-time sales signal engine. Seismic for Agentforce ensures every RM has the right approved collateral at the exact moment they need it. LWC-embedded PowerBI dashboards put institutional portfolio analytics inside the client relationship context, not a separate tool. EventBridge pipes convert Opportunity closures into automated onboarding workflows. And NCIE closes the final loop — converting client frustration signals, including Kathleen's AI agent transcripts, into Stuart's product backlog. Every layer of this architecture exists to make Nomura's distribution teams faster, smarter, and more contextually aware at every client touchpoint."*

---

## Document Metadata

| Field | Value |
|---|---|
| **Prepared by** | Calvin Lee |
| **Target audience** | Shaw Levin (VP, Technology Partnerships), Stuart Mumley (Director, Client Platforms), Kathleen Hess McNamara (ED, Client Digital Solutions) |
| **Role** | ED, Head of Client Technology — Nomura Asset Management International |
| **Purpose** | Interview case study demonstrating Tier 1 Digital Layer + Salesforce Distribution Intelligence Platform architecture |
| **Related documents** | [README.md](README.md) · [BUSINESS_PARTNERSHIP_README.md](BUSINESS_PARTNERSHIP_README.md) · [NOMURA_CLIENT_INSIGHT_ENGINE.md](NOMURA_CLIENT_INSIGHT_ENGINE.md) · [BIAN_FRAMEWORK_ASSET_MANAGEMENT.md](BIAN_FRAMEWORK_ASSET_MANAGEMENT.md) |
| **Last updated** | February 2026 |

---

*This document demonstrates how Tier 1 is not a standalone frontend layer — it is the presentation tier of a fully integrated, event-driven platform where every client signal, distribution action, and product decision has a technical mechanism and a measurable outcome.*
