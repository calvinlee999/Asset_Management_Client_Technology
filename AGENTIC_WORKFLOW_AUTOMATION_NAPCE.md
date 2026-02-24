# Nomura Agentic Proposal & Compliance Engine (NAPCE)
## Agentic Workflow Automation — Case Study 4

**Prepared for:** Kathleen Hess McNamara (ED, Client Experience & AI) | Stuart Mumley (Director, Client Platforms) | Shaw Levin (VP, Technology Partnerships)
**Role:** Executive Director, Head of Client Technology — Nomura Asset Management International
**System Name:** NAPCE — Nomura Agentic Proposal & Compliance Engine

---

> **Positioning Statement:** NAPCE is the fourth surface of the Nomura Uninterrupted Journeys Platform — the agentic layer that compresses the 12-24 month institutional mandate lifecycle. Where the AI Chatbot (Case Study 3) serves the internal support co-pilot, NAPCE serves the external distribution engine: auto-populating DDQs in hours instead of weeks, monitoring GIPS composites in real time, and accelerating institutional sales pipelines from prospect to mandate. It is orchestrated by Amazon MWAA, made fault-tolerant by the Saga pattern, and grounded by Amazon Bedrock agents that are domain-bound, IAM-scoped, and auditable.

---

## Table of Contents

1. [The Problem — Manual Friction in Institutional Distribution](#1-the-problem)
2. [Three-Layer Architecture — NAPCE System Design](#2-three-layer-architecture)
3. [Three Use Cases — DDQ, GIPS, Mandate Pipeline](#3-three-use-cases)
4. [AI Engine Design — Bedrock Agents + MWAA DAG Specs](#4-ai-engine-design)
5. [Knowledge Base Construction — Textract + OpenSearch](#5-knowledge-base-construction)
6. [Saga Distributed Transaction Workflow](#6-saga-distributed-transaction-workflow)
7. [MWAA DAG Architecture — Monte Carlo + S3 Pipeline](#7-mwaa-dag-architecture)
8. [5-Phase Implementation Roadmap](#8-implementation-roadmap)
9. [Lemons Tables — Risk Identification & Mitigation](#9-lemons-tables)
10. [Panel Talking Points & Executive Clincher](#10-panel-talking-points)

---

## 1. The Problem

### The Manual Burden in Institutional Distribution

Nomura Asset Management International competes for institutional mandates — pension funds, endowments, sovereign wealth funds — in a market where $50M–$5B+ decisions take 12–24 months to close. Three bottlenecks throttle every deal:

**Bottleneck 1 — Due Diligence Questionnaires (DDQ)**

An institutional investor sends a DDQ before awarding any mandate. A single DDQ contains 50 to 200+ structured questions across six categories: Company Information, Financial Health, Compliance & Legal (GDPR, HIPAA, AML), IT & Data Security, Operational Resilience, and ESG Policy. Answering each question requires cross-referencing live operational data, prior approved responses, legal documentation, and compliance-verified language. Traditionally, a dedicated analyst team spends 3–6 weeks per DDQ — reading every question manually, hunting legacy systems for current answers, formatting responses, and routing through compliance review. A single incorrect answer about technology controls or GIPS methodology can disqualify the entire bid.

**Bottleneck 2 — GIPS-Compliant Performance Reporting**

The CFA Institute's Global Investment Performance Standards (2019 edition, effective January 1, 2020) require that all investment managers claiming GIPS compliance present performance using firmwide-adopted composite construction. As of 2024, 1,700+ firms worldwide claim GIPS compliance. Institutional due diligence always includes a GIPS section: Are composite assignments correct? Are time-weighted returns calculated per standard? Has a third-party verifier been engaged? Errors in GIPS composites are not just technical mistakes — they are reputational and regulatory failures. Manual composite validation means quarterly spot-checks, not continuous monitoring. NAPCE's Compliance Agent closes this gap.

**Bottleneck 3 — Institutional Sales Mandate Pipeline Velocity**

The 7-stage institutional mandate pipeline — Lead Generation → Qualification → Prospect Nurturing → Proposal / RFP Submission → Negotiation → Mandate Close → Post-Award Onboarding — spans 12–24 months across Salesforce CRM and Snowflake data warehouse. At each stage, the distribution team manually prepares bespoke materials: pitch decks, risk analytics, Monte Carlo simulation outputs, attribution data, and compliance certifications. At $1B pipeline targets with 25% win rates, every week of pipeline friction translates to delayed AUM growth. AI can compress proposal preparation from weeks to hours — but only when orchestration, fault tolerance, and data integrity are engineered to production standards.

### The Three Leaps NAPCE Makes

| Constraint Today | NAPCE Target State |
|---|---|
| 3–6 weeks per DDQ response | 60–80% faster — hours, not weeks |
| Quarterly GIPS spot-checks, manual composite validation | Continuous, automated GIPS compliance monitoring |
| 3–5 days to assemble institutional proposal with Monte Carlo simulation | Hours with automated MWAA DAG orchestration + Amazon Batch |

---

## 2. Three-Layer Architecture

### Architecture Diagram

```
╔═════════════════════════════════════════════════════════════════════════╗
║           NAPCE — NOMURA AGENTIC PROPOSAL & COMPLIANCE ENGINE          ║
╠═════════════════════════════════════════════════════════════════════════╣
║                                                                         ║
║  ┌─────────────────────────────────────────────────────────────────┐  ║
║  │          LAYER 1: ORCHESTRATION                                  │  ║
║  │          Amazon MWAA (Managed Workflows for Apache Airflow)      │  ║
║  │                                                                  │  ║
║  │  ┌──────────────┐  ┌──────────────────┐  ┌───────────────────┐  │  ║
║  │  │ DAG Trigger  │  │ Saga Seq Engine  │  │ S3 / Glue ETL     │  │  ║
║  │  │ (DDQ Upload) │  │ Compensate Tasks │  │ Integration       │  │  ║
║  │  └──────┬───────┘  └────────┬─────────┘  └───────────────────┘  │  ║
║  └─────────┼──────────────────┼────────────────────────────────────┘  ║
║            │                  │                                         ║
╠════════════▼══════════════════▼═════════════════════════════════════════╣
║  LAYER 2: AI AGENTIC (Domain-Bound, IAM-Scoped)                         ║
║                                                                          ║
║  ┌─────────────────────┐  ┌─────────────────────┐  ┌────────────────┐  ║
║  │  DDQ AGENT          │  │  RFP AGENT           │  │ COMPLIANCE     │  ║
║  │  Amazon Bedrock     │  │  Amazon Bedrock      │  │ AGENT          │  ║
║  │  Claude 3.5 Sonnet  │  │  Claude 3.5 Sonnet   │  │ Bedrock +      │  ║
║  │                     │  │                      │  │ SageMaker      │  ║
║  │  → OpenSearch KB    │  │  → OpenSearch KB     │  │                │  ║
║  │  → Textract parsed  │  │  → S3 data lake      │  │ → SageMaker    │  ║
║  │    DDQ questions    │  │  → Salesforce CRM    │  │   GIPS engine  │  ║
║  │  → IAM role-scoped  │  │  → Pipeline context  │  │ → EventBridge  │  ║
║  └─────────────────────┘  └─────────────────────┘  └────────────────┘  ║
║                                                                          ║
║  ┌──────────────────────────────────────────────────────────────────┐  ║
║  │  AI INFRASTRUCTURE                                               │  ║
║  │  Amazon Textract → Amazon OpenSearch → Amazon SageMaker          │  ║
║  │  Amazon Batch (Monte Carlo) → IAM + OIDC Identity Mesh           │  ║
║  └──────────────────────────────────────────────────────────────────┘  ║
╠══════════════════════════════════════════════════════════════════════════╣
║  LAYER 3: COMPLIANCE & REGULATORY                                        ║
║                                                                          ║
║  ┌──────────────────────────────────────────────────────────────────┐  ║
║  │  GIPS Compliance Monitor                                         │  ║
║  │  SageMaker (deterministic GIPS calculations) +                   │  ║
║  │  EventBridge triggers + Lambda alerts                            │  ║
║  │  → Continuous composite validation                               │  ║
║  │  → Zero-defect GIPS audit posture                                │  ║
║  └──────────────────────────────────────────────────────────────────┘  ║
╠══════════════════════════════════════════════════════════════════════════╣
║  SHARED DATA LAYER                                                       ║
║  Amazon S3 (data lake) · Snowflake (data warehouse) ·                    ║
║  Salesforce (CRM) · AWS Glue (ETL) · Amazon Lambda (serverless glue)     ║
╚══════════════════════════════════════════════════════════════════════════╝
```

### AWS Service Architecture Mapping

| AWS Service | NAPCE Function | Why This Service |
|---|---|---|
| **Amazon MWAA** | Orchestration backbone — schedules, sequences, and monitors autonomous agents via DAGs | Native Apache Airflow; deep S3/Glue integration; built-in retry and compensation logic |
| **Amazon Bedrock (Claude 3.5 Sonnet)** | Three domain-bound agents (DDQ, RFP, Compliance) — cognitive routing, answer generation, compliance cross-reference | Managed LLM with IAM-scoped data access; Bedrock Guardrails for GIPS output control |
| **Amazon SageMaker** | Deterministic GIPS composite calculations; risk model execution | AI does NOT calculate returns — SageMaker model does. Ensures mathematical fidelity |
| **Amazon Textract** | Document Intelligence — pre-processes multi-page institutional DDQ PDFs, complex tables | Structured extraction before Bedrock agent receives questions; handles nested tables |
| **Amazon Batch** | Monte Carlo risk simulations (100 per institutional proposal) | Compute-heavy, detached from LLM; MWAA triggers Batch as Step 1 of Saga |
| **Amazon OpenSearch** | Knowledge base for DDQ/RFP answer retrieval; centralized approved-answer corpus | Semantic search over 50,000+ approved DDQ responses; vector search for question similarity |
| **Amazon S3** | Simulation results storage (Saga Step 2); data lake integration | Durable intermediate storage; Saga compensation reads from S3 to rollback |
| **AWS Glue** | ETL pipelines feeding agent knowledge bases from Snowflake and operational data | Automated schema inference; populates OpenSearch from Snowflake tables |
| **AWS Lambda** | Serverless integrations connecting Bedrock agents with OpenSearch, SageMaker, and alerts | Event-driven; EventBridge + Lambda trigger GIPS monitoring on schedule |
| **IAM + OIDC** | Identity mesh — Bedrock Agent assumes IAM role mapped to user's permissions | Agent cannot retrieve fund data the requesting user isn't authorized to view |
| **Salesforce CRM** | Mandate pipeline tracking; 12–24 month institutional sales cycle management | Existing CRM; Saga scope includes Salesforce as a distributed transaction participant |
| **Snowflake** | Data warehouse — cross-system data consistency; DDQ factual data source | Zero-copy sharing to S3; Saga rollback scope covers Snowflake intermediate writes |

### BIAN Service Domain Alignment

NAPCE operates across four BIAN service domains:

| BIAN Domain | NAPCE Component | Behaviors |
|---|---|---|
| **Sales & Distribution** | Institutional mandate pipeline (RFP/DDQ stages) | Develop, Pursue, Fulfill — compressed by AI agents |
| **Customer & Party Agreement** | DDQ compliance and operational questions | Assess, Agree — automated by DDQ Agent |
| **Investment Portfolio Management** | GIPS composite data flowing into proposals | Monitor, Evaluate — automated by Compliance Agent |
| **Sales Enablement & Advisor Distribution** | RFP knowledge base and content governance | Develop, Distribute — automated by RFP Agent |

---

## 3. Three Use Cases

### Use Case 1 — DDQ Auto-Population

**Trigger:** Institutional prospect uploads a DDQ via the client portal or emails a PDF link.

**The Problem:** A 200-question DDQ spans six sections. Technology questions — about cybersecurity, disaster recovery, data governance, vendor controls, and business continuity — require answers from live operational systems, not cached templates. An analyst must pull current answers from 8+ internal systems, verify accuracy, format to the prospect's required structure, and route through compliance. Typical time: 3–6 weeks.

**NAPCE DDQ Agent workflow:**
```
Step 1: Amazon Textract
  → DDQ PDF uploaded to S3
  → Textract DocumentAnalysis API extracts questions from tables,
    multi-column layouts, nested sub-questions
  → Structured JSON question list extracted with section metadata

Step 2: Bedrock DDQ Agent (Claude 3.5 Sonnet)
  → Receives structured question list from Textract
  → For each question: semantic search OpenSearch knowledge base
    (vector similarity against 50,000+ approved DDQ responses)
  → High-confidence match (>0.85): draft response auto-populated
  → Low-confidence match (<0.85): flag for human SME review
  → Technology section questions (IT/cybersecurity/DR): agent
    dynamically retrieves live operational data from S3 data lake
    (not from cached, potentially stale knowledge base entries)

Step 3: MWAA Orchestration
  → DAG monitors question confidence scores across all sections
  → Routes human review queue to institutional sales team
  → Aggregates final responses into required DDQ format
  → Delivers completed draft within target SLA

Step 4: Human Review + Sign-Off
  → Compliance team reviews flagged responses
  → DDQ Agent audit log: what was auto-populated vs. what was reviewed
  → Final submission authorized by Head of Client Technology
    for all technology control sections
```

**KPI:** 60–80% reduction in DDQ preparation time; 100% technology section accuracy (live data retrieval, never stale cache).

---

### Use Case 2 — GIPS Compliance Monitoring

**Trigger:** Continuous — EventBridge schedule triggers Compliance Agent daily + on any portfolio data change event from Aladdin CDC.

**The Problem:** GIPS compliance requires that composite assignments, time-weighted returns, fee presentations (gross vs. net), and benchmark disclosures are all correctly maintained across the firm's entire portfolio universe. Manual validation happens quarterly at best. Between spot-checks, composite data can drift — portfolios misassigned, calculation parameters changed, benchmark references updated — creating audit exposure that only surfaces during third-party GIPS verification.

**NAPCE Compliance Agent workflow:**
```
Continuous Monitoring Loop:
  EventBridge trigger (daily schedule + portfolio change events)
        ↓
  Lambda invokes Bedrock Compliance Agent
        ↓
  Agent queries SageMaker GIPS Engine:
    → Composite construction rules (which portfolios belong to which composite)
    → Time-weighted return calculations (deterministic — SageMaker, not LLM)
    → Gross vs. net return spread validation
    → Benchmark assignment cross-reference
        ↓
  Agent cross-references against GIPS 2019 Standard rules:
    → Composite inception dates consistent with portfolio onboarding dates?
    → All discretionary portfolios assigned to at least one composite?
    → Terminated portfolios maintained in historical composite record
       for ≥5 years?
    → Performance advertising materials compliant with Rule 206(4)-1?
        ↓
  CRITICAL PRINCIPLE: LLM does NOT calculate returns.
    SageMaker model calculates returns deterministically.
    Bedrock agent monitors outputs for rule compliance only.
        ↓
  Alert routing:
    → Composite assignment drift: immediate alert to
      Performance Team + Head of Client Technology
    → GIPS rule violation: Salesforce case created,
      assigned to Compliance Manager
    → Audit trail: all monitoring results written to S3
      immutable log (CloudTrail-compliant)
```

**KPI:** Zero-defect GIPS compliance monitoring. Continuous validation replaces quarterly spot-checks. Third-party GIPS verifier receives automated audit package — no manual compilation.

---

### Use Case 3 — RFP / Institutional Mandate Pipeline Acceleration

**Trigger:** Salesforce opportunity moves to "Proposal Stage" (Stage 5 of 7 in the institutional mandate pipeline).

**The Problem:** An institutional RFP contains 50–150 questions about investment strategy, team, process, risk management, ESG policy, and performance track record. Preparing a bespoke RFP response requires coordinating across the investment team (strategy narrative), compliance (regulatory disclosures), technology (operational controls), and the quantitative team (Monte Carlo risk simulations and attribution data). Current preparation time: 2–5 days minimum. NAPCE compresses this to hours.

**NAPCE RFP Agent + MWAA workflow:**
```
MWAA DAG trigger: Salesforce opportunity reaches Proposal Stage
        ↓
Stage 1 — Agent Analysis:
  Bedrock RFP Agent receives:
    → Prospect profile from Salesforce (fund type, AUM tier, strategy interest)
    → Previous interaction history (what content they engaged with)
    → Regulatory jurisdiction (SEC, FCA, ASIC — affects disclosures)
  Agent defines proposal parameters:
    → Required Monte Carlo simulations: 100 scenarios
    → Strategy documents: which composites to include
    → ESG section: which framework (TCFD, SFDR)

Stage 2 — Task Execution (Saga Step 1):
  MWAA triggers Amazon Batch job:
    → 100 Monte Carlo risk simulations per proposal
    → Stress tests: interest rate shock ±200bps, credit spread widening ±150bps,
      equity market drawdown -30%
    → Worst-case / median / best-case portfolio outcomes generated

Stage 3 — Data Processing (Saga Step 2):
  Batch results extracted → written to S3 proposal folder
  Compensation trigger armed: if failure → rollback S3 intermediate data

Stage 4 — Response Assembly:
  Bedrock RFP Agent assembles proposal:
    → Knowledge base answers for strategy/team/process sections
    → Monte Carlo outputs for quantitative risk section
    → SageMaker GIPS Engine outputs for performance track record
    → Compliance Agent certification: "GIPS compliant — verified [date]"

Stage 5 — Finalization:
  Agent summarizes completed RFP proposal
  Alerts institutional sales RM via Salesforce notification
  Proposal ready for human review and client submission
```

**KPI:** 90% reduction in RFP/DDQ response times. 100% Monte Carlo simulation coverage (no proposals submitted without quantitative risk analysis). 12–24 month pipeline cycle compressed at every stage.

---

## 4. AI Engine Design

### Bedrock Agent Decision Matrix

| Agent | Input Source | Knowledge Base | Output | Confidence Routing |
|---|---|---|---|---|
| **DDQ Agent** | Textract-parsed question JSON | OpenSearch (50K+ approved DDQ responses) | Auto-populated DDQ draft | >0.85 auto-populate; <0.85 human queue |
| **RFP Agent** | Salesforce opportunity context + MWAA DAG parameters | OpenSearch + S3 data lake | RFP response package with Monte Carlo data | All responses human-reviewed before submission |
| **Compliance Agent** | SageMaker GIPS outputs + EventBridge triggers | GIPS 2019 Standard rules in OpenSearch | Composite validation report + alert routing | Rule violations escalate immediately; no confidence threshold — binary pass/fail |

### IAM Identity Mesh — Agent Permission Scoping

```
Institutional User (RM) authenticates via Cognito + OIDC
        ↓
IAM role assumed: arn:aws:iam::nomura::role/DDQAgent-[UserPermissionTier]
        ↓
Bedrock Agent inherits IAM role at invocation:
  → Can retrieve: data from S3 prefixes the user is entitled to view
  → Cannot retrieve: fund data from different Salesforce account hierarchy
  → Cannot retrieve: other users' proposal drafts (partition-key isolation)
        ↓
Every agent action audit-logged:
  → CloudTrail: IAM role assumed, data sources accessed
  → S3 immutable log: what question, which knowledge base entry, confidence score
  → Bedrock model invocation log: input prompt (sanitized), output, review status
```

**Security principle:** The Bedrock Agent cannot retrieve fund performance data that the RFP is not entitled to see. The identity mesh enforces this at the infrastructure layer — not at the application layer.

### MWAA DAG Specification

```python
# NAPCE DDQ Processing DAG — simplified structure
from airflow import DAG
from airflow.providers.amazon.aws.operators.lambda_function import LambdaInvokeFunctionOperator
from airflow.providers.amazon.aws.operators.batch import BatchOperator
from airflow.providers.amazon.aws.sensors.s3 import S3KeySensor
from airflow.operators.python import BranchPythonOperator

def route_by_confidence(**context):
    """Branch based on DDQ Agent confidence scoring result"""
    confidence_report = context['task_instance'].xcom_pull(
        task_ids='invoke_ddq_agent'
    )
    low_confidence_count = confidence_report.get('low_confidence_questions', 0)
    if low_confidence_count > 0:
        return 'route_to_human_review'
    return 'finalize_ddq_response'

with DAG(
    dag_id='napce_ddq_processing',
    schedule_interval=None,  # Event-driven: triggered on S3 DDQ upload
    catchup=False,
    tags=['napce', 'ddq', 'saga']
) as dag:

    # Step 1: Textract document analysis
    textract_analysis = LambdaInvokeFunctionOperator(
        task_id='textract_ddq_document',
        function_name='napce-textract-ddq-processor',
        payload='{{ dag_run.conf | tojson }}'
    )

    # Step 2: Wait for Textract output in S3
    wait_for_textract = S3KeySensor(
        task_id='wait_textract_output',
        bucket_name='nomura-napce-staging',
        bucket_key='textract-output/{{ dag_run.conf["ddq_id"] }}/complete.json'
    )

    # Step 3: Invoke DDQ Bedrock Agent
    invoke_ddq_agent = LambdaInvokeFunctionOperator(
        task_id='invoke_ddq_agent',
        function_name='napce-bedrock-ddq-agent',
        payload='{"ddq_id": "{{ dag_run.conf[\'ddq_id\'] }}"}'
    )

    # Step 4: Branch — route by confidence score
    confidence_branch = BranchPythonOperator(
        task_id='confidence_routing',
        python_callable=route_by_confidence
    )

    # Step 5a: Human review queue (low-confidence questions)
    route_human_review = LambdaInvokeFunctionOperator(
        task_id='route_to_human_review',
        function_name='napce-human-review-router',
        payload='{"ddq_id": "{{ dag_run.conf[\'ddq_id\'] }}"}'
    )

    # Step 5b: Finalize high-confidence DDQ (no human review needed)
    finalize_ddq = LambdaInvokeFunctionOperator(
        task_id='finalize_ddq_response',
        function_name='napce-ddq-finalizer',
        payload='{"ddq_id": "{{ dag_run.conf[\'ddq_id\'] }}"}'
    )

    # Saga compensation: clean S3 if pipeline fails mid-execution
    saga_compensate = LambdaInvokeFunctionOperator(
        task_id='saga_compensation_rollback',
        function_name='napce-saga-compensate',
        payload='{"ddq_id": "{{ dag_run.conf[\'ddq_id\'] }}", "action": "rollback"}',
        trigger_rule='one_failed'
    )

    # DAG dependency chain
    (
        textract_analysis
        >> wait_for_textract
        >> invoke_ddq_agent
        >> confidence_branch
        >> [route_human_review, finalize_ddq]
    )
    # Compensation fires if any upstream task fails
    [textract_analysis, invoke_ddq_agent] >> saga_compensate
```

---

## 5. Knowledge Base Construction

### Textract Document Intelligence Pipeline

Amazon Textract DocumentAnalysis API handles the most complex DDQ formats that standard PDF parsers fail on: multi-column tables, checkboxes, form fields, nested sub-questions, and cross-referenced sections.

```
TEXTRACT DOCUMENT PROCESSING PIPELINE

Input Sources:
  → Institutional DDQ PDFs (50–200 pages, complex table structure)
  → RFP documents with structured question grids
  → Previous DDQ response packages (to build knowledge base corpus)

Textract API Invocations:
  1. StartDocumentAnalysis:
       FeatureTypes = ["TABLES", "FORMS"]
       → Extracts structured tables (question number, question text, response field)
       → Extracts form key-value pairs (DDQ section headers, instructions)

  2. GetDocumentAnalysis (polling via MWAA S3Sensor):
       → Returns BLOCK objects: PAGE, TABLE, CELL, WORD, LINE, KEY_VALUE_SET
       → CONFIDENCE score per extracted element (flag <0.80 for human review)

  3. Post-processing Lambda:
       → Reconstructs nested table structures from BLOCK relationships
       → Maps question numbers to section categories
         (COMPANY_INFO, FINANCIAL, COMPLIANCE, IT_SECURITY, OPERATIONAL, ESG)
       → Produces structured JSON:
         {
           "ddq_id": "DDQ-2026-CALPERS-001",
           "questions": [
             {
               "number": "Q4.2",
               "section": "IT_SECURITY",
               "text": "Describe your disaster recovery architecture and RTO/RPO targets",
               "sub_questions": ["Q4.2a", "Q4.2b"],
               "textract_confidence": 0.94
             }
           ]
         }
```

### OpenSearch Knowledge Base Architecture

```
OPENSEARCH DDQ/RFP CORPUS

Index: napce-ddq-knowledge-base
  → ~50,000 approved DDQ question-answer pairs
  → 12 years of historical responses
  → Metadata: question_category, approval_date, approver, version,
    data_currency (static vs. requires_live_lookup)

Vector embeddings (Amazon Titan Embeddings):
  → Each approved answer indexed as 1,536-dimensional vector
  → Semantic similarity search at query time:
    "Describe your disaster recovery architecture"
    → Retrieves: "NAPCE DR Architecture: Multi-region active-passive,
      RTO < 2 hours, RPO < 15 minutes, tested quarterly..." (similarity 0.92)

Dynamic retrieval flag:
  → Answers tagged requires_live_lookup:
    → IT infrastructure questions: fetched from AWS infrastructure inventory
    → Compliance certifications: fetched from compliance system API
    → Staff counts / org structure: fetched from HR system
    → Financial stability metrics: fetched from finance system
  → Prevents stale content from reaching DDQ submissions
  → All live lookups are audit-logged with retrieval timestamp

Index: napce-gips-rules
  → GIPS 2019 Standard provisions (machine-readable)
  → Cross-referenced against SageMaker composite output
  → Updated when CFA Institute publishes guidance clarifications
```

### AWS Glue ETL — Knowledge Base Refresh

```python
# AWS Glue job: refresh OpenSearch knowledge base from Snowflake
# Runs nightly via MWAA schedule
import sys
from awsglue.context import GlueContext
from pyspark.context import SparkContext
from opensearchpy import OpenSearch

glue_context = GlueContext(SparkContext.getOrCreate())

# Read approved DDQ responses from Snowflake
approved_responses = glue_context.create_dynamic_frame.from_catalog(
    database="nomura_prod",
    table_name="ddq_approved_responses",
    transformation_ctx="approved_responses"
)

# Filter only current, compliance-approved entries
current_responses = approved_responses.filter(
    f=lambda x: x["approval_status"] == "APPROVED"
    and x["expiry_date"] > "2026-01-01"
)

# Upsert into OpenSearch knowledge base
# (Deleted/expired answers are automatically removed)
opensearch_client.bulk(
    index="napce-ddq-knowledge-base",
    body=format_bulk_upsert(current_responses.toDF().collect())
)
```

---

## 6. Saga Distributed Transaction Workflow

### Why Saga Is Required for NAPCE

A single NAPCE RFP proposal touches five systems: Amazon Batch (Monte Carlo simulation), Amazon S3 (intermediate results), Snowflake (data warehouse reads), Salesforce (CRM pipeline update), and OpenSearch (knowledge base queries). In a standard distributed system, a failure at Step 3 leaves orphaned data in S3 and a corrupted Salesforce pipeline stage — presenting a half-populated proposal to the institutional client or (worse) triggering a GIPS composite read from stale intermediate data.

The Saga pattern replaces a single distributed transaction (which would require two-phase commit across five systems) with a compensating transaction chain: each step either completes successfully or triggers a targeted rollback of only the steps that already ran.

### The 6-Step Saga Sequence

```
SAGA EXECUTION — NAPCE RFP PROPOSAL

              ┌──────────────────────────────────────────────────────┐
              │  INITIATION TRIGGER                                  │
              │  New DDQ/RFP uploaded via client portal             │
              │  → MWAA DAG starts: napce_rfp_proposal             │
              └─────────────────────┬────────────────────────────────┘
                                    │
              ┌─────────────────────▼────────────────────────────────┐
              │  STEP 1 — AGENT ANALYSIS                            │
              │  Bedrock RFP Agent parses request context           │
              │  Defines: 100 Monte Carlo scenarios, strategy X,   │
              │  GIPS composite IDs, regulatory jurisdiction         │
              │  On success: writes parameters to S3 staging        │
              │  On failure: → Compensation A (clean S3 staging)    │
              └─────────────────────┬────────────────────────────────┘
                                    │
              ┌─────────────────────▼────────────────────────────────┐
              │  STEP 2 — TASK EXECUTION (BATCH)                    │
              │  MWAA triggers Amazon Batch job                     │
              │  → 100 Monte Carlo risk simulations                 │
              │  Stress scenarios: rate shock, credit spread,       │
              │  equity drawdown                                     │
              │  On success: batch job ID written to S3             │
              │  On failure: → Compensation B (cancel Batch job,    │
              │               clean S3 staging)                     │
              └─────────────────────┬────────────────────────────────┘
                                    │
              ┌─────────────────────▼────────────────────────────────┐
              │  STEP 3 — DATA PROCESSING (S3 WRITE)                │
              │  Monte Carlo results extracted → written to         │
              │  S3: s3://nomura-napce-proposals/{rfp_id}/          │
              │  monte_carlo_results.parquet                        │
              │  On success: S3 key registered in DynamoDB state    │
              │  On failure: → Compensation C (delete S3 partial    │
              │               writes using DynamoDB manifest)       │
              └─────────────────────┬────────────────────────────────┘
                                    │
              ┌─────────────────────▼────────────────────────────────┐
              │  STEP 4 — GIPS VALIDATION                           │
              │  Compliance Agent reads composite data              │
              │  SageMaker validates GIPS rule compliance           │
              │  Composite certification written to proposal        │
              │  On success: GIPS cert status: PASS                 │
              │  On failure: → Alert to Compliance team,            │
              │               proposal blocked from submission,     │
              │               Compensation D (revert GIPS state)    │
              └─────────────────────┬────────────────────────────────┘
                                    │
              ┌─────────────────────▼────────────────────────────────┐
              │  STEP 5 — AGENT SUMMARY + ASSEMBLY                  │
              │  Bedrock RFP Agent assembles final proposal:        │
              │  → Monte Carlo outputs (Batch results)             │
              │  → Strategy narrative (OpenSearch KB)              │
              │  → GIPS composite performance (SageMaker)          │
              │  → Compliance certification (Step 4)               │
              │  Proposal package written to S3 final location     │
              └─────────────────────┬────────────────────────────────┘
                                    │
              ┌─────────────────────▼────────────────────────────────┐
              │  STEP 6 — FINALIZATION + ALERT                      │
              │  Salesforce opportunity updated:                    │
              │   → Stage: "Proposal Draft Ready for Review"        │
              │   → Proposal S3 link attached to opportunity        │
              │  Institutional sales RM alerted via Salesforce      │
              │  notification + email                               │
              │  Saga COMMITTED: all steps complete                 │
              └──────────────────────────────────────────────────────┘

COMPENSATION CHAIN (failure path):
  If Step N fails → execute Compensating Transactions N, N-1, ... 1
  in reverse order, targeting only the systems that were written to.

  Compensation A: DELETE s3://napce-proposals/{rfp_id}/staging/*
  Compensation B: BatchClient.cancel_job(batch_job_id) +
                  Compensation A
  Compensation C: S3Client.delete_objects(keys=DynamoDB.get_manifest(rfp_id)) +
                  Compensation B
  Compensation D: SageMaker rollback composite state, alert Compliance +
                  Compensation C
```

### Saga Compensation Pseudocode

```python
# NAPCE Saga compensation Lambda
import boto3
import json

def lambda_handler(event, context):
    rfp_id = event['rfp_id']
    failed_step = event['failed_step']
    s3 = boto3.client('s3')
    batch = boto3.client('batch')
    dynamodb = boto3.resource('dynamodb')

    compensation_log = []

    # Compensate in reverse from failed_step
    if failed_step >= 3:
        # Step 3 compensation: delete all S3 partial writes
        manifest_table = dynamodb.Table('napce-saga-manifests')
        manifest = manifest_table.get_item(Key={'rfp_id': rfp_id})
        s3_keys = manifest['Item'].get('s3_keys_written', [])
        if s3_keys:
            s3.delete_objects(
                Bucket='nomura-napce-proposals',
                Delete={'Objects': [{'Key': k} for k in s3_keys]}
            )
            compensation_log.append(f"COMPENSATED: deleted {len(s3_keys)} S3 objects")

    if failed_step >= 2:
        # Step 2 compensation: cancel Batch job if still running
        batch_job_id = event.get('batch_job_id')
        if batch_job_id:
            status = batch.describe_jobs(jobs=[batch_job_id])['jobs'][0]['status']
            if status in ['SUBMITTED', 'PENDING', 'RUNNABLE', 'STARTING', 'RUNNING']:
                batch.cancel_job(
                    jobId=batch_job_id,
                    reason=f'NAPCE Saga compensation: Step {failed_step} failure'
                )
                compensation_log.append(f"COMPENSATED: cancelled Batch job {batch_job_id}")

    if failed_step >= 1:
        # Step 1 compensation: clean S3 staging area
        s3.delete_objects(
            Bucket='nomura-napce-staging',
            Delete={'Objects': [{'Key': f'rfp-params/{rfp_id}/'}]}
        )
        compensation_log.append("COMPENSATED: cleaned S3 staging")

    # Update Salesforce: reset opportunity stage, alert sales team
    # (via EventBridge → Salesforce outbound messaging)

    return {
        'statusCode': 200,
        'rfp_id': rfp_id,
        'compensation_actions': compensation_log,
        'saga_status': f'COMPENSATED_FROM_STEP_{failed_step}'
    }
```

---

## 7. MWAA DAG Architecture

### DAG Trigger Specifications

| DAG Name | Trigger | Frequency | Saga Scope |
|---|---|---|---|
| `napce_ddq_processing` | S3 event: new DDQ uploaded to `s3://nomura-napce-input/ddq/` | Event-driven | Textract → Bedrock DDQ Agent → OpenSearch → S3 output |
| `napce_rfp_proposal` | Salesforce webhook: opportunity stage = "Proposal Stage" | Event-driven | Bedrock RFP Agent → Batch → S3 → SageMaker → Salesforce update |
| `napce_gips_monitoring` | EventBridge schedule: daily 06:00 UTC + portfolio change events | Daily + event | EventBridge → Lambda → Bedrock Compliance → SageMaker → Alert |
| `napce_knowledge_refresh` | Scheduled: nightly 02:00 UTC | Daily | Glue ETL: Snowflake → OpenSearch knowledge base refresh |

### Monte Carlo Batch Integration

```
AMAZON BATCH — NAPCE MONTE CARLO JOB DEFINITION

Job Definition: napce-montecarlo-rfp
  type: container
  containerProperties:
    image: nomura-ecr/napce-montecarlo:latest
    vcpus: 4
    memory: 8192  # 8GB per simulation job
    jobRoleArn: arn:aws:iam::nomura::role/NapceBatchJobRole

Environment variables per job:
  RFP_ID: {rfp_id}
  NUM_SIMULATIONS: 100
  STRATEGY_ID: {strategy_composite_id}
  S3_OUTPUT_BUCKET: nomura-napce-proposals
  S3_OUTPUT_PREFIX: {rfp_id}/monte-carlo/

Simulation parameters:
  → Base case: current portfolio exposures (from Snowflake gold layer)
  → Stress scenarios:
       1. Rate shock +200bps (fixed income duration impact)
       2. Rate shock -200bps (refinancing benefit analysis)
       3. Credit spread widening +150bps (corporate bond portfolio stress)
       4. Equity drawdown -30% (equity strategy max drawdown)
       5. Combined scenario: rate shock +100bps + spread +75bps (correlation test)
  → 100 Monte Carlo paths per scenario (500 total simulation runs)
  → Output: portfolio P&L distribution, VaR at 95th/99th percentile,
    max drawdown, Sharpe ratio under stress

MWAA BatchOperator invocation:
  BatchOperator(
      task_id='run_monte_carlo_simulations',
      job_name='napce-montecarlo-{rfp_id}',
      job_definition='napce-montecarlo-rfp',
      job_queue='napce-compute-queue',
      overrides={
          'environment': [
              {'name': 'RFP_ID', 'value': '{{ dag_run.conf["rfp_id"] }}'},
              {'name': 'STRATEGY_ID', 'value': '{{ dag_run.conf["strategy_id"] }}'}
          ]
      }
  )
```

### S3 Pipeline Layout

```
nomura-napce-proposals/
  {rfp_id}/
    ├── input/
    │   └── original_rfp.pdf          # Uploaded by prospect
    ├── textract/
    │   └── structured_questions.json  # Textract DocumentAnalysis output
    ├── staging/
    │   └── agent_parameters.json      # Saga Step 1: Bedrock agent params
    ├── monte-carlo/
    │   └── simulation_results.parquet # Saga Step 2-3: Batch outputs
    ├── gips/
    │   └── composite_certification.json # Saga Step 4: SageMaker GIPS
    ├── draft/
    │   └── rfp_proposal_draft.pdf     # Saga Step 5: assembled proposal
    └── final/
        └── rfp_proposal_approved.pdf  # Human-reviewed, submission-ready

nomura-napce-staging/
  ddq-processing/
    {ddq_id}/
      ├── textract-output/
      └── agent-confidence-scores/

nomura-napce-audit/                    # Immutable audit log (S3 Object Lock)
  {year}/{month}/{day}/
    ├── agent-invocations.jsonl
    ├── saga-manifests.jsonl
    └── gips-monitoring-results.jsonl
```

---

## 8. Implementation Roadmap

### Phase 1 — Foundation: Knowledge Base + Document Intelligence (Months 1–3)

**Objective:** Build the data infrastructure that all NAPCE agents depend on.

| Initiative | Owner | Deliverable |
|---|---|---|
| Amazon Textract pipeline: DDQ PDF processing | Head of Client Technology + AWS SA | Textract DocumentAnalysis producing structured JSON from any DDQ format with >95% extraction accuracy |
| OpenSearch knowledge base seeding | Compliance Team + Technology | 50,000+ approved DDQ responses ingested, vectorized with Titan Embeddings, section-tagged |
| AWS Glue ETL: Snowflake → OpenSearch | Data Engineering | Nightly refresh job; approved-answer corpus always current |
| S3 bucket structure + IAM policies | Platform Team | nomura-napce-* bucket hierarchy; partition-keyed by rfp_id; Lake Formation RLS |
| GIPS rules index | Compliance Agent + SageMaker | GIPS 2019 Standard rules machine-readable in OpenSearch; SageMaker GIPS calculation model validated |
| MWAA environment provisioned | Platform Team | Airflow 2.7 on MWAA; Python 3.11; DAG repository connected to Git |

**Gate:** Textract successfully parses 10 real institutional DDQ PDFs with >95% question extraction accuracy. OpenSearch semantic search returns correct approved answer for 90% of test queries.

---

### Phase 2 — DDQ Agent Launch (Months 3–6)

**Objective:** Production DDQ Auto-Population for institutional prospects.

| Initiative | Owner | Deliverable |
|---|---|---|
| Bedrock DDQ Agent: Claude 3.5 Sonnet | AI Team | Agent with Bedrock Guardrails (no hallucinated compliance claims); IAM-scoped data access |
| MWAA DAG: napce_ddq_processing | Platform Team | End-to-end DAG with Saga compensation; tested against 5 live DDQ scenarios |
| Confidence-score routing | AI Team + Compliance | Human review queue for <0.85 confidence; SLA tracking per question category |
| Dynamic live-data retrieval | Integration Team | Technology section questions pull live from AWS infrastructure inventory + compliance APIs |
| Audit logging | Security Team | CloudTrail + S3 Object Lock immutable log for all DDQ agent invocations |

**Gate:** First live institutional DDQ completed in <3 days (vs. 3–6 week baseline). Compliance team reviews output and certifies zero factual errors in technology control sections.

---

### Phase 3 — GIPS Compliance Monitoring + Continuous Validation (Months 6–9)

**Objective:** Zero-defect GIPS compliance posture; eliminate quarterly spot-check exposure.

| Initiative | Owner | Deliverable |
|---|---|---|
| EventBridge + Lambda monitoring chain | Platform Team | Daily GIPS monitoring triggers; portfolio change events from Aladdin CDC |
| Bedrock Compliance Agent | AI Team + Performance Team | Agent monitors composite assignments, benchmark disclosures, terminated portfolio retention |
| SageMaker GIPS Engine | Quantitative Team | Deterministic GIPS calculation model; AI never touches return calculations |
| Alert + escalation routing | Compliance + Technology | Salesforce case creation for violations; immediate notification to Head of Client Technology |
| GIPS audit package automation | Compliance Team | Third-party GIPS verifier receives automated audit package from S3 immutable log |

**Gate:** GIPS Compliance Agent successfully detects 3 deliberately introduced composite assignment errors in UAT. Third-party GIPS verifier confirms automated audit package meets verification requirements.

---

### Phase 4 — RFP Agent + Monte Carlo Integration (Months 9–12)

**Objective:** Full institutional RFP proposal automation with quantitative risk analysis.

| Initiative | Owner | Deliverable |
|---|---|---|
| Bedrock RFP Agent | AI Team | Full proposal assembly: strategy narrative + GIPS performance + Monte Carlo outputs |
| Amazon Batch + Monte Carlo jobs | Quantitative Team | 100-scenario simulation per proposal; 5 stress scenarios; S3 output |
| MWAA DAG: napce_rfp_proposal | Platform Team | Full Saga workflow with 6 steps and compensation chain |
| Salesforce integration | Sales Technology | Opportunity stage webhook triggers DAG; proposal S3 link attached to opportunity |
| Cross-document exchange | Integration Team | NAPCE → NAIM: compliance data exchange; NAPCE → NCIE: mandate signals for feedback |

**Gate:** First live institutional RFP proposal prepared with NAPCE in <8 hours (vs. 2–5 day baseline). Monte Carlo simulations complete without Saga compensation being triggered. Sales team certifies proposal quality meets client submission standards.

---

### Phase 5 — Production Maturity + Scale (Months 12+)

**Objective:** Platform leverage — NAPCE capability feeding the full Uninterrupted Journeys Platform.

| Initiative | Owner | Deliverable |
|---|---|---|
| Multi-jurisdiction DDQ support | Head of Client Technology | SEC, FCA, ASIC jurisdiction-specific DDQ answer libraries |
| RFP win-rate feedback loop | AI Team | Proposal outcomes (won/lost) feed back to OpenSearch ranking model |
| NAPCE → AI Chatbot content surface | AI Team | Internal support agents use NAPCE GIPS certification data to answer client queries |
| FinOps review | Platform Team | MWAA + Batch + OpenSearch cost optimization; target <$5K/month per 100 proposals |
| Shaw-layer integration | Platform Team | NAPCE as DAG within broader data mesh: ADX zero-copy for portfolio data |

---

## 9. Lemons Tables

> Proactive risk identification — the "Lemon Detector" framework applied to NAPCE. Three risk tables covering the three highest-exposure areas: AI/Agentic Risk, Compliance & Regulatory Risk, and Data Governance & Integration Risk.

---

### Table A: AI & Agentic Workflow Risk

| # | Risk (The Lemon) | Impact | Probability | Mitigation | Owner |
|---|---|---|---|---|---|
| A1 | **Complex Table & PDF Extraction Failure** — Textract misreads multi-column nested DDQ table structures; questions split across columns are merged incorrectly, producing nonsensical question text that the DDQ Agent cannot match to knowledge base entries. | DDQ draft contains unanswered or incorrectly answered sections; compliance failure discovered at submission review. | Medium-High — institutional DDQ PDFs vary widely in format quality. | Amazon Textract DocumentAnalysis with TABLES + FORMS feature types; confidence score <0.80 flags for human review with original PDF section; structured JSON validation schema with type-check post-processing. | Head of Client Technology + AI Team |
| A2 | **Stale or Unapproved Knowledge Base Content** — OpenSearch knowledge base entry is approved but references outdated technology vendor, control, or regulatory position. DDQ Agent serves answer that was accurate 18 months ago but no longer reflects current firm state. | Client receives factually incorrect information about Nomura's operational controls; potential regulatory violation if answer pertains to AML/compliance controls. | Medium — knowledge base requires active governance to expire outdated answers. | All technology-section answers tagged `requires_live_lookup`; DDQ Agent dynamically retrieves from live operational APIs, never from cached knowledge base for time-sensitive control descriptions; AWS Glue nightly refresh with expiry-date enforcement on all entries. | Compliance Team + Technology |
| A3 | **LLM Hallucination on GIPS Calculations** — Bedrock Compliance Agent generates a GIPS return figure that appears reasonable but is mathematically incorrect; Saga does not detect this because the error is in the AI layer, not in the infrastructure layer. | Incorrect performance data reaches institutional client in RFP; GIPS compliance claim is false; reputational and regulatory violation. | Low — explicitly mitigated by design — but catastrophic if it occurs. | **Architecture boundary:** Bedrock agent NEVER calculates returns. SageMaker model calculates all returns deterministically. Bedrock agent monitors outputs for rule compliance only. Bedrock Guardrails configured to block any attempt by the agent to generate numerical performance outputs without SageMaker source reference. | Quantitative Team + AI Team |
| A4 | **MWAA DAG Failure Leaving Orphaned State** — Saga compensation is triggered but fails partway through (e.g., S3 delete fails due to permissions); S3 intermediate data remains; next DAG invocation reads stale partial results from previous failed run. | Corrupted Monte Carlo data used in proposal; incorrect risk analysis presented to institutional client. | Low-Medium — compensation chain is a secondary failure path. | DynamoDB saga manifest tracks all S3 keys written per rfp_id; compensation Lambda reads manifest and deletes precisely; S3 Object Lock prevents accidental overwrites; MWAA retry with exponential backoff on compensation tasks; dead-letter SQS queue for compensation failures requiring manual intervention. | Platform Team |
| A5 | **IAM Role Boundary Violation** — Service misconfiguration allows DDQ Agent to access performance data for strategies beyond the scope of the requesting RM's Salesforce account hierarchy; sensitive composite data exposed. | Fund-level data accessed without authorization; privacy breach; regulatory violation. | Low — IAM is well-understood technology, but misconfiguration risk exists. | Bedrock Agent assumes least-privilege IAM role per invocation; IAM boundary policies enforced at account level (not just application level); quarterly IAM permission boundary audit; CloudTrail alerts on anomalous Bedrock invocations accessing cross-account S3 prefixes; penetration test of identity mesh prior to Phase 2 launch. | Security Team |

---

### Table B: Compliance & Regulatory Risk

| # | Risk (The Lemon) | Impact | Probability | Mitigation | Owner |
|---|---|---|---|---|---|
| B1 | **GIPS Composite Assignment Drift** — Portfolio is added to a new strategy mid-quarter; GIPS composite assignment logic is not updated; new portfolio's returns are excluded from composite track record; GIPS composite performance is understated during institutional due diligence. | RFP/DDQ composite performance is inaccurate; institutional client makes allocation decision on incorrect track record; regulatory exposure under SEC Investment Adviser Act. | Medium — composite management requires continuous discipline, easy to drift in fast-growing strategy. | Bedrock Compliance Agent monitors composite assignments daily (not quarterly); Aladdin CDC events trigger re-validation whenever portfolio is created or strategy is modified; SageMaker GIPS Engine runs completeness check: all discretionary portfolios must be assigned to at least one composite; immediate Salesforce case created for any unassigned portfolio. | Head of Client Technology + Compliance + Performance Team |
| B2 | **Terminated Portfolio Exclusion Violation** — GIPS requires terminated portfolios to remain in composite historical returns for ≥5 years; portfolio is removed from active management system after termination; downstream reporting excludes it; composite historical returns are restated. | GIPS composite track record is falsified by exclusion; if discovered during third-party GIPS verification, firm must issue correction to all institutional clients who received the composite; reputational damage. | Low-Medium — once a portfolio is terminated, operations teams may not track the 5-year retention rule. | GIPS Engine maintains separate `terminated_portfolios` table with composite assignment history; Compliance Agent verifies terminated portfolios appear in historical composite returns for required retention period; S3 immutable storage for all composite history (CloudTrail-compliant); automated alert at 4-year 9-month mark for each terminated portfolio (allows proactive verification). | Performance Team + Compliance |
| B3 | **SEC Rule 206(4)-1 Performance Advertising Violation** — RFP Agent includes composite returns in proposal without required disclosures (e.g., gross vs. net return note, benchmark description, composite inception date); technically non-compliant performance advertising. | SEC enforcement action; required disclosure to all institutional clients who received the non-compliant document; reputational damage with potential fund redemptions. | Medium — disclosure requirements are precise and easy to omit when assembling large proposals under time pressure. | Bedrock Guardrails configured with mandatory disclosure templates: any proposal containing composite return figures must include: gross/net notation, benchmark full description, composite inception date, required footnote language; Compliance Agent validates every proposal output against disclosure checklist before Saga Step 5; proposal blocked if disclosure missing. | Compliance + AI Team |
| B4 | **DDQ Answer Inconsistency Across Simultaneous Prospects** — Two institutional prospects send DDQs simultaneously; DDQ Agent populates different answers to identical questions because Glue ETL refresh occurs between the two invocations, updating the knowledge base. | Two prospects receive different answers to the same compliance question; inconsistency discovered if prospects compare notes (common in consultant-driven searches); integrity of firm's compliance narrative questioned. | Low-Medium — knowledge base updates and concurrent DDQ processing can collide. | DDQ answers snapshot taken at DAG start time; all questions for a single DDQ answered from the same knowledge base version (point-in-time read from OpenSearch); Glue ETL refresh happens during off-hours; DDQ DAG uses optimistic locking to prevent knowledge base update during active DDQ processing. | Platform Team + Compliance |
| B5 | **GDPR / Data Residency Violation** — European institutional prospect sends DDQ containing EU client names or fund-specific PII; Textract processes the full PDF including sensitive data; data transits to US-based SageMaker endpoint in violation of GDPR data residency requirements. | GDPR Article 46 violation; regulatory fine up to 4% of global turnover; institutional prospect withdraws from due diligence process. | Low-Medium — DDQ PDFs from EU prospects may contain sensitive personal data. | Textract processing for EU-origin DDQs routes to eu-west-1 region endpoint (London); Bedrock Agent invocations for EU jurisdiction in eu-west-1; IAM condition keys enforce region constraint (`aws:RequestedRegion`); PII fields in DDQ questions identified and redacted before agent processing (Comprehend PII entity detection); GDPR DPA signed with AWS for all services in scope. | Security Team + Data Privacy Officer |

---

### Table C: Data Governance & Integration Risk

| # | Risk (The Lemon) | Impact | Probability | Mitigation | Owner |
|---|---|---|---|---|---|
| C1 | **Monte Carlo Simulation Producing Deterministic-Looking Results** — Amazon Batch Monte Carlo job uses incorrectly seeded random number generator; 100 simulations produce nearly identical outputs; proposal presents artificially narrow confidence interval; institutional client's risk committee does not see true tail risk. | Institutional mandate awarded on incomplete risk disclosure; performance under stress materially worse than shown; client redeems; legal exposure. | Low — but requires explicit validation in quantitative model review. | Quantitative model validation test: run 1,000 simulations and verify output distribution meets statistical properties (coefficient of variation >0.15 per scenario); independent review of random seed generation (hardware entropy source); Batch job includes simulation diversity metric in output metadata; MWAA Saga Step 3 validates distribution statistics before writing to S3 final location; rejects batch output if distribution metric fails. | Quantitative Team |
| C2 | **Snowflake Data Lag — Stale Portfolio Data in Proposals** — MWAA RFP DAG triggers before nightly Snowflake → S3 data sync; proposal Monte Carlo simulations use previous day's portfolio exposures; institutional client receives portfolio risk analysis based on stale positions. | Risk analysis presented to prospect is up to 24 hours out of date; during volatile market periods, this could materially misrepresent portfolio risk. | Medium-High — data sync timing and DAG scheduling are not co-ordinated by default. | DAG dependency: `napce_rfp_proposal` checks Snowflake data freshness timestamp before invoking Bedrock Agent; S3KeySensor waits for today's Snowflake sync completion marker before proceeding; if intraday proposal request, DAG triggers targeted Snowflake → S3 refresh for the relevant strategy composite only; data staleness > 4 hours blocks proposal generation and alerts Data Engineering. | Data Engineering + Platform Team |
| C3 | **OpenSearch Knowledge Base Indexing Failure** — AWS Glue ETL job completes but OpenSearch bulk upsert partially fails; some newly approved DDQ answers are indexed, but older conflicting answers are NOT removed; DDQ Agent retrieves both versions and returns incorrect answer from deprecated entry. | DDQ response contains compliance-deprecated answer; submission to institutional prospect; potential compliance violation. | Low-Medium — partial index operations are a known risk in distributed search systems. | Glue ETL job uses write-and-swap pattern: writes to new index version, validates completeness (count match against Snowflake source), then atomically switches OpenSearch alias to new version; old index retained for 48 hours as rollback; MWAA monitors index swap success and alerts on partial completion; DDQ Agent queries current alias only, never previous index versions. | Data Engineering |
| C4 | **Salesforce API Rate Limit Under Proposal Volume** — NAPCE reaches high institutional prospect pipeline volume; 50+ concurrent RFP DAG instances trigger simultaneous Salesforce API calls to update opportunity stages; Salesforce API governor limit hit; DAG fails at Step 6 after proposal is complete. | Completed proposal is in S3 but Salesforce is not updated; RM has no notification; proposal sits pending; client SLA missed. | Low-Medium — Salesforce governor limits are enforced at org level. | Salesforce API calls in MWAA use a dedicated integration user with its own API limit budget; Bulk API used for opportunity updates (5,000 records/batch vs. REST 100/call limit); MWAA DAG implements exponential backoff + retry on Salesforce API calls; if Step 6 fails after 3 retries, separate recovery DAG polls S3 final location and attempts Salesforce update independently; success notification sent to RM via email as fallback. | Integration Team |
| C5 | **Cross-Document Data Exchange Inconsistency** — NAPCE exchanges compliance data with NAIM (Support Agent Co-Pilot) and client signals with NCIE (Client Insight Engine); schema changes in NAPCE output format break downstream consumers without coordination. | NAIM receives malformed compliance data; support agents see incorrect information about GIPS status; support quality degrades for institutional clients with active mandates. | Low-Medium — schema drift is a common distributed system risk across multiple autonomous services. | OpenSearch-mediated data exchange uses schema registry (AWS Glue Schema Registry); all NAPCE output schemas versioned with semantic versioning; breaking changes require consumer notification 2 sprints ahead; NAPCE publishes to NAIM/NCIE via EventBridge event bus with schema version in event header; consumers validate schema version before processing; mismatched schema version triggers dead-letter routing rather than silent data corruption. | Integration Team + Platform Team |

---

## 10. Panel Talking Points

### For Kathleen Hess McNamara (ED, Client Experience & AI)

**Opening — extend her DDQ/RFP work at Macquarie:**
> *"Kathleen, you operationalized the first AI DDQ/RFP agent at Macquarie Asset Management — you know the governance challenge more than most. NAPCE is designed on the same foundation: the agent is only as trustworthy as the knowledge base it draws from. The two things I'd add to what you built are: (1) Amazon Textract as the document intelligence pre-processing layer, so structured DDQ tables don't defeat the agent before it even starts; and (2) the live-lookup flag — technology control questions that need to reflect this week's infrastructure state, not last year's approved template. The content governance model stays exactly as you designed it. The agent stays grounded."*

**On GIPS Compliance Agent:**
> *"The GIPS monitoring architecture is designed around a critical separation of concerns: Bedrock does not calculate returns. SageMaker does. That boundary exists because GIPS compliance requires mathematical determinism — an LLM cannot meet that standard. What Bedrock does is monitor SageMaker's outputs against GIPS rule requirements and flag composite assignment drift, terminated portfolio violations, and disclosure gaps. Continuous monitoring instead of quarterly spot-checks. Your third-party GIPS verifier receives an automated audit package from S3, not a manually compiled spreadsheet."*

**On the 12–24 month mandate pipeline:**
> *"The institutional mandate cycle is fundamentally constrained by manual assembly time at every stage. NAPCE compresses Stage 4 (RFP/DDQ response) from weeks to hours, and Stage 5 (Proposal) from days to hours. That does not shorten relationship-building time between the RM and the prospect — it recovers time that analysts were spending on assembly and routing. The RM can spend that time with the client instead."*

**Anticipated Kathleen question:** *"How do we ensure the DDQ agent doesn't hallucinate a compliance answer?"*

**Response:**
> *"Three controls. First, Bedrock Guardrails are configured to prevent the agent from generating compliance claims without a direct knowledge base citation — if the agent cannot point to an approved entry, it routes to human review, it does not generate. Second, the confidence threshold of 0.85 is tuned from analyzing our historical DDQ response corpus — below 0.85, the semantic similarity to an approved entry is low enough that substituting the wrong answer is a real risk. Third, all compliance and technology section answers tagged 'time-sensitive' always trigger live data retrieval — the IT/security answers are pulled from AWS CloudTrail, our compliance system API, and infrastructure inventory at the moment the question is answered, not from a cached template."*

---

### For Stuart Mumley (Director, Client Platforms)

**Opening — Salesforce as Saga participant:**
> *"Stuart, the institutional mandate pipeline you've built in Salesforce is exactly the coordination layer NAPCE needs as the final step of every Saga. When a proposal is complete — Monte Carlo simulations done, GIPS certified, Bedrock assembly finished — the MWAA DAG's last action is updating the Salesforce opportunity stage and attaching the proposal S3 link. The RM gets a notification inside Salesforce. No separate RM workflow to manage. The architecture respects what you've built: Salesforce owns the pipeline, MWAA feeds it."*

**On platform integration boundaries:**
> *"NAPCE sits in the Distribution Engine layer — it automates the work that happens between prospect identification and mandate award. The integration surfaces are: Salesforce webhook triggers DAG start; Salesforce opportunity receives DAG completion; Snowflake provides portfolio exposure data for Monte Carlo; OpenSearch provides approved DDQ/RFP answers. NAPCE never owns pipeline state — Salesforce does. NAPCE never owns portfolio data — the data layer does. Clean domain boundaries."*

**On MWAA vs. Step Functions for orchestration:**
> *"MWAA over Step Functions for this use case because the DAGs have complex branching (confidence routing, Saga compensation chains, multi-step failure paths) and the team already operates Airflow patterns for data engineering. The operational familiarity matters for a system this business-critical. Step Functions JSON state machines become harder to reason about at the complexity level NAPCE requires. MWAA DAGs are Python — the team can debug them in the same environment as everything else."*

**Anticipated Stuart question:** *"What's the FinOps profile of running MWAA + Batch at RFP volume?"*

**Response:**
> *"Rough estimate at 100 proposals/month: MWAA environment = ~$800/month (multi-AZ Airflow 2.7, m5.xlarge workers); Amazon Batch per RFP = ~$2.50 (4 vCPU × 8GB × ~30 min simulation time × 100 scenarios, spot pricing); OpenSearch domain = ~$400/month (2 × r6g.large.search); Textract = ~$1 per 1,000 pages. Total: under $3K/month for 100 proposals. Compare to the analyst team hours at 3–6 weeks per DDQ — NAPCE pays for itself on the first institutional mandate it helps win."*

---

### For Shaw Levin (VP, Technology Partnerships)

**Opening — serverless-first, FinOps-tuned:**
> *"Shaw, NAPCE is built serverless-first by design: Lambda for service integrations, MWAA workers that scale to zero between DAG runs, Amazon Batch for compute-heavy simulations (spot pricing, no idle EC2), OpenSearch with auto-scaling. The only persistently running infrastructure is the MWAA environment and the OpenSearch domain — both justified by constant load. Everything event-driven. No provisioned capacity sitting idle between proposal cycles."*

**On the Saga pattern as FinOps:**
> *"The Saga pattern is as much a FinOps decision as an engineering decision. If a proposal pipeline fails partway through and we can't roll back S3 intermediate state, we accumulate storage for orphaned partial proposals. The DynamoDB saga manifest that tracks every S3 key written means the compensation Lambda cleans up precisely — no orphaned objects, no incremental storage cost from failed runs. At 1,000 proposals a year, this is meaningful."*

**On ADX integration — NAPCE as data consumer:**
> *"NAPCE is a consumer of the ADX zero-copy data mesh you built. Portfolio exposure data from Snowflake enters the Monte Carlo simulation via the gold layer of the medallion architecture. NAPCE DAGs don't own data — they orchestrate agents who query governed data products from the platform. The IAM + Lake Formation entitlement model you established for the data mesh is exactly what NAPCE's identity mesh is built on top of. One investment in data governance, multiple agentic surfaces consuming it safely."*

**Anticipated Shaw question:** *"How does NAPCE handle a simultaneous flood of DDQs — e.g., institutional sales push during quarter-end?"*

**Response:**
> *"MWAA workers auto-scale based on queued DAG run count — up to 10 concurrent DDQ processing DAGs without manual intervention. Amazon Batch uses a compute environment with maxvCpus configured for peak load; Monte Carlo jobs queue rather than fail. OpenSearch auto-scaling handles query volume spikes. The only bottleneck is the Bedrock API — which we address by tracking token consumption per DAG run and implementing a semaphore on concurrent Bedrock agent invocations. If we hit the Bedrock service quota during a spike, the MWAA sensor waits and retries; the Saga compensation only triggers on hard failures, not on throttling. The client SLA buffer is built into the DAG: we promise 48-hour DDQ turnaround, the median execution time is 4 hours, giving us 44 hours of headroom."*

---

### Executive Clincher — NAPCE as the Fourth Surface of the Uninterrupted Journeys Platform

> *"When we talk about the Nomura Uninterrupted Journeys Platform, we mean a client experience with no friction across any surface they interact with. NCIE (the Client Insight Engine) ensures feedback never gets lost. NAIM (the Support Co-Pilot) ensures internal support agents have the right context in real time. The AI Chatbot ensures the RM has the right proposal content in 60 seconds instead of 30 minutes. NAPCE is the fourth surface — it ensures the institutional mandate journey, from the first DDQ received to the proposal submitted to the mandate awarded, happens at the speed of a technology firm, not at the pace of spreadsheets and email threads.*
>
> *The Saga pattern guarantees that journey never corrupts. The MWAA central nervous system guarantees it's always orchestrated. The Bedrock agents guarantee it's always grounded in approved content. And the SageMaker GIPS Engine guarantees that when we tell an institutional investor our performance track record is GIPS compliant, that claim is verified in real time — not quarterly.*
>
> *What Kathleen built at Macquarie was the first surface. NAPCE builds the second surface on top of that foundation: all four agents consuming the same content governance layer, all four Saga chains rolling back cleanly when something breaks, all four DAGs feeding one Salesforce pipeline. The platform leverage is the point — not the individual capability."*

---

## Document Metadata

| Field | Value |
|---|---|
| **Prepared by** | Calvin Lee |
| **Target audience** | Kathleen Hess McNamara, Stuart Mumley, Shaw Levin (all three panel members) |
| **Role** | ED, Head of Client Technology — Nomura Asset Management International |
| **Purpose** | Case Study 4 — Agentic Workflow Automation interview preparation |
| **Architecture** | Amazon MWAA + Saga Pattern + Bedrock Agents + SageMaker + Textract + Amazon Batch |
| **Evaluation** | [Eval Cycle 1](evaluation/EVALUATION_NAPCE_CYCLE_1.md) · [Eval Cycle 2](evaluation/EVALUATION_NAPCE_CYCLE_2.md) · [Eval Cycle 3 — Certified 9.90/10 ✅](evaluation/EVALUATION_NAPCE_CYCLE_3_FINAL_CERTIFICATION.md) |
| **BIAN Domains** | Sales & Distribution · Customer & Party Agreement · Investment Portfolio Mgmt · Sales Enablement & Advisor Distribution |
| **Cross-references** | [NAIM Support Agent Co-Pilot](SUPPORT_AGENT_COPILOT_NAIM.md) · [NCIE](NOMURA_CLIENT_INSIGHT_ENGINE.md) · [AI Chatbot + CRM](AI_CHATBOT_CRM_SALESCLOUD_INTEGRATION.md) · [BIAN Framework](BIAN_FRAMEWORK_ASSET_MANAGEMENT.md) |
| **Last updated** | February 2026 |
