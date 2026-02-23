# Power BI Omnichannel Wealth Dashboards - Evaluation Cycle 1
## Cross-Functional Product & Analytics Review

**Evaluator Panel**: Principal Data Analyst + Product Manager + Sales Lead + Asset Management Client Stakeholder  
**Date**: February 19, 2026  
**Review Target**: POWERBI_WEALTH_OMNICHANNEL_GUIDE.md  
**Session Duration**: 90 minutes

---

## EVALUATION SUMMARY

| Criteria | Score | Status |
|----------|-------|--------|
| **Concept Clarity** | 9.1/10 | ✅ Excellent |
| **8th-Grade Explanation Quality** | 9.3/10 | ✅ Excellent |
| **Technical Accuracy (Consumer Wealth)** | 8.8/10 | ✅ Very Good |
| **Technical Accuracy (Hedge Fund)** | 8.5/10 | ✅ Very Good |
| **Persona Differentiation** | 9.4/10 | ✅ Excellent |
| **Use Case Relevance** | 9.2/10 | ✅ Excellent |
| **Data Governance** | 8.3/10 | ⚠️ Good (gaps identified) |
| **Security & Compliance** | 8.1/10 | ⚠️ Good (needs detail) |
| **Actionability** | 9.0/10 | ✅ Very Good |
| **Business Impact Articulation** | 9.1/10 | ✅ Excellent |
| **OVERALL SCORE** | **8.78/10** | **STRONG BASELINE** |

---

## DETAILED EVALUATION

### Strengths (9.0+/10)

#### 1. Persona Differentiation (9.4/10)
**What's Excellent**:
- The guide clearly captures two radically different user needs:
  - Retail: "I want to play with my retirement, find hidden tax savings, and execute immediately" (emotional, interactive, mobile-first)
  - Hedge Fund: "I need millisecond-level data consistency between my PM's screen and my algorithm" (technical, API-driven, sync-guaranteed)
- The "spoon vs. sword" metaphor perfectly encapsulates why PDF reports fail for both personas

**Quote from Principal Data Analyst**:
> "This is the first time I've seen a single Power BI guide that truly separates consumer and institutional. Most teams try to build one dashboard for everyone and end up pleasing no one. This nails the segmentation."

---

#### 2. 8th-Grade Explanation Quality (9.3/10)
**What's Excellent**:
- Video game analogy is perfect: "Game engine = Snowflake, Player screen = Power BI, Controller = DAX"
- DAX formula explanations use plain English after the code: "Translation: If the stock price today is less than what they paid for it, mark it TRUE"
- "Fat Finger" challenge naming is memorable and immediately understood
- Color-coded threat matrix for retirement age zones (green/yellow/red) is intuitive

**Quote from Sales Lead**:
> "I could literally hand this to our wealth advisors with zero technical background, and they'd understand 'DirectQuery' vs. 'Import' thanks to the 'copying a picture' vs. 'live video feed' analogy."

---

#### 3. Omnichannel Strategy (9.2/10)
**What's Excellent**:
- Mobile layout explicitly prioritizes: big balance number, single slider, red alert, one execution button
- Hedge fund layout explicitly packs: matrix, scatter plot, outlier table, dense data
- Power Apps bridge: Moving from passive alerting ("here's your tax loss") to active execution ("execute this trade") is a game-changer
- GraphQL API design allows quants to query without Power BI layer

---

### Gaps Identified (Cycle 2 Focus)

#### GAP 1: DAX Formula Validation & Testing (Score: 7.5/10)

**Issue Identified**:
The guide shows DAX formulas for Monte-Carlo projection:
```dax
Projected_Balance_At_Retirement = 
    CurrentBalance * POWER(1 + AnnualReturn, YearsUntilRetirement)
```

This is a **compound interest calculation**, but NOT a true Monte-Carlo simulation (which should run 10K+ iterations with random draws). The formula is called "Monte-Carlo" but behaves like a deterministic projection.

**Principal Data Analyst's Concern**:
> "A retail client seeing $2.3M projection feels confident. But if that's a point estimate (not a confidence interval), and market returns are actually -20% next year, the client loses trust. We need to show 'likely range' not just 'most likely outcome'."

**Recommendation for Cycle 2**:
Provide actual Monte-Carlo implementation using Power BI's Python integration:
```python
# Inside Power BI Python script
import numpy as np

def monte_carlo_retirement(current_balance, years, num_simulations=10000):
    results = []
    for _ in range(num_simulations):
        annual_return = np.random.normal(0.07, 0.15)  # 7% mean, 15% std dev
        final_balance = current_balance * (1 + annual_return) ** years
        results.append(final_balance)
    
    return {
        'p10': np.percentile(results, 10),
        'p50': np.percentile(results, 50),
        'p90': np.percentile(results, 90)
    }
```

Return a **confidence interval** ("You'll likely have between $1.8M and $3.2M") rather than a point estimate.

---

#### GAP 2: Tab-Loss Harvesting Wash-Sale Logic (Score: 7.8/10)

**Issue Identified**:
The guide flags stocks with losses and suggests immediate harvesting:
```dax
IsTaxLossOpportunity = IF(CurrentPrice < PurchasePrice, TRUE, FALSE)
```

But **IRS Wash-Sale Rule**: You can't harvest a loss, then buy the same stock back within 30 days. If you violate this, the loss is disallowed.

**Asset Management Client Comment**:
> "If we tell a client to harvest a loss on Microsoft at 10am, and they accidentally buy Microsoft again at 2pm, we've destroyed the tax benefit and potentially created a compliance violation. We need guardrails."

**Recommendation for Cycle 2**:
Add wash-sale detection logic:
```dax
IsSafeToHarvest = 
    IF(
        [IsTaxLossOpportunity] = TRUE AND
        NOT([WasSoldIn_Last30Days]) AND
        NOT([PlannedBuyWithin_Next30Days]),
        TRUE,
        FALSE
    )
```

Plus: Show advisor a warning banner when matched pairs are detected.

---

#### GAP 3: Hedge Fund DirectQuery Performance (Score: 8.0/10)

**Issue Identified**:
The guide recommends DirectQuery for hedge fund data ("live window into Snowflake"). But DirectQuery has latency tradeoffs:
- First query: 2-3 seconds (Snowflake cold start)
- Subsequent queries: 0.5-1 second (warm cache)
- If PM clicks 5 different filters in succession: 5 seconds wait time

For a hedge fund that's used to sub-100ms latency, this feels slow.

**Principal Data Analyst's Concern**:
> "DirectQuery is technically correct but operationally painful. By the time the scatter plot renders, market conditions have changed. Hedge fund PMs will switch back to stale static reports because they're faster."

**Recommendation for Cycle 2**:
Hybrid approach:
1. **Cached aggregates** (Import mode): Daily OHLC, factor exposures → instant visuals
2. **DirectQuery drill-down**: When PM clicks "Show me the 50 biggest trades," DirectQuery pulls details in real-time
3. **Result**: Dashboard loads in <200ms, drill-downs stay <500ms

---

#### GAP 4: API Consistency Guarantee (Score: 7.9/10)

**Issue Identified**:
The guide mentions "Snowflake Time Travel" to keep PM's Power BI and robot's API in sync:
```
Both query AT (TIMESTAMP => '2026-02-19 14:30:00'::TIMESTAMP_NTZ)
```

But **who chooses the timestamp?** If Power BI uses 14:30:00 and the algorithm uses 14:30:01, they're looking at different data (trades executed in that 1 second).

**Principal Data Analyst's Concern**:
> "We need a shared 'consensus timestamp' that both PM and robot lock into. Otherwise, we're lying about consistency."

**Recommendation for Cycle 2**:
Implement a **Data Snapshot Service**:
```
1. Every 100ms, Snowflake publishes a new snapshot version
2. Version = (timestamp, checksum)
3. Power BI query uses version N
4. Robot's algorithm also uses version N
5. Verified: Both see identical data
```

Document this as a hard SLA requirement for hedge fund onboarding.

---

### Evaluator Panel Discussion (Round 1)

---

**Product Manager**: 
> "I love the 'spoon vs. sword' opening. It immediately sells the problem. But I'm worried about scope creep. We're building TWO completely different products (mobile app + desktop Power BI + API). That's not one project, that's three. Should we phase this? Start with mobile?"

**Sales Lead**: 
> "I agree on phasing. Retail wealth clients are easier to land (50-100 client pilot in Month 1). Hedge funds need 6 months of relationship-building just to get IT security approval. Timeline should reflect that."

**Principal Data Analyst**: 
> "Strong technical foundation, but we need to nail down the DAX formulas before any code review. The Monte-Carlo gap is fixable, but it needs to be in Cycle 2. Similarly, the DirectQuery latency trade-off and the Snowflake Time Travel timestamp collision—these need architectural hardening."

**Asset Management Client Stakeholder**: 
> "From the business side, this is gold. Goal-based retirement planning + tax-loss execution? That's a $500M AUM growth story right there. The hedge fund use case is equally strong—synced PM + robot = zero execution delays = millions in saved slippage. Let's move forward, but tighten the technical details."

---

## KEY FINDINGS & RECOMMENDATIONS

### Summary of Cycle 1

| Finding | Severity | Cycle 2 Action |
|---------|----------|---|
| DAX formulas use point estimates, not confidence intervals | Medium | Replace with true Monte-Carlo (Python/NumPy) |
| Wash-sale rule not enforced in tax-loss logic | High | Add 30-day window detection + advisor warning |
| DirectQuery latency may be operationally unacceptable | Medium | Implement hybrid (cached aggregates + DirectQuery drill-down) |
| Snowflake Time Travel timestamp collision not resolved | High | Build Data Snapshot Service with version locking |
| No mention of Power BI row-level security (RLS) for data isolation | Medium | Document RLS policy per client fund + retail cohort |
| No data refresh SLAs documented | Medium | Define refresh cadence (mobile: 1hr, hedge fund: 100ms) |

---

## CYCLE 1 RECOMMENDATIONS

### GO / NO-GO Decision

**DECISION: 🟢 GO (Conditional)**

The document provides an excellent strategic and conceptual foundation. The persona differentiation is outstanding, the 8th-grade explanations are accessible, and the omnichannel strategy is sound.

**However**, proceed to Cycle 2 with these mandatory fixes:

1. ✋ **HOLD** on committing DAX formulas until they're validated with actual financial data
2. ✋ **HOLD** on DirectQuery recommendation until latency testing is complete
3. ✋ **HOLD** on Snowflake Time Travel guarantee until versioning service is designed
4. ✅ **GO** on mobile-first design direction (thumb-friendly layout is validated)
5. ✅ **GO** on power Apps integration for tax-loss execution (powerful and implementable)
6. ✅ **GO** on GraphQL API spec for hedge fund quants

---

## PHASE RECOMMENDATIONS FOR PHASED GO-LIVE

### Phase 1 (Month 1): Retail Mobile Pilot
- 50 HNW clients
- Simple retirement age slider (deterministic projection, not Monte-Carlo yet)
- Tax-loss harvesting flagged, but advisor must approve before execution
- Power Apps integration: Advisor sees client request, can execute within 2 hours
- **Success metric**: 40%+ engagement rate, 0 wash-sale violations

### Phase 2 (Month 2-3): Hedge Fund Pilot
- 3 client funds
- Power BI desktop dashboards (hybrid cached + DirectQuery)
- GraphQL API for trusted quant partners
- Snowflake Time Travel versioning service operational
- **Success metric**: PM + robot synchronized to <1ms clock skew

### Phase 3 (Month 4+): Production Scale
- Roll out to full retail base, full hedge fund base
- Advanced features: True Monte-Carlo, ML-powered factor models

---

**Cycle 1 Final Score: 8.78/10 ✅ STRONG BASELINE**

**Recommendation: Proceed to Cycle 2 (Architectural Hardening & Implementation Details)**

