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

## Section 9: Multi-Region Disaster Recovery & Failover

### 9.1 Strategy: Active-Active Across AWS Regions

**Architecture Pattern**: Nomura maintains active ADX data products in **us-east-1** (primary) and **eu-west-1** (secondary) with automatic failover.

```plaintext
Primary Region (us-east-1)
├─ S3 Gold Lake: nomura-data-lake-useast1
│  └─ Continuous replication via S3 CRR → eu-west-1
├─ Redshift Cluster: RA3, 2 nodes
│  └─ Read replica auto-syncs across region
└─ ADX Data Products: 5 products (ESG, Portfolio, Risk, Macro, Benchmarks)
   └─ Revisions auto-replicated to secondary region

                ↓ (S3 CRR + Multi-Region Redshift Replica)

Secondary Region (eu-west-1)
├─ S3 Gold Lake: nomura-data-lake-euwest1 (copy of primary)
├─ Redshift Cluster: RA3, 2 nodes (read-only, becomes read-write on failover)
└─ ADX Data Products: Same 5 products (available to eu-west-1 clients)

Client Failover (Automatic):
├─ Client in eu-west-1 → First queries eu-west-1 data
├─ If eu-west-1 fails → Route to us-east-1 via multi-region endpoint
└─ Cost: Minimal (no data egress charges; only query charges)
```

### 9.2 Implementation: S3 CRR + Redshift Failover

#### Step 1: S3 Cross-Region Replication (CRR)

```bash
# Enable versioning on primary bucket
aws s3api put-bucket-versioning \
  --bucket nomura-data-lake-useast1 \
  --versioning-configuration Status=Enabled \
  --region us-east-1

# Create replication role (IAM)
cat > replication-role-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetReplicationConfiguration",
        "s3:ListBucket"
      ],
      "Resource": "arn:aws:s3:::nomura-data-lake-useast1"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObjectVersionForReplication",
        "s3:GetObjectVersionAcl"
      ],
      "Resource": "arn:aws:s3:::nomura-data-lake-useast1/*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:ReplicateObject",
        "s3:ReplicateDelete"
      ],
      "Resource": "arn:aws:s3:::nomura-data-lake-euwest1/*"
    }
  ]
}
EOF

aws iam create-role \
  --role-name S3CRRRole \
  --assume-role-policy-document file://trust-policy.json

aws iam put-role-policy \
  --role-name S3CRRRole \
  --policy-name S3CRRPolicy \
  --policy-document file://replication-role-policy.json

# Enable replication on primary bucket
cat > replication-config.json << 'EOF'
{
  "Role": "arn:aws:iam::111111111111:role/S3CRRRole",
  "Rules": [
    {
      "Status": "Enabled",
      "Priority": 1,
      "DeleteMarkerReplication": {
        "Status": "Enabled"
      },
      "Filter": {
        "Prefix": "gold/"
      },
      "Destination": {
        "Bucket": "arn:aws:s3:::nomura-data-lake-euwest1",
        "ReplicationTime": {
          "Status": "Enabled",
          "Time": {
            "Minutes": 15
          }
        },
        "Metrics": {
          "Status": "Enabled",
          "EventThreshold": {
            "Minutes": 15
          }
        },
        "StorageClass": "STANDARD_IA"
      }
    }
  ]
}
EOF

aws s3api put-bucket-replication \
  --bucket nomura-data-lake-useast1 \
  --replication-configuration file://replication-config.json \
  --region us-east-1
```

**RTO/RPO for S3**:
- **RPO (Recovery Point Objective)**: 15 minutes (S3 CRR guarantee)
- **RTO (Recovery Time Objective)**: <1 minute (failover is automatic)

#### Step 2: Redshift Multi-Region Read Replica

```bash
# Create read replica in secondary region
aws redshift create-cluster-snapshot \
  --snapshot-identifier nomura-snapshot-$(date +%s) \
  --cluster-identifier nomura-redshift-primary \
  --region us-east-1

# Copy snapshot to secondary region
aws redshift copy-cluster-snapshot \
  --source-snapshot-identifier nomura-snapshot-123456 \
  --source-region us-east-1 \
  --target-snapshot-identifier nomura-snapshot-123456-copy \
  --region eu-west-1

# Restore snapshot to new cluster in secondary region
aws redshift restore-from-cluster-snapshot \
  --cluster-identifier nomura-redshift-secondary \
  --snapshot-identifier nomura-snapshot-123456-copy \
  --region eu-west-1

# Enable cross-region read replica synchronization
aws redshift modify-cluster \
  --cluster-identifier nomura-redshift-primary \
  --availability-zone-relocation \
  --region us-east-1
```

**RTO/RPO for Redshift**:
- **RPO**: 1 hour (sync frequency)
- **RTO**: 15 minutes (manual promotion of read replica to primary)

#### Step 3: ADX Data Product Multi-Region Replication

```java
@Service
public class MultiRegionADXPublisher {
  
  @Autowired private AmazonDataExchange adxClientUsEast;
  @Autowired private AmazonDataExchange adxClientEuWest;
  
  public void publishDataProductMultiRegion(String productId, String revisionId) {
    try {
      // 1. Publish to primary region (us-east-1)
      String primaryRevisionId = publishToRegion(
        adxClientUsEast, 
        productId, 
        "us-east-1", 
        "nomura-data-lake-useast1"
      );
      logger.info("Primary region revision: {}", primaryRevisionId);
      
      // 2. Wait for S3 CRR to replicate data (RPO: 15 min max)
      Thread.sleep(Duration.ofMinutes(1).toMillis()); // Give CRR time
      
      // 3. Publish same product to secondary region (eu-west-1)
      String secondaryRevisionId = publishToRegion(
        adxClientEuWest,
        productId,
        "eu-west-1",
        "nomura-data-lake-euwest1"
      );
      logger.info("Secondary region revision: {}", secondaryRevisionId);
      
      // 4. Log cross-region sync event (for disaster recovery testing)
      logMultiRegionSync(productId, primaryRevisionId, secondaryRevisionId);
      
    } catch (Exception ex) {
      logger.error("Multi-region publish failed", ex);
      // Alert: Manual intervention required
      alertOnCall("ADX multi-region sync failure: " + ex.getMessage());
    }
  }
  
  private String publishToRegion(
    AmazonDataExchange adxClient,
    String productId,
    String region,
    String bucketName
  ) {
    // Create revision pointing to region-specific S3 bucket
    DataSetRevision revision = adxClient.createRevision(productId);
    revision.addAsset("esg_insights_" + region, bucketName + "/gold/esg-insights/");
    
    RevisionFinalizationResponse response = adxClient.finalizeRevision(revision);
    return response.getRevisionId();
  }
}
```

### 9.3 Failover Procedures

#### Scenario 1: us-east-1 Region Outage (Automatic)

```
Monitoring Alert: "us-east-1 s3.amazonaws.com unreachable"
  ↓
API Gateway Health Check Fails (tries us-east-1 Redshift)
  ↓
Auto-trigger Health Check for eu-west-1 endpoint
  ↓
If eu-west-1 healthy:
  ├─ Route all queries to eu-west-1 (within 30 seconds)
  └─ ADX notifies subscribers: "Data now served from eu-west-1"
  ↓
Clients experience:
  - Query latency increase (if outside eu-west-1 region)
  - Data freshness: 1 hour delay (last Redshift sync)
  - Subscription still active (no re-authentication)
```

**Estimated Downtime**: 30-60 seconds (health check + DNS failover)

#### Scenario 2: Data Corruption in Primary (Manual)

```bash
# 1. Detect corruption (automated validation)
aws s3api head-object \
  --bucket nomura-data-lake-useast1 \
  --key gold/esg-insights/latest/data.parquet \
  --metadata-directive COPY

# 2. Verify secondary is clean
aws s3api head-object \
  --bucket nomura-data-lake-euwest1 \
  --key gold/esg-insights/latest/data.parquet

# 3. Disable replication (prevent corruption from spreading)
aws s3api put-bucket-replication \
  --bucket nomura-data-lake-useast1 \
  --replication-configuration '{"Role": "...", "Rules": []}' \
  --region us-east-1

# 4. Promote secondary to primary
aws route53 change-resource-record-sets \
  --hosted-zone-id Z1234567890ABC \
  --change-batch file://failover-dns.json

# 5. Enable reverse replication (eu-west-1 → us-east-1 for recovery)
# (manual: after corruption source identified and fixed)

# 6. Monitor for 24 hours, then restore primary
```

---

## Section 10: Unit Testing Strategy & Examples

### 10.1 Testing Philosophy: Constructor Injection + Mockito

**Refactored Service** (Constructor Injection):

```java
@Service
public class ADXSubscriberOnboardingService {
  
  private final AmazonDataExchange adxClient;
  private final SalesforceService salesforceService;
  private final AmazonEventBridge eventBridgeClient;
  
  // Constructor injection (modern Spring pattern)
  public ADXSubscriberOnboardingService(
    AmazonDataExchange adxClient,
    SalesforceService salesforceService,
    AmazonEventBridge eventBridgeClient
  ) {
    this.adxClient = Objects.requireNonNull(adxClient);
    this.salesforceService = Objects.requireNonNull(salesforceService);
    this.eventBridgeClient = Objects.requireNonNull(eventBridgeClient);
  }
  
  public void provisionClientADXAccess(String contractId) throws Exception {
    // Same implementation as before
  }
}
```

### 10.2 Unit Tests: Happy Path & Error Cases

```java
@ExtendWith(MockitoExtension.class)
class ADXSubscriberOnboardingServiceTest {
  
  @InjectMocks
  private ADXSubscriberOnboardingService service;
  
  @Mock
  private AmazonDataExchange mockAdxClient;
  
  @Mock
  private SalesforceService mockSalesforceService;
  
  @Mock
  private AmazonEventBridge mockEventBridgeClient;
  
  private Contract testContract;
  
  @BeforeEach
  void setUp() {
    testContract = Contract.builder()
      .id("CTR-2024-001")
      .clientName("Hedge Fund Partners")
      .awsAccountId("222222222222")
      .dataProducts(List.of("esg-insights-v2", "portfolio-analytics"))
      .clientEmail("data@hedgefund.com")
      .build();
  }
  
  // Test 1: Happy Path - Successful Provisioning
  @Test
  void testProvisionClientADXAccess_Success() throws Exception {
    // Arrange
    when(mockSalesforceService.getContractById("CTR-2024-001"))
      .thenReturn(testContract);
    
    when(mockAdxClient.putEvents(any(PutEventsRequest.class)))
      .thenReturn(new PutEventsResult());
    
    // Act
    service.provisionClientADXAccess("CTR-2024-001");
    
    // Assert: Verify create S3 data access grant called twice (2 data products)
    verify(mockAdxClient, times(2)).putEvents(any(PutEventsRequest.class));
    verify(mockSalesforceService).getContractById("CTR-2024-001");
  }
  
  // Test 2: Contract Not Found
  @Test
  void testProvisionClientADXAccess_ContractNotFound() {
    // Arrange
    when(mockSalesforceService.getContractById("CTR-INVALID"))
      .thenThrow(new ContractNotFoundException("Contract not found"));
    
    // Act & Assert
    assertThrows(ContractNotFoundException.class, () -> {
      service.provisionClientADXAccess("CTR-INVALID");
    });
  }
  
  // Test 3: ADX API Failure (Throttling)
  @Test
  void testProvisionClientADXAccess_ADXThrottling() {
    // Arrange
    when(mockSalesforceService.getContractById("CTR-2024-001"))
      .thenReturn(testContract);
    
    when(mockAdxClient.putEvents(any(PutEventsRequest.class)))
      .thenThrow(new ThrottlingException(
        "Request rate exceeded. Please retry after 30 seconds."
      ));
    
    // Act & Assert: Expect exception (service should retry)
    assertThrows(ThrottlingException.class, () -> {
      service.provisionClientADXAccess("CTR-2024-001");
    });
    
    // Verify we attempted to put events
    verify(mockAdxClient, atLeastOnce()).putEvents(any(PutEventsRequest.class));
  }
  
  // Test 4: Empty Data Products List (Edge Case)
  @Test
  void testProvisionClientADXAccess_NoDataProducts() throws Exception {
    // Arrange
    Contract emptyProductsContract = testContract.toBuilder()
      .dataProducts(Collections.emptyList())
      .build();
    
    when(mockSalesforceService.getContractById("CTR-2024-001"))
      .thenReturn(emptyProductsContract);
    
    // Act
    service.provisionClientADXAccess("CTR-2024-001");
    
    // Assert: Should not call ADX for any products
    verify(mockAdxClient, never()).putEvents(any());
  }
}
```

### 10.3 Lambda Authorization Handler Tests

```java
@ExtendWith(MockitoExtension.class)
class ADXAPIAuthorizerTest {
  
  @InjectMocks
  private ADXAPIAuthorizer authorizer;
  
  @Mock
  private AmazonDataExchange mockAdxClient;
  
  @Mock
  private SalesforceService mockSalesforceService;
  
  private JsonWebTokenContext mockTokenContext;
  
  @BeforeEach
  void setUp() {
    mockTokenContext = JsonWebTokenContext.builder()
      .accountId("222222222222")
      .resourceId("fund-123")
      .methodArn("arn:aws:execute-api:us-east-1:111111111111:abc123/prod/GET/funds/fund-123/nav")
      .build();
  }
  
  @Test
  void testAuthorize_ActiveSubscriber_ReturnsAllow() {
    // Arrange
    SubscriptionStatus activeSubscription = SubscriptionStatus.builder()
      .isActive(true)
      .expiresAt(LocalDateTime.now().plusMonths(12))
      .build();
    
    when(mockAdxClient.getSubscriptionStatus("222222222222"))
      .thenReturn(activeSubscription);
    
    Contract contract = Contract.builder()
      .authorizedFunds(List.of("fund-123", "fund-456"))
      .build();
    
    when(mockSalesforceService.findContractByAwsAccount("222222222222"))
      .thenReturn(contract);
    
    // Act
    AuthorizerResponse response = authorizer.authorize(mockTokenContext);
    
    // Assert
    assertEquals("Allow", response.getEffect());
    assertEquals("222222222222", response.getPrincipalId());
    assertEquals("READ_ONLY", response.getContext().get("accessLevel"));
  }
  
  @Test
  void testAuthorize_InactiveSubscription_ReturnsDeny() {
    // Arrange
    SubscriptionStatus inactiveSubscription = SubscriptionStatus.builder()
      .isActive(false)
      .reason("Payment overdue")
      .build();
    
    when(mockAdxClient.getSubscriptionStatus("222222222222"))
      .thenReturn(inactiveSubscription);
    
    // Act
    AuthorizerResponse response = authorizer.authorize(mockTokenContext);
    
    // Assert
    assertEquals("Deny", response.getEffect());
  }
  
  @Test
  void testAuthorize_UnauthorizedFund_ReturnsDeny() {
    // Arrange
    SubscriptionStatus activeSubscription = SubscriptionStatus.builder()
      .isActive(true)
      .build();
    
    when(mockAdxClient.getSubscriptionStatus("222222222222"))
      .thenReturn(activeSubscription);
    
    Contract contract = Contract.builder()
      .authorizedFunds(List.of("fund-999"))  // fund-123 NOT in list
      .build();
    
    when(mockSalesforceService.findContractByAwsAccount("222222222222"))
      .thenReturn(contract);
    
    // Act
    AuthorizerResponse response = authorizer.authorize(mockTokenContext);
    
    // Assert
    assertEquals("Deny", response.getEffect());
  }
}
```

---

## Section 11: Blue-Green Deployment Strategy for API Authorizer

### 11.1 Safe Deployment: Zero Downtime for Authorization Updates

**Scenario**: Update API Gateway authorizer to add new validation logic (e.g., deny requests from suspended clients)

#### Step 1: Blue Environment (Current Production)

```
API Gateway
├─ Primary Authorizer: arn:aws:lambda:...:function:ADXAPIAuthorizer
│  └─ Version: v1.2.3 (current production)
│     ├─ Validates subscription status
│     ├─ Checks fund access
│     └─ Logs all auth events
└─ Traffic: 100% → Blue Authorizer
```

#### Step 2: Green Environment (New Version)

```java
// New authorizer version adds suspended client check
@Service
public class ADXAPIAuthorizerV2 {  // New version
  
  public AuthorizerResponse authorize(JsonWebTokenContext context) {
    try {
      // 1. Validate subscription (existing)
      SubscriptionStatus subscription = adxClient.getSubscriptionStatus(context.getAccountId());
      if (!subscription.isActive()) {
        return denyAccess("Subscription inactive");
      }
      
      // 2. NEW: Check if client is suspended (new validation)
      ClientProfile clientProfile = salesforceService.getClientProfile(context.getAccountId());
      if ("SUSPENDED".equals(clientProfile.getStatus())) {
        // Alert: Client attempting queries while suspended
        alertSecurityTeam("Suspended client attempted query: " + context.getAccountId());
        return denyAccess("Client account suspended");
      }
      
      // 3. Existing fund access check
      boolean hasAccess = checkFundAccess(context.getAccountId(), context.getResourceId());
      if (!hasAccess) {
        return denyAccess("Not authorized for fund " + context.getResourceId());
      }
      
      return AuthorizerResponse.builder()
        .principalId(context.getAccountId())
        .effect("Allow")
        .build();
      
    } catch (Exception ex) {
      logger.error("Authorization failed", ex);
      return denyAccess("Authorization error");
    }
  }
}
```

#### Step 3: Deployment Process

```bash
#!/bin/bash
# blue-green-deployment.sh

set -e  # Exit on first error

FUNCTION_NAME="ADXAPIAuthorizer"
REGION="us-east-1"
AWS_ACCOUNT="111111111111"
API_GATEWAY_ID="abc123def456"

# Step 1: Build new authorizer
echo "[1/7] Building new authorizer..."
mvn clean package -DskipTests

# Step 2: Create new Lambda version (Green)
echo "[2/7] Creating new Lambda version (Green)..."
NEW_VERSION=$(aws lambda publish-version \
  --function-name $FUNCTION_NAME \
  --description "v2.0.0 - Added suspended client check" \
  --region $REGION \
  --query 'Version' \
  --output text)

echo "New version: $NEW_VERSION"

# Step 3: Run smoke tests against Green
echo "[3/7] Running smoke tests against new version..."
./test-scripts/smoke-tests.sh $NEW_VERSION

if [ $? -ne 0 ]; then
  echo "❌ Smoke tests failed! Aborting deployment."
  exit 1
fi

# Step 4: Canary: 5% traffic to Green (old clients see Blue)
echo "[4/7] Starting canary (5% traffic to Green)..."
aws lambda update-alias \
  --function-name $FUNCTION_NAME \
  --name live \
  --routing-config "AdditionalVersionWeight=0.05,RoutingConfig={FunctionVersion=$NEW_VERSION}" \
  --region $REGION

sleep 300  # Monitor for 5 minutes

# Check CloudWatch error rate
ERROR_RATE=$(aws cloudwatch get-metric-statistics \
  --namespace AWS/Lambda \
  --metric-name Errors \
  --dimensions Name=FunctionName,Value=$FUNCTION_NAME \
  --start-time $(date -u -d '5 minutes ago' +%Y-%m-%dT%H:%M:%S) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
  --period 300 \
  --statistics Sum \
  --region $REGION \
  --query 'Datapoints[0].Sum' \
  --output text)

if [ "$ERROR_RATE" != "None" ] && [ $(echo "$ERROR_RATE > 5" | bc) -eq 1 ]; then
  echo "❌ Error rate too high during canary ($ERROR_RATE%)! Rolling back."
  aws lambda update-alias \
    --function-name $FUNCTION_NAME \
    --name live \
    --function-version $(aws lambda list-versions-by-function --function-name $FUNCTION_NAME --query 'Versions[-2].Version' --output text) \
    --region $REGION
  exit 1
fi

# Step 5: Gradually shift traffic: 25% → Green
echo "[5/7] Shifting 25% traffic to Green..."
for weight in 0.25 0.50 0.75; do
  aws lambda update-alias \
    --function-name $FUNCTION_NAME \
    --name live \
    --routing-config "AdditionalVersionWeight=$weight,RoutingConfig={FunctionVersion=$NEW_VERSION}" \
    --region $REGION
  sleep 120  # Monitor for 2 minutes each
done

# Step 6: Full cutover to Green
echo "[6/7] Full cutover to Green..."
aws lambda update-alias \
  --function-name $FUNCTION_NAME \
  --name live \
  --function-version $NEW_VERSION \
  --region $REGION

# Step 7: Cleanup & notification
echo "[7/7] Deployment complete!"
aws sns publish \
  --topic-arn "arn:aws:sns:$REGION:$AWS_ACCOUNT:deployment-notifications" \
  --subject "✅ ADX Authorizer deployed to version $NEW_VERSION" \
  --message "Blue-green deployment successful. New suspended client check is now live."
```

**Deployment Timeline**:
- **T+0**: Deploy new version (0 impact, no traffic)
- **T+5min**: Canary 5% (5 suspicious clients try → 95% unaffected)
- **T+12min**: 25% traffic shift (monitor)
- **T+14min**: 50% traffic (confirm no errors)
- **T+16min**: 75% traffic (continue monitoring)
- **T+18min**: 100% cutover to Green
- **Total**: 18 minutes end-to-end, zero customer impact

---

## Section 12: Team Composition & Hiring Plan for Tier 4

### 12.1 Staffing Requirements: 10 FTE Team

| Role | FTE | Responsibilities | Skills Required | Salary Range |
|------|-----|------------------|-----------------|--------------|
| **Principal Architect** (Lead) | 1.0 | Design decisions, ADX strategy, governance | AWS certifications, Nomura experience, Zero-ETL patterns | $250K - $350K |
| **Platform Engineer** | 1.0 | ADX APIs, automation, Lambda orchestration | Spring Cloud, EventBridge, Step Functions | $180K - $220K |
| **Senior Java Engineer** | 1.0 | API authorizer, Lambda handlers, testing | Spring Boot 3.x, Mockito, JUnit 5, concurrent systems | $160K - $200K |
| **Java Engineer** | 1.0 | Code integration, feature development | Java 17+, Spring Boot, Maven | $140K - $180K |
| **Data Engineer** | 1.0 | Lake Formation, Glue Data Catalog, S3 optimization | SQL, AWS Glue, Spark, data modeling | $150K - $190K |
| **Security Engineer** | 1.0 | IAM policies, compliance audits, zero-trust | AWS IAM, Lake Formation, SOC2, GDPR | $170K - $210K |
| **DevOps Engineer** | 1.0 | CI/CD, Lambda deployment, blue-green orchestration | Terraform, GitHub Actions, CloudFormation | $150K - $190K |
| **QA Automation** | 1.0 | Integration testing, performance testing | Java, Selenium, Gatling, chaos engineering | $120K - $160K |
| **Solutions Architect** (PM liaison) | 0.5 | Client onboarding, documentation, training | Technical writing, customer empathy | $100K - $140K (part-time equiv) |
| **Engineering Manager** (part-time lead) | 0.5 | Team coordination, dependencies, escalations | Leadership, budget management | Included in manager's base |

**Total Annual Payroll**: $1.5M - $1.9M

### 12.2 Hiring Timeline: 6-Month Ramp

| Month | Hires | Focus | Milestones |
|-------|-------|-------|-----------|
| **Month 1** | Architect + Platform Engineer | Learning + design phase | Complete ADX architecture; define automation flows |
| **Month 2** | + 1 Senior Java Engineer | Coding ramp | Implement API authorizer + Lambda handlers |
| **Month 3** | + 1 Data Engineer + Security Engineer | Hardening phase | Lake Formation setup; IAM policies defined |
| **Month 4** | + 1 Java Engineer + DevOps Engineer | Test + deploy phase | CI/CD pipeline; blue-green deployment |
| **Month 5** | + QA Automation + Solutions Architect (0.5) | QA + docs phase | Integration tests; customer onboarding docs |
| **Month 6+** | Ongoing support | Go-live | 50+ clients onboarded; monitoring + ops |

---

## Section 13: Knowledge Transfer Plan

### 13.1 Documentation Standards

```
Every ADX feature must have:

1. Architecture Decision Record (ADR)
   - Title: "ADR-00X: [Decision Name]"
   - Context: Why this decision matters
   - Decision: What we chose and why
   - Consequences: Trade-offs
   - Alternatives: What we rejected

2. Implementation Guide
   - Purpose: What problem does this solve?
   - Prerequisites: What must be set up first?
   - Step-by-step setup: 1. Create IAM role, 2. Deploy Lambda, etc.
   - Troubleshooting FAQ: Common errors + fixes

3. Code Comments
   - Non-obvious logic: Explain the "why" not the "what"
   - IAM decisions: Why this specific permission?
   - Performance trade-offs: Why did we choose this approach?

4. Runbook (On-Call SOP)
   - Issue title: "Lambda Authorization Failures Spike"
   - Detection: How to know problem exists
   - Investigation: What to check first
   - Resolution: Step-by-step fix
   - Escalation: Who to notify if stuck
```

### 13.2 Cross-Training Program

```
Pairing Assignments (every engineer owns 1 ADX feature):
- Engineer A: Owns "ADX Revision Publishing" (pairs with Architect Month 1)
- Engineer B: Owns "API Authorizer" (pairs with Senior Java engineer Month 2)
- Engineer C: Owns "Event-Driven Cache Refresh" (pairs with Platform engineer Month 2)
- Engineer D: Owns "Lake Formation RLS Configuration" (pairs with Data engineer Month 3)
- Engineer E: Owns "Blue-Green Deployment" (pairs with DevOps Month 4)

Ownership includes:
- Implement the feature
- Document ADR + runbook
- Maintain the component (first 3 months)
- Mentor new team member on this component
- Present monthly architecture review on component health
```

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

## Section 14: Performance SLAs & Benchmarks

### 14.1 Target Latency by Component

| Component | Operation | p95 Latency | p99 Latency | SLA | Notes |
|-----------|-----------|-------------|-------------|-----|-------|
| **API Authorization** | SigV4 validation + subscription check | 30ms | 50ms | 99.9% | Cache subscription status → refresh hourly |
| **Redshift Query** | SELECT funds table (1K rows) | 200ms | 350ms | 99.95% | Partition pruning on partition key |
| **S3 GetObject** | Single object read from Gold layer | 100ms | 150ms | 99.9% | S3 in same region as Redshift |
| **CrossAccount S3 Access** | IAM assume role + read | 150ms | 250ms | 99.9% | STS token cached in Lambda |
| **Lake Formation RLS** | Row filtering on 10M row table | 50ms | 75ms | 99.95% | In-memory filter with B-tree index |
| **ADX Grant** | IAM policy attachment via Step Functions | 2-5s | 10s | 99% (async) | Separate from query path |
| **End-to-End API** | Client → API Gateway → Authorizer → Lambda → Athena | 400ms | 600ms | 99.5% | p95 target: <500ms SLA |

### 14.2 Throughput Targets
- Concurrent API Clients: 10,000+ (API Gateway limit 20K)
- Queries/Second: 1,000 QPS (Athena: 100 concurrent max; queue via SQS)
- Revisions/Day: 10 (ADX limit: 5 rev/min = 7,200/day)
- Entitlements/Hour: 100 new clients (Step Functions: 100 parallel)

---

## Section 15: Strategic Risk Assessment & Mitigation

### 15.1 Risk: Vendor Lock-In (AWS Proprietary)

**Mitigation**: "Petabyte-Scale Portability"
- All data stored in S3 (standard format)
- Export via AWS Glue → Parquet (open format, readable by Azure/GCP/on-prem)
- Contract clause: "90-day data portability guarantee upon termination"
- Strategic advantage: Competitors locked-in premise-managed silos; Nomura data is portable

### 15.2 Risk: Data Breach via Compromised Credentials

**Mitigation**: Defense-in-Depth (5 layers)
1. **AWS IAM**: MFA on all logins, credential rotation daily, STS tokens (1hr expiry)
2. **Lake Formation RLS**: Each client sees ONLY their funds (row-level filtering)
3. **Encryption**: TLS 1.3 in-transit, S3 SSE-KMS at-rest, quarterly key rotation
4. **Network**: VPC endpoints (no internet routing), security groups, private subnets
5. **Detection**: GuardDuty + Security Hub with 60-second auto-disable for compromised roles

**Result**: Even if one layer breached, cross-tenant data remains inaccessible

### 15.3 Risk: Data Quality Issues (Corrupted Revisions)

**Mitigation**: Multi-Stage Quality Framework
- Pre-publication: Row count, schema, statistical anomaly checks
- Staged publication: DRAFT → TESTING (internal only) → CANARY (5% clients) → PRODUCTION
- Auto-rollback: If >5% clients report errors, revert to previous revision
- Post-incident: Root cause + corrected revision within 1 hour

---

## Section 16: Customer Support Troubleshooting Guide

### Quick-Start Flowchart (Issue: "Can't query Nomura data")
1. **Check 1**: Do you see "nomura_external_data" in Athena?
   - NO → S3 grant pending; wait 15 min + refresh data source
   - YES → Check 2
2. **Check 2**: Can you list S3 bucket?
   - NO → IAM permission missing; contact AWS admin + Nomura support
   - YES → Check 3
3. **Check 3**: Test Athena query: `SELECT COUNT(*) FROM nomura_esg_insights LIMIT 1`
   - SUCCESS ✅ → Proceed to dashboard setup
   - "Table not found" → Refresh Glue catalog manually
   - Syntax error → Verify query format

### Tier 2 Escalation (Nomura Support)
- S3 access granted but 0 rows returned → Check if data published today
- Athena error "Access Denied" → Check Lake Formation RLS logs
- Multi-region failover active → Expect 1-hour stale data from secondary region
- SLA for Tier 2 response: <2 hours

---

## Section 17: SOC2 & GDPR Compliance Checklist

### SOC2 Type II Status (Annual Audit)
- ✅ CC6.1: Written security policy (see Section 15: Defense-in-Depth)
- ✅ A1.2: Data access authorization via Lake Formation RLS + IAM
- ✅ C1.2: Data encrypted in transit (TLS 1.3) + at-rest (SSE-KMS)
- ✅ C1.3: Access controls via MFA + Secrets Manager rotation
- ✅ P5.3: Incident response procedures documented (section 15.3)
- ✅ P6.2: Continuous monitoring via GuardDuty + CloudWatch alerts

### GDPR Compliance (Data Subject Rights)
| Right | Timeline | Implementation |
|------|----------|-----------------|
| Right to Access | 30 days | Export client data to Parquet via Glue |
| Right to Erasure | 30 days | Mark client data "deleted" in Lake Formation RLS |
| Right to Portability | 30 days | Glue job exports to client's S3 bucket (Parquet) |
| Right to Object | Immediate | ADX data product marked "PAUSED" for client |
| Breach Notification | 72 hours | SNS alert + regulatory notification |

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

