# Asset Management Client Technology - Architecture Overview

## Executive Summary

This repository documents the comprehensive technology architecture for an Assessment Management FinTech company's **Client Technology and Integration** domain. It represents a strategic shift from traditional push-based reporting to a multi-modal, data-driven distribution mesh that enables institutional clients (hedge funds, pension funds, asset managers) to consume proprietary insights through their preferred integration channels.

**Mission**: Enable 60% reduction in client onboarding time for data delivery while maintaining 100% automated entitlement synchronization across platforms.

---

## 1. Strategic Context

### Business Problem
Asset management firms face increasing pressure to:
- **Accelerate client adoption** of proprietary data and insights
- **Reduce operational overhead** in manual data delivery and entitlement management
- **Enable omnichannel access** (Web, API, Streaming, Marketplace)
- **Ensure security and scalability** across thousands of institutional clients

### Role & Mandate
As the senior engineering leader for Client Technology and Integration, you are accountable for:
- **End-to-end delivery** (requirements → architecture → development → release → support)
- **Strategic roadmap development** aligned with KPIs and business partner needs
- **Architecture governance** ensuring patterns, security, run costs, and UX excellence
- **Team leadership** of principal engineers, architects, and software engineering managers
- **Digitalization & automation** initiatives with business partners (Sales, Marketing, Distribution)

---

## 2. Multi-Modal Data Distribution Mesh Architecture

The core architectural pattern evolves from a "push-based" reporting model to a **4-Tier Distribution Mesh**, allowing clients to consume data via their preferred channel.

```
┌─────────────────────────────────────────────────────────────┐
│           NOMURA ASSET MANAGEMENT DATA SERVICES              │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌─────────────┐  ┌──────────────┐  ┌────────────┐          │
│  │   Digital   │  │    SaaS      │  │   Async    │          │
│  │ Dashboards  │  │   REST APIs  │  │   Kafka    │          │
│  └──────┬──────┘  └──────┬───────┘  └────┬───────┘          │
│         │                │               │                   │
│         └────────────────┼───────────────┴──────┐            │
│                          │                      │            │
│                          ▼                      ▼            │
│              ┌─────────────────────────────────────┐         │
│              │   Core Data Processing Layer        │         │
│              │   (Medallion Architecture)          │         │
│              │   Bronze → Silver → Gold            │         │
│              └──────────────┬──────────────────────┘         │
│                             │                                 │
│          ┌──────────────────┼──────────────────────┐         │
│          ▼                  ▼                      ▼         │
│   ┌────────────┐    ┌──────────────┐     ┌────────────┐   │
│   │ AWS Data   │    │ Data Product │     │ Audit &    │   │
│   │ Exchange   │    │ Marketplace  │     │ Compliance │   │
│   │ (ADX)      │    │              │     │ Layer      │   │
│   └────────────┘    └──────────────┘     └────────────┘   │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### 2.1 Tier 1: Digital Layer (Web/Mobile/Dashboards)

**Purpose**: Real-time visualization and interactive analytics for Portfolio Managers and Relationship Managers

**Technology Stack**:
- **Frontend**: React + TypeScript on AWS Amplify
- **Visualization**: PowerBI for self-service analytics
- **Backend Compute**: Amazon Athena + S3 (Medallion Architecture)
- **Real-time Data**: Amazon QuickSight for dashboards
- **Authentication**: AWS Cognito + SAML integration

**Key Characteristics**:
- Low-latency queries (<5 seconds for complex aggregations)
- Multi-tenancy with role-based access control (RBAC)
- Real-time performance attribution
- Customizable dashboards per client

**Sample Data Products**:
- NAV history and performance analytics
- Risk factor decomposition
- Fee impact analysis
- Client reporting templates

---

### 2.2 Tier 2: SaaS Tier (REST APIs)

**Purpose**: Programmatic access for client developers integrating with their own systems

**Technology Stack**:
- **API Gateway**: AWS API Gateway (REST + GraphQL)
- **Compute**: AWS Lambda (serverless) with auto-scaling
- **Authentication**: OIDC/OAuth 2.0 + IAM roles
- **Rate Limiting**: Token bucket algorithm with per-client quotas
- **Caching**: Amazon ElastiCache (Redis) for frequently accessed endpoints

**Key Characteristics**:
- RESTful and GraphQL endpoints
- Request-Response synchronous pattern
- Secure credential management via AWS Secrets Manager
- Webhook support for event notifications
- API versioning and deprecation management

**Sample Endpoints**:
```
GET  /v2/funds/{fundId}/nav
GET  /v2/funds/{fundId}/performance?period=YTD
POST /v2/reports/generate
GET  /v2/entitlements/me
```

**SLA**: 99.99% uptime, p95 latency <200ms

---

### 2.3 Tier 3: Async Tier (Apache Kafka / Streaming)

**Purpose**: High-frequency institutional trading and real-time risk engines

**Technology Stack**:
- **Message Broker**: Amazon Managed Streaming for Apache Kafka (MSK)
- **Producers**: Custom ingestion services (Java/Python)
- **Consumers**: Client applications or data warehouses
- **Schema Registry**: Confluent Schema Registry
- **Monitoring**: Prometheus + Grafana

**Key Characteristics**:
- Topics act as domain contracts (immutable schemas)
- Event sourcing pattern for audit trail
- Multi-partition distribution for high throughput
- Consumer group management with offset commits
- Exactly-once semantics for critical flows

**Sample Kafka Topics**:
```
asset-management.nav.intraday          // Streaming NAV updates
asset-management.trades.execution      // Trade confirmations
asset-management.risk.alerts           // Risk threshold breaches
asset-management.performance.metrics   // Real-time performance metrics
```

**Throughput**: 100K+ events/second per topic

---

### 2.4 Tier 4: Marketplace Tier (AWS Data Exchange)

**Purpose**: Zero-copy data distribution for institutional data scientists and quant shops

**Technology Stack**:
- **Distribution Platform**: AWS Data Exchange (ADX)
- **Storage Backend**: S3 + Redshift
- **Billing Engine**: AWS ADX native billing
- **Entitlements**: Salesforce + AWS IAM sync
- **Data Catalog**: AWS Glue Data Catalog

**Key Characteristics**:
- Fully serverless, no infrastructure management
- Automatic billing and subscriber management
- Data products discoverable via AWS Marketplace
- Subscribers query data in their own AWS environment
- "Zero-copy" = no data egress charges for Nomura

**Data Products**:
```
Nomura ESG Insights                // Quarterly ESG scores
Nomura Private Credit Performance  // NAV and cash flow analysis
Nomura Hedge Fund Benchmarks       // Cross-fund performance rankings
Nomura Macro Signals               // Leading indicators
```

**Business Model**: Subscription-based with usage-based pricing

---

## 3. Core Integration Patterns

### 3.1 Entitlement Automation: Salesforce ↔ AWS Data Exchange

**Problem**: Manual ticket process delays data access provisioning by 2-5 days

**Solution**: Event-driven entitlement sync pipeline

```
┌──────────────────────────────────────────────────────────┐
│ Sales Executive closes deal in Salesforce Sales Cloud    │
│ (Contract = "Nomura ESG Insights" dataset)               │
└────────────────────────┬─────────────────────────────────┘
                         │
                         ▼
        ┌────────────────────────────────────┐
        │ Salesforce Change Data Capture     │
        │ (Outbound Messages)                │
        └────────────────┬───────────────────┘
                         │
                         ▼
        ┌────────────────────────────────────┐
        │ AWS Event Bridge                   │
        │ (Route to Step Functions)          │
        └────────────────┬───────────────────┘
                         │
                         ▼
        ┌────────────────────────────────────┐
        │ AWS Step Functions Orchestration   │
        │ 1. Validate client record          │
        │ 2. Lookup client AWS Account ID    │
        │ 3. Call ADX API to grant access    │
        │ 4. Send confirmation to Salesforce │
        └────────────────┬───────────────────┘
                         │
                         ▼
        ┌────────────────────────────────────┐
        │ Client AWS Account now has access  │
        │ to ESG Insights dataset            │
        │ (Zero manual intervention)         │
        └────────────────────────────────────┘
```

**Implementation Details**:
- **Latency**: <2 minutes from Sales Cloud click to active in ADX
- **Compliance**: Audit trail in CloudTrail + Salesforce
- **Rollback**: Automatic revocation upon contract termination
- **Error Handling**: Dead letter queue for manual intervention

---

### 3.2 Data Integrity & Quality Framework

#### The "Lemons" Problem
When data quality issues arise (stale NAV, missing performance data), clients experience friction and lose confidence.

#### Mitigation Strategy

| Issue | Mitigation | Implementation |
|-------|-----------|-----------------|
| **Data Staleness** | Real-time freshness monitoring | CloudWatch alarms if NAV not updated by 5PM |
| **Missing Values** | Null/NaN detection before publishing | AWS Glue data validation jobs |
| **Schema Drift** | Strict schema evolution governance | Confluent Schema Registry with BACKWARD compatibility |
| **PII Exposure** | Automated PII scanning | AWS Macie on all data products before ADX publish |
| **Cross-Fund Contamination** | Multi-tenancy isolation tests | Row-level security (RLS) in Athena + Redshift |

---

### 3.3 Run Cost Optimization

**Key Innovation: Shift from Nomura Pull → Client Push**

**Traditional Model** (Expensive):
- Nomura egresses data to thousands of clients (expensive AWS Data Transfer)
- Nomura pays for compute to transform + deliver
- Result: $500K+/year for large client bases

**New Model** (Cost-Efficient):
- Nomura publishes data once to S3 (Medallion Gold Layer)
- AWS ADX handles multi-tenancy at storage layer
- Clients query via their own compute (they pay for it)
- Result: Nomura pays only for master dataset storage (~$50K/year)

**Cost KPI**: 85% reduction in data delivery run costs

---

## 4. Technology Stack & Choices

### 4.1 Compute & Data

| Layer | Service | Reasoning |
|-------|---------|-----------|
| **Storage** | S3 + Data Lake | Unlimited scalability, cost-effective |
| **Data Warehouse** | Redshift Spectrum | Federated queries across S3 + structured data |
| **Real-Time Analytics** | Amazon Athena | Serverless SQL, automatic query optimization |
| **Streaming** | Amazon MSK | Managed Kafka, 99.99% SLA |
| **Data Catalog** | AWS Glue | Automated schema discovery, lineage tracking |
| **API Runtime** | Spring Boot 3.x on ECS/Lambda | Java ecosystem; Spring Cloud integration |
| **Service Mesh** | AWS App Mesh (optional) | Traffic management, observability |

### 4.2 Integration & APIs

| Layer | Technology | Standard |
|-------|-----------|----------|
| **API Protocol** | REST + GraphQL | Industry standard for integrations |
| **API Framework** | Spring Boot WebFlux | Reactive, non-blocking for high throughput |
| **Authentication** | OIDC / OAuth 2.0 + Spring Security | Zero-trust security model |
| **Event Streaming** | Kafka + CloudEvents | CNCF standard for events |
| **Kafka Clients** | Spring Cloud Stream + Kafka Streams | Java/JVM ecosystem |
| **Data Format** | Parquet + JSON | Columnar storage for analytics; JSON for APIs |
| **Observability** | Micrometer + Prometheus + CloudWatch | Standard JVM metrics collection |

### 4.3 Security & Compliance

| Domain | Implementation |
|--------|-----------------|
| **Identity & Access** | AWS IAM + SAML + MFA + Spring Security |
| **Data Encryption** | TLS 1.3 (transit), AES-256 (at rest) |
| **Secrets Management** | AWS Secrets Manager + Spring Cloud Config |
| **Audit Logging** | CloudTrail + ClamAV scanning + Spring Audit |
| **Compliance** | SOC 2 Type II, FINRA CAT reporting |
| **API Rate Limiting** | Token bucket + Spring Cloud Gateway | Prevent abuse and cascading failures |

### 4.4 Cost Estimation (Monthly Baseline)

**Assumptions**: 1000 API clients, 100K NAV updates/day, ADX: 50 subscribers

| Service | Volume | Unit Cost | Monthly Cost |
|---------|--------|-----------|-------------|
| **S3 (Data Lake)** | 50 TB | $0.023/GB | $1,150 |
| **Redshift** | 1 cluster (Ra3, 2 nodes) | $2.96/node/hr | $4,300 |
| **Lambda (APIs)** | 500M requests/mo | $0.20/1M | $100 |
| **MSK (Kafka)** | 3 brokers, 2.8TB/mo | $0.28/broker/hr + storage | $1,800 |
| **Athena (Queries)** | 5K queries/mo | $5/TB scanned | $250 |
| **AWS Data Exchange** | 50 subscribers | $0-100% share of margin | $5,000 |
| **NAT Gateway** | 1 TB/mo egress | $0.045/GB | $45 |
| **CloudWatch** | Logs + metrics | ~50GB ingest | $500 |
| **Development/Testing** | - | ~30% of production | $4,000 |
| **TOTAL** | - | - | **~$17,145/month (~$205K/year)** |

**Savings vs. Legacy (push-based model)**: $450K/year (69% reduction)

---

## 5. KPIs & Success Metrics

### 5.1 Operational KPIs

| KPI | Target | Current | Status |
|-----|--------|---------|--------|
| **Client Onboarding Time** | <24 hours | 3-5 days | 🎯 In Progress |
| **Entitlement Sync Automation** | 100% | 30% (manual) | 🎯 In Progress |
| **Data Freshness** | <5 min latency | 4 hours (batch) | 🎯 In Progress |
| **API Success Rate** | 99.99% | 99.2% | ⚠️ Needs Improvement |
| **Time to Market (new data product)** | <1 week | 4 weeks | 🎯 In Progress |

### 5.2 Business Impact KPIs

| KPI | Target | Benefit |
|-----|--------|---------|
| **Run Cost Reduction** | 85% | $450K annual savings |
| **Client Friction Score** | <2/10 | Improved NPS by 15 points |
| **Data Adoption Rate** | 80%+ | Increased sticky revenue (APIs/ADX) |
| **Marketplace Revenue** | $2M annual | New revenue stream |

---

## 6. Roadmap & Phasing

### Phase 1 (Q1 2026): MVP - Multi-Modal Foundation
- ✅ Digital Layer: PowerBI dashboards (internal + first 5 pilot clients)
- ✅ SaaS Tier: REST APIs for NAV + Performance endpoints
- ⏳ Async Tier: Kafka topics for trade execution events
- ⏳ Entitlement Automation: Salesforce → Step Functions pipeline

**Deliverable**: 60% of client base consuming via at least 2 channels

### Phase 2 (Q2 2026): Marketplace & Extensibility
- ✅ AWS Data Exchange product launch
- ✅ Automated entitlement sync (100% automation)
- ✅ GraphQL API layer (for complex queries)
- ⏳ Client self-service dashboard builder

**Deliverable**: $1M ADX subscription revenue; <24hr onboarding

### Phase 3 (Q3 2026): Intelligence & Optimization
- ⏳ AI-powered anomaly detection (data quality)
- ⏳ Predictive entitlement provisioning
- ⏳ Cost optimization (reserved capacity negotiation)
- ⏳ Advanced analytics (consumption patterns, churn prediction)

**Deliverable**: 80% data adoption; 85% cost reduction achieved

---

## 7. Engineering Team Structure

### Organization Chart

```
Senior Engineering Leader (You)
├── Principal Software Engineer (Platform Architecture)
│   ├── Senior Engineer (APIs & Integration)
│   ├── Senior Engineer (Data Pipeline)
│   └── Senior Engineer (Security)
├── Principal Architect (Solutions)
│   ├── Architect (Data Platform)
│   ├── Architect (Client Experience)
│   └── Architect (Operations)
└── Engineering Manager
    ├── Team Lead (Backend Services) - 4 engineers
    ├── Team Lead (Data / Analytics) - 4 engineers
    └── Team Lead (DevOps / Infra) - 3 engineers
```

### Responsibilities by Role

| Role | Key Responsibilities |
|------|----------------------|
| **You (Senior Leader)** | Vision, roadmap, stakeholder alignment, hiring, architecture governance |
| **Principal Engineers** | Technical depth, mentorship, design reviews, innovation initiatives |
| **Architects** | Solution design, cross-team collaboration, risk mitigation |
| **Engineering Manager** | Team velocity, hiring, 1-on-1s, delivery accountability |
| **Individual Contributors** | Feature delivery, code quality, documentation, on-call support |

---

## 8. Critical Success Factors

### 8.1 Technical Pillars
1. **Scalability**: Handle 10x growth in client base and data volume
2. **Reliability**: 99.99% SLA across all tiers
3. **Security**: Zero-trust architecture; zero data breaches
4. **Observability**: Real-time alerting, end-to-end tracing
5. **Automation**: Self-healing, minimal manual operations

### 8.2 Organizational Pillars
1. **Alignment**: Daily sync with Sales, Marketing, ops teams
2. **Empowerment**: Teams own their services end-to-end
3. **Learning**: Knowledge sharing, technical talks, mentorship
4. **Accountability**: Clear KPIs tied to compensation
5. **Autonomy**: Minimal process overhead, velocity focus

---

## 9. Architecture Decision Records (ADRs)

### ADR-001: Multi-Modal Distribution over Single Channel
**Decision**: Implement 4-tier distribution mesh (Web, API, Kafka, ADX) instead of single "API-first" approach

**Rationale**:
- **Clients have diverse technical maturity**: Some want dashboards (business users), others want Kafka streams (quant shops)
- **Different consumption patterns**: Batch (dashboards), interactive (APIs), real-time (Kafka)
- **Cost efficiency**: ADX zero-copy model only viable if clients can query their own compute
- **Time-to-market**: Phase in channels incrementally (Web → API → Kafka → ADX)

**Consequences**:
- ✅ Higher client adoption
- ❌ More operational complexity (4 platforms to manage)
- ❌ Testing matrix explodes (cross-channel consistency)

---

### ADR-002: AWS Data Exchange over Self-Managed Marketplace
**Decision**: Use AWS Data Exchange (managed) instead of building custom data marketplace

**Rationale**:
- **No infrastructure to manage**: ADX handles billing, entitlements, discovery
- **Time-to-value**: 2 weeks to launch vs. 6 months for custom
- **Security**: AWS managed compliance; we focus on data quality
- **Scalability**: Auto-scales to thousands of subscribers

**Consequences**:
- ✅ Faster go-to-market
- ✅ Lower run cost
- ❌ Vendor lock-in to AWS
- ❌ Limited customization (we adapt to ADX, not vice versa)

---

### ADR-003: Spring Boot + Reactive Streams for Java Services
**Decision**: Use Spring Boot 3.x with Spring Cloud Kafka + WebFlux for microservices

**Rationale**:
- **Non-blocking I/O**: WebFlux handles 10K+ req/sec without thread explosion
- **Ecosystem**: Spring Cloud Stream abstracts Kafka complexity
- **Testability**: Testcontainers for Kafka, embedded servers for Spring Boot
- **Team expertise**: Java/Spring is institutional standard

**Consequences**:
- ✅ Developer productivity
- ✅ Proven enterprise stack
- ❌ Reactive programming is hard to onboard (need training)
- ❌ Memory footprint higher than Go/Python alternatives

---

### ADR-004: Event Sourcing for Audit Trail
**Decision**: Publish all changes to Kafka topics (audit trail) alongside database writes

**Rationale**:
- **Compliance**: FINRA requires 7-year data retention
- **Debugging**: Replay events to reproduce issues
- **Temporal queries**: "Give me state of Fund X as of 2024-01-15"

**Consequences**:
- ✅ Complete audit history
- ✅ Debugging capability
- ❌ Storage overhead (~2x)
- ❌ Complexity in idempotency (duplicate events in Kafka)

---

## 9.1 Code Examples & Implementation Patterns

### Example 1: REST API for NAV Lookup (Spring Boot)

```java
// pom.xml dependencies
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-aws-secrets-manager-config</artifactId>
</dependency>
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>

// Controller
@RestController
@RequestMapping("/v2/funds")
@CrossOrigin(origins = "*", maxAge = 3600)
public class FundController {

  @Autowired private FundService fundService;
  @Autowired private MeterRegistry meterRegistry;
  
  private static final Logger logger = LoggerFactory.getLogger(FundController.class);

  @GetMapping("/{fundId}/nav")
  @Operation(summary = "Get current NAV for fund")
  public Mono<ResponseEntity<NavResponse>> getNAV(
      @PathVariable String fundId,
      @RequestParam(required = false) LocalDate asOfDate) {
    
    return fundService.fetchNAV(fundId, asOfDate)
      .doOnNext(nav -> {
        meterRegistry.counter("nav.requests.success").increment();
        logger.info("NAV fetched for fund={}, nav={}", fundId, nav.getValue());
      })
      .doOnError(ex -> {
        meterRegistry.counter("nav.requests.error").increment();
        logger.error("Error fetching NAV for fund={}", fundId, ex);
      })
      .map(nav -> ResponseEntity.ok(new NavResponse(nav, LocalDateTime.now())))
      .onErrorResume(ex -> Mono.just(
        ResponseEntity.status(500).body(
          new NavResponse(null, LocalDateTime.now())
        )
      ));
  }
}

// Service with retry logic
@Service
public class FundService {

  @Autowired private FundRepository fundRepo;
  @Autowired private AthenaClient athenaClient;
  
  private static final int MAX_RETRIES = 3;
  private static final int RETRY_DELAY_MS = 100;

  public Mono<NAVData> fetchNAV(String fundId, LocalDate asOfDate) {
    return Mono.fromCallable(() -> fundRepo.findNAVByFundId(fundId, asOfDate))
      .retryWhen(Retry.backoff(MAX_RETRIES, Duration.ofMillis(RETRY_DELAY_MS))
        .doBeforeRetry(signal -> {
          logger.warn("Retry attempt {}", signal.totalRetries() + 1);
        })
      )
      .subscribeOn(Schedulers.boundedElastic());
  }
}
```

### Example 2: Kafka Producer for NAV Events

```java
// KafkaProducerConfig.java
@Configuration
public class KafkaProducerConfig {

  @Value("${kafka.bootstrap.servers}")
  private String bootstrapServers;

  @Bean
  public ProducerConfig producerConfig() {
    return new ProducerConfig()
      .put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers)
      .put(ProducerConfig.RETRIES_CONFIG, 3)
      .put(ProducerConfig.ACKS_CONFIG, "all")  // Wait for all replicas
      .put(ProducerConfig.COMPRESSION_TYPE_CONFIG, "snappy")
      .put(ProducerConfig.LINGER_MS_CONFIG, 100)  // Batch for latency-throughput trade-off
      .put(ProducerConfig.BATCH_SIZE_CONFIG, 32768);
  }

  @Bean
  public KafkaTemplate<String, NAVUpdateEvent> kafkaTemplate() {
    return new KafkaTemplate<>(
      new DefaultKafkaProducerFactory<>(producerConfig().asProperties())
    );
  }
}

// NAVProducer.java
@Component
public class NAVProducer {

  @Autowired private KafkaTemplate<String, NAVUpdateEvent> kafkaTemplate;
  private static final String TOPIC = "asset-management.nav.intraday";
  private static final Logger logger = LoggerFactory.getLogger(NAVProducer.class);

  public void publishNAVUpdate(String fundId, BigDecimal nav, LocalDateTime timestamp) {
    NAVUpdateEvent event = NAVUpdateEvent.builder()
      .fundId(fundId)
      .nav(nav)
      .timestamp(timestamp)
      .eventId(UUID.randomUUID().toString())  // Idempotency key
      .build();

    kafkaTemplate.send(TOPIC, fundId, event)
      .addCallback(
        result -> logger.info("NAV event published: {}", fundId),
        ex -> logger.error("Failed to publish NAV event", ex)
      );
  }
}
```

### Example 3: Entitlement Automation - AWS Step Functions (JSON)

```json
{
  "Comment": "Automate entitlement provisioning from Salesforce to ADX",
  "StartAt": "ValidateSalesforceEvent",
  "States": {
    "ValidateSalesforceEvent": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:ACCOUNT:function:ValidateSalesforceEvent",
      "Retry": [{"ErrorEquals": ["States.TaskFailed"], "IntervalSeconds": 2, "MaxAttempts": 3, "BackoffRate": 2}],
      "Catch": [{"ErrorEquals": ["States.ALL"], "Next": "PublishErrorNotification"}],
      "Next": "LookupClientAWSAccountId"
    },
    "LookupClientAWSAccountId": {
      "Type": "Task",
      "Resource": "arn:aws:states:::dynamodb:getItem",
      "Parameters": {
        "TableName": "ClientAwsAccounts",
        "Key": {"ClientId": {"S.$": "$.salesforceClientId"}}
      },
      "Next": "CallADXGrantAccess"
    },
    "CallADXGrantAccess": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:ACCOUNT:function:ADXGrantDatasetAccess",
      "Parameters": {
        "DatasetId.$": "$.datasetId",
        "ClientAwsAccountId.$": "$.Item.AwsAccountId.S",
        "ExpirationDate.$": "$.contractEndDate"
      },
      "Retry": [{"ErrorEquals": ["ADXApiThrottled"], "IntervalSeconds": 5, "MaxAttempts": 5, "BackoffRate": 2}],
      "Catch": [{"ErrorEquals": ["States.ALL"], "Next": "PublishErrorNotification"}],
      "Next": "UpdateSalesforceCompletion"
    },
    "UpdateSalesforceCompletion": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:ACCOUNT:function:UpdateSalesforceField",
      "Parameters": {
        "RecordId.$": "$.salesforceRecordId",
        "Field": "ADX_Provisioned__c",
        "Value": "true"
      },
      "Next": "SuccessState"
    },
    "PublishErrorNotification": {
      "Type": "Task",
      "Resource": "arn:aws:states:::sns:publish",
      "Parameters": {
        "TopicArn": "arn:aws:sns:us-east-1:ACCOUNT:ADXProvisioningErrors",
        "Subject": "ADX Entitlement Provisioning Failed",
        "Message.$": "$"
      },
      "Next": "FailureState"
    },
    "SuccessState": {
      "Type": "Succeed"
    },
    "FailureState": {
      "Type": "Fail",
      "Error": "EntitlementProvisioningFailed",
      "Cause": "See CloudWatch logs for details"
    }
  }
}
```

---

## 9.2 Operational Runbooks

### Runbook 1: API Latency Spike Investigation

**Trigger**: CloudWatch alarm - p95 latency > 500ms

1. **Check Dashboard**: `aws cloudwatch get-dashboard --name FundAPIMetrics`
   - Look for: CPU %, Lambda cold start rate, Athena query time

2. **If Lambda cold starts high**:
   - Check: Recent deployments (CodeDeploy history)
   - Action: Increase Lambda provisioned concurrency
   - Rollback: `aws lambda put-provisioned-concurrency-config ...`

3. **If Athena query slow**:
   - Check: `SELECT COUNT(*) FROM s3://data-lake/gold/nav WHERE date = today()`
   - Root cause: Partition pruning failing?
   - Action: Add partition projection to table metadata

4. **If all metrics normal but clients report latency**:
   - Check: CloudFront cache hit ratio
   - Action: Invalidate cache if data updated: `aws cloudfront create-invalidation`

5. **Escalate if unresolved**: Page on-call Principal Engineer

---

### Runbook 2: Kafka Topic Offset Lag

**Trigger**: Consumer offset lag > 10,000 messages

1. **Identify lagging consumer group**:
   ```bash
   kafka-consumer-groups --bootstrap-server $KAFKA_BROKER --group $GROUP --describe
   ```

2. **Check consumer process**:
   ```bash
   ps aux | grep spring-cloud-stream
   kubectl get pods -l app=kafka-consumer
   ```

3. **If consumer is crashed**:
   - Restart: `kubectl rollout restart deployment/kafka-consumer`
   - Verify: `kubectl logs -f deployment/kafka-consumer`

4. **If consumer is running but still lagging**:
   - Check: Is it processing slowly? (Check application metrics)
   - Action: Increase consumer thread pool: `spring.cloud.stream.kafka.binder.concurrency=10`

5. **If lag continues after restart**:
   - Option A: Skip to latest offset (lose messages): `kafka-consumer-groups --reset-offsets --to-latest`
   - Option B: Investigate downstream system (is it slow to consume?)

---

### Runbook 3: Salesforce → ADX Provisioning Failure

**Trigger**: Execution role closed deal, but ADX shows no access after 5 minutes

1. **Check Step Function execution**:
   ```bash
   aws stepfunctions describe-execution --execution-arn <EXECUTION_ARN>
   aws stepfunctions get-execution-history --execution-arn <EXECUTION_ARN>
   ```

2. **Likely root causes**:
   | Error | Action |
   |-------|--------|
   | `LookupClientAWSAccountId` failed | Client record not in DynamoDB; ask sales team for AWS account ID |
   | `CallADXGrantAccess` timeout | ADX API might be throttling; retry with backoff |
   | `UpdateSalesforceCompletion` failed | Salesforce API key expired; rotate via AWS Secrets Manager |
   | Lambda timeout (>15min) | Hypothesis: ADX dataset too large; increase Lambda timeout |

3. **Manual recovery**:
   ```bash
   # Manually grant access to ADX (if automation stuck)
   aws dataexchange create-event-action \
     --action Name=SendDataSetNotification,Parameters='{"SnsTopicArn":"value"}' \
     --event Name=RevisionPublished,Parameters='{"DataSetId":"value"}'
   ```

4. **Notify client**: "ADX access provisioned manually; please refresh your AWS Console (refresh = 5min cache)"

---

## 9.3 Monitoring & Alerting Strategy

### Key Metrics Dashboard

| Metric | Threshold | Action |
|--------|-----------|--------|
| **REST API p95 latency** | > 500ms | Page on-call engineer |
| **REST API error rate** | > 1% (5xx errors) | Check Lambda logs, possible DDoS |
| **Kafka consumer lag** | > 10,000 messages | Check consumer app health |
| **ADX subscription failure rate** | > 5% | Check Salesforce-to-Step-Functions pipeline |
| **Data freshness (NAV)** | > 5 minutes stale | Check ETL job status |
| **Athena query cost/day** | > $500 | Investigate unoptimized queries |

---

## 10. Getting Started

### Prerequisites
- AWS Account with appropriate permissions (AdministratorAccess for setup)
- Salesforce System Admin access
- Kafka cluster (MSK or self-managed)
- .NET 9.0+, Python 3.11+, Node.js 18+, Java 17+
- Docker for local development
- Terraform 1.5+ for infrastructure provisioning

### Quick Start Guide

1. **Clone and setup**:
   ```bash
   git clone https://github.com/calvinlee999/Asset_Management_Client_Technology.git
   cd Asset_Management_Client_Technology
   
   # Configure AWS credentials
   export AWS_PROFILE=default
   aws sts get-caller-identity  # Verify access
   
   # Set environment variables
   cp .env.example .env
   source .env
   ```

2. **Deploy infrastructure**:
   ```bash
   cd infrastructure
   terraform init
   terraform plan -out=tfplan
   terraform apply tfplan
   ```

3. **Build & deploy services**:
   ```bash
   # API service
   cd ../src/apis
   ./gradlew bootRun -Dspring.profiles.active=local
   
   # Kafka producer
   cd ../kafka-producers
   python -m pip install -r requirements.txt
   python producer.py --env local
   ```

4. **Verify deployment**:
   ```bash
   # Test REST API
   curl http://localhost:8080/v2/funds/FND001/nav
   
   # Test Kafka
   kafka-console-consumer --bootstrap-server localhost:9092 \
     --topic asset-management.nav.intraday --from-beginning
   ```

### Repository Structure
```
Asset_Management_Client_Technology/
├── README.md                          # This file
├── EVALUATION_CYCLE_1.md              # Training feedback logs
├── docs/
│   ├── ADR/                           # Architecture Decision Records
│   ├── runbooks/                      # Operational procedures
│   │   ├── api-latency-investigation.md
│   │   ├── kafka-lag-resolution.md
│   │   └── salesforce-adx-provisioning.md
│   ├── data-lineage.md                # Data flow diagrams
│   ├── security-framework.md          # Compliance matrix
│   └── onboarding.md                  # New team member guide
├── src/
│   ├── apis/
│   │   ├── fund-service/              # Spring Boot REST APIs
│   │   └── performance-service/
│   ├── data-pipeline/
│   │   ├── etl/                       # Spark jobs (Bronze → Gold)
│   │   └── dbt/                       # Data transformation (dbt)
│   ├── kafka-producers/               # Event producers (Scala/Python)
│   ├── salesforce-integration/        # Entitlement sync Lambda
│   └── infrastructure/
│       ├── main.tf                    # Terraform root module
│       ├── networking.tf              # VPC, ECS, MSK
│       ├── data-lake.tf               # S3, Glue, Athena
│       └── monitoring.tf              # CloudWatch, Prometheus
├── tests/
│   ├── integration/                   # E2E API + Kafka tests
│   ├── performance/                   # Load testing (k6, JMeter)
│   └── security/                      # OWASP scanning, penetration tests
├── .github/
│   └── workflows/                     # CI/CD pipelines
│       ├── test.yml                   # Run on every PR
│       └── deploy.yml                 # Automated deployment
└── scripts/
    ├── api-smoke-tests.sh
    ├── generate-sample-data.sh
    └── backup-s3-data.sh
```

---

## 10.1 Team Onboarding Guide

### Day 1: Environment Setup
- [ ] Create AWS IAM user and configure local credentials
- [ ] Clone repository
- [ ] Setup IDE (VS Code with Java/Kotlin extensions)
- [ ] Spin up local Kafka cluster: `docker-compose -f docker/docker-compose.kafka.yml up`
- [ ] Run unit tests: `gradle test`

**Success criteria**: Can run `mvn spring-boot:run` and see API responding

### Week 1: Architecture Deep Dive
- [ ] Read [ADR-001 through ADR-004](#991-architecture-decision-records-adrs)
- [ ] Review [4-Tier Distribution Mesh](#2-multi-modal-data-distribution-mesh-architecture) section
- [ ] Pair with senior engineer on REST API walkthrough (2 hours)
- [ ] Run Kafka producer locally and consume messages

**Success criteria**: Can explain why ADX is better than self-managed marketplace

### Week 2: Hands-On Contribution
- [ ] Pick an issue labeled `good-first-issue`
- [ ] Submit first PR (even if small bug fix or documentation)
- [ ] Attend architecture review meeting (learn how team makes decisions)

**Success criteria**: First PR merged

### Week 3-4: Service Ownership Ramp
- [ ] Assigned to one service (e.g., "Fund API service")
- [ ] Become on-call shadow (pair with on-call engineer)
- [ ] Complete security training (AWS IAM, encryption, PII handling)

**Success criteria**: Can independently deploy hotfix to production (with 2-person review)

### Ongoing: Mentorship
- **1:1 with manager**: Bi-weekly, 30min
- **Peer programming**: 1x/week to learn from Principal Engineers
- **Tech talks**: Monthly brown-bag sessions on Kafka, observability, etc.
- **Career development**: 6-month career conversation

---

## 10.2 Definition of Done (DoD) - Service Delivery Checklist

Before any feature ships to production:

- [ ] **Code**
  - [ ] Unit test coverage ≥ 80%
  - [ ] Code reviewed by 2 engineers (1 must be Principal+)
  - [ ] No hardcoded secrets (use AWS Secrets Manager)
  - [ ] Passes checkstyle + SpotBugs + SonarQube gate

- [ ] **Testing**
  - [ ] Integration tests pass (E2E with Kafka + Athena)
  - [ ] Load test passes (>10K req/sec with <500ms p95)
  - [ ] Security scan passes (OWASP, dependency check)

- [ ] **Documentation**
  - [ ] API endpoints documented in OpenAPI/Swagger
  - [ ] Database schema documented
  - [ ] Runbook added if new operational procedure
  - [ ] ADR updated if architecture changed

- [ ] **Observability**
  - [ ] Metrics emitted (latency, errors, custom business metrics)
  - [ ] CloudWatch alarms configured
  - [ ] Logs in JSON format with structured fields

- [ ] **Deployment**
  - [ ] Terraform/CloudFormation updated for IaC
  - [ ] Feature flag added (for gradual rollout)
  - [ ] Database migration (if applicable) is backward compatible
  - [ ] Rollback plan documented

- [ ] **Sign-Off**
  - [ ] Product Manager approves feature
  - [ ] Platform lead approves architecture
  - [ ] On-call engineer approves operability
  - [ ] Security team approves compliance

---

## 11. Glossary

| Term | Definition |
|------|-----------|
| **ADX** | AWS Data Exchange - Marketplace for data products |
| **RBAC** | Role-Based Access Control for multi-tenancy |
| **MSK** | Amazon Managed Streaming for Apache Kafka |
| **RLS** | Row-Level Security - data isolation per client |
| **SLA** | Service Level Agreement - uptime and latency guarantees |
| **KPI** | Key Performance Indicator - business metrics |
| **PII** | Personally Identifiable Information - sensitive data |
| **ADR** | Architecture Decision Record - document to capture design decisions |
| **DoD** | Definition of Done - checklist before code ships to production |
| **ETL/ELT** | Extract-Transform-Load / Extract-Load-Transform - data integration patterns |

---

## 12. References & Learning Resources

- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [Confluent Kafka Best Practices](https://docs.confluent.io/)
- [Medallion Architecture](https://www.databricks.com/blog/2022/06/30/five-people-each-interpret-data-lakehouse-architecture-one-patterns.html)
- [Zero-Trust Security Model](https://www.nist.gov/publications/zero-trust-architecture)
- [Data as a Product](https://martinfowler.com/articles/data-mesh-principles.html)
- [Spring Boot Production Ready](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html)
- [AWS Data Exchange Developer Guide](https://docs.aws.amazon.com/data-exchange/latest/userguide/)
- [FINRA Regulatory Guidance](https://www.finra.org/rules-guidance)
- [Domain-Driven Design](https://www.domainlanguage.com/ddd/)
- [Gartner Data Mesh Principles](https://www.gartner.com/smarterwithgartner/what-is-a-data-mesh)

---

## 13. Contact & Support

For questions or contributions:
- **Technical Issues**: Open GitHub issue with label `bug` or `question`
- **Architecture Discussions**: Schedule sync with Senior Engineering Leader
- **Access Requests**: Contact infrastructure team via Slack #infrastructure-support
- **Security Concerns**: Email security@nomura.com (confidential)

### On-Call Escalation
- **Level 1**: Service team (first 15min)
- **Level 2**: Principal Engineer (if P1 issue)
- **Level 3**: VP of Engineering (if multiple services impacted)

---

**Last Updated**: February 18, 2026  
**Version**: 1.1 (Enhanced with Code Examples, ADRs, Runbooks, Onboarding)  
**Status**: Ready for Evaluation Cycle 2
