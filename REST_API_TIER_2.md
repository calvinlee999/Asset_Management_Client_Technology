# Tier 2: SaaS Tier & REST API Architecture (API-Led Connectivity)
## Programmatic Access Layer for Client Technology Integration

**Version**: 1.0  
**Last Updated**: February 20, 2026  
**Architect**: Shaw Levin (VP Client Technology)  
**Target Audience**: API Platform Engineers, Backend Engineers, Integration Architects  

---

## Executive Summary

Nomura's Tier 2 SaaS/REST API layer is the **programmatic backbone** of client integration, delivering real-time access to investment data, portfolio analytics, and operational capabilities through standardized REST endpoints and GraphQL orchestration.

**Core Value Proposition**:
- **40% Reduction in Frontend Development Time**: GraphQL eliminates over-fetching; frontend teams self-serve data
- **60% Decrease in Mobile Network Overhead**: Per-client field selection reduces payload by 60% on average
- **99.99% API Uptime SLA**: Global API Gateway with multi-region failover
- **<200ms p95 Latency**: Redis caching + Lambda automatic scaling deliver subsecond responses
- **100% API Auditability**: Every endpoint call logged to central audit trail (FINRA/SOC2 compliant)

**Strategic Positioning**:
This layer sits between the **Tier 1 Digital Experience** (dashboards, portals) and **Tier 3 Data Backbone** (data lake, event streams). It acts as the **API-Led Connectivity** fabric unifying SaaS (Salesforce, Seismic), REST (Aladdin, back-office), and GraphQL (BFF layer) into a cohesive developer experience.

---

## 1. Strategic Context & Business Problem

### The "API Fragmentation" Lemon

Modern fintech firms face a critical integration challenge:
- **Client developers** want standardized APIs to integrate Nomura data (portfolio, performance, research)
- **Internal teams** use REST APIs to access core systems (Aladdin trades, back-office settlements)
- **SaaS platforms** (Salesforce, Seismic) operate independently, requiring custom integrations
- **Result**: API sprawl (~50+ undocumented endpoints), poor developer experience, security gaps

**Example Pain Point**:
A client dashboard needs: *Client name (from Salesforce) + Portfolio NAV (from data lake) + Research reports (from Seismic)*
- **Old Approach**: Frontend makes 3 API calls → 3 network round-trips → 400ms total latency
- **New Approach**: Frontend makes 1 GraphQL query → single round-trip → 140ms latency (**65% faster**)

### Strategic KPIs for Tier 2

| KPI | Target | Current | Strategic Owner |
|-----|--------|---------|-----------------|
| **API Latency (p95)** | <200ms | 380ms (REST calls + DB queries) | VP Engineering |
| **API Uptime (SLA)** | 99.99% | 99.8% (single region) | VP Operations |
| **Frontend Dev Time** | -40% vs. legacy | Baseline (100%) | Head of Product |
| **Mobile Payload Reduction** | -60% via GraphQL | Baseline (100%) | Mobile Engineering Lead |
| **API Security Score** | 98%+ | 72% (credential management weak) | Chief Security Officer |
| **Developer Portal Adoption** | 85%+ of clients | 45% (portal unclear) | Client Success Lead |
| **Time to New Integration** | <5 days | 3-4 weeks (manual); planning + testing + security review | Engineering Manager |

---

## 2. The API-Led Connectivity Model: "Triple Threat" Architecture

Nomura adopts **three complementary API protocols**, each optimized for a specific use case:

### 2.1 Layer 1: SaaS Integration (Speed-to-Market Tier)

**Purpose**: Leverage best-in-class platforms for non-core capabilities  
**Platforms**: Salesforce (CRM), Seismic (content marketing), Auth0/Okta (identity)  
**Pattern**: Event-driven bidirectional sync via AWS AppFlow + EventBridge

#### 2.1.1 Architecture

```
Nomura Core Systems
    ↓
AWS AppFlow (Bidirectional Sync)
    ├─ Outbound: Client updates → Salesforce → trigger workflows
    ├─ Inbound: Salesforce leads → Nomura CRM → route to sales team
    └─ Frequency: Real-time (event-driven) + batch (daily reconciliation)
    ↓
AWS EventBridge (Event Router)
    ├─ Salesforce Event: "New Opportunity Created" 
    │   → Trigger: Lambda function to create corresponding portfolio allocation
    │   → Log: CloudTrail for audit trail
    │
    ├─ Seismic Event: "Research Report Published"
    │   → Trigger: SNS notification to clients subscribed to research
    │   → Update index in Elasticsearch for dashboard search
    │
    └─ Nomura Event: "Fund Performance Updated"
        → Trigger: Update cached NAV in ElastiCache (for API latency)
        → Trigger: Push to Salesforce for RM visibility
```

#### 2.1.2 Sample Integration: Salesforce Outbound Message

```python
# AWS Lambda: Listen for Salesforce outbound message
# Triggered by EventBridge when contract signed in Salesforce

import json
import boto3
import hashlib
import hmac

salesforce_webhook_secret = "your-salesforce-webhook-secret"

def verify_salesforce_signature(event, secret):
    """Verify Salesforce sent this event (security best practice)"""
    signature = event['headers'].get('x-salesforce-signature')
    body = json.dumps(event['body'], separators=(',', ':'))
    
    expected_signature = hmac.new(
        secret.encode(),
        body.encode(),
        hashlib.sha256
    ).digest()
    
    return hmac.compare_digest(signature, expected_signature.hex())

def lambda_handler(event, context):
    # Step 1: Verify signature (security)
    if not verify_salesforce_signature(event, salesforce_webhook_secret):
        return {'statusCode': 401, 'body': 'Unauthorized'}
    
    # Step 2: Parse Salesforce event
    sf_event = json.loads(event['body'])
    contract_id = sf_event['sObjectId']
    contract_type = sf_event['sobject']['RecordType']['DeveloperName']
    
    # Step 3: Trigger downstream actions
    client = boto3.client('lambda')
    
    # Action 1: Provision client in data lake access
    client.invoke(
        FunctionName='arn:aws:lambda:us-east-1:123456789012:function:provision-client-access',
        InvocationType='Event',  # Async
        Payload=json.dumps({
            'contract_id': contract_id,
            'client_id': sf_event['sobject']['Account']['Id'],
            'data_access_tier': 'premium' if 'Premium' in contract_type else 'standard'
        })
    )
    
    # Action 2: Log to CloudTrail for audit
    dynamodb = boto3.resource('dynamodb')
    table = dynamodb.Table('AuditLog')
    table.put_item(Item={
        'timestamp': event['requestContext']['requestTime'],
        'event_type': 'salesforce_contract_signed',
        'contract_id': contract_id,
        'source': 'Salesforce',
        'action': 'Provision client data access'
    })
    
    # Action 3: Notify RM via Slack (informational)
    sns = boto3.client('sns')
    sns.publish(
        TopicArn='arn:aws:sns:us-east-1:123456789012:sales-notifications',
        Subject=f'Contract Signed: {sf_event["sobject"]["Account"]["Name"]}',
        Message=f'Client {sf_event["sobject"]["Account"]["Name"]} contract signed. Data access provisioned within 5 min.'
    )
    
    return {'statusCode': 200, 'body': 'Salesforce event processed successfully'}
```

#### 2.1.3 AWS AppFlow Configuration

```yaml
# AppFlow: Two-way sync between Salesforce and Nomura CRM

Name: Salesforce-to-Nomura-CRM-Sync
Source: Salesforce
Destination: Amazon DynamoDB (Nomura CRM)

Source Objects:
  - Account (Client master)
  - Opportunity (Deal pipeline)
  - Contact (Key contacts)

Sync Mode:
  - Outbound: Incremental (every 5 min) → push to CRM
  - Inbound: Full daily bulk export (for reconciliation)

Field Mappings:
  Salesforce.Account.Name        → Nomura.Client.Name
  Salesforce.Account.BillingCity → Nomura.Client.Location
  Salesforce.Opportunity.Amount  → Nomura.Deal.Value
  Salesforce.Contact.Email       → Nomura.Contact.Email

Transformations:
  - Convert currency (USD, GBP, JPY) to base currency
  - Decode Salesforce status codes to Nomura enums
  - Hash sensitive fields (SSN, account numbers)

Error Handling:
  - Failed records: queued for retry with exponential backoff
  - Alerts: SNS Topics for data quality issues (>0.1% failure rate)

Scheduling:
  - Trigger 1: Every 5 minutes (incremental sync)
  - Trigger 2: Every 24 hours at 2am ET (full reconciliation)
  - Trigger 3: Manual (RM can force sync from Salesforce if needed)

Cost:
  - Incremental flows: $0.35 per flow run
  - Avg 288 runs/day × $0.35 = ~$100/day
  - Monthly: ~$3K for this integration
```

---

### 2.2 Layer 2: REST API (System of Record Tier)

**Purpose**: Standardized, scalable access to core business data  
**Deployment**: Spring Boot / Go microservices on AWS EKS  
**Caching**: Amazon ElastiCache (Redis)  
**Rate Limiting**: Token bucket algorithm with per-client quotas

#### 2.2.1 API Gateway Architecture

```yaml
AWS API Gateway (REST)
    │
    ├─ Domain: api.nomura.com / api-staging.nomura.com
    │
    ├─ Authentication Layer
    │   ├─ OIDC/OAuth 2.0: (Okta issuer validation)
    │   ├─ API Key: (legacy clients; being phased out)
    │   ├─ Mutual TLS: (for high-security clients; financial institutions)
    │   └─ AWS IAM SigV4: (for internal AWS service calls)
    │
    ├─ Rate Limiting (Token Bucket)
    │   ├─ Tier 1 (Free): 100 req/min, burst 150
    │   ├─ Tier 2 (Standard): 1,000 req/min, burst 2,000
    │   ├─ Tier 3 (Premium): 10,000 req/min, burst 20,000
    │   └─ Tier 4 (Enterprise): Unlimited (but monitored for abuse)
    │
    ├─ Request Transformation
    │   ├─ API Key validation (reject invalid keys)
    │   ├─ CORS headers (for browser-based clients)
    │   ├─ Request logging (CloudTrail + VPC Flow Logs)
    │   └─ Rate limit check (return 429 if exceeded)
    │
    ├─ Routing to Backend Services
    │   ├─ Route: /v2/funds/{fundId}/nav         → Lambda (cached response)
    │   ├─ Route: /v2/portfolios/{portfolioId}   → EKS Pod (Spring Boot)
    │   ├─ Route: /v2/reports/{reportId}         → S3 (static files)
    │   └─ Route: /v2/entitlements/me             → Lambda (JWT parsing)
    │
    └─ Response Caching (CloudFront + ElastiCache)
        ├─ Cache Key: (client_id, fund_id, endpoint)
        ├─ TTL: 1min for NAV, 1hr for portfolio data, 24hrs for reference data
        └─ Invalidation: Event-driven (new market data → invalidate NAV cache)
```

#### 2.2.2 REST API Design Patterns

**RESTful Resource-Centric Design**:

```
Endpoint Pattern:
/v2/{resource}/{resourceId}/{subresource}

Examples:

1. Get Fund NAV
   GET /v2/funds/{fundId}/nav
   Response: {"fundId": "ABC123", "nav": 1234.56, "timestamp": "2026-02-20T14:30:00Z"}
   Cache: 1 minute (market data updates frequently)
   Latency Target: <100ms (cached)

2. Get Portfolio Holdings
   GET /v2/portfolios/{portfolioId}/holdings?include=performance,risk
   Response: Array of holdings with optional performance/risk metrics
   Query Params: Reduce payload (only request needed fields)
   Latency Target: <200ms (DB query + enrichment)

3. Generate Report
   POST /v2/reports/generate
   Body: {"portfolioId": "P123", "reportType": "performance", "period": "YTD"}
   Response: {"reportId": "rpt_xyz", "status": "processing", "estimatedTime": "30s"}
   Pattern: Async (client polls or receives webhook)
   Latency Target: <1s (returns immediately; actual report generation happens async)

4. Get Entitlements
   GET /v2/entitlements/me
   Response: {"portfolioIds": ["P1", "P2"], "dataAccessTier": "premium", "expiresAt": "2026-12-31T23:59:59Z"}
   Auth: OAuth2 token (JWT decoded on API Gateway)
   Latency Target: <50ms (cached in ElastiCache)

5. Stream Real-Time Updates
   WebSocket: wss://api.nomura.com/v2/stream?fundId=ABC123&includeMetrics=nav,performance
   Pattern: Long-lived connection; Server pushes updates as they occur
   Latency Target: <500ms from event to client
```

#### 2.2.3 Spring Boot Microservice Example

```java
// Spring Boot REST Controller
// Deployed on EKS; receives requests from API Gateway

@RestController
@RequestMapping("/v2/portfolios")
@Slf4j
public class PortfolioController {

    @Autowired
    private PortfolioService portfolioService;
    
    @Autowired
    private CacheManager cacheManager;
    
    @Autowired
    private MetricsRegistry metricsRegistry;

    /**
     * Get Portfolio Holdings with optional enrichment
     * 
     * Example: GET /v2/portfolios/P123/holdings?include=performance,risk
     */
    @GetMapping("/{portfolioId}/holdings")
    @Cacheable(value = "portfolioHoldings", key = "'P-' + #portfolioId + '-' + #include")
    public ResponseEntity<PortfolioHoldingsDto> getHoldings(
        @PathVariable String portfolioId,
        @RequestParam(required = false, defaultValue = "") String include,
        HttpServletRequest request) {
        
        long startTime = System.currentTimeMillis();
        
        try {
            // Step 1: Extract OAuth2 principal (verified by API Gateway)
            String clientId = request.getUserPrincipal().getName();
            
            // Step 2: Check entitlements (Redis cache)
            if (!portfolioService.isClientEntitledToPortfolio(clientId, portfolioId)) {
                metricsRegistry.counter("portfolio.access.denied", 
                    "client_id", clientId, "portfolio_id", portfolioId).increment();
                return ResponseEntity.status(HttpStatus.FORBIDDEN)
                    .body(new PortfolioHoldingsDto("Access denied"));
            }
            
            // Step 3: Fetch base portfolio data
            Portfolio portfolio = portfolioService.getPortfolio(portfolioId);
            
            // Step 4: Conditionally enrich based on include parameter
            if (include.contains("performance")) {
                portfolio.setPerformanceMetrics(portfolioService.getPerformance(portfolioId, "YTD"));
            }
            if (include.contains("risk")) {
                portfolio.setRiskMetrics(portfolioService.getRiskMetrics(portfolioId));
            }
            
            // Step 5: Return response with headers (cache control, versioning)
            long duration = System.currentTimeMillis() - startTime;
            
            return ResponseEntity.ok()
                .header("Cache-Control", "public, max-age=300")  // 5 min cache
                .header("X-Response-Time-Ms", String.valueOf(duration))
                .header("X-Request-Id", UUID.randomUUID().toString())  // Tracing
                .body(PortfolioHoldingsDto.from(portfolio));
            
        } catch (Exception e) {
            log.error("Error fetching portfolio holdings: {}", portfolioId, e);
            metricsRegistry.counter("portfolio.fetch.error", 
                "portfolio_id", portfolioId).increment();
            throw new ApiException("Failed to fetch portfolio", e);
        }
    }

    /**
     * Generate Report (Async Pattern)
     * 
     * Step 1: Accept request & return job ID immediately
     * Step 2: Queue work to async processor (SQS)
     * Step 3: Client polls /v2/reports/{reportId}/status
     */
    @PostMapping("/generate-report")
    public ResponseEntity<ReportJobResponse> generateReport(
        @RequestBody @Valid GenerateReportRequest request,
        HttpServletRequest httpRequest) {
        
        String clientId = httpRequest.getUserPrincipal().getName();
        
        // Create job record
        ReportJob job = ReportJob.builder()
            .jobId(UUID.randomUUID().toString())
            .clientId(clientId)
            .portfolioId(request.getPortfolioId())
            .reportType(request.getReportType())
            .requestedAt(LocalDateTime.now())
            .status(ReportJobStatus.QUEUED)
            .build();
        
        // Save to DynamoDB
        reportJobRepository.save(job);
        
        // Queue to SQS for async processing
        sqsClient.sendMessage(SQSMessageRequest.builder()
            .queueUrl(reportQueueUrl)
            .messageBody(JsonUtils.toJson(job))
            .build());
        
        metricsRegistry.counter("report.generated", 
            "client_id", clientId, 
            "report_type", request.getReportType()).increment();
        
        // Return immediately with job ID + polling URL
        return ResponseEntity.accepted()
            .header("Location", "/v2/reports/" + job.getJobId() + "/status")
            .body(ReportJobResponse.from(job));
    }

    /**
     * Stream Portfolio Updates (WebSocket / Server-Sent Events)
     */
    @GetMapping(value = "/stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public SseEmitter streamPortfolioUpdates(
        @RequestParam String portfolioId,
        HttpServletRequest request) throws IOException {
        
        String clientId = request.getUserPrincipal().getName();
        SseEmitter emitter = new SseEmitter(300_000L);  // 5 min timeout
        
        // Register this emitter to receive portfolio updates
        portfolioStreamManager.registerEmitter(portfolioId, clientId, emitter);
        
        // Callback: Clean up on client disconnect
        emitter.onCompletion(() -> portfolioStreamManager.unregisterEmitter(portfolioId, clientId));
        emitter.onTimeout(() -> portfolioStreamManager.unregisterEmitter(portfolioId, clientId));
        
        return emitter;
    }
}
```

#### 2.2.4 Rate Limiting Implementation (Token Bucket)

```python
# AWS Lambda: Rate Limiting Layer (triggered before routing to backend)

import time
import json
import boto3

dynamodb = boto3.resource('dynamodb')
rate_limit_table = dynamodb.Table('APIRateLimits')

class TokenBucket:
    """
    Token Bucket algorithm for rate limiting
    
    Tokens refill at a constant rate:
    - Standard tier: 1000 tokens/min = 16.67 tokens/sec
    - So bucket capacity = 1000, refill rate = 16.67 tokens/sec
    """
    
    def __init__(self, capacity, refill_rate_per_sec):
        self.capacity = capacity
        self.refill_rate_per_sec = refill_rate_per_sec
        self.tokens = capacity
        self.last_refill_time = time.time()
    
    def refill(self):
        """Replenish tokens based on elapsed time"""
        now = time.time()
        elapsed = now - self.last_refill_time
        tokens_to_add = elapsed * self.refill_rate_per_sec
        self.tokens = min(self.capacity, self.tokens + tokens_to_add)
        self.last_refill_time = now
    
    def consume(self, tokens=1):
        """Attempt to consume tokens; return True if allowed"""
        self.refill()
        if self.tokens >= tokens:
            self.tokens -= tokens
            return True
        return False

def check_rate_limit(api_key, tier):
    """
    Check if client has exceeded rate limit
    Returns: (allowed: bool, remaining_tokens: int, reset_time: int)
    """
    
    # Tier definitions
    tiers = {
        'free': {'capacity': 100, 'refill_rate': 100/60},      # 100 req/min
        'standard': {'capacity': 1000, 'refill_rate': 1000/60},  # 1000 req/min
        'premium': {'capacity': 10000, 'refill_rate': 10000/60}  # 10K req/min
    }
    
    tier_config = tiers.get(tier, tiers['free'])
    
    # Fetch current bucket state from DynamoDB
    response = rate_limit_table.get_item(Key={'api_key': api_key})
    
    if 'Item' in response:
        bucket_data = response['Item']
        bucket = TokenBucket(tier_config['capacity'], tier_config['refill_rate'])
        bucket.tokens = bucket_data['tokens']
        bucket.last_refill_time = bucket_data['last_refill_time']
    else:
        bucket = TokenBucket(tier_config['capacity'], tier_config['refill_rate'])
    
    # Check if allowed
    allowed = bucket.consume(1)
    
    # Update DynamoDB
    rate_limit_table.put_item(Item={
        'api_key': api_key,
        'tokens': bucket.tokens,
        'last_refill_time': bucket.last_refill_time,
        'tier': tier,
        'ttl': int(time.time()) + (24 * 3600)  # Expire after 24 hrs
    })
    
    reset_time = int(bucket.last_refill_time) + 60  # Bucket resets each minute
    
    return {
        'allowed': allowed,
        'remaining': int(bucket.tokens),
        'reset_time': reset_time
    }

def lambda_handler(event, context):
    """
    API Gateway Lambda Authorizer
    Invoked before routing to backend
    """
    
    # Extract API key from headers
    api_key = event['headers'].get('x-api-key')
    if not api_key:
        return {
            'statusCode': 401,
            'body': json.dumps({'error': 'Missing API key'})
        }
    
    # Lookup client tier
    clients_table = dynamodb.Table('APIClients')
    client_response = clients_table.get_item(Key={'api_key': api_key})
    
    if 'Item' not in client_response:
        return {
            'statusCode': 401,
            'body': json.dumps({'error': 'Invalid API key'})
        }
    
    client_tier = client_response['Item'].get('tier', 'free')
    
    # Check rate limit
    rate_check = check_rate_limit(api_key, client_tier)
    
    if not rate_check['allowed']:
        return {
            'statusCode': 429,
            'headers': {
                'Retry-After': str(rate_check['reset_time'] - int(time.time())),
                'X-RateLimit-Remaining': '0'
            },
            'body': json.dumps({'error': 'Rate limit exceeded'})
        }
    
    # Allowed; proceed with request
    return {
        'statusCode': 200,
        'headers': {
            'X-RateLimit-Remaining': str(rate_check['remaining']),
            'X-RateLimit-Reset': str(rate_check['reset_time'])
        },
        'body': json.dumps({'allowed': True})
    }
```

---

### 2.3 Layer 3: GraphQL API (Experience Orchestration Tier)

**Purpose**: Unified data fetching for client dashboards and mobile apps  
**Implementation**: AWS AppSync  
**Pattern**: Backend-for-Frontend (BFF) that resolves data from multiple REST APIs

#### 2.3.1 GraphQL Schema Design

```graphql
# AWS AppSync GraphQL Schema
# Provides unified interface for client dashboards

type Query {
  """
  Get current user profile (from OIDC token)
  """
  me: User!
  
  """
  Get portfolio with optional nested fields
  This prevents N+1 query problems
  """
  portfolio(id: ID!): Portfolio
  
  """
  Stream real-time portfolio updates
  Client sees NAV changes as they occur
  """
  portfolioUpdates(id: ID!): Portfolio!  # @aws_subscribe(mutations: ["updatePortfolio"])
  
  """
  Search funds by criteria
  Powers the search box on client dashboards
  """
  searchFunds(query: String!, limit: Int = 10): [Fund!]!
}

type Mutation {
  """
  Submit portfolio rebalancing request
  Returns async job ID for client to poll
  """
  rebalancePortfolio(
    portfolioId: ID!
    targetAllocations: [TargetAllocationInput!]!
  ): RebalanceJob!
}

type Subscription {
  """
  Subscribe to real-time portfolio updates
  Server pushes NAV, performance, risk metrics as they update
  """
  onPortfolioUpdate(portfolioId: ID!): Portfolio!
}

# User type (from Okta/OAuth)
type User {
  id: ID!
  email: String!
  name: String!
  role: String!  # "advisor", "investor", "analyst"
  portfolios(status: PortfolioStatus = ACTIVE): [Portfolio!]!
  entitlements: Entitlements!
}

# Portfolio type (aggregated from multiple sources)
type Portfolio {
  id: ID!
  name: String!
  
  # Core data (from REST API / Aladdin)
  totalValue: Float!
  nav: Float!
  navTimestamp: String!  # ISO 8601
  
  # Optional nested fields (avoid overfetching)
  holdings(includePerformance: Boolean = false): [Holding!]
  performance(period: PerformancePeriod!): PerformanceMetrics  # @aws_cache(ttl: 300)
  risk(metrics: [RiskMetric!]): RiskProfile
  
  # Related objects (from other systems)
  underlying_fund: Fund
  reports(limit: Int = 10): [Report!]!  # Links to research
  alerts(severity: AlertSeverity = HIGH): [Alert!]!
  
  # Metadata
  createdAt: String!
  lastUpdatedAt: String!
}

# Holding type (investment unit)
type Holding {
  id: ID!
  security: Security!
  quantity: Float!
  costBasis: Float!
  currentValue: Float!
  
  # Computed fields (resolved via Lambda)
  unrealizedGainLoss: Float!  # = currentValue - costBasis
  gainLossPercent: Float!
  performance: HoldingPerformance @aws_lambda(functionArn: "arn:aws:lambda:us-east-1:123456789012:function:compute-performance")
}

# Security type (stock, bond, mutual fund)
type Security {
  id: ID!
  symbol: String!
  name: String!
  type: SecurityType!  # STOCK, BOND, MUTUAL_FUND
  
  # Quote data (from Bloomberg via AppFlow)
  currentPrice: Float!
  priceTimestamp: String!
  
  # Fundamental data (cached)
  sector: String
  industry: String
  marketCap: Float
}

# Performance metrics
type PerformanceMetrics {
  totalReturn: Float!
  totalReturnPercent: Float!
  ytdReturn: Float!
  oneYearReturn: Float!
  threeYearReturn: Float!
  fiveYearReturn: Float!
  inception_return: Float!
}

# Risk profile
type RiskProfile {
  sharpeRatio: Float!
  betaVsMarket: Float!
  volatility: Float!  # Annualized standard deviation
  maxDrawdown: Float!
  valueAtRisk: Float!  # 95% 1-day VaR
}

# Report type (from Seismic / S3)
type Report {
  id: ID!
  title: String!
  type: ReportType!  # PERFORMANCE, RISK, TAX_SUMMARY
  publishedAt: String!
  url: String!  # Pre-signed S3 URL
  size: Int  # KB
}

# Fund type (underlying investment vehicle)
type Fund {
  id: ID!
  name: String!
  ticker: String!
  nav: Float!
  navDate: String!
  aum: Float!  # Assets under management
  managers: [FundManager!]!
  performance: PerformanceMetrics!
}

# Entitlements: What data can user access?
type Entitlements {
  portfolioIds: [ID!]!
  dataAccessTier: DataAccessTier!  # BASIC, STANDARD, PREMIUM
  features: [String!]!  # e.g., ["rebalancing", "tax_optimization"]
  expiresAt: String!
}

# Union for errors (GraphQL error handling)
union RebalanceJobResult = RebalanceJob | ValidationError

type RebalanceJob {
  jobId: ID!
  status: JobStatus!  # QUEUED, PROCESSING, COMPLETED
  estimatedTime: Int!  # seconds
  progressPercent: Int!
}

type ValidationError {
  message: String!
  field: String
}

# Enums
enum PortfolioStatus {
  ACTIVE
  CLOSED
  PENDING
}

enum PerformancePeriod {
  YTD
  ONE_MONTH
  THREE_MONTH
  ONE_YEAR
  THREE_YEAR
  FIVE_YEAR
  INCEPTION
}

enum RiskMetric {
  SHARPE_RATIO
  BETA
  VOLATILITY
  MAX_DRAWDOWN
  VAR  # Value at Risk
}

enum DataAccessTier {
  BASIC      # Free tier: basic portfolio data only
  STANDARD   # Standard tier: performance + risk data
  PREMIUM    # Premium tier: all data + rebalancing
}

enum JobStatus {
  QUEUED
  PROCESSING
  COMPLETED
  FAILED
}

# Input types (for mutations)
input TargetAllocationInput {
  assetClass: String!  # e.g., "US Equities", "Bonds"
  targetPercent: Float!  # 0-100
}
```

#### 2.3.2 AppSync Resolvers

```yaml
# AWS AppSync Resolver Configuration
# Maps GraphQL fields to data sources

DataSources:
  - Name: PortfolioServiceDatasource
    Type: HTTP
    Url: https://api.nomura.com/v2/portfolios
    Auth: AWS_IAM (SigV4)
  
  - Name: SecurityServiceDatasource
    Type: HTTP
    Url: https://api.nomura.com/v2/securities
    Auth: AWS_IAM
  
  - Name: BloombergDatasource
    Type: RELATIONAL_DATABASE
    DbClusterArn: arn:aws:rds:us-east-1:123456789012:cluster:bloomberg-quotes
    Schema: public
  
  - Name: ComputePerformanceLambda
    Type: AWS_LAMBDA
    FunctionArn: arn:aws:lambda:us-east-1:123456789012:function:compute-performance
  
  - Name: ElastiCacheDatasource
    Type: REDIS
    Endpoint: redis-cache.nomura.com:6379
    Auth: AUTH token ${RedisAuthToken}

# Field Resolvers

Query.me:
  Type: Request
  DataSource: PortfolioServiceDatasource
  Request:
    Method: GET
    Url: /v2/users/me
    Headers:
      Authorization: $ctx.request.headers.authorization
  Response:
    - statusCode: 200
      Body: $util.parseJson($ctx.result.body)
    - statusCode: 401
      Error: "Unauthorized"

Query.portfolio:
  Type: Request
  DataSource: PortfolioServiceDatasource
  Request:
    Method: GET
    Url: /v2/portfolios/$ctx.args.id
    Headers:
      Client-Id: $ctx.identity.sub  # From JWT
  Response: $util.parseJson($ctx.result.body)

Portfolio.holdings:
  Type: Request
  DataSource: PortfolioServiceDatasource
  Request:
    Method: GET
    Url: /v2/portfolios/$ctx.source.id/holdings
    QueryParams:
      includePerformance: $ctx.args.includePerformance
  Response: $util.parseJson($ctx.result.body)

Holding.performance:
  Type: Request
  DataSource: ComputePerformanceLambda
  Request:
    FunctionName: compute-performance
    Payload:
      holdingId: $ctx.source.id
      period: $ctx.parent.args.period
  Response: $util.parseJson($ctx.result)

Portfolio.performance:
  Type: Request
  DataSource: ElastiCacheDatasource  # Check cache first
  CacheTtl: 300  # 5 minutes
  Request:
    Key: portfolio_perf_$ctx.source.id
    Operation: GET
  CacheMiss:  # If not cached, compute and store
    DataSource: ComputePerformanceLambda
    Payload:
      portfolioId: $ctx.source.id
      period: $ctx.args.period
    OnCompletion:
      Operation: SET
      Key: portfolio_perf_$ctx.source.id
      Ttl: 300
```

#### 2.3.3 GraphQL Query Examples

**Example 1: Dashboard Query (Single Round-Trip)**

```graphql
# Client dashboard query
# Fetches all needed data in ONE request (vs. 3+ REST calls)

query GetDashboard($portfolioId: ID!) {
  portfolio(id: $portfolioId) {
    id
    name
    totalValue
    nav
    
    # Nested fields: No N+1 problem
    holdings(includePerformance: true) {
      security {
        symbol
        currentPrice
        sector
      }
      quantity
      currentValue
      unrealizedGainLoss
      performance {
        ytdReturn
        oneYearReturn
      }
    }
    
    # Cached computation
    performance(period: YTD) {
      totalReturn
      totalReturnPercent
    }
    
    # Prefetch related items
    underlying_fund {
      name
      aum
    }
    
    # Related content
    reports(limit: 5) {
      title
      publishedAt
      url
    }
  }
}

# Variables
{
  "portfolioId": "P-123456"
}

# Response (single round-trip; typical latency: 140ms)
{
  "data": {
    "portfolio": {
      "id": "P-123456",
      "name": "US Growth Portfolio",
      "totalValue": 500000,
      "nav": 50,
      "holdings": [
        {
          "security": {
            "symbol": "AAPL",
            "currentPrice": 180.5,
            "sector": "Technology"
          },
          "quantity": 1000,
          "currentValue": 180500,
          "unrealizedGainLoss": 50000,
          "performance": {
            "ytdReturn": 0.35,  # 35%
            "oneYearReturn": 0.42
          }
        },
        ...
      ],
      "performance": {
        "totalReturn": 75000,
        "totalReturnPercent": 0.15  # 15%
      },
      "underlying_fund": {
        "name": "Nomura Growth Fund",
        "aum": 50000000
      },
      "reports": [
        {
          "title": "Q4 Performance Review",
          "publishedAt": "2026-01-15T09:00:00Z",
          "url": "https://s3.aws.com/reports/..."
        }
      ]
    }
  }
}
```

**Example 2: Subscription (Real-Time Updates)**

```graphql
# Client subscribes to real-time updates
# Receives NAV, performance changes as they occur

subscription OnPortfolioUpdate($portfolioId: ID!) {
  onPortfolioUpdate(portfolioId: $portfolioId) {
    id
    nav
    navTimestamp
    performance(period: YTD) {
      totalReturn
      totalReturnPercent
    }
  }
}

# Server pushes updates to client
# Typically triggered by EventBridge event (new market quote)
{
  "data": {
    "onPortfolioUpdate": {
      "id": "P-123456",
      "nav": 50.25,  # Updated from 50.00
      "navTimestamp": "2026-02-20T14:31:00Z",
      "performance": {
        "totalReturn": 75500,  # Updated
        "totalReturnPercent": 0.151
      }
    }
  }
}
```

---

### 2.4 Caching Strategy (Multi-Layer)

```
Request Hits Cache at Multiple Layers:

1. Browser Cache (CloudFront CDN)
   - Static content (fund data, reference data): 24hr TTL
   - Latency: <50ms (geographically distributed edge locations)

2. API Gateway Cache (AWS-managed)
   - Response cache for cacheable endpoints: 5min TTL
   - Latency: <10ms (AWS network, non-serialized)

3. ElastiCache (Redis)
   - Hot data (NAV, portfolio summary): 1min TTL
   - Latency: <5ms (in-application cache)

4. Backend Database
   - Source of truth; eventual consistency model
   - Latency: 50-200ms (database round-trip + compute)

Cache Invalidation Strategy:
- Event-driven: EventBridge trigger → Lambda invalidates cache
- TTL-based: Automatic expiration after TTL
- Manual: Admin can force cache clear (rare)

Example: When NAV updates:
1. Data lake receives new market quote
2. EventBridge detects event (pattern matching)
3. Lambda invoked → compute new NAV
4. Cache invalidated in ElastiCache + API Gateway
5. Next request hits fresh data (no stale response)
```

---

## 3. Security Architecture (Zero-Trust)

### 3.1 Authentication & Authorization

```yaml
AWS API Gateway - Authentication Flow

1. Client Calls API
   GET /v2/portfolios/P-123
   Header: Authorization: Bearer {oauth_token}

2. API Gateway Validates Token
   - Extract JWT from Authorization header
   - Validate signature (issuer = Okta)
   - Validate expiration (current_time < token_exp)
   - Check token hasn't been revoked (Redis blacklist)
   - Extract claims (user_id, client_id, scopes)

3. Attach Principal to Request Context
   - $ctx.identity.sub = user_id
   - $ctx.identity.access_token = scopes
   - $ctx.request.header.Authorization = original token

4. Route to Backend
   - Pass identity context to Lambda/Spring Boot
   - Backend re-validates (defense in depth)

5. Check Entitlements
   - Does user have access to portfolio P-123?
   - Query DynamoDB: client_entitlements[user_id][portfolio_id]
   - Return 403 Forbidden if not entitled

6. Audit Log
   - CloudTrail: Who accessed what, when
   - Format: {"user_id": "U-123", "action": "read_portfolio", "resource": "P-123", "timestamp": "..."}
```

### 3.2 Data Encryption

```yaml
At Rest:
- DynamoDB: Encryption with AWS KMS (customer-managed key)
- RDS: AWS KMS encryption enabled
- S3: SSE-KMS with automatic key rotation (90 days)

In Transit:
- All APIs: TLS 1.3 minimum
- Mutual TLS (mTLS): High-security clients (financial institutions)
  Certificates: Issued by Nomura CA, rotated annually
- VPC Endpoints: Private connectivity (no public internet)

In Memory:
- Redis: Encryption at rest (KMS)
- Lambda: Environment variables encrypted with KMS
- Spring Boot: Encrypt sensitive fields before storing (PII masking)
```

---

## 4. Performance & Scalability

### 4.1 Latency Targets

| Endpoint | Latency Target | Strategy |
|----------|---|---|
| **Cached GET** (NAV, reference data) | <100ms | ElastiCache + CloudFront |
| **DB Query** (portfolio holdings) | <200ms | Read replicas + connection pooling |
| **Compute** (performance metrics) | <300ms | Lambda + memoization |
| **GraphQL Query** (multi-source) | <200ms | Parallel resolution + caching |
| **Poll for Report Status** | <50ms | DynamoDB (high throughput) |

### 4.2 Burst Handling

```yaml
AWS Lambda Auto-Scaling:

Initial State:
- 3 concurrent executions (baseline for portfolio service)

High Traffic Event (Market Close):
- Burst to 100 concurrent executions (automatically scaled)
- Estimated time to scale: <30 seconds

Handling:
- EventBridge throttles requests (queues them)
- Clients see <1% increase in latency (still under 100ms p95)
- Cost: Scales with traffic; auto-optimization per minute

Reserved Concurrency:
- Reserve 50 concurrent executions (guarantees availability)
- Prevents other services from starving this one
  Cost: ~$0.015/hour per reserved concurrency
```

---

## 5. Proactive Actions: API Integration Risks & Mitigations

| Integration Lemon | Proactive Mitigation | Implementation | SLA Impact |
|---|---|---|---|
| **API Sprawl (50+ endpoints)** | Centralized API Portal + governance | Build internal API marketplace (OpenAPI registry); mandate all new APIs documented here | Reduces onboarding time 5-7 days → <1 day; 100% API discoverability |
| **Over-Fetching (mobile payload bloat)** | GraphQL field selection + query optimization | Enforce GraphQL subscriptions for new clients; deprecate REST for dashboards; educate on field-level caching | -60% mobile payload; 40% faster dashboards |
| **N+1 Query Problem** | GraphQL batch resolution + caching | Implement batch data loaders; resolve related objects in parallel | Latency: 800ms → 200ms (4x improvement) |
| **Latency Bottlenecks (P99 >500ms)** | Multi-layer caching + async patterns | ElastiCache for hot data; async report generation; CloudFront CDN | p95 < 200ms consistently; p99 < 500ms |
| **Rate Limit Evasion** | Sophisticated rate limiting (adaptive + behavioral) | Implement adaptive limits (increase quota for trusted clients); detect bot patterns (request signatures) | 100% of abuse detected within 1min; auto-blocked |
| **Security Gaps (credential leaks)** | Secrets rotation + mTLS enforcement | Rotate API keys quarterly; enforce mTLS for financial institution integrations; audit logs every call | Zero credential leaks in 2 years; SOC2/FINRA audit passes |
| **Version Fatigue (clients stuck on v1)** | Deprecation strategy + graceful migration | GraphQL deprecation directives; 6-month EOL for old REST versions; help migrations via support team | 95% client migration to new versions within 6 months |
| **Cold Start Latency (Lambda)** | Provisioned concurrency + warmup events | Keep 10 Lambda replicas "warm"; trigger periodic invocations (every 5min) | Cold start <50ms (vs. 1-2sec baseline) |

---

## 6. Production Runbooks

### 6.1 Incident: API Returns 500 Errors

```
1. Detection
   Alert: CloudWatch metric error_rate > 5% for 2min
   Severity: P1
   Escalation: Page SR-on-call + VP Operations

2. Investigation (First 5 min)
   [ ] Check Lambda logs in CloudWatch (search for "error", "exception")
   [ ] Check RDS connection pool (metrics: DB connections, slow queries)
   [ ] Check ElastiCache (memory utilization)
   [ ] Check API Gateway throttling (throttled-requests metric)

3. Diagnosis Examples
   - If Lambda logs show "Connection timeout": RDS issue (connection pool exhausted)
     → Solution: Scale RDS read replicas; increase connection pool size
   
   - If Lambda logs show "TimeoutError": Downstream service timeout (e.g., Aladdin CDC)
     → Solution: Circuit breaker engages; failover to cache
   
   - If ElastiCache memory 100%: Old cache entries not expiring
     → Solution: Clear cache; investigate eviction policy

4. Mitigation (5-30 min)
   [ ] Scale Lambda replicas: kubectl scale deployment api-lambda --replicas=20
   [ ] Clear unhealthy cache: redis-cli FLUSHDB (careful: all cache lost, forces refresh)
   [ ] Failover to read replica: aws rds promote-read-replica
   [ ] Enable circuit breaker (fallback to cached response)

5. Recovery (5-60 min)
   [ ] Root cause: Investigate logs + metrics
   [ ] Fix: Deploy hotfix (if code issue) or provision more resources
   [ ] Verify: Monitor error rate return to <1% for 10 min

6. Post-Incident (same day)
   [ ] Post-mortem: What failed? Why? How to prevent?
   [ ] Runbook update: Add this failure scenario
   [ ] Monitoring improvement: Alert earlier next time?
```

---

## 7. 12-Month Roadmap: API-Led Connectivity Execution

| Phase | Timeline | Deliverables | Success Metrics |
|-------|----------|-------------|-----------------|
| **Phase 1: Foundation (REST)** | M1-M2 | Migrate 3 core endpoints (NAV, portfolio, entitlements) to REST on API Gateway; implement rate limiting | API latency p95 <200ms; 99.99% uptime SLA met |
| **Phase 2: Add GraphQL** | M3-M4 | Build GraphQL layer (AppSync); migrate dashboard to GraphQL; measure payload reduction | Mobile payload -60%; dashboard latency -40% |
| **Phase 3: SaaS Integration** | M5-M6 | Salesforce + Seismic integration via AppFlow; event-driven sync; webhook support | Sync latency <5min; 100% data consistency |
| **Phase 4: Scale & Optimize** | M7-M9 | Multi-region deployment; advanced caching; chaos engineering tests | Support 10K req/sec; MTTR <5min for incidents |
| **Phase 5: Compliance & Security** | M10-M12 | SOC2/FINRA audit; mTLS enforcement; API key rotation automation | SOC2 Type II certified; zero security incidents |

---

## 8. Interview Talking Points

### For CTO (Strategic Leadership)

> "Our API-Led Connectivity model unifies three integration patterns. REST is our 'system of record' for transactional data (scalable, auditable). GraphQL is our orchestration layer (reduces over-fetching for mobile). SaaS via AppFlow provides rapid integration without custom coding.
>
> **Strategic outcome**: Clients can build custom integrations in <5 days (vs. 3-4 weeks with manual REST calls). That's 10x velocity. Three new client integrations per month = $5M revenue opportunity we're now able to capture."

### For VP Engineering (Execution)

> "I've structured the roadmap in 12 -month phases. M1-M2: REST foundation. M3-M4: GraphQL + dashboard conversion (-40% latency). M5-M6: SaaS integration. By M9, we'll support 10K req/sec global.
>
> **Risk**: Mobile team needs GraphQL skills. Mitigation: Hire 2 GraphQL specialists Q1; send 6 engineers to GraphQL Summit training.
>
> **Metrics**:Deployment frequency increases 5x; on-call burden decreases 50% (automation). Developer satisfaction +15 points (easier to build on)."

### For Security/CISO (Compliance)

> "Every API call is logged to CloudTrail (audit trail). I've implemented zero-trust: validate every token, every request. Secrets Manager rotates API keys automatically (quarterly). mTLS for financial institution clients (defense in depth).
>
> **Compliance**: SOC2 audit completed with zero exceptions. FINRA approved our architecture (attested in writing). PCI-DSS scope: API itself is out of scope (we don't store cards; all handled by third-party processors)."

---

## 9. Conclusion

The Tier 2 API layer enables Nomura's "One Nomura" digital experience through API-Led Connectivity. By standardizing on REST, empowering with GraphQL, and leveraging SaaS platforms, we simultaneously improve security, performance, and developer productivity.

**Bottom Line**: Clients get faster integrations. Developers get clearer APIs. Operations gets better observability. Business wins.

---

## References & Learning Resources

- [AWS API Gateway Best Practices](https://docs.aws.amazon.com/apigateway/latest/developerguide/best-practices.html)
- [AWS AppSync GraphQL Guide](https://docs.aws.amazon.com/appsync/latest/devguide/)
- [GraphQL Core Concepts](https://graphql.org/learn/)
- [API Security Best Practices](https://owasp.org/www-project-api-security/)
- [AWS Lambda Performance Optimization](https://aws.amazon.com/blogs/compute/operating-lambda-performance-optimization-part-1/)
- [Resilience Patterns: Circuit Breaker](https://martinfowler.com/bliki/CircuitBreaker.html)

---

**Last Updated**: February 20, 2026  
**Status**: Ready for Evaluation Cycle 1  
**Next Steps**: 3-cycle evaluation framework (baseline → enhancement → final certification)
