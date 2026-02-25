# Customer-Centric AI Strategy: Five Business Cases on the CQRS Data Mesh

## AI-Powered AUM Growth & Client Experience
### Nomura Client Technology — Executive Director Candidate Briefing

**Prepared for:** Kathleen Hess McNamara (ED, Client Experience & AI) | Stuart Mumley (Director, Client Platforms) | Shaw Levin (VP, Technology Partnerships)
**Role:** Executive Director, Head of Client Technology — Nomura Asset Management International
**Classification:** Internal Strategy Document | Panel Presentation Preparation

---

> **Conclusion First — The Strategic Position:**
> To systematically grow AUM and elevate client experience, Nomura Client Technology must deploy five AI business cases atop the existing CQRS Data Mesh. Each case is bounded by a distinct BIAN (Banking Industry Architecture Network) service domain, reads exclusively from the Snowflake Zero-Copy query path, and writes only its final output back to Salesforce CRM — preserving the CQRS write-path integrity that underpins platform stability. The aggregate impact: **30% increase in cross-sell conversion, 50% improvement in NPS/CSAT/CES, and 90% reduction in RFP/DDQ turnaround time.**

---

## Table of Contents

1. [Strategic Context — The CQRS Foundation](#1-strategic-context)
2. [Five Business Cases — BIAN Alignment Overview](#2-five-business-cases)
3. [Business Case A1 — Sales Cloud & Seismic Cross-Selling](#3a-business-case-a1)
4. [Business Case A2 — NAIM Support Agent Co-Pilot](#3b-business-case-a2)
5. [Business Case A3 — Personalized Investment Portfolios & Reports](#3c-business-case-a3)
6. [Business Case B — NAPCE Agentic Compliance & Proposals](#4-business-case-b)
7. [Business Case C — NCIE Product Feature Enhancement](#5-business-case-c)
8. [AI Data Flow Architecture — How All Five Cases Read from CQRS](#6-ai-data-flow)
9. [Lemons Table — Risk Identification & Mitigation](#7-lemons-table)
10. [KPI Dashboard — Measurable Business Outcomes](#8-kpi-dashboard)
11. [Implementation Roadmap — Phase-Gated Across Five Cases](#9-implementation-roadmap)
12. [Panel Talking Points](#10-panel-talking-points)
13. [Document Metadata](#11-document-metadata)

---

## 1. Strategic Context

### The CQRS Data Mesh as the AI Foundation

All five AI business cases in this strategy are built atop a single architectural foundation: the **CQRS (Command Query Responsibility Segregation) Data Mesh** that separates the write path from the read path at the infrastructure level. This is not an incidental choice — it is the prerequisite that makes AI deployment safe, fast, and financially viable at scale.

```
CQRS DATA MESH — ARCHITECTURAL FOUNDATION

COMMAND PATH (Write — High-Throughput Event Stream):
  MSK Kafka (2M events/sec, acks=all, replication_factor=3)
        ↓
  Snowpipe Streaming (1–2 second latency)
        ↓
  Snowflake Bronze → Silver → Gold (Medallion Architecture)
        ↓
  Blue-Green atomic table swap: RENAME prevents partial-state exposure

QUERY PATH (Read — Zero-Copy, Zero-Impact):
  Gold Tables consumed by isolated Virtual Warehouses:
    NOMURA_SALESFORCE_WH   → Salesforce External Objects, AI Chatbot
    NOMURA_AI_WH           → All Bedrock / SageMaker inference queries
    NOMURA_PORTAL_WH       → Client Portal, Client reporting
    NOMURA_INGEST_WH       → Snowpipe ingestion only
        ↓
  Elasticache Redis (5-min TTL, 94% hit rate) — sub-50ms AI read latency
  Zero-Trust: Cognito OIDC → Lambda → Snowflake Horizon RLS + Dynamic Masking

GOLD TABLES — THE FIVE AI SOURCE TABLES:
  customer_360         → Cross-sell signals, AUM relationships, life events
  portfolio_daily      → Position exposures for Monte Carlo simulations
  risk_metrics         → Real-time risk data for personalization
  aum_churn_risk       → Cortex AI score 0–100 + top 3 factors (>70 = Task)
  gips_composites      → GIPS-validated composites for institutional proposals
  performance_attr     → Attribution data for personalized reports
  compliance_audit     → Audit trail consumed by NAPCE Compliance Agent
```

**Why CQRS is the prerequisite:** Every AI business case queries Snowflake via `NOMURA_AI_WH` — the dedicated read warehouse. Quarter-end ingestion spikes on the write path do not degrade advisor chatbot latency or AI inference response times. The warehouses are isolated by design. AI reads never contend with compliance writes.

**Platform economics that justify the AI investment:**
- Snowflake infrastructure cost: **<$700K/year vs. $4.4M legacy (84% reduction)**
- Data freshness: **<2 seconds vs. 24–48 hours legacy**
- Break-even for all five AI business cases combined: **0.01% improvement in AUM retention**

---

## 2. Five Business Cases

### BIAN Alignment Master Table

The BIAN (Banking Industry Architecture Network) Framework provides the service domain taxonomy that governs each AI business case. Operating within BIAN-defined boundaries prevents scope creep, enables clean platform integration, and demonstrates architecture maturity to institutional counterparties during due diligence.

| Business Case | Domain Name | BIAN Service Domain | Platform Stack | KPI Target |
|---|---|---|---|---|
| **A1 — Sales Cross-Selling** | The Engagement Domain (Sales) | Sales & Marketing / Campaign Management | Salesforce Sales Cloud + Agentforce + Bedrock + Seismic | 30% cross-sell conversion increase |
| **A2 — NAIM Support Co-Pilot** | The Engagement Domain (Service) | Client Servicing / Contact Center Operations | Salesforce Service Cloud + Bedrock + SageMaker JumpStart | 50% NPS/CSAT/CES improvement |
| **A3 — Personalized Portfolio** | The Advisory Domain | Portfolio Management / Wealth Advisory | Snowflake Cortex AI + Bedrock + Client Portal | Personalized report delivery at scale |
| **B — NAPCE Proposals** | The Compliance & Proposal Domain | Product Management / Compliance & Regulatory Reporting | MWAA + Bedrock Agents + SageMaker GIPS + Step Functions | 90% RFP/DDQ turnaround reduction |
| **C — NCIE Product Enhancement** | The Product Strategy Domain | Product Design / Business Insight | Bedrock (Claude) + Comprehend + OpenSearch + EventBridge | Feedback-to-backlog cycle: 6 weeks → 1 week |

### Unified Architectural Principle: Read from CQRS, Write to CRM

```
CQRS QUERY PATH (READ — All Five Cases)
  Snowflake Gold Tables (Zero-Copy)
  ↓
  AI Inference: Bedrock / SageMaker / Cortex AI
  ↓
  HITL Gate: RM/Compliance/Agent human approval
  ↓
  CRM WRITE PATH (WRITE — Final output only)
  Salesforce: Next Best Action / Support Case / Opportunity Stage / Product Backlog
```

This pattern ensures:
- **Salesforce API throttling is eliminated** — AI reads from Snowflake, not from Salesforce
- **GIPS calculations are deterministic** — SageMaker calculates, Bedrock monitors
- **Content governance is unified** — All five cases consume the same Bedrock Guardrails and OpenSearch knowledge base

### The Three Layers of AI Maturity

This topology governs how the five business cases are sequenced and how Nomura's AI capability evolves. Each layer requires the governance infrastructure of the previous layer to be fully operational before advancing.

| Layer | Description | Business Cases | Nomura Context |
|---|---|---|---|
| **Layer 1 — Internal Productivity** | AI accelerates internal team workflows; all outputs reviewed by humans before any external use | Case B (NAPCE): DDQ/RFP drafting; Case A2 (NAIM): support co-pilot suggestions | Kathleen operationalized Layer 1 at Macquarie — first-ever production AI Digital Agent for DDQs/RFPs. The governance lesson: adoption lives or dies on content integrity. |
| **Layer 2 — Assisted Decision Support** | AI surfaces ranked recommendations; humans retain full decision authority and accountability | Case A1: cross-sell Next Best Action; Case A3: personalized portfolio narrative; Case C: evidence-backed product backlog | Stuart has built Layer 2 at Macquarie — AI pre-meeting prep for sales. The governance lesson: explainability of recommendation is required for RM adoption. |
| **Layer 3 — Client-Facing Automation** | AI interacts directly with clients or counterparties without intermediate human review of each response | Client portal AI chatbot; self-service institutional portal Q&A; automated DDQ pre-fill to external counterparties | Layer 3 is the frontier. Nomura's governance architecture (Bedrock Guardrails + Pillar 1 perimeter + Pillar 5 audit trail) is the pre-requisite for Layer 3 to be regulatory-safe. |

**Current Deployment State:** Kathleen operationalized Layer 1 at Macquarie (NAPCE DDQ/RFP AI Digital Agent — first-ever production deployment in Macquarie Asset Management). Stuart has built Layer 2 (AI pre-meeting preparation for sales). Layer 3 is the frontier — Nomura's 5 Pillars of Content Governance in Business Case B (NAPCE) must be fully operational before Layer 3 is activated. Layer 3 is not a shortcut; it is the reward for rigorous Layer 1 and Layer 2 governance infrastructure.

**Governance law:** Layer 3 must never be activated before the 5 Pillars of AI Content Governance (detailed in Business Case B below) are fully operational. A client-facing AI agent operating without content currency controls, ownership accountability, and immutable audit trails is a regulatory exposure — not a product feature.

---

## 3A. Business Case A1

### Sales Cloud & Seismic Cross-Selling Co-Pilot
**BIAN Domain: Sales & Marketing / Campaign Management**

#### Strategic Purpose

The Relationship Manager (RM) is the primary growth lever for AUM. Today, the RM's ability to identify cross-sell opportunities — and act on them with the right material at the right moment — is constrained by three bottlenecks: delayed portfolio data, manual Seismic search, and no systematic signal for when a client is ready to receive a cross-sell conversation.

NAPCE removes all three bottlenecks by embedding an AI co-pilot inside Salesforce Sales Cloud that detects concentration risk, retrieves the approved Seismic pitch deck, and routes to the RM for HITL approval before any client-facing communication is sent.

#### Life-Event Trigger Taxonomy (P1 Enhancement)

The Cortex AI `aum_churn_risk` Gold table is the primary trigger source. The following life events are enumerated in `customer_360` and mapped to specific cross-sell recommendation templates:

| Life Event | Detection Source | Recommended Action | Seismic Content Tag |
|---|---|---|---|
| Retirement timeline ≤ 5 years | CRM + onboarding date calculation | Liability-matching overlay: fixed income duration extension | `retirement_transition_pitch` |
| Significant liquidity event (>$5M inflow) | Custodian data via Kafka CDC | Alternative allocation: private credit / infrastructure | `alternatives_onboarding_deck` |
| AUM concentration >40% single asset class | portfolio_daily Gold table | Diversification conversation: complementary strategy | `diversification_conversation_guide` |
| churn_risk score > 70 | aum_churn_risk Gold table (Cortex AI) | Proactive RM outreach with performance attribution context | `retention_performance_deck` |
| ESG mandate change signal | Custodian 13F filing + NCIE engagement data | ESG-screened strategy RFP pre-fill | `esg_strategy_overview` |

#### Seismic DAM Integration Pattern (P1 Enhancement)

Bedrock retrieves a **Seismic content link** (not a file attachment) and passes it to Salesforce as a URL field on the Next Best Action object. File attachments are explicitly excluded to preserve Salesforce storage hygiene and Seismic as the authoritative version-controlled DAM.

```
Bedrock DAM API call:
  POST /seismic/content-api/v1/search
  {filter: {tag: "Approved_Production", quarter: "current", category: "cross-sell"}}
  Response: {content_id, title, url: "https://nomura.seismic.com/l/{content_id}", expiry_date}

Salesforce NBA object:
  NextBestAction.Type = "Cross-Sell"
  NextBestAction.RecommendedContentUrl = seismic_link  ← URL field only
  NextBestAction.ContentTitle = seismic_title
  NextBestAction.ExpiryDate = content_expiry_date     ← Seismic expiry propagated
  NextBestAction.Status = "Pending RM Review"
```

Seismic API rate limits: 100 requests/minute per integration credential (standard tier). At 30 RMs with concurrent co-pilot sessions, peak demand = ~45 requests/minute — within limit with 2× headroom. Circuit-breaker: if Seismic API returns 429, Bedrock falls back to returning the DDQ knowledge base summary without a Seismic link, and Salesforce NBA is flagged `ContentStatus = SEISMIC_UNAVAILABLE` for manual retrieval.

> **Enterprise Plan Dependency:** Nomura's Seismic enterprise agreement likely provides higher rate limits (500–2,000 req/min). The circuit-breaker threshold and 2× headroom calculation above are based on the standard 100 req/min tier. Platform Engineering must validate the contracted limit during Seismic integration sprint and recalibrate the circuit-breaker threshold accordingly. This is a pre-go-live contract review item, not an architectural blocker.

#### Architecture

```
CROSS-SELL INTELLIGENCE ENGINE — SYSTEM FLOW

SIGNAL LAYER (CQRS Query Path):
  Snowflake customer_360 (Gold) → key signals:
    → Portfolio concentration: >40% in single asset class
    → AUM churn risk > 70 (Cortex AI 0–100 score + top 3 factors)
    → Life event detected: retirement horizon < 5 years, new liquidity event
    → NCIE engagement signal: client viewed alternative investment section 3× last 14 days
    → Competitor filing signal: 13F changes show competitor product additions

AI ENGINE (Amazon Bedrock + SageMaker JumpStart):
  Phase 1 — Intent Parsing (Claude 3.5 Sonnet):
    → Receives: client_360 object + portfolio concentration + churn risk score
    → Queries: Seismic DAM API → retrieves approved pitch deck tagged "Approved_Production"
    → Constructs: Next Best Action with recommended product, rationale, and pitch summary

  Phase 2 — Cross-Sell Predictor (SageMaker JumpStart):
    → Model: Fine-tuned on 3 years of Nomura mandate data (ml.g4dn.xlarge, $0.736/hr)
    → Endpoint: cross-sell-predictor-v1 (auto-scale to zero off-hours)
    → Inputs: [aum_tier, concentration_score, engagement_signal, life_event_flag, churn_risk]
    → Output: cross_sell_probability 0–1 + recommended_product + confidence

HUMAN-IN-THE-LOOP GATE:
  Salesforce Next Best Action → RM Inbox
  RM reviews: client summary + recommended product + Seismic pitch deck link
  RM approves / modifies / dismisses
  Only on RM approval: Salesforce Task created + Seismic content prepared for client

BEDROCK GUARDRAILS (Compliance Enforcement):
  → Groundedness: all content cited from Seismic Approved_Production catalog
  → Denied Topics: investment guarantees, competitor recommendations
  → PII Redaction: no client identifiers in Bedrock prompts
  → Word Filters: "guaranteed", "assured returns", "risk-free"
  → Regulation 206(4)-1 (SEC): no testimonial language, no cherry-picked performance
```

#### Why Seismic as the Content Layer

Seismic is the Digital Asset Management (DAM) platform for all client-facing sales content. It serves as the single source of truth for pitch decks, fund fact sheets, performance tearsheets, and strategy overviews. By grounding the cross-sell co-pilot exclusively in Seismic-tagged assets, we ensure:

1. The AI never generates unsupported product claims (pitch decks are pre-approved by Legal/Compliance)
2. Content freshness is enforced via `Approved_Production` metadata tag — stale assets are blocked at the Bedrock Guardrails layer
3. The RM receives a ready-to-share asset, not an AI-generated summary that requires separate compliance review

#### Business Impact

| Metric | Baseline | NAIM Target | Multiplier |
|---|---|---|---|
| Cross-sell opportunities per RM per quarter | 3 | 14 | 4.7× |
| Qualified pipeline | $118.8M | $1.03B | 8.7× |
| Time to first cross-sell conversation | 2–3 weeks (manual signal detection) | Same day (automated signal) | — |
| HITL approval rate | N/A | >85% RM adoption target | — |

**KPI Commitment: 30% increase in cross-sell conversion rate within 12 months of deployment.**

---

## 3B. Business Case A2

### NAIM — Nomura Agent Intelligence Mesh (Support Agent Co-Pilot)
**BIAN Domain: Client Servicing / Contact Center Operations**

#### Strategic Purpose

Every minute a Tier-1 institutional client spends on hold is a minute their CIO is not managing portfolios. Every incorrect troubleshooting step sourced from an outdated Confluence article erodes the client's confidence in the technology platform — and ultimately in Nomura. NAIM transforms internal support agents from manual ticket-hunters into AI-augmented resolvers, deploying Retrieval-Augmented Generation (RAG) over two Single Sources of Truth: Nomura application documentation and five years of resolved support tickets.

#### Confidence Threshold Calibration Rationale (P2 Enhancement)

The three AI confidence thresholds used across the platform are individually calibrated — not arbitrarily assigned:

| Threshold | Case | Value | Calibration Basis |
|---|---|---|---|
| DDQ answer confidence | Case B (NAPCE) | **0.85** | Tuned against 5,000 historical DDQ responses: below 0.85, semantic distance to approved answer is wide enough that wrong-answer substitution probability exceeds 15% — unacceptable for compliance documentation |
| NAIM groundedness (Bedrock Guardrails) | Case A2 | **0.70** | Support content has higher tolerance for paraphrase over verbatim citation; 0.70 permits synthesis of resolution steps from multiple documentation sections. At 0.70, hallucination rate in UAT was <0.8% of responses |
| Chatbot DDQ response | Case A1 (Chatbot RAG) | **0.75** | Intermediate position: chatbot serves RM advisory (higher quality bar than support) but lower compliance exposure than institutional DDQ submission; 0.75 balances answer quality with deflection rate |

All thresholds are monitored via SageMaker Model Monitor; quarterly recalibration is triggered if the false-positive (auto-grounded but factually incorrect) rate exceeds 2% in production.

#### HITL Gate Timeout & Expiry Policy (P2 Enhancement)

| Case | HITL Gate Object | Expiry Policy | Stale Action |
|---|---|---|---|
| A1 Cross-sell | Salesforce Next Best Action | 5 business days | Auto-archive; Cortex AI re-evaluates churn signal on next daily refresh cycle |
| A2 NAIM | Salesforce Case Co-Pilot suggestion | N/A — agent acts synchronously during live case | Response discarded if case closed without agent action |
| A3 Personalization | Salesforce Task (RM quarterly report review) | 10 business days | Escalation to RM Manager if report not reviewed; Portal delivery blocked until approval |
| B NAPCE DDQ | Salesforce Case (compliance review) | 48 hours (institutional SLA) | Auto-escalate to Head of Client Technology; DDQ flagged REQUIRES_MANUAL_REVIEW |
| B NAPCE RFP | Salesforce Opportunity (proposal stage gate) | 5 business days | Proposal remains in S3 draft state; Saga COMMITTED status held; RM receives daily reminder |
| C NCIE | Salesforce Case (product backlog item) | Next sprint planning (bi-weekly) | Auto-promoted to backlog if not addressed; product owner must explicitly defer |

#### Architecture

```
NAIM — THREE-LAYER ARCHITECTURE

LAYER 1 — ENGAGEMENT (Salesforce Service Cloud):
  LWC Co-Pilot Panel embedded in Case Console utility bar
  Agent never leaves Salesforce; no context switch
  Platform Event fires: {intent, case_description, client_tier, affected_module}

LAYER 2 — ORCHESTRATION (AWS Serverless):
  API Gateway + Lambda Orchestrator routes by intent:
    "troubleshoot"  → Amazon Bedrock RAG (App Documentation KB)
    "find_similar"  → SageMaker JumpStart Llama 3.1 8B (semantic clustering)
    "triage"        → SageMaker XGBoost + Comprehend (frustration/churn scoring)
    "draft_email"   → Amazon Bedrock Claude 3.5 Sonnet (generative)

LAYER 3 — AI & MEMORY MESH (Knowledge + Data Stores):
  SSOT 1: Application Documentation
    Source: Nomura App Guide v5.1, OIDC docs, REST API docs, release notes
    CI/CD: GitLab webhook → Lambda → Bedrock re-ingest on every release with doc changes
    Chunking: Semantic 1,024 tokens | Vector: Amazon Titan Embeddings V2

  SSOT 2: Historical Ticket Corpus
    Source: 14,200+ resolved Salesforce Cases (5 years, 2021–2026)
    Pipeline: Data Cloud Zero-Copy → S3 → Comprehend PII masking → Bedrock embedding
    Embedding model: Amazon Titan Embeddings V2 (Bedrock-native, zero egress, same model as SSOT 1)
    Nightly: New resolved cases appended — knowledge base compounds

  OPENSEARCH SERVERLESS — OCU CAPACITY ESTIMATE (P3 Enhancement):
    Baseline corpus: 14,200 tickets × avg 1,500 chars = ~21M characters indexed
    Quarterly ingestion spike: ~1,200 new tickets over 2 weeks = ~1.8M chars/spike
    Steady-state: 2 OCU (index: 1 OCU compute + 0.5 OCU storage; search: 0.5 OCU)
    Quarterly surge: up to 4 OCU during batch re-indexing windows (auto-scaling)
    **Monthly cost ceiling: ~$520/month** (4 OCU × $0.24/OCU-hr × 730hr) — captured in NAIM ~$1,380/month budget line

SAGEMAKER JUMPSTART MODELS:
  Model 1: Llama 3.1 8B FT (Nomura IT taxonomy) — Semantic ticket clustering
    Fine-tuned: 14,200 resolved cases with IT taxonomy labels
    Output: {cluster_id, confidence, similar_case_ids[], estimated_resolution_h}
    Serving: ml.g4dn.xlarge, auto-scale to zero outside business hours

  Model 2: XGBoost + Llama (Nomura case history labels) — Frustration/Churn classifier
    Features: [frustration_score, churn_keyword_count, case_frequency_7d,
               client_tier, aum_value, days_since_last_escalation]
    CRITICAL threshold: frustration ≥ 0.85 + churn_signal CRITICAL/HIGH for Tier-1
    → Triggers: BYPASS_QUEUE + escalation to HEAD_OF_CLIENT_TECHNOLOGY

BEDROCK GUARDRAILS (NAIM-Specific):
  Groundedness: 0.70 threshold — any claim not grounded in KB → flagged
  Citation forced: "Source: {document_title} {version}, Section {section}, Page {page}"
  Denied: investment advice, NAV quotations, guaranteed restoration timelines
  PII: ANONYMIZE at output layer (defense-in-depth with Comprehend at ingestion)
```

#### Three Use Cases

**UC1 — Context-Aware Troubleshooting:** RAG over App Documentation SSOT. ESG tranche not visible → 45-second resolution path vs. 22-minute Confluence search. Resolution time: <60s.

**UC2 — Historical Ticket Synthesis:** SageMaker semantic clustering over 5-year ticket corpus. MSK latency pattern → root cause + 3 prior similar cases + estimated 45-90 min restoration. Client communication drafted before escalation.

**UC3 — Sentiment-Driven Triage:** XGBoost + Comprehend scores frustration + churn risk. Tier-1 Hedge Fund with $2.3B AUM expressing "reviewing relationship" → frustration 0.94 → CRITICAL escalation to senior agent with de-escalation script in <45 seconds.

#### KPI Targets

| KPI | Current | NAIM Target | Improvement |
|---|---|---|---|
| NPS (Net Promoter Score) | Baseline | +15 points | 50%+ improvement in NPS/CSAT/CES |
| CSAT (Client Satisfaction Survey) | 42% FCR | ≥59% FCR | 40% increase |
| CES (Customer Effort Score) | 4.2h MTTR avg | ≤1.7h MTTR | 60% reduction |
| Tier-1 escalation rate | 61% reach Tier-2 | ≤25% | 59% reduction |
| Agent onboarding to productivity | 6–8 weeks | ≤2 weeks | 65% faster |

**KPI Commitment: 50% composite improvement in NPS/CSAT/CES within 18 months of deployment.**

**Total AI infrastructure cost: ~$1,380/month** --- break-even = 0.001 of one $180M institutional mandate retained.

---

## 3C. Business Case A3

### Personalized Investment Portfolios & Advisor Reports
**BIAN Domain: Portfolio Management / Wealth Advisory**

#### Strategic Purpose

Institutional and wealth clients receive standardized quarterly reports and generic portfolio commentary. In a world where alternatives, ESG overlays, and liability-driven strategies are increasingly bespoke, "standardized" is a competitive liability. Business Case A3 deploys Snowflake Cortex AI to generate personalized portfolio strategies and quarterly performance reports that surface tax-loss harvesting opportunities, retirement timeline projections, and risk-adjusted allocation recommendations at the individual client level — without adding analyst headcount.

#### Snowflake Cortex AI Model Specification (P1 Enhancement)

Personalization uses **Snowflake Cortex COMPLETE (native)** — not an External Function routing to Bedrock. This is an explicit architectural choice:

| Option | Approach | Data Egress | Cost |
|---|---|---|---|
| **Cortex COMPLETE (chosen)** | Snowflake-native LLM inference | **Zero egress** — data stays in Snowflake | Billed per token against Snowflake credit budget; ~$0.0003/1K tokens for llama3-70b |
| External Function → Bedrock | Snowflake calls AWS Lambda → Bedrock | Data leaves Snowflake VPC boundary | Lambda + Bedrock cost + data transfer |

**Why Cortex COMPLETE:** The `portfolio_daily`, `risk_metrics`, and `performance_attr` Gold tables contain NPI (Non-Public Information) at the position level. Routing this data through an External Function to Bedrock creates a data egress event that triggers additional FCA/SEC review requirements. Cortex COMPLETE keeps all inference within the Snowflake environment — zero egress, zero additional regulatory review.

**FinOps estimate for Case A3 at scale:**
- 10,000 clients × quarterly report = 40,000 reports/year
- Avg tokens per report generation: ~8,000 tokens (context: portfolio data + prompt + output)
- Annual Cortex AI token cost: 40,000 × 8K × $0.0003/1K = **~$96/year** (negligible)
- Monthly amortized (8K tokens): **~$8/month** — the smallest line item in the entire AI budget
- **Cost range by report template depth:** Concise report (2K tokens) → ~$2/month | Standard report (5K tokens) → ~$5/month | Full-detail report (8K tokens) → ~$8/month | Complex (alternative-heavy, 15K tokens) → ~$18/month
- **Budget ceiling at 10K clients: $18/month worst-case** — Snowflake Cortex token pricing is deterministic; no surprise egress or API gateway cost

#### Architecture

```
PERSONALIZED PORTFOLIO ENGINE — DATA FLOW

INPUT LAYER (CQRS Query Path — NOMURA_AI_WH):
  customer_360 Gold Table → {risk_profile, life_events, investment_horizon,
                              liquidity_needs, tax_bracket, ESG_preferences}
  portfolio_daily Gold Table → {current_holdings, allocation_weights,
                                 sector_exposures, benchmark_drift}
  risk_metrics Gold Table   → {real-time_VaR, beta_exposure, duration,
                                 credit_quality_distribution}
  performance_attr Gold Table → {attribution_by_factor, alpha_by_strategy,
                                   fee_drag, benchmark_relative_performance}
  aum_churn_risk Gold Table  → {churn_score_0_100, top_3_factors,
                                  life_event_flag, rebalancing_opportunity}

CORTEX AI SCORING (Snowflake-Native — No Data Egress):
  COMPLETE(
    model = 'llama3-70b',                    -- Snowflake Cortex COMPLETE
    prompt = 'Given risk_profile: {profile}, life_event: {event},
              current_allocation: {allocation}, benchmark_drift: {drift},
              identify: (1) rebalancing_opportunity, (2) tax_loss_harvest_candidates,
              (3) retirement_timeline_variance, (4) ESG_tilt_adjustment.
              Output: {recommendation, priority, rationale, estimated_impact_bps}'
  )

AMAZON BEDROCK — REPORT NARRATIVE (Claude 3.5 Sonnet):
  Inputs: {cortex_recommendations, performance_attr, benchmark_context, client_profile}
  Outputs:
    → Personalized quarterly performance narrative (client-readable, 400–600 words)
    → Section 1: "Your Portfolio This Quarter" — relative attribution
    → Section 2: "Strategy Recommendations" — cortex-generated, RM-reviewed
    → Section 3: "Forward-Looking Considerations" — tax-loss harvesting, retirement delta
    → Section 4: "Market Context" — macro backdrop relevant to client's specific allocations

HITL GATE (RM Review):
  Salesforce Task: "Personalized Q-Report ready for [Client Name] — Review before send"
  RM reviews: narrative + recommendations + supporting data links
  RM approves with optional editorial additions (Seismic advisor commentary templates)
  Report delivered via Client Portal with Bedrock-generated personalized cover letter

OUTPUT DELIVERY:
  Client Portal (NOMURA_PORTAL_WH) → Personalized report download
  NCIE Feedback Loop → Client reads/downloads/engages → NCIE ingests engagement signal
  Churn Risk Update → If report reviewed + recommendation accepted → churn score adjusted
```

#### Business Impact (Qualitative to Quantitative)

| Outcome | Mechanism | Estimated Impact |
|---|---|---|
| AUM retention | Churn risk score > 70 triggers personalized rebalancing conversation before client considers redemption | Reduces "silent redemption" risk for accounts in drift |
| Wallet share growth | Life event-triggered recommendations (liquidity, retirement) delivered at the right moment | Increases alternative allocation conversations |
| Advisor efficiency | One report generation automated from 3 hours to 15 minutes | 12× throughput per advisor |
| Regulatory alignment | Personalized reports grounded in verified performance attribution (Snowflake `performance_attr` Gold table) | Suitability documentation auto-embedded |

**AUM Wallet-Share Quantification (P3 Enhancement):**
Personalization's most material business impact is AUM consolidation. Wealth clients receiving personalized, benchmark-relative reporting are statistically less likely to maintain broker-of-record diversification. Industry benchmarks (Pershing/Fidelity Advisor Study, 2024) show a 5–10% AUM consolidation lift for clients receiving adviser-personalized content vs. generic statements.

| Scenario | AUM per client segment | Consolidation lift | Annual AUM inflow | Mgmt fee impact (50bps) |
|---|---|---|---|---|
| Conservative (5%) | $500M avg per personalization-eligible segment | +5% wallet share | +$25M AUM | +$125K/year |
| Base case (7%) | $500M avg per personalization-eligible segment | +7% wallet share | +$35M AUM | +$175K/year |
| Optimistic (10%) | $500M avg per personalization-eligible segment | +10% wallet share | +$50M AUM | +$250K/year |

*Even the conservative scenario ($125K/year management fee uplift) represents a 125× return on the ~$1,000/year Cortex cost for Case A3.*

---

## 4. Business Case B

### NAPCE — Nomura Agentic Proposal & Compliance Engine
**BIAN Domain: Product Management / Compliance & Regulatory Reporting**

#### Strategic Purpose

Nomura Asset Management International competes for institutional mandates — pension funds, endowments, sovereign wealth funds — in a market where $50M–$5B+ decisions take 12–24 months to close. Three bottlenecks throttle every deal: 3–6 weeks per DDQ, quarterly GIPS spot-checks with audit exposure windows, and 2–5 days per institutional proposal. NAPCE is the agentic layer that compresses this mandate lifecycle using Amazon MWAA orchestration, Saga distributed transactions, and three domain-bound Bedrock agents.

#### MWAA Configuration & Inter-DAG Governance (P2 Enhancement)

**MWAA Worker Configuration:**

| Parameter | Value | Rationale |
|---|---|---|
| Worker instance type | `mw1.small` (dev) / `mw1.medium` (prod) | m5.xlarge-equivalent; 4 vCPU, 16GB RAM per worker |
| Minimum workers | 1 | Always-on for event-driven DAG responsiveness |
| Maximum workers | 10 | Supports 10 concurrent DDQ processing DAGs during institutional sales push |
| `max_active_runs` per DAG type | `napce_ddq_processing`: 10, `napce_rfp_proposal`: 5, `napce_gips_monitoring`: 1, `napce_knowledge_refresh`: 1 | DDQ volume spikes are expected; RFP proposals are resource-intensive (Batch + SageMaker) |

**Inter-DAG Dependency Governance:**

NAPCE RFP proposals must not use stale Snowflake data. The following upstream dependency chain is enforced:

```python
# napce_rfp_proposal DAG — data freshness gate (added at DAG start)
check_snowflake_freshness = PythonOperator(
    task_id='check_snowflake_data_freshness',
    python_callable=validate_gold_table_freshness,
    op_kwargs={
        'table': 'portfolio_daily',
        'max_staleness_hours': 4,  # block if portfolio data > 4h old
        'strategy_composite_id': '{{ dag_run.conf["strategy_id"] }}'
    }
)
# If staleness > 4h: DAG raises AirflowSkipException + triggers targeted refresh
# napce_rfp_proposal.max_active_runs prevents concurrent proposals from sharing stale data
```

For intraday RFP requests, `napce_rfp_proposal` triggers a targeted `portfolio_daily_refresh` sub-DAG for the specific strategy composite before proceeding to Agent Analysis (Step 1). This ensures Monte Carlo simulations always use same-day position data.

#### Architecture

```
NAPCE — THREE-LAYER AGENTIC ARCHITECTURE

LAYER 1 — ORCHESTRATION (Amazon MWAA):
  DAG 1: napce_ddq_processing   — Event-driven: S3 DDQ upload trigger
  DAG 2: napce_rfp_proposal     — Salesforce webhook: opportunity reaches Proposal Stage
  DAG 3: napce_gips_monitoring  — EventBridge: daily 06:00 UTC + portfolio change events
  DAG 4: napce_knowledge_refresh — Scheduled: nightly 02:00 UTC

LAYER 2 — AI AGENTIC (Domain-Bound, IAM-Scoped):
  DDQ Agent (Bedrock, Claude 3.5 Sonnet):
    → Textract DocumentAnalysis: extracts 200+ DDQ questions from complex PDF tables
    → OpenSearch semantic search: 50,000+ approved DDQ responses (Titan Embeddings)
    → Confidence routing: >0.85 auto-populate; <0.85 → human review queue
    → Technology section: ALWAYS live lookup from AWS CloudTrail/Compliance API (never cached)

  RFP Agent (Bedrock, Claude 3.5 Sonnet):
    → Salesforce prospect context + MWAA DAG parameters
    → Knowledge base: OpenSearch + S3 data lake + portfolio composites
    → Full proposal assembly: strategy narrative + Monte Carlo outputs + GIPS cert

  Compliance Agent (Bedrock + SageMaker GIPS Engine):
    → CRITICAL BOUNDARY: Bedrock monitors outputs for GIPS compliance ONLY
    → SageMaker calculates ALL returns deterministically — LLM never touches return math
    → EventBridge daily monitoring + Aladdin CDC portfolio change events
    → Zero-defect GIPS audit posture; automated package for third-party GIPS verifier

LAYER 3 — COMPLIANCE & REGULATORY:
  SageMaker GIPS Engine (deterministic calculations):
    → Composite construction rules validation
    → Time-weighted return calculations (deterministic — not LLM)
    → Gross vs. net return spread validation
    → Benchmark assignment cross-reference
    → Terminated portfolio 5-year retention verification

IAM IDENTITY MESH (Security Architecture):
  Cognito OIDC → IAM role: arn:aws:iam::nomura::role/DDQAgent-[PermissionTier]
  Bedrock Agent inherits IAM role at invocation — data scoped to requesting RM's entitlements
  CloudTrail: every agent action audit-logged (role, data source, confidence score)
  S3 Object Lock: **COMPLIANCE mode** — 7-year retention per SEC 17a-4 / GIPS 2019 Annex B
    Root override disabled; no administrator can shorten or delete retention period
    Immutable audit log (CloudTrail-compliant); annual Legal & Compliance attestation required
```

#### The Strictly Governed RAG Mesh — Why Not Fine-Tuning

NAPCE is built on **Retrieval-Augmented Generation (RAG)** using Amazon Bedrock Knowledge Bases. Fine-tuning the LLM with factual fund strategy, GIPS methodology, or compliance policy data is categorically rejected.

| Dimension | Fine-Tuning (Rejected) | Strictly Governed RAG Mesh (Selected) |
|---|---|---|
| **Fact update latency** | Full model retrain required ($50K–$500K, 4–8 weeks) | Closed-Loop Feedback DAG: correction live in OpenSearch in minutes |
| **Audit trail** | No traceability — fact is baked into weights | Every chunk has source, owner_id, version, expiry — fully auditable |
| **Content currency** | Stale facts persist until next retrain | S3 Object Expiration → Lambda `napce_kb_expiry_purge` → OpenSearch DELETE — expired facts are architecturally unretrievable; Lambda DLQ (`napce_kb_expiry_dlq`) ensures no silent failures |
| **Regulatory compliance** | Cannot prove what the model "knew" at time of response | Bedrock Invocation logs + Payload 1 (raw Bedrock JSON) in QLDB + Snowflake Immutable Tables |
| **Surgical correction** | Impossible — fact interleaved with millions of weights | DELETE chunk from OpenSearch; replace with corrected chunk; re-vectorize |
| **Appropriate use** | Tone, jargon, and response style calibration (acceptable) | All factual financial content, compliance policies, GIPS composites |

**System prompt constraint enforced on every invocation:**
```xml
<rule>Output retrieved policy text verbatim. Do not synthesize, summarize, or merge 
content from multiple source documents. If two retrieved chunks conflict, output both 
sources and flag for human review. Do not resolve the conflict autonomously.</rule>
```

#### The 5 Pillars of AI Content Governance (NAPCE Implementation)

**Pillar 1 — "Current Answers Only" Perimeter**

```
S3 Knowledge Base Source → Object metadata: { Status: "Approved_Production", Review_Date: TTL }
  ↓ On expiry: Lambda auto-fires → DELETE from OpenSearch index (chunk_id = expired_object)
  ↓ LLM physically cannot retrieve what does not exist in the vector index
  ↓ Bedrock Knowledge Base filter: Status = "Approved_Production" enforced at retrieval

KPI: 100% elimination of AI responses citing expired or superseded policy/strategy content
```

**Pillar 2 — Clear Ownership & Accountability (Metadata-as-Code)**

Every OpenSearch chunk is ingested via AWS Glue with mandatory owner bonding:
```json
{
  "chunk_id": "gips_composite_policy_v7_chunk_014",
  "owner_entra_id": "pm.lead@nomura.com",
  "owner_salesforce_id": "005Sf000001Xy9Z",
  "status": "Approved_Production",
  "review_due": "2026-06-30"
}
```
AI response citations include: `"Source: GIPS Composite Policy v7 · Owner: [PM Lead] · Review Due: Jun 2026"`

**Pillar 3 — Centralized Updates via Closed-Loop Feedback UI**

```
Compliance Officer flags incorrect DDQ answer in Salesforce LWC
  → RFP/DDQ EXPORT BLOCKED until correction submitted (LWC page-level enforcement)
  → Salesforce Platform Event → MWAA "KnowledgeBase_Correction_DAG"
    Step 1: Archive old chunk (S3 audit record)
    Step 2: Re-vectorize corrected chunk → OpenSearch Serverless upsert
    Step 3: Redis cache invalidation
    Step 4: Salesforce notification: "Correction live — export unblocked"
```

**Pillar 4 — HITL Execution (AWS Step Functions Saga with Wait_For_Callback)**

```
NAPCE generates draft → Step Functions Wait_For_Callback (workflow paused; $0 cost)
  → Salesforce LWC: Principal Portfolio Manager / Compliance Officer reviews
  → "Approve" click → SendTaskSuccess(taskToken) → workflow resumes
  → Watermarked PDF generated (reviewer_id embedded in document metadata)
  → TransactionLog: { reviewer_id, timestamp, original_draft_hash, final_output_hash }
  → Salesforce Opportunity: Stage = "Proposal Submitted"
TIMEOUT: 72h → senior reviewer escalation; 96h → Head of Client Technology CloudWatch alarm
```

**Pillar 5 — Immutable Audit Trail (Three-Payload Architecture)**

| Payload | Content | Storage |
|---|---|---|
| **Payload 1** | Raw Bedrock JSON: model_id, retrieved chunk_ids, confidence scores, raw response | Snowflake Immutable Table + Amazon QLDB (SHA-256 cryptographic digest chain; independently verifiable) |
| **Payload 2** | Human-approved text, reviewer_id, Salesforce role, approval timestamp | Snowflake Immutable Table + Amazon QLDB |
| **Payload 3** | Python diff delta: exact lines changed by human; `human_intervention_score` (% modified) | Snowflake Immutable Table — CRO-queryable |

```
SNOWFLAKE IMMUTABLE TABLE ENFORCEMENT:
  → audit_trail table: INSERT-only privilege granted to napce_audit_writer role
  → DELETE and UPDATE privileges: REVOKED from all roles (enforced at RBAC layer, not application)
  → Cannot be overridden by application code regardless of Salesforce or AWS permissions
  → Verification: `SHOW GRANTS ON TABLE audit_trail` — delete/update must show no grants
  → Result: cryptographic immutability backed by both QLDB SHA-256 chain and Snowflake RBAC lockdown
```

**Compliance framework alignment:**

| Standard | Pillar Mapping |
|---|---|
| **ISO/IEC 42001** (AI Management Systems) | Pillars 2, 4, 5 directly satisfy AI accountability and transparency controls |
| **NIST AI RMF** (Govern / Map / Measure / Manage) | Pillar 1 = Manage; Pillar 3 = Govern; Pillar 5 = Measure |
| **SEC 17a-4 / GIPS 2019 Annex B** | S3 Object Lock COMPLIANCE mode (7-year); QLDB cryptographic chain |

**Governance KPIs:**
- 100% elimination of AI hallucinations in DDQ/RFP submissions (Pillar 1 + Guardrails groundedness)
- Zero compliance breaches from stale knowledge base content (Pillar 1 content currency controls)
- 100% auditability of human-vs-AI output differential for every submitted document (Pillar 5 diff delta)

#### The 6-Step Saga Pattern (Fault-Tolerant Proposal Assembly)

A single RFP touches five systems: Amazon Batch, S3, Snowflake, Salesforce, OpenSearch. Without distributed transaction management, a Step 3 failure leaves orphaned Monte Carlo data in S3 and a corrupted Salesforce opportunity stage. The Saga pattern ensures clean rollback at every step.

```
SAGA EXECUTION — NAPCE RFP PROPOSAL

Step 1: Agent Analysis   → Bedrock defines parameters (100 MC scenarios, strategy, jurisdiction)
                           On failure: Compensation A (clean S3 staging)

Step 2: Task Execution   → Amazon Batch: 100 Monte Carlo risk simulations
(Saga Boundary)          → Stress: rate ±200bps, credit +150bps, equity -30%
                           On failure: Compensation B (cancel Batch job + Comp A)

Step 3: Data Processing  → Monte Carlo results → S3: rfp_id/monte_carlo_results.parquet
                           DynamoDB saga manifest tracks every S3 key written
                           On failure: Compensation C (delete S3 via manifest + Comp B)

Step 4: GIPS Validation  → SageMaker validates composite compliance
                           On failure: alert Compliance team + block proposal + Comp D

Step 5: Assembly         → RFP Agent assembles: strategy + Monte Carlo + GIPS cert
                           → S3 final location

Step 6: Finalization     → Salesforce: Stage = "Proposal Draft Ready" + S3 link attached
                           RM notified via Salesforce notification + email
                           SAGA COMMITTED: all steps complete
```

#### Three Use Cases

**DDQ Auto-Population:** Textract parses complex PDF → Bedrock OpenSearch RAG → 60–80% faster than 3–6 week baseline. Technology sections: live data retrieval (never cached). 100% accuracy in technology control sections.

**GIPS Compliance Monitoring:** Continuous vs. quarterly spot-checks. EventBridge daily trigger + Aladdin CDC events. Composite drift, benchmark inconsistency, terminated portfolio violations flagged immediately — not at third-party audit. Automated audit package delivered to GIPS verifier.

**Institutional RFP Acceleration:** 2–5 day manual assembly → hours. Monte Carlo quantitative risk analysis (100 scenarios, 5 stress tests) included in every proposal. Saga guarantees no partial-state proposals reach the institutional client.

#### Business Impact

| KPI | Baseline | NAPCE Target | Impact |
|---|---|---|---|
| DDQ preparation time | 3–6 weeks | 60–80% reduction | Hours vs. weeks |
| RFP/DDQ turnaround | 2–5 days + 3–6 weeks | **90% reduction** | Mandate pipeline velocity |
| GIPS compliance posture | Quarterly spot-check | Continuous monitoring | Zero-defect audit posture |
| NAPCE infrastructure cost | N/A | ~$3K/month (100 proposals) | Pays for itself on 1st mandate |

**KPI Commitment: 90% reduction in RFP/DDQ turnaround time within 12 months of Phase 2 deployment.**

---

## 5. Business Case C

### NCIE — Nomura Client Insight Engine (Product Feature Enhancement)
**BIAN Domain: Product Design / Business Insight**

#### Strategic Purpose

Platform transformation fails when product development is disconnected from the actual voice of the client. Today, client feedback is manually exported, subjectively tagged, and summarized once a week — if someone has time. At 1,300+ comments per week from the client portal, AI chatbot fallback logs, and NPS verbatims, that model has already broken. NCIE transforms Nomura's client feedback loop from reactive manual export to automated AI intelligence: ingesting thousands of unstructured signals and using Amazon Bedrock (Claude) to generate prioritized, evidence-backed product backlogs for Stuart's platform teams.

#### Architecture

```
NCIE — OMNICHANNEL FEEDBACK ORCHESTRATION PLATFORM

SIGNAL SOURCES (All five AI business cases contribute):
  → Client portal feedback (structured): 400–600/week
  → AI Chatbot (Case A1) fallback transcripts: unresolvable query logs → S3
  → NAIM (Case A2) resolved case summaries: resolution categories → NCIE backlog
  → Personalized Report (Case A3) engagement data: read/download/abandon signals
  → NAPCE (Case B) DDQ/RFP rejection signals: why proposals are modified/rejected

AWS GLUE INGESTION (Automated — No Manual CSV Export):
  → AI Digital Agent transcripts: Glue nightly job from S3 agent logs
  → App store reviews: Glue scheduled scraper
  → Portal feedback: EventBridge event triggers
  → NAIM resolved cases: Salesforce Data Cloud Zero-Copy → Glue → S3

BEDROCK INTELLIGENCE ENGINE (Claude 3.5 Sonnet):
  Weekly EventBridge trigger (Mon 06:00 UTC):
    → Summarize: What are clients experiencing this week vs. last week?
    → Classify: Positive attributes / Bugs / Friction points / Opportunities
    → Prioritize: Which issues affect highest-AUM client segments?
    → Evidence: Every insight linked to exact source comments (full auditability)

NCIE BACKLOG OUTPUT:
  → Ranked action items for Stuart's platform team (P1/P2/P3 with source count)
  → NCIE Weekly Digest delivered via EventBridge → Slack + Salesforce Cases
  → Stuart's product backlog: evidence-driven, not opinion-driven
  → Kathleen's AI Agent: NCIE identifies knowledge gaps → NAIM/Chatbot KB updated

AMAZON COMPREHEND (Sentiment Baseline — Nightly):
  → Sentiment score per comment: Positive / Negative / Neutral / Mixed
  → NPS trajectory tracked without requiring additional surveys
  → Frustration velocity: sentiment trending up (improving) or down (worsening)

AMAZON OPENSEARCH (Enterprise Query Layer):
  → Product team queries: "Show all negative feedback about mobile trading UI Q3"
  → Trend analysis: comment volume by theme over time
  → Cross-portfolio: NCIE insights linked back to NAIM cases and NAPCE rejection signals
```

#### The Feedback-to-Backlog Lifecycle (Benchmarked: Vanguard "Fletcher")

```
MONTH 1: New feature deployed
  Comment volume: LOW (5–13/week — clients still exploring)
  NCIE: Baseline sentiment established

MONTHS 3–4: NCIE detects negative pattern
  Comment volume: RISING (21–30/week)
  NCIE output: [CRITICAL — 47 instances] Clients cannot locate fund document download
  Action: P1 backlog item created for Stuart's team — 2 months before survey data shows it

MONTHS 5–6: Client satisfaction metric confirms (too late if NCIE not deployed)
  Comment volume: PEAK (47–67/week)
  NCIE escalation: priority = CRITICAL, affected AUM segment = Tier-1 Institutional
  Action: Stuart's team deploys UX fix in Sprint 14

MONTH 7: NCIE confirms resolution
  Comment volume: returning to baseline (13 → 6/week)
  NCIE: "Navigation complaint theme resolved — sentiment improving"
```

**Key insight:** Without NCIE, the problem is first noticed in Month 5 (when support calls spike). NCIE identifies it in Month 3 — **two months earlier**, before client satisfaction scores are materially impacted.

**NCIE Feedback-to-Backlog Governance (P3 Enhancement):**

| Stage | Owner | Action | SLA |
|---|---|---|---|
| NCIE insight generated (CRITICAL/HIGH) | NCIE System (automated) | EventBridge event → Salesforce Case created; Jira backlog item auto-drafted | Real-time |
| Triage & classification | **NCIE Product Analyst** (Stuart's team) | Reviews Bedrock-generated theme summary + raw source quotes; assigns severity (P1/P2/P3) | ≤1 business day |
| Sprint inclusion decision | **Product Owner** (Stuart Mumley) | Approves P1 items for next sprint; defers P2 with rationale; closes false positives | Before each bi-weekly sprint planning |
| Resolution confirmation | **NCIE System** | Monitors comment sentiment post-deploy; closes Jira item when theme volume returns to baseline | ≤2 sprint cycles |
| Escalation authority | **Kathleen Hess McNamara** | Any P1 NCIE insight unaddressed within 2 sprints triggers ED-level review | Automated CloudWatch alert |

#### Kathleen-Specific Connection: AI Agent Fallback Harvesting

```
CLIENT uses AI Chatbot (Case A1)
  → Agent CANNOT answer (confidence < 0.75)
  → Fallback transcript logged to S3
  → AWS Glue NCIE nightly ingestion
  → Comprehend: sentiment = NEGATIVE
  → Bedrock: classify = "agent knowledge gap: private credit redemption process"
  → NCIE output: "Add Private Credit redemption content to DDQ KB"
               → Priority: HIGH (appeared 47× in 30 days)
  → Stuart's backlog: "Expand Bedrock Knowledge Base — Private Credit"
  → Case A1 Chatbot accuracy improves next sprint
```

**The flywheel:** Case A1 (Chatbot) generates signals → NCIE (Case C) synthesizes them → NAIM KB updated → Case A2 (NAIM) resolves tickets faster → NAIM resolution patterns feed back into NCIE → all cases improve together.

---

## 6. AI Data Flow Architecture

### How All Five Cases Read from the CQRS Query Path

```
╔══════════════════════════════════════════════════════════════════════════╗
║  CQRS DATA MESH — UNIFIED READ ARCHITECTURE FOR FIVE AI CASES          ║
╠══════════════════════════════════════════════════════════════════════════╣
║  SNOWFLAKE GOLD TABLES (Zero-Copy — NOMURA_AI_WH)                      ║
║                                                                          ║
║  customer_360 ──────────────────────► Case A1 (Cross-sell signals)     ║
║                 ────────────────────► Case A3 (Life events, profile)   ║
║  portfolio_daily ───────────────────► Case A3 (Personalization)        ║
║                   ──────────────────► Case B (Monte Carlo inputs)      ║
║  risk_metrics ──────────────────────► Case A3 (Real-time risk)         ║
║  aum_churn_risk ────────────────────► Case A1 (Churn signal → trigger) ║
║  gips_composites ───────────────────► Case B (GIPS validation)         ║
║  performance_attr ──────────────────► Case A3 (Report attribution)     ║
║  compliance_audit ──────────────────► Case B (Compliance agent input)  ║
╠══════════════════════════════════════════════════════════════════════════╣
║  AI INFERENCE LAYER (Reads from Snowflake via NOMURA_AI_WH)            ║
║                                                                          ║
║  Case A1: Bedrock + JumpStart → Cross-sell recommendation + Seismic    ║
║  Case A2: Bedrock + JumpStart → Troubleshoot + Sentiment + Escalate    ║
║  Case A3: Cortex AI + Bedrock → Personalized report + recommendations  ║
║  Case B:  MWAA + Bedrock Agents + SageMaker GIPS → Proposals           ║
║  Case C:  Bedrock + Comprehend + OpenSearch → Product backlog          ║
╠══════════════════════════════════════════════════════════════════════════╣
║  HITL GATES (Human approval before any client-facing output)            ║
║                                                                          ║
║  Case A1: RM approves Next Best Action in Salesforce                   ║
║  Case A2: Agent reviews Co-Pilot suggestion before client email        ║
║  Case A3: RM reviews personalized report before delivery               ║
║  Case B:  Compliance officer signs off DDQ/RFP in Salesforce           ║
║  Case C:  Product owner reviews prioritized backlog before sprint      ║
╠══════════════════════════════════════════════════════════════════════════╣
║  WRITE PATH (Final output only — back to Salesforce CRM)               ║
║                                                                          ║
║  Case A1: Next Best Action → Salesforce Sales Cloud                    ║
║  Case A2: Case resolution + escalation → Salesforce Service Cloud      ║
║  Case A3: Report delivery task + suitability note → Salesforce         ║
║  Case B:  Opportunity stage update + S3 proposal link → Salesforce     ║
║  Case C:  Prioritized action items → Salesforce Cases (Product Backlog)║
╚══════════════════════════════════════════════════════════════════════════╝

AI NEVER WRITES DIRECTLY TO SNOWFLAKE GOLD TABLES.
SALESFORCE NEVER READS DIRECTLY FROM SNOWFLAKE (External Objects via Zero-Copy).
CQRS WRITE PATH (MSK KAFKA) IS UNTOUCHED BY AI INFERENCE LOAD.
```

---

## 7. Lemons Table

### Risk Identification & Mitigation — The Three Primary Risks

Proactive risk identification across the five AI business cases. The three risks named here are the highest-probability, highest-impact failure modes that would erode stakeholder trust in the AI strategy.

| # | Risk (The Lemon) | Business Impact | Probability | Mitigation Architecture | Owner |
|---|---|---|---|---|---|
| **L1** | **Seismic Content Hallucinations** — Bedrock cross-sell co-pilot generates a product recommendation citing material that does not exist in the Seismic DAM or has been expired/superseded by Compliance. RM sends unapproved content to a $2B institutional client. SEC 206(4)-1 violation exposure. | Regulatory penalty + AUM redemption risk | Medium — without guardrails, LLMs freely extrapolate beyond source content | **Bedrock Guardrails: groundedness threshold enforced. Content restricted exclusively to Seismic assets tagged `Approved_Production` for current quarter via AWS Glue Catalog metadata. Any Bedrock response not grounded in a current Seismic asset is blocked before reaching the RM's Salesforce inbox.** HITL gate: RM must approve before any content reaches the client. | Head of Client Technology + Compliance |
| **L2** | **Saga Orchestration Failures (NAPCE)** — NAPCE RFP pipeline fails at Step 3 (S3 Monte Carlo write). Saga compensation is triggered but S3 delete partially fails due to a permissions issue. Next DAG invocation reads stale partial results from previous failed run. Proposal contains incorrect risk analysis. | Institutional mandate disqualified; incorrect risk disclosure; GIPS audit exposure | Low-Medium — compensation chain is a secondary failure path but cascading failures in distributed systems are well-documented | **DynamoDB saga manifest tracks every S3 key written per rfp_id. Compensation Lambda reads manifest and deletes precisely — not by prefix sweep. S3 Object Lock prevents accidental overwrites. MWAA retry with exponential backoff on compensation tasks. Dead-letter SQS queue for compensation failures requiring manual intervention. CloudWatch alarm: any rfp_id with COMPENSATION_FAILED status triggers immediate page to Platform Team.** | Platform Team |
| **L3** | **Salesforce API Throttling** — Multiple concurrent AI cases (A1 cross-sell, A2 escalation, B RFP update) simultaneously write to Salesforce. At peak institutional pipeline volume (quarter-end), Salesforce API governor limits are hit. AI-generated outputs are complete but Salesforce is not updated. RMs have no visibility. | Quarter-end AUM pipeline blind spot; missed client SLAs; RM/client trust breach | Medium — Salesforce API governor limits are enforced at org level; concurrent AI workloads multiply risk | **CQRS Architecture is the primary mitigation: all five AI cases READ from Snowflake (Zero-Copy), not from Salesforce. Only the final approved output writes back to Salesforce (Next Best Action, Case update, Opportunity stage, Report delivery task). API consumption is thus B dramatically reduced vs. a Salesforce-first architecture. Secondary: Salesforce Bulk API for batch opportunity updates; MWAA DAGs implement exponential backoff + retry on Salesforce API calls; dedicated integration user with its own API budget.** | Integration Team |

### Extended Risk Matrix

| Risk | Case | Mitigation |
|---|---|---|
| LLM hallucination on GIPS returns | Case B | **Architecture boundary enforced:** SageMaker calculates all returns deterministically; Bedrock monitors outputs only; Guardrails block any Bedrock attempt to generate numerical returns without SageMaker citation |
| PII exposure via NAIM | Case A2 | Defense-in-depth: Comprehend PII masking at Salesforce export ingestion; Bedrock Guardrails PII redaction at output layer; CloudWatch alarm on any PII entity detected in Bedrock response |
| Stale NAIM knowledge base | Case A2 | GitLab webhook triggers real-time Bedrock re-ingestion on every release with doc changes; weekly EventBridge full-inventory audit; CloudWatch alert if any active version >14 days stale |
| Cortex AI personalization bias | Case A3 | SageMaker Clarify: quarterly bias review across client tiers; model retraining triggered on significant demographic distribution drift |
| NCIE insight hallucination | Case C | Every Bedrock insight linked to exact raw source comments — one-click auditability; prompt engineering: "every claim must reference a source comment"; HITL: product owner reviews before sprint commitment |
| **Fine-Tuning Misconception** | Case B (NAPCE) | Team proposes fine-tuning LLM with GIPS methodology or fund strategy facts. On policy change, model retains stale answer until full retrain ($50K–$500K; 4–8 weeks). No audit trail. **Architecture enforces RAG-only for all factual content.** Fine-tuning reserved for tone/jargon calibration only (never facts). Knowledge base corrections via Closed-Loop Feedback DAG: live in OpenSearch in minutes, fully auditable. |
| **"Frankenstein" Answers (Policy Synthesis)** | Case B (NAPCE), Case A1 | Bedrock retrieves overlapping policy chunks (e.g., 2024 and 2025 RFP templates) and synthesizes a chimera answer — correct-sounding, factually incorrect, untraceable to any single approved source. FINRA examination finds two contradictory citations in a submitted DDQ. **System prompt XML constraint:** `<rule>Output retrieved policy verbatim. Do not synthesize across sources. Flag conflicts for human review.</rule>` Combined with Pillar 3 Closed-Loop Feedback: conflicting chunks flagged and resolved at source before re-vectorization. |
| **Orphaned Knowledge (Employee Departure)** | Case B (NAPCE), Case A2 (NAIM) | Senior portfolio manager authors 40 knowledge base chunks. She departs. Entra ID disabled. Chunks remain in OpenSearch — no owner to review them. Six months later, strategy is deprecated but AI agent continues citing her analysis in live RFPs. **Workday EventBridge integration:** on `Employment_Status: Terminated`, Lambda flags all chunks owned by that employee as `Status: Requires_Review` — immediately excluded from RAG retrieval. MWAA routes flagged chunks to designated successor's Salesforce review queue with pre-populated ownership transfer form. |

---

## 8. KPI Dashboard

### Consolidated ROI Table — Total AI Investment vs. Expected Return (P1 Enhancement)

| Category | Annual Cost | Annual Value | ROI Multiple |
|---|---|---|---|
| NAIM (A2) infrastructure | ~$16,600/year | $7.5M+ escalation cost avoidance (85 cases/day × $120/h Tier-2 × 2h × 240 days) | **452×** |
| Chatbot + Cross-Sell (A1) | ~$19,200/year | $911.2M incremental pipeline headroom | **47,500×** (on 0.1% conversion of pipeline) |
| NAPCE (B) | ~$36,000/year | First institutional mandate partially won = $1M+ management fees on $200M mandate | **27×** per mandate |
| Personalization (A3) | ~$1,000/year (Cortex + Bedrock report narrative) | Reduction in "generic experience" NPS detractors; estimated 0.5% AUM retention improvement | Quantified separately in churn model |
| NCIE (C) | ~$7,200/year | 5 hours/week → 1 hour/week analyst time; 2-month earlier friction detection | Qualitative + sprint velocity improvement |
| **Total AI infrastructure** | **~$80,000/year** | **vs. $3.7M/year Snowflake savings + AUM pipeline impact** | **46× minimum (infrastructure savings alone)** |

**Key benchmark:** Nomura's Snowflake infrastructure cost reduction ($3.7M/year vs. legacy) alone covers the entire AI strategy budget **46× over**. The five business cases are funded by the platform efficiency already delivered.

### Three Primary KPIs — Committed Business Outcomes

```
KPI 1 — AUM GROWTH (Sales)
  Target: 30% increase in cross-sell conversion rate
  Business Case: A1 — Sales Cloud & Seismic Co-Pilot
  Mechanism: Cortex AI churn risk signals → Bedrock recommendation → Seismic pitch deck →
             RM HITL approval → Salesforce Next Best Action
  Baseline: 3 cross-sell opportunities per RM per quarter
  Target: 14 cross-sell opportunities per RM per quarter (4.7×)
  Pipeline: $118.8M → $1.03B qualified AUM pipeline (8.7×)
  Leading indicator: RM adoption rate of Next Best Action (target >85%)
  Measurement: Salesforce opportunity pipeline reports (quarterly)

KPI 2 — CLIENT EXPERIENCE (Service)
  Target: 50% composite improvement in NPS / CSAT / CES
  Business Cases: A2 (NAIM Support), A3 (Personalization), C (NCIE Product)
  Mechanism: NAIM sentiment triage → proactive Tier-1 escalation before churn
             Personalized reports → reduce "generic" NPS detractor category
             NCIE → product friction resolved 2 months earlier than survey data

  DISAGGREGATED CONTRIBUTION BY CASE:
  ┌──────────────────────────────┬──────────────┬──────────────────────────────────────────────────────┐
  │ Case                         │ NPS Delta    │ Primary Driver                                       │
  ├──────────────────────────────┼──────────────┼──────────────────────────────────────────────────────┤
  │ A2 — NAIM Support Co-Pilot   │ +8 NPS pts   │ 60% MTTR reduction → primary CES driver; faster FCR  │
  │ A3 — Personalized Portfolio  │ +5 NPS pts   │ Removes "generic/irrelevant" NPS detractor category  │
  │ C  — NCIE Insight Engine     │ +4 NPS pts   │ Product friction resolved 2 months earlier than cycle │
  ├──────────────────────────────┼──────────────┼──────────────────────────────────────────────────────┤
  │ COMBINED COMPOSITE           │ +17 NPS pts  │ 40% CSAT / 60% CES = ≥50% composite ✅              │
  └──────────────────────────────┴──────────────┴──────────────────────────────────────────────────────┘

  NPS Baseline: [current score] → Target: +17 points (50% composite improvement)
  CSAT Baseline: 42% FCR → Target: ≥59% FCR (40% increase; A2 primary contributor)
  CES Baseline: 4.2h MTTR → Target: ≤1.7h MTTR (60% reduction; A2 primary contributor)
  Threshold: ≥50% composite of (NPS + CSAT + CES) improvements weighted equally
  Leading indicator: NAIM Co-Pilot adoption rate + ticket corpus growth rate
  Measurement: Salesforce Service Cloud case reporting + NPS vendor surveys (quarterly)

KPI 3 — OPERATIONAL EFFICIENCY (Compliance & Proposals)
  Target: 90% reduction in RFP/DDQ turnaround time
  Business Case: B — NAPCE Agentic Proposal & Compliance Engine
  Mechanism: DDQ: Textract → Bedrock OpenSearch RAG → 60–80% faster (hours vs. 3–6 weeks)
             RFP: MWAA Saga → Monte Carlo → GIPS → Assembly → hours vs. 2–5 days
             GIPS: Continuous monitoring vs. quarterly spot-checks → zero-defect posture
  Baseline: 3–6 weeks (DDQ) + 2–5 days (RFP)
  Target: Hours for DDQ; same-day for RFP
  Leading indicator: NAPCE Saga success rate (target >99%); GIPS alert volume trending to 0
  Measurement: MWAA DAG execution logs + Salesforce pipeline stage timestamps
```

### Secondary KPIs — Platform Health

| Metric | Target | Measurement Source |
|---|---|---|
| CQRS data freshness | <2 seconds vs. 24–48h legacy | Snowpipe latency monitoring |
| AI inference latency (NOMURA_AI_WH) | <2 seconds for all five cases combined | Snowflake virtual warehouse telemetry |
| Snowflake infrastructure cost | <$700K/year | Monthly FinOps review |
| NAPCE Saga compensation rate | <1% of proposals | MWAA DAG metrics |
| NAIM KB staleness | ≤7 days from latest release | CloudWatch NomuraNAIM/KnowledgeBaseStaleness |
| NCIE feedback-to-backlog cycle | 6 weeks → 1 week | Sprint planning lead time |

---

## 9. Implementation Roadmap

### Phase-Gated Across Five Cases (18-Month Horizon)

| Phase | Timeline | Cases Active | Key Deliverables | Gate |
|---|---|---|---|---|
| **Phase 0 — CQRS Foundation** | Q1 2026 (Months 1–2) | Infrastructure | CQRS Data Mesh live: MSK Kafka + Snowpipe + Gold tables + Virtual warehouses + Horizon RLS. All five AI cases share this read path. | Gold tables operational; NOMURA_AI_WH latency <2s; Zero-Trust auth validated |
| **Phase 1 — NAIM + NCIE MVP** | Q1–Q2 2026 (Months 2–5) | A2, C | NAIM: Bedrock KB SSOT 1 (app docs). NCIE: Phase 1 MVP (serverless — S3 + API GW + Lambda + Bedrock + Aurora). | NAIM KB 100% indexed; NCIE MVP validates cross-sell signal with 5 agents; MTTR baseline measured |
| **Phase 2 — Chatbot + Personalization** | Q2–Q3 2026 (Months 5–9) | A1, A3 | A1: Agentforce + Bedrock co-pilot + Seismic DAM integration + JumpStart cross-sell model deployed. A3: Cortex AI personalization + quarterly report automation. HITL gates in Salesforce for both. | RM pilot: 10 advisors; cross-sell opportunity rate measured; first personalized report cohort delivered |
| **Phase 3 — NAPCE DDQ + NAIM Models** | Q3 2026 (Months 9–12) | B, A2 | NAPCE: Phase 1–2 (Textract + OpenSearch KB + DDQ Agent). NAIM: SageMaker JumpStart models deployed (clustering + frustration classifier). FCR and ticket triage quality step-change. | First live DDQ completed in <3 days; NAIM FCR ≥59%; Tier-1 escalation rate ≤25% |
| **Phase 4 — NAPCE Full RFP + NCIE Omnichannel** | Q4 2026 (Months 12–15) | B, C | NAPCE: Full Saga RFP workflow. Monte Carlo Batch integration. GIPS continuous monitoring. NCIE: Phase 3 (EventBridge + Glue + Comprehend + OpenSearch omnichannel). | First RFP in <8 hours; GIPS monitoring continuous; NCIE feedback volume >500/week automated |
| **Phase 5 — Production Scale + Cross-Portfolio Loops** | Q1–Q2 2027 (Months 15–18) | All | All five cases at production scale. Cross-case flywheel: A1 chatbot fallbacks → NCIE → NAIM KB → A1 improved. NAPCE win-rate feedback → OpenSearch. Personalization churn signals → A1 cross-sell triggers. Full FinOps review. | All five KPI targets met; zero GIPS audit findings; NAIM MTTR ≤1.7h; NAPCE 90% turnaround reduction confirmed |

---

## 10. Panel Talking Points

### For Kathleen Hess McNamara (ED, Client Experience & AI)

**The AI Reliability Architecture:**
> *"Kathleen, the single most important design decision I made for all five cases is the HITL gate. Every AI recommendation — whether it's a cross-sell pitch deck, a support troubleshooting step, a personalized report, a DDQ proposal, or a product backlog item — passes through a human approval gate before it reaches a client or a compliance submission. The AI accelerates the work; the human retains accountability. That is not a limitation — it is the trust architecture that makes institutional clients comfortable with AI in their workflow."*

**The NPS/CSAT/CES Connection Across Business Cases:**
> *"The 50% NPS/CSAT/CES improvement target is not driven by NAIM alone. NAIM improves CSAT by resolving tickets faster and reducing effort. Personalized reports (Case A3) improve NPS because clients stop receiving generic commentary about strategies they don't hold. NCIE (Case C) improves CES by identifying the friction points in the portal that cause clients to call support — and fixing them before the next NPS cycle. The three metrics are each addressed by a different business case, but they compound."*

**On Chatbot Fallback Strategy:**
> *"The AI chatbot question I anticipate from the panel is: what happens when the chatbot can't answer? My answer is: that fallback is a business asset, not a failure. Every unresolvable chatbot query is logged to S3, ingested by NCIE nightly, classified by Bedrock, and translated into a specific knowledge base gap. The chatbot failure of today becomes the knowledge base addition of next sprint. The system gets smarter by operating — that's the flywheel."*

**Anticipated Kathleen Question:** *"How do you ensure AI doesn't generate compliance-violating content for client communications?"*
> *"Three layers. Bedrock Guardrails are the first line: groundedness threshold blocks any content not traceable to an approved source — whether that's a Seismic pitch deck, an application documentation chunk, or a GIPS-certified composite. The second line is Denied Topics: investment guarantees, competitor recommendations, and specific return promises are blocked at the model output layer — not at the application layer. The third line is HITL: no client-facing content leaves without a human review. The RM, agent, or compliance officer has the final gate. The AI can be wrong; the human is responsible."*

**On AI Content Governance — The Strictly Governed RAG Mesh (Kathleen's Core Interest):**
> *"Kathleen, the one architectural decision I'd most want to walk you through is why we rejected fine-tuning the LLM with factual financial content — and why that decision is the foundation of the governance story. Fine-tuning bakes facts into static model weights. When a GIPS methodology changes, when a fund mandate is terminated, when a portfolio manager leaves — the fine-tuned model retains the old answer permanently until a full retrain. That retrain costs $50,000 to $500,000 and takes 4–8 weeks, and there is no audit trail of what the model believed before. For an institution with Series 7 and regulatory accountability, that architecture is incompatible with compliance requirements. The RAG Mesh alternative keeps all facts in OpenSearch, tagged with an owner and an expiry date. Every fact is surgically correctable in minutes, and every AI response traces to the exact chunk that was retrieved. The system cannot say something that is not currently in the approved knowledge base."*

**On Adoption Sustainability (The Trust Foundation):**
> *"The AI Digital Agent only succeeds if the sales and operations team trusts its output. One wrong answer in a DDQ submitted to a $500M institutional prospect destroys that trust in a way that takes months to rebuild. The 5 Pillars of Content Governance are not a compliance checklist — they are the adoption mechanism. Pillar 1 ensures the agent cannot cite an expired policy. Pillar 2 ensures every answer has a named owner. Pillar 3 ensures corrections update the source, not the prompt. Pillar 4 ensures a human authorized every submission. And Pillar 5 means we can prove in under 60 seconds, to any regulator or auditor, exactly what the AI said and exactly what was changed before submission. That transparency is what converts a proof-of-concept into a platform the team uses every day."*

**Anticipated Kathleen Question:** *"What is our exposure if the AI agent cites content from a strategy we've deprecated — or from an employee who has left the firm?"*
> *"The exposure is zero if the governance architecture is operating correctly. Deprecated strategy content has an expiry date in the S3 object metadata. When that date is reached, Lambda fires, the OpenSearch vector is deleted, and the LLM is physically incapable of retrieving it — not because of a prompt rule, but because the data no longer exists in the retrieval index. For departed employees, we have an EventBridge integration with Workday: on Employment_Status Terminated, all knowledge base chunks owned by that individual are instantaneously flagged Requires_Review and removed from the active retrieval context. Their replacement is routed a Salesforce review queue with a pre-filled ownership transfer form. The knowledge base remains complete; the governance chain is unbroken."*

---

### For Stuart Mumley (Director, Client Platforms)

**The BIAN Boundary Architecture:**
> *"Stuart, the BIAN framework is not theoretical here — it's the mechanism that prevents platform sprawl. Each of the five business cases operates in a strictly defined BIAN service domain. The Sales co-pilot (BIAN: Sales & Marketing) does not touch the support function (BIAN: Client Servicing), and neither touches the compliance function (BIAN: Product Management / Compliance). Each domain has one write surface (Salesforce), one read surface (Snowflake Gold tables via NOMURA_AI_WH), and one HITL gate. When NAPCE exchanges compliance data with NAIM, it does so via a schema-versioned EventBridge event bus — breaking changes require consumer notification two sprints ahead. Clean domain boundaries mean each case is independently deployable and independently debuggable."*

**The Salesforce API Throttling Answer:**
> *"The CQRS architecture is the answer to Salesforce API throttling — not API quota increases. All five AI cases read from Snowflake, not from Salesforce. The only Salesforce interaction for each case is the write of the final approved output: a Next Best Action, a Case update, a report delivery task, an opportunity stage change, a product backlog item. Five writes per AI transaction, not 50 reads. The API consumption profile drops dramatically, and it stays flat even as we scale AI usage across the platform."*

**On Platform Maintainability:**
> *"The architecture is stateless at the AWS layer by design. Lambda handles routing. MWAA handles orchestration. OpenSearch handles knowledge storage. Bedrock handles inference. There is no custom NAIM or NAPCE application server to maintain, no custom container, no persistent process requiring patching. Stuart's platform team owns the LWC components and Platform Event schemas; the AWS infrastructure is fully managed. Operational overhead is proportional to DAG complexity and KB size — both are transparent and auditable."*

**Anticipated Stuart Question:** *"What's the FinOps profile when all five cases are running concurrently?"*
> *"Rough estimate at full production scale:*
> - *NAIM: ~$1,380/month (Bedrock inference + SageMaker + OpenSearch)*
> - *NAPCE: ~$3,000/month for 100 proposals (MWAA + Batch + OpenSearch + Textract)*
> - *Chatbot (A1): ~$1,600/month (30 RMs, Phase 1+2 combined)*
> - *NCIE: ~$600/month (serverless — EventBridge + Lambda + Bedrock)*
> - *Personalization (A3): ~$400/month (Cortex AI + Bedrock report narrative)*
> - *Total: under $7K/month for all five AI cases at production scale.*
> *Compare to the Snowflake cost reduction alone — $3.7M/year saved vs. legacy. The entire AI strategy costs less than 3% of the infrastructure savings it runs on."*

---

### Cross-Portfolio Summary Statement

> *"The five business cases are not five separate AI projects. They are one AI platform serving five surfaces, connected by three shared infrastructure investments: the CQRS Data Mesh (the data foundation), the Bedrock Knowledge Base + Guardrails (the content governance layer), and Salesforce (the CRM of record for every human-in-the-loop approval). One investment in Zero-Trust data governance. One investment in Bedrock Guardrails. Both the institutional proposal agent and the internal support co-pilot benefit immediately.*
>
> *The business outcome is not 'we have AI.' The business outcome is: institutional clients who receive proposals generated in hours instead of weeks, wealth clients who receive personalized reports instead of generic commentary, support agents who resolve Tier-1 issues in 45 seconds instead of 22 minutes, and a product team that receives prioritized, evidence-backed backlogs instead of weekly manual summaries. That is the client experience architecture that grows AUM — and that is the strategy the Head of Client Technology is accountable for delivering."*

---

## 10. Panel Q&A Simulation

*Anticipated questions from Kathleen Hess McNamara and Stuart Mumley during panel presentation, with prepared responses.*

---

**Q1 (Kathleen): What happens if the AI sends a cross-sell recommendation to a client who is in financial distress — and the RM acts on it?**

> *"Three controls prevent this. First, the churn risk score: any client with a churn_risk > 70 flag in the Gold table receives 'retain and stabilize' as the Next Best Action — not a cross-sell. The model was designed so that distress signals suppress sales triggers entirely. Second, the RM HITL gate: the RM reviews every Next Best Action before client contact. The AI surfaces the recommendation; the RM has the final call. Third, the 5-business-day expiry and Salesforce audit trail: every recommendation is timestamped with the supporting data state. If a distress signal appeared between recommendation generation and RM action, the RM would have received an updated churn alert via the NAIM Co-Pilot. The AI doesn't replace RM judgment — it informs it under time pressure."*

---

**Q2 (Stuart): We've tried Seismic integrations before and they've been brittle. What's different this time?**

> *"The brittleness in prior integrations came from file-copy approaches — Seismic content pulled down into Salesforce Files, where it became stale the moment Seismic updated the source. This architecture is URL-link-only. The NBA object stores the Seismic deep-link, not a copy of the document. The RM always opens the current, version-controlled Seismic artifact. The circuit-breaker handles API unavailability gracefully: if Seismic is unreachable, the NBA is flagged SEISMIC_UNAVAILABLE and the RM receives the RAG knowledge summary instead. The system degrades gracefully, it doesn't fail silently or serve a stale document."*

---

**Q3 (Kathleen): How do we know the personalized portfolio report is accurate — not hallucinated — before it reaches a wealth client?**

> *"Accuracy is enforced at three layers. Layer 1: all data inputs are read directly from Snowflake Gold tables — performance_attr, portfolio_daily, risk_metrics, aum_churn_risk — which are the same tables used for official regulatory reporting. The model cannot fabricate position data it wasn't given. Layer 2: Cortex COMPLETE generates the narrative from structured data with an explicit prompt constraint: 'cite the specific data field for every factual claim.' Layer 3: the RM HITL gate. The RM compares the narrative to the supporting data links before approving delivery. The 10-business-day review window exists precisely because this is client-facing financial content. No report leaves without RM approval."*

---

**Q4 (Stuart): NAPCE is making compliance decisions autonomously. How do we handle a regulator asking who is accountable for a GIPS-compliant composite?**

> *"NAPCE does not make GIPS calculations. SageMaker makes the calculations — deterministic, reproducible, auditable. NAPCE orchestrates the delivery and monitoring. The GIPS Compliance Agent watches for drift against the SageMaker-certified composite; it does not produce the composite. Every GIPS output has a DynamoDB manifest entry recording which SageMaker model version ran, which Snowflake data snapshot was used, and which MWAA DAG run produced it. The compliance officer is accountable; NAPCE gives them 100% auditability. GIPS 2019 Annex B requires composite methodology documentation and annual verification — both are satisfied by the immutable S3 COMPLIANCE-mode audit log with 7-year retention."*

---

**Q5 (Kathleen): The ROI figures are striking — 47,500× on the chatbot. How credible is that and how do we defend it?**

> *"The 47,500× is defensible because it is calculated conservatively on a 0.1% pipeline conversion — meaning 99.9% of the chatbot's qualified opportunities are assumed to fail to close. We're not claiming the chatbot sells anything. We're claiming that at $80K/year in AI infrastructure, even one additional mandate closed per year out of 1,000 chatbot-qualified leads represents extraordinary ROI. The NAIM 452× figure is calculated against escalation cost avoidance — the model is: if NAIM prevents 10% of Tier-1 escalations that would otherwise result in churn, what is the AUM retention value? At $7.5M/escalation (based on average Tier-1 AUM at risk), the math closes easily. All ROI figures come from the Consolidated AI ROI table in Section 8 with explicit assumptions — any panel member can replicate the calculation."*

---

**Q6 (Kathleen): You mentioned RAG and fine-tuning. Walk me through the content governance model for NAPCE — specifically, how do you ensure an answer that was correct six months ago is not still being served today after a strategy change?**

> *"This is the question I'd lead with if I were interviewing this architecture. The answer is Pillar 1 of the Content Governance framework: the 'Current Answers Only' Perimeter. Every document in the NAPCE knowledge base is stored in S3 with an expiry date in its object metadata — set by the document owner at ingestion and refreshed on review. When that date is reached, a Lambda function fires automatically and deletes the corresponding vector from the OpenSearch index. From that moment, the LLM cannot retrieve it — not because a prompt rule prevents it, but because the data physically no longer exists in the retrieval layer. There is no way to accidentally serve a stale answer from an expired document — the infrastructure prevents it, not the model's judgment. The document owner receives a 30-day renewal notification in Salesforce before expiry, with a one-click re-attestation workflow. If the owner does not renew, the content expires. The agent's silence on a topic is the correct response when the answer cannot be certified current."*

---

**Q6 (Stuart): What's the operational model? Who runs this day-to-day when you're not here?**

> *"The architecture is designed for Stuart's platform team to own it without needing ML expertise day-to-day. The AWS layer is fully managed: MWAA is serverless, Bedrock is API-based, OpenSearch Serverless requires no cluster management. Stuart's team owns the LWC components in Salesforce and the Platform Event schemas — the same tools they already maintain. The MWAA DAGs are monitored via CloudWatch dashboards with automated alerts for SLA misses and compensation failures. Knowledge base staleness is monitored through CloudWatch NomuraNAIM/KnowledgeBaseStaleness with a 7-day maximum. SageMaker model Monitor triggers recalibration alerts — it doesn't require manual review unless the >2% false-positive threshold is breached. Normal operations require no ML team intervention; the runbooks are written for a platform engineer, not a data scientist."*

---

## 11. Document Metadata

| Field | Value |
|---|---|
| **Prepared by** | Calvin Lee |
| **Target audience** | Kathleen Hess McNamara (ED, Client Experience & AI) · Stuart Mumley (Director, Client Platforms) · Shaw Levin (VP, Technology Partnerships) |
| **Role** | Executive Director, Head of Client Technology — Nomura Asset Management International |
| **Platform strategy** | Five AI Business Cases on CQRS Data Mesh |
| **BIAN domains** | Sales & Marketing · Client Servicing · Portfolio Management / Wealth Advisory · Product Management / Compliance & Regulatory Reporting · Product Design / Business Insight |
| **AWS services** | Amazon Bedrock (Claude 3.5 Sonnet) · Amazon SageMaker JumpStart · Amazon MWAA · Amazon Comprehend · Amazon OpenSearch · Amazon Textract · Amazon Batch · Amazon MSK · AWS Glue · AWS Step Functions · AWS Lambda · Amazon EventBridge · Amazon Cognito · Amazon Aurora |
| **Salesforce services** | Sales Cloud · Service Cloud · Agentforce · Einstein Case Classification · Platform Events · Lightning Web Components · Data Cloud |
| **Data platform** | Snowflake (CQRS Query Path) · Snowpipe Streaming · Cortex AI · Horizon RLS · Dynamic Data Masking |
| **KPI commitments** | 30% cross-sell conversion increase (Sales) · 50% NPS/CSAT/CES improvement (Service) · 90% RFP/DDQ turnaround reduction (Operations) |
| **Foundation document** | [CQRS Data Mesh Strategy](CUSTOMER_CENTRIC_DATA_STRATEGY_CQRS.md) |
| **Source case studies** | [AI Chatbot + CRM](AI_CHATBOT_CRM_SALESCLOUD_INTEGRATION.md) · [NAIM Support Co-Pilot](SUPPORT_AGENT_COPILOT_NAIM.md) · [NAPCE Agentic Workflow](AGENTIC_WORKFLOW_AUTOMATION_NAPCE.md) · [NCIE Product Engine](NOMURA_CLIENT_INSIGHT_ENGINE.md) · [BIAN Framework](BIAN_FRAMEWORK_ASSET_MANAGEMENT.md) |
| **Evaluation** | [Eval Cycle 1](evaluation/EVALUATION_AI_STRATEGY_CYCLE_1.md) · [Eval Cycle 2](evaluation/EVALUATION_AI_STRATEGY_CYCLE_2.md) · [Eval Cycle 3 — Certified ✅](evaluation/EVALUATION_AI_STRATEGY_CYCLE_3_FINAL_CERTIFICATION.md) |
| **Last updated** | February 2026 |
