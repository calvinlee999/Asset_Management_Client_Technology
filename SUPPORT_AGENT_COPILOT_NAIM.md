# Nomura Agent Intelligence Mesh (NAIM) — Support Agent Co-Pilot

## Salesforce Service Cloud · AWS Bedrock · SageMaker JumpStart · OpenSearch Serverless · Amazon Comprehend

### Case Study 3 — Executive Director, Head of Client Technology

#### Target Audience: Kathleen Hess McNamara (ED, Client Digital Solutions) · Stuart Mumley (Director, Client Platforms) · Shaw Levin (VP, Technology Partnerships)

> **Strategic Purpose:** This case study articulates the architecture and business rationale for
> deploying the **Nomura Agent Intelligence Mesh (NAIM)** — a Salesforce Service Cloud Co-Pilot
> powered by Amazon Bedrock and SageMaker JumpStart that transforms internal support agents
> from manual ticket-hunters into AI-augmented resolvers. NAIM uses Retrieval-Augmented Generation
> (RAG) over two Single Sources of Truth — the Nomura client-facing application manuals and five
> years of solved support tickets — to deliver instant troubleshooting steps, historical pattern
> recognition, and sentiment-driven triage inside the agent's existing Salesforce workflow.
> The architecture is deliberately complementary to the external AI Digital Agent (Case Study 2):
> **the same Bedrock Knowledge Base, the same OpenSearch Serverless vector store, the same
> compliance guardrails — one infrastructure investment delivering both external client deflection
> and internal agent augmentation.** KPI targets: 60% reduction in MTTR, 40% increase in FCR.

Integration heritage: Built on the AI + CRM architecture established in
[AI_CHATBOT_CRM_SALESCLOUD_INTEGRATION.md](./AI_CHATBOT_CRM_SALESCLOUD_INTEGRATION.md),
the feedback orchestration platform in
[NOMURA_CLIENT_INSIGHT_ENGINE.md](./NOMURA_CLIENT_INSIGHT_ENGINE.md), and the Salesforce
integration blueprint in
[DIGITAL_LAYER_SALESFORCE_INTEGRATION.md](./DIGITAL_LAYER_SALESFORCE_INTEGRATION.md).
NAIM closes the internal loop: when institutional clients or wealth advisors experience
friction in the Nomura application, they experience **"Uninterrupted Journeys"** — not
hold music and Confluence searches.

---

## Table of Contents

1. The Problem: Support Friction and the Cost of Unresolved Client Journeys
2. The NAIM Architecture: Three-Layer Integration Blueprint
3. Three Priority Use Cases: The Co-Pilot in Action
4. AI Engine Design: Bedrock + SageMaker JumpStart Integration Pattern
5. Knowledge Base Construction: The Two SSOTs
6. The LWC Co-Pilot Interface: Embedded Inside Service Cloud
7. EventBridge Automation: Proactive Intelligence Pipeline
8. Implementation Roadmap
9. Proactive Risk Mitigation: The Lemons Tables
10. Interview Strategy: Talking Points for Kathleen, Stuart, and Shaw

---

## 1. The Problem: Support Friction and the Cost of Unresolved Client Journeys

### The Current Support Workflow — Five Failure Points

When an institutional client or wealth advisor encounters a technical issue in the Nomura
client-facing application — portal access, entitlement errors, reporting dashboard anomalies,
data latency in the analytics module — they escalate to the Nomura support team via phone,
email, or case portal. The agent's workflow unfolds like this:

| Step | Action | Time Cost | Failure Mode |
|------|--------|-----------|--------------|
| 1 | Open Salesforce Service Cloud Case — no application context auto-populated | 3–5 min | Agent manually types issue description; no structured triage |
| 2 | Search legacy Confluence wiki for troubleshooting steps | 8–15 min | Wrong search terms; outdated articles; dead links to retired UI |
| 3 | Check whether the guide found reflects the current application version | 3–5 min | App UI changes weekly; Confluence not tied to release schedule |
| 4 | Attempt resolution; fail; escalate to Tier-2 engineering team | 24–48 h wait | Tier-2 queue is the bottleneck for all Tier-1 failures |
| 5 | Case resolved; resolution steps not captured anywhere structured | 0 min | Institutional knowledge leaves with the agent; next agent repeats |

**Total MTTR (Mean Time to Resolution): 4.2 hours average for Tier-1 technical issues.**
Every minute on hold is a minute a CIO is not managing portfolios, a wealth advisor is not
serving a client, and a Tier-1 AUM relationship is at risk.

### KPI Gap — The Business Case for NAIM

| KPI | Current State | NAIM Target | Improvement |
|-----|--------------|-------------|-------------|
| Mean Time to Resolution (MTTR) | 4.2 hours | ≤1.7 hours | **60% reduction** |
| First Contact Resolution (FCR) | 42% | ≥59% | **40% increase** |
| Tier-1 Escalation Rate | 61% of cases reach Tier-2 | ≤25% | **59% reduction in escalations** |
| Agent Onboarding Time to Productivity | 6–8 weeks (Confluence learning curve) | ≤2 weeks (NAIM-guided) | **65% faster ramp** |
| Client Wait Time (Tier-1 Hedge Fund SLA) | 4+ hours first response | <15 minutes | **SLA adherence for top-tier clients** |

### The Institutional Client Impact

A Tier-1 Hedge Fund client managing $2.3B through Nomura cannot access the ESG
Securitization tranche in their analytics dashboard. Their prime brokerage operations
team raises a ticket at 08:17 EST — 43 minutes before a risk committee meeting. The agent
spends 22 minutes in Confluence before finding a guide last updated in September 2024. The
UI has changed since. The agent escalates. The client calls the RM directly.

**That is not a support failure — it is a relationship failure. And it costs AUM.**

### The Confluence Problem — Why Traditional Knowledge Management Fails

```
WHAT EXISTS TODAY:
  Confluence wiki         → 847 articles, updated manually by engineers
                            → 34% out of date (last audit: Q3 2025)
  Support email threads   → Previous resolutions buried in inboxes
  Agent tribal knowledge  → Lives in individual agents' heads; exits with turnover
  JIRA ticket history     → Structured but not searchable in natural language

WHAT SHOULD EXIST:
  Agent opens Service Cloud Case for: "ESG Securitization tranche not visible"
  NAIM Co-Pilot (LWC panel):
    "I found 3 relevant resolution paths for this issue.
     Most likely cause: OIDC entitlement scope missing 'ESG_TRANCHE' permission.
     Resolution steps (Source: Nomura App Guide v5.1, Section 12.3):
      1. Click 'Advanced Filters' → 'Asset Class' → 'ESG'
      2. If not visible: check client's OIDC entitlement in the Identity Provider
      3. If entitlement present but filter missing: clear browser cache / CORS token refresh
     Similar case resolved Aug 2025: [Case #47221] — OIDC scope provisioning gap
     [Insert into Case Notes]  [Draft Client Email]  [Escalate with Context]"
  Time to resolution: 45 seconds vs. 22 wasted minutes.
```

---

## 2. The NAIM Architecture: Three-Layer Integration Blueprint

### Full System Architecture

```
╔══════════════════════════════════════════════════════════════════════════════╗
║  LAYER 1: ENGAGEMENT LAYER  (Salesforce Service Cloud — Agent-Facing)       ║
║                                                                              ║
║  Support Agent working within Service Cloud Case Console                    ║
║  ┌──────────────────────────────────────────────────────────────────────┐   ║
║  │  Lightning Web Component (LWC) — NAIM Co-Pilot Panel                │   ║
║  │  (Embedded in Case Console; agent never leaves Salesforce)           │   ║
║  │                                                                      │   ║
║  │  "I can't see the ESG Securitization tranche on my dashboard"       │   ║
║  │  "Data latency in reporting module — when did this last happen?"    │   ║
║  │  "Hedge Fund client is furious — what's the escalation script?"     │   ║
║  └──────────────────────────┬─────────────────────────────────────────-┘   ║
║                             │ Salesforce Platform Event → API Gateway        ║
╚═════════════════════════════╪════════════════════════════════════════════════╝
                              │
                              ▼  Amazon API Gateway + AWS Lambda (Orchestrator)
╔═════════════════════════════╪════════════════════════════════════════════════╗
║  LAYER 2: ORCHESTRATION &   │  API LAYER (AWS Serverless)                   ║
║                             ▼                                                ║
║  Lambda Routing Decision:                                                    ║
║  ┌──────────────────────────────────────────── ─────────────────────────┐   ║
║  │  intent = "troubleshoot"  →  Bedrock RAG (application manuals KB)   │   ║
║  │  intent = "find_similar"  →  SageMaker semantic clustering endpoint  │   ║
║  │  intent = "triage"        →  SageMaker NLP + Comprehend sentiment   │   ║
║  │  intent = "draft_email"   →  Bedrock generative (Claude 3.5 Sonnet) │   ║
║  └──────────────────────────────────────────────────────────────────────┘   ║
╚══════════════╤═════════════════════════════╤══════════════╤═════════════════╝
               │                             │              │
               ▼                             ▼              ▼
╔══════════════╪═════════════╗  ╔════════════╪════════╗  ╔═╪═══════════════════╗
║  BEDROCK     ▼             ║  ║ SAGEMAKER  ▼        ║  ║ COMPREHEND ▼        ║
║  (Generative Engine)       ║  ║ (Predictive Engine) ║  ║ (Sentiment Engine)  ║
║                            ║  ║                     ║  ║                     ║
║  Claude 3.5 Sonnet         ║  ║ Model 1: Semantic   ║  ║ Entity extraction   ║
║  Knowledge Base RAG:       ║  ║ ticket clustering   ║  ║ PII masking         ║
║  • App documentation       ║  ║ (Llama 3.1 8B FT)  ║  ║ Frustration signal  ║
║  • Solved case summaries   ║  ║                     ║  ║ extraction          ║
║  Bedrock Guardrails:       ║  ║ Model 2: Sentiment  ║  ║ before KB ingest    ║
║  • Source citation forced  ║  ║ NLP + triage score  ║  ║                     ║
║  • Hallucination controls  ║  ║ (XGBoost + Llama)  ║  ║                     ║
║  • PII denial at output    ║  ║                     ║  ║                     ║
╚══════════════╤═════════════╝  ╚════════════╤════════╝  ╚═════════════════════╝
               │                             │
               ▼                             ▼
╔══════════════════════════════════════════════════════════════════════════════╗
║  LAYER 3: AI & MEMORY MESH  (Knowledge + Data Stores)                       ║
║                                                                              ║
║  Amazon OpenSearch Serverless (Vector Database)                              ║
║  ┌──────────────────────────────────────────────────────────────────────┐   ║
║  │  SSOT 1: Application Documentation Index                            │   ║
║  │  Source: PDF/Markdown — Nomura client portal guide, API docs,       │   ║
║  │          entitlement model documentation, release notes             │   ║
║  │  CI/CD: GitLab webhook → Lambda → Bedrock re-ingest on release      │   ║
║  │                                                                      │   ║
║  │  SSOT 2: Historical Ticket Index                                     │   ║
║  │  Source: Solved Salesforce Cases → Zero-Copy to S3 (Data Cloud)    │   ║
║  │          → Comprehend PII masking → Bedrock embedding → index       │   ║
║  │  Nightly: new resolved cases appended; knowledge base grows richer  │   ║
║  └──────────────────────────────────────────────────────────────────────┘   ║
║                                                                              ║
║  Amazon Aurora (Structured Operational Data)   Amazon S3 (Raw Data Lake)    ║
║  ┌────────────────────────────┐               ┌──────────────────────────┐  ║
║  │ Ticket metadata            │               │ Ingested PDF documents   │  ║
║  │ Resolution categories      │               │ PII-masked case exports  │  ║
║  │ Agent performance KPIs     │               │ Model training datasets  │  ║
║  │ SLA tracking data          │               │ Release notes archive    │  ║
║  └────────────────────────────┘               └──────────────────────────┘  ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

### The Reference Architecture Mapping (From AWS Diagram)

The AWS reference architecture provided (Week/Nightly EventBridge automation with
Bedrock + Comprehend + OpenSearch) maps directly to NAIM:

| Architecture Component | Reference Architecture | NAIM Application |
|------------------------|----------------------|------------------|
| `/summaries` API | Read & Save Lambda | Application doc ingestion + summary generation |
| `/comments` API | batch-create Lambda | Historical ticket ingestion (batch nightly) |
| `/logging` API | logging Lambda | Every Co-Pilot interaction logged to Case Feed |
| API Gateway + authorizer | Request layer | Salesforce Platform Event → API Gateway with JWT token |
| Bedrock (prompt Lambda) | LLM generation layer | Troubleshooting step generation + email drafts |
| Comprehend (summary Lambda) | Sentiment analysis | Client frustration scoring + PII redaction |
| OpenSearch (fletcher-db) | Vector search database | App manuals + ticket corpus similarity search |
| Weekly EventBridge | Automation layer | Trigger batch re-vectorization of updated app docs |
| Nightly EventBridge (nightly-job) | Automation layer | Score new day's tickets; append resolved cases to KB |
| comment-catalog → S3 | Data persistence | PII-masked ticket catalog → Zero-Copy to Bedrock KB |

---

## 3. Three Priority Use Cases: The Co-Pilot in Action

### Use Case 1 — Context-Aware Troubleshooting (The "How-To" Engine)

**Trigger:** Support agent receives a new Service Cloud Case:
`"I can't see the new ESG Securitization tranche on my dashboard"`

**Salesforce Case metadata auto-populated:**
```
Client:       [Institutional Hedge Fund X — Tier 1, $2.3B AUM]
Affected app: Nomura Analytics Dashboard v5.1
Module:       Portfolio Holdings → Asset Class Filter
Priority:     HIGH (auto-set by Einstein Case Classification)
```

**NAIM Co-Pilot action sequence:**
```
1. LWC fires Platform Event to API Gateway:
   {intent: "troubleshoot",
    case_description: "ESG Securitization tranche not visible on dashboard",
    affected_module: "portfolio_holdings_filter",
    app_version: "v5.1",
    client_tier: "TIER_1"}

2. Lambda routes to: Bedrock RAG (Knowledge Base: App Documentation SSOT)

3. Bedrock Knowledge Base query:
   Semantic search over Nomura App Guide v5.1 + entitlement model docs
   Top retrieved chunks (cosine similarity > 0.82):
   • "Advanced Filters — ESG Asset Class" — App Guide v5.1, Section 8.4, Page 47
   • "OIDC scope requirements for ESG_TRANCHE entitlement" — Identity Model v3.2
   • "Known Issue #2847: ESG filter not visible after CORS token expiry" — Release Notes Feb 2026

4. Claude 3.5 Sonnet synthesizes step-by-step resolution:
   "The most likely cause for the ESG Securitization tranche not appearing is one of
    three conditions. Resolution path:
    Step 1: Ask client to click 'Advanced Filters' → 'Asset Class' → 'ESG'
    Step 2: If filter not present: verify OIDC entitlement scope includes 'ESG_TRANCHE'
            in the client's Identity Provider provisioning record
    Step 3: If entitlement is present but filter still missing: advise client to clear
            browser cache and re-authenticate (CORS token refresh — Known Issue #2847)
    Step 4: If all steps fail: provision escalation to Tier-2 with entitlement audit log

    Source: Nomura App Guide v5.1, Section 8.4. Identity Model v3.2.
    Related: Known Issue #2847 (Feb 2026 release notes)"

5. LWC renders in Co-Pilot panel with action buttons:
   [Insert into Case Notes]  [Draft Client Email with Steps]  [Escalate with Pre-loaded Context]

6. Resolution logged to Salesforce Case Feed:
   "NAIM Co-Pilot: Troubleshooting steps surfaced — App Guide v5.1 + Identity Model v3.2
    Confidence: 0.89. Agent acknowledged steps at 08:47 EST."
```

**Business value:** Agent resolves or escalates with precise context in **under 60 seconds** vs.
22 minutes of manual Confluence search. Source citation means clients can be told exactly
which version of the guide the resolution comes from — building credibility, not eroding it.

---

### Use Case 2 — Historical Ticket Synthesis (The "We've Seen This Before" Engine)

**Trigger:** New Severity-2 Case opened: `"Data latency in reporting module — afternoon
prices not refreshing. Client's risk committee at 14:00."`

**NAIM Co-Pilot action sequence:**
```
1. LWC fires Platform Event:
   {intent: "find_similar",
    case_description: "Data latency in reporting module — afternoon prices stale",
    affected_module: "reporting_prices",
    severity: "SEV2",
    client_meeting_deadline: "14:00 EST same day"}

2. Lambda routes to: SageMaker JumpStart semantic clustering endpoint

3. SageMaker Model 1 (Llama 3.1 8B FT on Nomura IT taxonomy):
   Input: case_description embedding → cosine similarity over 5 years of resolved cases
   Output:
   {
     cluster_id: "MSK_PARTITION_LATENCY",
     cluster_confidence: 0.91,
     similar_cases: [
       {case_id: "#38421", date: "Nov 2025", root_cause: "AWS MSK partition delay — batch offset",
        resolution: "MSK ops team rebalanced partition; 45-min service restoration"},
       {case_id: "#31809", date: "Q3 2025", root_cause: "MSK consumer lag — throttling on gold layer Athena query",
        resolution: "Increased MSK throughput allocation; Athena workgroup isolated"},
       {case_id: "#27145", date: "Apr 2025", root_cause: "MSK schema registry stale topic",
        resolution: "Schema registry refresh; topic tombstone cleared"}
     ],
     estimated_resolution_time: "45-90 minutes based on prior pattern"
   }

4. Bedrock Claude drafts client-facing communication:
   Subject: "Nomura Analytics — Reporting Data Refresh — Status Update"
   Body: "We are aware of a data latency issue affecting afternoon price refreshes
          in the analytics module. Our engineering team has identified the root cause
          and is actively working on resolution. Based on prior incidents of this
          type, we estimate full restoration within 45–90 minutes. We will provide
          an update every 30 minutes. Your data as of the morning session is fully
          accurate. If your 14:00 committee requires morning prices confirmed,
          we can supply a manual extract immediately — please confirm."

5. LWC renders:
   "Historical pattern match: AWS MSK partition delay (3 prior incidents, 91% cluster confidence)
    Estimated resolution: 45–90 min. Prior resolution steps loading...
    [Send Draft Email to Client]  [Alert Engineering: MSK Incident]  [Escalate SEV2 with Context]"
```

**Business value:** Agent arrives at the root cause and client communication in **under 90 seconds**
rather than re-investigating from scratch. The client's 14:00 risk committee has a credible
status update within 3 minutes of the case opening. The RM relationship is protected.

---

### Use Case 3 — Sentiment-Driven Triage & Next Best Action

**Trigger:** New email ticket arrives from a Tier-1 Hedge Fund client:

```
Email body: "This is the THIRD time this week we cannot access our alternatives dashboard.
Our CIO is beyond frustrated. Every time we call, we get told to wait 48 hours.
We have $2.3 billion with Nomura and we can't get basic reporting to work.
This is completely unacceptable and we are reviewing our relationship with you."
```

**NAIM Co-Pilot action sequence:**
```
1. Email-to-Case auto-routing creates Salesforce Case
   Einstein Case Classification: priority = UNDEFINED (severity not self-identified by client)

2. NAIM nightly-job (or real-time trigger for Tier-1 clients) fires sentiment analysis:
   intent: "triage"

3. SageMaker Model 2 (XGBoost + Llama fine-tuned on Nomura case history):
   + Amazon Comprehend sentiment
   Input: {email_body, client_tier: "TIER_1", case_count_this_week: 3, prior_resolution_time_avg: "47 hours"}
   Output:
   {
     frustration_score: 0.94,          // 0=calm, 1=extreme
     technical_complexity: 0.71,       // 0=simple, 1=complex
     churn_risk_signal: "CRITICAL",    // based on "reviewing our relationship" + AUM tier
     triage_decision: "BYPASS_QUEUE",
     escalation_target: "HEAD_OF_CLIENT_TECHNOLOGY"
   }

4. Automated escalation sequence triggered:
   a) Case priority auto-set to CRITICAL (bypass standard queue)
   b) Salesforce notification → Qasim Ahmad team (Head of Client Technology)
      "⚠️ NAIM CRITICAL ALERT: Tier-1 Hedge Fund ($2.3B AUM). Frustration: 0.94.
       Churn Risk: CRITICAL. 3rd issue this week. Relationship review mentioned.
       Agent assigned: [Senior Agent]. Expected response SLA: 15 minutes."
   c) Bedrock drafts senior de-escalation script for assigned agent:
      "Opening acknowledgment: 'I am the senior manager on your account.
       I am personally taking ownership of this case right now.' (Do NOT delegate)
       De-escalation sequence:
       — Validate their experience without defensiveness: 'Three interruptions in a
         week is not acceptable and I want to address that directly.'
       — Short-term action: 'I am assigning you a dedicated Tier-2 engineer for the
         next 72 hours — you will have a direct line, not a queue.'
       — Systemic commitment: 'I will personally send you a root cause analysis and
         remediation plan within 24 hours.'
       Avoid: Price discussions, blame on systems, future commitments you cannot keep."
   d) NAIM generates full case history summary for senior agent brief:
      Case history, affected modules, prior resolution times, all contacts to date

5. LWC renders for senior agent:
   "CRITICAL — Tier-1 Client — AUM: $2.3B
    Frustration: ●●●●● 0.94  Churn Risk: CRITICAL
    Prior cases this week: 3  Avg wait: 47h
    [View De-escalation Script]  [Review Case History]  [Notify RM: Stuart's Account Team]"
```

**Business value:** A relationship-threatening ticket is identified as CRITICAL within
**45 seconds** of email arrival, escalated to the right team, and the senior agent enters
the first call with a briefed de-escalation script rather than no context. The difference
between a lost $2.3B relationship and a retained one is often how the first 90 seconds
of that call are handled.

---

## 4. AI Engine Design: Bedrock + SageMaker JumpStart Integration Pattern

### Decision Matrix: What Routes Where

```
ROUTE TO AMAZON BEDROCK (Claude 3.5 Sonnet) WHEN:
  ✅ Troubleshooting step generation (RAG over app documentation — requires natural language synthesis)
  ✅ Client-facing email draft (requires tone calibration, regulatory-safe language)
  ✅ De-escalation script generation (requires empathetic register, situational nuance)
  ✅ Case summary for escalation handoff (requires narrative structuring of unstructured data)
  ✅ Any task where quality of generated text is the primary output

ROUTE TO SAGEMAKER JUMPSTART WHEN:
  ✅ Semantic ticket clustering (trained on Nomura IT taxonomy — outperforms general LLM)
  ✅ Frustration / sentiment scoring (trained on Nomura case history labels — calibrated to firm's client base)
  ✅ Churn risk classification (fine-tuned on AUM tier + language patterns unique to asset management clients)
  ✅ Ticket categorization automation (Nomura-specific IT taxonomy: MSK, OIDC, CORS, ADX, reporting modules)
  ✅ Any predictive task where Nomura-specific domain training measurably outperforms a zero-shot prompt
```

### Bedrock Architecture for NAIM

```
KNOWLEDGE BASE CONSTRUCTION (Amazon Bedrock):

Source 1 — Application Documentation:
  Nomura App Guide (PDF, 287 pages, v5.1)
  OIDC / Entitlement Model documentation (PDF, v3.2)
  REST API documentation (Markdown, 156 endpoints)
  Release notes archive (Q1 2024 → Q1 2026, 8 versions)
        ↓
  S3 bucket: s3://nomura-naim-docs/app-guides/
        ↓
  Bedrock Knowledge Base ingestion:
    Chunking strategy: Semantic chunking at 1,024 tokens
                       (respects section boundaries in structured documentation)
    Embedding model:   Amazon Titan Embeddings V2
    Vector store:      Amazon OpenSearch Serverless (app_docs_index)
    Metadata fields:   {document_title, version, section, page, last_updated, doc_type}

Source 2 — Historical Ticket Corpus:
  Salesforce Cases (solved, 2021–2026): ~14,200 cases
        ↓
  Salesforce Data Cloud → Zero-Copy → S3: s3://nomura-naim-tickets/resolved/
        ↓
  Amazon Comprehend PII Masking Pipeline:
    Lambda: invoke_comprehend_pii_detection(text) → redact NAMES, ACCOUNT_NUMBERS, IBANs
    Output: PII-free case text with [REDACTED_PII] placeholders
        ↓
  Bedrock Knowledge Base ingestion (ticket_corpus_index):
    Chunking: Fixed 512 tokens (tickets are shorter, structured)
    Embedding: Titan Embeddings V2
    Metadata: {case_id, date_resolved, root_cause_category, resolution_time_hrs, affected_module}
```

### Bedrock Guardrails Configuration (NAIM-Specific)

```yaml
GuardrailConfig:
  Name: "NAIM-Support-Agent-Guardrails"
  ContentFilters:
    - Type: HATE_SPEECH         Threshold: HIGH
    - Type: INSULTS             Threshold: MEDIUM  # agents must see frustrated language for context
  PII_Redaction:
    - EntityType: NAME           Action: ANONYMIZE
    - EntityType: ACCOUNT_NUMBER Action: BLOCK
    - EntityType: CREDIT_DEBIT   Action: BLOCK
  Groundedness:
    GroundingThreshold: 0.70
    RelevanceThreshold: 0.65
    # If response cannot be grounded in retrieved KB source → flag to agent
  DeniedTopics:
    - "Investment advice or portfolio recommendations"
    - "Specific price quotations or NAV values"
    - "Competitor platform capabilities"
    - "Guaranteed service restoration timelines" # regulatory risk
  WordFilters:
    - "guaranteed"
    - "assured resolution"
    - "we promise"          # support agents must not over-commit SLAs in writing
  ContextualGrounding:
    # Force citation: every factual claim must cite KB source document + section
    CitationFormat: "Source: {document_title} {version}, Section {section}, Page {page}"
```

### SageMaker JumpStart Models for NAIM

```
MODEL 1: Semantic Ticket Clustering Engine
  Framework:        Llama 3.1 8B Instruct (SageMaker JumpStart pre-trained base)
  Fine-tuning data: 14,200 resolved Nomura cases with IT taxonomy labels
  Training job:     ml.p3.2xlarge ($3.06/hr); estimated 8 hours → one-time $24.48
  Validation:       SageMaker Clarify — bias check by client tier (no Tier-1 case
                    gets deprioritized by the model due to language style)
  Output format:    {cluster_id, confidence, similar_case_ids[], estimated_resolution_h}
  Serving:          ml.g4dn.xlarge ($0.736/hr) — auto-scale to zero outside business hours

MODEL 2: Frustration + Churn Risk Classifier
  Framework:        XGBoost (tabular features) + Comprehend (NLP signal)
  Training data:    5 years of Nomura case escalations with outcome labels
                    (escalated vs. resolved; retained vs. churned client post-incident)
  Feature set:      [frustration_score, churn_keyword_count, case_frequency_7d,
                     client_tier, aum_value, days_since_last_escalation, module_affected]
  Threshold:        CRITICAL escalation triggered at frustration_score ≥ 0.85 AND
                    churn_risk_signal in {"CRITICAL", "HIGH"} for Tier-1 clients
  Serving:          Same ml.g4dn.xlarge endpoint as Model 1 (multi-model serving)

COMBINED MONTHLY COST ESTIMATE:
  Bedrock Claude 3.5 Sonnet:   ~500 cases/day × 2,000 tokens avg = 30M tokens/month
                                → ~$900/month (input + output)
  SageMaker endpoint:          ml.g4dn.xlarge × 8h/day × 22 business days = $129/month
  OpenSearch Serverless:       2 OCUs minimum → ~$350/month
  Total AI infrastructure:     ~$1,380/month
  Break-even:                  0.001 of one additional $180M institutional mandate retained
```

---

## 5. Knowledge Base Construction: The Two SSOTs

### SSOT 1: Application Documentation — The Living Manual

The Nomura client-facing application changes with every release. The single biggest failure
mode in support knowledge management is documentation that lags the product by one version.
NAIM solves this with an **Event-Driven CI/CD Knowledge Base pipeline**:

```
DOCUMENTATION CI/CD PIPELINE:

Developer pushes new release to GitLab (Nomura internal)
        ↓
GitLab Webhook → API Gateway → Lambda: "detect_release_with_doc_changes"
        ↓
Lambda checks: has /docs/ folder changed in this commit diff?
  YES: Extract updated PDFs / Markdown → upload to S3 (s3://nomura-naim-docs/)
  NO:  Skip (no re-vectorization needed)
        ↓
Weekly EventBridge (fallback): Every Sunday 02:00 EST
  → batch-create Lambda: full inventory check — are all active doc versions indexed?
  → Any un-indexed version → trigger ingestion pipeline regardless of webhook
        ↓
Bedrock Knowledge Base ingestion job:
  Status: {new_chunks_added: 47, chunks_updated: 12, chunks_removed: 3}
  S3 notification → Slack #naim-platform-ops: "KB updated — App Guide v5.2 indexed"
        ↓
NAIM Co-Pilot routes all app documentation queries to freshest version automatically
```

**Result:** NAIM's application documentation is **always aligned with the deployed production
version** — not the version from last quarter's documentation sprint. The agent citing
"App Guide v5.2, Page 47" cites the guide the client is actually using today.

### SSOT 2: Historical Ticket Corpus — The Institutional Memory

Five years of solved Salesforce Cases contain every root cause, every resolution path,
and every edge case Nomura has successfully navigated. NAIM makes this memory accessible
in real time instead of locked in closed Case records.

```
TICKET CORPUS INGESTION PIPELINE (Nightly):

Salesforce Cases (Status = Closed, HasResolution = TRUE)
        ↓
Salesforce Data Cloud (Zero-Copy → no data movement)
  → query: new cases closed in last 24 hours with resolution documented
        ↓
S3 Export: s3://nomura-naim-tickets/raw/ (append only — audit trail preserved)
        ↓
Lambda: Comprehend PII Masking
  Input:  "John Smith at Goldman Sachs could not access fund XYZ-7842..."
  Output: "[REDACTED_NAME] at [REDACTED_COMPANY] could not access [REDACTED_FUND_ID]..."
  PII entities masked: PERSON, ORGANIZATION, ACCOUNT_NUMBER, FINANCIAL_INFO
        ↓
Masked output → S3: s3://nomura-naim-tickets/masked/ (SSOT for training + RAG)
        ↓
Bedrock Knowledge Base incremental ingestion:
  New chunks appended to ticket_corpus_index in OpenSearch Serverless
  Metadata tags: {root_cause_category, resolution_time_h, affected_module, severity}
        ↓
nightly-job completion → comment-catalog updated → Aurora metrics:
  {cases_added_today: 47, total_corpus_size: 14,392, coverage_modules: 23}
```

**Result:** Every case resolved today makes the system smarter tomorrow. NAIM's historical
intelligence **compounds** — the longer it is deployed, the more resolution patterns it
recognises, and the higher FCR climbs.

---

## 6. The LWC Co-Pilot Interface: Embedded Inside Service Cloud

### Lightning Web Component Design Principles

The NAIM Co-Pilot panel is a **Lightning Web Component (LWC) embedded directly in the
Service Cloud Case Console utility bar.** The agent never leaves Salesforce. There is no
separate tool, no new login, no context switch.

```
CASE CONSOLE LAYOUT (Two-Pane Design):

┌─────────────────────────────────────────────────────────────────────────────┐
│ SALESFORCE SERVICE CLOUD — CASE CONSOLE                                     │
├────────────────────────────────┬────────────────────────────────────────────┤
│ CASE DETAIL (LEFT PANE)        │ NAIM CO-PILOT (RIGHT PANE)                │
│                                │                                            │
│ Case #: 00089241               │ ┌────────────────────────────────────────┐ │
│ Account: Hedge Fund X ($2.3B)  │ │ 🔍 NAIM Co-Pilot — Active             │ │
│ Contact: Operations Lead       │ │────────────────────────────────────────│ │
│ Subject: ESG tranche missing   │ │ [Troubleshoot Issue]                   │ │
│ Priority: HIGH (auto-set)      │ │ [Find Similar Cases]                   │ │
│ Status: In Progress            │ │ [Assess Client Sentiment]              │ │
│                                │ │────────────────────────────────────────│ │
│ Case Description:              │ │ RESULT (Auto-triggered):               │ │
│ "I cannot see the new ESG      │ │ Confidence: 0.89 ✅                    │ │
│  Securitization tranche on     │ │                                        │ │
│  my analytics dashboard since  │ │ Step 1: Adv. Filters → ESG            │ │
│  the v5.1 update"              │ │ Step 2: Verify OIDC scope             │ │
│                                │ │ Step 3: CORS token refresh            │ │
│ Resolutions:                   │ │ Step 4: Escalate if all fail          │ │
│ [Auto-populated by NAIM ↓]     │ │                                        │ │
│                                │ │ 📄 Source: App Guide v5.1, Sec 8.4   │ │
│                                │ │ 🔗 Similar: Case #47221 (Aug 2025)   │ │
│                                │ │                                        │ │
│                                │ │ [Insert into Case Notes]              │ │
│                                │ │ [Draft Client Email]                  │ │
│                                │ │ [Escalate with Context]               │ │
│                                │ └────────────────────────────────────────┘ │
└────────────────────────────────┴────────────────────────────────────────────┘
```

### Audit Trail — Every Co-Pilot Interaction Logged

Every NAIM interaction is appended to the Salesforce Case Feed as a structured log entry:

```
Case Feed Entry (auto-generated):
───────────────────────────────────────────────────────
NAIM Co-Pilot Interaction
Time: 2026-02-24 08:47:32 UTC
Agent: [User_ID: S_00042]
Intent: troubleshoot
Query: "ESG securitization tranche not visible on dashboard"
KB Sources retrieved: ["App Guide v5.1 Sec 8.4", "Identity Model v3.2", "Release Notes Feb 2026 #2847"]
Bedrock model: Claude 3.5 Sonnet (Guardrails: NAIM-Support-Agent-Guardrails v2.1)
Confidence: 0.89 (above 0.70 groundedness threshold — no escalation required)
Agent action taken: "Inserted steps into Case Notes + Drafted client email"
───────────────────────────────────────────────────────
```

This audit trail provides:
- **FINRA-equivalent compliance** for any regulated communication linked to support
- **SOC 2 Type II evidence** demonstrating AI-generated content is reviewed by humans
- **Agent quality assurance** — supervisor can review what NAIM suggested vs. what agent sent
- **Model improvement feedback loop** — "Agent used all 4 steps" vs. "Agent modified steps"
  feeds back into SageMaker Model Monitor

---

## 7. EventBridge Automation: Proactive Intelligence Pipeline

### The Three Automated Triggers

```
TRIGGER 1: WEEKLY — Knowledge Base Freshness Check (Sundays 02:00 EST)

EventBridge Rule: cron(0 7 ? * SUN *)
        ↓
batch-create Lambda: Inventory check
  → List all docs in s3://nomura-naim-docs/ with LastModified > 7 days ago
  → Compare against Bedrock KB metadata index
  → Any doc in S3 not in KB index → trigger ingestion job
        ↓
CloudWatch Metric: NomuraNAIM/KnowledgeBaseStaleness
  → Alert if any active app guide version > 14 days without KB refresh
        ↓
Slack #naim-platform-ops: "Weekly KB health: 100% coverage — no stale indices"
```

```
TRIGGER 2: NIGHTLY — New Resolved Case Ingestion (Weeknights 23:30 EST)

EventBridge Rule: cron(30 4 ? * MON-FRI *)
        ↓
nightly-job Lambda:
  1. Query Salesforce Data Cloud: Cases closed today with ResolutionNotes != null
  2. Export to S3 raw tier
  3. Invoke Comprehend PII masking batch job
  4. Ingest masked cases into Bedrock KB (ticket_corpus_index)
  5. Update Aurora: {cases_added, corpus_total, coverage_pct}
        ↓
comment-catalog S3 object updated (append-only — audit trail)
        ↓
CloudWatch Dashboard: "NAIM Corpus Growth" — total indexed cases, FCR correlation trend
```

```
TRIGGER 3: REAL-TIME — GitLab Release Webhook (Triggered on each app release)

GitLab webhook → API Gateway → Lambda: "detect_doc_changes"
  IF /docs/ folder changed:
    → Extract: new PDFs, updated Markdown files
    → Upload to S3 docs tier
    → Bedrock KB: incremental re-ingestion (only changed chunks re-embedded)
    → Bedrock version tag: app_doc_version = "v5.2" (replaces v5.1 in metadata)
    → NAIM Co-Pilot now serves v5.2 content for all new queries
        ↓
Platform notification: "App Guide v5.2 indexed — NAIM reflecting current release"
```

### Proactive Sentiment Alert Pipeline

Beyond the real-time triage in Use Case 3, NAIM runs a proactive sentiment health check:

```
DAILY at 07:00 EST (before US support shift opens):

nightly-job Lambda runs Comprehend sentiment sweep on:
  → All open cases where last client email is > 24 hours without agent response
  → Any Tier-1/Tier-2 client with 3+ cases open simultaneously
        ↓
SageMaker Model 2 re-scores each case: frustration velocity (sentiment trending up or down?)
        ↓
Report sent to Support Ops Manager:
  "At-risk cases today: 3
   Case #00089241 — Hedge Fund X ($2.3B) — Frustration: 0.94 — 47h without resolution
   Case #00089188 — Pension Fund Y ($890M) — Frustration: 0.71 — trending UP
   Case #00089104 — RIA Z ($210M) — Frustration: 0.58 — stable"
        ↓
Support Ops Manager proactively calls head of client technology team
BEFORE the client escalates — not after
```

**This converts reactive incident management into proactive relationship management.**
The support team's morning stand-up gets a data-driven agenda rather than a surprise queue.

---

## 8. Implementation Roadmap

### Five-Phase Delivery Plan

| Phase | Timeline | Key Deliverables | Success KPI | Guardrail |
|-------|----------|-----------------|-------------|-----------|
| **Phase 1: Knowledge Base Foundation** | Q1 2026 (Weeks 1–6) | Audit existing Confluence content. Migrate current-version app guides to S3. Configure Bedrock KB with SSOT 1 (app documentation only). Deploy Comprehend PII masking pipeline for SSOT 2 preparation. | 100% of active app guide versions indexed in Bedrock KB; zero PII in ticket corpus | Do NOT deploy LWC to agents until KB quality is validated — spot-check 50 sample queries before rollout |
| **Phase 2: Ticket Corpus Ingestion** | Q2 2026 (Weeks 7–14) | Zero-Copy from Salesforce Data Cloud → S3. Run Comprehend PII masking on 5-year backlog. Ingest masked corpus into Bedrock KB (SSOT 2). Configure nightly EventBridge pipeline for new case ingestion. | 14,000+ historical cases indexed; nightly pipeline operational; FCR baseline established | Validate PII masking coverage at 99.9% before ticket corpus goes live in KB |
| **Phase 3: LWC Co-Pilot MVP** | Q2–Q3 2026 (Weeks 12–22) | Deploy LWC in Service Cloud Case Console utility bar. Bedrock integration for Use Cases 1 and 3 (troubleshooting + sentiment triage). Lambda orchestrator + API Gateway. | Agent troubleshooting lookup time < 60s; MTTR reduced ≥30% at pilot; LWC adoption >70% | Pilot with 5 agents before full rollout; measure KB confidence scores and agent override rate |
| **Phase 4: SageMaker JumpStart Models** | Q3–Q4 2026 (Weeks 22–34) | Fine-tune Llama 3.1 8B on Nomura ticket taxonomy (Model 1: clustering). Fine-tune XGBoost on escalation labels (Model 2: frustration/churn). Deploy Use Case 2 (historical ticket synthesis). | FCR ≥59%; MTTR ≤1.7h; Tier-1 escalation rate ≤25% | Clarify bias check by client tier before production; F1 ≥0.72 required for ticket clustering |
| **Phase 5: Proactive Automation** | Q4 2026–Q1 2027 (Weeks 34–52) | Weekly KB freshness EventBridge. GitLab webhook CI/CD for doc re-indexing. Daily sentiment sweep. Closed-case learning loop fully operational. | KB always ≤7 days from latest app release; FCR upward trend confirmed; zero PII incidents | Monitor Bedrock KB staleness CloudWatch alarm; SLA: any release must be in KB within 4 hours of deployment |

---

## 9. Proactive Risk Mitigation: The Lemons Tables

### AI & Knowledge Base Risk

| Risk | Proactive Mitigation | Expected Outcome |
|------|---------------------|------------------|
| **Agent Hallucination (Bedrock invents non-existent UI flow)** | Bedrock Guardrails: groundedness threshold 0.70 — any claim not traceable to KB source is blocked + flagged. Every response cites source document, version, and section. Agent sees source citation and can verify. | Client receives only responses grounded in verified application documentation. Zero invented button paths reach clients. |
| **Stale Knowledge Base (app UI changes, KB not refreshed)** | Event-driven CI/CD: GitLab webhook triggers Bedrock re-ingestion on every release with doc changes. Weekly EventBridge fallback runs full coverage audit. CloudWatch alert if any active version is >14 days stale. | Knowledge Base continuously aligned with deployed production version. NAIM never cites a deprecated UI path. |
| **SageMaker ticket clustering model performance drift** | SageMaker Model Monitor: daily data drift report comparing live case distributions vs. training distribution. Quarterly trigger: automatic retraining job if drift threshold exceeded (Wasserstein distance > 0.15). | Ticket clustering stays accurate as Nomura's application evolves and new issue categories emerge. |
| **Frustration classifier over-escalating (alert fatigue)** | SageMaker Clarify: validate precision/recall on holdout set before production. Threshold tuned to minimize false positives for Tier-1 clients specifically. Weekly review of escalation rate by Tier: alert if >30% Tier-1 cases auto-escalate (suggests threshold miscalibration). | Senior support team receives actionable alerts, not noise. Trust in NAIM escalation signal maintained. |
| **PII appearing in Co-Pilot response (client data leak)** | Bedrock Guardrails PII redaction at output layer (defense in depth: Comprehend masks at ingestion; Guardrails blocks at output). CloudWatch alarm: any PII entity detected in Bedrock response → immediate pipeline halt + security team notification. | Zero client PII exposed to agents through NAIM. GDPR / SEC data privacy obligations met architecturally, not by policy. |

### CRM & Integration Risk

| Risk | Proactive Mitigation | Expected Outcome |
|------|---------------------|------------------|
| **LWC Co-Pilot panel fails to load (Salesforce component error)** | Graceful degradation: if NAIM API call fails or times out (>3s), LWC shows "Co-Pilot temporarily unavailable — manual Confluence search fallback." Agent workflow never blocked by NAIM dependency failure. | Support operations continue without NAIM if needed — zero dependency-created downtime. |
| **API Gateway latency spike (Bedrock response slow)** | Lambda timeout set to 8s; if Bedrock response > 5s, Lambda streams partial response to LWC with "Searching..." status. SQS FIFO queue for non-urgent requests (ticket corpus lookups). Real-time triage (sentiment) always prioritized over historical searches. | Agent gets partial guidance quickly rather than waiting for full response. Triage decisions never delayed. |
| **Salesforce Data Cloud Zero-Copy pipeline failure** | Step Functions idempotency key = CaseID + CloseDate; retry with exponential backoff × 3. CloudWatch alarm: if last successful nightly job > 25 hours ago → alert #naim-platform-ops. | Ticket corpus updates re-tried automatically; knowledge base never more than 48 hours stale from case history. |
| **Salesforce Platform Event delivery guarantee** | NAIM LWC uses enhanced-throughput Platform Events (guaranteed delivery). If event queue backup detected (>1000 unprocessed events), auto-scale Lambda concurrency. | Co-Pilot requests processed in order, without loss, even during peak support hours. |
| **EventBridge weekly trigger fails silently** | CloudWatch Events monitoring with alarm: `NomuraNAIM/WeeklyKBRefresh` metric must increment every Sunday between 02:00–04:00 EST or alarm fires. Runbook published for on-call team. | Knowledge base freshness guarantee maintained even if automation has silent failures. |

### Data Governance & Compliance Risk

| Risk | Proactive Mitigation | Expected Outcome |
|------|---------------------|------------------|
| **GDPR/SEC: Client PII in historical ticket KB** | Comprehend PII masking runs on every ticket before S3 storage (no raw PII in S3 or OpenSearch at any time). Masking audit: 1% daily sample checked by compliance — any PII miss triggers immediate reprocessing of entire daily batch. | Historical ticket KB is GDPR-compliant by construction — personal data never enters OpenSearch. |
| **Agent sends AI-generated content without review** | Bedrock response always renders with: "⚠️ AI-Assisted Response — Review before sending to client." LWC UI never has a one-click "Send to Client" from the Co-Pilot panel — agent must copy to Case Email and review. Audit log captures whether agent modified response before dispatch. | Human-in-the-loop enforced architecturally. Every client communication reviewed by a human. |
| **Knowledge Base contains outdated compliance language** | Legal/Compliance team has quarterly KB review access. Any document tagged `doc_type: regulatory` or `doc_type: compliance` cannot be updated via GitLab webhook alone — requires Compliance approval step (Salesforce Flow gating S3 upload for these doc types). | Compliance-sensitive documentation flows through an additional human approval gate before being re-indexed. |
| **Support agent uses NAIM for investment advice** | Bedrock Guardrails DeniedTopics: "Investment advice" and "Portfolio recommendations" are blocked at output. Any query routed through NAIM that generates a blocked response triggers a CloudWatch event for agent supervision review. | System architecture prevents out-of-scope use. NAIM stays in its lane: application support, not financial guidance. |
| **Model bias: deprioritizing non-Tier-1 clients** | SageMaker Clarify bias detection runs on all model outputs by client segment before production deployment. Monthly audit: compare MTTR by client tier before/after NAIM to confirm no discriminatory treatment of smaller clients. | NAIM improves support quality for all clients, not just the largest AUM relationships. |

---

## 10. Interview Strategy: Talking Points for Kathleen, Stuart, and Shaw

### For Kathleen — Client Experience + AI Infrastructure Architecture

**The Shared Architecture Hook:**

> "Kathleen, the external AI Digital Agent you've built for DDQ deflection and this internal
> Support Co-Pilot are the same foundational architecture. Both use Amazon Bedrock as the
> generative layer, both use an OpenSearch Serverless vector store as the knowledge repository,
> and both require Bedrock Guardrails for compliance controls. Here is what that means in
> practice: the Knowledge Base you build for the external agent — fund documentation, strategy
> overviews, the approved DDQ answer set — and the Knowledge Base NAIM uses for application
> manuals and ticket history share the same technical infrastructure. You invest in OpenSearch
> once. You invest in Bedrock guardrails once. Both the client-facing agent and the agent
> support Co-Pilot benefit immediately."

**The Client Journey Continuity Angle:**

> "There is a client experience dimension to this that goes beyond the support department.
> When a client calls because they cannot use the portal, the support agent's response quality
> is part of the client experience — it is not a back-office problem. If the agent gives the
> client an incorrect resolution path sourced from a 14-month-old Confluence guide, that
> affects NPS. NCIE's feedback capture would collect that negative sentiment. The irony would
> be that the AI Agent handling external queries is performing better than the human supported
> by outdated internal tools. NAIM closes that gap: the internal agent co-pilot is as
> current as the application itself because the knowledge base updates the moment we deploy
> a new release."

**The Clincher — Same Platform, Both Surfaces:**

> "What I am describing is not two AI projects — it is one AI platform serving two surfaces.
> The client-facing Agent resolves Tier-0 queries. NAIM empowers human agents for Tier-2
> complexity. Both consume the same Bedrock Knowledge Base. When the agent is giving advice,
> it comes from the same source that the external bot uses. The client gets a consistent
> answer regardless of which channel they use to reach Nomura — and that consistency is
> what builds trust in the long run."

---

### For Stuart — Client Platforms + Salesforce Architecture

**The LWC "Glass and Engine" Architecture Hook:**

> "Stuart, I want to describe the architecture using the boundary we care about: Salesforce
> is the glass — it is what the agent sees and touches — and AWS is the engine. The
> Lightning Web Component lives inside Service Cloud and surfaces results directly in the
> Case Console. The agent never leaves Salesforce. But underneath that glass, API Gateway
> and Lambda are routing the query to either Bedrock or SageMaker depending on the intent.
> We use SageMaker for the quantitative work — ticket clustering, frustration scoring,
> churn probability — where a fine-tuned model on Nomura's own case taxonomy outperforms
> any off-the-shelf prompt. We use Bedrock for the qualitative generation work — the
> troubleshooting synthesis, the client email draft, the de-escalation script — where Claude's
> language quality is the primary output. Separating concerns between SageMaker and Bedrock
> is exactly the right FinOps move: we pay for inference only, not idle compute."

**The Stateless Architecture Narrative:**

> "The specific architectural choice that matters for Stuart's platform team is this: NAIM
> is stateless at the AWS layer. Lambda receives a query, routes it, gets a response, and
> disappears. The state lives in two places only: Salesforce (Case Feed audit trail) and
> OpenSearch Serverless (knowledge vectors). There is no custom NAIM application server to
> maintain. No custom container. No persistent process that needs patching. The engineering
> overhead is minimal once the knowledge base and models are deployed. Stuart's team owns
> the LWC component and the Platform Event schema. AWS infrastructure is entirely managed."

**The Platform Flywheel:**

> "Every case NAIM resolves enhances NAIM's ability to resolve the next one. Resolved
> cases ingested nightly into the vector corpus make the semantic clustering more accurate.
> Agent feedback on whether they used the Co-Pilot steps tells us which knowledge base chunks
> are high-quality. Model Monitor catches drift before it affects resolution quality. The
> platform improves without a manual re-engineering cycle — it improves by operating. That
> is the flywheel that makes the ongoing infrastructure cost de minimis relative to the
> growing value it delivers."

---

### For Shaw — Technology Partnerships + AWS Architecture

**The Reference Architecture Alignment Hook:**

> "Shaw, the reference architecture in the AWS diagram maps directly to NAIM:
> the batch-create Lambda handling /summaries and /comments API endpoints is our document
> ingestion and ticket corpus pipeline. The prompt Lambda calling Bedrock is our Co-Pilot
> response generation. The summary Lambda calling Comprehend is our PII masking and sentiment
> pipeline. The nightly-job EventBridge automation is our closed-case learning loop. The
> OpenSearch 'fletcher-db' equivalent is our ticket_corpus_index. This is not a novel
> architecture — it is a proven AWS pattern applied to Nomura's specific domain with
> proprietary training data that no competitor can replicate."

**The FinOps Case:**

> "The cost model is bounded and transparent. Bedrock Phase 1 at ~500 cases/day averages
> approximately $900/month in inference costs. SageMaker JumpStart endpoint for both
> predictive models runs on a single ml.g4dn.xlarge instance with auto-scale to zero outside
> business hours — approximately $129/month. OpenSearch Serverless at the minimum 2 OCU
> configuration is $350/month. Total: under $1,400/month for the entire AI infrastructure.
> The FCR improvement from 42% to 59% — conservatively — means we resolve 85 more cases
> per day without Tier-2 involvement. At a fully loaded Tier-2 engineering cost of $120/hour
> and 2 hours average for escalated cases, those 85 cases represent $20,400 in daily
> engineering cost avoidance. The AI infrastructure pays for itself in under one business day."

**The Competitive Differentiation Angle:**

> "The SageMaker JumpStart models trained on Nomura's 5-year ticket taxonomy are
> the architectural moat. Competitors can deploy Bedrock tomorrow. They cannot deploy a
> ticket clustering model trained on Nomura's own IT taxonomy, Nomura's own application
> modules, Nomura's own resolution patterns. Every quarter we retrain on new case data,
> the model improves. Every year that passes, the competitive advantage from domain-specific
> fine-tuning widens. That is the case for the JumpStart investment: not 'we have AI,'
> but 'we have AI that knows how Nomura's specific application breaks and how to fix it.'"

---

## Executive Clincher: The "Uninterrupted Journeys" Platform

> "What I am proposing is a shift from reactive, Confluence-dependent support to an
> **AI-augmented, knowledge-current, sentiment-aware Support Intelligence Platform.**
> The NAIM Co-Pilot makes the right answer ambient — agents always have the exact
> troubleshooting step for the application version the client is running, grounded in
> the documentation that reflects today's deployment, not last quarter's sprint.
> The historical ticket synthesis makes institutional memory searchable — five years of
> solved problems are available in 90 seconds, not buried in closed JIRA tickets.
> The sentiment-driven triage makes relationship risk visible before it becomes a
> relationship loss — a Tier-1 Hedge Fund with $2.3B AUM gets a senior agent briefed
> and calling within 15 minutes, not a 48-hour auto-reply.
>
> This is not a support efficiency initiative. It is a client retention architecture.
> Because when institutional clients experience 'Uninterrupted Journeys' — their
> complex technical issues are resolved in seconds by a well-briefed agent rather than
> putting them on hold to search through legacy wikis — they do not question the
> technology platform they are invested on. And that is the outcome the Head of
> Client Technology is responsible for: platforms that make clients more committed
> to Nomura, not less."

---

## Document Metadata

| Field | Detail |
|-------|--------|
| Prepared by | Calvin Lee |
| Target audience | Kathleen Hess McNamara · Stuart Mumley · Shaw Levin |
| Role | ED, Head of Client Technology — Nomura Asset Management International |
| Purpose | Case Study 3: NAIM — Support Agent Co-Pilot with Salesforce Service Cloud + Bedrock + SageMaker JumpStart |
| Architecture pattern | AWS Reference Architecture (Bedrock + Comprehend + OpenSearch + EventBridge) mapped to Nomura support domain |
| Complementary documents | AI_CHATBOT_CRM_SALESCLOUD_INTEGRATION.md (Case Study 2) · NOMURA_CLIENT_INSIGHT_ENGINE.md · DIGITAL_LAYER_SALESFORCE_INTEGRATION.md · README.md |
| Last updated | February 2026 |

This document demonstrates that the NAIM support architecture is not a standalone proof-of-concept —
it is the operational intelligence layer that protects the AUM relationships that the
distribution intelligence platform wins. External clients are acquired by the Distribution
Intelligence Platform (Case Study 2). Internal support teams are empowered by NAIM (Case Study 3)
to ensure those clients stay. The two architectures share one infrastructure. One investment.
Two surfaces. Complete client lifecycle coverage.
