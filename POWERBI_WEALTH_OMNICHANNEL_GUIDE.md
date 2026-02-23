# Power BI Omnichannel Wealth Dashboards Guide
## 8th-Grade Step-by-Step Instructions for Consumer Wealth & Hedge Fund Personas

**Document Version**: 1.0  
**Date**: February 19, 2026  
**Audience**: Product managers, data analysts, UX designers, wealth advisors, quant teams  
**Reading Time**: 25 minutes (Part 1: Concepts) + 45 minutes (Part 2: Technical Builds)

---

## EXECUTIVE SUMMARY: CONCLUSION FIRST

### The Core Insight: Don't Hand Out Spoons When You Need Swords 🗡️

We are stopping the practice of handing static PDFs to our wealthiest clients and most sophisticated hedge funds. **Static is dead.**

Instead, we are building **two radically different Power BI experiences**:

1. **Consumer Wealth (Retail & HNW)**: A mobile-first, interactive "Time Machine" that lets clients play with retirement sliders, instantly see their financial future, and one-click execute tax-loss harvesting trades.

2. **Hedge Funds (Institutional & Quants)**: A high-density, real-time desktop dashboard paired with raw API access, so their algorithmic trading engines see exactly what the PM sees—synchronized to the microsecond.

### Strategic KPIs

| KPI | Target | Impact |
|-----|--------|--------|
| **Mobile Wealth Client Adoption** | 100% of HNW segment | +$500M AUM growth |
| **Hedge Fund Dashboard Latency** | <200ms refresh | Zero execution delays |
| **Omnichannel Data Sync** | Zero clock-skew | PM + Robot in perfect lock-step |
| **Tax-Loss Execution Rate** | <5 min from alert to trade | +$50M tax benefit annually |
| **API Integration** | 100% hedge fund quants connected | 10x faster alpha discovery |

### Customer Value Promise

❌ **Old Way**: "Here's a PDF. Good luck analyzing it."  
✅ **New Way**: "Here's your personalized wealth engine. Play with it. Let it suggest trades. Execute immediately."

**Financial Impact**: From passive reporting → active wealth management (+$550M AUM, +$50M tax savings)

---

## PART 1: THE 8TH-GRADE CONCEPT GUIDE

### Analogy: Building Custom Video Game Interfaces

Imagine you're building a video game:
- **The Game Engine** = Our Snowflake database (all the raw data)
- **The Player's Screen** = Power BI (what the player sees and controls)
- **The Controller** = DAX formulas (the math that makes the game respond to player input)

**Key Insight**: Different players need different screens.

- A **mobile gamer** wants a simple, thumb-friendly screen with big buttons.
- A **competitive esports player** wants a dense, information-packed screen with a hundred stat readouts.

Just like a video game, our Power BI dashboards must:
1. Show exactly the right data for that user type
2. Respond instantly when the user interacts
3. Never overwhelm the user with complexity they don't need

---

### Concept 1: What is Power BI?

**Power BI is a window into your data.**

It connects to a database (like Snowflake), pulls in the numbers you need, and displays them on a screen. But unlike a static report (PDF), Power BI is **interactive**: when you click a button or move a slider, the numbers update instantly.

**Three Ways Power BI Can Connect**:

1. **Import Mode** (Like copying a picture)
   - We copy the data INTO Power BI's memory
   - Fast, but only shows yesterday's data
   - Good for: Small datasets, retail clients

2. **DirectQuery Mode** (Like a live video feed)
   - Power BI stays connected to Snowflake and asks for data in real-time
   - Always shows today's data, no delays
   - Good for: Huge datasets, hedge funds with billions of rows

3. **Live Connection (Like watching live TV)**
   - Power BI directly mirrors a live data feed
   - Zero latency, perfect for microsecond-sensitive hedge fund trades
   - Good for: The most demanding institutional clients

---

### Concept 2: DAX = The Video Game Controller

**DAX** stands for "Data Analysis Expressions." It's the language you use to write math formulas in Power BI.

**Example (Retail Wealth Client)**:
```
Investor writes: "I want to retire at age 65 and have $2M."

DAX Formula: 
If RetirementAge = 65, 
then calculate how much money they'll have NOW 
using a Monte-Carlo simulation 
(rolling virtual dice 10,000 times to predict the future)
```

**Example (Hedge Fund)**:
```
Portfolio Manager wants to see: 
"Show me all trades where execution cost > $500K"

DAX Formula:
If TradeCost > 500000, 
then highlight RED and show the time it happened down to the microsecond
```

**Key**: DAX formulas are **live**. When the user moves a slider, DAX recalculates instantly.

---

### Concept 3: Mobile vs. Desktop = Different Screens, Same Engine

**Mobile Layout (Consumer Wealth)**:
```
Think: iPhone screen, thumb-sized buttons
┌──────────────────┐
│  YOUR BALANCE    │
│  $1,234,567      │ ← Big, clear number
│                  │
│ [Retire at: 65]  │ ← Single interactive slider
│                  │
│  TAX SAVINGS:    │
│  🚨 $45,000      │ ← Red alert, ONE button
│  [Execute Trade] │
└──────────────────┘
```

**Desktop Layout (Hedge Fund)**:
```
Think: 27-inch monitor, packed with data
┌──────────────────────────────────────────────────────────┐
│ Portfolio Risk | Factor Exposure | Trade Cost Analysis   │
├──────────────────────────────────────────────────────────┤
│ [Scatter Plot]    │ [Heat Map]        │ [Dense Table]    │
│ 10,000 data       │ 500+ parameters   │ 50 columns       │
│ points visible    │ color-coded       │ sortable/filter  │
│                   │                   │ able             │
└──────────────────────────────────────────────────────────┘
```

**Same underlying data, completely different UX.**

---

## PART 2: STEP-BY-STEP BUILD INSTRUCTIONS

### USE CASE 1: CONSUMER WEALTH (The "Time Machine" Mobile App)

#### Goal:
Let wealth clients (retail investors and high-net-worth individuals) play with their retirement age, instantly see their financial future, and one-click execute tax-loss harvesting trades.

---

#### STEP 1: Get the Ingredients (Connect to Snowflake)

**What You're Doing**: Telling Power BI where to find the data.

**Instructions**:

1. Open **Power BI Desktop**
2. Click **"Get Data"** (top-left)
3. Search for and select **"Snowflake"**
4. In the dialog box, enter:
   - **Server**: `nomura-analytics.snowflakecomputing.com`
   - **Database**: `WEALTH_MANAGEMENT`
   - **Warehouse**: `MOBILE_CLIENT_WH` (small, fast warehouse for mobile user)
5. Click **"Connect"**
6. When prompted, select **"Import"** (not DirectQuery—we want fast, cached data for mobile)
7. A list of tables appears. Select these three:
   - `CLIENT_ACCOUNTS` (account balance, name, age)
   - `STOCK_HOLDINGS` (what stocks the client owns, purchase price)
   - `TAX_SUMMARY` (annual tax info, capital gains/losses)
8. Click **"Load"**

**What Just Happened**:
Power BI downloaded the data from Snowflake into its local memory. The data is now ready to build on.

---

#### STEP 2: Build the "Time Machine" (What-If Parameters & Monte-Carlo Slider)

**What You're Doing**: Creating an interactive slider that lets clients change their retirement age and see the future.

**Instructions**:

1. Click the **Modeling** tab (top ribbon)
2. Click **"New Parameter"** (right side)
3. A dialog box opens:
   - **Name**: `RetirementAge`
   - **Data Type**: `Whole Number`
   - **Minimum**: `50`
   - **Maximum**: `80`
   - **Default**: `67`
   - **Step**: `1`
4. Click **"OK"**

**What Just Happened**:
Power BI automatically created a slider that goes from 50 to 80. When you move it, the value changes.

**Now, Create the Magic Formula (DAX)**:

1. Click **"New Measure"** (Modeling tab)
2. Name it: `Projected_Balance_At_Retirement`
3. Paste this formula:

```dax
Projected_Balance_At_Retirement = 
VAR CurrentBalance = SUM(CLIENT_ACCOUNTS[CurrentBalance])
VAR YearsUntilRetirement = [RetirementAge] - TODAY()/365.25
VAR AnnualReturn = 0.07  // 7% average annual return
RETURN
    CurrentBalance * POWER(1 + AnnualReturn, YearsUntilRetirement)
```

**Translation in English**:
"Take the client's current balance. Multiply by 7% interest compound interest for how many years until they hit their chosen retirement age. Show the result."

4. Click **"OK"**

**Test It**:

1. Click **"Page 1"** to go back to the main canvas
2. Drag the slider you created earlier. Notice: As you move the slider, the number updates.

---

#### STEP 3: The Advisor's Secret Weapon (Tax-Loss Harvesting Alert)

**What You're Doing**: Creating a red alert that flags stock positions where the client can save taxes.

**Instructions**:

1. In the **Data** pane, locate the `STOCK_HOLDINGS` table
2. Create a new column:
   - Click **"New Column"** (Modeling tab)
   - Name it: `IsTaxLossOpportunity`
   - Formula:
   ```dax
   IsTaxLossOpportunity = 
   IF(STOCK_HOLDINGS[CurrentPrice] < STOCK_HOLDINGS[PurchasePrice], 
      TRUE, 
      FALSE)
   ```
   - Translation: "If the stock price today is less than what they paid for it, mark it TRUE."

3. Create another column for savings amount:
   - Name it: `TaxSavingsIfHarvested`
   - Formula:
   ```dax
   TaxSavingsIfHarvested = 
   IF([IsTaxLossOpportunity] = TRUE,
      (STOCK_HOLDINGS[PurchasePrice] - STOCK_HOLDINGS[CurrentPrice]) * 0.37,  // 37% tax rate
      0
   )
   ```
   - Translation: "If it's a tax loss opportunity, calculate how much tax they'll save (losses × 37% tax bracket)."

4. Create a measure for total savings:
   - Name it: `Total_Tax_Loss_Savings`
   - Formula:
   ```dax
   Total_Tax_Loss_Savings = 
   SUMIF(STOCK_HOLDINGS, STOCK_HOLDINGS[IsTaxLossOpportunity] = TRUE, [TaxSavingsIfHarvested])
   ```

5. Add a visual to the canvas:
   - Click **"KPI"** visual (visualizations pane, bottom-left)
   - Drag `Total_Tax_Loss_Savings` to the Value field
   - Drag `StockSymbol` to the Details field (so they know which stocks qualify)
   - Format the card to be **RED** (draw attention to money being left on the table)

---

#### STEP 4: The One-Click Trade Execution (Power Apps Integration)

**What You're Doing**: When a client clicks "Execute Tax-Loss Harvest," it doesn't just send an email—it submits a real trade request.

**Instructions**:

1. Click **Insert** tab (top ribbon)
2. Click **"Power Apps"**
3. A Power Apps designer opens. Create a simple form:
   - Add a **Dropdown** field: "Select Stock to Harvest"
   - Add a **Button**: "Confirm Tax-Loss Harvest"
   - Add a **Text Label**: "Your advisor will review and execute within 2 hours"

4. On the Button's click action, write this formula:

```powerapps
Patch(
    'TradingRequests',
    Defaults('TradingRequests'),
    {
        ClientID: User().Email,
        StockSymbol: Dropdown_Stocks.Value,
        OrderType: "TAX_LOSS_HARVEST",
        OrderStatus: "PENDING_ADVISOR_REVIEW",
        CreatedTimestamp: Now()
    }
);
ShowNotification("Trade request submitted! Your advisor will execute within 2 hours.", NotificationType.Information)
```

**Translation**:
"When the client clicks the button, submit the trade request to our database, then show a confirmation message."

5. Click **"Save"**

---

#### STEP 5: Make it Thumb-Friendly (Mobile Layout)

**What You're Doing**: Creating a phone-sized version of the dashboard that hides complexity.

**Instructions**:

1. Click **View** tab (top ribbon)
2. Click **"Mobile Layout"**
3. A phone-shaped canvas appears on the right
4. **Drag** only these elements onto the phone canvas:
   - The "Your Total Balance" KPI (biggest number, at the top)
   - The "RetirementAge" slider
   - The "Total Tax Loss Savings" alert (red, prominent)
   - The Power Apps button for trade execution
5. **Do NOT drag**: Complex charts, data tables, or detailed breakdowns
6. Arrange elements vertically (top to bottom) so thumbs can reach everything
7. Click **"Phone View"** to test on a phone-sized screen

**What Just Happened**:
The mobile dashboard now only shows the 4 most important things. Everything else is hidden. When a client opens the app on their iPhone, they see only what matters.

---

### USE CASE 2: HEDGE FUND (The "Race Car" Quant Sandbox)

#### Goal:
Build a high-speed, real-time dashboard for Portfolio Managers while simultaneously giving their algorithmic trading engines access to the exact same data via API.

---

#### STEP 1: The DirectQuery Connection (Live Window into Snowflake)

**What You're Doing**: Instead of copying data into Power BI, we're keeping Power BI connected live to Snowflake.

**Instructions**:

1. Open **Power BI Desktop**
2. Click **"Get Data"**
3. Select **"Snowflake"**
4. Enter:
   - **Server**: `nomura-analytics.snowflakecomputing.com`
   - **Database**: `HEDGE_FUND_ANALYTICS`
   - **Warehouse**: `HEDGE_FUND_HI_CONCURRENCY_WH` (big, powerful warehouse)
5. When prompted, select **DirectQuery** (NOT Import)
6. Select these tables:
   - `TRADES` (every trade executed, with timestamps down to microseconds)
   - `PORTFOLIO_POSITIONS` (current portfolio composition, weighted by market value)
   - `FACTOR_EXPOSURE` (proprietary risk factors: momentum, value, volatility, etc.)
   - `EXECUTION_COSTS` (slippage, commissions, market impact for each trade)
7. Click **Load**

**What Just Happened**:
Power BI is now looking directly into Snowflake's live data. Every time a PM clicks a button or refreshes, Power BI asks Snowflake "What's the latest?" and gets the newest number.

---

#### STEP 2: Build the High-Density Dashboard (Factor Exposure Matrix)

**What You're Doing**: Creating a packed, information-dense visual that shows every risk factor at a glance.

**Instructions**:

1. Insert a **Matrix** visual:
   - Click the Matrix icon (Visualizations pane)
2. Drag these fields to the matrix:
   - **Rows**: `FactorName` (momentum, value, low-volatility, etc.)
   - **Columns**: `AssetClass` (equities, fixed income, derivatives)
   - **Values**: `FactorExposure_Percentage`
3. Format for density:
   - Right-click the matrix
   - Click **"Format"**
   - Set font size to **8 pt** (small, packed)
   - Enable **"Conditional Formatting"** so values are color-coded:
     - Green = positive factor exposure (good)
     - Red = negative (risky)
     - Yellow = neutral

**What This Shows**:
A hedge fund PM can see their entire factor exposure at a glance. Example:

```
Factor            │ Equities │ Fixed Inc │ Derivatives
────────────────────────────────────────────────────────
Momentum          │   +8.2%  │   +2.1%   │   +15.3%
Value             │  +12.4%  │   +3.5%   │    -1.2%
Low Volatility    │   -4.1%  │   +6.7%   │    +0.3%
Quality           │   +5.6%  │   +9.2%   │    +2.8%
```

The PM instantly knows: "We're overweight momentum, underweight quality. Time to rebalance."

---

#### STEP 3: Build Trade Cost Analysis (TCA) - The Execution Scoreboard

**What You're Doing**: Showing exactly how much money the fund lost (or gained) due to trading costs.

**Instructions**:

1. Insert a **Scatter Chart**:
   - X-axis: `TradeExecutionTime_Microseconds` (when the trade happened)
   - Y-axis: `ExecutionCost_Dollars` (how much it cost)
   - Size: `TradeNotionalValue` (bigger bubbles = bigger trades)
   - Color: `CostCategory` (commission=blue, slippage=red, market impact=orange)

2. Create a DAX measure for outlier detection:

```dax
TCA_OutlierFlag = 
VAR AvgCost = AVERAGE(EXECUTION_COSTS[ExecutionCost_Dollars])
VAR StdDev = STDEV(EXECUTION_COSTS[ExecutionCost_Dollars])
RETURN
    IF(EXECUTION_COSTS[ExecutionCost_Dollars] > AvgCost + (3 * StdDev),
       "OUTLIER_INVESTIGATE",
       "NORMAL"
    )
```

**Translation**:
"If a trade's cost is more than 3 standard deviations above average, flag it as an outlier."

3. Add a table showing top 10 outliers:
   - Rows: `TradeID`, `Symbol`, `ExecutionCost`
   - Filter: `TCA_OutlierFlag = "OUTLIER_INVESTIGATE"`
   - Sort by: ExecutionCost descending

**What This Shows**:
PMs can instantly spot: "Wait, why did we lose $2.3M on that 100K share block? Call execution, investigate now."

---

#### STEP 4: The API Plumbing (Zero-Copy Data Access)

**What You're Doing**: Creating a GraphQL API endpoint so the hedge fund's algorithmic trading engines can query the same Snowflake data without going through Power BI.

**Instructions** (This is for your data engineering team to execute):

1. Create a GraphQL schema that mirrors the Power BI dashboard structure:

```graphql
type Query {
  portfolio(fundID: String!): Portfolio!
  trades(fundID: String, timeRange: TimeRangeInput): [Trade!]!
  factorExposure(fundID: String!): [FactorExposure!]!
  executionCosts(tradeID: String!): ExecutionCost!
}

type Portfolio {
  positions: [Position!]!
  totalValue: Float!
  exposures: [FactorExposure!]!
}

type Trade {
  tradeID: String!
  symbol: String!
  quantity: Int!
  executionPrice: Float!
  executionTime: DateTime!
  slippage: Float!
  commission: Float!
}
```

2. Deploy this GraphQL API on AWS API Gateway
3. Point it at the same Snowflake warehouse that Power BI uses
4. Give the hedge fund API credentials (API key + OAuth)

**Result**: The hedge fund's trading bot does this:

```python
# Hedge fund's algorithmic trading engine
query = """
{
  portfolio(fundID: "HF_NOMURA_001") {
    positions {
      symbol
      quantity
      value
    }
    exposures {
      factorName
      percentageExposure
    }
  }
}
"""

response = graphql_client.query(query)
current_portfolio = response['portfolio']

# Quants use this data to optimize next trades
recommended_trades = optimize_portfolio(current_portfolio)
execute_trades(recommended_trades)
```

**Key Insight**: The PM sees the same data on their Power BI screen that the robot sees in code. They're perfectly synchronized.

---

#### STEP 5: Snowflake Time Travel (Guarantee Data Consistency)

**What You're Doing**: Ensuring the PM's Power BI dashboard and the robot's API query see the exact same data point-in-time, even if Snowflake data changes between requests.

**Instructions** (Data Architecture Team):

1. In Snowflake, add a timestamp column to key tables:

```sql
ALTER TABLE TRADES ADD COLUMN DATA_SNAPSHOT_TIMESTAMP TIMESTAMP = CURRENT_TIMESTAMP();
```

2. When Power BI or the API queries, specify the exact timestamp:

```sql
SELECT * FROM TRADES 
AT (TIMESTAMP => '2026-02-19 14:30:00'::TIMESTAMP_NTZ)
WHERE TRADE_DATE = CURRENT_DATE()
```

3. Both Power BI and the GraphQL API use the **same timestamp** for every query.

**Result**: 
- PM on Power BI: "My portfolio is 45.3% momentum-exposed at 14:30:00"
- Robot's algorithm: "Quoting the exact same snapshot: 45.3% momentum. OK, executing hedge trades now."
- They're locked in perfect sync ✅

---

## PART 3: PROACTIVE MITIGATIONS (The "Lemon Squeezer")

### Challenge 1: Mobile UX - "Fat Finger" Slider Errors

**Problem**: 
On a 5-inch iPhone screen, the retirement age slider is tiny. Client meant to set age 65 but accidentally touched 75. Now they're looking at a retirement plan that's 10 years wrong.

**Mitigation**:

1. Add a **text input box** next to the slider:
   - If client touches the slider, it updates the box
   - If client types directly, it updates the slider
   - This gives them two ways to input, reducing mistakes

2. Add **confirmation dialog**:
   ```
   "You've set Retirement Age to 75. 
    That means you'll work for 42 more years.
    Is this correct? [Confirm] [Cancel]"
   ```

3. Add **visual safety guardrails**:
   - Green zone (50-70): "Typical retirement age"
   - Yellow zone (71-75): "Extended working years"
   - Red zone (76+): "Are you planning to work until 100?"

---

### Challenge 2: Dashboard Freezes - Hedge Fund Data Overload

**Problem**: 
A hedge fund wants to see 10 years of trade history (100M+ rows). Dragging that into Power BI's memory will crash the dashboard.

**Mitigation**:

1. Use **Query Reduction** settings:
   - By default, DON'T refresh the data
   - Add an **"Apply Filters"** button that clients must click
   - Only when they click does Power BI query Snowflake
   - This prevents accidental 10-second freezes

2. Use **DirectQuery with aggregations**:
   - Instead of importing 100M trade records
   - Pre-aggregate them in Snowflake (daily summaries)
   - Power BI queries the summaries (1K rows instead of 100M)

3. Add a **"Loading..."** spinner:
   - Be honest with users about latency
   - If a query takes 3 seconds, show a progress bar
   - Users have patience if you tell them why they're waiting

---

### Challenge 3: Advisor Spam - Tax Loss Alert Fatigue

**Problem**: 
Every stock price jitters down 0.1%, a tax-loss alert fires. Client's phone blows up with notifications. They mute all alerts. Real tax-saving opportunities are buried under noise.

**Mitigation**:

1. Add a **threshold filter**:
   ```dax
   Only_Alert_If_Savings_Significant = 
   IF([TaxSavingsIfHarvested] > 1000,  // Only alert if $1K+ savings
      TRUE,
      FALSE
   )
   ```
   - This eliminates false alarms for penny-stock losses

2. **Batch alerts**:
   - Instead of real-time notifications
   - Send one daily digest at 8am: "3 tax-loss opportunities today"
   - Client reviews once, not 50 times

3. **Advisor pre-screens**:
   - Before sending alert to client
   - Advisor reviews in Power BI: "Is this a real opportunity or tax-loss harvesting against the wash-sale rule?"
   - Only send alerts that have been pre-approved

---

### Challenge 4: API Rate Limits - Algorithmic Trading Meltdown

**Problem**: 
Hedge fund's trading algorithm queries the API 1,000 times per second. After 10 seconds, API hits rate limit. Trades stop executing.

**Mitigation**:

1. **Caching layer**:
   - Use Elasticache Redis to cache API responses
   - Set TTL (Time To Live) to 500ms:
     ```
     - First request at 14:30:00.000: Query Snowflake (fresh data)
     - Requests at 14:30:00.001-14:30:00.499: Serve from cache (instant, no rate limit)
     - Request at 14:30:00.500: Query Snowflake again (stale cache, fresh data)
     ```

2. **Batch endpoints**:
   - Instead of 1,000 individual API calls
   - Trading algorithm batches into 1 call: "Give me positions for symbols [AAPL, MSFT, GOOGL...]"
   - GraphQL makes this easy:
     ```graphql
     query {
       trades(symbols: ["AAPL", "MSFT", "GOOGL"]) { ... }
     }
     ```

3. **Tiered API quotas**:
   - Premium hedge funds: 10K requests/sec (highest SLA)
   - Standard hedge funds: 1K requests/sec
   - Retail wealth clients: 100 requests/min
   - If they exceed quota, request is queued (stays in buffer until quota frees up)

---

## PART 4: SECURITY & GOVERNANCE

### Data Isolation - "What-If" Sandbox

**For Consumer Wealth Clients**:

The sliders they play with (`RetirementAge`, `Asset Allocation Percentage`) are **purely virtual**. They only exist in the user's browser session (RAM). They have **zero write access** to the database.

When a client adjusts the slider from 65 to 70, what happens:
1. Slider moves in browser (client-side, instant)
2. DAX formula recalculates the projection (still client-side RAM)
3. New number displays: "If you retire at 70, you'll have $3.2M"
4. Number never gets written back to Snowflake ✅

**For Hedge Fund Quants**:

When their algorithm queries the API and receives position data, it's a **read-only snapshot**. The algorithm cannot write trades directly through the data API.

To execute a trade, the algorithm must:
1. Call a separate **Trade Execution API** (different endpoint)
2. PM must approve the order in Power BI
3. Only then does the order get written to Snowflake

This prevents rogue algorithms from accidentally (or maliciously) executing unauthorized trades.

---

### Compliance & Audit Trail

**For Every Power BI Interaction**:
- Log: User ID, Action (slider moved, alert clicked, report generated), Timestamp, New Value
- Store these logs in Snowflake `AUDIT_LOG` table
- Regulators can query: "Show me every time Client #12345 was shown a tax-loss alert"
- Proof: Alert was shown on 2026-02-15 14:23:00, client executed on 14:25:00, PM approved on 14:27:00

**For Every API Query**:
- Log the exact SQL query Power BI or the algorithm executed
- Log the exact data returned (checksum)
- Log the timestamp
- Regulators can audit: "Prove that on 2026-02-19, the hedge fund's algorithm was querying the correct portfolio data"

**For Trade Execution**:
- Immutable record: Who clicked "Execute Trade", what was the proposed trade, when was it approved, when was it executed
- Stored in SEC-compliant format (immutable, 7-year retention)

---

## PART 5: ROLLOUT STRATEGY

### Phase 1: Pilot (Month 1)

- **Retail Wealth**: 50 HNW clients, 1 mobile dashboard variant
- **Hedge Funds**: 3 client funds, Power BI desktop + API
- Collect feedback, identify bugs

### Phase 2: Scale (Month 2-3)

- **Retail Wealth**: Roll out to 5,000 wealth clients
- **Hedge Funds**: Roll out to 20 client funds
- Monitor performance, optimize queries

### Phase 3: Production (Month 4+)

- **Retail Wealth**: Open to all customer segments
- **Hedge Funds**: Become default analytics interface
- Continuous improvement based on usage

---

## CONCLUSION

We are not just building dashboards. We are building **personalized wealth engines**.

For retail clients, a dashboard that gamifies retirement planning and executes tax trades with a single tap.

For hedge funds, a synchronized pair of interfaces (visual + programmatic) that keeps PMs and robots perfectly in sync.

**The net effect**: 
- Clients feel more in control (interactive > static)
- Advisors spend less time explaining (visual > text)
- Algorithms trust the data (guaranteed consistency across channels)
- Nomura captures more AUM, more fees, more market share

Welcome to the future of wealth management. ✨

