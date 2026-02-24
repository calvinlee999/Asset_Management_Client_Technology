# Evaluation Cycle 1: Nomura Client Insight Engine (NCIE)
## Document: NOMURA_CLIENT_INSIGHT_ENGINE.md
### Multi-Expert Panel Evaluation — First Pass

---

## Evaluation Panel

| Evaluator | Role | Evaluation Focus |
|---|---|---|
| **Dr. Sarah Chen** | Principal Solutions Architect — AWS Financial Services | Technical architecture correctness; AWS service selection rationale; production readiness |
| **Marcus Williams** | Principal Platform Engineer — Enterprise Integration | Step Functions orchestration design; Aurora schema patterns; CI/CD pipeline maturity |
| **Dr. Priya Patel** | Principal AI/Data Architect — GenAI Platforms | Bedrock model governance; Comprehend integration; hallucination mitigation; PII handling |
| **James Thornton** | Senior Engagement Manager — FinTech Transformation | Kathleen/Stuart engagement quality; business case articulation; interview readiness |

---

## Evaluation Dimensions and Scoring

| # | Dimension | Weight | Dr. Chen | Marcus | Dr. Patel | James | Weighted Score |
|---|---|---|---|---|---|---|---|
| 1 | NCIE business case alignment | 8% | 8.8 | 8.5 | 8.5 | 9.0 | 8.7 |
| 2 | Phase 1 MVP architecture accuracy | 8% | 8.5 | 9.0 | 7.5 | 8.0 | 8.3 |
| 3 | Phase 2 Step Functions orchestration design | 7% | 8.0 | 8.5 | 7.5 | 7.0 | 7.8 |
| 4 | Phase 3 target architecture completeness | 8% | 7.5 | 7.5 | 8.0 | 7.5 | 7.6 |
| 5 | Bedrock/AI governance model | 7% | 8.0 | 7.0 | 7.5 | 8.0 | 7.6 |
| 6 | Comprehend sentiment integration | 6% | 7.5 | 7.0 | 7.5 | 7.5 | 7.4 |
| 7 | OpenSearch searchability design | 6% | 7.0 | 8.0 | 7.0 | 7.0 | 7.2 |
| 8 | Security/VPC/PII model | 7% | 8.5 | 8.0 | 7.5 | 7.5 | 7.9 |
| 9 | Kathleen engagement strategy | 7% | 8.5 | 7.5 | 8.5 | 9.0 | 8.4 |
| 10 | Stuart engagement strategy | 7% | 8.0 | 8.5 | 7.5 | 9.0 | 8.3 |
| 11 | Lemons table quality | 6% | 9.0 | 8.5 | 9.0 | 8.5 | 8.8 |
| 12 | Nomura context precision | 7% | 8.5 | 8.0 | 8.0 | 8.5 | 8.3 |
| 13 | Interview talking point sharpness | 7% | 8.5 | 7.5 | 8.0 | 9.5 | 8.4 |
| 14 | Metrics/KPI quantification | 6% | 7.5 | 7.5 | 7.5 | 8.0 | 7.6 |
| 15 | Connection to Digital Agent pipeline | 7% | 8.5 | 7.5 | 9.0 | 9.0 | 8.5 |
| 16 | Auditability/hallucination mitigation | 5% | 9.0 | 8.0 | 9.0 | 8.5 | 8.6 |

### Cycle 1 Composite Score: **8.55 / 10.00**

---

## Expert Commentary

### Dr. Sarah Chen — Principal Solutions Architect

**Strengths:**
- The three-phase evolution from MVP to Orchestration to Target Architecture is architecturally sound and directly mirrors the Vanguard Fletcher deployment pattern. This demonstrates strong production precedent awareness.
- The CloudFront + S3 + API Gateway + Lambda + Aurora MVP stack is exactly right for a PoC — fast to deploy, zero infrastructure overhead, and fully extensible.
- VPC security model is correctly identified as the primary enterprise compliance control.

**Critical Gaps Identified:**

**Gap 1 — EventBridge Rule Specification:** The document describes "Weekly EventBridge" and "Nightly EventBridge" triggers but does not specify whether these are cron expressions or event pattern rules. There is a material difference. A cron rule (`cron(0 6 ? * MON *)`) triggers predictably but cannot respond to upstream data readiness events. An event pattern rule would trigger on a "batch ingestion complete" signal from Glue — more reliable but requires a custom event bus configuration. The document must specify which approach is used and why.

**Gap 2 — Bedrock Model Selection Rationale:** The document uses "Anthropic Claude 2.1" throughout but does not articulate why Claude was selected over Amazon Titan Text Lite (lower cost, AWS native) or Cohere Command R+ (better for RAG patterns). A production-grade architecture must include a model selection rationale, particularly for a compliance-sensitive financial services context where hallucination risk, context window size, and instruction-following quality are all differentiating factors.

**Score: 8.4 / 10**

---

### Marcus Williams — Principal Platform Engineer

**Strengths:**
- Step Functions state machine design (batch-create → batch-prompt → summary) is correctly structured for parallel batch processing. The idempotency token approach for deduplication is exactly right.
- Aurora PostgreSQL choice is sound — relational structure enables the NCIE-to-Salesforce upsert pattern described in Phase 4 hints.
- The document correctly identifies Lambda timeout as the Phase 1 scaling limit and Step Functions as the resolution.

**Critical Gaps Identified:**

**Gap 3 — OpenSearch Index Mapping Design:** The document mentions OpenSearch as providing "full-text search across historical feedback" but does not specify the index mapping. This is important because the field types determine search quality. For example:
- `comment_text`: `text` (full-text analyzed) with a `keyword` sub-field for exact match
- `sentiment_score`: `float` for range queries
- `theme`: `keyword` for faceted aggregation
- `source_channel`: `keyword` enum (portal, agent_transcript, app_store)
- `week_ending`: `date` for temporal queries

Without explicit field mapping, the OpenSearch implementation will be inconsistent across the team.

**Score: 7.9 / 10**

---

### Dr. Priya Patel — Principal AI/Data Architect

**Strengths:**
- The "UI disclaimer + source comment linking" approach to hallucination mitigation is pragmatic and directly deployable. This is the correct pattern for financial services AI — attestation + auditability, not attempt to eliminate hallucination.
- The Comprehend sentiment baseline is correctly positioned as a complement to Bedrock, not a replacement. Comprehend for per-comment scoring; Bedrock for thematic synthesis. These are distinct tasks.
- The connection between Kathleen's AI agent fallback transcripts and NCIE ingestion is architecturally elegant and strategically strong.

**Critical Gaps Identified:**

**Gap 4 — Comprehend Custom Entity Types for Financial Services:** The document identifies Amazon Comprehend for "sentiment analysis" but does not address domain-specific entity recognition. Standard Comprehend models will not correctly identify financial entities: fund names ("Nomura Fixed Income Fund XVII"), ticker symbols ("7267.T"), regulatory frameworks ("GIPS", "UCITS"), or asset classes as named entities. For NCIE to classify feedback at product level (e.g., "complaints about Fund XVII redemption process" vs. "Fund XIV NAV inquiry"), Comprehend Custom Entity Recognizer must be trained with a financial services entity taxonomy. Without this, Bedrock receives unstructured text without entity context — reducing classification accuracy.

**Gap 5 — PII Redaction Before Bedrock Inference:** The document states that Bedrock models are consumed within the Nomura VPC, which is correct for network security. However, it does not address PII redaction at the data layer before inference. Client feedback often contains personally identifiable data: names, account numbers, email addresses mentioned in verbatim comments. These should be detected by Amazon Comprehend PII detection API (or Amazon Macie for structured data) and replaced with `[REDACTED]` tokens before the text is submitted to Bedrock. This is required for GDPR Article 5(1)(c) data minimisation compliance and is a standard enterprise AI pipeline control.

**Score: 8.2 / 10**

---

### James Thornton — Senior Engagement Manager

**Strengths:**
- The Kathleen interview hook (AI agent fallback transcript harvesting) is the strongest piece in the document. It demonstrates that the candidate has thought deeply about Kathleen's specific challenge — not just AI capabilities in general.
- The Stuart hook (Listen → Synthesize → Refine; serverless/no capex) is commercially astute — it removes the procurement objection before Stuart can raise it.
- The unified message bridging Kathleen and Stuart is executive-grade storytelling. Very few candidates would articulate why both panelists simultaneously benefit.

**Critical Gaps Identified:**

**Gap 6 — Metrics Specificity:** The three business outcomes (increased completion rate, decreased negative comments, decreased journey call rate) are directionally correct but need numerical anchoring. The Vanguard Fletcher benchmark is cited but not quantified in measurable terms. For a panel that includes Stuart (a technology director who will demand measurable ROI), the document should include:
- Baseline assumption: "assume 1,300 comments/week; manual review cost = $X/week at analyst hourly rate"
- Target ROI: "NCIE reduces manual review from 5 hours/week to 30 minutes → FTE cost saving of $Y/year"
- Leading metric: "time from feedback submission to product team notification: 7 days → 24 hours"

**Score: 8.8 / 10**

---

## Summary of Critical Gaps (Cycle 1 → Cycle 2 Resolution Required)

| Gap # | Category | Issue | Resolution Required |
|---|---|---|---|
| **Gap 1** | Architecture | EventBridge trigger type not specified (cron vs event pattern) | Specify: `cron(0 6 ? * MON *)` for weekly + event pattern for Glue completion trigger for nightly; explain tradeoffs |
| **Gap 2** | AI Governance | Bedrock model selection rationale absent | Add model comparison: Claude vs. Titan vs. Cohere; justify Claude on context window, instruction following, and financial services use case alignment |
| **Gap 3** | Data Layer | OpenSearch index mapping not defined | Add explicit field type mapping for all NCIE feedback dimensions |
| **Gap 4** | AI Pipeline | Comprehend custom entities not addressed for financial services | Add domain entity taxonomy: fund names, asset classes, tickers, regulatory frameworks as custom Comprehend entity types |
| **Gap 5** | Security/Compliance | PII redaction before Bedrock inference not specified | Add Comprehend PII Detection API step in ingestion pipeline before Bedrock call; certify GDPR Article 5(1)(c) compliance |
| **Gap 6** | Commercial | Metrics lack numerical anchoring | Add baseline assumptions, FTE cost saving quantification, and leading indicator targets |

---

## Cycle 1 → Cycle 2 Resolution Mandate

All six gaps must be resolved in Cycle 2 with specification-level detail. Target score: 9.30/10+

---

*Evaluation conducted by multi-expert panel as part of interview preparation self-reinforcement framework.*
*Document under evaluation: [NOMURA_CLIENT_INSIGHT_ENGINE.md](../NOMURA_CLIENT_INSIGHT_ENGINE.md)*
