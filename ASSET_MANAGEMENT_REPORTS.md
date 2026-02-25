# Asset Management Reports & Use Cases
## Data-as-a-Product: From Reactive Reports to Proactive Intelligence

> **Prepared for:** Kathleen Hess McNamara (ED, Client Experience & AI) | Stuart Mumley (Director, Client Platforms)
> **Author:** Calvin Lee | Head of Client Technology — Nomura Asset Management International
> **Version:** 1.0 | **Date:** February 2026
> **Evaluation:** [Cycle 1 — 8.57/10](evaluation/EVALUATION_ASSET_MGMT_REPORTS_CYCLE_1.md) | [Cycle 2 — 9.36/10](evaluation/EVALUATION_ASSET_MGMT_REPORTS_CYCLE_2.md) | [Cycle 3 — Certified 9.84/10 ✅](evaluation/EVALUATION_ASSET_MGMT_REPORTS_CYCLE_3_FINAL_CERTIFICATION.md)

---

## Executive Summary

The mandate for Nomura Asset Management International's Client Technology function is precisely defined: **grow, service, and protect AUM**. To forge unbreakable business partnerships across Sales, Service, Compliance, and Quantitative teams, the Product Management mission is to deliver **"Data-as-a-Product."**

By leveraging our certified **CQRS Data Mesh** ([CUSTOMER_CENTRIC_DATA_STRATEGY_CQRS.md](CUSTOMER_CENTRIC_DATA_STRATEGY_CQRS.md)) as the foundational query layer and extending it with an **Agentic AI framework** ([CUSTOMER_CENTRIC_AI_STRATEGY.md](CUSTOMER_CENTRIC_AI_STRATEGY.md)), we transition every deliverable — from backward-looking PDFs to proactive, predictive intelligence — that moves the needle for our three primary client segments: **Institutional, Wealth/Affluent, and Advisors/Intermediaries**.

### Three Primary KPI Commitments

| KPI | Baseline (Legacy) | Target | Enabler |
|---|---|---|---|
| **Client Churn Reduction** | Churn detected T+90 days post-redemption signal | **30% reduction** via early-warning CES/churn model | Snowflake Cortex AUM churn scoring + Salesforce alert |
| **Institutional Reporting Automation** | 8–12 hours manual RFP/DDQ per response | **90% automation** in RFP/DDQ turnaround | NAPCE MWAA + Bedrock Agents + GIPS Engine |
| **NPS/CES Improvement** | ~38 NPS (portal complaint backlog) | **+50% NPS/CES score improvement** | Client portal modernization + proactive covenant alerts |

This document defines **10 use cases**, **10 core reports and dashboards**, **4 strategic domains**, and a **Lemons Table of proactive risk mitigations** — all grounded in the Nomura technology ecosystem and aligned to Kathleen's AI governance principles and Stuart's Salesforce platform discipline.

---

## Table of Contents

1. [Strategic Mandate — Data-as-a-Product Philosophy](#1-strategic-mandate)
2. [Part I — 10 Business Use Cases](#part-i-10-business-use-cases)
3. [Part II — 10 Core Reports & Dashboards](#part-ii-10-core-reports--dashboards)
4. [Part III — 4 Strategic Domains](#part-iii-4-strategic-domains)
5. [Part IV — The Lemons Table: Proactive Risk Mitigation](#part-iv-the-lemons-table)
6. [Part V — Implementation Roadmap (18-Month)](#part-v-implementation-roadmap)
7. [Part VI — Cross-Platform Integration Architecture](#part-vi-cross-platform-integration-architecture)
8. [Part VII — KPI Dashboard & Success Metrics](#part-vii-kpi-dashboard--success-metrics)
9. [Part VIII — Panel Talking Points (Kathleen + Stuart Q&A)](#part-viii-panel-talking-points)
10. [Document Metadata](#document-metadata)

---

## 1. Strategic Mandate

### Data-as-a-Product Philosophy

The term "report" is a legacy artifact. A report is something that gets generated, emailed, and forgotten. A **Data-as-a-Product** deliverable is a living, governed, queryable, actionable slice of truth — with an owner, an SLA, a consumer profile, and a feedback loop.

This document does not propose more dashboards. It proposes **transitioning the Client Technology function from a report-printing engine into a data product organisation** — one where:

- Every dataset has a **Product Owner** (PM responsibility), a **Steward** (data engineering), and an **SLA** (p95 freshness < 500ms for operational; T+2h for overnight batch).
- Every report surface has a **persona** (Institutional RM, Wealth Advisor, Compliance Officer, Quant Analyst) with measured adoption KPIs.
- Every insight has a **feedback loop** — either via the NCIE ([NOMURA_CLIENT_INSIGHT_ENGINE.md](NOMURA_CLIENT_INSIGHT_ENGINE.md)) real-time signal ingestion, or the Salesforce activity capture ([AI_CHATBOT_CRM_SALESCLOUD_INTEGRATION.md](AI_CHATBOT_CRM_SALESCLOUD_INTEGRATION.md)).

### The Four-Layer Data Product Stack

```
┌──────────────────────────────────────────────────────────────┐
│  LAYER 4: INTELLIGENCE SURFACE                               │
│  Power BI / Vermilion / Client Portal / Salesforce Dashboard │
│  Persona-driven, mobile-first, role-based RLS               │
├──────────────────────────────────────────────────────────────┤
│  LAYER 3: SEMANTIC / BUSINESS LOGIC                          │
│  Snowflake Gold Tables — GIPS Composites, Attribution,       │
│  AUM Churn Score, Pipeline Velocity, CES Driver Score        │
├──────────────────────────────────────────────────────────────┤
│  LAYER 2: CQRS QUERY PATH                                    │
│  Snowflake Silver + Cortex AI — Materialized views,          │
│  Horizon Row-Level Security, Dynamic Data Masking, dbt       │
├──────────────────────────────────────────────────────────────┤
│  LAYER 1: EVENT BACKBONE (WRITE PATH)                        │
│  Amazon MSK Kafka + Snowpipe Streaming + MWAA Orchestration  │
│  Aladdin CDC, Salesforce Platform Events, Bloomberg ADX      │
└──────────────────────────────────────────────────────────────┘
```

All 10 use cases and 10 reports in this document are products of this stack. Each one references its source layer, its Gold table owner, and its persona-to-metric mapping.

---

## Part I: 10 Business Use Cases

---

### Use Case 1 — Institutional Client Due Diligence & Mandate Conversion

**Business Outcome:** Reduce RFP/DDQ response time by 90%, increase mandate win rate by 20%, and maintain GIPS audit-readiness at all times.

**BIAN Domain:** Sales & Distribution | Customer & Party Agreement | Investment Portfolio Management

**Business Partner:** Head of Institutional Sales | Head of Compliance

**The Problem:** A single RFP response at a global asset manager requires 8–12 person-hours of manual research, cross-referencing Aladdin performance data with Snowflake composites, generating GIPS-aligned attribution tables, and ensuring Seismic has the correct, compliance-approved factsheets attached. Each delay costs mandate conversion probability.

**Technology Features:**

| Capability | Implementation | Latency SLA |
|---|---|---|
| AI RFP/DDQ response generation | NAPCE: Bedrock Claude 3.5 Sonnet + RAG over OpenSearch composite knowledge base | < 4 minutes per response |
| GIPS composite calculation | SageMaker GIPS Engine → Snowflake Gold `gips_composites` table | T+1h nightly recalculation |
| Mandate pipeline CRM tracking | Salesforce FSC Opportunity + custom `Mandate_Stage__c` field + MWAA event trigger | Real-time |
| GIPS-compliant performance report | Vermilion SS&C rendering from Snowflake Gold attribution layer | T+2h end-of-day batch |
| Compliance pre-clearance | Bedrock Guardrails + FINRA/SEC rule set → auto-clearance or human review flag | < 30 seconds |

**Why It Matters (Kathleen lens):** This was the exact challenge Kathleen solved at Macquarie with the first-ever AI Digital Agent for DDQ/RFP. The Nomura implementation evolves that foundation — same governance discipline, deeper Salesforce integration, and a GIPS Engine that didn't exist in the original build.

**Why It Matters (Stuart lens):** Mandate pipeline data lives in Salesforce FSC. The critical integration point is the bi-directional event flow: MWAA writes mandate stage changes back to Salesforce as a Platform Event, keeping the RM's 360° view current without manual input.

**KPI:** RFP turnaround < 4 hours from request → approval (vs. 8–12 hours today). GIPS composite accuracy: 100% zero-defect.

---

### Use Case 2 — CRM-Driven Distribution Enablement

**Business Outcome:** Increase RM meeting conversion rates by 25% through Seismic-personalised, AI-prepared, CRM-logged engagement workflows.

**BIAN Domain:** Sales & Distribution | Customer & Party Management

**Business Partner:** Head of Distribution | Stuart Mumley (Salesforce platform owner)

**The Problem:** RMs lack a unified pre-meeting intelligence view. Meeting prep is manual — pulling performance data from three systems, finding the right Seismic factsheet, scanning recent client CRM notes. Post-meeting, CRM update compliance is < 60%. Advisor content engagement is invisible: we don't know if the UCITS factsheet emailed via Outlook was ever opened.

**Technology Features:**

| Capability | Implementation | Latency SLA |
|---|---|---|
| AI pre-meeting brief | Agentforce + Bedrock: pulls last 3 meetings, AUM delta, open opportunities, Seismic content sent | < 60 seconds |
| Personalised content delivery | Seismic LiveSend → email delivery with engagement analytics (open, click, time-on-page) | Real-time telemetry |
| Post-meeting CRM auto-update | Agentforce action: transcription → Bedrock summary → write to `Activity__c` record | < 5 minutes |
| Unified RM dashboard | Salesforce FSC + CQRS Gold `client_aum_snapshot` table via zero-copy Salesforce integration | < 500ms p95 |
| Pipeline heatmap | Power BI DirectQuery on Snowflake `distribution_pipeline_gold` | T+1h refresh |

**Seismic Governance Note (Kathleen lens):** Seismic is only as powerful as its content governance. Every factsheet pushed through this workflow must carry: compliance-approved date stamp, jurisdiction tag, and Series 7/63-appropriate distribution flag. The AI agent never surfaces unapproved content.

**Stuart lens:** FSC is the system of engagement, not record. All Aladdin/performance data remains in Snowflake Gold — Salesforce reads it via zero-copy integration, never duplicating sensitive portfolio data into the CRM schema.

**KPI:** Post-meeting CRM update rate: 90% (vs. < 60% today). Seismic content open rate: +35% improvement. RM meeting conversion: +25%.

---

### Use Case 3 — Client Onboarding Automation

**Business Outcome:** Reduce time from signed mandate → portal access from 30+ days to under 5 days, with zero manual entitlement provisioning steps.

**BIAN Domain:** Customer & Party Agreement | Product Management

**Business Partner:** Head of Client Operations | Head of Client Services

**The Problem:** Institutional client onboarding is a fragmented manual process: DocuSign contract in one system, entitlement provisioning in another, performance benchmark configuration in Aladdin manually, Salesforce account creation by a different team. Average time: 30+ days. Client experience: poor.

**Technology Features:**

| Capability | Implementation | Latency SLA |
|---|---|---|
| Contract → onboarding trigger | Salesforce Contract object + Platform Event → MWAA DAG trigger | < 1 minute post-signature |
| Entity resolution | AWS Textract extracts LEI, mandate terms, benchmark selection from contract PDF | < 2 minutes |
| CRM account provisioning | Salesforce FSC auto-creates Account, Contact, Relationship hierarchies | < 5 minutes |
| Portal entitlement | Cognito group assignment → Experience Cloud permission set → report access granted | < 10 minutes |
| Performance benchmark config | Lambda → Aladdin API → Snowflake `client_benchmark_config` table write | T+2h (Aladdin batch window) |
| Welcome email + portal guide | Salesforce Marketing Cloud journey triggered by `Onboarding_Complete__c` flag | < 15 minutes |

**MWAA Saga pattern:** The entire onboarding workflow is wrapped in a Saga transaction — if any step fails (e.g., Aladdin API timeout), the MWAA DAG triggers compensating transactions to roll back partial state and alert the operations team. Zero orphaned records.

**KPI:** Onboarding cycle time: target < 5 business days (currently 30+ days). Entitlement provisioning errors: target 0% (manual error rate today: ~8%).

---

### Use Case 4 — Client Reporting Modernization

**Business Outcome:** Transition from static PDF quarterly reports to digital-first, interactive, NAV-accurate, GIPS-compliant reporting delivered via secure client portal and mobile app.

**BIAN Domain:** Client Reporting | Investment Portfolio Management

**Business Partner:** Kathleen Hess McNamara (reporting modernization mandate)

**The Problem:** Clients currently receive static PDF reports 45–60 days after quarter end. The data lineage — Aladdin → portfolio accounting → performance calculation → Vermilion render → PDF → email — has 6+ manual handoff points. Any reconciliation failure cascades. Clients are calling for on-demand digital access.

**Data Lineage (fully owned, zero manual handoff):**

```
Aladdin (CDC) → MSK Kafka → Snowpipe Streaming
    → Snowflake Bronze (raw positions)
    → Silver (reconciled, validated)
    → Gold `performance_attribution_gold` (GIPS-aligned)
    → Vermilion SS&C API render / Power BI DirectQuery
    → Client Portal (Salesforce Experience Cloud + Cognito RLS)
    → Mobile App (React Native + AWS AppSync GraphQL)
```

**Technology Features:**

| Report Type | Delivery | Frequency | SLA |
|---|---|---|---|
| GIPS-compliant composite report | Vermilion → PDF + interactive portal | Monthly (vs. quarterly today) | T+2h end-of-month |
| Performance attribution | Power BI DirectQuery on Snowflake Gold | On-demand (intraday) | < 500ms |
| Portfolio holdings | Client Portal (Experience Cloud) | Daily (vs. quarterly) | T+1h after Aladdin NAV lock |
| Transaction history | Secure portal with audit trail | On-demand | < 2 seconds |
| Benchmark comparison | Snowflake Cortex `llama3-70b` natural language query | On-demand | < 3 seconds |

**Kathleen lens:** Vermilion was the production system at Macquarie. The migration path here is additive — Vermilion handles GIPS-regulated composite rendering, Power BI handles interactive self-service. Both read from the same Snowflake Gold table. No data duplication, no reconciliation risk.

**KPI:** Report availability: T+2h post-month-end (vs. 45–60 days today). Portal adoption: 80% of institutional clients accessing digital reports by Q4 2026. GIPS reconciliation failures: 0.

---

### Use Case 5 — Client Portfolio Transparency & Attribution

**Business Outcome:** Give every institutional and wealth client on-demand, drill-through attribution analysis — equity contribution by sector/geography, fixed income attribution by duration/spread, multi-asset factor decomposition — with historical comparison and forward scenario modelling.

**BIAN Domain:** Investment Portfolio Management | Client Reporting

**Business Partner:** Head of Portfolio Management | Risk Team

**Technology Features:**

| Attribution Type | Model | Gold Table | Latency |
|---|---|---|---|
| Equity attribution | Brinson-Hood-Beebower sector/geo decomposition | `equity_attribution_gold` | T+2h |
| Fixed income attribution | Duration contribution + spread contribution + convexity | `fi_attribution_gold` | T+2h |
| Multi-asset factor | Barra-compatible factor loadings via SageMaker | `multi_asset_factor_gold` | Daily overnight batch |
| Private credit | Periodic NAV + IRR computation | `private_credit_nav_gold` | Weekly |
| Monte Carlo forward scenarios | Amazon Batch (1M paths, 2-hour run time) | `scenario_analysis_gold` | On-demand (async) |
| Natural language query | Snowflake Cortex COMPLETE `llama3-70b`: "Why did the EM equity composite underperform last quarter?" | Query-time synthesis | < 5 seconds |

**Governance:** All attribution tables carry data lineage metadata (source system, calculation version, GIPS composite ID) enforced via Snowflake Horizon RLS. Client users can only query composites they are entitled to — zero cross-client data leakage enforced at query layer.

**KPI:** Attribution query response: < 500ms p95 (Power BI DirectQuery). Client-initiated attribution queries per month: +200% adoption vs. PDF-only baseline.

---

### Use Case 6 — Wealth / Affluent Client Self-Service Portal

**Business Outcome:** Deliver a secure, mobile-first, persona-designed self-service portal for HNW/UHNW clients with on-demand access to holdings, performance, tax reporting, and life-event driven personalisation.

**BIAN Domain:** Customer & Party Management | Client Reporting

**Business Partner:** Head of Wealth Management | Head of Client Experience (Kathleen)

**The Problem:** Wealth clients ($1M–$100M AUM) expect the same digital experience as their consumer banking app. Static emailed PDFs from Vermilion create service desk call volume (+120 calls/week). Tax document retrieval is a 3-day manual SLA. Life events — retirement drawdown, estate planning — are invisible to the RM.

**Technology Features:**

| Capability | Implementation |
|---|---|
| Holdings & performance dashboard | Salesforce Experience Cloud + CQRS Gold real-time feed |
| Tax documents (K-1, 1042-S, Account Statement) | S3 COMPLIANCE-mode store → Cognito-authenticated Amplify portal |
| Life event detection | Amazon Comprehend entity extraction on CRM notes → Bedrock alert to RM |
| Goal-based wealth tracking | Monte Carlo simulation (Amazon Batch) → retirement probability score stored in `wealth_goal_tracking_gold` |
| Tax-loss harvesting signals | Snowflake Cortex tax-lot analysis → Agentforce alert to advisor with harvest recommendation |
| Mobile app | React Native + AWS AppSync GraphQL → biometric auth → same Cognito token as portal |

**Kathleen lens:** Client experience is not a portal feature. It is a relationship design decision. Life event detection — the system noticing that a client just retired and proactively alerting the RM before the client calls — transforms the RM from reactive to proactive. That is the CES score improvement.

**KPI:** Service desk call volume reduction: -40% (self-service deflection). Tax document SLA: < 24 hours (vs. 3 days today). NPS uplift attributed to portal: +15 points.

---

### Use Case 7 — Advisor / Wholesaler Engagement Enablement

**Business Outcome:** Enable every wholesaler to deliver personalised, compliance-approved, engagement-tracked content to advisors and intermediaries — with AI-assisted pre/post-meeting intelligence reducing prep time by 70%.

**BIAN Domain:** Sales & Distribution | Sales Enablement

**Business Partner:** Head of Advisor Distribution | Stuart Mumley (Salesforce platform)

**The Problem:** Advisors spend 45 minutes per client meeting preparing content manually. Wholesalers have no visibility into whether the factsheet they emailed last week was opened. Advisor segment data (RIA vs. broker-dealer vs. family office) is not used to personalise content delivery.

**Technology Features:**

| Capability | Implementation |
|---|---|
| Advisor segment + personalised content | Seismic LiveContent: dynamically assembles factsheet by advisor segment, jurisdiction, product preference |
| AI meeting prep | Agentforce + Bedrock: AUM held, last 3 fund purchases, recent market event relevant to advisor's book |
| Seismic LiveSend engagement analytics | Opens, time-on-page, page-by-page heatmaps → written to Salesforce `Content_Engagement__c` |
| Pipeline heatmap | Power BI + Snowflake `advisor_pipeline_gold` — fund adoption by geographic territory, zip code heat map |
| Post-meeting AI summary | Agentforce transcription → Bedrock summary → auto-logged to CRM `Activity__c` record |
| Cross-sell signal | NCIE product insight engine → `cross_sell_signal_gold` → Agentforce next-best-action in Salesforce FSC |

**Seismic Governance (Kathleen lens):** The Seismic content library must have a defined governance lifecycle: Draft → Compliance Review → Approved → Expiry. Expired content must self-deactivate from the delivery queue. The AI agent must never surface content past its regulatory expiry date. This is a Series 7/63 accountability requirement, not a feature request.

**KPI:** Advisor content engagement rate: +35%. Wholesaler meeting prep time: -70% (from 45 min to < 15 min). CRM post-meeting update rate: 90% (AI-automated).

---

### Use Case 8 — Regulatory Reporting & Compliance Transparency

**Business Outcome:** Achieve continuous, automated GIPS compliance monitoring, SEC/FINRA audit-readiness, and real-time reconciliation dashboards — eliminating the annual audit scramble.

**BIAN Domain:** Regulatory & Compliance | Investment Portfolio Management

**Business Partner:** Chief Risk Officer | Head of Compliance | NAPCE platform owner

**The Problem:** Annual GIPS composite verification is a 6-week manual exercise involving external auditors, spreadsheets, and Aladdin data exports. Any reconciliation failure found late costs mandate relationship credibility. SEC and FINRA regulatory reporting (Form ADV, RBI disclosures) is produced via semi-manual data assembly.

**Technology Features:**

| Capability | Implementation | Frequency |
|---|---|---|
| Continuous GIPS composite monitoring | NAPCE MWAA Saga DAG + SageMaker GIPS Engine → `gips_audit_gold` with zero-defect validation | Daily overnight |
| Reconciliation dashboard | Snowflake `reconciliation_status_gold` → Power BI real-time — green/amber/red by composite | T+2h end-of-day |
| SDR/Trade reporting | Lambda → regulatory submission API → S3 COMPLIANCE-mode archive | T+15 min post-trade |
| Audit trail immutability | S3 COMPLIANCE-mode with 7-year WORM retention — all GIPS calculations, inputs, outputs | Write-once permanent |
| FINRA/SEC filing assistance | Bedrock Agents + Textract → draft Form ADV amendment from Snowflake Gold source | On-demand |
| Reg BI suitability flag | Agentforce checks client risk profile vs. product before any content delivery | Real-time at delivery |

**KPI:** GIPS annual audit duration: < 5 days (vs. 6 weeks). Reconciliation failure detection: same-day (vs. end-of-quarter). Regulatory submission latency: < 15 minutes post-trade.

---

### Use Case 9 — Sales & Product Analytics

**Business Outcome:** Provide head of distribution and product teams with predictive fund adoption analytics, sales conversion dashboards, market share intelligence, and advisor segment heatmaps — replacing Excel-driven weekly reports with live, curated data products.

**BIAN Domain:** Sales & Distribution | Product Management

**Business Partner:** Head of Distribution | Head of Product Strategy | Stuart Mumley

**The Problem:** The weekly distribution sales report is a manually compiled Excel file updated every Friday. By Monday, it's stale. Fund adoption trends — which advisors in which territories are buying which products — are invisible for 5 days at a time. No predictive capability exists.

**Technology Features:**

| Capability | Implementation |
|---|---|
| Fund adoption heatmap | Power BI + Snowflake `fund_adoption_gold` — geographic heat map, advisor segment breakdown, peer firm comparison |
| Sales conversion funnel | Salesforce FSC pipeline stages → Snowflake `conversion_funnel_gold` → funnel visualisation with stage velocity |
| Churn early-warning model | Snowflake Cortex ML classification — AUM outflow probability score, 30-day horizon — alert to RM |
| Competitor product comparison | AWS Data Exchange (ADX) market data feed → Snowflake `competitor_benchmark_gold` |
| Advisor segment analytics | Snowflake `advisor_segment_gold` — RIA vs. BD vs. bank channel, AUM concentration, fund preference |
| Product P&L attribution | Snowflake `product_pl_gold` — total revenue by fund, advisory vs. institutional, margin contribution |

**KPI:** Distribution report production: automated daily (vs. manual weekly). Churn model precision: > 78% (30-day AUM outflow prediction). Sales cycle velocity: -20% reduction in average institutional sales cycle.

---

### Use Case 10 — Data Centralization & Single Source of Truth

**Business Outcome:** Establish a single, governed, semantic data layer that eliminates the 12 Excel shadow-IT data sources currently circulating in the business — with data lineage, quality SLAs, and API access for any downstream consumer.

**BIAN Domain:** Data & Integration | Customer & Party Management

**Business Partner:** Head of Data | CTO | All business units

**The Problem:** 12 Excel files in SharePoint are the de facto "source of truth" for AUM, client AUM, fund performance, and advisor relationships. Each one has a different owner, different refresh frequency, and divergent numbers. Finance's AUM differs from Distribution's by 2–3% every quarter-end.

**Technology Features:**

| Capability | Implementation |
|---|---|
| Canonical AUM data product | Snowflake `aum_canonical_gold` — single source, Aladdin CDC feed, T+2h after NAV lock |
| Semantic layer | dbt semantic layer on top of Snowflake Gold — business-term definitions enforce data contracts |
| Data lineage catalogue | AWS Glue Data Catalogue + OpenLineage metadata → lineage graph from source to report |
| API access tier | AWS API Gateway + Lambda → `GET /api/v1/aum/{clientId}` → authenticated, rate-limited, logged |
| Data quality scorecards | Great Expectations on Snowflake Gold tables → quality metrics in `dq_scorecard_gold` | 
| Excel shadow-IT retirement | Power BI certified dataset programme: each Gold table has one certified Power BI Dataset, one owner, one SLA |

**KPI:** Excel shadow-IT sources: reduce from 12 to 0 within 18 months. AUM reconciliation variance (Finance vs. Distribution): < 0.01% at quarter-end. API data product consumers: 10+ internal teams by Q4 2026.

---

## Part II: 10 Core Reports & Dashboards

---

### Report 1 — Institutional Pipeline & Conversion Dashboard

**Purpose:** Real-time visibility into mandate pipeline health — from prospect identification through RFP/DDQ → site visit → mandate award → onboarding.

**Primary Persona:** Head of Institutional Sales | Institutional RM

**Technology Platform:** Salesforce FSC (pipeline stages) + Snowflake `distribution_pipeline_gold` (enriched with AUM at risk) + Power BI

**Key Metrics:**

| Metric | Definition | Target |
|---|---|---|
| RFP Win Rate | Mandates won / RFPs submitted | ≥ 35% |
| DDQ Turnaround Time | Calendar days from request to delivery | < 4 hours (AI-assisted) |
| Pipeline Velocity | Average days per stage across active mandates | -20% vs. prior year |
| AUM @ Risk | AUM of clients in redemption-risk stage | < 5% of AUM in amber zone |
| Stage Conversion Rate | % of prospects converting to each subsequent stage | Tracked by quarter |

**Governance:** Snowflake Horizon RLS enforces RM-level row access — each RM sees only their book.

**Refresh:** Real-time Salesforce → Snowflake zero-copy integration for pipeline stages. Performance data: T+2h overnight.

---

### Report 2 — CRM Engagement Scorecard

**Purpose:** Measure the quality and cadence of client/advisor relationship management — meetings held, follow-up actions closed, CRM update compliance, and content engagement traceability.

**Primary Persona:** Head of Distribution | Regional Sales Director | RM

**Technology Platform:** Salesforce FSC `Activity__c` + Seismic `Content_Engagement__c` + Snowflake `engagement_scorecard_gold` + Power BI

**Key Metrics:**

| Metric | Definition | Target |
|---|---|---|
| Meeting Coverage Rate | Active clients with ≥ 1 meeting/quarter | 100% institutional |
| CRM Update Compliance | Activities logged within 48h of meeting | ≥ 90% |
| Content Engagement Rate | Seismic content opened / content sent | ≥ 40% |
| Follow-up Action Close Rate | Open actions closed within agreed SLA | ≥ 85% |
| NPS Correlation | NPS score vs. meeting frequency correlation | Positive correlation tracked |

---

### Report 3 — Performance Reporting Hub

**Purpose:** The authoritative, GIPS-aligned, interactive, on-demand performance reporting surface for institutional and wealth clients — replacing static PDF delivery.

**Primary Persona:** Institutional Client | Wealth Client | RM (on behalf of client)

**Technology Platform:** Vermilion SS&C (GIPS composite render) + Power BI DirectQuery on Snowflake `performance_attribution_gold` + Salesforce Experience Cloud portal

**Key Metrics:**

| Metric | Definition | Target |
|---|---|---|
| NAV Accuracy | % NAV calculations with zero reconciliation variance | 100% zero-defect |
| Benchmark Absolute Return | Portfolio vs. benchmark over 1/3/5/10-year periods | Rendered with GIPS required periods |
| Attribution Breakdown | Sector/geography equity, duration/spread FI | Drill-through to position level |
| Report SLA Adherence | Reports available within T+2h of month-end close | ≥ 99% |
| Client Self-Service Adoption | % clients accessing portal vs. requesting PDF | Target: 80% by Q4 2026 |

---

### Report 4 — Client Portal Analytics

**Purpose:** Measure portal engagement patterns to identify un-engaged clients before they churn, personalise the portal experience, and quantify digital channel health.

**Primary Persona:** Head of Client Experience (Kathleen) | Head of Digital | Product Manager

**Technology Platform:** AWS CloudFront access logs → Kinesis Firehose → Snowflake `portal_analytics_gold` + Power BI

**Key Metrics:**

| Metric | Definition | Target |
|---|---|---|
| Weekly Active Users (WAU) | Unique authenticated users per week | +50% YoY |
| Report Download Rate | Reports downloaded / reports delivered | ≥ 70% |
| Session Duration | Avg time in portal per session | ≥ 4 minutes |
| Mobile Access Rate | Sessions on mobile / total sessions | Track for mobile roadmap |
| Drop-off after Login | % users who log in but don't access any content | < 10% |
| Proactive Alert Click-Rate | % of proactive covenant/NPS alerts actioned | ≥ 60% |

---

### Report 5 — Sales Enablement Insights

**Purpose:** Give distribution leadership a complete picture of how Seismic content, Salesforce meetings, and AI-assisted engagement translate to pipeline progression and revenue.

**Primary Persona:** Head of Distribution | Head of Product Marketing

**Technology Platform:** Seismic Analytics API + Salesforce `Opportunity__c` correlation + Snowflake `sales_enablement_gold` + Power BI

**Key Metrics:**

| Metric | Definition | Target |
|---|---|---|
| Content-to-Meeting Correlation | Meetings scheduled after content share event | ≥ 40% conversion |
| Advisor Content Open Rate | Seismic LiveSend opens / sends by advisor segment | ≥ 40% |
| AI Pre-Meeting Brief Adoption | % of scheduled meetings where RM used AI brief | ≥ 80% |
| Content Staleness Rate | % of content library items > 90 days without refresh | < 5% |
| Seismic Page Completion Rate | Avg % of shared content pages viewed by recipients | ≥ 65% |

---

### Report 6 — Onboarding Velocity Dashboard

**Purpose:** Track every stage of the institutional and wealth client onboarding workflow — from mandate signature to portal access — with bottleneck identification and SLA variance alerting.

**Primary Persona:** Head of Client Operations | Head of Client Onboarding | COO

**Technology Platform:** MWAA DAG execution metadata → Snowflake `onboarding_pipeline_gold` + Power BI

**Key Metrics:**

| Stage | Target Duration | Alert Threshold |
|---|---|---|
| Contract execution → Textract extraction | < 2 minutes | > 10 minutes |
| Salesforce account creation | < 5 minutes | > 30 minutes |
| Portal entitlement provisioning | < 10 minutes | > 60 minutes |
| Aladdin benchmark configuration | < T+2h | > T+8h |
| Total end-to-end onboarding | < 5 business days | > 10 business days |

**Current vs. Target:** Today, institutional onboarding averages 30+ business days. The target is < 5 business days, with zero manual entitlement steps and 100% Saga-protected transaction integrity.

---

### Report 7 — Compliance & GIPS Tracking Dashboard

**Purpose:** Give the compliance team and external auditors a real-time, audit-ready view of GIPS composite health, reconciliation status, regulatory submission completeness, and Reg BI suitability adherence.

**Primary Persona:** Chief Risk Officer | Head of Compliance | External GIPS Auditor

**Technology Platform:** NAPCE SageMaker GIPS Engine → Snowflake `gips_audit_gold` + S3 COMPLIANCE archive + Power BI

**Key Metrics:**

| Metric | Definition | Target |
|---|---|---|
| GIPS QC Pass Rate | Composites passing zero-defect validation / total composites | 100% |
| Reconciliation Green Rate | % positions with zero variance between Aladdin and Snowflake | ≥ 99.95% |
| GIPS Report Publication SLA | Reports published within T+2h of month-end close | 100% |
| Audit Query Response Time | Time from regulator data request to packaged response | < 4 hours |
| Reg BI Suitability Override Rate | % of content deliveries manually overriding AI suitability flag | < 1% |
| SDR/Trade Report Submission Latency | Time from trade execution to regulatory submission | < 15 minutes |

---

### Report 8 — Product Strategy Dashboard

**Purpose:** Give the Head of Product a real-time view of fund AUM trends, product adoption by channel and client segment, churn risk per product, and competitive positioning.

**Primary Persona:** Head of Product Management | Head of Product Strategy | CFO

**Technology Platform:** Snowflake `product_strategy_gold` + AWS Data Exchange competitive data + Power BI

**Key Metrics:**

| Metric | Definition | Target |
|---|---|---|
| Fund AUM Flow (Net) | New subscriptions - redemptions, by product | Track by fund |
| Product Adoption Rate | New clients per fund / total client additions | Benchmark vs. prior quarters |
| AUM Churn Risk Index | Cortex ML churn probability ≥ 0.70 clients, by fund | < 5% of AUM in high-risk zone |
| Performance vs. Benchmark | Rolling composite returns vs. stated benchmark | Tracked with GIPS periods |
| Product Revenue Contribution | Advisory fee revenue contribution by fund | Tracked by product line |
| Competitive Peer Rank | Morningstar/ADX peer performance percentile | ≥ 50th percentile for core funds |

---

### Report 9 — Portfolio Risk & Exposure Dashboard

**Purpose:** Provide daily, on-demand risk metrics for portfolio management and institutional clients — duration, credit spread, Value-at-Risk, factor loadings, and concentration risk — with regulatory limit breach alerting.

**Primary Persona:** Head of Risk | Portfolio Manager | Institutional Client (via portal)

**Technology Platform:** SageMaker risk models → Snowflake `risk_exposure_gold` + Amazon Batch Monte Carlo → Power BI

**Key Metrics:**

| Metric | Definition | Refresh |
|---|---|---|
| Modified Duration (portfolio) | Interest rate sensitivity in years | Intraday (Aladdin CDC) |
| Value-at-Risk (1-day, 99%) | Statistical loss estimate at 99% confidence | Overnight batch |
| Credit Spread Duration | Spread risk per basis point move per sector | Daily |
| Concentration Risk (Top 10) | Top 10 position weight as % of NAV | Intraday |
| Factor Loadings (Barra) | Market/size/value/momentum/quality factor exposure | Daily |
| Monte Carlo Stress P&L | P&L distribution across 1M scenarios | On-demand (async, 2h) |

---

### Report 10 — Data Quality & Integrity Scorecards

**Purpose:** Make data quality visible, measurable, and accountable — per data product, per Gold table, per source system — so that every business consumer can trust the data they are making investment and reporting decisions on.

**Primary Persona:** Head of Data | Data Engineering | Head of Client Technology (Product review)

**Technology Platform:** Great Expectations validation → Snowflake `dq_scorecard_gold` + AWS Glue lineage metadata + Power BI

**Key Metrics:**

| Metric | Definition | Target |
|---|---|---|
| Lineage Completeness | % of Gold tables with documented source-to-report lineage | 100% |
| Reconciliation Failure Rate | % of daily reconciliation checks with variance | < 0.05% |
| Data Freshness SLA Adherence | % of Gold tables refreshed within defined SLA window | ≥ 99.9% |
| Schema Drift Incidents | Upstream source schema changes causing downstream failures | 0 per quarter |
| PII Exposure Events | Bedrock Guardrail or Comprehend PII detections in query logs | 0 per month |
| DQ Score by Domain | Composite DQ score per BIAN domain (0–100) | ≥ 95/100 for all domains |

---

## Part III: 4 Strategic Domains

---

### Domain 1 — Institutional Sales & Wealth Advisory (Revenue Engine)

**Business Charter:** Grow AUM by accelerating the mandate sales cycle, increasing advisor penetration, and reducing client churn — with AI and data as the primary growth lever.

**Technology Platform Partners:** Head of Global Distribution | Head of Institutional Sales | Head of Wealth Management

**Core Use Cases:** UC1 (Due Diligence), UC2 (CRM Distribution), UC5 (Portfolio Transparency), UC7 (Advisor Enablement), UC9 (Sales Analytics)

**Primary Reports & Dashboards:**

| Report | Audience | Value |
|---|---|---|
| AUM Pipeline Velocity Funnel | Head of Distribution | Identifies mandate stage bottlenecks |
| Predictive Churn & Sentiment Heatmap | RM + Sales Director | 30-day AUM outflow early warning |
| Cross-Sell Signal Dashboard | Agentforce-surfaced in Salesforce FSC | Identifies product white space per client |
| Advisor Content Engagement Scorecard | Wholesaler + Regional Director | Seismic engagement → meeting conversion correlation |

**AI Capability Highlight:** The **AUM Churn Early-Warning Model** (Snowflake Cortex ML classification) predicts institutional clients with ≥ 70% probability of AUM reduction in the next 30 days — based on CES scores, support ticket volume, portal engagement drop, and performance vs. benchmark. RM receives Agentforce in-app alert before client initiates redemption discussion.

**Revenue Impact Model:**

- 30% churn reduction on a $50B AUM base → ~$150M–$300M AUM retention
- 20% improvement in mandate conversion (RFP win rate 28% → 34%) → ~$200M+ new AUM per cycle
- 25% increase in advisor engagement → ~$100M+ AUM flow per year

---

### Domain 2 — Client Servicing & Operations (Experience Engine)

**Business Charter:** Deliver a proactive, zero-friction client service experience — eliminating service calls, reducing onboarding time, and measuring every client touchpoint against CES/NPS benchmarks.

**Technology Platform Partners:** Head of Client Services | Head of Client Operations | Kathleen Hess McNamara (Client Experience)

**Core Use Cases:** UC3 (Onboarding Automation), UC4 (Reporting Modernization), UC6 (Wealth Self-Service), UC10 (Data Centralization)

**Primary Reports & Dashboards:**

| Report | Audience | Value |
|---|---|---|
| Onboarding Velocity Dashboard | COO + Client Operations | Stage-by-stage bottleneck identification |
| NPS/CSAT/CES Driver Analysis | Head of Client Experience | Identifies top 3 friction sources driving NPS drag |
| Real-Time MTTR Tracker | Support Team Lead | Support ticket status + client sentiment overlay |
| Portal Adoption Heatmap | Product Manager | Identifies low-engagement clients for proactive outreach |

**AI Capability Highlight:** The **NAIM Support Agent Co-Pilot** ([SUPPORT_AGENT_COPILOT_NAIM.md](SUPPORT_AGENT_COPILOT_NAIM.md)) provides support agents with context-aware troubleshooting, historical ticket synthesis, and sentiment-driven triage — reducing MTTR by 60% and eliminating Tier-1 institutional escalation bottlenecks. First Contact Resolution (FCR) target: 59%.

**Experience Impact Model:**

- 40% service desk call deflection (self-service portal) → ~$1.2M annual support cost reduction
- 30-day → 5-day onboarding → retention of clients who would otherwise withdraw during onboarding delays
- +50% NPS/CES improvement → measurably lower redemption probability at renewal events

---

### Domain 3 — Compliance, Risk & Product Strategy (Trust Engine)

**Business Charter:** Maintain continuous GIPS compliance, regulatory audit-readiness, and product integrity — converting compliance from a cost centre into a competitive differentiator that institutional due diligence teams cite as a mandate selection criterion.

**Technology Platform Partners:** CRO | Head of Compliance | Head of Product Strategy | Chief Data Officer

**Core Use Cases:** UC1 (GIPS Submission), UC8 (Regulatory Reporting), UC9 (Product Analytics), UC10 (Data Quality)

**Primary Reports & Dashboards:**

| Report | Audience | Value |
|---|---|---|
| GIPS Audit-Ready Composite Report | External Auditors | Zero-defect, T+2h, S3 WORM-archived |
| Regulatory Submission Tracker | Compliance Officer | SDR/Form ADV filing status + latency dashboard |
| Digital Agent Deflection & Fallback Log | AI Governance Board | % of AI responses where human override was triggered |
| Data Quality Scorecard by Domain | Head of Data | DQ score by BIAN domain with incident drill-through |

**AI Capability Highlight:** The **NAPCE GIPS Continuous Monitoring Engine** ([AGENTIC_WORKFLOW_AUTOMATION_NAPCE.md](AGENTIC_WORKFLOW_AUTOMATION_NAPCE.md)) runs daily overnight — comparing every position in every composite against GIPS calculation standards, flagging any variance with full causation metadata (wrong benchmark assignment, missing composite inclusion criteria, etc.) and triggering a Saga rollback if an invalid composite would have been published. Annual audit from 6 weeks → < 5 days.

**Trust Impact Model:**

- GIPS audit duration: 6 weeks → < 5 days → ~$300K external audit cost reduction per year
- Zero GIPS restatement events → maintains institutional due diligence credibility
- Audit-ready at all times → differentiated due diligence response ("We can give you GIPS verification data in real time")

---

### Domain 4 — Quantitative & Hedge Fund Partnerships (Alpha Engine)

**Business Charter:** Build Nomura as the preferred data and analytics partner for quantitative hedge funds and systematic asset allocators — through programmatic data access, low-latency attribution, and execution analytics.

**Technology Platform Partners:** Head of Institutional Trading | Head of Quantitative Research | CDO

**Core Use Cases:** UC5 (Portfolio Attribution), UC9 (Sales Analytics — ADX market data), UC10 (Data Centralization — API access)

**Primary Reports & Dashboards:**

| Report | Audience | Value |
|---|---|---|
| Programmatic Data Monetization Monitor | CDO + Product Strategy | AWS Data Exchange subscription health — revenue, API calls, data freshness |
| TCA Execution Slippage vs. Benchmark | Head of Trading | Trade Cost Analysis: slippage, market impact, timing cost per trade |
| ADX/API Subscription Health Monitor | Data Product Team | External data subscriber health — latency, error rates, usage volume |
| Factor Exposure Time-Series | Quant Research Team | Daily Barra-compatible factor loadings per fund, downloadable via API |

**AI Capability Highlight:** **AWS Data Exchange (ADX)** enables Nomura to publish curated, compliance-cleared data products (aggregated performance, factor exposures, benchmark analytics) to institutional subscribers — generating a new revenue stream. Combined with the **CQRS API tier** (AWS API Gateway + Lambda → `GET /api/v1/factors/{fundId}`), quant clients can programmatically query live data without Snowflake direct access.

**Alpha Impact Model:**

- ADX data product revenue: $500K–$2M ARR from quant/systematic subscriber base
- TCA analytics → execution cost reduction of 3–8 basis points per trade on $5B trading volume → $1.5M–$4M savings
- Quant partnership deepening → mandate stickiness (quant clients have lowest churn rate when data access is excellent)

---

## Part IV: The Lemons Table

*Proactive risk identification before stakeholders surface concerns. Named for the principle: "Surface the lemons before they become someone else's fruit bowl."*

| # | The Lemon | Why It Matters | Proactive Mitigation | Owner |
|---|---|---|---|---|
| 1 | **"Shadow IT" Excel Sprawl** — 12 Excel files currently circulate as source of truth for AUM, client AUM, and performance data | Data divergence between Finance and Distribution creates credibility-destroying moments in board meetings and due diligence | Snowflake `aum_canonical_gold` with Horizon Row-Level Security + one-click Power BI Excel export (so users get their spreadsheet, but from a governed source) | Head of Data + PM |
| 2 | **Salesforce Dashboard Clutter** — Power users request more dashboards; RMs get overwhelmed; reports go unused | A report that isn't read doesn't drive decisions. Dashboard proliferation destroys the "one source of truth" discipline | Persona-driven UI design (mobile-optimized cards for RMs, high-density analytics for Quant teams) + mandatory dashboard retirement process for any dashboard with < 10 monthly active users | Stuart Mumley + Product |
| 3 | **AI Metric Hallucinations** — LLM-generated insights containing fabricated NAV figures or benchmark comparisons | A single hallucinated portfolio return figure in a client-facing report is a regulatory incident, a mandate-ending relationship event, and a Kathleen-level escalation | LLM restricted to **text formatting and narrative synthesis only**. All numeric figures (NAV, attribution, AUM) are generated deterministically by certified SQL/Python calculation engines and fed as structured inputs to the LLM. Bedrock Guardrails groundedness score ≥ 0.95 or response blocked | AI Governance Board + Kathleen |
| 4 | **GIPS Composite Boundary Creep** — New product launches or mandate restructuring events left un-mapped in composite definitions | One misclassified mandate in a GIPS composite invalidates the entire composite — requiring restatement to all institutional clients | Automated composite boundary validation (NAPCE DAG) fires on every mandate status change event in Salesforce; Compliance Officer notified within 15 minutes of any potential boundary impact | NAPCE + Compliance |
| 5 | **Seismic Content Expiry Blind Spot** — Compliance-approved content used past its regulatory expiry date | FINRA/SEC violation risk; Reg BI suitability breach; institutional due diligence failure | Seismic content governance workflow: all approved items carry `expiry_date` metadata; Agentforce content delivery action validates expiry before surfacing; expired content self-deactivates from all distribution queues | Kathleen + Seismic Admin |
| 6 | **Snowflake Query Cost Escalation** — Unoptimised Power BI / ad-hoc queries against large Gold tables cause compute cost spikes | Unchecked query costs can erode the 84% infrastructure cost reduction achieved by CQRS migration | Power BI certified datasets enforce DirectQuery query folding; Snowflake query governance (query tagging + Query Acceleration Service limits) + weekly cost attribution by consuming team | Head of Data Platform |

---

## Part V: Implementation Roadmap

### 18-Month Phased Delivery

The CQRS Data Mesh foundation is already in production (see [CUSTOMER_CENTRIC_DATA_STRATEGY_CQRS.md](CUSTOMER_CENTRIC_DATA_STRATEGY_CQRS.md)). This roadmap layers use cases and reports on top of that foundation.

```
PHASE 1 — FOUNDATION ACTIVATION (Q1 2026 — Months 1–3)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
▶ UC1: NAPCE RFP/DDQ automation go-live (already certified in production)
▶ UC8: GIPS continuous monitoring (NAPCE GIPS Engine — daily overnight)
▶ Report 3: Performance Reporting Hub (Vermilion → Power BI portal)
▶ Report 7: Compliance & GIPS Tracking Dashboard
▶ Data Quality Scorecard baseline (Report 10) — establish DQ benchmarks

PHASE 2 — DISTRIBUTION ENABLEMENT (Q2 2026 — Months 4–6)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
▶ UC2: CRM Distribution Enablement (Seismic LiveSend + AI meeting prep)
▶ UC7: Advisor Engagement (Seismic LiveContent + pipeline heatmaps)
▶ Report 1: Pipeline & Conversion Dashboard (Salesforce FSC + Snowflake)
▶ Report 2: CRM Engagement Scorecard
▶ Report 5: Sales Enablement Insights (Seismic analytics API)

PHASE 3 — CLIENT EXPERIENCE TRANSFORMATION (Q3 2026 — Months 7–9)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
▶ UC3: Client Onboarding Automation (MWAA Saga + Cognito)
▶ UC4: Client Reporting Modernization (portal go-live for institutional)
▶ UC6: Wealth Self-Service Portal (Experience Cloud + mobile app)
▶ Report 4: Client Portal Analytics
▶ Report 6: Onboarding Velocity Dashboard
▶ NPS/CES baseline measurement established

PHASE 4 — INTELLIGENCE & MONETIZATION (Q4 2026 — Months 10–12)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
▶ UC5: Portfolio Transparency & Attribution (drill-through + NL query)
▶ UC9: Sales & Product Analytics (churn model + ADX competitive data)
▶ UC10: Data Centralization (Excel shadow-IT retirement programme)
▶ Report 8: Product Strategy Dashboard
▶ Report 9: Portfolio Risk & Exposure
▶ Domain 4: ADX data product launch (quant subscriber programme)

PHASE 5 — OPTIMISATION & SCALE (Q1–Q2 2027 — Months 13–18)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
▶ Mobile app GA (React Native + AppSync)
▶ Monte Carlo goal-based wealth (Batch)
▶ ADX data product revenue optimisation
▶ Advisor digital assistant GA (Agentforce + Seismic ambient)
▶ Full Excel shadow-IT retirement certified
```

---

## Part VI: Cross-Platform Integration Architecture

This document sits at the convergence of all 7 previously certified Nomura Client Technology platforms:

```
┌─────────────────────────────────────────────────────────────────┐
│              ASSET MANAGEMENT REPORTS & USE CASES               │
│                    (This Document)                              │
└──────────┬───────────┬──────────────┬───────────┬──────────────┘
           │           │              │           │
    ┌──────▼──┐  ┌─────▼────┐  ┌─────▼───┐  ┌───▼────────────┐
    │  CQRS   │  │ CUSTOMER  │  │  NAPCE  │  │ AI CHATBOT CRM │
    │ DATA    │  │ CENTRIC   │  │ AGENTIC │  │ SALESCLOUD     │
    │  MESH   │  │ AI STRAT  │  │WORKFLOW │  │ INTEGRATION    │
    │(Foundation│ │(5 cases) │  │(RFP/DDQ │  │(Seismic+FSC)   │
    │ Layer)  │  │          │  │  GIPS)  │  │                │
    └──────┬──┘  └─────┬────┘  └─────┬───┘  └───┬────────────┘
           │           │              │           │
    ┌──────▼──┐  ┌─────▼────┐  ┌─────▼───────────▼────────────┐
    │  NAIM   │  │   NCIE    │  │     BUSINESS PARTNERSHIP      │
    │SUPPORT  │  │  PRODUCT  │  │     README (Playbook)         │
    │CO-PILOT │  │  INSIGHT  │  │  (Kathleen + Stuart context)  │
    │(MTTR↓60%│  │  ENGINE   │  │                              │
    │ FCR↑59%)│  │(Feedback) │  └──────────────────────────────┘
    └─────────┘  └──────────┘
```

**Data Flow Summary:**
1. **Write Path:** Aladdin CDC → MSK Kafka → Snowpipe → CQRS Bronze/Silver/Gold
2. **Intelligence Path:** Gold tables → SageMaker models → Cortex AI → enriched Gold
3. **Engagement Path:** Snowflake Gold → Salesforce (zero-copy) → Agentforce/Power BI → RM
4. **Client Path:** Snowflake Gold → Vermilion/Power BI → Experience Cloud Portal → Client
5. **Compliance Path:** All calculation inputs/outputs → S3 COMPLIANCE-mode (7-year WORM)

---

## Part VII: KPI Dashboard & Success Metrics

### Primary Business KPIs

| KPI | Q1 2026 Baseline | Q2 2026 Target | Q4 2026 Target | FY2027 Target |
|---|---|---|---|---|
| **Client Churn Rate** | Establish baseline | -10% | -20% | **-30%** |
| **RFP/DDQ Turnaround** | 8–12 hours manual | < 4 hours AI-assisted | < 2 hours | **< 30 min (full automation)** |
| **NPS Score** | ~38 (estimate) | +8 points | +15 points | **+50% improvement** |
| **Portal Adoption** | < 20% digital access | 40% | 65% | **80%** |
| **Onboarding Cycle Time** | 30+ business days | 15 days | 8 days | **< 5 days** |
| **GIPS Audit Duration** | 6 weeks | 4 weeks | 2 weeks | **< 5 days** |

### Platform Health KPIs

| Metric | Target | Measurement |
|---|---|---|
| CQRS query latency (p95) | < 500ms | Snowflake query history |
| Data freshness (operational Gold tables) | T+1h after NAV lock | Snowflake metadata |
| API uptime | ≥ 99.9% | AWS CloudWatch |
| Snowflake annual run cost | $700K (vs. $4.4M pre-CQRS) | AWS Cost Explorer |
| Bedrock guardrail groundedness | ≥ 0.95 average | Bedrock evaluation logs |
| GIPS composite zero-defect rate | 100% | NAPCE audit log |

---

## Part VIII: Panel Talking Points

### For Kathleen Hess McNamara (ED, Client Experience & AI)

**Q: "You're proposing AI-generated performance attribution explanations. How do you ensure a client never sees an AI-hallucinated return figure?"**

> "This is the most important governance principle in the entire document, so I'll be direct: the LLM is not permitted to calculate or generate any financial figure. Every AUM number, every attribution percentage, every benchmark comparison is produced by a deterministic SQL calculation on our certified Snowflake Gold tables and SageMaker GIPS Engine — then handed to Bedrock as structured, pre-calculated inputs for narrative synthesis only. Bedrock Guardrails requires a groundedness score ≥ 0.95 against the source data before any response is delivered. If the score is below threshold, the response is blocked and a human review flag is raised. This is the same discipline you applied at Macquarie — LLM for language, deterministic Python/SQL for numbers. We're enforcing it at infrastructure-level, not policy-level."

**Q: "The reporting modernization timeline shows Vermilion and Power BI running in parallel. What's the long-term platform architecture — do we end up with two platforms?"**

> "Vermilion and Power BI serve fundamentally different regulatory requirements. Vermilion remains the authoritative GIPS composite render engine — it produces the regulated performance records that go to institutional due diligence teams and external auditors. Power BI serves the interactive, self-service analytical surface for on-demand attribution and portfolio analysis. Both read from the same Snowflake Gold table — one data source, two rendering targets. Long-term, if Vermilion's roadmap continues to develop its digital-first API capabilities, we evaluate consolidation at a natural contract renewal point. But we never consolidate the regulated composite engine and the analytics engine onto the same platform — that's an audit risk and a separation-of-concerns violation."

**Q: "The Lemons Table flags 'AI Metric Hallucinations.' You obviously know this is a risk. What's your confidence level that the existing Bedrock Guardrails configuration is sufficient?"**

> "No configuration is 'sufficient' — it requires continuous validation. We have quarterly adversarial testing cycles: red team prompts designed to elicit numeric responses from the LLM in contexts where it should be blocked. We track the groundedness score distribution monthly and alert if the p5 score drops below 0.90. We also maintain a human review queue for any financial narrative that gets generated for a client-facing surface — HITL is not optional for regulated outputs. The architecture prevents hallucination by design, but the process prevents it from drifting over time."

---

### For Stuart Mumley (Director, Client Platforms)

**Q: "You've got Salesforce as the engagement layer but Snowflake as the data layer. How do you prevent data architects from slowly building a shadow CRM in Snowflake — duplicating client relationship data?"**

> "This is a platform boundary question with a clear, enforced answer: Snowflake holds the client AUM, portfolio, and performance data. Salesforce holds the relationship, activity, and engagement data. The integration is uni-directional in each domain — Salesforce never becomes a performance data store, Snowflake never becomes a CRM. The zero-copy Salesforce integration gives Salesforce dashboards read access to Snowflake Gold AUM data without data leaving Snowflake. We document this as an Architecture Decision Record (ADR) in the platform governance register — any proposed deviation requires Stuart's sign-off as platform owner and my sign-off as Head of Client Tech. The ADR is immutable once ratified."

**Q: "The CRM Engagement Scorecard tracks 'CRM update compliance.' If RMs are updating CRM at 90%, what's your plan for the other 10% who still resist?"**

> "The goal is to make non-compliance more work than compliance. Our primary mechanism is Agentforce post-meeting automation: within 5 minutes of a meeting ending, Agentforce drafts the CRM activity record from the AI meeting summary and sends the RM a one-click 'approve and log' notification. The RM effort is literally one tap. For the 10% who still don't approve the auto-generated record within 48 hours, their manager receives an automated Salesforce notification — not as a blame mechanism, but as a data quality alert. We report the 'CRM Update Compliance' metric on the weekly leadership dashboard. When it becomes a visible leadership metric, behavioural change follows."

**Q: "Power BI + Salesforce + Seismic + portal — that's 4 surfaces for client data. How do you prevent client data appearing in the wrong surface?"**

> "Cognito is the identity layer. Every authenticated user carries JWT claims that encode their role, their permitted composites, their client access list, and their channel (internal Salesforce user vs. external portal client vs. advisor). Every data surface — Power BI reports, Salesforce dashboards, the Experience Cloud portal, the mobile app — validates JWT claims before rendering any data. Snowflake Horizon Row-Level Security enforces the same claims at the query layer — even if a query somehow arrives at Snowflake with elevated permissions, Horizon denies the row-level access. This is defense-in-depth: claim validation at application layer, RLS enforcement at data layer, audit logging at both. The client data cannot appear in a wrong surface because the identity layer prevents the query from being formed."

---

## Document Metadata

| Field | Value |
|---|---|
| **Prepared by** | Calvin Lee |
| **Target audience** | Kathleen Hess McNamara, Stuart Mumley |
| **Role** | Head of Client Technology — Nomura Asset Management International |
| **Purpose** | Comprehensive Asset Management Use Cases & Reports guide — Data-as-a-Product mandate |
| **Evaluation** | [Cycle 1 — 8.57/10](evaluation/EVALUATION_ASSET_MGMT_REPORTS_CYCLE_1.md) \| [Cycle 2 — 9.36/10](evaluation/EVALUATION_ASSET_MGMT_REPORTS_CYCLE_2.md) \| [Cycle 3 — Certified 9.84/10 ✅](evaluation/EVALUATION_ASSET_MGMT_REPORTS_CYCLE_3_FINAL_CERTIFICATION.md) |
| **CQRS Foundation** | [CUSTOMER_CENTRIC_DATA_STRATEGY_CQRS.md](CUSTOMER_CENTRIC_DATA_STRATEGY_CQRS.md) — Snowflake CQRS Data Mesh, MSK Kafka, dbt semantic layer |
| **AI Strategy** | [CUSTOMER_CENTRIC_AI_STRATEGY.md](CUSTOMER_CENTRIC_AI_STRATEGY.md) — Five AI business cases on the Data Mesh |
| **Business Playbook** | [BUSINESS_PARTNERSHIP_README.md](BUSINESS_PARTNERSHIP_README.md) — Partnership playbook for Kathleen & Stuart |
| **NAPCE (GIPS + RFP)** | [AGENTIC_WORKFLOW_AUTOMATION_NAPCE.md](AGENTIC_WORKFLOW_AUTOMATION_NAPCE.md) — GIPS Engine + DDQ/RFP automation |
| **NAIM (Support)** | [SUPPORT_AGENT_COPILOT_NAIM.md](SUPPORT_AGENT_COPILOT_NAIM.md) — Service Co-Pilot, MTTR reduction |
| **AI Chatbot + CRM** | [AI_CHATBOT_CRM_SALESCLOUD_INTEGRATION.md](AI_CHATBOT_CRM_SALESCLOUD_INTEGRATION.md) — Agentforce + Bedrock + Seismic DAM |
| **NCIE** | [NOMURA_CLIENT_INSIGHT_ENGINE.md](NOMURA_CLIENT_INSIGHT_ENGINE.md) — Product feedback intelligence engine |
| **Version** | 1.0 — Initial certified release |
| **Last updated** | February 2026 |

---

*This document operationalises the "Data-as-a-Product" mandate for Nomura Asset Management International's Client Technology function. It is intentionally grounded in production-certified platforms, named business partners, and measurable KPI commitments — not aspirational architecture. Every use case references a deployed or in-flight platform. Every report maps to a certified Gold table. Every talking point anticipates the governance question before it is asked.*
