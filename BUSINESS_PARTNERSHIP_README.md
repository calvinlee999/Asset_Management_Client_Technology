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

### What to Say in the Room

> *"At FIS I led AI-enabled document processing for capital markets workflows. The lesson I took from it is that the model is rarely the bottleneck — content governance is. The questions I'd want to answer before scaling the agent to new use cases are: Do we have clear ownership of every answer in the knowledge base? Do we have a process for updates when strategy or policy changes? And do we have accuracy metrics that give the business confidence in what the agent produces? Once those are solid, the expansion roadmap becomes much more straightforward."*

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

## Document Metadata

| Field | Value |
|---|---|
| **Prepared by** | Calvin Lee |
| **Target audience** | Kathleen Hess McNamara, Stuart Mumley |
| **Role** | ED, Head of Client Technology — Nomura Asset Management International |
| **Purpose** | Business partnership fluency and panel preparation |
| **Technical architecture** | See main [README.md](README.md) |
| **Last updated** | February 2026 |

---

*This document is the business partnership companion to the technical architecture README. It is intended to demonstrate that the Head of Client Technology understands the business outcomes the platform must deliver — not just the technology that delivers them.*
