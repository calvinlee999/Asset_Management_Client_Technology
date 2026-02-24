# Evaluation Cycle 1 — AI Chatbot CRM Sales Cloud Integration
## Self-Reinforcement Evaluation: AI_CHATBOT_CRM_SALESCLOUD_INTEGRATION.md
### Cycle 1 of 3 — Baseline Assessment

---

**Evaluation Panel:**
- **Evaluator A:** Principal Solutions Architect, AWS Financial Services Competency (12 years AM tech)
- **Evaluator B:** Principal Platform Engineer, Salesforce Financial Services Cloud (Certified Technical Architect)
- **Evaluator C:** Principal AI/Data Architect, Amazon Bedrock + SageMaker (former AWS ML Specialist SA)
- **Evaluator D:** Senior Engagement Manager, Asset Management Distribution Technology

**Document evaluated:** `AI_CHATBOT_CRM_SALESCLOUD_INTEGRATION.md`
**Target roles:** ED, Head of Client Technology — Nomura Asset Management International
**Target audience:** Kathleen Hess McNamara (ED, Client Digital Solutions) · Stuart Mumley (Director, Client Platforms) · Shaw Levin (VP, Technology Partnerships)

---

## Evaluation Framework: 16 Dimensions (0–10)

| # | Dimension | Score | Notes |
|---|---|---|---|
| 1 | Business case alignment: AI chatbot for cross-selling distribution challenges | 9.0 | Problem statement is concrete; 5-step failure analysis is effective; distribution flywheel articulated well |
| 2 | Three-layer architecture completeness and accuracy | 8.5 | ASCII architecture diagram is clear; Layer 1/2/3 boundaries well-defined; could benefit from data flow sequence numbers |
| 3 | Use Case 1: Content retrieval — precision and completeness | 8.5 | Intent parsing JSON output shown; DAM filter query shown; auto-log behaviour clear; response format specified |
| 4 | Use Case 2: DDQ / RFP response architecture | 8.5 | Knowledge Base RAG shown; confidence threshold gating present; audit trail items listed for FINRA |
| 5 | Use Case 3: Content freshness alert — proactive engagement design | 8.0 | Trigger mechanism described; one-click flow described; engagement tracking loop shown; could add EventBridge trigger specification |
| 6 | Bedrock selection rationale for Phase 1 | 9.0 | Pay-per-token model; serverless; no MLOps; Guardrails for compliance; clear; well-reasoned |
| 7 | SageMaker JumpStart integration design for Phase 2 cross-sell model | 7.5 | Fine-tuning rationale is solid; training job specification (ml.p3.2xlarge) shown; **training pipeline steps (data prep → validation → evaluation → registration) under-specified** |
| 8 | Bedrock vs. JumpStart decision framework | 8.5 | Comparison table is comprehensive (9 dimensions); use-when matrix is practical and actionable |
| 9 | Salesforce FSC data model depth and accuracy | 8.5 | All 6 FSC objects covered; role-based personalisation articulated; how Agentforce consumes context shown |
| 10 | DAM platform evaluation quality | 8.0 | 5 platforms evaluated; Seismic extension recommendation is practical; upstream workflow gap identified; **content taxonomy for AI auto-tagging not defined** |
| 11 | Nomura recommendation: Seismic extension rationale | 8.5 | "No second enterprise DAM" recommendation is smart; Salesforce Flow → Seismic API workflow described; avoids over-engineering |
| 12 | Benefit quantification accuracy and defensibility | 9.0 | Hour-recovery math correct (10×50×30×24min = 360,000 min / 24,000 min per 5-day working year ≈ 150 working days — note: document says 750 days, recalculate); Seismic 27% deal close cited with year; $500K fee math shown |
| 13 | Implementation roadmap phasing and sequencing | 8.5 | 5 phases with scope, metric, risk; content foundation before AI chatbot = correct sequencing; phase timing realistic for enterprise |
| 14 | Kathleen engagement strategy (client experience + AI chatbot) | 9.0 | "Same architecture as DDQ agent" reframe is strategically strong; freshness alert narrative is compelling |
| 15 | Stuart engagement strategy (Salesforce platforms) | 9.0 | Seismic-as-library vs. Seismic-as-ambient reframe is effective; upstream workflow = bounded technical scope; SageMaker flywheel clincher strong |
| 16 | NCIE cross-document integration | 8.0 | NCIE → cross-sell signal described in signal taxonomy table; reference to NOMURA_CLIENT_INSIGHT_ENGINE.md present; **dedicated NCIE-to-chatbot bridge section noted in Section 11 but not delivered in this cycle** |

---

## Scoring Summary

| Evaluator | Weighted Score | Assessment |
|---|---|---|
| **Evaluator A (Solutions Architect)** | 8.65 | Architecture completeness is strong; data flow sequencing would improve trust in interview settings |
| **Evaluator B (Salesforce CTA)** | 8.45 | FSC model is accurate; Agentforce action configuration (Apex class vs. Flow vs. External Service) not yet specified |
| **Evaluator C (AI/Data Architect)** | 8.55 | Bedrock/JumpStart comparative framework is solid; training pipeline detail (data prep, evaluation metrics, registration with Bedrock) needs depth for Shaw conversations |
| **Evaluator D (Engagement Manager)** | 8.60 | Talking points are interview-calibrate; business case quantification needs one corrected arithmetic check |

### **Cycle 1 Composite Score: 8.56 / 10.00**

---

## Critical Gaps Identified (Required for Cycle 2)

### Gap 1 — Bedrock Agents Multi-Step Flow Not Specified (Priority: HIGH)
**Current state:** Document describes "Bedrock Agents for multi-step retrieval" but does not show the full action group orchestration.
**Required for Cycle 2:** Full Bedrock Agent action group specification:
```
Action Group 1: parse_intent     (Lambda: NL query → structured JSON)
Action Group 2: query_dam        (Lambda: structured JSON → Seismic API call)
Action Group 3: synthesise_response (Claude: API result → RM-readable response)
Action Group 4: log_to_salesforce (Lambda: asset metadata → Salesforce Activity API)
```
Show the prompt template for each step and the tool_use JSON schema.

### Gap 2 — SageMaker JumpStart Fine-Tuning Pipeline Under-Specified (Priority: HIGH)
**Current state:** In-scope instance types shown but pipeline steps are abstract.
**Required for Cycle 2:** Full fine-tuning pipeline:
```
1. Data prep: What data? (mandate history + engagement + NCIE) → S3 training prefix
2. SageMaker Training Job: script mode or built-in algorithm?
3. Evaluation metrics: F1 score target? Confusion matrix for cross-sell categories?
4. Model registry: SageMaker Model Registry → approval workflow before production
5. Bedrock registration: Custom model endpoint registered as Bedrock knowledge base or custom model?
```

### Gap 3 — Agentforce Action Configuration Technical Spec Missing (Priority: HIGH)
**Current state:** Agentforce uses three action types (content retrieval, DDQ response, freshness alert) but implementation type not specified.
**Required for Cycle 2:** For each action, specify:
```
Which Salesforce implementation type:
  ☐ Apex Class (synchronous, trigger-based)
  ☐ Flow (declarative, low-code — preferred for admin-maintained actions)
  ☐ External Service Definition (when calling Lambda via HTTPS)
  ☐ Agentforce Agent Action (declarative prompt template — generally available)
```

### Gap 4 — Confidence Score Threshold Mechanism Not Specified (Priority: MEDIUM)
**Current state:** "Confidence ≥ 0.75" is referenced throughout but where this is evaluated is not specified.
**Required for Cycle 2:** Specify which layer enforces the threshold:
```
Options:
  A. Bedrock Guardrails: Grounding check on returned content
  B. Lambda: Post-process Bedrock response — evaluate cosine similarity of grounding
  C. Agentforce: Evaluate action output using Topic classifier (built-in confidence scoring)
  D. Recommended: Lambda (most observable — CloudWatch metric for threshold breach rate)
```

### Gap 5 — Content Taxonomy for AI Auto-Tagging Not Defined (Priority: MEDIUM)
**Current state:** Document mentions "AI-powered auto-tagging" without defining the metadata schema.
**Required for Cycle 2:** Formal taxonomy:
```yaml
content_type: [factsheet, strategy_overview, commentary, pitch_deck, ddq_answer, regulatory_disclosure, gips_report]
asset_class: [global_fixed_income, em_debt, global_high_yield, private_credit, em_equity, systematic, multi_asset]
audience: [institutional, intermediary, wealth, internal]
jurisdiction: [UK, US, APAC, Japan, Germany, France, global]
compliance_status: [DRAFT, LEGAL_REVIEW, APPROVED, EXPIRED]
expiry_date: ISO8601
version: semver (1.0.0)
approved_by: Salesforce User_ID (string reference)
```

### Gap 6 — Board-Level Benefit Quantification Arithmetic Check (Priority: MEDIUM)
**Current state:** Document states "180,000 – 345,000 minutes lost annually" in Section 1 and "750 working days" in Section 7.
**Required correction:**
```
Section 1: 30 RMs × 10 retrievals/week × 50 weeks × (12–23 min) = 180,000 – 345,000 minutes ✅
Section 7: 30 × 10 × 50 × 24 min saved = 360,000 minutes ÷ 480 min/day = 750 working days ✅
Note: "750 working days" = 3 working years is not comparable to "working days of a person"
Reframe as: 6,000 person-hours per year (30 × 10 × 50 × 24 min / 60 min) — more defensible in interview
```

---

## Structural Enhancements for Cycle 2

1. **Section 3 (AI Engine):** Add "Combined Architecture Diagram" showing Bedrock conversation layer + JumpStart cross-sell model as two distinct inference paths, both served through same Salesforce Agentforce interface
2. **Section 4 (Use Cases):** Add Mermaid-style sequence description for Use Case 2 showing the full DDQ response flow with confidence branch
3. **New Section 11:** Complete the NCIE cross-document bridge section — NCIE output types mapped to specific Agentforce chatbot triggers
4. **Section 12 (Benefit quantification):** Replace "750 working days" with "6,000 person-hours/year" everywhere for metric consistency

---

## Panel Certification: Cycle 1

| Evaluator | Certify for next cycle? | Condition |
|---|---|---|
| Evaluator A | ✅ Yes | Address Bedrock Agents multi-step spec (Gap 1) |
| Evaluator B | ✅ Yes | Specify Agentforce action implementation types (Gap 3) |
| Evaluator C | ✅ Yes | Complete SageMaker fine-tuning pipeline (Gap 2) and confidence threshold spec (Gap 4) |
| Evaluator D | ✅ Yes | Fix arithmetic framing (Gap 6) and add NCIE bridge section (Gap 5) |

**Status: PROCEED TO CYCLE 2**
**Score required for Cycle 2 certification: ≥ 9.20 / 10.00**
