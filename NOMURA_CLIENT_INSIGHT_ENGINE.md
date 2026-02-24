# Nomura Client Insight Engine (NCIE)
## AWS AI-Powered Client Feedback Orchestration Platform
### Interview Case Study — Executive Director, Head of Client Technology
#### Target Audience: Kathleen Hess McNamara (ED, Client Digital Solutions) · Stuart Mumley (Director, Client Platforms)

---

> **Strategic Purpose:** NCIE transforms Nomura's client experience feedback loop from a reactive, manual weekly export into an automated AI engine that ingests thousands of unstructured client signals — including transcripts from Kathleen's AI Digital Agent — and uses Amazon Bedrock (Claude) to generate prioritized, evidence-backed product backlogs for Stuart's platform teams. It is the architectural bridge between client frustration and platform action.
>
> **Benchmark:** Informed by enterprise best practices from Vanguard's production "Fletcher" tool — an AWS-native feedback automation platform deployed at scale across financial services product teams.

---

## Table of Contents

1. [The Problem: From Manual Export to Automated Intelligence](#1-the-problem-from-manual-export-to-automated-intelligence)
2. [Why AWS: The Four Strategic Pillars](#2-why-aws-the-four-strategic-pillars)
3. [Value Proposition: Three Measurable Business Outcomes](#3-value-proposition-three-measurable-business-outcomes)
4. [Phase 1: The MVP — Fast, Painless, Malleable](#4-phase-1-the-mvp--fast-painless-malleable)
5. [Phase 2: The Orchestration Upgrade — Cleaner and Organized](#5-phase-2-the-orchestration-upgrade--cleaner-and-organized)
6. [Phase 3: The Target Architecture — Automated Omnichannel Ingestion](#6-phase-3-the-target-architecture--automated-omnichannel-ingestion)
7. [NCIE in Action: The Feedback-to-Backlog Lifecycle](#7-ncie-in-action-the-feedback-to-backlog-lifecycle)
8. [Proactive Risk Mitigation: The Lemons Table](#8-proactive-risk-mitigation-the-lemons-table)
9. [Interview Strategy: Talking Points for Kathleen and Stuart](#9-interview-strategy-talking-points-for-kathleen-and-stuart)
10. [Platform-to-BIAN Mapping for NCIE](#10-platform-to-bian-mapping-for-ncie)

---

## 1. The Problem: From Manual Export to Automated Intelligence

### The Current State (The "From")

| Manual Pain Point | Business Impact |
|---|---|
| Export weekly feedback CSV from the client portal | 3–5 hours per week of analyst time — zero value-add |
| Tag comments manually for sentiment, category, and theme | Subjective tagging; inconsistent across reviewers; no longitudinal trend analysis |
| Identify bugs, action items, and opportunities | Heavily dependent on individual reviewer's attention and domain knowledge |
| Send manually curated summary to the team each week | Slow, biased, and unscalable — breaks entirely above 200 comments/week |

**The trigger question that exposes the problem:** *"With Kathleen's AI Digital Agent now generating chat transcripts at scale, who is reading the fallback logs to understand what the agent couldn't answer — and why?"*

### The Future State (The "To")

```
CLIENT SIGNAL SOURCES
  → Client portal feedback (structured)
  → Kathleen's AI Digital Agent transcripts (unstructured)
  → App store reviews and NPS verbatims (unstructured)
        ↓
NCIE INGESTION LAYER
  → Automatic ingestion — no CSV export, no manual upload
  → Amazon Comprehend sentiment baseline per comment
        ↓
BEDROCK INTELLIGENCE ENGINE (Claude)
  → Summarize: What are clients experiencing this week?
  → Classify: Positive attributes / Negative experiences / Bugs / Opportunities
  → Prioritize: Which issues have the highest business impact?
        ↓
PRODUCT BACKLOG OUTPUT
  → Ranked action items for Stuart's platform teams
  → Each item linked to exact source comments (full auditability)
  → Delivered weekly via automated digest — no human curation required
```

**The business shift:** We move from asking *"How satisfied are you?"* to actively synthesizing *"What is causing friction, and what should we build next?"*

---

## 2. Why AWS: The Four Strategic Pillars

| Pillar | Description | Nomura Fit |
|---|---|---|
| **Existing AWS Foundation** | Nomura's "New Cloud" platform (EKS, MSK, S3 Medallion, ADX) provides the infrastructure substrate. NCIE is additive, not greenfield. | Reduced developer time — reuses IAM, VPC, security groups, and monitoring already in production |
| **Broad AWS Bedrock Model Support** | Plug-and-play access to Anthropic Claude, Amazon Titan, Cohere, and AI21 Labs — no model management overhead | Model can be swapped as the field evolves without re-architecting the pipeline |
| **Enterprise Security & Compliance** | Bedrock models are served entirely within the AWS VPC boundary — client feedback containing PII never leaves the Nomura environment | Satisfies FCA, SEC, and Nomura's internal data governance requirements without bespoke security engineering |
| **Rapid POC via Serverless** | Lambda + Step Functions + API Gateway = zero infrastructure provisioning, sub-hour deployment | MVP can be demonstrated to Kathleen and Stuart within 2 weeks; ROI validated before capital approval |

---

## 3. Value Proposition: Three Measurable Business Outcomes

| Business Outcome | Metric | Mechanism |
|---|---|---|
| **Quicker identification of client frustration** | Decreased client journey drop-off rates | Bedrock identifies friction patterns within 24 hours of comment ingestion vs. 1–2 week manual review lag |
| **Proposed action items to address client frustration** | Increased portal completion rates (onboarding, report download, AI agent interactions) | NCIE generates a ranked backlog — Stuart's team builds to evidence, not opinion |
| **Quicker time to implementation** | Decreased journey call rate (clients calling support to complete tasks they should finish digitally) | Feedback-to-fix cycle compresses from ~6 weeks (manual) to ~1 week (NCIE-driven automated triage) |

---

## 4. Phase 1: The MVP — Fast, Painless, Malleable

### Architecture Rationale

The Goal: Rapid POC deployment using serverless architecture to prove value without heavy infrastructure commitment. Directly benchmarked against Vanguard's initial Fletcher deployment pattern.

### Architecture Diagram

```
FRONTEND                        BACKEND                     AI ENGINE
────────                        ───────                     ─────────
React Web App                   API Gateway                 fletcher.lambda
  hosted on                          │                           │
  Amazon S3         ──────►    Fletcher              ──────► Bedrock
  + CloudFront              Authorizer Lambda                 (Claude 2.1)
  (global CDN)                       │                           │
                                      │                      Generated
Portfolio Managers                    ▼                      insights
& UX researchers              /prompts endpoint              saved to
upload CSV files                      │                           │
                              /saveSummary endpoint              ▼
                                      │                   Amazon Aurora
                              /retrieveSummary             (Relational DB
                               endpoint                     for stability)
                                      │
                               fletcher.save-results.lambda
                               fletcher.retrieve-results.lambda
```

### Component Breakdown

| Component | Technology Choice | Rationale |
|---|---|---|
| **Frontend** | React + TypeScript hosted on Amazon S3, distributed via Amazon CloudFront | Global CDN delivery with no server management; sub-100ms page load globally |
| **API Security** | Amazon API Gateway + authorizer Lambda | Only authenticated Nomura personnel access the tool; JWT validation at the gateway layer |
| **AI Processing** | AWS Lambda → Amazon Bedrock (Anthropic Claude) | Serverless execution means zero idle cost; scales to zero overnight |
| **Persistence** | Amazon Aurora (PostgreSQL-compatible) | Relational data model provides stability for structured insight storage; sets up Phase 2 deduplication |
| **Phase 1 Workflow** | Single Lambda per request | Acceptable for PoC volume (<200 comments/week); rapid iteration without orchestration overhead |

### Phase 1 MVP Characteristics

- **Fast:** Lambda functions allowed build, deploy, and iterate within hours — no container orchestration, no provisioning
- **Painless:** Used earliest available Bedrock models (Claude V2) with plug-and-play integration
- **Malleable:** Schema-less Aurora initially; easy to pivot data model as product understanding grows

### Phase 1 User Workflow

```
1. Select your team          → Dropdown: Wealth / Institutional / Platform
2. Upload Feedback CSV file  → Drag and drop, max 1,000 responses
3. Add custom prompt         → Optional: "Focus on mobile app navigation issues"
4. Process the feedback      → Bedrock generates summary in ~30 seconds
5. Review Feedback Digest    → Positive attributes + Negative experiences + Action items
                               Each insight links to the source comments
```

---

## 5. Phase 2: The Orchestration Upgrade — Cleaner and Organized

### The Problem This Phase Solves

Processing 1,300+ comments per week via a single Lambda function produces two production failures:
- **Timeout failures:** Lambda has a 15-minute maximum execution limit; large comment batches hit it
- **Duplicate data:** Without orchestration, concurrent uploads create duplicate insight records in Aurora

### Architecture Diagram

```
ENDPOINTS                    ORCHESTRATION              AI / STORAGE
─────────                    ─────────────              ────────────
/document-upload  ──────►  API Gateway                Step Function
                                  │              ┌──────────────────────┐
/retrieveSummary  ──────►  Fletcher         ──►  │  batch-create Lambda │
                           Authorizer            │         ↓            │
/logging          ──────►  Lambda                │  batch-prompt Lambda │
                                  │              │         ↓            │
                                  │              │  summary Lambda      │──► Bedrock
                                  │              │         ↓            │
                                  │              │  (deduplication)     │
                                  │              └──────────────────────┘
                                  │                        │
                                  │              Amazon Aurora
                                  ↓              (Fletcher-summary-results)
                           logging Lambda
                                  ↓
                           retrieve-results Lambda
                                  ↓
                           aurora Lambda
                                  ↓
                           Fletcher-summary-results (DB)
```

### Step Functions: The Deduplication Engine

AWS Step Functions orchestrates the workflow in three stages:

```
STATE: batch-create
  → Chunk incoming comments into batches of 50
  → Assign idempotency token per batch
  → Prevent duplicate batch creation if re-triggered

STATE: batch-prompt
  → Execute Bedrock Claude prompt per batch in parallel
  → Aggregate partial results
  → Retry failed batches with exponential backoff (max 3×)

STATE: summary
  → Deduplicate insights across batches (hash-based deduplication)
  → Merge insights by theme cluster
  → Write final deduplicated summary to Aurora
  → Trigger notification to team
```

### Phase 2 Characteristics

- **Cleaner:** Step Functions introduces an additional processing layer that de-duplicates summaries before they reach the database
- **Organized:** Orchestrated processing with full execution history and retry logic
- **Stable:** Relational data sets up Phase 3's searchable, cross-team analytics layer

---

## 6. Phase 3: The Target Architecture — Automated Omnichannel Ingestion

### The Architectural Leap

Phase 3 eliminates all manual ingestion. Instead of uploading CSVs, the system automatically ingests from three signal sources:

1. **Chat transcripts from Kathleen's AI Digital Agent** (via AWS Glue nightly job)
2. **App store reviews** (via AWS Glue scheduled scraper)
3. **Portal feedback** (via EventBridge event triggers)

### Full Architecture Diagram

```
SIGNAL SOURCES                EVENT TRIGGERS              AI ENRICHMENT
──────────────                ──────────────              ─────────────
/summaries   ──────►  API Gateway + Authorizer            Amazon Bedrock
/comments    ──────►     + logging Lambda                 (Claude — weekly)
/logging     ──────►          │                                 │
                              ▼                           Amazon Comprehend
                Weekly Event: EventBridge ──────────►    (sentiment — nightly)
                (Mon 06:00 UTC)                                  │
                              │                           Amazon OpenSearch
                              ▼                           (search index)
                       Step Function
                   ┌──────────────────────┐
                   │  batch-create Lambda │
                   │  prompt Lambda       │──► Bedrock
                   │  summary Lambda      │
                   └──────────────────────┘
                              │
                         Aurora + fletcher-db
                              │
                       ┌──────┴──────────┐
                       ▼                 ▼
Nightly Event:   nightly-job       comment-catalog
(Every night)    Lambda                  │
      │               │                 ▼
      ▼               ▼           comments S3
EventBridge    Comprehend          (raw archive)
               (sentiment
                per comment)
                    │
                    ▼
             OpenSearch
             (full-text search
              + trend queries)
```

### Phase 3 Component Roles

| Component | Role | Business Value |
|---|---|---|
| **AWS Glue** | Automated ingestion of Kathleen's AI agent transcripts, App store reviews, portal feedback CSVs | Zero manual work — team spends time acting on insights, not collecting them |
| **Amazon EventBridge** | Weekly trigger (Mon 06:00 UTC) for full processing; nightly trigger for incremental comment ingestion | Automated weekly digest to all platform teams without any human scheduling |
| **Amazon Comprehend** | Sentiment analysis on every individual comment (Positive / Negative / Neutral / Mixed) | Establishes a baseline sentiment score per client interaction — tracks NPS trajectory without surveys |
| **Amazon Bedrock (Claude)** | Summarization, insight generation, action item identification, opportunity classification | The intelligence layer — converts raw frustration into structured product decisions |
| **Amazon OpenSearch** | Full-text search index across all processed feedback and Bedrock summaries | Stuart's product team can query: *"Show me all negative feedback about the mobile trading UI in Q3"* — instant answer |
| **Amazon Aurora** | Structured storage for insights, action items, and audit trail | Relational stability for reporting; data product for Salesforce integration in Phase 4 |

### The Kathleen-Specific Connection: AI Agent Fallback Harvesting

```
CLIENT uses AI Digital Agent
        ↓
Agent CANNOT ANSWER (low-confidence response)
        ↓
Fallback transcript logged to S3
        ↓
AWS Glue NCIE ingestion job (nightly)
        ↓
Comprehend: sentiment = NEGATIVE (client frustrated)
        ↓
Bedrock: classify — "agent knowledge gap: private credit fund redemption process"
        ↓
NCIE output: Action item → Knowledge base gap for Private Credit team
             Priority score: HIGH (appeared 47× in last 30 days)
        ↓
Stuart's platform backlog: "Add Private Credit redemption content to DDQ knowledge base"
```

---

## 7. NCIE in Action: The Feedback-to-Backlog Lifecycle

### What "NCIE in Action" Looks Like (Benchmarked Against Fletcher)

The Vanguard Fletcher case study demonstrates the measurable impact of this architecture pattern:

```
MONTH 1: New feature implemented
  Comment volume: LOW (5–13/week — clients still exploring)
  NCIE status: Baseline established

MONTHS 3–4: Negative insight identified by NCIE
  Comment volume: RISING (21–30/week — frustration building)
  NCIE output: "Recurring theme: clients cannot find statement download in new nav"
  Action: Product team notified via automated digest

MONTHS 5–6: Drop in client satisfaction scores confirmed
  Comment volume: PEAK (47–67/week — issue widespread)
  NCIE escalation: Bedrock priority score = CRITICAL
  Action: Stuart's team creates P1 ticket; UX fix scoped

MONTH 7: Experience change implemented
  Comment volume: RETURNING TO BASELINE
  Comment volume: 13 → 6/week
  NCIE confirmation: "Navigation complaint theme resolved — sentiment improving"
```

**Key insight:** Without NCIE, the team would have noticed the problem in Month 5 (when support calls spike). With NCIE, the negative insight was flagged in Month 3 — **two months earlier**, before the client satisfaction drop became visible in survey data.

### The Feedback Digest Output Format

```
NCIE Weekly Digest — Week of [Date]
Team: Wealth Client Portal

Feedback results (4/3/2025 – 5/5/2025)

⚠️ NOTICE: All summarized outputs are created using generative AI.
   All insights are linked to source comments for verification.

POSITIVE (7 themes):
1. Users appreciate application reliability, comprehensive features, and
   user-friendly interface. The platform offers a smooth trading experience.
   [Comments ▼]

2. The application provides strong security measures and responsive customer
   support, which gives users confidence in the platform.
   [Comments ▼]

NEGATIVE (4 themes):
1. [CRITICAL — 47 instances] Clients cannot locate the fund document download
   function after the November navigation redesign.
   [Comments ▼]

ACTION ITEMS (3 items):
1. [P1] Restore direct Fund Documents link to primary navigation bar
   Source: 47 comments across wealth channel and advisor portal
2. [P2] Add search functionality to document library
   Source: 23 comments requesting "find my statement" capability
3. [P3] Evaluate mobile app session timeout — clients report frequent re-login
   Source: 18 comments
```

---

## 8. Proactive Risk Mitigation: The Lemons Table

| Risk Area (The "Lemon") | Proactive Mitigation Strategy (The "Squeezer") | Value Delivered |
|---|---|---|
| **Manual Review Fatigue** | NCIE automates the read-tagging-summarize workflow. Human effort shifts from reading hundreds of comments to acting on AI-generated, prioritized proposals. | UX team time redirected from 5 hours/week of data wrangling to 1 hour/week of decision-making |
| **Generative AI Hallucinations** | UI explicitly states "All summarized outputs are created using generative AI." Every single insight links directly to the exact raw client comments that generated it — full auditability. | Stuart can verify any AI claim in one click; compliance team can audit AI outputs |
| **Enterprise PII & Security** | Bedrock models consumed entirely within the Nomura VPC. Client feedback containing PII never traverses the public internet. Bedrock's architecture provides data isolation by default. | Seamless compliance with FCA, SEC, and GDPR without bespoke security engineering |
| **Lambda Timeout at Scale** | Phase 2 Step Functions orchestration batches 1,300+ comments into parallel 50-comment batches, each processed within Lambda limits. | Scales to any comment volume without architectural rework |
| **Data Duplication** | Step Functions idempotency tokens + hash-based insight deduplication before Aurora write. | Clean, reliable data product from day one |
| **Bias in Insight Generation** | Bedrock prompt engineering explicitly instructs: "Synthesize insights without assumption or editorial bias. Every claim must reference a source comment." | Reduced cognitive bias vs. manual analyst-driven thematic review |
| **Stale AI Knowledge** | Bedrock model version pinned in Lambda environment variable. Model upgrade path: swap environment variable, run regression test suite, deploy with zero downtime. | Model upgrades require hours, not a re-architecture |

---

## 9. Interview Strategy: Talking Points for Kathleen and Stuart

### For Kathleen — The Client Experience & AI Agent Angle

**The Hook:**
> *"Kathleen, as you implement the new AI Digital Agent, your biggest challenge won't be answering client questions — it will be understanding why clients abandon the chat. Currently, reviewing those fallback logs is a manual, unscalable process. There is no systematic way to turn those failure transcripts into product decisions."*

**The Architecture Bridge:**
> *"By routing your agent's fallback transcripts into NCIE via an AWS Glue nightly job, we use Amazon Bedrock to synthesize those frustrations without bias. We run every comment through Amazon Comprehend to establish a baseline sentiment score. Then Bedrock classifies each fallback — agent knowledge gap, product friction, or user error — and generates a ranked list of knowledge base additions for your agent and UX fixes for the portal."*

**The Clincher:**
> *"The critical governance control: every AI-generated insight links directly to the exact source comments. When compliance asks 'how do you know clients are frustrated about the redemption process?', we can show them the 47 transcripts, not a summary. That auditability is what makes AI-generated insights trusted rather than questioned."*

---

### For Stuart — The Platform & Digital Transformation Angle

**The Hook:**
> *"Stuart, platform transformation fails when product development is disconnected from the actual voice of the client. Today, feedback is manually read, subjectively tagged, and summarized once a week — if someone has time. At 1,300+ comments a week, that model has already broken."*

**The Architecture Narrative:**
> *"I want to build a continuous feedback loop: Listen, Synthesize, and Refine. NCIE implements this in three evolutionary phases that directly mirror the Vanguard enterprise pattern. Phase 1 is a fast, malleable MVP — CloudFront, API Gateway, Lambda, Bedrock, Aurora. We validate the concept in two weeks. Phase 2 introduces Step Functions for deduplication and batch orchestration. Phase 3 is the target architecture: EventBridge triggers, Glue ingestion from Kathleen's agent, Comprehend for sentiment, OpenSearch for enterprise-wide query."*

**The Clincher:**
> *"We do not need a massive capital expenditure to start. The MVP uses serverless architecture — the only cost is per invocation. Once we prove the ROI — and we will, because the Vanguard benchmark shows a measurable drop in support call rates within 60 days — we scale. The business case pays for Phase 2 and 3 out of the cost reduction in manual review hours and client support volume."*

---

### The Unified Message: Bridging Kathleen and Stuart

> *"Here is why this matters for both of you simultaneously. Kathleen's AI Digital Agent generates the raw signal — every time the agent cannot answer, it surfaces a knowledge gap or a product friction point. NCIE converts that signal into Stuart's product backlog. The loop is: Kathleen deploys the agent, NCIE harvests the gaps, Stuart's team closes them, the agent improves. That is a continuous improvement architecture — not a quarterly survey."*

---

### The "Fletcher in Action" Benchmark Response

When the panel asks: *"Do you have evidence this architecture pattern delivers business outcomes?"*

> *"Yes — Vanguard's production implementation of this exact architecture, internally called Fletcher, demonstrates a clear causal chain: a negative insight identified in Month 3 by the AI engine, a satisfaction drop confirmed by metrics in Months 5–6, and a return to baseline in Month 7 after the UX fix was deployed. The elapsed time from 'problem identified' to 'client satisfaction restored' was approximately 4 months. Without the automated insight engine, the problem would not have been identified until Month 5 when call volume spiked — meaning two months of avoidable client frustration and support cost."*

---

## 10. Platform-to-BIAN Mapping for NCIE

| NCIE Component | BIAN Service Domain | Behavior Qualifier | Integration Note |
|---|---|---|---|
| **AWS Glue ingestion** | Document Management | Initiate, Collect | Receives and classifies incoming unstructured feedback signals |
| **Amazon Comprehend** | Data Management & Integration | Collect, Maintain | Enriches raw comments with sentiment scores before Knowledge Management |
| **Amazon Bedrock (Claude)** | Knowledge Management + AI-Assisted Processing | Retrieve, Execute | Grounded in source comments; outputs to human review queue for flagged items |
| **Step Functions** | Onboarding Orchestration (reused pattern) | Initiate, Execute, Complete | Batch orchestration with idempotency — same pattern as client onboarding workflow |
| **Amazon OpenSearch** | Data Management & Integration | Distribute, Maintain | Provides full-text search across historical insights for product teams |
| **NCIE Digest Output** | Sales Enablement & Advisor Distribution | Evaluate | Delivers actionable product priorities to Stuart's platform teams via automated notification |
| **Amazon Aurora** | Data Management & Integration | Maintain, Audit | Structured persistence of all insights with full lineage to source comments |
| **Salesforce Integration (Phase 4)** | Customer & Party Management | Update, Monitor | NCIE-generated action items flow as Cases into Salesforce Service Cloud for tracking |

---

## Document Metadata

| Field | Value |
|---|---|
| **Prepared by** | Calvin Lee |
| **Target audience** | Kathleen Hess McNamara (ED, Client Digital Solutions), Stuart Mumley (Director, Client Platforms) |
| **Role** | ED, Head of Client Technology — Nomura Asset Management International |
| **Purpose** | Interview case study demonstrating AWS AI architecture for client experience improvement |
| **Benchmark reference** | Vanguard "Fletcher" — AWS-native feedback automation (presented at AWS FinTech summit 2025) |
| **Related documents** | [README.md](README.md) · [BUSINESS_PARTNERSHIP_README.md](BUSINESS_PARTNERSHIP_README.md) · [BIAN_FRAMEWORK_ASSET_MANAGEMENT.md](BIAN_FRAMEWORK_ASSET_MANAGEMENT.md) |
| **Last updated** | February 2026 |

---

*NCIE is not a product pitch — it is an architectural demonstration. It shows Kathleen that her AI Digital Agent's output has a home, a processing pipeline, and a direct path to product improvement. It shows Stuart that platform prioritization can be driven by evidence at scale, not by the loudest voice in the room. And it shows both that the candidate understands the full loop: client signal → AI synthesis → product action → outcome measurement.*
