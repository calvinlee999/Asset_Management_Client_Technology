# AWS Data Exchange (ADX) Marketplace Tier: Zero-ETL Architecture
## Chief Enterprise Architect Edition - Levin's Pattern Implementation

**Date**: February 18, 2026  
**Target Architecture**: Petabyte-Scale, Zero-Copy Data Distribution for Institutional Clients  
**Business KPI**: Enable 60% client onboarding reduction; 100% automated entitlements

---

## Executive Summary

The **Marketplace Tier** represents the strategic evolution of Nomura's data distribution from traditional "push-based" ETL models (FTP/SFTP/manual exports) to a **Zero-Copy, Petabyte-Scale, Metadata-Only Exchange**. 

This tier embodies three architectural innovations pioneered at AWS during Pete Levin's tenure:
1. **Zero-ETL Architecture** (Metadata-Only Exchange with Redshift Data Sharing)
2. **Unified API Pattern** (ADX for APIs with IAM-based authorization)
3. **Event-Driven Lifecycle** (CloudWatch/EventBridge triggers for near-real-time value delivery)

### Why This Matters
- **Cost KPI**: Nomura pays for storage only; clients pay for compute. Eliminates egress charges at petabyte scale.
- **Speed KPI**: Clients onboard in minutes, not weeks (from custom ETL development to live queries)
- **Security KPI**: IAM Policy-Based Grants replace raw credential sharing (zero-trust by default)
- **Customer Experience**: "Frictionless Alpha" — institutional clients access proprietary insights directly into their AWS environments

---

## Section 1: The Zero-ETL Architecture (Petabyte-Scale)

### 1.1 Core Pattern: Metadata-Only Exchange

**Traditional Model** (Legacy - Push-Based):
```
Nomura (Provider) → ETL Extract → Copy Data → S3/Cloud → Customer FTP → Customer Data Warehouse
Issues: 
  - Weeks of custom ETL development per client
  - Egress costs: $0.02/GB × petabytes = $20M+ annual
  - Data staleness: Daily batch cycles
  - Manual credential management
```

**Modern Model** (Levin's Zero-ETL):
```
Nomura (Provider) → S3/Redshift (Master Location) ← Live Query ← Customer AWS Account
  ↓ (IAM Grant, not copy)
Customer Query Engine (in customer's AWS environment)

Benefits:
  - Instant access (metadata pointers, not data copies)
  - Zero egress cost (metadata overhead << data copy)
  - Real-time freshness (customer queries latest data)
  - IAM-native security (customer uses own AWS roles)
```

### 1.2 Technical Architecture: Redshift Data Sharing + S3 Data Access

#### Layer 1: Nomura's Master Data Layer (Provider)

```plaintext
┌─────────────────────────────────────────────────────┐
│         Nomura Asset Management Platform             │
├─────────────────────────────────────────────────────┤
│                                                      │
│  S3 Gold Layer (Medallion Architecture)             │
│  ├─ s3://nomura-data-lake/gold/nav/daily/          │
│  ├─ s3://nomura-data-lake/gold/performance/        │
│  ├─ s3://nomura-data-lake/gold/risk-factors/       │
│  └─ s3://nomura-data-lake/gold/esg-insights/       │
│                                                      │
│  Redshift Cluster (Provider Warehouse)              │
│  ├─ Schema: data_products                           │
│  ├─ Table: funds_nav (with UNLOAD to S3)           │
│  ├─ Table: performance_metrics                      │
│  └─ Table: benchmark_indices                        │
│                                                      │
│  Lake Formation (Governance)                        │
│  ├─ Data Catalog (Glue)                            │
│  ├─ Access Policies (Row-Level Security)           │
│  └─ Audit Logs (all query activity)                │
│                                                      │
└─────────────────────────────────────────────────────┘
         ↓ (IAM Grants via ADX)
```

#### Layer 2: AWS Data Exchange (ADX) - The Broker

```plaintext
┌──────────────────────────────────────────────────────┐
│    AWS Data Exchange (ADX): Entitlement Broker       │
├──────────────────────────────────────────────────────┤
│                                                       │
│  Data Product: "Nomura ESG Insights"                │
│  ├─ Revision: v2024-02-18 (Latest)                 │
│  ├─ Provider: Nomura (AWS Account: 111111111111)   │
│  ├─ Subscriber Policy:                             │
│  │   {                                              │
│  │     "Action": ["s3:GetObject", "s3:ListBucket"],│
│  │     "Resource": "arn:aws:s3:::nomura-data-lake/│
│  │                 gold/esg-insights/*",            │
│  │     "Principal": "arn:aws:iam::222222222222:root"│
│  │     (Subscriber's AWS Account)                  │
│  │   }                                              │
│  ├─ Billing: Usage-based (requests + data scanned) │
│  └─ Lifecycle: Auto-renew on subscription           │
│                                                       │
│  Data Product: "Nomura Portfolio Analytics"        │
│  ├─ Share Type: Redshift Data Share                │
│  ├─ Access Pattern: JOIN across customer clusters  │
│  ├─ Subscriber Cluster: Can run queries without    │
│  │   moving data (shares Nomura's Redshift data)   │
│  └─ Latency: <100ms (metadata-only layer)          │
│                                                       │
└──────────────────────────────────────────────────────┘
         ↓ (Authenticated via IAM + SigV4)
```

#### Layer 3: Customer's Query Layer (Subscriber)

```plaintext
┌──────────────────────────────────────────────────────┐
│    Client AWS Account (e.g., Hedge Fund)             │
├──────────────────────────────────────────────────────┤
│                                                       │
│  Client's Redshift Cluster                          │
│  ├─ Schema: external_data_products                   │
│  ├─ External Table: nomura_esg_insights             │
│  │   (points to Nomura's S3 via Data Share)         │
│  └─ Query: SELECT AVG(esg_score) FROM              │
│              nomura_esg_insights WHERE ...          │
│                (instant result, sub-second)        │
│                                                       │
│  Client's Athena Query Engine                       │
│  ├─ Data Source: S3 (via IAM cross-account grant)  │
│  ├─ Cost: Client pays for scan + query compute    │
│  └─ Performance: Partition pruning on Nomura's     │
│     data automatically applied                     │
│                                                       │
│  Client's AWS Glue Data Catalog                     │
│  ├─ Discovers: Nomura's data schemas (read-only)  │
│  ├─ Lineage: Tracks which Nomura data is consumed |
│  └─ Integration: Automaps to client's own schema  │
│                                                       │
└──────────────────────────────────────────────────────┘
```

### 1.3 Implementation: Step-by-Step Deployment

#### Step 1: Enable Lake Formation & Data Catalog (Nomura Side)

```bash
# 1. Create Nomura's master data location in Lake Formation
aws lakeformation register-resource \
  --resource-arn arn:aws:s3:::nomura-data-lake \
  --use-service-linked-role

# 2. Create Glue Data Catalog tables
aws glue create-table \
  --database-name nomura_data_products \
  --table-input '...NAV_HISTORY table definition...'

# 3. Set Lake Formation permissions (by row at minimum)
aws lakeformation grant-permissions \
  --principal DataLakePrincipalIdentifier=arn:aws:iam::222222222222:root \
  --permissions SELECT \
  --resource '{"Table": {"DatabaseName": "nomura_data_products", "Name": "nav_history"}}'
```

#### Step 2: Create ADX Data Product

```bash
# 1. Create revision (snapshot of data + schema)
aws dataexchange create-revision \
  --data-set-id <data-set-id> \
  --comment "2024-02-18 ESG Insights with Q1 2024 data"

# 2. Create asset pointing to S3
aws dataexchange create-asset \
  --data-set-id <data-set-id> \
  --revision-id <revision-id> \
  --asset-type S3_SNAPSHOT \
  --asset '{"S3SnapshotAsset": {"Bucket": "nomura-data-lake", "Key": "gold/esg-insights/"}}'

# 3. Finalize revision (now available on ADX Marketplace)
aws dataexchange update-revision \
  --data-set-id <data-set-id> \
  --revision-id <revision-id> \
  --finalized
```

#### Step 3: Subscriber Onboarding (Automatic via Salesforce)

```java
// AWS Step Function (triggered by Salesforce contract signature)
@Service
public class ADXSubscriberOnboardingService {
  
  @Autowired private AmazonDataExchange adxClient;
  @Autowired private SalesforceService salesforceService;
  
  public void provisionClientADXAccess(String contractId) throws Exception {
    // 1. Fetch contract details from Salesforce
    Contract contract = salesforceService.getContractById(contractId);
    String clientAwsAccountId = contract.getCustomField("AWS_Account_ID__c");
    List<String> dataProductIds = contract.getCustomField("ADX_Data_Products__c");
    
    // 2. Grant IAM permissions via Data Exchange
    for (String productId : dataProductIds) {
      DataSet dataSet = adxClient.getDataSet(productId);
      
      // Create S3 Data Access grant (zero-copy)
      String grantArn = createS3DataAccessGrant(
        clientAwsAccountId,
        dataSet.getS3Bucket(),
        dataSet.getS3Prefix()
      );
      
      // 3. Log in ADX (audit trail)
      adxClient.putEvents(new PutEventsRequest()
        .withEventDetails(Map.of(
          "action", "SUBSCRIBER_GRANTED_ACCESS",
          "client_aws_account", clientAwsAccountId,
          "data_set_id", productId,
          "timestamp", LocalDateTime.now().toString()
        ))
      );
    }
    
    // 4. Send welcome email to client with query examples
    String athenaQueryTemplate = generateAthenaQueryTemplate(dataProductIds);
    sendWelcomeEmail(contract.getClientEmail(), athenaQueryTemplate);
  }
  
  private String createS3DataAccessGrant(String clientAccount, String bucket, String prefix) {
    String policyDocument = String.format("""
      {
        "Version": "2012-10-17",
        "Statement": [
          {
            "Effect": "Allow",
            "Principal": {
              "AWS": "arn:aws:iam::%s:root"
            },
            "Action": ["s3:GetObject", "s3:ListBucket"],
            "Resource": [
              "arn:aws:s3:::%s",
              "arn:aws:s3:::%s/%s*"
            ]
          }
        ]
      }
      """, clientAccount, bucket, bucket, prefix);
    
    // Apply bucket policy
    return applyS3BucketPolicy(bucket, policyDocument);
  }
}
```

---

## Section 2: The Unified API Pattern (AWS Data Exchange for APIs)

### 2.1 Architecture: ADX + API Gateway + IAM Authorization

**Traditional API Model** (API Key-Based):
```
Client → API Request + API Key → Lambda → Nomura DB → Response
Issues:
  - API keys are static credentials (rotation required)
  - No fine-grained resource control (all or nothing)
  - Auditing is application-level (not infrastructure-level)
```

**Levin's Unified API Pattern** (IAM-Native):
```
Client (using AWS SDK) → Signs request with SigV4 (AWS IAM credential)
  ↓
API Gateway (with ADX Authorizer)
  ├─ Validates SigV4 signature (verifies client AWS account)
  ├─ Checks ADX subscription status (client has paid/permission?)
  └─ Enforces least-privilege (only allowed data sets)
  ↓
Lambda (executes with scoped permissions)
  ├─ Accesses only data matching client's ADX grant
  └─ Returns filtered response
  ↓
CloudWatch Logs (infrastructure-level audit)
  ├─ Logs every request (who, what, when, where)
  ├─ Fed to compliance team automatically
  └─ Immutable (stored in S3 via CloudWatch Log Sink)
```

### 2.2 Implementation: SigV4 + ADX Authorizer

```yaml
# API Gateway Configuration (OpenAPI/Swagger)
openapi: 3.0.0
info:
  title: Nomura Asset Management APIs
  version: 2024-02-18
servers:
  - url: https://api.nomura-data.aws/v2
    description: Production API
security:
  - aws_sigv4: []

paths:
  /funds/{fundId}/nav:
    get:
      summary: Get current NAV for a fund
      operationId: getNAV
      parameters:
        - name: fundId
          in: path
          required: true
          schema:
            type: string
      responses:
        '200':
          description: NAV data
          content:
            application/json:
              schema:
                type: object
                properties:
                  fundId:
                    type: string
                  nav:
                    type: number
                  timestamp:
                    type: string
        '403':
          description: Client not authorized for this data product
      x-amazon-apigateway-integration:
        httpMethod: POST
        type: aws_proxy
        uri: arn:aws:apigateway:us-east-1:lambda:path/2015-03-31/functions/arn:aws:lambda:us-east-1:111111111111:function:GetNAVFunction/invocations
        credentials: arn:aws:iam::111111111111:role/APIGatewayLambdaRole
        responses:
          default:
            statusCode: '200'

components:
  securitySchemes:
    aws_sigv4:
      type: apiKey
      name: Authorization
      in: header
      x-amazon-apigateway-authtype: awsSigv4
```

```java
// Authorizer Lambda (validates ADX subscription + SigV4)
@Component
public class ADXAPIAuthorizer {
  
  @Autowired private AmazonDataExchange adxClient;
  @Autowired private SalesforceService salesforceService;
  
  public AuthorizerResponse authorize(JsonWebTokenContext tokenContext) {
    try {
      // 1. Extract AWS account from request context
      String subscriberAwsAccount = tokenContext.getAccountId();
      
      // 2. Verify subscription status in ADX
      SubscriptionStatus subscription = adxClient.getSubscriptionStatus(subscriberAwsAccount);
      if (!subscription.isActive()) {
        return denyAccess("Subscription inactive or expired");
      }
      
      // 3. Extract resource requested
      String fundId = tokenContext.getResourceId();
      
      // 4. Check entitlement: is this subscriber allowed to access this fund's data?
      boolean hasAccess = checkFundAccess(subscriberAwsAccount, fundId);
      if (!hasAccess) {
        return denyAccess("Not authorized for Fund: " + fundId);
      }
      
      // 5. Success: return policy allowing Lambda invoke + context data
      return AuthorizerResponse.builder()
        .principalId(subscriberAwsAccount)
        .effect("Allow")
        .action("execute-api:Invoke")
        .resource(tokenContext.getMethodArn())
        .context("subscriberAwsAccount", subscriberAwsAccount)
        .context("fundId", fundId)
        .context("accessLevel", "READ_ONLY")
        .build();
      
    } catch (Exception ex) {
      logger.error("Authorization failed", ex);
      return denyAccess("Authorization service error");
    }
  }
  
  private AuthorizerResponse denyAccess(String reason) {
    return AuthorizerResponse.builder()
      .principalId("unknown")
      .effect("Deny")
      .action("execute-api:Invoke")
      .context("reason", reason)
      .build();
  }
  
  private boolean checkFundAccess(String subscriberAwsAccount, String fundId) {
    // Check ADX data grant via Salesforce contract
    Contract contract = salesforceService.findContractByAwsAccount(subscriberAwsAccount);
    return contract != null && contract.hasAccessToFund(fundId);
  }
}

// Lambda Handler (executes only after authorization)
public class GetNAVHandler implements RequestHandler<APIGatewayProxyRequestEvent, APIGatewayProxyResponseEvent> {
  
  @Override
  public APIGatewayProxyResponseEvent handleRequest(
    APIGatewayProxyRequestEvent input,
    Context context
  ) {
    Map<String, String> requestContext = (Map<String, String>) input.getRequestContext().get("authorizer");
    String subscriberAwsAccount = requestContext.get("subscriberAwsAccount");
    String fundId = input.getPathParameters().get("fundId");
    
    logger.info("Fetching NAV for Fund {} for subscriber {}", fundId, subscriberAwsAccount);
    
    try {
      // Query Athena (via S3 + Glue Data Catalog) with subscriber's access level
      NAVData navData = athenaClient.queryNAV(fundId);
      
      // Audit log (will be picked up by CloudWatch Log Sink → S3 → Athena → Compliance)
      logger.info("NAV_QUERY_SUCCESS fundId={} subscriber={} nav={}", 
        fundId, subscriberAwsAccount, navData.value);
      
      return new APIGatewayProxyResponseEvent()
        .withStatusCode(200)
        .withBody(navData.toJson());
      
    } catch (Exception ex) {
      logger.error("NAV_QUERY_FAILED fundId={} subscriber={} error={}", 
        fundId, subscriberAwsAccount, ex.getMessage());
      return new APIGatewayProxyResponseEvent()
        .withStatusCode(500)
        .withBody("{\"error\": \"Internal server error\"}");
    }
  }
}
```

---

## Section 3: The Event-Driven Lifecycle (CloudWatch Events + EventBridge)

### 3.1 Pattern: Push-Signal vs. Pull-Poll

**Traditional Model** (Polling):
```
Client Daemon (Every 1 hour):
  → Checks Nomura: "Any new data?"
  → Nomura responds: "Yes/No"
  → If yes: Client downloads (network I/O)
  → If no: Wasted request

Issues:
  - Staleness: Data could be 59 minutes old
  - Inefficiency: 50% of polls return no data
  - Network cost: Redundant request/response cycles
```

**Levin's Event-Driven** (Push-Signal):
```
Nomura (Data Team) → Updates dataset in S3
  ↓ (Triggers)
ADX Revision Event (CloudWatch Event)
  ├─ Event: "data_product_revision_published"
  ├─ Data: {productId, revisionId, timestamp, dataSize}
  └─ Target: Client's SNS topic / EventBridge rule
  ↓
Client's EventBridge Rule (auto-triggered)
  ├─ Condition: "IF data_product = 'ESG_INSIGHTS'"
  └─ Action: Invoke Lambda function (client-side)
  ↓
Client's Lambda (executes immediately)
  ├─ Refreshes local cache: Query Nomura's Athena
  ├─ Updates dashboard: Push update to portfolio managers
  └─ Logs: "New ESG data ingested at 2024-02-18 14:32:01"

Benefit: Real-time data freshness with zero polling overhead
```

### 3.2 Implementation: ADX Revision Events → EventBridge

```java
// Nomura Service: Publishes data update
@Service
public class DataProductPublisher {
  
  @Autowired private AmazonDataExchange adxClient;
  @Autowired private AmazonEventBridge eventBridgeClient;
  
  public void publishDatasetRevision(String productId, String revisionId) {
    try {
      // 1. Create new revision in ADX
      DataSetRevision revision = adxClient.createRevision(productId);
      
      // 2. Add assets (point to latest S3 gold data)
      String s3Key = "gold/esg-insights/2024-02-18/";
      revision.addAsset("esg_insights_latest", s3Key);
      
      // 3. Finalize revision (triggers ADX event)
      RevisionFinalizationResponse response = adxClient.finalizeRevision(revision);
      
      // 4. Emit custom event to EventBridge (for subscribers to listen)
      PutEventsRequest eventRequest = new PutEventsRequest()
        .withEntries(new PutEventsRequestEntry()
          .withSource("nomura.data-platform")
          .withDetailType("DataSetRevisionPublished")
          .withDetail(new JSONObject()
            .put("productId", productId)
            .put("revisionId", response.getRevisionId())
            .put("timestamp", LocalDateTime.now().toString())
            .put("subscribers", countSubscribers(productId))
            .put("dataSize", calculateDataSize(s3Key))
            .toString()
          )
        );
      
      eventBridgeClient.putEvents(eventRequest);
      logger.info("Revision published: productId={} revisionId={}", 
        productId, response.getRevisionId());
      
    } catch (Exception ex) {
      logger.error("Failed to publish revision", ex);
      throw new RuntimeException(ex);
    }
  }
}
```

```yaml
# Client Side: EventBridge Rule (in Client's AWS Account)
Name: nomura-esg-insights-auto-refresh
EventPattern: |
  {
    "source": ["nomura.data-platform"],
    "detail-type": ["DataSetRevisionPublished"],
    "detail": {
      "productId": ["esg-insights-v2"]
    }
  }
Targets:
  - Arn: arn:aws:lambda:us-east-1:222222222222:function:RefreshESGInsightsCache
    RoleArn: arn:aws:iam::222222222222:role/EventBridgeInvokeRole
    RetryPolicy:
      MaximumRetryAttempts: 2
      MaximumEventAge: 3600
    DeadLetterConfig:
      Arn: arn:aws:sqs:us-east-1:222222222222:nomura-dlq
```

```java
// Client-Side Lambda: Triggered on data update
public class RefreshESGInsightsCacheHandler implements RequestHandler<EventBridgeEvent<String, DataSetRevisionPublishedDetail>, String> {
  
  @Override
  public String handleRequest(EventBridgeEvent<String, DataSetRevisionPublishedDetail> event, Context context) {
    
    DataSetRevisionPublishedDetail detail = event.getDetail();
    logger.info("Nomura published new ESG data: revision={}", detail.getRevisionId());
    
    try {
      // 1. Query Nomura's Athena (via cross-account S3 grant)
      String query = String.format("""
        SELECT fund_id, esg_score, updated_at 
        FROM external_nomura_data.esg_insights
        WHERE updated_at > CAST('%s' AS TIMESTAMP)
      """, detail.getTimestamp());
      
      AthenaQueryResult results = athenaClient.executeQuery(query);
      
      // 2. Cache results locally
      dynamoDbClient.batchWriteItem("EsgInsightsCache", results.toItems());
      
      // 3. Trigger dashboard refresh
      apiGatewayClient.publishMessage("websocket-connection-id", 
        Map.of("event", "ESG_DATA_UPDATED"));
      
      // 4. Audit log
      logger.info("ESG cache refreshed: records={} revisionId={}", 
        results.getRowCount(), detail.getRevisionId());
      
      return "SUCCESS";
      
    } catch (Exception ex) {
      logger.error("Cache refresh failed", ex);
      // EventBridge will retry (see DeadLetterConfig in rule)
      throw new RuntimeException(ex);
    }
  }
}
```

---

## Section 4: Strategic Security Patterns

### 4.1 "Least Privilege Entitlements"

**Anti-Pattern** (Shared Credentials):
```
Nomura issues credentials to client:
  "Username: client_123"
  "Password: SomePassword123"
  "Can access: ALL data"

Problems:
  - Credentials stored in plaintext in client code
  - Impossible to revoke individual data access (must change password)
  - No audit trail at infrastructure level
  - Compliance nightmare (SOC2 requires no shared secrets)
```

**Levin's Approach** (IAM Policy-Based):
```
Instead of credentials, Nomura grants IAM permissions:
  
Nomura AWS Account (111111111111):
  └─ S3 Bucket: nomura-data-lake (with data)

Client AWS Account (222222222222):
  └─ IAM Role: nomura-data-subscriber
     ├─ Trust: Can be assumed by client's applications
     ├─ Permissions: Specific to ESG Insights dataset only
     │   {
     │     "Effect": "Allow",
     │     "Action": ["s3:GetObject", "s3:ListBucket"],
     │     "Resource": "arn:aws:s3:::nomura-data-lake/gold/esg-insights/*"
     │   }
     └─ Session Duration: 1 hour (short-lived, auto-expires)

Client's Lambda (running under this role) queries data:
  + Credentials: Temporary STS token (expires automatically)
  + Scope: Only S3 GetObject (no DeleteObject permissions)
  + Audit: CloudTrail logs every request with role session ID

Result: Zero standing credentials; infrastructure-level audit trail 🔒
```

---

## Section 5: The "Levin Edition" Blueprint - Complete Reference Architecture

### 5.1 Logical Architecture Table

| Layer | Component | AWS Service | Purpose | Levin Pattern |
|-------|-----------|-------------|---------|---------------|
| **Ingestion** | Medallion Lakehouse | S3 + AWS Glue + Lake Formation | Organize data into Bronze/Silver/Gold layers | Data governance foundation |
| **Entitlement** | Identity Mesh | AWS IAM + ADX Data Grants | Fine-grained access control | Least-privilege security |
| **Distribution (Sync)** | Zero-Copy Share | AWS Data Exchange for S3 + Redshift | Metadata-only exchange (no copy) | Petabyte-scale efficiency |
| **Distribution (Async)** | Managed API | AWS Data Exchange for APIs + API Gateway + Authorizer | SigV4-authenticated APIs with ADX validation | Unified, auditable API |
| **Change Capture** | Event-Driven Lifecycle | ADX Revision Events via EventBridge | Push-signal on data update | Real-time value delivery |
| **Audit & Compliance** | Immutable Logs | CloudWatch Logs → S3 Sink | Infrastructure-level audit trail | SOC2-ready compliance |

### 5.2 AWS Services Deployment Checklist

- [ ] **S3**: Create `nomura-data-lake` bucket (versioning + encryption enabled)
- [ ] **AWS Glue**: Create Data Catalog tables for Gold layer (nav_history, esg_insights, performance_metrics)
- [ ] **Lake Formation**: Register S3 location; set up data grants
- [ ] **Redshift**: Create cluster; configure Redshift Data Sharing for clients
- [ ] **AWS Data Exchange**: Create data products (ESG Insights, Portfolio Analytics, Benchmarks)
- [ ] **API Gateway**: Deploy REST API with custom authorizer Lambda
- [ ] **EventBridge**: Set up rules to emit DataSetRevisionPublished events
- [ ] **CloudWatch**: Configure log group for API Gateway + Lambda logs
- [ ] **CloudTrail**: Enable for S3 + Redshift (audit trail)

---

## Section 6: The "Frictionless Ingestion" UX - Client Onboarding Path

### 6.1 5-Minute Onboarding Journey (From Contract → Live Queries)

**Minute 0: Contract Signed**
- Nomura Executive Director closes deal in Salesforce
- Contract record updated: `ADX_Data_Products__c = ["esg-insights-v2", "portfolio-analytics"]`
- Salesforce Workflow trigger fires

**Minute 1: Entitlement Automation**
- AWS Step Function invoked automatically
- Validates client AWS account ID from contract
- Grants S3 + Redshift Data Share permissions

**Minute 2: Client Notification**
- Welcome email sent to client's data engineering lead
- Includes: pre-built Athena query template, Redshift connection string
- Sample query: `SELECT * FROM nomura_esg_insights LIMIT 10`

**Minute 3: Client Tests Connectivity**
- Client runs Athena query in their own AWS console
- Results return in <1 second (metadata pointer to Nomura's S3)
- Sees live ESG data without any ETL setup ✅

**Minute 4: Integration**
- Client's BI tool (Tableau/PowerBI) now connects to Redshift Data Share
- Builds first dashboard using Nomura data

**Minute 5: Success**
- Client's portfolio managers viewing ESG insights
- Real-time data (Nomura updates every 1hr, client sees it immediately)
- No IT escalations, no data engineering required

### 6.2 Sample Welcome Email (Sent Automatically)

```
Subject: [ACTION REQUIRED] Your Nomura ESG Insights Access is Ready! 🚀

Hi [Client Data Lead],

Congratulations! Your access to Nomura's ESG Insights data product is now active.

📊 Quick Start (5 minutes):

1. Verify Access:
   Open AWS Console → Athena → Copy/paste this query:
   
   SELECT 
     fund_id, 
     esg_score, 
     environmental_rating, 
     updated_at 
   FROM external_nomura_data.esg_insights 
   LIMIT 10;
   
2. Connect BI Tool:
   Add data source: 
   - Type: Amazon Athena OR Redshift (your choice)
   - Region: us-east-1
   - Database: nomura_external_data
   - Table: esg_insights
   
3. Build Dashboard:
   Recommended fields:
   - X-axis: Fund ID
   - Y-axis: ESG Score
   - Filter: Date range (leave blank for latest)

❓ Questions?
- Technical: Email [support@nomura-data.aws](mailto:support@nomura-data.aws)
- Billing: [adx-billing@nomura.com](mailto:adx-billing@nomura.com)

Your data is live now! 🎉

Best,
Nomura Data Products Team
```

---

## Section 7: Strategic Interview Talking Points (Levin-Certified)

### 7.1 Opening Statement (90 seconds)

> "I architected a **Zero-ETL, Petabyte-Scale Data Distribution Mesh** for Nomura's institutional clients. Rather than copying terabytes of data across networks—which traditionally costs millions in egress fees—I implemented a **Metadata-Only Exchange** pattern using AWS Data Exchange + S3 + Redshift. 
>
> The result: clients onboard in **5 minutes instead of 5 weeks**; Nomura eliminates **$20M+ in annual egress costs**; and we maintain **zero-trust security** by using IAM Policy-based grants instead of shared credentials.
>
> The architecture follows **three Levin-era AWS patterns**: Zero-ETL (Metadata-Only), Unified APIs (SigV4-authenticated), and Event-Driven Lifecycle (push-signal vs. pull-poll). This is how AWS handles petabyte-scale data distribution at customers like Amazon Advertising and Stripe."

### 7.2 Deep-Dive Questions & Answers

**Q1: "Walk me through incident response if an ADX revision fails to publish."**

A: "In my design, the ADX revision lifecycle is event-driven with dead-letter queues (DLQs). Here's the flow:

1. **Failure Detection** (automated): Nomura's data pipeline attempts to publish revision → ADX API returns 500 → EventBridge automatically routes to DLQ
2. **Notification** (immediate): SNS alert sent to data ops team + Slack channel
3. **Rollback** (optional): If data quality issue, mark revision as 'blocked' in Lake Formation
4. **Retry Logic** (configurable): Step Functions retries with exponential backoff (30s, 1min, 5min)
5. **Client Communication** (transparent): If revision blocked for >2 hours, email blast to subscribers explaining delay

The key principle is **fail loudly and fast**, with infrastructure-level observability. Clients remain unaffected because they query against previous revision (immutable snapshot)."

**Q2: "What's your approach to cost optimization at petabyte scale?"**

A: "I follow Levin's **shared responsibility cost model**:

1. **Nomura pays for**: S3 storage (master dataset) + cold/warm compute (Redshift cluster baseline)
   - Cost: ~$50K/month (fixed, no variable scaling)
   
2. **Client pays for**: Query compute (Athena scans) + their own Redshift replica queries
   - They only pay for what they use
   - Incentivizes efficient queries (partition pruning, column selection)

3. **Petabyte Scale**: At 10PB of Nomura data, egress model would cost $200M+ (10 × 10^15 bytes × $0.02/GB). Zero-ETL pattern: $0 egress cost (only metadata overhead ~100KB/query).

4. **Governance**: Lake Formation row-level security + S3 bucket policies ensure clients only access entitled datasets (cost control + security)"

**Q3: "How do you ensure zero-trust security in this architecture?"**

A: "Three layers:

1. **Entry Layer** (API Gateway + ADX Authorizer):
   - Client must provide SigV4 signature (AWS SDK handles automatically)
   - No API keys passed around → no credentials in code/version control
   
2. **Identity Layer** (IAM Roles):
   - Client's Lambda assumes cross-account role
   - Permissions: `s3:GetObject` on specific S3 prefix only (not DeleteObject, PutObject)
   - Session duration: 1 hour (auto-expires)
   
3. **Data Layer** (Lake Formation + Row-Level Security):
   - When client queries 'fund_id = 123', Lake Formation intercepts
   - Checks: Does this IAM role have permission to view this fund?
   - If no: Query returns 0 rows (client doesn't know the data exists)

Result: No shared credentials, minimal standing privileges, infrastructure-level audit."

---

## Section 8: Frequently Asked Questions (FAQs)

### FAQ 1: Won't zero-copy increase latency compared to local copies?

**Answer**: No. In fact, it's *faster*. Here's why:

- Traditional: Copy 1TB → Write to client's storage → Client queries → 3sec latency (network I/O)
- Zero-Copy: Client queries metadata pointer → Nomura's Athena returns result → 200ms latency (only query time, no copy)

The only overhead is metadata resolution (~10-50ms), tiny compared to network copy. Plus, Nomura can optimize queries server-side (partition pruning, query rewrite).

### FAQ 2: What if a client's AWS account gets compromised?

**Answer**: Damage is *contained* to that client's access level via IAM.

- Attacker can: Read Nomura ESG Insights data (if that client has access)
- Attacker *cannot*: Read other clients' data (IAM policy limits to this client only)
- Attacker *cannot*: Delete data (no s3:DeleteObject permission)

Compare to API key model: If key is stolen, attacker can access ALL data → massive breach.

### FAQ 3: How do we handle data compliance (GDPR, CCPA)?

**Answer**: Lake Formation + ADX integration enables compliance:

1. **Data Lineage**: Every query logged with IAM session → CloudTrail → S3 Sink → Athena (for audit)
2. **Right to Deletion**: Client requests Nomura delete their data → Nomura removes from ADX revision → Auto-revokes past revisions
3. **Data Localization**: S3 bucket policy restricts data to specific AWS region (e.g., eu-west-1 for GDPR)

All audit-ready for compliance teams.

---

## Deployment Checklist & Success Criteria

### Phase 1: Foundation (Month 1)
- [ ] Lake Formation + S3 Gold Layer setup
- [ ] Glue Data Catalog tables created
- [ ] First ADX data product published
- [ ] IAM roles + policies defined
- **Success**: 1 pilot client onboarded; queries run in <100ms

### Phase 2: API Layer (Month 2)
- [ ] API Gateway deployed
- [ ] Custom authorizer Lambda (ADX validation)
- [ ] EventBridge rules configured
- [ ] SigV4 authentication working
- **Success**: Client SDK calls API; queries authorized via IAM

### Phase 3: Scale (Month 3+)
- [ ] Multi-region failover (Redshift + S3)
- [ ] Cost optimization (reserved capacity)
- [ ] Compliance audit-ready (CloudTrail logs in Athena)
- **Success**: 50+ clients; $5M ARR via ADX; zero security incidents

---

## References & Further Reading

- Mihir Popat Medium: [Unlocking the Power of Data with AWS Data Exchange](https://mihirpopat.medium.com/)
- AWS Data Exchange: [Official Documentation](https://docs.aws.amazon.com/data-exchange/)
- Redshift Data Sharing: [Cross-Account Sharing Guide](https://docs.aws.amazon.com/redshift/latest/mgmt/datashare)
- Lake Formation: [Best Practices](https://docs.aws.amazon.com/lake-formation/latest/userguide/best-practices.html)
- AWS IAM: [Least Privilege Access](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)

---

**Last Updated**: February 18, 2026  
**Status**: Enterprise Architect Review - Ready for Implementation  
**Next Steps**: Evaluate with principal engineers + conduct iterative feedback cycles

