# AI Chatbot for CRM Sales Cloud Improvement & Cross-Selling
## Salesforce Agentforce · Amazon Bedrock · SageMaker JumpStart · DAM · Sales Cloud
### Case Study 2 — Executive Director, Head of Client Technology
#### Target Audience: Kathleen Hess McNamara (ED, Client Digital Solutions) · Stuart Mumley (Director, Client Platforms) · Shaw Levin (VP, Technology Partnerships)

---

> **Strategic Purpose:** This case study articulates the architecture and business rationale for deploying a Salesforce Agentforce AI chatbot — powered by Amazon Bedrock or SageMaker JumpStart — to transform how Nomura's distribution teams access content, identify cross-sell opportunities, and orchestrate personalised client engagement. The integration creates an **Ambient Content Layer**: the right asset, reaching the right person, at the right moment — without search, without context-switching, and without compliance risk. This is not a technology initiative; it is a distribution advantage measured in AUM.
>
> **Integration heritage:** Built on the three-layer CRM-to-Insights architecture established in [DIGITAL_LAYER_SALESFORCE_INTEGRATION.md](DIGITAL_LAYER_SALESFORCE_INTEGRATION.md) and the AI feedback loop architecture in [NOMURA_CLIENT_INSIGHT_ENGINE.md](NOMURA_CLIENT_INSIGHT_ENGINE.md). This case study extends the platform from analytics delivery to conversational intelligence.

---

## Table of Contents

1. [The Problem: Distribution Friction at Scale](#1-the-problem-distribution-friction-at-scale)
2. [The Solution: Three-Layer Architecture](#2-the-solution-three-layer-architecture)
3. [AI Engine Decision: Amazon Bedrock vs. SageMaker JumpStart](#3-ai-engine-decision-amazon-bedrock-vs-sagemaker-jumpstart)
4. [Salesforce Agentforce: Three Core Use Cases](#4-salesforce-agentforce-three-core-use-cases)
5. [Cross-Selling Intelligence Engine](#5-cross-selling-intelligence-engine)
6. [DAM Platform Evaluation for Nomura](#6-dam-platform-evaluation-for-nomura)
7. [Salesforce Sales Cloud as Orchestration Layer](#7-salesforce-sales-cloud-as-orchestration-layer)
8. [Implementation Roadmap](#8-implementation-roadmap)
9. [Proactive Risk Mitigation: The Lemons Tables](#9-proactive-risk-mitigation-the-lemons-tables)
10. [Interview Strategy: Talking Points for Kathleen, Stuart, and Shaw](#10-interview-strategy-talking-points-for-kathleen-stuart-and-shaw)

---

## 1. The Problem: Distribution Friction at Scale

### The Current RM Content Workflow — Five Failure Points

When a Nomura RM or wholesaler needs a specific piece of content for an imminent client interaction, the current workflow is:

| Step | Current Manual Action | Time Cost | Risk |
|---|---|---|---|
| 1 | Remember where the content might be (Seismic? Intranet? Email?) | 2–5 min | Cognitive load; depends on individual memory |
| 2 | Search across Seismic, intranet, email attachments, or shared drives | 5–10 min | Wrong system first; search terms insufficient |
| 3 | Check whether the version found is current and compliance-approved | 3–5 min | Easy to miss — version date is not always obvious |
| 4 | Download and attach to email or meeting prep | 2–3 min | File may be outdated if cached locally |
| 5 | Log the content retrieval activity in Salesforce | **Usually skipped** | Activity history incomplete; Einstein cannot learn |

**Total: 12–23 minutes per content retrieval. Multiplied across 30 RMs × 10 retrievals/week × 50 weeks = 180,000 – 345,000 minutes lost annually to mechanics, not selling.**

### The Fundamental Mis-alignment

```
WHAT EXISTS:
  Seismic library      → RMs must remember to visit
  DAM / intranet       → RMs must know the taxonomy
  Salesforce           → CRM is a logging burden, not an intelligence engine

WHAT SHOULD EXIST:
  RM opens [Institutional Pension Fund] in Salesforce
  AI chatbot (Agentforce): "You have a meeting with [fund] in 47 minutes.
    They are in due diligence for Global Fixed Income.
    Here are the 3 pieces of content most used in this stage with similar clients.
    The CIO previously engaged with Q3 FI commentary — the February update is out.
    One click to pre-attach to your meeting prep note."

  Time: 0 seconds of search. Content: compliance-approved, jurisdiction-correct, version-current.
```

### Why Cross-Selling Fails Without This Integration

| Cross-Sell Signal | Current State | Integrated State |
|---|---|---|
| Client 8% allocated to FI vs. 15% benchmark | PM knows; RM doesn't; Salesforce doesn't | PowerBI surfaces under-allocation → Agentforce creates Suggested Task → RM receives alert with recommended FI content |
| Client engaged with Private Credit factsheet 3× | Seismic knows; Salesforce doesn't; RM doesn't | Seismic LiveSend → EventBridge → Salesforce Engagement Activity → Agentforce: "High intent signal — suggest Private Credit fund call" |
| Competitor filing shows client expanding alternatives mandate | External signal | SageMaker fine-tuned model + Comprehend entity extraction → surfaces as Opportunity Alert in Salesforce |

---

## 2. The Solution: Three-Layer Architecture

### Full System Architecture

```
╔══════════════════════════════════════════════════════════════════════════════╗
║  LAYER 1: CONVERSATIONAL AI  (Salesforce Agentforce — User-Facing)          ║
║                                                                              ║
║  RM / Wholesaler asks in natural language within Salesforce:                 ║
║  ┌──────────────────────────────────────────────────────────────────────┐   ║
║  │  "Find the latest EM debt fund fact sheet for institutional UK"       │   ║
║  │  "Show compliance-approved Global High Yield pitch deck"             │   ║
║  │  "What has changed in the Fixed Income commentary this month?"       │   ║
║  │  "Which of my APAC clients are underweight alternatives this Q?"     │   ║
║  └──────────────────────────┬───────────────────────────────────────────┘   ║
║                             │ Intent + Salesforce Account Context           ║
╚═════════════════════════════╪════════════════════════════════════════════════╝
                              │
                              ▼  Amazon Bedrock (Claude) OR SageMaker JumpStart
╔═════════════════════════════╪════════════════════════════════════════════════╗
║  AI INFERENCE LAYER         │  (See Section 3 for Bedrock vs JumpStart)      ║
║                             ▼                                                ║
║  Intent Parser: {"intent": "content_retrieval",                              ║
║                  "asset_class": "EM_debt",                                   ║
║                  "content_type": "factsheet",                                ║
║                  "audience": "institutional",                                ║
║                  "jurisdiction": "UK",                                       ║
║                  "recency": "latest"}                                        ║
╚═════════════════════════════╪════════════════════════════════════════════════╝
                              │ Structured query to DAM API
╔═════════════════════════════╪════════════════════════════════════════════════╗
║  LAYER 2: DAM               ▼  (Seismic / Aprimo / Bynder)                  ║
║                                                                              ║
║  Content Repository:                                                         ║
║  ┌──────────────────────────────────────────────────────────────────────┐   ║
║  │  AI-powered auto-tagging: content_type, asset_class, audience,       │   ║
║  │                             jurisdiction, expiry date, version       │   ║
║  │  Compliance workflow state: DRAFT → LEGAL_REVIEW → APPROVED          │   ║
║  │  Version control: only APPROVED and CURRENT assets surfaced to RM    │   ║
║  │  Search API → returns asset_id + signed S3 URL + metadata            │   ║
║  └──────────────────────────┬───────────────────────────────────────────┘   ║
║                             │ Asset metadata + signed download URL           ║
╚═════════════════════════════╪════════════════════════════════════════════════╝
                              │
╔═════════════════════════════╪════════════════════════════════════════════════╗
║  LAYER 3: CRM ORCHESTRATION ▼  (Salesforce Sales Cloud + Service Cloud)     ║
║                                                                              ║
║  Salesforce Financial Services Cloud:                                        ║
║  ┌──────────────────────────────────────────────────────────────────────┐   ║
║  │  Account context: client type, AUM tier, jurisdiction, stage         │   ║
║  │  Opportunity: strategy of interest, decision timeline, stage         │   ║
║  │  Activity log: asset retrieved + sent (auto-logged — never skipped)  │   ║
║  │  Seismic LiveSend: engagement tracked (opened, time spent, pages)    │   ║
║  │  Cross-sell trigger: engagement signal → Einstein Suggested Task      │   ║
║  └──────────────────────────────────────────────────────────────────────┘   ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

### The Compliant-Path Architecture Principle

The most important design choice in this architecture is: **make the compliant path the path of least resistance**.

```
BEFORE INTEGRATION:
  Compliant path:  RM searches Seismic → finds content → checks version → logs in Salesforce
  Non-compliant path: RM emails themselves a cached file → sends to client
  Reality: 40–60% of the time, the non-compliant path wins (it is faster)

AFTER INTEGRATION:
  Compliant path:  RM asks Agentforce → receives approved content → one-click send → auto-logged
  Non-compliant path: still exists but is now harder than asking the chatbot
  Result: Architecture makes compliance the default — not discipline, not training, not policy
```

---

## 3. AI Engine Decision: Amazon Bedrock vs. SageMaker JumpStart

### The Strategic Decision Framework

Choosing between Amazon Bedrock and Amazon SageMaker JumpStart is not a binary decision — it is a phased adoption strategy that starts with Bedrock for rapid deployment and selectively introduces SageMaker JumpStart where financial-services-specific fine-tuning delivers measurably better results.

### Feature Comparison: Bedrock vs. JumpStart for Nomura

| Dimension | Amazon Bedrock | Amazon SageMaker JumpStart |
|---|---|---|
| **Infrastructure management** | Fully managed and serverless — AWS manages all compute | User-managed EC2 compute instances; requires MLOps configuration |
| **Deployment effort** | Single API call — deploy in hours | Instance provisioning, container config, endpoint setup — days |
| **Model availability** | Curated: Anthropic Claude, Amazon Titan, Cohere, Meta Llama, Mistral | Extensive: Hugging Face (70K+), SageMaker built-in, proprietary models |
| **Model customisation** | Limited — prompt engineering, RAG knowledge bases, guardrails | Deep — full fine-tuning with Nomura proprietary data, architecture modification |
| **Context window** | Claude 3.5: 200K tokens (batch of 1,300 comments) | Model-dependent; Llama 3.1 70B: 128K; custom fine-tuned models: configurable |
| **Target user profile** | Developers seeking rapid integration; low MLOps overhead | ML practitioners needing domain-specific model performance |
| **Cost model** | Pay-per-token (input + output); no idle cost | Instance-hour billing; idle instances incur cost even when not inferring |
| **Financial services compliance** | AWS BAA coverage; Bedrock Guardrails for hallucination + PII + toxicity | Configurable; requires manual compliance tooling configuration |
| **RAG (Retrieval-Augmented Generation)** | Native Knowledge Bases — embed Seismic content, NCIE digests, DDQ answers | Requires LangChain/LlamaIndex integration; more flexible but more setup |
| **Integration with SageMaker suite** | Limited (Bedrock agents can call SageMaker endpoints) | Full: Pipelines, Clarify (bias detection), Model Monitor, Feature Store |
| **Multi-model orchestration** | Bedrock agents: chain multiple models and tools in managed environment | Requires custom orchestration layer (Step Functions + Lambda) |

### Nomura Deployment Decision Matrix

```
USE AMAZON BEDROCK WHEN:
  ✅ Phase 1 MVP chatbot deployment (Agentforce natural language → DAM API)
  ✅ NCIE feedback summarisation (Claude 200K context for batch processing)
  ✅ DDQ / RFP response generation (managed guardrails for compliance)
  ✅ Content freshness alerts (simple prompt over metadata)
  ✅ RAG over Seismic content library (native Knowledge Base integration)
  ✅ Any use case where development speed > model performance optimization

USE SAGEMAKER JUMPSTART WHEN:
  ✅ Fine-tuned cross-sell prediction model (trained on 3 years of Nomura mandate data)
  ✅ Custom financial entity recogniser (Nomura fund names, CUSIP, ISIN, internal strategy codes)
  ✅ Proprietary research summarisation (fine-tuned on Nomura CIO commentary style and taxonomy)
  ✅ Compliance NLP (fine-tuned on FCA/SEC regulatory text for automatic compliance flagging)
  ✅ Any use case where Nomura-specific domain knowledge measurably outperforms a general model
```

### Phase 1: Bedrock Architecture (Rapid Deployment)

```
AGENTFORCE CHATBOT (Salesforce)
        ↓
Lambda: Parse RM natural language query
        ↓
Amazon Bedrock (Claude 3.5 Sonnet — Anthropic)
  Prompt: "You are a Nomura asset management assistant.
          Parse the intent from this RM query and return
          a structured JSON with: {intent, asset_class,
          content_type, audience, jurisdiction, recency}.
          Query: {{rm_query}}
          Account context: {{salesforce_account_metadata}}"
        ↓
Structured JSON intent → DAM API call
        ↓
Bedrock Knowledge Base (RAG over Seismic content index)
  → Returns: top 3 relevant assets with metadata
        ↓
Claude summarizes: "Here are the 3 best assets for your meeting.
                    The Bellwether Global FI Overview was updated
                    last Thursday. The UK Pension Fund deck was
                    last sent to this client 47 days ago."
        ↓
Agentforce renders response + one-click Salesforce actions
```

**Bedrock Guardrails configuration for financial services:**
```
Content filtering: Block toxicity, hate speech, self-harm
PII redaction: Mask client names and account numbers in chatbot responses
Groundedness: Require all factual claims to be grounded in retrieved sources
Denied topics: Block competitor fund recommendations, specific investment advice
Word filters: Block regulatory red-flag language (guarantees, assured returns)
```

### Phase 2: SageMaker JumpStart for Cross-Sell Intelligence

```
TRAINING DATA (Nomura proprietary — never leaves VPC):
  3 years of mandate history (won + lost)
  Salesforce Account metadata (AUM tier, strategy exposure, region)
  Seismic engagement data (content interactions preceding mandate wins)
  NCIE feedback themes (client friction → churn signal)
        ↓
SageMaker JumpStart: Fine-tune Llama 3.1 8B on mandate prediction task
  Training script: SageMaker Training Job (ml.p3.2xlarge — $3.06/hr)
  Validation: SageMaker Clarify — bias check by client segment
  Output: Custom endpoint: "cross-sell-predictor-v1"
        ↓
Deployed to: SageMaker Real-time Endpoint (ml.g4dn.xlarge — $0.736/hr)
        ↓
Salesforce Agentforce calls endpoint via Lambda:
  Input: {account_id, current_exposure, recent_engagement, market_context}
  Output: [{strategy: "Private Credit", probability: 0.87, evidence: "Competitor filings show alternatives expansion"},
           {strategy: "EM Equity", probability: 0.62, evidence: "3× accessed EM content in 30 days"}]
        ↓
Salesforce: Creates "Suggested Opportunities" with evidence rationale
```

### Integration: Using Both Together

```
ARCHITECTURE PATTERN: Bedrock handles conversation; JumpStart powers prediction

Agentforce conversation → Bedrock (Claude): "What should I discuss with this client?"
                                    ↓
Bedrock calls SageMaker endpoint as Tool:
  cross_sell_prediction(account_id) → [{strategy, probability, evidence}]
                                    ↓
Bedrock composes response: "Based on [client]'s current 8% FI exposure vs. their
  15% benchmark target, and their engagement with our FI outlook commentary last month,
  I recommend leading with Global Fixed Income in tomorrow's call.
  The latest UK Institutional pitch deck [LINK] was compliance-approved on Feb 18."
                                    ↓
RM has AI reasoning + content + action — in one Salesforce interaction
```

---

## 4. Salesforce Agentforce: Three Core Use Cases

### Use Case 1 — Content Retrieval (DAM Query at Point of Need)

**Trigger:** RM opens Account record 45 minutes before a client meeting

**RM query:** *"I'm preparing for a meeting with a UK pension fund considering our Global Fixed Income strategy. What materials should I bring?"*

```
Agentforce Intent Parsing (Bedrock):
  audience:       institutional
  jurisdiction:   UK
  strategy:       Global Fixed Income
  meeting_stage:  pre-meeting (inferred from "preparing for meeting")

DAM API Query executed:
  filter: jurisdiction=UK AND strategy=GlobalFixedIncome
          AND content_type IN (strategy_overview, factsheet, commentary)
          AND compliance_status=APPROVED
          AND version=CURRENT

Results returned:
  1. "Nomura Global Fixed Income Strategy Overview — Q1 2026" [PDF, 12 pages]
     → Last updated: Feb 14, 2026
     → Previously sent to this account: never
     → Compliance expiry: Apr 30, 2026 ✅

  2. "GIPS Performance Record — Global Fixed Income — 5Y" [PDF, 4 pages]
     → Last updated: Jan 31, 2026
     → Required for UK institutional due diligence
     → FCA-compliant disclosure format ✅

  3. "Nomura CIO Commentary — February 2026 — Duration & Yield Curve" [PDF, 6 pages]
     → Last updated: Feb 20, 2026
     → Highly engaged: 23 UK institutional clients opened this month ✅

Activity auto-logged to Salesforce:
  Activity Type: "Content Retrieval — Agentforce"
  Assets retrieved: [3 assets listed above]
  Linked to: Account XYZ, Contact [CIO name], Opportunity [Fund XVII consideration]
  Timestamp: Feb 24, 2026 09:47 UTC
```

**Business value:** RM walks into the meeting with 3 current, jurisdiction-correct, compliance-approved assets in 8 seconds vs. 15–23 minutes. Activity logged automatically — zero manual CRM entry.

---

### Use Case 2 — RFP / DDQ Instant Response at Point of Need

**Trigger:** Prospect sends DDQ question via email; RM is in Salesforce composing reply

**RM query:** *"The prospect just asked about our investment risk management framework in their DDQ. What's our approved response?"*

```
Agentforce routes to Knowledge Base (Bedrock RAG):
  Query embedded → cosine similarity search over DDQ knowledge base
  Knowledge base contains: 847 approved DDQ/RFP answer chunks
                           Last ingested: Feb 23, 2026

Top-ranked response returned:
  Topic: "Investment Risk Management Framework"
  Approved response: [full answer text]
  Confidence: 0.91 (above 0.75 threshold — no human escalation required)
  Compliance metadata: Approved by [legal name], Feb 10, 2026
  Version: 4.2 (current)

Agentforce output to RM:
  "Here is the current approved DDQ response for 'Investment Risk Management Framework'.
   Confidence: HIGH (0.91). Last approved: Feb 10, 2026.
   ⚠️ This is an AI-assisted response. Review before sending.
   [Copy to email] [Log as DDQ Activity] [Escalate to SME]"

If confidence < 0.75:
  "I have a partial match but my confidence is LOW (0.68).
   Routing to [Compliance SME — Jane Doe] for human review before you use this.
   Expected response: 4 hours."
```

**Audit trail (FINRA requirement):**
```
Every DDQ response generated is logged with:
  - Timestamp
  - RM identity (Salesforce User_ID)
  - Knowledge base version used (v4.2 — immutable S3 object)
  - Confidence score
  - Whether human review was triggered (yes/no)
  - Whether RM modified the response before sending (diff tracked)
```

---

### Use Case 3 — Content Freshness Alert (Proactive Engagement Trigger)

**Trigger:** DAM detects a new approved version of an asset previously sent to a client

**Agentforce (proactive notification to RM):**
```
"The fund fact sheet for Nomura Global High Yield that you sent to [Pension Fund X]
 on Feb 10 has been updated. The performance data now reflects February month-end.
 Sending the updated version now shows you are active and on top of market data.

 [Send Updated Version]  [Dismiss]  [Schedule for Monday]"
```

**One-click re-send flow:**
```
RM clicks [Send Updated Version]
  ↓
Salesforce email drafted with:
  - Updated factsheet attached (signed S3 URL — expires in 72 hours)
  - Personalised subject: "Updated: Nomura Global High Yield — February 2026"
  - CRM-tracked send (Seismic LiveSend link embedded)
  ↓
Activity logged: Content update notification sent
  ↓
Seismic: engagement tracking activated
  → If opened within 5 days → Einstein alert: "Engagement signal — suggest call"
  → If not opened in 7 days → Agentforce: "No engagement. Try a different format?"
```

**Strategic value:** Converts a passive "content library" into an active, proactive client engagement engine. The RM is perceived as attentive and commercially sharp — without doing anything beyond one click.

---

## 5. Cross-Selling Intelligence Engine

### Architecture: From Client Signal to Salesforce Opportunity

```
SIGNAL SOURCES (Four inputs into the Cross-Sell Engine)

1. SEISMIC ENGAGEMENT DATA
   Client engagement:  PDF opened × 4, pages 3-5 dwell = 4m 30s (Private Credit overview)
   Signal weight:      HIGH INTENT on Private Credit content
        ↓ EventBridge pipe

2. NCIE FEEDBACK SIGNALS (via NOMURA_CLIENT_INSIGHT_ENGINE.md)
   NCIE action item:  "Client expressed interest in alternatives allocation in portal chat"
   Bedrock entity:    "ASSET_CLASS: Private Credit" → HIGH INTENT classification
        ↓ Lambda → Salesforce

3. PORTFOLIO ANALYTICS (PowerBI → Salesforce bidirectional)
   Current exposure:  Private Credit = 0% (peer median: 8%, benchmark: 5%)
   Signal:            Significant under-allocation vs. benchmark
        ↓ PowerBI → Salesforce via AppFlow

4. SAGEMAKER CROSS-SELL MODEL (JumpStart fine-tuned)
   Model input:       {account_id, current_exposure, engagement_signals, ncie_signals}
   Model output:      Private Credit cross-sell probability: 0.89
   Evidence:          "3 engagement signals + 0% current allocation + positive sentiment in feedback"
        ↓ SageMaker endpoint → Lambda → Salesforce

CONVERGENCE IN SALESFORCE SALES CLOUD:
   Three signals fire within same 30-day window
        ↓
   Einstein Opportunity Score: VERY HIGH
        ↓
   Agentforce generates Suggested Task for RM:
   "Cross-sell Opportunity: Private Credit
    Based on [client]'s engagement with Private Credit content (×4 opens),
    under-allocation vs. benchmark, and positive portal feedback, this account
    shows strong intent signals for a Private Credit allocation conversation.

    Recommended action: Schedule 30-min strategy call. Lead with Q1 2026 Private
    Credit Market Outlook (attached — compliance-approved Feb 15).

    [Create Opportunity]  [Schedule Call]  [Send Overview Doc]  [Dismiss]"
```

### Cross-Sell Signal Taxonomy

| Signal Type | Source | Bedrock/JumpStart Role | Salesforce Action |
|---|---|---|---|
| **Content engagement (>3× opens)** | Seismic LiveSend | Bedrock classifies intent from content metadata | Einstein activity score update |
| **Portal feedback keyword** | NCIE + Comprehend | Comprehend entity → Bedrock classifies strategy intent | Suggested Task created |
| **Under-allocation vs. benchmark** | PowerBI + Athena Gold Layer | SageMaker model: probability from portfolio delta | Opportunity recommended |
| **Competitor filing (public data)** | Glue scraper → Comprehend | Comprehend NER: competitor + fund strategy extracted | Competitive alert in account |
| **Meeting mention (AI transcription)** | Agent transcript + Glue | Bedrock: meeting transcript → intent extraction | Call summary + follow-up task |
| **Mandate renewal approaching** | Aurora (contract data) | Rule-based trigger (no AI needed) | Renewal task + content package |

### Cross-Sell ROI Model

| Metric | Baseline | With Cross-Sell Engine | Source |
|---|---|---|---|
| **Cross-sell opportunities identified per RM/quarter** | 3 (manual observation) | 14 (AI-surfaced signals) | 4.7× improvement |
| **Opportunity-to-meeting conversion rate** | 22% (cold outreach) | 41% (evidence-driven outreach) | Salesforce benchmark for high-intent engagement |
| **Average mandate size at Nomura** | $180M | $180M | Nomura AM APAC distribution profile |
| **Cycle 1 revenue impact** | 3 × 22% × $180M = $118.8M pipeline | 14 × 41% × $180M = $1.03B pipeline | **8.7× improvement in qualified pipeline** |
| **Time to identify signal** | 2–4 weeks (review cycle) | <24 hours (EventBridge real-time) | Architecture-driven improvement |

---

## 6. DAM Platform Evaluation for Nomura

### Platform Comparison Matrix

| DAM Platform | Salesforce Integration | AI Tagging | Compliance Workflow | Best Match For | Nomura Fit |
|---|---|---|---|---|---|
| **Seismic (extended)** | Native FSC — existing deployment; LiveSend analytics; Agentforce action in preview | AI content recommendation; intent-based surfacing | Sales-layer approval workflow | Distribution front-line — RMs and wholesalers | **PRIMARY** — extend upstream to compliance review layer |
| **Aprimo** | Native connector; AI-driven tagging; predictive search | Full content lifecycle AI from strategy through compliance | Robust multi-stage compliance with version lock | Enterprise AM with high content volume + complex regulatory requirements | **SECONDARY** — best for production workflow tier if Seismic upstream gaps cannot be closed |
| **Bynder** | Salesforce Marketing Cloud direct + CDP | AI-powered brand DAM; creative team workflow focus | Marketing approval; lighter than Aprimo | Wealth/retail distribution with brand and campaign focus | Wealth channel only |
| **OpenText OTMM** | Salesforce MC connector; browse in-platform | Large-scale media management | Institutional compliance-grade | Institutional/compliance-heavy with large media library | Institutional channel only |
| **Acquia DAM (Widen)** | Custom Salesforce connector | Strong metadata management | Mid-market standard | Mid-size AM expanding from shared drives | Not recommended at Nomura scale |

### Nomura Recommended Architecture: Seismic as the Primary DAM

```
RECOMMENDED ARCHITECTURE: Extend Seismic upstream

CURRENT STATE:
  Portfolio team → email to compliance → compliance approves → manual Seismic upload
  Problems: version drift, manual steps, 3–5 day cycle, no API trail

FUTURE STATE (Seismic + Salesforce Flow):
  Portfolio team → Salesforce Flow (automated compliance workflow)
        ↓
  Step 1: Draft asset uploaded → compliance team notified (Salesforce Case)
        ↓
  Step 2: Legal review (Salesforce Task with SLA = 48h for standard content)
        ↓
  Step 3: Compliance approval → Seismic Asset API: auto-publish with metadata
          {asset_class, audience, jurisdiction, version, expiry_date, approved_by}
        ↓
  Step 4: Bedrock Knowledge Base ingestion triggered (Lambda)
          → New asset indexed for Agentforce RAG retrieval
        ↓
  Step 5: RM notification: "New asset available — [Fund Name] Q1 Commentary"
          → Available immediately in Agentforce chatbot

CYCLE TIME: Manual 3–5 days → Automated 4–8 hours for standard content
COMPLIANCE AUDIT TRAIL: Every step logged in Salesforce with timestamp + approval identity
GAP CLOSED: Production → compliance review → Seismic governs in a single platform
```

---

## 7. Salesforce Sales Cloud as Orchestration Layer

### Why FSC Context Makes Agentforce 10× More Valuable

Without Salesforce Financial Services Cloud context, the Agentforce chatbot serves generic content. With FSC context, it serves personalised, situationally relevant content because it has access to:

| FSC Data Object | Data Available | How Agentforce Uses It |
|---|---|---|
| **Account** | Client type, AUM tier, jurisdiction, allocated strategies | Filters DAM content by audience type + jurisdiction automatically |
| **Opportunity** | Stage, strategy of interest, decision timeline | Returns stage-appropriate content (awareness vs. due diligence vs. close) |
| **Activity / Task** | Past meetings, emails, content send history, engagement | Prevents re-sending; identifies content gaps; detects follow-up signals |
| **Campaign Member** | Email engagement (opens, clicks, downloads) | Informs Seismic recommendation engine of content-to-engagement correlation |
| **Case (Service Cloud)** | Client service inquiries, NAV queries, report issues | Surfaces technical fund documents, NAV calculation breakdowns instantly |
| **Contact** | Role (CIO, CFO, Compliance, Ops), relationship owner | Personalises content to decision-maker role, not just account-level |

### Salesforce EventBridge Automation: Closed-Won → Full Client Setup

```
TRIGGER: Salesforce Opportunity Stage = "Closed Won" (new institutional mandate)
        ↓
Amazon EventBridge (Salesforce-to-AWS Pipe):
  Event: {opportunity_id, account_id, strategy, mandate_size, jurisdiction}
        ↓
AWS Step Functions (4-step onboarding workflow):
  Step 1: Vermilion API → provision client reporting profile
          {fund_config, GIPS_template, jurisdiction=UK, delivery=portal+email}

  Step 2: PowerBI REST API → provision client workspace
          {data_source=Athena_Gold, RLS_config=account_id, template=institutional}

  Step 3: OIDC provisioning → Salesforce Experience Cloud portal SSO
          {user_email, profile=Institutional_Client, entitlements=[fund_XVII]}

  Step 4: Seismic → provision personalised content experience
          {audience_segment=institutional, strategies=[GFI], jurisdiction=UK}
        ↓
Salesforce Account record updated:
  "Client portal provisioned. PowerBI dashboard live.
   Agentforce context loaded. Onboarding complete — 47 seconds."
        ↓
RM notification + welcome pack automatically sent
```

### Sales Cloud Intelligence Layer: Einstein Features at Nomura

| Einstein Feature | Configuration | Nomura Use Case |
|---|---|---|
| **Einstein Relationship Insights** | Ingest consulting firm networks, institutional contacts, LinkedIn data | Map CIO's relationships to Nomura PMs; surface warm introductions for new mandates |
| **Einstein Activity Capture** | Auto-log Outlook + calendar to Salesforce account records | Eliminate manual activity entry; ensure complete client history for AI pattern detection |
| **Einstein Opportunity Scoring** | Trained on Nomura's 3-year mandate win/loss data via JumpStart | Score every prospect account: "72% probability of mandate award in next 90 days" |
| **Einstein Lead Scoring** | Inbound enquiries from portal + events + ADX | Route high-score leads to senior RMs; auto-nurture low-score leads with Agentforce content |
| **Einstein Forecasting** | Salesforce Opportunity pipeline + cross-sell signals | Quarterly AUM guidance accurate to ±5%; replaces manager gut-feel with model confidence |

---

## 8. Implementation Roadmap

### Five-Phase Delivery Plan

| Phase | Timeline | Scope | Success Metric | Risk Mitigation |
|---|---|---|---|---|
| **Phase 1: Content Foundation** | Q1 2026 (Weeks 1–6) | Audit Seismic library: identify content with missing metadata, expired assets, missing jurisdiction tags. Export full DAM inventory. | 100% of active assets tagged with: asset_class, audience, jurisdiction, compliance_status, expiry_date | Brownfield analysis first — do not ask RMs to change behaviour until the content is clean |
| **Phase 2: Compliance Workflow** | Q2 2026 (Weeks 7–16) | Build Portfolio team → compliance review → Seismic publish workflow using Salesforce Flow + Seismic Asset API | 48-hour content delivery cycle for standard content types; zero manual uploads after go-live | Parallel run: old manual process + new automated process for 4 weeks before cutover |
| **Phase 3: Agentforce MVP** | Q2–Q3 2026 (Weeks 14–22) | Deploy Agentforce chatbot with Seismic DAM API; Bedrock Claude intent parser; 3 use cases (content retrieval, DDQ response, freshness alert) | RM content retrieval time <60 seconds; DDQ response accuracy >85% (spot-checked by compliance) | Pilot with 5 RMs before full rollout; measure adoption and accuracy before scaling |
| **Phase 4: Cross-Sell Intelligence** | Q3–Q4 2026 (Weeks 22–34) | Fine-tune SageMaker JumpStart cross-sell model on Nomura data; EventBridge signal pipeline; Salesforce Suggested Opportunity integration | Cross-sell opportunities per RM/quarter: 3 → 10+ | Model validation: test on historical data (backtest win/loss); require 0.70+ F1 before production |
| **Phase 5: Full Personalisation** | Q4 2026–Q1 2027 (Weeks 34–52) | FSC account/opportunity data → Seismic LiveContent auto-populated pitch decks; full analytics loop (engagement → Salesforce → NCIE → backlog) | Auto-generated personalised decks for 80% of meetings; RM prep time <5 minutes | Build with template library first; allow RM override for 20% manual customisation cases |

---

## 9. Proactive Risk Mitigation: The Lemons Tables

### AI Model Risk

| Risk (Lemon) | Mitigation (Squeezer) | Value Delivered |
|---|---|---|
| **Bedrock hallucination in DDQ response** | Confidence threshold gate (≥0.75 required); all claims grounded in Knowledge Base; human escalation path for low confidence | Compliance team trusts AI-assisted responses; zero rogue content reaches clients |
| **SageMaker cross-sell model drift** | SageMaker Model Monitor: data drift + prediction quality alerts; quarterly retraining with new mandate data | Model stays relevant as Nomura's product mix and client base evolves |
| **Bedrock vs. JumpStart cost overrun** | Bedrock: per-token billing — auto-scales to zero when no queries; JumpStart: auto-scale policy on endpoint (min 0 instances overnight) | AI infrastructure cost bounded; fully elastic; "pay for what you use" model |
| **LLM biased cross-sell recommendations** | SageMaker Clarify: bias detection by client segment before production; ongoing monitoring | FCA compliance: discriminatory product recommendation cannot result from AI-driven distribution |
| **Prompt injection via RM query** | Agentforce action schema validation; Bedrock Guardrails deny-list of injection patterns; API input sanitization | Enterprise security maintained; malicious actors cannot use chatbot to extract confidential data |

### CRM & Integration Risk

| Risk (Lemon) | Mitigation (Squeezer) | Value Delivered |
|---|---|---|
| **Salesforce EventBridge pipe failure** | Step Functions idempotency key = Opportunity_ID; retry with exponential backoff × 3; alert Slack #platform-ops | Closed-Won provisioning runs exactly once; no double-provisioning of client portals |
| **Seismic API rate limiting** | Agentforce queues DAM requests via Lambda SQS FIFO; 100 requests/minute max; backoff on 429 | Content retrieval degrades gracefully; RM receives "Searching... expected in 10 seconds" not an error |
| **Activity Capture missing Outlook data** | Exclude list for "Confidential" and "Privileged" email subjects; legal hold compliance maintained | Complete activity history without exposing sensitive communications |
| **Zero-copy connector data drift** | Verification Lambda runs nightly: Salesforce Data Cloud NAV vs. Athena Gold Layer value — alert if delta >$0.01 | Single source of truth guarantee enforced programmatically, not by policy |

### DAM Governance Risk

| Risk (Lemon) | Mitigation (Squeezer) | Value Delivered |
|---|---|---|
| **Outdated content surfaced by Agentforce** | DAM API query always includes `compliance_status=APPROVED AND expiry_date > TODAY()` — expired content is invisible to Agentforce | Architecture makes the compliant path the easy path — outdated content cannot reach distribution |
| **Jurisdiction mismatch (UK content to APAC client)** | Agentforce intent parser extracts `jurisdiction` from client Account record; DAM query enforces jurisdiction filter | Regulatory compliance is automated — RM cannot accidentally send UK-only disclosure to a US client |
| **Seismic upstream workflow bottleneck** | SLA monitoring via Salesforce Case aging report: flag any compliance review Case open >48h to team manager | Compliance team accountability; 48-hour SLA enforced by platform, not by email chase |

---

## 10. Interview Strategy: Talking Points for Kathleen, Stuart, and Shaw

### For Kathleen — Client Experience + AI Digital Agent

**The Hook:**
> *"Kathleen, the DDQ use case you're solving with the AI Digital Agent and the chatbot use case for RM content retrieval are the same architecture. Both use Bedrock for natural language understanding, both use a Knowledge Base for grounded responses, and both require the same compliance controls — confidence thresholds, audit logging, and human escalation paths. Building one well means you build both well."*

**The Content Freshness Angle:**
> *"One of the highest-impact but least-discussed client experience wins is content freshness. When a client's RM sends them an outdated factsheet — even inadvertently — it signals that the firm is not on top of their account. With the proactive freshness alert use case, the moment the compliance team approves an updated version of a document previously sent to a client, the RM gets a one-click re-send option. It is a tiny action that communicates 'we are watching your account.' Those signals drive retention — and NCIE captures the positive sentiment in the feedback data to prove it."*

**The Clincher:**
> *"The AI agent is the front-of-house; the chatbot is the back-of-house for RMs. The client-facing agent and the RM-facing chatbot are both consuming the same Knowledge Base, the same Bedrock model, and the same compliance guardrails. The investment is made once. The returns compound: better RM preparation → better client conversations → higher NPS → more NCIE feedback signal → the agent improves. It is one platform, two surfaces, one feedback loop."*

---

### For Stuart — Client Platforms + Salesforce

**The Hook:**
> *"Stuart, Seismic is already deployed at Nomura. The gap is that it is a library RMs have to remember to visit — it is pull, not push. Agentforce makes it ambient: the right content surfaces automatically based on what the RM is doing right now in Salesforce. That doesn't require a new platform. It requires wiring together what we already have."*

**The Architecture Narrative:**
> *"The two technical steps are: first, improve the upstream content pipeline — build a Salesforce Flow that automates the portfolio team → compliance → Seismic publish workflow. Today, that is entirely manual and takes 3–5 days. With a configured Flow + Seismic Asset API, it is 4–8 hours and fully auditable. Second, connect Seismic to Agentforce via the Seismic Content API. When an RM asks for content, Agentforce calls the API with a structured query from Bedrock — jurisdiction, asset class, audience, current version only. The RM gets the answer in seconds. Seismic logs the retrieval. Salesforce logs the activity automatically. Zero manual CRM entry."*

**The SageMaker Cross-Sell Clincher:**
> *"The platform flywheel is: Agentforce content delivery teaches the system what content works. Seismic LiveSend tells us what clients engage with. NCIE tells us what clients say about the experience. Salesforce tells us which accounts converted. We feed all four signals into a SageMaker JumpStart cross-sell model fine-tuned on Nomura's own mandate history. When the model says 'Private Credit cross-sell probability: 0.89,' it is not a guess — it is trained on three years of deals we won and lost. That is a distribution intelligence capability competitors cannot buy off the shelf."*

---

### For Shaw — Technology Partnerships + AWS

**The Hook:**
> *"Shaw, the Bedrock vs. SageMaker JumpStart decision is the core architectural question for this platform. My recommendation is a phased approach: Bedrock delivers the MVP fast with managed infrastructure and negligible overhead — exactly right for Phase 1 where we need to prove the use case. SageMaker JumpStart delivers the differentiated intelligence in Phase 2: a cross-sell model fine-tuned exclusively on Nomura's mandate data. Our competitors will implement Bedrock; Nomura's edge comes from the JumpStart model we train on data they will never have."*

**The FinOps Angle:**
> *"The cost model is important. Bedrock Phase 1: pay per token, scales to zero overnight — estimated $800–1,200/month for 30 RMs with 20 queries/day each. SageMaker JumpStart Phase 2: ml.g4dn.xlarge at $0.736/hr with auto-scale to zero off-hours — approximately $400/month for the cross-sell endpoint. Total AI infrastructure: under $2,000/month. Revenue impact at 8.7× pipeline improvement: we need to close 0.001 of one additional mandate to recover the annual infrastructure cost. The ROI conversation is trivial."*

**The Platform Integration Clincher:**
> *"The integration architecture I've described doesn't require net-new infrastructure. It uses EventBridge we already have, Step Functions we already have, Lambda, S3, Athena Gold Layer, and Salesforce + Seismic already deployed. The only net-new investments are the Bedrock API integration, the SageMaker training job (one-time + quarterly update), and the Agentforce action configuration in Salesforce. That is a defined, bounded scope — not an open-ended platform programme. I can phase-gate every investment decision with a measurable ROI checkpoint before the next phase is approved."*

---

## Executive Clincher: The Distribution Intelligence Platform

> *"What I am proposing is a shift from Salesforce-as-a-contact-book to Salesforce-as-a-Distribution-Intelligence-Platform. The AI chatbot (Agentforce + Bedrock) makes content ambient — RMs always have the right material at the right moment. The cross-sell engine (SageMaker + EventBridge + NCIE) makes opportunity identification proactive — RMs know which clients to call before the client's competitor calls them. And the analytics loop (Seismic + NCIE + Salesforce) closes the feedback cycle — what works in distribution practice is continuously reinforced, and what doesn't is eliminated. This is not three separate projects. It is one platform delivering one outcome: more AUM closed, faster, with less client friction."*

---

## Document Metadata

| Field | Value |
|---|---|
| **Prepared by** | Calvin Lee |
| **Target audience** | Kathleen Hess McNamara · Stuart Mumley · Shaw Levin |
| **Role** | ED, Head of Client Technology — Nomura Asset Management International |
| **Purpose** | Case Study 2: AI Chatbot + CRM Sales Cloud Improvement + Cross-Selling with Bedrock/JumpStart |
| **Heritage** | Section 20 of [BUSINESS_PARTNERSHIP_README.md](BUSINESS_PARTNERSHIP_README.md) — extended and enterprise-architected |
| **Related documents** | [README.md](README.md) · [NOMURA_CLIENT_INSIGHT_ENGINE.md](NOMURA_CLIENT_INSIGHT_ENGINE.md) · [DIGITAL_LAYER_SALESFORCE_INTEGRATION.md](DIGITAL_LAYER_SALESFORCE_INTEGRATION.md) · [BIAN_FRAMEWORK_ASSET_MANAGEMENT.md](BIAN_FRAMEWORK_ASSET_MANAGEMENT.md) |
| **Last updated** | February 2026 |

---

*This document demonstrates that the AI chatbot strategy is not a standalone proof-of-concept — it is the conversational access layer for the full distribution intelligence platform, grounded in production AWS architecture, designed for enterprise compliance, and measured by AUM outcomes.*
