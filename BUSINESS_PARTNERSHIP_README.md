# Asset Management Client Technology — Business Partnership Guide
## Prepared for: Kathleen Hess McNamara (ED, Client Experience & AI) | Stuart Mumley (Director, Client Platforms)
### Role: Executive Director, Head of Client Technology — Nomura Asset Management International

---

> **How to use this document**: This is not an architecture document. It is a business partnership playbook — translating the technical platform into the language of client experience, distribution enablement, and measurable business outcomes. Every section maps technology decisions to the outcomes Kathleen and Stuart care about most.

---

## Table of Contents

1. [The Business We Are Enabling](#1-the-business-we-are-enabling)
2. [Client Channels & How Technology Serves Each](#2-client-channels--how-technology-serves-each)
3. [Investment Products & Platform Implications](#3-investment-products--platform-implications)
4. [The Distribution Engine: Sales to Advisors & Institutions](#4-the-distribution-engine-sales-to-advisors--institutions)
5. [Key Business Processes & Technology Touchpoints](#5-key-business-processes--technology-touchpoints)
6. [AI Digital Agent: DDQ/RFP Automation](#6-ai-digital-agent-ddqrfp-automation)
7. [Client Reporting Modernization](#7-client-reporting-modernization)
8. [Salesforce as Engagement Orchestration Layer](#8-salesforce-as-engagement-orchestration-layer)
9. [Seismic: Sales Enablement Platform](#9-seismic-sales-enablement-platform)
10. [Regulatory Context: Series 7, Series 63 & Compliance](#10-regulatory-context-series-7-series-63--compliance)
11. [Business Glossary](#11-business-glossary)
12. [Partnership Principles: Product + Technology](#12-partnership-principles-product--technology)
13. [KPIs That Matter to This Business](#13-kpis-that-matter-to-this-business)
14. [Questions for the Panel](#14-questions-for-the-panel)
15. [Building Trust, Relationship & Influence](#15-building-trust-relationship--influence)
16. [Early Alignment: Business Intent & Technical Constraints](#16-early-alignment-business-intent--technical-constraints)
17. [Product-Technology Collaboration Model](#17-product-technology-collaboration-model)
18. [Vendor Application Evaluation Framework](#18-vendor-application-evaluation-framework)
19. [Asset Management Content Distribution & Digital Asset Management](#19-asset-management-content-distribution--digital-asset-management)
20. [AI Chatbot + DAM + Salesforce Integration Architecture](#20-ai-chatbot--dam--salesforce-integration-architecture)

---

## 1. The Business We Are Enabling

### What Nomura Asset Management Does

Nomura Asset Management International raises capital from clients — institutional investors (pension funds, endowments, sovereign wealth funds) and wealth clients (high-net-worth individuals through advisors) — and invests it across equities, fixed income, multi-asset, and private credit strategies. Revenue is a percentage of **AUM (Assets Under Management)**. The firm earns management fees and sometimes performance fees.

**The technology leader's mandate is simple:** every platform decision must either protect existing AUM (retention) or enable new AUM growth (distribution). Everything else is noise.

### The Three Business Divisions Relevant to This Role

| Division | What They Do | Tech Platforms Serving Them |
|---|---|---|
| **Wealth Management** | Serve affluent clients through advisors and private bankers | Salesforce CRM, Seismic, Client Portal, Reporting |
| **Investment Management** | Manage portfolios, generate returns, report performance | Reporting platforms (Vermilion/PowerBI), Data layer, GIPS automation |
| **Wholesale / Distribution** | Institutional sales, RFPs, DDQs, mandate pipeline | AI Digital Agent, CRM, DDQ/RFP automation, entitlement workflows |

### What "Client Technology" Means in This Context

The Head of Client Technology owns the platforms that connect the investment management business to its clients and distribution partners. This is not back-office infrastructure. It is the **face of the firm** from a digital perspective — what clients see, what advisors use, what relationship managers depend on every day.

---

## 2. Client Channels & How Technology Serves Each

### Channel 1: Institutional Clients

**Who they are:** Pension funds, endowments, foundations, sovereign wealth funds, insurance companies, family offices. They allocate $50M–$5B+ at a time. Examples: CalPERS, university endowments, state pension plans.

**How they are served:**
- Long sales cycle (12–24 months from prospect to mandate)
- Extensive due diligence process (RFPs, DDQs, site visits)
- Ongoing reporting requirements (GIPS-compliant performance, attribution, risk)
- Dedicated Relationship Managers (RMs) managing the relationship

**Technology touchpoints:**
- **CRM (Salesforce):** Tracks prospect pipeline, meeting history, mandate status
- **AI Digital Agent:** Automates DDQ/RFP responses from approved knowledge base
- **Client Portal:** Secure access to reports, fund documents, performance data
- **Reporting Platform (Vermilion/PowerBI):** GIPS-compliant performance reporting, attribution

**Business outcome technology must deliver:**
> Reduce friction in the due diligence process. Make reporting so transparent and accurate that institutional clients never question data integrity. Enable RMs to spend time building relationships, not assembling reports.

---

### Channel 2: Wealth / Affluent Clients

**Who they are:** High-net-worth and ultra-high-net-worth individuals, typically with $1M–$100M+ in investable assets. Served through financial advisors, private bankers, or directly.

**How they are served:**
- Relationship-driven; advisor is the primary point of contact
- Products: mutual funds, ETFs, separately managed accounts (SMAs)
- Quarterly performance reporting, tax reporting
- Digital portal for self-service access to statements and holdings

**Technology touchpoints:**
- **Client Portal:** Statement access, performance vs. benchmark, holdings view
- **Reporting:** Simplified performance reporting (less complex than institutional)
- **CRM:** Advisor relationship tracking, client event history

**Business outcome technology must deliver:**
> Give advisors tools that make them look good in front of their clients. Give clients digital access that reduces service calls and increases satisfaction scores.

---

### Channel 3: Financial Advisors & Intermediaries

**Who they are:** Registered Investment Advisors (RIAs), broker-dealers, wirehouse advisors (Merrill Lynch, Morgan Stanley, UBS, Wells Fargo Advisors). They are the distribution layer between Nomura and affluent end clients.

**How they are served:**
- Nomura's internal wholesalers call on advisors to get funds on approved lists
- Advisors need current, compliant content to present to their clients
- Ongoing education on product positioning, market commentary, fund updates

**Technology touchpoints:**
- **Seismic:** The sales enablement platform delivering approved content to advisors at the right moment
- **Salesforce:** Tracks wholesaler activity, advisor engagement, territory management
- **Events platforms:** Virtual and in-person client events (Stuart's team built 500+ attendee digital events at Macquarie)

**Business outcome technology must deliver:**
> Make advisors feel equipped and supported. Ensure every piece of content they use is current, compliant, and easy to find. Measure engagement so internal sales teams know which advisors are most active and prioritize accordingly.

---

## 3. Investment Products & Platform Implications

### Mutual Funds

**What they are:** Pooled investment vehicles with professional management. Priced once daily at **NAV (Net Asset Value)**. Available to both retail and institutional investors.

**Platform implication:** Daily NAV calculation and distribution is a core obligation. If NAV is wrong, late, or unavailable — client trust erodes immediately. Vermilion reporting must reflect accurate, timely NAV. This is a zero-defect zone.

**Key metric for technology:** NAV delivery time, NAV accuracy rate, client report generation SLA.

---

### ETFs (Exchange-Traded Funds)

**What they are:** Similar to mutual funds but traded on exchanges throughout the day like stocks. Typically passively managed, tracking an index (e.g., S&P 500 ETF). Lower cost than actively managed funds.

**Platform implication:** Unlike mutual funds, ETFs have intraday pricing. Client-facing dashboards should reflect near-real-time pricing. Attribution reporting is simpler than active funds but must still be accurate.

---

### Fixed Income

**What it is:** Bonds and debt instruments. The issuer (government or corporation) pays periodic interest (the **coupon**) and returns principal at **maturity**. Fixed income is the core of Kathleen's Credit Suisse background.

**Types:**
- **Government bonds (Treasuries):** Lowest risk, issued by governments
- **Corporate bonds:** Higher yield, higher risk, issued by companies
- **Credit derivatives (Kathleen's specialty):** Instruments to transfer credit risk, including **CDS (Credit Default Swaps)** — essentially bond insurance

**Platform implication:** Fixed income portfolios require complex **attribution reporting** — explaining performance by duration contribution, credit quality, sector, and issuer. Data lineage from trading systems to client reports must be clean. Kathleen managed these trading workflows at Credit Suisse — she understands how complex this data is.

---

### Equities

**What they are:** Stocks — ownership stakes in public companies. Portfolio performance measured against equity benchmarks (S&P 500, MSCI World, Russell 2000).

**Platform implication:** Attribution breaks down returns into: market timing, sector allocation, and individual stock selection. Client reporting shows portfolio vs. benchmark performance over 1-, 3-, 5-year periods.

---

### Private Credit

**What it is:** Lending to mid-size companies outside traditional bank loans or public bond markets. Illiquid (difficult to sell), typically higher yield than public debt. Nomura specifically names private credit as a strategy.

**Platform implication:** No daily market price — valuation is periodic (monthly or quarterly). Reporting is more complex than public markets. Institutional clients require detailed cash flow reports, loan-level transparency, and manager commentary. This is an area of growing client demand and a reporting modernization opportunity.

---

### Multi-Asset & Alternatives

**Multi-asset:** Strategies that blend equities, fixed income, alternatives, and real assets. Designed for diversification. Kathleen's TIFF experience was a fund-of-funds investing in alternatives on behalf of endowments — she has deep operational experience here.

**Alternatives:** Hedge funds, private equity, real estate, infrastructure. Require specialized reporting, longer settlement cycles, and complex due diligence.

**Platform implication:** Multi-asset reporting requires aggregating data across multiple asset classes with different data sources, pricing frequencies, and benchmarks. This is the core complexity Kathleen addressed with Vermilion at Macquarie.

---

## 4. The Distribution Engine: Sales to Advisors & Institutions

### How Institutional Sales Works

```
Prospect Identified
        ↓
Initial RM Outreach (Salesforce activity logged)
        ↓
Prospect Sends RFP (Request for Proposal)
  → Nomura describes strategy, team, risk controls, fees
  → AI Digital Agent drafts from approved knowledge base
        ↓
Prospect Sends DDQ (Due Diligence Questionnaire)
  → 100–300 operational questions (IT, compliance, risk, operations)
  → AI Digital Agent + human review
        ↓
Site Visit / Investment Committee Meetings
        ↓
Mandate Awarded (contract signed)
        ↓
Client Onboarding (account setup, entitlements, reporting configured)
        ↓
Ongoing Servicing (quarterly reports, RM touchpoints, portal access)
```

**Technology accelerates every stage.** The AI agent compresses weeks of RFP/DDQ work into hours. CRM ensures no prospect falls through the cracks. Onboarding automation reduces time-to-first-report from weeks to days. Reporting transparency reduces client service calls and increases retention.

---

### How Advisor / Wealth Distribution Works

```
Wholesaler Identifies Target Advisor
        ↓
Pre-Meeting Prep (Stuart's AI meeting prep tool)
  → CRM shows advisor history, AUM, current product holdings
  → Seismic surfaces most relevant content for this advisor's client profile
        ↓
Meeting with Advisor
  → Wholesaler presents fund fact sheets, performance, market commentary
  → All content pulled from Seismic (approved, current, compliant)
        ↓
Advisor Places Fund on Approved List
        ↓
Advisor Recommends to Clients
        ↓
Post-Meeting Follow-Up (Stuart's AI post-meeting action generator)
  → Automated meeting notes, follow-up tasks logged in CRM
        ↓
Ongoing Advisor Engagement
  → Seismic tracks content engagement (did advisor open the fact sheet?)
  → CRM triggers re-engagement if advisor goes quiet
```

**Stuart built exactly this workflow at Macquarie** — AI pre-meeting prep, Seismic integration, Salesforce CRM as the backbone. He will expect the new Head of Client Technology to understand and extend this model at Nomura.

---

## 5. Key Business Processes & Technology Touchpoints

### RFP — Request for Proposal

**What it is:** When an institutional prospect is evaluating investment managers, they send an RFP asking the firm to describe its strategy, investment process, team, performance history, risk management approach, and fees. Answering RFPs accurately and quickly is a competitive advantage.

**The business pain:** A single RFP can have 50–150 questions. Nomura may receive dozens per quarter. Each response must be accurate, consistent with prior responses, and comply with regulatory standards. Traditionally this is highly manual.

**Technology solution:** Kathleen's AI digital agent grounds an LLM in an authoritative, versioned knowledge base of approved answers. The agent drafts responses; human reviewers verify and approve. This is not replacing humans — it is eliminating the blank page problem and ensuring consistency.

**What to say in the room:**
> *"The AI agent isn't the hard part — content governance is. The agent is only as trustworthy as the knowledge base it draws from. Who owns each answer? How does an answer get updated when strategy or policy changes? How do reviewers know what changed? That governance architecture is what makes the difference between a pilot and a production system that the business trusts."*

---

### DDQ — Due Diligence Questionnaire

**What it is:** The operational deep-dive companion to the RFP. Where an RFP asks about investment strategy, a DDQ asks about operations, technology, compliance, and risk controls. Questions like: "Describe your disaster recovery plan." "How is client data protected?" "What is your policy on employee trading?" "Describe your order management system."

**Why this touches you directly:** Many DDQ questions are about the technology platform — cybersecurity, data governance, business continuity, system redundancy. As Head of Client Technology, you are responsible for the answers about IT infrastructure and controls. Kathleen will expect you to be a collaborative partner in answering technology DDQ sections accurately.

**What to say in the room:**
> *"I've been on both sides of DDQs — as the technology leader responsible for the controls being described, and as the partner helping document them clearly. I'd want to work with Kathleen's team to make sure our technology DDQ answers are always accurate and that when we make platform changes, the relevant DDQ answers are updated in the knowledge base."*

---

### GIPS — Global Investment Performance Standards

**What it is:** The global standard (administered by CFA Institute) defining how investment performance must be calculated, presented, and verified. Institutional clients require GIPS-compliant performance presentation. It ensures apples-to-apples comparison across managers.

**Key concepts:**
- **Composite:** A grouping of portfolios with similar investment mandates, used to represent a strategy's track record
- **Benchmark:** The index or reference portfolio performance is measured against (e.g., Bloomberg Aggregate for fixed income)
- **Gross vs. Net returns:** Gross is before fees; net is after fees. Both must be presented clearly
- **Verification:** Third-party auditors verify GIPS compliance

**Platform implication:** Your data layer must calculate composites correctly, maintain historical performance consistency, and generate GIPS-compliant presentations for inclusion in RFPs and client reports. Any error in GIPS reporting is not just a client experience problem — it is a regulatory and reputational risk.

**What to say in the room:**
> *"GIPS compliance isn't just a reporting layer concern — it starts at the data layer. If composite construction logic isn't governed consistently, or if there's ambiguity about which portfolios belong in which composite, the numbers won't hold up under verification. Getting the data governance right upstream is what makes GIPS reporting trustworthy downstream."*

---

### Portfolio Attribution

**What it is:** The process of explaining why a portfolio performed the way it did, decomposing returns into contributions from different decisions.

**For equity portfolios:** Breaks down into sector allocation effect, security selection effect, and interaction effect vs. benchmark.

**For fixed income portfolios:** Breaks down into duration contribution, yield curve positioning, credit spread contribution, and currency effect.

**For multi-asset portfolios:** Breaks down by asset class allocation and within-asset-class selection.

**Platform implication:** Attribution data must flow cleanly from portfolio management systems through the data layer to client-facing reports. The relationship manager uses attribution to explain performance to clients. If the data is wrong or the presentation is unclear, the RM is in a difficult conversation with a client.

**What to say in the room:**
> *"Attribution is one of those areas where data lineage really matters. If a client asks why their portfolio underperformed last quarter and the RM can't trace the attribution to clean, auditable source data, that erodes trust quickly. The reporting modernization work is really a data lineage problem as much as a presentation problem."*

---

### Client Onboarding

**What it is:** The process of setting up a new client after a mandate is won — account opening, legal documentation, data entitlements, reporting configuration, portal access provisioning.

**The business pain:** Traditional onboarding takes weeks or months. Every week of delay is a week the client relationship starts on a negative note. Kathleen's background in operations (TIFF Deputy Head of Member Services, Credit Suisse middle office) means she has lived this pain directly.

**Technology solution:** Stuart's team at Macquarie reduced onboarding friction through automated entitlement provisioning (Salesforce → workflow → data access granted). Your repo documents this: Salesforce contract signed → Step Functions → ADX access provisioned in under 2 minutes vs. days manually.

**What to say in the room:**
> *"Client onboarding is often the first impression of what working with the firm will feel like operationally. If it's slow and manual, it signals to the client that the operational infrastructure isn't modern. If it's fast and seamless, it reinforces the confidence they had in awarding the mandate. I'd want to understand what the current onboarding experience feels like from the client's perspective — where are the handoffs that slow things down?"*

---

## 6. AI Digital Agent: DDQ/RFP Automation

### What Kathleen Built at Macquarie

Kathleen operationalized Macquarie Asset Management's **first-ever AI Digital Agent** for completing DDQs and RFPs. This was not a proof of concept — she took it to production. This is a significant operational achievement in an industry that moves slowly on AI adoption due to regulatory and accuracy concerns.

### The Three Layers of AI Maturity (Use This Framework)

| Layer | Description | Example |
|---|---|---|
| **1. Internal Productivity** | AI helps internal teams work faster | DDQ/RFP agent drafting responses |
| **2. Assisted Decision Support** | AI provides recommendations; humans decide | Pre-meeting client insights for RMs |
| **3. Client-Facing Automation** | AI interacts directly with clients | Self-service portal with AI-powered Q&A |

Kathleen has built Layer 1. Stuart has built Layer 2 (AI pre-meeting prep for sales). Layer 3 is the frontier.

### What Kathleen Will Want to Know You Understand

**Content governance** is the critical success factor. The AI agent is only trustworthy if:
1. The knowledge base contains only approved, current answers
2. Every answer has a clear owner who is responsible for keeping it current
3. When policy or strategy changes, the relevant answers are updated in the knowledge base — not just in the next RFP response
4. The agent's outputs are reviewed by qualified humans before client submission
5. There is an audit trail of what the agent produced vs. what was sent

**Adoption sustainability** depends on the sales and operations team trusting the output. If the agent produces an answer that is wrong once, the team loses confidence and reverts to manual process. Building trust requires transparent accuracy metrics, clear escalation paths, and honest communication about what the agent can and cannot do.

### The Architectural Answer: Strictly Governed RAG Mesh (Not Fine-Tuning)

The most important architectural decision in an AI Digital Agent for DDQ/RFP work is how facts enter and leave the system. There are two approaches. Only one is governance-compatible.

**The wrong approach — Fine-Tuning:**

Fine-tuning bakes facts into the AI model's static weights. When a fund strategy changes, when GIPS methodology is updated, or when a portfolio manager leaves the firm — the fine-tuned model retains the old answer permanently. Correcting it requires a full model retrain costing $50,000–$500,000 and taking 4–8 weeks. There is no audit trail of what the model "knew" at the time it generated a response. For a firm with FINRA and SEC obligations, this is architecturally incompatible with compliance.

**The right approach — Strictly Governed RAG Mesh:**

> RAG (Retrieval-Augmented Generation) keeps all facts in a governed knowledge base (Amazon Bedrock Knowledge Bases / OpenSearch). The LLM supplies language fluency and reasoning — it never stores facts. Every fact has a source, an owner, and an expiry date. Any fact can be surgically corrected in minutes, with a full audit trail, without touching the model.

| Dimension | Fine-Tuning | Governed RAG Mesh |
|---|---|---|
| **Correction speed** | Full retrain: 4–8 weeks, $50K–$500K | Knowledge base update: minutes |
| **Audit trail** | None — fact baked into weights | Full: source document, owner, version, expiry |
| **Expired content** | Persists until retrain | Auto-purged by infrastructure on expiry date |
| **FINRA/SEC compliance** | Cannot prove what model "knew" | Complete provenance for every response |
| **Appropriate use** | Tone and jargon calibration | ALL factual financial and compliance content |

Kathleen will recognize this distinction immediately. She has operated a production system and knows that the governance question — "can we prove this answer was current and approved at the time of submission" — cannot be answered with a fine-tuned model.

### The 5 Pillars of AI Content Governance (Business Language)

These are the five operational disciplines that make an AI Digital Agent trustworthy enough for regulated financial submissions. This is how you describe the system to Kathleen.

**Critical business case connection:** The NAPCE architecture projects a **90% reduction in DDQ/RFP turnaround time** — from 3–6 weeks to hours. That reduction is only credible to an institutional prospect if the governance architecture is airtight. A 90% faster DDQ that contains a stale policy citation is worse than a slow DDQ — it combines speed with error at scale. The 5 Pillars are not overhead; they are the mechanism that makes the 90% turnaround commitment defensible.

**Pillar 1 — "Current Answers Only" Perimeter**

Every piece of content in the knowledge base has an expiry date. When that date arrives, the content is automatically removed from the retrieval system — the AI agent cannot access it. No policy change, strategy update, or compliance brief from yesterday can contaminate today's DDQ response. The boundary is enforced by infrastructure, not by a prompt instruction the model could ignore.

*What to say: "The agent is architecturally incapable of citing an expired policy. The content has been removed from the retrieval layer — it no longer exists for the model to find."*

**Pillar 2 — Clear Ownership & Accountability**

Every chunk of knowledge is bonded to a named human owner at the moment it enters the system. The owner's name and review date appear in every AI-generated citation. When a client sees a response that cites "ESG Policy v4 · Owner: Jane Doe · Next Review: May 2026," Jane Doe is the person accountable for that answer being current and accurate.

*What to say: "Every answer has a face on it. There is no anonymous policy citation — there is an owner with their name on the line."*

**Pillar 3 — Centralized Updates, No Ad-Hoc Fixes**

When an answer is wrong, you cannot fix it by editing the next response. The fix must go back to the knowledge base source. A correction workflow in Salesforce routes the fix through governance, re-vectorizes the corrected answer, and makes it live across all future responses within minutes. The DDQ/RFP export is blocked until the correction is submitted — there is no workaround.

*What to say: "A correction to 'this response' is a correction to all future responses. The governance pipeline ensures knowledge base integrity is maintained, not just this document."*

**Pillar 4 — Human-in-the-Loop for Every Submission**

No DDQ or RFP leaves without explicit human authorization. AWS Step Functions pauses the workflow while the Principal Portfolio Manager or Compliance Officer reviews the AI draft in Salesforce. The "Approve" click triggers the final document and generates an immutable transaction record identifying the reviewer, the timestamp, and what was changed. The AI accelerates document production by 90%; the human retains legal and regulatory accountability.

If no action is taken within 72 hours, the workflow automatically escalates to a senior reviewer. At 96 hours, a CloudWatch alarm notifies the Head of Client Technology. There is no path to an unapproved submission — the system escalates; it does not time out silently.

*What to say: "The AI produces the draft in hours instead of weeks. The human authorizes the submission. The 72-hour escalation policy means there is no scenario where a document sits in review indefinitely. The accountability chain is unbroken."*

**Pillar 5 — Immutable Audit Trail**

Every AI-generated document produces three permanent records: the raw AI output (what the model said), the human-approved output (what was submitted), and a machine-readable diff (exactly what the human changed, expressed as a percentage of the total document). These records are stored in Snowflake Immutable Tables and Amazon QLDB — append-only, cryptographically verified, and queryable by Compliance and Legal. When a regulator asks "who approved this and what did the AI originally say," the answer is retrievable in under 60 seconds.

> *Technical note: Amazon QLDB uses a cryptographic digest chain (SHA-256 journal hash chain), not a distributed blockchain. Each entry is independently verifiable against the journal hash without requiring consensus from other parties — making it ideal for regulated audit trail use cases.*

*What to say: "The audit trail is not a log file. It is a cryptographically verified chain of custody from AI draft to human-approved submission — with every modification documented. A regulator can reconstruct what the AI said, what the human changed, and who authorized submission — in under 60 seconds."*

### The Three Governance Anti-Patterns (Know These Before You Walk In)

These are the three failure modes Kathleen has either seen or guarded against in production. Knowing these signals that you understand the operational reality.

**Anti-Pattern 1 — The Fine-Tuning Misconception**

> *"We should just retrain the model with our latest fund information."*

This conflates a language model with a database. Language models are not databases. Retraining a model to update a fact costs $50,000–$500,000. The old fact cannot be surgically removed — it competes with the new fact in the model's weights. The correct answer: all facts live in the governed knowledge base (RAG), not in the model.

**Anti-Pattern 2 — "Frankenstein" Answers**

The agent retrieves fragments from two overlapping policy documents (e.g., the 2024 and 2025 RFP templates) and synthesizes a chimera answer that cites neither source accurately. The answer is fluent and confident. It is also wrong. Mitigation: explicit system prompt rules prohibiting synthesis across sources, combined with a groundedness threshold that flags conflicts for human review rather than resolving them autonomously.

**Anti-Pattern 3 — Orphaned Knowledge**

A senior portfolio manager authors 40 knowledge base entries for a niche credit strategy. She leaves the firm. Her entries remain in the knowledge base — no owner, no review cadence, gradually diverging from current strategy. Six months later, the AI agent cites her analysis in live RFPs. Mitigation: Workday HR system integration. On `Employment_Status: Terminated`, all knowledge base entries owned by that individual are automatically flagged as requiring review and removed from active retrieval until a successor takes ownership.

### Compliance Framework Alignment

Kathleen holds Series 7 and Series 63 licenses. She understands that content governance is a regulatory obligation, not a product preference. Two international frameworks define the bar:

| Standard | What It Requires | How the Architecture Addresses It |
|---|---|---|
| **ISO/IEC 42001** (AI Management Systems Standard) | Defined accountability for AI systems; transparency of AI decision-making; systematic risk management | Pillars 2 (ownership), 4 (HITL authorization), and 5 (immutable audit trail) directly satisfy the accountability and transparency requirements of ISO 42001 Annex A |
| **NIST AI RMF** (AI Risk Management Framework) | Govern (policies and accountability); Map (risk identification); Measure (performance tracking); Manage (risk controls) | Govern = Pillar 3 (centralized update governance); Map = the Three Anti-Patterns identification; Measure = Pillar 5 diff delta and hallucination KPIs; Manage = Pillar 1 content perimeter |

*Knowing these frameworks — and being able to map the architecture to them — signals to Kathleen that you are thinking about AI governance at the same level of rigor she applies as a licensed professional with regulatory accountability.*

### What to Say in the Room

> *"At FIS I led AI-enabled document processing for capital markets workflows. The lesson I took from it is that the model is rarely the bottleneck — content governance is. There are three questions I'd want to answer before scaling any AI agent to new use cases: Do we have clear ownership of every answer in the knowledge base — not just who wrote it, but who is accountable for it being current today? Do we have a process for updates when strategy or policy changes that updates the source rather than patching the next response? And do we have an immutable audit trail that lets us prove, to a regulator or an institutional client, exactly what the AI produced and exactly what a human authorized? Once those three are solid — what I'd call the Current Answers perimeter, the Ownership layer, and the Audit trail — the expansion roadmap becomes straightforward and the team actually trusts the system enough to use it. Those three pillars map directly to ISO/IEC 42001, the international standard for AI management systems — Pillar 2 satisfies the accountability controls, Pillar 3 satisfies the governance controls, and Pillar 5 satisfies the transparency and measurement requirements. One wrong answer in a DDQ submitted to a $500M prospect can set adoption back by months. The governance architecture is the pre-condition for scale, not an after-thought."*

---

## 7. Client Reporting Modernization

### Kathleen's Reporting Background

Kathleen modernized client reporting at Macquarie using **Vermilion** — a specialized reporting platform widely used in asset management. She transitioned the firm to "efficient, digital-first reporting solutions." This was not a minor upgrade — it was a platform transformation.

### What Vermilion Is

Vermilion (now part of SS&C Technologies) is an enterprise reporting platform built specifically for investment management. It handles:
- **Performance reporting:** GIPS-compliant composite reports
- **Client statements:** Customized portfolio summaries
- **Attribution reports:** Breaking down portfolio performance
- **Regulatory reports:** Mandatory disclosures
- **Automated distribution:** Delivering reports to clients via portal or email on schedule

It is the industry standard alongside platforms like Advent (Geneva), Eagle PACE, and FactSet.

### What Modern Client Reporting Looks Like

**From:** Manual PDF generation, emailed as attachments, updated quarterly, static data, one-size-fits-all format.

**To:** Digital-first portal access, on-demand refresh, interactive charts, customizable views, clear data lineage, mobile-friendly, integrated with CRM so RMs know when clients viewed their reports.

### The Data Lineage Challenge

Every number in a client report must trace back to an authoritative source:
```
Trade execution in portfolio management system (Aladdin)
        ↓
Portfolio accounting system (reconciled positions)
        ↓
Performance calculation engine (composite construction)
        ↓
Reporting platform (Vermilion / PowerBI)
        ↓
Client-facing report
```

If any step in this chain has ambiguity about data ownership or calculation methodology, the report cannot be trusted — by the client or by the regulators.

### What to Say in the Room

> *"When I think about reporting modernization, I start with what the client is actually trying to understand — not what data we have available. The best reporting I've seen starts with the client decision: 'Is my portfolio meeting my objectives?' and works backwards into the data architecture. What does the current reporting experience feel like from a client's perspective — is the feedback more about timing, accuracy, or depth of information?"*

---

## 8. Salesforce as Engagement Orchestration Layer

### Stuart's Salesforce Depth

Stuart has implemented Financial Services Cloud for Asset Management at Macquarie, with focus on Wealth Retail and Alternatives-to-Wealth. He has also built Salesforce for Macquarie Capital (infrastructure asset tracking), WHSE applications on Salesforce, and has deep experience with Salesforce as a platform — not just a CRM.

### The Critical Distinction: Engagement Layer vs. System of Record

**System of Record** = where authoritative data lives (portfolio management system for positions, accounting system for NAV, HR system for employee data).

**System of Engagement** = where relationship management happens, where activities are logged, where the business user interacts with the data.

**Salesforce must be the system of engagement — not the system of record.**

The failure mode is when teams start storing authoritative data in Salesforce because it is convenient — fund performance, NAV, positions. This creates data duplication, ownership confusion, and eventually data quality problems that are expensive to untangle.

### The Right Integration Model

```
Authoritative Data Sources (Portfolio System, Data Lake)
        ↓ (clean, governed API or zero-copy integration)
Salesforce reads data — does not own it
        ↓
Relationship Manager sees unified client view in Salesforce
  → Portfolio performance embedded (read from data layer)
  → Meeting history (owned by Salesforce)
  → Seismic content engagement (fed from Seismic)
  → Open service cases (owned by Salesforce)
        ↓
RM takes action in Salesforce
  → Logs meeting notes
  → Creates follow-up tasks
  → Triggers onboarding workflow
```

### Salesforce Products in the Asset Management Stack

| Product | Purpose | Who Uses It |
|---|---|---|
| **Sales Cloud** | Pipeline management, prospect tracking, mandate lifecycle | Institutional sales, RMs |
| **Service Cloud** | Client servicing, case management, onboarding workflows | Client operations, investor relations |
| **Financial Services Cloud** | Wealth-specific CRM with household, financial goal tracking | Wealth management, advisor relations |
| **Marketing Cloud / Pardot** | Advisor marketing automation, event invitations | Marketing team |
| **Experience Cloud** | Client-facing portal built on Salesforce platform | Clients, advisors |
| **Seismic (integrated)** | Sales content management | Wholesalers, RMs |

### What to Say in the Room (to Stuart)

> *"At First Republic I integrated Salesforce Sales and Service Cloud to give relationship managers a unified view across client portfolio, onboarding status, and servicing history. The most important architectural decision we made was being explicit about what Salesforce owns versus what it reads. Once that boundary is clear and enforced, the platform stays maintainable. Where do you feel that boundary is clearest today, and where has it been harder to hold the line?"*

---

## 9. Seismic: Sales Enablement Platform

### What Seismic Does

Seismic is the leading sales enablement platform in financial services. It manages, personalizes, and distributes sales content to client-facing teams. In asset management, this means:

- **Content library:** Fund fact sheets, pitch decks, market commentary, due diligence documents — all in one governed repository
- **Content governance:** Compliance-approved versions, expiration dates, usage tracking
- **Personalization:** Automatically customizing a pitch deck with the advisor's name, their clients' demographic, and relevant fund data
- **Analytics:** Did the advisor open the fact sheet? How long did they spend on it? This triggers CRM follow-up

### Why This Matters to the Business

A wholesaler meeting with 20 advisors per week cannot manually find and customize the right content for each meeting. Seismic surfaces the right content automatically based on the advisor's profile, client base, and recent interactions. It also ensures the content is always the current, compliance-approved version — not a 6-month-old PDF saved on someone's laptop.

**Stuart built Seismic integration at Macquarie** as part of the sales enablement stack. He will expect you to understand Seismic's role and have a perspective on how it connects to the broader CRM and data ecosystem.

### The Governance Angle

Because Kathleen holds Series 7 and 63 licenses, she understands that content going to clients and advisors is a regulatory matter — not just a UX matter. Seismic's content governance (approval workflows, expiration dates, compliance tagging) is as important as its distribution capabilities.

### What to Say in the Room

> *"Seismic governance is the piece I'd want to understand better — specifically how content ownership and the approval workflow is structured today. The platform is only as valuable as the content currency. If advisors find outdated material, they stop trusting the platform and go back to email. What does the content lifecycle look like from authoring through approval through distribution and retirement?"*

---

## 10. Regulatory Context: Series 7, Series 63 & Compliance

### Why Kathleen's Licenses Matter to You

Kathleen holds Series 7 (General Securities Representative) and Series 63 (Uniform Securities Agent) licenses. This means:

1. She has personal regulatory accountability for client communications
2. She thinks about content governance as a compliance obligation, not just UX
3. She understands that what goes to clients must be reviewed, approved, and tracked
4. She has experienced FINRA examination and knows what regulators look for

**Implication for you:** When Kathleen pushes back on content workflows, portal design, or reporting processes — some of that pushback is regulatory awareness, not just preference. Treat it as a compliance requirement, not an obstacle.

### Key Regulatory Concepts

**FINRA (Financial Industry Regulatory Authority):** The self-regulatory body for broker-dealers in the US. Oversees licensing (Series 7, 63), advertising and communications rules, and examinations.

**SEC (Securities and Exchange Commission):** Federal regulator for investment advisers and investment companies. Asset managers are registered as Investment Advisers with the SEC.

**Investment Adviser Act:** Governs fiduciary duty — the legal obligation to act in the client's best interest. This shapes how performance is reported and how conflicts of interest are disclosed.

**Reg BI (Regulation Best Interest):** SEC rule requiring broker-dealers to act in clients' best interest. Affects how products are recommended and documented.

**GIPS (Global Investment Performance Standards):** Covered in Section 5. Relevant to performance reporting and regulatory examination.

**SOC 2 Type II:** The audit standard for technology platforms handling client data. Your platforms will be assessed against SOC 2 controls — security, availability, processing integrity, confidentiality, and privacy.

### What This Means for Platform Design

Every design decision in client-facing platforms must account for:
- **Audit trail:** Who saw what, when. What was sent to which client, on what date
- **Approved content only:** No ad hoc, unreviewed materials reaching clients or advisors
- **Data accuracy:** Errors in client-facing numbers are regulatory risk, not just UX problems
- **Access controls:** Entitlement management ensures clients only see their own data

---

## 11. Business Glossary

A quick-reference guide to terms you will hear Kathleen and Stuart use naturally.

| Term | Plain English Definition | Platform Relevance |
|---|---|---|
| **AUM** | Assets Under Management — total value of assets the firm manages | Revenue driver; platform investment is justified by protecting/growing AUM |
| **NAV** | Net Asset Value — per-share price of a fund, calculated daily | Core reporting obligation; zero-defect zone |
| **Basis Points (bps)** | 1/100th of 1%. 50 bps = 0.50% | Fees, performance, cost discussions |
| **Composite** | Grouping of similar portfolios for GIPS performance presentation | Core to performance reporting architecture |
| **Benchmark** | Reference index performance is measured against (S&P 500, Bloomberg Agg) | Attribution reporting |
| **Attribution** | Breaking down portfolio performance into its sources | Key client reporting deliverable |
| **Mandate** | When a client awards capital to manage | The win at the end of the institutional sales cycle |
| **Redemption** | Client withdrawing money from a fund | Leading indicator of dissatisfaction; platform affects retention |
| **RFP** | Request for Proposal — prospect asks firm to describe its strategy | AI agent use case |
| **DDQ** | Due Diligence Questionnaire — deep operational questions from prospects | AI agent use case; technology sections are your responsibility |
| **GIPS** | Global Investment Performance Standards — how performance must be reported | Regulatory compliance; data layer must support GIPS composites |
| **Vermilion** | Enterprise reporting platform (SS&C) — Kathleen's specialty | The reporting platform you may inherit or integrate with |
| **Seismic** | Sales enablement platform — Stuart's specialty | Content governance and advisor distribution |
| **SMA** | Separately Managed Account — portfolio managed individually for one client | Common wealth management product |
| **Fund of Funds** | Fund that invests in other funds — Kathleen's TIFF background | Kathleen's operational experience base |
| **CDS** | Credit Default Swap — credit derivative; Kathleen's Credit Suisse specialty | Background context for Kathleen's technical finance knowledge |
| **Aladdin** | BlackRock's portfolio management system, widely used in asset management | May be Nomura's system of record for portfolio data |
| **Bloomberg** | Market data provider — prices, indices, news | Key data source for portfolio analytics |
| **FactSet** | Financial data and analytics provider | Research and analytics data source |
| **Prime Broker** | Provides services to hedge funds (financing, securities lending) | Institutional client context |
| **Series 7** | FINRA license to sell securities | Kathleen holds this — she has regulatory accountability |
| **Series 63** | State securities license | Kathleen holds this — adds state-level regulatory awareness |
| **FINRA** | Financial Industry Regulatory Authority — self-regulatory body | Governs broker-dealer activity, licensing, communications |
| **SOC 2** | Technology audit standard for data security and availability | Your platforms will be audited against this |
| **Entitlement** | Authorization controlling which client can access which data | Core platform governance function |
| **RM** | Relationship Manager — manages the institutional client relationship | Primary internal user of CRM and reporting tools |
| **Wholesaler** | Internal sales person who calls on financial advisors | Primary user of Seismic and Salesforce in advisor channel |
| **Fact Sheet** | One-two page fund summary — performance, strategy, holdings | Core Seismic content asset |
| **KIID** | Key Investor Information Document — European regulatory disclosure | International fund distribution context |

---

## 12. Partnership Principles: Product + Technology

### How to Position the Technology-Product Relationship

The Head of Client Technology is **not a service provider** to Kathleen and Stuart. The relationship is a **peer partnership** where:

- Product (Kathleen, Stuart) defines desired outcomes and client value
- Technology (you) evaluates feasibility, scalability, long-term implications
- Both parties share accountability for delivery and business results

### The Partnership Model

| Situation | Wrong Response | Right Response |
|---|---|---|
| Product requests a feature that creates tech debt | Block it or silently comply | Make the tradeoff explicit: "We can do this in 6 weeks with a repayment plan, or 12 weeks done correctly. Your call." |
| Architecture conflicts with roadmap timeline | Enforce standards rigidly | Offer sequencing options that balance speed with long-term maintainability |
| Business pushes for AI before governance is ready | Say no | Say "here's what we need to make this trustworthy — let's build that in parallel" |
| Product wants a new platform added to the stack | Resist to protect simplicity | Evaluate objectively: does it solve a problem no existing platform solves? |

### What Kathleen Needs From You

- **AI partnership:** Extend and govern the DDQ/RFP agent she built. Don't redesign it — evolve it
- **Reporting expertise:** Understand Vermilion's role and bring data architecture that makes reporting trustworthy
- **Ops credibility:** She came from operations. She will respect someone who understands operational workflow — not just technology
- **Human-centered perspective:** She has facilitated design workshops. She values solutions built around user needs, not technical elegance

### What Stuart Needs From You

- **Salesforce governance:** Protect the platform integrity he has built while enabling product requests
- **Roadmap discipline:** He manages $22M budgets. He needs a technology partner who sequences investments logically and delivers on commitments
- **Platform leverage thinking:** He evaluates initiatives by what platform capability they create for future work — not just their standalone value
- **Delivery credibility:** He has led teams of 14. He needs to know you can run an engineering organization that delivers

---

## 13. KPIs That Matter to This Business

### Client Experience KPIs

| KPI | Why It Matters | Technology Lever |
|---|---|---|
| **RFP/DDQ response time** | Faster response = competitive advantage in winning mandates | AI agent reducing drafting time from days to hours |
| **Client portal adoption rate** | Higher adoption = less manual servicing, higher satisfaction | UX quality, data accuracy, report timeliness |
| **Report delivery SLA** | Late reports erode institutional client trust | Reporting platform automation and data pipeline reliability |
| **Client onboarding time** | First impression of operational quality | Entitlement automation, workflow orchestration |
| **Advisor content engagement** | Measures distribution effectiveness | Seismic adoption, content currency, analytics |

### Distribution KPIs

| KPI | Why It Matters | Technology Lever |
|---|---|---|
| **Sales cycle length** | Shorter cycle = faster AUM growth | AI meeting prep, streamlined DDQ process |
| **Advisor penetration rate** | How many target advisors are actively using Nomura content | Seismic analytics, CRM tracking |
| **Mandate pipeline velocity** | Speed from prospect to mandate award | CRM workflow automation |
| **RFP win rate** | Quality of proposal response affects win probability | AI agent accuracy and consistency |

### Platform Operational KPIs

| KPI | Why It Matters | Technology Lever |
|---|---|---|
| **NAV delivery accuracy** | Zero tolerance for errors | Data pipeline governance |
| **Platform uptime** | Downtime affects client and advisor trust | SRE practices, monitoring |
| **Data freshness** | Stale data in reporting erodes confidence | Real-time data pipelines |
| **Run cost per AUM** | Technology cost efficiency | FinOps discipline, architecture choices |

---

## 14. Questions for the Panel

### Questions for Kathleen (Client Experience, AI, Reporting)

**On AI and DDQ/RFP:**
> *"As you think about scaling the AI agent beyond DDQ and RFP — where does the governance model need to mature first before you extend it to new use cases? I want to understand what you've learned about what makes adoption sustain past the initial launch."*

**On client reporting:**
> *"When you think about the current reporting experience — not from a platform perspective, but from how it actually feels to an institutional client or their operations team — where is the biggest gap between what they expect and what we're delivering today?"*

**On client experience:**
> *"Where do you feel the most friction today between what you want to deliver for clients and what the current platform architecture allows? I'd like to understand where the gap is most acute."*

---

### Questions for Stuart (Platform, Salesforce, Roadmap)

**On Salesforce governance:**
> *"How do you currently think about the integration boundary between what lives in Salesforce versus what gets handled in the upstream data layer? I want to make sure my instincts on CRM architecture align with how the platform is structured here."*

**On roadmap:**
> *"Where in the current roadmap are the initiatives that are clearly sequenced, and where are the ones where prioritization is still genuinely open? I want to understand where there's room to bring a fresh perspective versus where I should be building on what's already decided."*

**On partnership:**
> *"What would success look like for you at the six-month mark — not in terms of deliverables, but in terms of how the technology and product relationship is working day to day?"*

---

---

## 15. Building Trust, Relationship & Influence

> *Trust is the operating currency of an Executive Director. It is not built in a single conversation — it is built through consistency, transparency, and demonstrated judgment over time. In a peer-level partnership with Kathleen and Stuart, influence without authority is the only kind that matters.*

### Why This Is a Distinct Competency for This Role

The Head of Client Technology does not manage Kathleen or Stuart. There is no reporting line that gives this role authority over product decisions. The only instrument of influence is credibility — established through four behaviors that distinguish trusted partners from capable technologists.

---

### The Four Trust Behaviors — Mapped to Nomura Scenarios

#### Trust Behavior 1: Deliver What You Promise, Exactly When You Say

At the Executive Director level, the baseline trust contract is: **what you commit to is what gets done, when you said it would**. Not approximately. Not mostly. Every exception requires advance communication, not post-hoc explanation.

**Nomura scenario — Reporting SLA commitment:**
If you commit to Kathleen that the reporting pipeline will deliver NAV data to the portal by 6:00 AM EST, and on day 14 it arrives at 7:30 AM without warning, you have made a withdrawal from the trust account. The technical reason is irrelevant. The failure is in communication, not in the pipeline.

**The counterplay:** Build a 15-minute early-warning alert into every reporting SLA. If the pipeline is running late, Kathleen's operations team knows before clients notice. This transforms a potential trust failure into a trust deposit — you told them before they had to ask.

**Rule:** *Never let a stakeholder discover a problem before you tell them about it.*

---

#### Trust Behavior 2: Make Tradeoffs Visible — Never Hide Technical Complexity

Business partners lose trust in technologists who surface only good news, or who present solutions without naming the tradeoffs. Kathleen and Stuart have enough platform experience to know that every technical decision involves a tradeoff. If you don't name it, they assume you didn't see it — or worse, that you are managing them.

**Nomura scenario — AI agent scaling request:**
Kathleen asks: *"Can we extend the AI agent to cover client-facing Q&A on the portal by Q3?"* The wrong answer: *"Yes, we can look at that."* The right answer:

> *"We can — and here is what the tradeoff looks like. Extending to client-facing Q&A requires a different content governance model than DDQ/RFP, because the client audience is broader and the regulatory bar for accuracy is higher. We would need to solve three things before Q3: (1) define the scope of questions the agent can answer vs. what routes to a human, (2) establish the compliance review process for new content types, and (3) build a confidence-threshold routing mechanism so uncertain answers go to a human in real time. If we start now, Q3 is achievable for a pilot — but not production scale. Do you want to sequence it that way?"*

This response builds trust because it demonstrates that you have thought through the problem further than the question asked, and you are giving Kathleen the information she needs to make a good decision — not just a yes or a no.

**Rule:** *Always show your working. Never give a clean answer to a complex question without naming what made it complex.*

---

#### Trust Behavior 3: Protect Partners in the Room and Behind Closed Doors

Trust is damaged when partners discover that you said something different in a different room. At the Executive Director level, consistency between what you say to Kathleen and Stuart directly, what you say to your engineering team, and what you say to the broader organization is a non-negotiable.

**Nomura scenario — Roadmap prioritization conflict:**
Stuart has championed a Salesforce enhancement that requires 6 weeks of engineering capacity. Your engineering lead tells you privately that the enhancement will create an integration dependency that will slow down a different initiative by 4 weeks. The wrong response: align with Stuart in the room, then quietly slow down the other initiative and hope he doesn't notice. The right response:

> *"Stuart, my engineering team has identified something I want to bring to you directly. The Salesforce enhancement we discussed is achievable, but it creates a shared dependency with the ADX onboarding pipeline — which means the onboarding initiative would slip by 4 weeks. I want to put both options on the table: we can prioritize the Salesforce enhancement and slip onboarding, or we can sequence them differently. This is your call to make — I just want you to make it with the full picture."*

This behavior demonstrates that you do not hide information to preserve your own position or avoid difficult conversations. It builds the kind of trust that sustains through hard moments — not just easy ones.

**Rule:** *What you say in the room must be what you say outside it. Consistency is the operating definition of integrity at this level.*

---

#### Trust Behavior 4: Invest in Understanding Before Prescribing Solutions

The fastest way to erode trust with an experienced operator like Kathleen is to arrive with solutions before you have demonstrated that you understood the problem. She has managed complex operational workflows — DDQ processes, reporting pipelines, member services at TIFF. She will recognize immediately whether your proposed solution reflects genuine understanding of her operational context or a generic technology pattern applied without that understanding.

**Nomura scenario — First 30 days:**
Resist the instinct to arrive with a platform assessment or improvement proposal in week 2. Instead, invest the first 30 days doing three things: (1) shadow a DDQ response cycle end-to-end, (2) sit with an RM while they prepare for a client reporting call, and (3) listen to a client service phone call where an advisor has a question about a holding. Then bring your observations — not your prescriptions — to Kathleen and Stuart.

> *"After the first 30 days, here are the three things I observed that I didn't expect — and I'd like to understand whether they represent gaps or intentional design decisions before I have a view on what to change."*

This behavior signals humility without passivity. It positions you as someone whose assessment can be trusted because it is grounded in observed reality, not imported assumptions.

**Rule:** *Demonstrate that you understood the problem before you describe the solution. The sequence matters.*

---

### Influence Without Authority — The ED-Level Model

At Executive Director level, you influence peers (Kathleen, Stuart) through five levers — not organizational hierarchy:

| Influence Lever | How It Works | Nomura Application |
|---|---|---|
| **Expert Credibility** | Your architectural and technical judgment is trusted because it has been accurate | When you say the Snowflake zero-copy integration is the right model, people accept it because prior recommendations proved out |
| **Business Fluency** | You speak the language of AUM, RFP win rates, advisor penetration — not just systems | Kathleen and Stuart include you in business strategy conversations because you add value there |
| **Anticipatory Thinking** | You see problems before they become crises and raise them proactively | You tell Stuart that a Salesforce governor limit will become a constraint in 6 months before it blocks a delivery |
| **Coalition Building** | You build relationships laterally — with compliance, legal, operations — so platform decisions have natural allies | When you propose a change to the DDQ workflow, the compliance team is already aligned because you've been in dialogue with them |
| **Track Record** | Every delivered commitment builds the next conversation's credibility | After the onboarding automation delivers in under 2 minutes as promised, the budget conversation for the next initiative is easier |

---

### Building Capital with Kathleen — Specific Playbook

**What builds trust with Kathleen:**
- Understand her AI governance work at Macquarie at a deep enough level to extend it — not re-explain it to her
- Demonstrate operational credibility: she came from middle office and member services, not IT. She respects people who understand operational workflow complexity from the inside
- Keep client experience at the centre of every technology conversation. She is not evaluating the elegance of your architecture — she is evaluating whether it makes clients feel valued and served
- Be direct. She has Series 7 and 63 licenses — she is accountable for client communications. She does not have time for hedged, qualified answers that don't make a recommendation
- Bring her problems before she discovers them

**What damages trust with Kathleen:**
- Proposing AI capabilities that don't have a content governance model behind them
- Presenting reporting solutions that are technically correct but ignore what the client is actually trying to understand from the report
- Using jargon that distances technology from the client outcome
- Reconfiguring the DDQ/RFP agent she built at Macquarie without first deeply understanding why it was built the way it was

---

### Building Capital with Stuart — Specific Playbook

**What builds trust with Stuart:**
- Demonstrate that you understand the Salesforce platform architecture at a depth that allows you to evaluate proposals — not just accept or reject them
- Show roadmap discipline: manage to the sequence, communicate variance early, close delivery gaps
- Use his language: platform leverage, roadmap velocity, foundation investments vs. feature delivery
- Reference the work he has done at Macquarie specifically — Seismic integration, AI pre-meeting prep, Financial Services Cloud configuration — as the foundation you are building on, not starting over
- Be explicit about the budget implications of technical decisions. He manages $22M budgets. He needs a technology partner who connects architecture choices to cost outcomes

**What damages trust with Stuart:**
- Making architectural decisions that affect his platforms without consulting him
- Introducing new platforms to the stack without demonstrating that they solve a problem no existing platform solves
- Overpromising delivery dates and under-delivering without advance communication
- Treating his roadmap initiatives as technical requests rather than business investments

---

## 16. Early Alignment: Business Intent & Technical Constraints

> *Product partnerships that align early on business intent and technical constraints are more successful because they create a shared, strategic foundation — leading to smoother collaboration and stronger long-term platform decisions. By integrating these elements at the start, companies avoid the pitfalls of "bolted-on" integrations, ensuring technical capabilities directly support business goals.* — LinkedIn / Harvard Business Review, 2023

### Why Early Alignment Fails in Asset Management Technology

In the absence of structured early alignment, the typical failure pattern in asset management client technology looks like this:

1. Product identifies a client need and proposes a solution (e.g., *"we should add real-time portfolio Q&A to the client portal"*)
2. Technology scopes it in isolation, either discovering too late that it conflicts with existing architecture or delivering something that doesn't match what product envisioned
3. The delivery disappoints. Both sides blame the other for misaligned expectations
4. The next initiative starts with less trust than this one did

The alternative is a structured early alignment model — executed before any solution is proposed, scoped, or committed to.

---

### The Three-Pillar Early Alignment Framework

#### Pillar 1: Aligning Business Intent

Before any technical discussion, product and technology must define — in writing — the shared business objective the initiative is meant to serve.

**The five questions that define business intent:**
1. Which client segment does this serve, and what does success look like from *their* perspective?
2. Does this initiative protect existing AUM, grow new AUM, or reduce operating cost — and by approximately how much?
3. What is the current experience that this replaces, and what specifically is wrong with it?
4. What is the minimum viable outcome that would constitute success for this initiative?
5. What does failure look like — and who bears the consequence if this doesn't land?

**Nomura example — Private credit reporting:**
Business intent statement: *"Institutional clients investing in Nomura's private credit strategy need monthly cash flow reports and loan-level transparency in a format that their operations teams can reconcile with their own portfolio management systems. Currently, this is delivered via email attachment with a 10-day delay. The initiative must reduce delivery to 3 days and provide portal-based access."*

This statement gives technology everything it needs to scope the initiative correctly: the client segment (institutional), the data requirement (loan-level), the format constraint (reconcilable), the SLA (3 days), and the delivery channel (portal). No ambiguity. No assumption.

---

#### Pillar 2: Defining Technical Constraints

Equally important: technology must surface architectural limitations, security requirements, and platform dependencies *before* the business has committed to a timeline or a client-facing promise.

**The five technical constraints that must be documented at initiative start:**
1. **Data residency and sovereignty**: Where does the underlying data live, and what jurisdictional constraints govern its movement?
2. **Platform dependencies**: Which existing systems does this touch, and what are their current capacity and change-management constraints?
3. **Security and access control**: What entitlement model governs who can see this data, and at what granularity?
4. **Compliance and audit requirements**: What logging, retention, and access audit obligations apply to this data or user interaction?
5. **Performance and scalability envelope**: What are the expected peak-load conditions, and does the current architecture support them without redesign?

**Nomura example — AI agent extension to client-facing Q&A:**
Technical constraints surfaced early: (1) the current knowledge base is scoped for internal DDQ/RFP use — client-facing content requires a different compliance review workflow; (2) LLM API rate limits become a constraint at portal scale — caching and routing architecture must be designed for; (3) client PII cannot flow through the LLM API without a DPA with the provider; (4) all AI-generated responses to clients require an audit trail for FINRA examination; (5) confidence-threshold routing logic does not yet exist and requires 6 weeks of engineering development.

Surfacing these constraints early prevents the business from committing to a Q2 launch that the platform cannot support.

---

#### Pillar 3: Integrating into the Product Roadmap

Successful partnerships treat integration as a **core product decision**, prioritizing it on the product roadmap rather than maintaining it as a separate partner or engineering backlog.

**What this means in practice at Nomura:**

| Initiative | Business Intent | Technical Prerequisite | Roadmap Sequence |
|---|---|---|---|
| Private Credit Portal Reporting | Loan-level transparency for institutional clients | Multi-cadence data pipeline (separate from public market feed) | Q2: pipeline; Q3: portal |
| AI Agent Extension — Client Q&A | Self-service portal Q&A for advisors | Compliance review workflow, PII DPA, confidence routing | Q3: governance design; Q4: pilot |
| Salesforce 360 Client View | Unified RM view: performance + service + meetings | Zero-copy integration from data lake to SF (read-only) | Q1: data contract; Q2: SF embed |
| Seismic Content Auto-Refresh | Current, compliant content always surfaced | Content API from product publishing system to Seismic | Q2: API build; Q3: Seismic sync |
| Onboarding Automation Phase 2 | Sub-2-minute entitlement for new strategies | Step Functions extension + new ADX dataset provisioning | Q1: design; Q2: delivery |

This table is the artifact that early alignment produces. Both product (Kathleen, Stuart) and technology share it, own it, and use it to sequence conversations with leadership.

---

### The Discovery Session — Structure and Agenda

Every significant initiative should begin with a structured discovery session involving product and technology jointly. This is not a requirements-gathering meeting — it is a shared problem-definition exercise.

**Discovery Session Agenda (90 minutes):**

```
Part 1 — Business Intent (30 min)
  - What client or distribution problem are we solving? (15 min)
  - What does success look like, and how do we measure it? (10 min)
  - What happens if we do nothing? (5 min)

Part 2 — Technical Constraints Inventory (30 min)
  - What platform surfaces are touched? (10 min)
  - What data, security, or compliance constraints are in play? (10 min)
  - What is currently unknown that could affect scope or timeline? (10 min)

Part 3 — Shared Sequencing (30 min)
  - Given business intent and technical constraints, what is the earliest credible delivery? (15 min)
  - What decisions need to be made by whom to unlock progress? (10 min)
  - What does the team agree to document and circulate within 48 hours? (5 min)
```

**Output artifacts (must be produced within 48 hours of the session):**
- Business intent statement (1 paragraph, agreed by product and technology)
- Technical constraints register (5–7 items, each owned by a named individual)
- Draft roadmap position (initiative placed on the shared roadmap with prerequisites named)
- Open decision log (decisions that must be made, by whom, by when)

---

### Documentation Discipline — Replacing Verbal Agreements

A 2023 Harvard Business Review survey found that the most common source of misalignment between leadership and implementation teams is reliance on verbal agreements made in strategy meetings that are never written down. The solution is a documentation discipline that is light enough to be maintained but structured enough to remove ambiguity.

**The three documents that govern every Nomura platform initiative:**

**Document 1 — Initiative Charter (1 page)**
Owner: Product Manager (Kathleen's team)
Content: Business intent, success metrics, client segment, AUM impact estimate, go/no-go criteria
Purpose: Everyone working on this initiative reads this before writing a line of code

**Document 2 — Technical Constraints & Architecture Decision Record (ADR)**
Owner: Head of Client Technology (you)
Content: Platform dependencies, data residency, security model, performance envelope, open decisions with resolution dates
Purpose: The record of the technical considerations that shaped the solution approach — consulted during delivery and referenced during SOC 2 / DDQ review

**Document 3 — Shared Roadmap Entry**
Owner: Joint (Product + Technology)
Content: Initiative name, business intent (1 line), technical prerequisites, delivery phase, budget owner, success KPI
Purpose: The single source of truth for sequencing conversations with the CTO and Nomura leadership

---

### Governance Cadence — Sustaining Alignment After the Discovery Session

Early alignment is not a one-time event. It requires a structured cadence to detect drift before it becomes a delivery problem.

| Cadence | Frequency | Participants | Agenda |
|---|---|---|---|
| **Product-Technology Weekly Sync** | Weekly (30 min) | You + Product leads | Open decisions, blockers, dependency updates |
| **Roadmap Review** | Bi-weekly (60 min) | You + Kathleen + Stuart | Initiative progress, sequencing changes, new intake |
| **Architecture Review Board** | Monthly (90 min) | You + engineering leads + platform architect | Proposed significant changes to platform architecture |
| **Strategic Alignment Session** | Quarterly (half-day) | You + Kathleen + Stuart + CTO | AUM impact review, platform investment sequencing, next-quarter priorities |
| **DDQ/Compliance Review** | Per-initiative | You + Kathleen + Legal/Compliance | Technology DDQ sections updated to reflect platform changes |

**The rule that prevents governance decay:** Every open decision has a named owner and a resolution date in the shared open decision log. If a decision misses its date, it automatically escalates to the roadmap review. No silent slippage.

---

## 17. Product-Technology Collaboration Model

> *The goal is not for technology to serve product, or for product to defer to technology. The goal is a collaborative model where each function brings its domain expertise to a shared problem, and the solution is better because both perspectives were applied together.*

### Role Division — The Clarity Model

Confusion about role boundaries is the single largest source of friction in product-technology partnerships. The following model makes the division explicit — and sustainable.

```
+------------------------------------------------------------------+
|                     SHARED ACCOUNTABILITY                        |
|          Business Outcome · Client Value · Delivery              |
+----------------------------+------------------------------------+-+
                             |
       +---------------------+----------------------+
       |                                            |
+------v-----------------+           +--------------v--------------+
|   PRODUCT MANAGER      |           |  HEAD OF CLIENT TECHNOLOGY  |
|  (Kathleen / Stuart)   |           |          (You)              |
+------------------------+           +-----------------------------+
| • Product vision       |           | • Solution architecture     |
| • Business case        |           | • Data architecture         |
| • Client requirements  |           | • Infrastructure/cloud arch |
| • Product architecture |           | • Platform governance       |
| • Roadmap sequencing   |           | • Build/buy/integrate eval  |
| • Stakeholder mgmt     |           | • Engineering org design    |
| • Success metrics      |           | • Technical risk management |
| • Compliance intent    |           | • SOC 2 / security posture  |
| • UX and content       |           | • FinOps / run cost         |
|   governance           |           |                             |
+------------------------+           +-----------------------------+
       |                                            |
       +---------------------+----------------------+
                             |
              +--------------v--------------+
              |       COLLABORATIVE ZONE    |
              | Requirements clarification  |
              | Solutioning workshops       |
              | Vendor evaluation           |
              | Architecture trade-offs     |
              | Roadmap sequencing          |
              | Incident response           |
              +-----------------------------+
```

**The rule for the Collaborative Zone:** Decisions made in this space are owned jointly. Neither product nor technology can override the other unilaterally. If consensus cannot be reached, it escalates to a named decision-maker — agreed in advance.

---

### What Technology Always Brings to Product Discussions

To make collaboration productive, technology must consistently contribute the five questions that product discussions often skip:

**The Five Technology Clarifying Questions:**

1. **Who owns the data, and where does it live?**
   Product proposes a feature involving portfolio performance data. Technology asks: is this data from Aladdin, Bloomberg, the reporting platform, or the data lake — and have we established a governed integration for this source?

2. **What is the failure state, and who does it affect?**
   Product proposes automated NAV delivery to the portal. Technology asks: when the pipeline is late or delivers a reconciliation error, what is the client experience — and what is the RM communication protocol?

3. **Is this a one-time requirement or a platform capability?**
   Product requests a custom report for a specific institutional client. Technology asks: is this a one-off or the first instance of a capability we need to build properly for the platform? Answering this determines whether we build a bespoke solution or extend the reporting engine.

4. **What does compliance need to know about this?**
   Product proposes extending Seismic to a new content type. Technology asks: has compliance been involved in defining the approval workflow for this content category? What are the FINRA implications for how this is tagged and tracked?

5. **What breaks if this changes downstream?**
   Product wants to modify the Salesforce opportunity model. Technology asks: what downstream processes depend on the current model — Step Functions workflows, ADX entitlement triggers, reporting queries — and what is the change impact?

These questions are not blockers. They are the infrastructure of a good decision. Bringing them consistently establishes technology as a thoughtful partner — not a gatekeeper.

---

### Collaborative Solutioning — Worked Example: Seismic Content Discovery Problem

**The business problem (as presented by Stuart):**
> *"Wholesalers are spending 20–30 minutes before each advisor meeting finding and manually customizing the right content. Seismic is supposed to solve this, but adoption is low because the content surfacing isn't matching what advisors actually need. Can we fix it?"*

**Step 1 — Technology's initial clarifying questions (not solutions):**
- What signals are currently used to surface content in Seismic — is it advisor geography, AUM tier, fund holdings, or meeting history?
- Is the problem that the wrong content is suggested, or that the right content exists but isn't findable?
- Do wholesalers have time in their workflow to provide feedback on suggestions, or does any fix need to be zero-friction at the point of use?
- Is Salesforce meeting data (topic, agenda, client stage) currently flowing into Seismic's recommendation engine?

**Step 2 — Joint problem redefinition (product + technology together):**
> *"The content surfacing problem is actually a data signal problem. Seismic is surfacing content based on advisor tier alone. It does not have access to meeting topic, advisor's recent fund discussions, or client AUM profile from Salesforce. The fix is a richer data integration between Salesforce meeting records and Seismic's recommendation API — not a Seismic configuration change."*

**Step 3 — Technology's solution options (with tradeoffs named):**

| Option | Description | Time to Deliver | Tradeoff |
|---|---|---|---|
| **Option A** | Push Salesforce meeting topic + client AUM tier to Seismic via existing API | 4 weeks | Only uses existing data signals; limited personalization depth |
| **Option B** | Build a recommendation micro-service aggregating Salesforce + fund performance + content engagement history feeding Seismic | 12 weeks | Richer signals; requires new engineering capability and ongoing maintenance |
| **Option C** | Augment Option A with a wholesaler-facing feedback button ("this content was right / wrong") feeding a retraining loop | 6 weeks total (A + 2 weeks) | Best long-term quality; requires product to manage the feedback data governance |

**Step 4 — Product makes the decision (technology enables it):**
Stuart selects Option C: start with Option A for quick adoption improvement, add the feedback loop in week 6 for continuous improvement. Technology owns the API integration and data pipeline. Product owns the Seismic content taxonomy and the feedback governance model.

**Step 5 — Shared success metric defined:**
> *"Wholesaler pre-meeting prep time reduced from 20–30 minutes to under 5 minutes for 80% of meetings, measured 90 days after deployment. Seismic opens from suggested content increase by 40% vs. baseline."*

This is collaborative solutioning: product frames the problem and owns the success definition; technology frames the solution options and owns the delivery; both share accountability for the outcome.

---

### Helping Product Understand Requirements in More Detail

One of the most valuable contributions technology can make to product is helping product managers articulate requirements with the precision that architecture and engineering need — without doing product management for them.

**The requirement clarification framework — five lenses:**

| Lens | Product's Natural Language | Technology's Clarifying Prompt |
|---|---|---|
| **User** | "Advisors need to find content faster" | "Which advisors — all, or a specific tier? In what workflow — pre-meeting, during meeting, post-meeting?" |
| **Data** | "Show portfolio performance on the portal" | "Which performance calculation — time-weighted, money-weighted, or both? Gross fees, net fees, or both? Against which benchmark?" |
| **Timing** | "Clients want up-to-date data" | "What does 'up-to-date' mean for this use case — same-day NAV, real-time intraday, or as-of prior close?" |
| **Exception** | "Make reporting more reliable" | "What is currently unreliable — the data, the timing, the calculation, or the presentation? What does the failure look like to the client?" |
| **Scale** | "We need this to work for all clients" | "How many concurrent sessions? What is the distribution between institutional and wealth? What are the peak periods — end of quarter, January tax season?" |

Using these lenses systematically transforms vague product requests into scoped, buildable requirements — without turning product conversations into engineering specification meetings.

---

## 18. Vendor Application Evaluation Framework

> *Vendor evaluation in asset management technology is not a procurement exercise — it is a platform decision. The platform you choose today creates the integration dependencies, data governance constraints, and vendor relationships that shape the next five years of delivery capacity. Evaluating a vendor wrong costs more than the license fee.*

### Why Vendor Evaluation Requires Product-Technology Partnership

The most common vendor evaluation failure mode in asset management technology is **sequential evaluation** — where product evaluates a vendor for business fit, selects it, and then hands it to technology for integration scoping. By the time technology surfaces architectural constraints, the business has already formed a relationship with the vendor's executive team, and walking it back is politically costly.

The alternative is **parallel, joint evaluation**: product and technology evaluate simultaneously using separated but equally weighted criteria, and no selection is made until both lenses are cleared.

---

### The Joint Evaluation Framework

#### Product-Weighted Criteria (Kathleen & Stuart evaluate)

| Criterion | Weight | What to Assess |
|---|---|---|
| **Business capability fit** | 25% | Does the vendor solve the specific business problem better than alternatives? |
| **User experience for target persona** | 20% | Would an RM, wholesaler, or client actually use this without training friction? |
| **Content governance model** | 15% | Does the vendor's compliance tagging and content control match our regulatory requirements? |
| **Client channel alignment** | 15% | Is this platform designed for institutional, wealth, or both — and does it serve our mix? |
| **Vendor roadmap and stability** | 15% | Is the vendor investing in the capabilities we need over the next 3 years? |
| **Reference customer quality** | 10% | Are there comparable asset managers using this successfully? |

#### Technology-Weighted Criteria (You evaluate)

| Criterion | Weight | What to Assess |
|---|---|---|
| **API and integration architecture** | 25% | Does the vendor expose clean, documented APIs? Is there a published integration pattern for our stack (Salesforce, Snowflake, ADX)? |
| **Data ownership and residency** | 20% | Who owns the data stored in the vendor platform? Can we extract it? What are the residency implications for EU client data? |
| **Security and compliance posture** | 20% | SOC 2 Type II certified? GDPR-compliant? Penetration tested? Can they provide evidence, not just attestation? |
| **Platform scalability envelope** | 15% | What are the performance SLAs? What is the degradation model under peak load? |
| **Vendor lock-in risk** | 10% | What is the exit cost if we replace this vendor in 3 years? Is the data portable? Are integrations proprietary? |
| **Total cost of ownership** | 10% | License cost + integration engineering + ongoing maintenance + opportunity cost of team hours required |

---

### The Phased Evaluation Process

```
Phase 1 — Intake & Shortlisting (Weeks 1–2)
  Product: Define business requirements, success criteria, non-negotiables
  Technology: Define architecture requirements, integration constraints, security baseline
  Joint: Agree on longlist (4–6 vendors); agree on evaluation scoring model

Phase 2 — Discovery & RFI (Weeks 3–4)
  Joint: Send structured RFI to all vendors with identical business AND technical questions
  Product: Score business capability responses
  Technology: Score architecture and security responses
  Joint: Shortlist to 2–3 vendors for proof of concept

Phase 3 — Proof of Concept (Weeks 5–8)
  Product: Define 2–3 representative business scenarios for PoC
  Technology: Define integration test: can this vendor connect to our SF/Snowflake/ADX
              stack in a sandbox?
  Joint: Score PoC results against pre-agreed criteria (no moving the goalposts)

Phase 4 — Reference Checks & Commercial Negotiation (Weeks 9–10)
  Product: Reference calls with business users at comparable asset managers
  Technology: Architecture review call with vendor engineering; security questionnaire review
  Joint: Commercial terms reviewed with legal; data ownership and exit rights
         explicitly negotiated

Phase 5 — Decision & Documentation (Week 11)
  Joint: Present scored results to CTO and business leadership
  Product: Business recommendation with success metrics
  Technology: Architecture recommendation with integration plan and risk register
  Joint: Decision documented in ADR — rationale, alternatives considered,
         tradeoffs accepted
```

**Timing principle:** No phase is skipped, even under timeline pressure. The Phase 3 PoC is the single most important phase — it reveals integration complexity that neither vendor claims nor RFI responses will disclose.

---

### Current Nomura Platform Evaluation Posture

| Platform Category | Current State | Evaluation Trigger | Key Technology Question |
|---|---|---|---|
| **Client Portal** | Existing portal (vendor TBD) | Client experience gap, institutional self-service demand | Does the portal vendor expose a clean API for embedding Vermilion/PowerBI reporting? |
| **AI Agent Platform** | Macquarie-built LLM agent (Kathleen) | Scaling to new use cases, multi-model governance | Does the current platform support model versioning, confidence scoring, and SOC 2-level audit logging? |
| **Reporting Platform** | Vermilion (SS&C) candidate | Reporting modernization initiative | Can it support private credit valuation cadences alongside public market data? |
| **Sales Enablement** | Seismic (Stuart) | Content discovery improvement | Does Seismic's recommendation API accept external signal enrichment (Salesforce meeting data)? |
| **Data Platform** | ADX + Snowflake candidate | Multi-asset reporting aggregation, GIPS composite engine | Can Snowflake zero-copy sharing serve as the integration layer between portfolio system, Salesforce, and reporting simultaneously? |

---

### Vendor Evaluation Pitfalls — and How to Avoid Them

| Pitfall | How It Manifests | Prevention |
|---|---|---|
| **Sequential evaluation** | Product selects, technology inherits the integration problem | Establish joint evaluation from intake — never sequential |
| **Demo-driven selection** | Vendor demo looks impressive; integration reality is hidden | Require a working PoC in your sandbox before shortlisting |
| **Lowest-friction wins** | The vendor with the best sales relationship gets selected | Score against pre-agreed criteria; separate relationship from capability |
| **License cost dominates** | TCO not calculated; integration cost hidden | Always calculate 3-year TCO: license + integration engineering + maintenance |
| **Data ownership not negotiated** | Vendor owns client data by default in SaaS contract | Negotiate data ownership and exit rights before signing — not as an amendment |
| **Compliance not in the room** | Security and regulatory requirements discovered after selection | Include compliance/legal in the Phase 2 shortlisting decision |
| **Reference calls too late** | References checked after decision is effectively made | Reference calls in Phase 4, before commercial terms are finalized |
| **Vendor roadmap not stress-tested** | Promised roadmap items never delivered | Ask for the last 3 roadmap commitments and whether they shipped on time |

---

### What to Say in the Room (on Vendor Evaluation)

**To Kathleen:**
> *"My evaluation model separates business fit from technical fit deliberately. I don't want a technology preference to override a genuine business capability advantage — or vice versa. I'd want your team to evaluate the user experience and content governance dimensions in parallel with my architecture review, and then we compare notes before either of us has a preference. That way the selection reflects the full picture, and we both own the decision."*

**To Stuart:**
> *"The one thing I always insist on is a working PoC in a sandbox that touches our actual integration surfaces — Salesforce, Snowflake, and ADX. A vendor demo in their environment tells us nothing about the 12 weeks of integration engineering we'll face in ours. If a vendor won't commit to a PoC, that's useful signal in itself. What has your experience been with vendor PoCs at Macquarie — where have they revealed integration complexity that the demo didn't show?"*

---

## 19. Asset Management Content Distribution & Digital Asset Management

> *Content distribution in asset management is not a marketing function — it is a client experience function. Every fund fact sheet, performance attribution report, regulatory disclosure, and thought leadership piece that reaches an advisor or institutional client is a direct touchpoint with the brand. The platform that governs that content governs the quality of the relationship.*

### Why Content Distribution Is a Technology Leadership Responsibility

In most asset managers, content distribution is treated as a marketing operations problem. The mistake is in that categorisation. At Nomura Asset Management International, the Head of Client Technology owns the platforms through which content reaches advisors, wealth clients, and institutional investors — Seismic, the client portal, Salesforce engagement templates, and the reporting layer. Each of these is a content distribution surface. The discipline that governs them is **Digital Asset Management (DAM)**.

DAM is not a single product — it is an architectural capability: the ability to centralise, version-control, govern, and distribute approved marketing materials, reports, presentations, and multimedia content across all channels and all audiences, with compliance assurance built into the workflow.

---

### The Five Pillars of Content Distribution in Asset Management

#### Pillar 1: Centralised Control — Single Source of Truth

The foundational problem DAM solves is **scattered, ungoverned content** — fund fact sheets living in individual advisor email attachments, performance attribution reports stored on SharePoint folders with inconsistent versioning, regulatory disclosures updated by compliance without a distribution workflow that ensures the old version is retired.

A DAM system (Aprimo, Bynder, OpenText Media Management, Acquia/Widen, Seismic Content Hub) provides:
- A single authoritative repository for all approved brand and investment content
- Version control that automatically retires superseded materials
- Metadata tagging by asset class, audience type (institutional/wealth/advisor), regulatory jurisdiction, and content category
- Access permissions aligned to distribution role (internal RM, external advisor, institutional client portal)

**Nomura application:** Today, Seismic functions as the sales enablement layer of DAM — it governs what content wholesalers and RMs use in client-facing interactions. The gap is that Seismic is not a complete DAM: it does not govern the full content lifecycle from production (investment writing, compliance review, design) to distribution. Integrating a upstream DAM (or extending Seismic's content management capabilities) would give the platform team a governed production→distribution pipeline.

**What to say in the room (to Kathleen):**
> *"The content governance question I want to explore with you is whether Seismic is serving as our DAM or whether we need a layer upstream of it. If compliance-approved content is still reaching Seismic through manual uploads from individual team members, we have a version control risk and a brand consistency risk that a proper DAM integration would eliminate."*

---

#### Pillar 2: Omnichannel Strategy — Every Channel, Correct Format

Content that exists in only one format is not distributed — it is housed. Asset managers must reach advisors, institutional analysts, wealth clients, and internal teams across different contexts: advisor portals, email campaigns, LinkedIn, webinar follow-ups, client portal downloads, and RFP responses. Each channel requires a different content format.

**The omnichannel content distribution model for Nomura:**

| Content Type | Source Format | Distributed Via | Target Audience |
|---|---|---|---|
| Fund Fact Sheet | PDF (monthly) | Client Portal, Seismic, Email | Advisors, Institutional RFP teams |
| Performance Attribution | PDF + Interactive Report (Vermilion/PowerBI) | Client Portal, RM reporting package | Institutional clients, Wealth clients |
| Investment Commentary | Long-form PDF | Seismic, Email campaign, LinkedIn (excerpt) | Advisors, Wealth clients, Prospects |
| Webinar Recording | Video | Portal, Email follow-up, Seismic library | Advisors, Registered institutional prospects |
| DDQ/RFP Response | Structured document | AI Digital Agent output, direct email | Institutional due diligence teams |
| Regulatory Disclosure | PDF (FINRA-approved) | Portal, Seismic (compliance-tagged) | All distribution channels |
| Product Pitch Deck | PowerPoint / PDF | Seismic (customised per audience tier) | Wholesalers, Advisors |
| Social Media Content | Short-form image, quote card, video clip | LinkedIn, firm social profiles | Advisors, prospects, general market |

**The content atomisation principle:** Effective omnichannel distribution requires **breaking down larger content pieces into smaller, reusable formats**. A 60-minute webinar becomes: a 90-second highlight clip for LinkedIn, a 3-slide summary PDF for Seismic, a quote card for social media, and a full-recording link for the portal. This is not a marketing preference — it is a reach multiplier. Advisors who will not watch 60 minutes will engage with 90 seconds.

**Nomura application:** Seismic's LiveSend and content analytics capability already supports content atomisation tracking — the platform can tell you which content formats drove the most advisor engagement. Connecting this analytics signal to the content production workflow (informing investment writing and design teams which formats perform) creates a feedback loop that compounds distribution effectiveness over time.

---

#### Pillar 3: Targeted Distribution — Data-Driven Audience Segmentation

The era of broadcast content distribution — the same fact sheet to every advisor — is over. Advisors with different book sizes, client profiles, geographic markets, and product specialisations have different content needs. Institutional investors at different stages of the sales cycle (awareness → due diligence → mandate → ongoing) need different content. Delivering the wrong content at the wrong stage damages the relationship.

**The segmentation model for Nomura content distribution:**

| Audience Segment | Distribution Signal | Content Priority |
|---|---|---|
| Advisors — AUM tier 1 ($500M+) | Salesforce account tier, relationship depth | Institutional-grade attribution, CIO letter, strategy deep dives |
| Advisors — AUM tier 2 ($50M–500M) | Salesforce account tier | Fund fact sheets, performance summaries, investment commentary |
| Advisors — Prospect (no current allocation) | Salesforce opportunity stage | Product pitch decks, competitive positioning, webinar invitations |
| Institutional — RFP stage | AI agent DDQ trigger, Salesforce opportunity stage | DDQ/RFP automation, strategy overview, GIPS performance |
| Institutional — Ongoing mandate | Contract confirmed, reporting cadence active | Attribution reports, risk reports, portfolio commentary |
| Wealth clients (direct) | Client portal login profile, holdings | Personalised performance reports, curated commentary |

**Technology enabler:** The Salesforce + Seismic integration is the primary segmentation engine. Seismic's LiveContent personalization capability can populate a pitch deck with advisor-specific data (their clients' asset allocation, relevant strategy performance) using Salesforce account data as the input. This transforms a generic product presentation into a personalised client conversation — without requiring the RM to manually customise it.

---

#### Pillar 4: Efficiency Gains — Automated Content Workflows

The traditional content distribution process in asset management is manual, fragmented, and slow:
1. Portfolio team writes monthly commentary (takes 2–3 days)
2. Marketing formats it (takes 1–2 days)
3. Compliance reviews it (takes 2–5 days)
4. Distribution team publishes to portal and emails advisors (takes 1 day)
5. Total cycle time: 6–11 business days from writing to advisor

Automated content workflows compress this cycle:

**Automated workflow model:**
```
Portfolio Manager → Content CMS (structured template)
        ↓          (pre-approved structure reduces compliance review)
Compliance queue → AI-assisted review flags / auto-clears low-risk content
        ↓          (2 hours for templated content vs. 2 days for free-form)
DAM / Seismic → Auto-publishes to portal + Seismic library
        ↓          (zero manual upload step)
Email campaign platform → Automated send to segmented advisor list
        ↓
Advisor engagement analytics → Back to Seismic recommendation engine
```

**Nomura target cycle time:** 48 hours from portfolio team submission to advisor delivery for standard monthly content.

**AEM Assets reference:** Adobe Experience Manager Assets (AEM Assets) is one model for this automated workflow — it combines DAM capabilities with workflow orchestration, allowing asset creation, compliance review, and multi-channel publishing to occur within a single governed pipeline. The principle applies whether the implementation uses AEM, Aprimo, or a Seismic-native workflow: **eliminate every manual handoff between content creation and content delivery**.

---

#### Pillar 5: Compliance & Security — Governed Distribution at Scale

In a registered investment adviser environment, content distribution is not just a marketing function — it is a regulatory function. FINRA rules govern what investment companies can communicate to retail investors and how. SEC advertising rules (Rule 206(4)-1) govern what can be in performance marketing materials. Every piece of content that leaves the firm must be pre-approved, properly disclosured, and auditable.

**The compliance requirements that DAM must satisfy:**

| Requirement | DAM Capability Required |
|---|---|
| FINRA pre-approval of retail communications | Workflow that routes new content through compliance before publication |
| SEC performance advertising standards | Version control that locks approved performance data; prevents unauthorised edits |
| Content expiry for time-sensitive materials | Automatic unpublishing of expired fact sheets, outdated performance data |
| Audit trail for distributed content | Record of who accessed, modified, or distributed each asset |
| Access entitlement by jurisdiction | Geographic access controls (EU GDPR, UK FCA, US SEC — different materials for different markets) |
| Broker-dealer content supervision | Logging of what content RMs sent to clients via Seismic LiveSend |

**FMG/Vestorly model:** The FMG acquisition of Vestorly (advisor content marketing platform) is instructive — it demonstrates that the financial services market is moving toward **curated, compliance-pre-cleared content libraries** that advisors can draw from without individual compliance review, because the governance is built into the library, not the distribution step. This is the architectural direction Nomura's content platform should move toward: pre-cleared content in Seismic, distributed with confidence, with Salesforce tracking the distribution record.

---

### The Content Distribution Technology Stack — Nomura Model

```
+------------------------------------------------------------------+
|              CONTENT PRODUCTION                                  |
|  Portfolio team → Marketing → Investment Writing → Design        |
+---------------------------+----------------------------------+---+
                            |
           +----------------v----------------+
           |    DAM / Content Hub            |
           |  (Seismic Content Hub or        |
           |   Aprimo/Bynder upstream)        |
           |  • Version control              |
           |  • Compliance workflow          |
           |  • Metadata tagging             |
           |  • Auto-expiry                  |
           +----------------+----------------+
                            |
        +-------------------+--------------------+
        |                   |                    |
+-------v------+   +--------v-------+   +--------v---------+
| SEISMIC      |   | CLIENT PORTAL  |   | EMAIL CAMPAIGNS  |
| Sales Enable.|   | Institutional  |   | Advisor segments |
| Wholesalers  |   | Wealth clients |   | Prospect lists   |
| Advisors     |   | NAV reports    |   | Commentary sends |
+--------------+   +----------------+   +------------------+
        |                   |                    |
        +-------------------+--------------------+
                            |
           +----------------v----------------+
           |  ENGAGEMENT ANALYTICS           |
           |  (Salesforce + Seismic)         |
           |  • What content was opened      |
           |  • Who downloaded what          |
           |  • Which formats drove meetings |
           +----------------+----------------+
                            |
           +----------------v----------------+
           |  CONTENT OPTIMISATION LOOP      |
           |  Inform next content cycle:     |
           |  what to produce, in what       |
           |  format, for which segment      |
           +---------------------------------+
```

---

### Connecting Content Distribution to AUM Outcomes

| Content Distribution Capability | Business Outcome | AUM Impact |
|---|---|---|
| Personalised Seismic pitch decks (Salesforce data) | Advisors present relevant strategies, not generic product | Higher wallet share conversion per RM meeting |
| Automated fact sheet distribution within 48 hrs | Advisors have current data before client meetings | Reduced advisor attrition from content delays |
| Compliance-pre-cleared content library | Zero re-review for standard content; faster to market | Competitive advantage in time-sensitive market moments |
| Content engagement analytics → RM follow-up | Salesforce triggers alert when advisor engages with content | Higher meeting conversion from content-to-conversation |
| Webinar → atomised content (clip, PDF, social) | 5× distribution reach from single content production investment | Broader advisor pipeline engagement |
| Content expiry + version control | No outdated performance data reaches advisors or clients | Elimination of compliance/regulatory risk from stale content |

---

## 20. AI Chatbot + DAM + Salesforce Integration Architecture

> *The convergence of AI-powered conversational interfaces, centralised digital asset management, and CRM creates the "ambient content layer" — where the right asset reaches the right person at the right moment, without search, without context-switching, and without compliance risk. For an asset manager, this is not a technology capability — it is a distribution advantage.*

### Why This Integration Matters at Nomura

Today, an RM or wholesaler at Nomura who needs a specific piece of content for a client interaction takes this workflow:
1. Remember where the content might be
2. Search across Seismic, the intranet, email attachments, or shared drives
3. Check whether the version found is the current, compliance-approved version
4. Download it or forward it
5. Log the activity in Salesforce (usually skipped)

This workflow loses to a well-configured AI chatbot + DAM + Salesforce integration in every dimension: speed, accuracy, compliance assurance, and auditability. The integrated model allows an RM to say, *"Find the latest private credit strategy overview for an institutional prospect in the UK,"* and receive the correct, compliance-approved, jurisdiction-appropriate document in under 10 seconds — without leaving Salesforce, without knowing its location, and with the interaction automatically logged.

---

### System Architecture: Three Integrated Layers

```
+-------------------------------------------------------------+
|  LAYER 1: CONVERSATIONAL AI (User-Facing Interface)         |
|                                                             |
|  Salesforce Agentforce / Einstein AI Chatbot                |
|  ┌────────────────────────────────────────────────┐        |
|  │  Natural Language Query Interface               │        |
|  │  "Find the latest EM debt fund fact sheet"      │        |
|  │  "Show compliance-approved UK pitch deck"        │        |
|  │  "What changed in last month's commentary?"     │        |
|  └───────────────────┬────────────────────────────┘        |
+---------------------│-------------------------------------------+
                       │ API call with parsed intent + metadata
+---------------------v-------------------------------------------+
|  LAYER 2: DIGITAL ASSET MANAGEMENT (Content Repository)     |
|                                                             |
|  Aprimo / Bynder / OpenText OTMM / Acquia (Widen)          |
|  ┌─────────────────────────────────────────────────┐       |
|  │  AI-powered auto-tagging (content type, asset   │       |
|  │  class, audience, jurisdiction, expiry date)    │       |
|  │  Version control + compliance workflow status   │       |
|  │  Usage rights management                        │       |
|  │  Search API → returns asset ID + download URL   │       |
|  └──────────────────────┬──────────────────────────┘       |
+------------------------│---------------------------------------+
                          │ Asset metadata + secure download link
+------------------------v---------------------------------------+
|  LAYER 3: CRM ORCHESTRATION (Salesforce Sales & Service)    |
|                                                             |
|  Salesforce Financial Services Cloud                        |
|  ┌─────────────────────────────────────────────────┐       |
|  │  RM / Wholesaler working context                │       |
|  │  Account record (client, opportunity, stage)   │       |
|  │  Activity log: asset retrieved + sent          │       |
|  │  Email/message send via Salesforce channel      │       |
|  │  Content performance feed back to Seismic       │       |
|  └─────────────────────────────────────────────────┘       |
+-------------------------------------------------------------+
```

---

### The AI Chatbot Component — Agentforce / Einstein at Nomura

Salesforce Agentforce (formerly Einstein Copilot, evolving into the AI Agent layer) is the conversational interface that sits inside Salesforce and acts as the RM's intelligent assistant. At Nomura, a properly configured Agentforce deployment would serve three distinct use cases:

**Use Case 1 — Content Retrieval (DAM Query)**
RM asks: *"I'm preparing for a meeting with a UK pension fund considering our global fixed income strategy. What materials should I bring?"*

Agentforce behavior:
- Parses intent: audience (institutional, UK), strategy (global fixed income), stage (pre-meeting)
- Queries DAM via API: filters for UK-jurisdiction content, global fixed income tag, current version, compliance-approved status
- Returns: strategy overview PDF, last 4 quarters of GIPS performance, CIO investment commentary, and any relevant ESG disclosure required for UK institutional
- Logs retrieval in Salesforce activity timeline against the Opportunity record
- Records which assets were viewed/sent for Seismic analytics feedback

**Use Case 2 — RFP / DDQ Instant Response**
RM asks: *"The prospect just asked about our investment risk management framework in their DDQ portal. What's our approved response?"*

Agentforce behavior:
- Queries the AI Digital Agent knowledge base (Section 6 — existing DDQ automation layer)
- Returns the current, compliance-approved DDQ response chunk for "risk management framework"
- Flags confidence score: if below threshold, routes to subject matter expert for human review before RM uses it
- All responses generated are audit-logged with timestamp, version used, and RM identity (FINRA requirement)

**Use Case 3 — Content Freshness Alert**
Proactive: *"The fund fact sheet for Nomura Global High Yield that you sent to your prospect 14 days ago has been updated. The performance data now reflects February month-end. Would you like to send the updated version?"*

Agentforce behavior:
- DAM integration detects new approved version of previously sent asset
- Pushes notification to RM via Salesforce notification panel
- One-click re-send with updated content via Salesforce email with activity log
- Prospect engagement tracked (open, click, download)

---

### The DAM Component — Platform Evaluation for Nomura

| DAM Platform | Salesforce Integration | Key Capability | Best Fit For |
|---|---|---|---|
| **Aprimo** | Native connector, AI-driven tagging, predictive search | Full content lifecycle management from planning to delivery; robust compliance workflow | Enterprise asset managers with high content volume and complex compliance requirements |
| **Bynder** | Salesforce Marketing Cloud direct integration, CDP connector | Brand experience focus; AI-powered DAM with strong creative team workflow | Wealth / retail distribution with heavy brand and campaign focus |
| **OpenText OTMM** | Salesforce Marketing Cloud connector; browse and use in-platform | Large-scale media management; proven in regulated industries | Institutional / compliance-heavy environments where content governance is primary |
| **Acquia DAM (Widen)** | AI assistant in workflow tool (Acquia Workflow) for conversational content review | Strong metadata management; mid-market pricing | Mid-size asset managers expanding DAM capability from basic shared drives |
| **Pics.io** | Custom tab in Salesforce — browse DAM library without switching apps | Lightweight, rapid deployment | Smaller teams needing quick DAM-Salesforce connection without enterprise integration complexity |
| **Seismic (extended)** | Native within Salesforce FSC; existing Nomura deployment | Sales-layer DAM with personalisation engine; LiveSend tracking | Already deployed — primary distribution surface; extend upstream to cover compliance workflow |

**Nomura recommendation:** The most pragmatic path is to **extend Seismic's content management capabilities upstream** (into the compliance review and version control workflow) rather than deploying a separate enterprise DAM alongside it. Seismic's Content Hub already provides libraries, versioning, and compliance controls for sales-facing content. The gap to close is the **production-to-Seismic workflow** — how content travels from portfolio team through compliance review into Seismic in a governed, automated way. A lightweight workflow tool (Aprimo Workfront-style, or even a configured Salesforce Flow) connecting the compliance review step to Seismic's asset upload API would close this gap without deploying a second enterprise platform.

---

### The Salesforce Component — Financial Services Cloud as Orchestration Layer

Salesforce Financial Services Cloud (FSC) is the integration hub that makes the AI chatbot + DAM model coherent. Without FSC context, the AI chatbot serves generic content. With FSC context, it serves **personalised, situationally relevant content** because it knows:

- Which account the RM is working on right now
- What stage the opportunity is at
- What content the prospect has previously received and whether they engaged with it
- What the advisor's book composition is (relevant strategy alignment)
- What events or meetings are upcoming (pre-meeting content preparation)

**The FSC data model that powers personalisation:**

| Salesforce Object | Data | Content Distribution Use |
|---|---|---|
| Account | Client type (institutional/wealth), AUM tier, jurisdiction | Filters DAM content by audience type and regulatory jurisdiction |
| Opportunity | Stage, strategy of interest, decision timeline | Serves stage-appropriate content (awareness vs. due diligence vs. close) |
| Activity / Task | Past meetings, emails, content sent history | Prevents re-sending content already received; identifies content gaps |
| Campaign Member | Email engagement (opens, clicks, downloads) | Informs Seismic recommendation engine of what content drove engagement |
| Case (Service Cloud) | Client service inquiries, issue history | Allows support agents to pull technical fund documents, NAV reports instantly |
| Contact | Role (CIO, CFO, Compliance, Operations), relationship owner | Personalises content to decision-maker role, not just account |

---

### Key Functional Benefits — Mapped to Nomura Business Outcomes

#### Benefit 1: Reduced Manual Labour → Faster Distribution Cycle

**Current state:** An RM manually searches three systems (Seismic, intranet, email archives), downloads a file, checks it for currency, and attaches it to an email. Average time: 15–25 minutes per content retrieval.

**Integrated AI + DAM + Salesforce state:** RM queries Agentforce in natural language, receives current, jurisdiction-correct, compliance-approved asset in one action. Average time: under 60 seconds. Retrieval is logged automatically.

**Annual time saving at Nomura scale:** If each RM performs 10 content retrievals per week, 50 weeks per year, across a 30-person distribution team: 10 × 50 × 30 × 24 minutes saved = **360,000 minutes = 6,000 hours = 750 working days** recovered annually for client-facing work.

#### Benefit 2: Brand Consistency → Compliance Risk Elimination

When content retrieval bypasses the DAM — which is the current state when RMs search their own email archives — outdated content reaches clients and advisors. An outdated fund fact sheet with last quarter's performance data is not just an embarrassment: in a registered investment adviser context, it may constitute a misleading communication.

**The AI + DAM integration makes the compliant path the path of least resistance.** If the easiest way to get content is through Agentforce, and Agentforce only serves DAM-approved content, outdated content does not reach distribution — not because RMs were disciplined, but because the system architecture made the non-compliant path harder than the compliant one.

#### Benefit 3: Sales Enablement → AUM Conversion

Seismic's own research (2024) indicates that sales teams with effective content enablement close deals 27% faster than those without. In asset management terms, a 27% improvement in mandate conversion speed on a 12-month institutional sales cycle means 3 months of fee billing recovered per closed mandate. On a $500M mandate at 40bps management fee, that is $500,000 in fees per mandate accelerated.

The AI + DAM + Salesforce integration is the technical architecture that makes this conversion acceleration possible at scale — not by changing what RMs do, but by eliminating the friction between knowing what to share and actually sharing the right thing.

#### Benefit 4: Engagement Analytics → Content ROI

The integrated model closes the loop between content distribution and content ROI. Today, Nomura's marketing team produces content without consistent data on which pieces drove advisor conversations, which formats drove meeting requests, or which strategies saw increased allocation following a content campaign.

With DAM + Salesforce + Seismic analytics:
- Every content send is tracked (Seismic LiveSend)
- Every engagement (open, download, click) is logged back to the Salesforce Opportunity or Account
- Patterns emerge: which content types correlate with meeting requests, which strategy commentary pieces preceded allocation decisions
- Content production investment is directed toward what works, not what looks good

---

### Implementation Roadmap — Nomura Phasing

| Phase | Timeline | Scope | Outcome |
|---|---|---|---|
| **Phase 1 — Foundation** | Q1–Q2 | Audit current Seismic content library: identify content with missing metadata, expired materials, missing jurisdiction tags | Clean, governable DAM baseline in Seismic |
| **Phase 2 — Workflow Automation** | Q2–Q3 | Build portfolio team → compliance review → Seismic publish workflow using Salesforce Flow or lightweight workflow tool | 48-hour content delivery cycle for standard content types |
| **Phase 3 — Agentforce Integration** | Q3–Q4 | Deploy Salesforce Agentforce with Seismic/DAM API integration; train on content taxonomy; enable natural language retrieval within Salesforce | RM content retrieval time reduced to under 60 seconds |
| **Phase 4 — Personalisation Engine** | Q4–Q1 next year | Connect Salesforce FSC account/opportunity data to Seismic LiveContent personalisation; enable auto-populated pitch decks | Personalised presentations without manual RM customisation |
| **Phase 5 — Analytics Loop** | Ongoing from Q3 | Connect Seismic LiveSend analytics to Salesforce Campaign and Opportunity objects; build content ROI dashboard in PowerBI | Content investment decisions grounded in conversion data |

---

### What to Say in the Room

**To Kathleen (Client Experience & AI focus):**
> *"The AI agent we're building for DDQ and RFP automation is the same conversational architecture we'd use to give RMs instant access to the right content without leaving Salesforce. The question I want to explore is whether we extend the agent's scope to include content retrieval — so that the same intelligence that drafts a DDQ response can also surface the right fact sheet or strategy overview for a pre-meeting brief. That would give us a single conversational AI layer for both response generation and content access, governed by the same compliance architecture."*

**To Stuart (Client Platforms & Salesforce focus):**
> *"The DAM + Salesforce integration is the missing link between Seismic and the RM workflow. Right now, Seismic is a library that RMs have to remember to visit. With Agentforce surfacing assets inside Salesforce based on what the RM is doing — which account they're on, which stage the opportunity is at — Seismic becomes ambient: the right content appears when it's needed, without any additional RM action. The integration is achievable within the Seismic + Salesforce FSC stack we already have. The first step is getting the Seismic content API scoped and the Agentforce action configured for content queries."*

---

## Document Metadata

| Field | Value |
|---|---|
| **Prepared by** | Calvin Lee |
| **Target audience** | Kathleen Hess McNamara, Stuart Mumley |
| **Role** | ED, Head of Client Technology — Nomura Asset Management International |
| **Purpose** | Business partnership fluency and panel preparation |
| **Technical architecture** | See main [README.md](README.md) |
| **BIAN Framework** | [BIAN_FRAMEWORK_ASSET_MANAGEMENT.md](BIAN_FRAMEWORK_ASSET_MANAGEMENT.md) — service domain architecture and interview vocabulary |
| **NCIE Case Study** | [NOMURA_CLIENT_INSIGHT_ENGINE.md](NOMURA_CLIENT_INSIGHT_ENGINE.md) — AWS AI feedback orchestration platform (Bedrock, Step Functions, Comprehend, OpenSearch) for Kathleen & Stuart panel |
| **Tier 1 Digital Layer** | [DIGITAL_LAYER_SALESFORCE_INTEGRATION.md](DIGITAL_LAYER_SALESFORCE_INTEGRATION.md) — React/Amplify/PowerBI/QuickSight + Salesforce Distribution Intelligence Platform for Shaw, Stuart & Kathleen |
| **AI Chatbot + CRM Cross-Sell** | [AI_CHATBOT_CRM_SALESCLOUD_INTEGRATION.md](AI_CHATBOT_CRM_SALESCLOUD_INTEGRATION.md) — Agentforce + Bedrock + SageMaker JumpStart AI chatbot for RM content retrieval, cross-sell intelligence, and DAM integration (Kathleen, Stuart & Shaw) — [Eval 3 Certified 9.84/10 ✅](evaluation/EVALUATION_AI_CHATBOT_CRM_CYCLE_3_FINAL_CERTIFICATION.md) |
| **NAIM Support Agent Co-Pilot** | [SUPPORT_AGENT_COPILOT_NAIM.md](SUPPORT_AGENT_COPILOT_NAIM.md) — Service Cloud LWC + Bedrock Agents + SageMaker JumpStart + Comprehend PII masking. Internal Support Co-Pilot: Context-Aware Troubleshooting, Historical Ticket Synthesis, Sentiment-Driven Triage (Kathleen, Stuart & Shaw) — [Eval 3 Certified 9.87/10 ✅](evaluation/EVALUATION_NAIM_CYCLE_3_FINAL_CERTIFICATION.md) |
| **NAPCE Agentic Workflow Automation** | [AGENTIC_WORKFLOW_AUTOMATION_NAPCE.md](AGENTIC_WORKFLOW_AUTOMATION_NAPCE.md) — Amazon MWAA + Saga Pattern + Bedrock Agents (DDQ/RFP/Compliance) + SageMaker GIPS Engine + Textract + Amazon Batch: DDQ automation (60–80% faster), continuous GIPS compliance monitoring, institutional mandate pipeline acceleration. BIAN domains: Sales & Distribution, Customer & Party Agreement, Investment Portfolio Management, Sales Enablement (Kathleen, Stuart & Shaw) — [Eval 3 Certified 9.90/10 ✅](evaluation/EVALUATION_NAPCE_CYCLE_3_FINAL_CERTIFICATION.md) |
| **Asset Management Reports & Use Cases** | [ASSET_MANAGEMENT_REPORTS.md](ASSET_MANAGEMENT_REPORTS.md) — Data-as-a-Product: 10 business use cases (Institutional Due Diligence, CRM Distribution Enablement, Onboarding Automation, Reporting Modernization, Portfolio Transparency, Wealth Self-Service, Advisor Engagement, Regulatory Reporting, Sales Analytics, Data Centralization), 10 core reports & dashboards, 4 strategic domains (Revenue Engine, Experience Engine, Trust Engine, Alpha Engine), Lemons Table (6 proactive mitigations). KPIs: 30% churn reduction, 90% reporting automation, 50% NPS/CES improvement. Snowflake CQRS Gold + Vermilion/Power BI + Salesforce FSC/Agentforce/Seismic + NAPCE GIPS Engine + Bedrock + Cognito RLS (Kathleen & Stuart) — [Eval 3 Certified 9.84/10 ✅](evaluation/EVALUATION_ASSET_MGMT_REPORTS_CYCLE_3_FINAL_CERTIFICATION.md) |
| **Version** | 3.0 — Enhanced with Content Distribution, DAM, AI+DAM+Salesforce Integration |
| **Last updated** | February 2026 |

---

*This document is the business partnership companion to the technical architecture README. It is intended to demonstrate that the Head of Client Technology understands the business outcomes the platform must deliver — not just the technology that delivers them.*
