# BIAN Framework: Asset Management Business Process Architecture
## Nomura Asset Management International — Client Technology Domain
### Role: Executive Director, Head of Client Technology

### What This Document Does for You

It gives you a credibility multiplier. Most technology candidates in this interview will talk about platforms — Salesforce, Seismic, Vermilion. You will talk about service domains — the business-recognized framework that explains why those platforms have the boundaries they do. That is an ED-level distinction.

The eight BIAN service domains covered and their direct connection to Kathleen and Stuart's work:

- **Customer & Party Management** → Why Salesforce is the system of engagement, not the system of record (Stuart's core concern)
- **Sales & Distribution** → The full institutional and advisor channel lifecycle with AI touchpoints mapped to exactly where Stuart and Kathleen have built
- **Client Onboarding & KYC** → Orchestration model reducing 4–8 weeks to under 3 business days
- **Investment Portfolio Management** → The data boundary between investment management and client technology — what you own vs. what Aladdin owns
- **Trade Execution & Settlement** → Kathleen's Credit Suisse world, mapped so you can converse with her on it
- **Client Reporting & Performance** → Vermilion, GIPS as a shared service, the reporting data lineage chain
- **Sales Enablement & Advisor Distribution** → Seismic, content governance, the insight that DDQ agent and Seismic should consume from one Content Governance service
- **Data & Integration** → The medallion architecture mapped to BIAN Collect/Consolidate/Distribute behaviors

The three end-to-end process flows give you complete narratives: prospect-to-first-report, advisor-engagement-to-fund-placement, and the AI DDQ agent as a BIAN service interaction chain.

The platform-to-BIAN mapping table is the interview cheat sheet — every platform Stuart and Kathleen own, mapped to its BIAN domain, behaviors, and integration boundary.

---

> **Purpose:** The Banking Industry Architecture Network (BIAN) provides a standardized, service-oriented framework that breaks complex banking and asset management functions into reusable, interoperable **Service Domains**. This document maps BIAN's service landscape to Nomura's client technology platform — giving the Head of Client Technology a shared vocabulary to discuss platform boundaries, service ownership, and integration patterns with both Kathleen (client experience, AI, reporting) and Stuart (Salesforce, platform roadmap, sales enablement).

---

## Table of Contents

1. [BIAN Fundamentals for Asset Management](#1-bian-fundamentals-for-asset-management)
2. [BIAN Service Landscape: Nomura Client Technology Map](#2-bian-service-landscape-nomura-client-technology-map)
3. [Service Domain 1: Customer & Party Management](#3-service-domain-1-customer--party-management)
4. [Service Domain 2: Sales & Distribution](#4-service-domain-2-sales--distribution)
5. [Service Domain 3: Client Onboarding & KYC](#5-service-domain-3-client-onboarding--kyc)
6. [Service Domain 4: Investment Portfolio Management](#6-service-domain-4-investment-portfolio-management)
7. [Service Domain 5: Trade Execution & Settlement](#7-service-domain-5-trade-execution--settlement)
8. [Service Domain 6: Client Reporting & Performance](#8-service-domain-6-client-reporting--performance)
9. [Service Domain 7: Sales Enablement & Advisor Distribution](#9-service-domain-7-sales-enablement--advisor-distribution)
10. [Service Domain 8: Data & Integration Architecture](#10-service-domain-8-data--integration-architecture)
11. [BIAN End-to-End Business Process Flows](#11-bian-end-to-end-business-process-flows)
12. [Platform-to-BIAN Mapping](#12-platform-to-bian-mapping)
13. [Interview Talking Points Using BIAN Vocabulary](#13-interview-talking-points-using-bian-vocabulary)

---

## 1. BIAN Fundamentals for Asset Management

### What BIAN Provides

BIAN defines a **Service Domain** as a single, discrete area of banking/financial services functionality with:
- A clearly defined **business purpose** and bounded context
- A single **Control Record** (the primary business object it manages)
- Defined **Behavior Qualifiers** (the actions it executes)
- Well-defined **Service Operations** (how it communicates with other domains)
- Independent deployability and replaceability

### Why BIAN Matters for the Head of Client Technology

In asset management, the failure mode of most platform architectures is **service sprawl** — CRM, reporting, data platforms, and AI tools accumulate undefined dependencies and overlapping ownership. BIAN provides the conceptual framework to:

- Define clear boundaries between what Salesforce owns vs. what the data layer owns
- Identify which service domains can be modernized independently
- Evaluate vendors against a standard service model rather than feature lists
- Communicate platform architecture to product leaders in business terms they recognize

### BIAN's Four Business Areas Relevant to Asset Management

| BIAN Business Area | Asset Management Relevance | Nomura Platform Layer |
|---|---|---|
| **Front Office** | Client acquisition, sales, distribution | Salesforce, Seismic, AI DDQ/RFP Agent |
| **Middle Office** | Portfolio management, risk, compliance | Aladdin, risk engines, GIPS |
| **Back Office** | Settlement, accounting, fund administration | Custodians, fund accounting systems |
| **Infrastructure** | Data, integration, reporting, security | Data mesh, APIs, Vermilion, PowerBI |

---

## 2. BIAN Service Landscape: Nomura Client Technology Map

```
╔═══════════════════════════════════════════════════════════════════════╗
║         NOMURA ASSET MANAGEMENT — BIAN SERVICE LANDSCAPE              ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  FRONT OFFICE — Client-Facing Services                                 ║
║  ┌───────────────┐  ┌──────────────────┐  ┌──────────────────────┐   ║
║  │ CUSTOMER &    │  │ SALES &          │  │ PRODUCT MANAGEMENT   │   ║
║  │ PARTY MGMT    │  │ DISTRIBUTION     │  │ (Fund Catalogue)     │   ║
║  │ [Salesforce]  │  │ [Salesforce +    │  │ [Seismic + Data]     │   ║
║  │               │  │  Seismic + AI]   │  │                      │   ║
║  └───────┬───────┘  └────────┬─────────┘  └──────────┬───────────┘  ║
║          │                   │                        │               ║
║  ┌───────▼───────────────────▼────────────────────────▼───────────┐  ║
║  │               CLIENT ONBOARDING & KYC                           │  ║
║  │         [Salesforce Service Cloud + Step Functions]             │  ║
║  └──────────────────────────────┬──────────────────────────────────┘  ║
║                                  │                                     ║
╠══════════════════════════════════╪════════════════════════════════════╣
║  MIDDLE OFFICE — Investment Operations                                 ║
║  ┌──────────────────┐  ┌─────────▼──────────┐  ┌──────────────────┐  ║
║  │ INVESTMENT       │  │ TRADE EXECUTION    │  │ RISK &           │  ║
║  │ PORTFOLIO MGMT   │  │ & SETTLEMENT       │  │ COMPLIANCE       │  ║
║  │ [Aladdin]        │  │ [Middle Office]    │  │ [Risk Engines]   │  ║
║  └────────┬─────────┘  └────────────────────┘  └──────────────────┘  ║
║           │                                                            ║
╠═══════════▼════════════════════════════════════════════════════════════╣
║  INFRASTRUCTURE — Data, Reporting, Integration                         ║
║  ┌───────────────────────────────────────────────────────────────┐    ║
║  │          DATA MANAGEMENT & INTEGRATION PLATFORM                │    ║
║  │    [S3/Snowflake Medallion + Kafka + APIs + ADX]               │    ║
║  └──────────┬─────────────────────┬──────────────────────────────┘    ║
║             │                     │                                    ║
║  ┌──────────▼──────────┐  ┌───────▼────────────┐  ┌───────────────┐  ║
║  │ CLIENT REPORTING    │  │ PERFORMANCE &      │  │ ENTITLEMENT   │  ║
║  │ [Vermilion/PowerBI] │  │ ATTRIBUTION        │  │ MANAGEMENT    │  ║
║  │                     │  │ [GIPS Engine]      │  │ [Lake Form.]  │  ║
║  └─────────────────────┘  └────────────────────┘  └───────────────┘  ║
╚═══════════════════════════════════════════════════════════════════════╝
```

---

## 3. Service Domain 1: Customer & Party Management

### BIAN Definition
**Control Record:** Customer Relationship
**Purpose:** Manage all information and interactions related to customers, prospects, and counterparties — the authoritative source for party identity, relationship hierarchy, and engagement history.

### Asset Management Context
"Parties" include institutional investors (pension funds, endowments, sovereign wealth funds), financial advisors, intermediaries (RIAs, broker-dealers), and internal relationship managers.

### BIAN Behavior Qualifiers

| Behavior Qualifier | Description | Nomura Platform |
|---|---|---|
| **Initiate** | Create new prospect or customer record | Salesforce — new account creation |
| **Update** | Maintain customer profile, contacts, preferences | Salesforce — account and contact management |
| **Retrieve** | Access customer info for RM meeting prep | Salesforce + AI pre-meeting tool (Stuart) |
| **Evaluate** | Assess relationship strength and engagement | Salesforce Einstein Relationship Insights |
| **Monitor** | Track relationship health signals and churn risk | Salesforce dashboards + engagement scoring |

### Service Interfaces (What This Domain Provides to Others)

```
Customer & Party Management
        ├──► Sales & Distribution
        │    (Party identity and relationship context for pipeline)
        ├──► Client Onboarding & KYC
        │    (Customer profile as input to onboarding workflow)
        ├──► Client Reporting
        │    (Account context for report personalization and delivery)
        └──► Entitlement Management
             (Party identity as basis for data access control)
```

### Platform Alignment: Salesforce as Customer & Party Service Domain

Salesforce is the **system of engagement** — not the system of record. Party master data (legal entity identifiers, account hierarchy, tax IDs) is owned by a reference data system. Salesforce enriches that data with relationship history and pipeline context.

**BIAN-aligned integration boundary:**
```
Reference Data System (system of record — party identity)
        ↓ (read-only feed via governed API)
Salesforce FSC
  → Stores:  meetings, tasks, pipeline stages, RM assignments
  → Reads:   legal name, entity type, account hierarchy
  → Never:   portfolio positions, NAV, performance data
```

**Interview talking point:**
> *"In BIAN terms, Salesforce is our Customer & Party Management service domain — it owns relationship context and engagement history. But party master data lives upstream in a reference data service. Keeping that boundary clean is what prevents the data quality problems that surface when CRM becomes the system of record for things it shouldn't own."*

---

## 4. Service Domain 2: Sales & Distribution

### BIAN Definition
**Control Record:** Sales Plan
**Purpose:** Manage the end-to-end sales process from prospect identification through mandate award, including pipeline management, opportunity tracking, and distribution channel management.

### Asset Management Context: Two Distinct Channels

**Institutional Channel:** Long-cycle mandate pursuit (12–24 months), RFP/DDQ-driven, RM-managed with pension funds, endowments, and sovereign wealth funds.

**Advisor/Wealth Channel:** Wholesaler-driven advisor engagement, content-led relationship building, Seismic-powered distribution to RIAs and wirehouse advisors.

### BIAN Behavior Qualifiers

| Behavior Qualifier | Institutional Channel | Advisor Channel |
|---|---|---|
| **Develop** | Build prospect strategy, target list | Identify target advisor territories |
| **Pursue** | RFP response, DDQ completion, meetings | Content-driven advisor outreach via Seismic |
| **Fulfill** | Mandate onboarding initiation | Fund placement on advisor approved lists |
| **Assess** | Win/loss analysis, pipeline health | Advisor engagement scoring |

### AI Touchpoints in the Sales & Distribution Lifecycle (Stuart's Work)

```
BIAN Sales Plan Lifecycle — AI Enhancement Points
─────────────────────────────────────────────────────
Initiate Meeting      → AI surfaces CRM history, fund holdings, activity
(Retrieve behavior)     [Stuart's pre-meeting prep tool]
        ↓
Execute Meeting       → Human-led relationship conversation
(Execute behavior)      [No AI — this is the relationship moment]
        ↓
Capture Outcomes      → AI generates meeting summary, follow-up tasks
(Update behavior)       [Stuart's post-meeting action generator]
        ↓
Pursue Opportunity    → AI drafts RFP/DDQ responses from knowledge base
(Develop behavior)      [Kathleen's DDQ/RFP agent]
        ↓
Award Mandate         → Human decision; AI provides analysis only
(Fulfill behavior)
─────────────────────────────────────────────────────
```

### Service Interfaces

```
Sales & Distribution
        ├──► Customer & Party Management (prospect identity and context)
        ├──► Product Management (fund catalogue, performance for pitches)
        ├──► Document Management/Seismic (approved sales content)
        └──► Client Onboarding (mandate award triggers onboarding)
```

---

## 5. Service Domain 3: Client Onboarding & KYC

### BIAN Definition
**Control Record:** Customer Agreement
**Purpose:** Manage the process of establishing a new client relationship — including KYC/AML, legal documentation, account provisioning, data entitlement setup, and reporting configuration.

### Asset Management Context

A single institutional client onboarding involves:
- **KYC/AML verification:** Legal entity validation, beneficial ownership, sanctions screening
- **Legal documentation:** Investment Management Agreement (IMA), subscription documents
- **Account provisioning:** Custodian setup, fund accounting system configuration
- **Data entitlements:** What reports, funds, and data products is the client authorized to access?
- **Portal provisioning:** Secure client portal access, reporting preferences configuration
- **Reporting setup:** GIPS composite assignment, benchmark selection, reporting frequency

### BIAN Behavior Qualifiers

| Behavior Qualifier | Description | Current State | Target State |
|---|---|---|---|
| **Initiate** | Trigger onboarding upon mandate award | Manual handoff from sales | Automated from Salesforce contract signature |
| **Assess** | KYC/AML screening and documentation review | Largely manual, 5–10 days | Automated screening with human exception handling |
| **Agree** | Capture and execute legal documentation | DocuSign or paper | Digital IMA with full audit trail |
| **Activate** | Provision accounts, entitlements, portal | Manual, 2–4 weeks | Automated provisioning in under 24 hours |
| **Complete** | Confirm all systems configured; notify client | Manual checklist | Automated with client notification |

### BIAN-Aligned Onboarding Architecture

```
MANDATE AWARDED (Salesforce Sales Cloud — contract signed)
        ↓
┌────────────────────────────────────────────────────────┐
│          ONBOARDING ORCHESTRATION LAYER                 │
│        (AWS Step Functions / Workflow Engine)           │
│                                                          │
│  Step 1: KYC Verification                              │
│    → Sanctions screening (automated)                   │
│    → Beneficial ownership validation                   │
│    → Human review for exceptions only                  │
│                                                          │
│  Step 2: Legal Documentation                           │
│    → IMA template generation                           │
│    → DocuSign execution and secure storage             │
│    → Audit trail captured                              │
│                                                          │
│  Step 3: Account Provisioning                          │
│    → Custodian account setup notification              │
│    → Fund accounting system configuration              │
│    → Aladdin portfolio setup                           │
│                                                          │
│  Step 4: Data Entitlement Activation                   │
│    → Salesforce entitlement → ADX data access          │
│    → Client portal account provisioned                 │
│    → Reporting preferences configured in Vermilion     │
│                                                          │
│  Step 5: Completion & Notification                     │
│    → Automated confirmation to client and RM           │
│    → First report generation triggered                 │
└────────────────────────────────────────────────────────┘

TARGET: Mandate award → First report delivery in under 3 business days
INDUSTRY AVERAGE: 4–8 weeks
```

### KYC as a Reusable BIAN Service

In BIAN, KYC is a **shared service** consumed by multiple processes — not embedded within onboarding alone:

- Client Onboarding (new clients)
- Counterparty Management (new trading counterparties)
- Periodic Review (annual KYC refresh for existing clients)
- Transaction Monitoring (event-triggered KYC re-verification)

**Why this matters for architecture:** Building KYC as a shared service means it can be upgraded or augmented (with AI document verification, for example) without touching the onboarding process itself.

---

## 6. Service Domain 4: Investment Portfolio Management

### BIAN Definition
**Control Record:** Investment Portfolio
**Purpose:** Manage the investment process — strategy definition, portfolio construction, order generation, position management, and performance monitoring.

### Asset Management Context

This domain is owned by the investment management division — **not** the Head of Client Technology. However, it is the **primary source of authoritative data** for everything downstream — client reporting, performance attribution, risk management, and client portal displays. If data from this domain is wrong, everything built on top of it is wrong.

### BIAN Behavior Qualifiers

| Behavior Qualifier | Description | Technology Platform |
|---|---|---|
| **Initiate** | Establish new portfolio based on client mandate | Aladdin — portfolio setup |
| **Update** | Adjust portfolio as mandate evolves | Aladdin — rebalancing |
| **Execute** | Generate and submit orders to trading | Aladdin OMS → trading desk |
| **Monitor** | Track portfolio vs. benchmark and risk limits | Aladdin real-time monitoring |
| **Evaluate** | Performance attribution and mandate compliance | Aladdin → performance engine → GIPS |

### The Data Flow from Portfolio to Client

```
Investment Decision (Portfolio Manager)
        ↓
Portfolio Management System (Aladdin)
  Authoritative source: positions, trades, exposures, mandate compliance
        ↓
Data Integration Layer (CDC via Kafka)
  Bronze: raw Aladdin events
  Silver: reconciled, validated positions
  Gold:   client-ready analytics and performance data
        ↓
Performance Calculation Engine
  → GIPS composite construction
  → Attribution analysis
  → Risk-adjusted return calculations
        ↓
Client-Facing Services
  → Vermilion reporting platform
  → PowerBI client dashboards
  → Client portal
  → RM analytics in Salesforce
```

### What the Head of Client Technology Owns at This Boundary

The investment management team owns **upstream** of the data integration layer.
The Head of Client Technology owns **the integration layer and everything downstream**.

Key responsibilities at this boundary:
- **Data fidelity:** Complete, accurate capture from Aladdin with no gaps or duplications
- **Latency management:** Define and enforce SLAs per consumer (near-real-time for portal, end-of-day for GIPS)
- **Entitlement propagation:** Portfolio setup in Aladdin automatically activates downstream data access — no manual tickets

---

## 7. Service Domain 5: Trade Execution & Settlement

### BIAN Definition
**Control Record:** Trading Book Entry
**Purpose:** Manage the lifecycle of securities transactions from order generation through confirmation, settlement, and position reconciliation.

### Asset Management Context

Kathleen's Credit Suisse background spans this domain deeply — 8 years across Corporate Bond Trading, Credit Derivatives, Leveraged Finance, Emerging Markets, and Structured Credit middle office. She understands trade lifecycle complexity from the inside.

### BIAN Behavior Qualifiers

| Behavior Qualifier | Description | Key System |
|---|---|---|
| **Initiate** | Order generation from portfolio manager | Aladdin OMS |
| **Execute** | Trade execution via broker/dealer | Execution Management System |
| **Confirm** | Trade confirmation matching with counterparty | Middle office matching |
| **Settle** | Cash and securities movement at custodian | DTC, Euroclear |
| **Reconcile** | Confirm positions match across systems | Reconciliation platform |
| **Report** | Post-trade reporting for compliance | Regulatory reporting |

### Middle Office Flow (Kathleen's Credit Suisse Context)

```
PRE-TRADE            TRADE DATE              POST-TRADE
─────────            ──────────              ──────────
Portfolio  →   Order execution    →   Trade confirmation
decision       (EMS / broker)          (matching system)
(Aladdin)
                    │
                    ▼
             Same-day P&L analysis   ← Kathleen managed
             (middle office)           this team at CS
                    │
                    ▼
             Settlement instructions → Custodian (T+1/T+2)
                    │
                    ▼
             Position reconciliation
             (internal vs. custodian)
                    │
                    ▼
             Portfolio accounting update
             (fund NAV impact)
```

### Relevance to Client Technology

Errors in Trade Execution & Settlement surface as client-facing problems:
- Wrong trade captured → Wrong portfolio position in client report
- Settlement failure → Cash balance discrepancy in client statement
- Reconciliation break → Performance calculation error in GIPS report

The Head of Client Technology must diagnose when a reporting problem originates in the trade lifecycle vs. the reporting platform.

---

## 8. Service Domain 6: Client Reporting & Performance

### BIAN Definition
**Control Record:** Financial Statement
**Purpose:** Produce and distribute accurate, timely financial statements and performance reports to clients, meeting all regulatory and contractual obligations.

### Asset Management Context

This is Kathleen's core domain at Macquarie — she modernized client reporting from manual PDF generation to an efficient, digital-first platform using Vermilion. This service domain has the highest direct client visibility of any domain in the platform.

### BIAN Behavior Qualifiers

| Behavior Qualifier | Description | Platform | Frequency |
|---|---|---|---|
| **Initiate** | Trigger report generation | Vermilion scheduler | Daily/monthly/quarterly |
| **Calculate** | GIPS composite and attribution computation | Performance engine | Daily NAV; monthly attribution |
| **Compile** | Assemble report from multiple data sources | Vermilion templating | Per report cycle |
| **Validate** | Data quality checks before publication | Automated validation rules | Every report |
| **Distribute** | Deliver reports via portal, email, or API | Vermilion + portal | Per client preference |
| **Archive** | Store reports with full audit trail | S3 immutable storage | Permanent (regulatory retention) |

### The Reporting Data Architecture

```
SOURCE SYSTEMS
┌──────────────┐  ┌─────────────┐  ┌──────────────────┐
│ Aladdin      │  │ Bloomberg/  │  │ Fund Accounting   │
│ (Positions)  │  │ FactSet     │  │ (NAV, cash)       │
└──────┬───────┘  └──────┬──────┘  └────────┬──────────┘
       └─────────────────┼──────────────────┘
                         ▼
         ┌───────────────────────────────┐
         │  DATA PLATFORM (Gold Layer)   │
         │  Validated, single truth      │
         └───────┬───────────────────────┘
                 │
     ┌───────────┼───────────────┐
     ▼           ▼               ▼
┌──────────┐ ┌──────────┐ ┌──────────────┐
│   GIPS   │ │Vermilion │ │   PowerBI    │
│  Engine  │ │(Reports) │ │ (Dashboards) │
└────┬─────┘ └────┬─────┘ └──────┬───────┘
     └────────────┼───────────────┘
                  ▼
     ┌────────────────────────┐
     │   DISTRIBUTION LAYER   │
     │   Client Portal        │
     │   Secure Email         │
     │   API (institutional)  │
     └────────────────────────┘
```

### GIPS as a Shared BIAN Service

BIAN's service-oriented approach makes an important architectural point: GIPS composite logic should be a **shared service** that multiple consumers reference — not embedded inside Vermilion.

```
GIPS Composite Management Service
        ├──► Client Reporting (composite performance in client reports)
        ├──► Sales & Distribution (composite track record in RFPs)
        ├──► Regulatory Reporting (performance advertising compliance)
        └──► Audit & Verification (third-party GIPS verification support)
```

If GIPS logic is embedded in Vermilion, upgrading the reporting platform requires re-implementing compliance logic. If GIPS is a discrete service, both can evolve independently.

---

## 9. Service Domain 7: Sales Enablement & Advisor Distribution

### BIAN Definition
**Control Record:** Sales Product Agreement
**Purpose:** Manage the distribution of investment products through intermediary channels — including content management, advisor relationship management, and distribution analytics.

### Asset Management Context

This domain maps directly to Stuart's Macquarie work — Seismic, advisor channel CRM, event management, and the AI pre/post meeting tools. It also connects to Kathleen's client experience work wherever the advisor is the intermediary between Nomura and the end client.

### BIAN Behavior Qualifiers

| Behavior Qualifier | Description | Platform |
|---|---|---|
| **Develop** | Create and approve sales content | Seismic content authoring + compliance workflow |
| **Distribute** | Deliver content to advisors at the right moment | Seismic — triggered by CRM context |
| **Track** | Monitor content engagement and advisor activity | Seismic analytics + Salesforce |
| **Enable** | Prepare sales team for advisor meetings | AI pre-meeting prep tool (Stuart) |
| **Evaluate** | Measure distribution effectiveness and ROI | Salesforce + Seismic analytics |

### Content Governance as a Distinct BIAN Service

Content governance is a **separate service** within Sales Enablement — not embedded in distribution:

```
CONTENT GOVERNANCE SERVICE
        │
        ├── Input:    Draft content from marketing/investment teams
        ├── Behavior: Review → Compliance approval → Version control
        │                    → Expiration dating → Audit logging
        ├── Output:   Approved, versioned content available to consumers
        └── Consumers:
              → Seismic (advisor-facing distribution)
              → DDQ/RFP Agent (knowledge base for institutional responses)
              → Client Portal (approved fund documents)
              → Regulatory Reporting (performance advertising compliance)
```

**Critical insight for Kathleen:** The DDQ/RFP agent and Seismic **both consume approved content from the same governance service**. If these workflows are managed separately, you get drift — different versions of the same answer in institutional DDQs vs. advisor-facing materials. One Content Governance service eliminates that risk.

### The Seismic-Salesforce BIAN Service Interaction

```
Wholesaler opens Salesforce account for Advisor X
        ↓
Customer & Party Management service provides:
  → Advisor X profile, AUM, client demographics,
    recent fund holdings, last meeting date
        ↓
Sales & Distribution service determines:
  → Advisor X is focused on retirement planning clients
    with $500K–$2M AUM
        ↓
Sales Enablement service (Seismic) surfaces:
  → Target date fund fact sheets
  → Retirement income whitepaper (current version)
  → Upcoming webinar invitation
        ↓
Wholesaler meets Advisor X with exactly the right content,
personalized to their client base — zero manual search
```

---

## 10. Service Domain 8: Data & Integration Architecture

### BIAN Definition
**Control Record:** Business Data
**Purpose:** Manage the authoritative storage, governance, and distribution of business data across all service domains — ensuring consistency, lineage, security, and accessibility.

### Asset Management Context

This is the infrastructure layer that all other service domains depend on. It maps to Nomura's data platform — Snowflake/S3 medallion architecture, Kafka streaming, API gateway — connecting Aladdin, Bloomberg, FactSet, Salesforce, and Vermilion.

### BIAN Behavior Qualifiers

| Behavior Qualifier | Description | Platform |
|---|---|---|
| **Collect** | Ingest data from source systems | Kafka CDC, AppFlow, API connectors |
| **Maintain** | Store and version data with full lineage | S3 Medallion (Bronze/Silver/Gold) |
| **Consolidate** | Reconcile data across sources into single truth | Snowflake / Redshift |
| **Distribute** | Provide data to consuming service domains | REST APIs, GraphQL, Zero-copy ADX |
| **Govern** | Enforce access control, data quality, retention | Lake Formation RLS, data catalog |
| **Audit** | Track all data access and changes | CloudTrail + audit log |

### The BIAN Data Architecture

```
COLLECTION LAYER (Bronze)
  → Raw events from Aladdin (CDC via Kafka)
  → Market data from Bloomberg/FactSet (ADX zero-copy)
  → CRM activity from Salesforce (outbound messages)
  → Fund accounting data
        ↓
CONSOLIDATION LAYER (Silver)
  → Reconciled positions (Aladdin vs. custodian confirmed)
  → Validated market prices (primary + backup source)
  → Linked party records (Aladdin portfolio ID linked to Salesforce account ID)
  → Quality-checked: completeness, consistency, timeliness
        ↓
DISTRIBUTION LAYER (Gold)
  → Client-ready analytics (NAV, attribution, performance vs. benchmark)
  → RM-ready insights (client AUM, engagement score, next best action)
  → Advisor-ready content triggers (Seismic personalization data)
  → Regulatory-ready data (GIPS composites, audit trails)
        ↓
CONSUMING SERVICE DOMAINS
  → Client Reporting (Vermilion reads Gold layer)
  → Customer Management (Salesforce reads Gold layer — no ownership)
  → Sales Enablement (Seismic personalizes from Gold layer)
  → Performance Engine (GIPS calculates from Gold layer)
  → Client Portal (displays from Gold layer with RLS)
```

### Party Reference Data: The Most Critical Shared Service

**The identity mesh problem in asset management:**

```
WITHOUT Party Reference Data service:
  Aladdin:   "BlackRock Alternatives Fund II LLC"
  Salesforce: "BlackRock - ALT Fund 2"
  Vermilion:  "BR ALTF2 - Institutional"
  Portal:     Account ID #4829
  → Four records for one client
  → Joining data across systems requires manual mapping
  → Any cross-system client report is unreliable

WITH Party Reference Data service:
  Canonical record: Legal Entity Identifier (LEI)
  → All systems reference the same LEI
  → Client data is joinable across all platforms
  → Client-level analytics are accurate and automated
```

---

## 11. BIAN End-to-End Business Process Flows

### Flow 1: Institutional Client — Prospect to First Report

```
STAGE 1: PROSPECTING
  BIAN Domain:  Sales & Distribution
  Action:  RM identifies pension fund prospect
  Systems: Salesforce (account created), AI pre-meeting prep tool
  Output:  Prospect record, initial engagement plan

STAGE 2: QUALIFICATION
  BIAN Domain:  Customer & Party Management
  Action:  Research prospect AUM, mandate, consultant relationships
  Systems: Salesforce + external research
  Output:  Qualified prospect with relationship context

STAGE 3: RFP RESPONSE
  BIAN Domain:  Sales & Distribution + Document Management
  Action:  AI agent drafts response from approved knowledge base
  Systems: Kathleen's AI DDQ/RFP Agent + compliance review
  Output:  Completed RFP submitted within defined SLA

STAGE 4: DDQ COMPLETION
  BIAN Domain:  Sales & Distribution + Compliance
  Action:  Technology sections owned by Head of Client Technology
  Systems: AI DDQ/RFP Agent + IT security documentation
  Output:  Completed DDQ — operational and technology sections current and accurate

STAGE 5: MANDATE AWARD
  BIAN Domain:  Sales & Distribution → Customer Agreement
  Action:  Contract signed; onboarding workflow triggered
  Systems: Salesforce (opportunity closed-won) → Step Functions
  Output:  Signed IMA; onboarding initiated automatically

STAGE 6: CLIENT ONBOARDING
  BIAN Domain:  Client Onboarding & KYC
  Action:  KYC screening, legal docs, account provisioning, entitlements
  Systems: KYC service + Step Functions + Salesforce + ADX
  Output:  Client fully provisioned; data access active; portal live

STAGE 7: FIRST REPORT DELIVERY
  BIAN Domain:  Client Reporting & Performance
  Action:  Generate first GIPS-compliant account report
  Systems: Aladdin → Data Platform → Performance Engine → Vermilion → Portal
  Output:  Client receives accurate first report within 3 business days
```

---

### Flow 2: Advisor Channel — Engagement to Fund Placement

```
STAGE 1: ADVISOR IDENTIFICATION
  BIAN Domain:  Customer & Party Management
  Action:  Wholesaler identifies target advisor via Salesforce territory management
  Output:  Target advisor list with AUM, client demographics, current product holdings

STAGE 2: PRE-MEETING PREPARATION
  BIAN Domain:  Sales Enablement
  Action:  AI surfaces relevant content; CRM provides relationship context
  Systems: Seismic + Salesforce + AI pre-meeting prep tool (Stuart)
  Output:  Personalized content package for the advisor's client base

STAGE 3: ADVISOR MEETING
  BIAN Domain:  Sales & Distribution
  Action:  Wholesaler presents fund strategy, performance, market commentary
  Systems: Seismic (content delivery), Salesforce (activity logging)
  Output:  Meeting notes; follow-up tasks generated by AI post-meeting tool

STAGE 4: FUND APPROVAL PROCESS
  BIAN Domain:  Sales & Distribution + Product Management
  Action:  Advisor submits fund for due diligence approval at their firm
  Systems: Seismic (documentation package), Salesforce (approval status tracking)
  Output:  Fund placed on advisor's approved product list

STAGE 5: ONGOING ENGAGEMENT
  BIAN Domain:  Sales Enablement + Customer & Party Management
  Action:  Seismic analytics triggers proactive re-engagement
  Systems: Seismic analytics → Salesforce alert → wholesaler action
  Output:  Active advisor remains engaged distributor
```

---

### Flow 3: AI DDQ/RFP Agent — BIAN Service Interaction

```
TRIGGER: Institutional prospect sends DDQ document

BIAN Domain: Document Management
  → Receive and classify incoming DDQ
  → Extract and categorize questions by domain
        ↓
BIAN Domain: Knowledge Management (Content Governance)
  → Query approved knowledge base for relevant answers
  → Retrieve versioned, ownership-confirmed responses
  → Flag questions with no approved answer → human escalation
        ↓
BIAN Domain: AI-Assisted Processing
  → LLM generates draft responses grounded in approved content
  → Confidence scoring for each answer
  → Low-confidence responses flagged for human review
        ↓
BIAN Domain: Compliance Review
  → Human review queue for flagged responses
  → Regulatory accuracy check (Series 7/63 awareness)
  → Final approval workflow
        ↓
BIAN Domain: Sales & Distribution
  → Assemble completed DDQ document
  → Route to RM for relationship context additions
  → Submit to prospect within SLA
        ↓
AUDIT: All agent outputs logged with version, timestamp,
       reviewer identity, and final vs. draft comparison
```

---

## 12. Platform-to-BIAN Mapping

| Platform / Tool | BIAN Service Domain | Key Behaviors | Integration Boundary |
|---|---|---|---|
| **Salesforce Sales Cloud** | Customer & Party Management + Sales & Distribution | Initiate, Update, Retrieve, Monitor | System of engagement — reads from reference data, never duplicates authoritative records |
| **Salesforce Service Cloud** | Client Onboarding + Case Management | Initiate, Assess, Activate | Orchestrates onboarding; does not own provisioning logic |
| **Salesforce FSC** | Customer & Party Management (Wealth) | Household tracking, financial goal management | Wealth-specific layer on top of core CRM |
| **Seismic** | Sales Enablement & Advisor Distribution | Develop, Distribute, Track | Consumes approved content from Content Governance; feeds engagement data to CRM |
| **AI DDQ/RFP Agent** | Document Management + Knowledge Management | Initiate, Retrieve, Execute | Reads from Content Governance; outputs to Sales & Distribution review queue |
| **AI Pre/Post Meeting Tool** | Sales Enablement | Enable, Capture | Reads CRM and Data Platform; writes activity back to CRM |
| **Vermilion** | Client Reporting & Performance | Calculate, Compile, Validate, Distribute | Reads Data Platform Gold layer; distributes to portal and secure delivery |
| **PowerBI** | Client Reporting (interactive layer) | Compile, Distribute | Reads Data Platform; embedded in Salesforce via LWC |
| **Aladdin** | Investment Portfolio Management | Initiate, Execute, Monitor | System of record for positions; feeds Data Platform via CDC |
| **Bloomberg/FactSet** | Market Data Management | Collect, Maintain | Feeds Data Platform via ADX zero-copy |
| **Data Platform (S3/Snowflake)** | Data Management & Integration | Collect, Maintain, Consolidate, Distribute | Foundation layer — single source of truth for all downstream consumers |
| **Kafka/MSK** | Event Stream Management | Collect (real-time), Notify | Real-time event bus connecting Portfolio Management to Data Platform |
| **AWS Step Functions** | Onboarding Orchestration | Initiate, Execute, Complete | Workflow orchestration across KYC, provisioning, and entitlement services |
| **AWS ADX** | Data Distribution (Marketplace) | Distribute (zero-copy) | Client-facing data product distribution; entitlement-controlled |
| **Lake Formation / IAM** | Entitlement Management | Activate, Govern, Audit | Row-level security enforcement across all data platform consumers |

---

## 13. Interview Talking Points Using BIAN Vocabulary

### With Stuart — On Salesforce Platform Boundaries

> *"In BIAN terms, Salesforce is our Customer & Party Management and Sales Distribution service domain — it owns relationship context and engagement history. But it is a consumer of the Data Management service, not a producer of authoritative data. Keeping that service boundary clean is what prevents the data quality problems that surface when CRM tries to own things outside its domain."*

---

### With Kathleen — On AI Agent Architecture

> *"The DDQ/RFP agent spans two BIAN service domains — Document Management and Knowledge Management. The agent is the execution layer, but the Knowledge Management service — your content governance model, ownership structure, version control — is what makes it trustworthy. I would focus first on maturing the Knowledge Management service before expanding the agent to new document types."*

---

### With Kathleen — On Content Governance Convergence

> *"One architectural opportunity I see is that the DDQ agent and Seismic currently consume approved content independently. In BIAN's model, both are consumers of a single Content Governance service. Routing both through the same governance layer means the answer in an institutional DDQ is always consistent with what an advisor sees in Seismic — which is a regulatory and a trust benefit simultaneously."*

---

### With Both — On Client Reporting

> *"GIPS compliance is most maintainable when it is treated as a shared service rather than embedded in the reporting platform. If composite logic lives inside Vermilion, upgrading the reporting layer requires re-implementing compliance logic. If GIPS is a discrete service that reporting, the RFP agent, and the portal all reference, each can evolve independently."*

---

### With Both — On Client Onboarding

> *"In the BIAN framework, onboarding is an orchestration domain — it coordinates across KYC, account provisioning, entitlement activation, and reporting setup. The business pain of slow onboarding is almost always a sequencing problem — these steps are done serially when most of them can run in parallel. The technology solution is an orchestration layer that manages parallel workstreams and tracks completion across all systems."*

---

### With Shaw — On Data Architecture

> *"The data platform is the infrastructure layer that all BIAN service domains depend on. In BIAN terms, it executes Collect, Consolidate, and Distribute behaviors — which maps exactly to the Bronze, Silver, and Gold layers of the medallion architecture. What makes it work is the governance layer: clear data product ownership, lineage tracking, and entitlement control so every consuming service domain trusts what it receives."*

---

### On Vendor Evaluation

> *"I use BIAN service domains as the evaluation frame for vendor selection. Before evaluating any vendor, I want to understand which service domain we are filling, what its interfaces are to adjacent domains, and whether the vendor respects those boundaries or tries to expand beyond them. A vendor that wants to own too many service domains — CRM, reporting, data, and portal all in one platform — usually creates more coupling than it eliminates."*

---

## Document Metadata

| Field | Value |
|---|---|
| **Prepared by** | Calvin Lee |
| **Framework** | Banking Industry Architecture Network (BIAN) v12 |
| **Target audience** | Kathleen Hess McNamara, Stuart Mumley, Shaw Levin |
| **Role** | ED, Head of Client Technology — Nomura Asset Management International |
| **Purpose** | BIAN service domain mapping for Nomura asset management client technology |
| **Related documents** | [README.md](README.md) · [BUSINESS_PARTNERSHIP_README.md](BUSINESS_PARTNERSHIP_README.md) |
| **Last updated** | February 2026 |

---

*BIAN provides the shared vocabulary for technology and business leaders to discuss platform architecture, service boundaries, and vendor evaluation in terms that are both technically precise and business-meaningful. This document is the architectural bridge between the technical platform (README.md) and the business partnership model (BUSINESS_PARTNERSHIP_README.md).*
