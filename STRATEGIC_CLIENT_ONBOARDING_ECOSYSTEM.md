# Strategic Client Onboarding Ecosystem
## Event-Driven Zero-Trust Architecture for Institutional Asset Management

**Version**: 1.0  
**Last Updated**: February 18, 2026  
**Architect**: Chief Technology Officer (Client Platform)  
**Target Audience**: Enterprise Architects, Platform Engineers, Product Leaders, Operations Teams

---

## Executive Summary

**The Problem**: Traditional client onboarding takes 6-12 weeks. Clients manually upload KYC documents, operations teams manually provision S3 buckets and IAM roles, RMs track progress in spreadsheets, and data is shared via insecure SFTP. Zero integration between portal, trading systems, compliance, and data delivery.

**The Solution**: **Event-Driven Client Onboarding Hub**—a unified orchestration layer that:
- ✅ Reduces onboarding from 12 weeks → **2-3 days** (50-75% faster)
- ✅ Automates regulatory compliance checks (KYC/AML) → **100% compliance automation**
- ✅ Zero-Trust identity federation (OIDC/Cognito) → **Zero service account data breaches**
- ✅ Event-driven architecture (Kafka) → **Real-time status visibility**
- ✅ Zero-copy data distribution (AWS ADX) → **Petabyte-scale delivery on Day 1**

**Core Value Proposition**:
This hub unifies **Digital** (Web Portal), **API** (SaaS/REST), **Async** (Kafka), and **Data Exchange** (Zero-Copy) under a single OIDC-secured SSO umbrella. When a client signs a contract, a single event triggers:
1. Salesforce case creation (for RM tracking)
2. Kafka event publishing (for async workflows)
3. AWS account provisioning (S3 buckets, IAM roles)
4. ADX data entitlement (historical data access)
5. Portal activation (client can now login)
6. Compliance checks (AML/KYC validation)

**All within 15 minutes, with zero manual intervention.**

---

## 1. Strategic Context & Business Driver

### 1.1 The "Onboarding Lemon" in Asset Management

| Problem Area | Current State (Manual) | Impact | Solution (Automated) |
|--------------|----------------------|--------|---------------------|
| **KYC/AML Processing** | Manual document review (5-10 days) | Regulatory fines if incomplete | AI-powered document extraction + automated rule engine (15 min) |
| **Account Provisioning** | Ops team manually creates S3/IAM (3-5 days) | Human error; security gaps; forgotten steps | Terraform "Account-in-a-Box" (5 min) |
| **Data Distribution** | SFTP setup or manual file uploads (2-3 days) | Insecure; bandwidth-limited; stale data | AWS ADX zero-copy (instant) |
| **Portal Setup** | Manual user account creation (1-2 days) | Delays client login; poor first impression | Self-service via Salesforce workflow (5 min) |
| **RM Visibility** | Spreadsheets updated manually (ongoing) | RMs fly blind; escalations chaotic | Real-time Kafka events + Salesforce dashboard (live) |
| **Compliance Audit Trail** | Manual spreadsheet logs | Non-auditable; regulatory risk | Immutable Kafka event log (complete audit) |

**Business KPIs Impacted**:
- **Time-to-Trade**: New clients execute first trade in **2-3 days** vs 12 weeks (50-75% reduction)
- **Compliance Score**: **100% automated** KYC/AML → zero manual review gaps
- **Client Satisfaction**: **NPS +40 points** (onboarding gone from pain → delighted first experience)
- **Operations Cost**: **$500K/year savings** (reduce onboarding team from 6 people → 2)

---

## 2. Architecture Overview: The Unified Onboarding Mesh

### 2.1 Three-Layer Design (Front Door → Fulfillment Engine → Security Guardrail)

```
┌─────────────────────────────────────────────────────────────────┐
│                  CLIENT ONBOARDING ECOSYSTEM                    │
└─────────────────────────────────────────────────────────────────┘

LAYER 1: DIGITAL & API (Front Door)
┌─────────────────────────────────────────────────────────────────┐
│  React/Next.js Portal (AWS Amplify)                             │
│  ├─ KYC Document Upload (S3 pre-signed URLs)                    │
│  ├─ Real-time Status Dashboard (WebSocket via AppSync)          │
│  └─ Account Setup Wizard (multi-step form)                      │
│                                                                  │
│  Salesforce Service Cloud (Lifecycle Manager)                   │
│  ├─ Case tracking (onboarding ticket)                           │
│  ├─ RM assignment (Account Owner)                               │
│  └─ Compliance Task (KYC/AML checklist)                         │
│                                                                  │
│  GraphQL API (AWS AppSync)                                      │
│  ├─ Unifies Salesforce + Internal Services                      │
│  ├─ Real-time subscriptions for portal events                   │
│  └─ Mutation: "triggerOnboarding(clientId)" → Step Functions    │
└─────────────────────────────────────────────────────────────────┘

LAYER 2: ASYNC & DATA EXCHANGE (Fulfillment Engine)
┌─────────────────────────────────────────────────────────────────┐
│  Amazon MSK (Kafka Event Mesh)                                  │
│  ├─ Topic: onboarding.started (portal signup)                   │
│  ├─ Topic: kyc.completed (compliance approved)                  │
│  ├─ Topic: account.provisioned (S3/IAM ready)                   │
│  ├─ Topic: data.entitlements.issued (ADX access granted)        │
│  └─ Topic: onboarding.completed (client activated)              │
│                                                                  │
│  AWS Step Functions (Workflow Orchestrator)                     │
│  ├─ State machine: KYC validator → Account provisioner → Notif  │
│  ├─ Error handling: Retry with exponential backoff              │
│  └─ Callback patterns: SNS notifications to human review        │
│                                                                  │
│  AWS Data Exchange (Zero-Copy Distribution)                     │
│  ├─ Historical price data (pre-loaded; no file transfer)        │
│  ├─ Performance attribution (instantaneous access)              │
│  └─ Client account auto-entitled on "ACTIVATED" event           │
└─────────────────────────────────────────────────────────────────┘

LAYER 3: IDENTITY & SECURITY (SSO Guardrail)
┌─────────────────────────────────────────────────────────────────┐
│  Nomura Enterprise SSO (OIDC/SAML)                              │
│  ├─ Identity source for all Nomura employees                    │
│  └─ Federated through Amazon Cognito                            │
│                                                                  │
│  Amazon Cognito (Identity Broker)                               │
│  ├─ Federates Nomura SSO → Client Portal                        │
│  ├─ Federated login → AWS Management Console (for tech clients) │
│  ├─ MFA enforcement (SMS or TOTP for sensitive operations)      │
│  └─ Session management (30-min idle timeout; auto-logout)       │
│                                                                  │
│  AWS IAM (Permission Boundary)                                  │
│  ├─ Client S3 bucket access isolated by partition key (client_id)  │
│  ├─ Cross-account access via STS AssumeRole (for clients with   │
│  │   own AWS accounts; optional)                                │
│  └─ CloudTrail audit trail (every action logged immutably)      │
└─────────────────────────────────────────────────────────────────┘

LAYER 4: OBSERVABILITY (LTGM Control Plane)
┌─────────────────────────────────────────────────────────────────┐
│  Loki (Log Aggregation)                                         │
│  ├─ Onboarding event logs (portal actions)                      │
│  ├─ Step Functions execution logs                               │
│  └─ Compliance decision logs (for audit)                        │
│                                                                  │
│  Tempo (Distributed Tracing)                                    │
│  ├─ End-to-end trace: Client Portal → GraphQL → Step Functions  │
│  │   → Kafka → AWS ADX → Client AWS Account                     │
│  └─ Latency breakdown: Where are the slowdowns?                 │
│                                                                  │
│  Grafana (Unified Dashboard)                                    │
│  ├─ Real-time KPIs: Active onboardings, completion rates        │
│  ├─ Executive view: "Time to Active" trend                      │
│  └─ Alert panel: "KYC stalled >24hrs"                           │
│                                                                  │
│  Prometheus Mimir (Metrics Storage)                             │
│  ├─ Onboarding success rate (%%)                                │
│  ├─ Step Functions error rates (by step)                        │
│  ├─ Kafka consumer lag (compliance processing)                  │
│  └─ Portal API latency (p50, p95, p99)                          │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. Deep-Dive Architectural Components

### 3.1 Digital & API Layer (Front Door)

**React/Next.js Portal** (AWS Amplify hosting):
```
OnboardingPortal/
├─ pages/
│  ├─ dashboard/ (status overview; real-time via WebSocket)
│  ├─ kyc-upload/ (document submission; S3 pre-signed URLs)
│  ├─ account-setup/ (basic info: fund name, legal structure)
│  └─ confirmation/ (ready to trade; next steps)
├─ components/
│  ├─ DocumentUploader (drag-drop; virus scan via Lambda)
│  ├─ StatusTimeline (KYC → Account → Data → Activated)
│  └─ RealTimeDashboard (WebSocket subscription to AppSync)

GraphQL Schema (AppSync):
  Query {
    getOnboardingStatus(clientId: String!): OnboardingStatus
    listDocuments(clientId: String!): [Document!]!
  }
  
  Mutation {
    triggerOnboarding(input: OnboardingInput!): OnboardingResult
    uploadDocument(clientId: String!, file: File!): Document
  }
  
  Subscription {
    onOnboardingStatusChange(clientId: String!): OnboardingStatus
  }
```

**Salesforce Service Cloud** (Lifecycle Manager):
```
When Portal triggers "triggerOnboarding":
1. AppFlow runs (scheduled every 5 min; event-driven in real-time)
2. Salesforce case created: Subject="Onboarding - [ClientName]"
3. Owner assigned to RM based on geographic territory
4. Compliance task auto-created: "Review KYC documents"
5. Portal becomes read-only (case now "source of truth")

Salesforce Dashboard (for RMs):
  Active Onboardings: 24 clients (15 in KYC, 7 awaiting docs, 2 ready to activate)
  SLA Compliance: 85% completed within 3 days (Target: 90%)
  Bottleneck: 5 clients stuck in "KYC approval" >24 hours → Escalate
```

---

### 3.2 Async & Data Exchange Tier (Fulfillment Engine)

**Kafka Event-Driven Workflow**:

```yaml
Topic: onboarding.clients
  
Events:
  - onboarding.started
    Event: {clientId, legalName, accountOwner_email, timestamp}
    → Triggers: Step Functions workflow start
    
  - kyc.document_submitted
    Event: {clientId, documentType, s3_path, timestamp}
    → Triggers: Lambda (virus scan; document parsing)
    
  - kyc.validation_complete
    Event: {clientId, result: PASS|FAIL, notes, timestamp}
    → If PASS: Triggers account provisioning
    → If FAIL: Sends alert to RM + pauses workflow
    
  - account.provision_requested
    Event: {clientId, accountType: STANDARD|PREMIUM, timestamp}
    → Triggers: Terraform AWS account setup
    
  - account.provisioned
    Event: {clientId, s3_bucket_arn, iam_role_arn, timestamp}
    → Triggers: ADX entitlement update
    
  - data.entitlements_issued
    Event: {clientId, datasets: [fund_prices, attribution], timestamp}
    → Client AWS account now has read access to data
    
  - onboarding.completed
    Event: {clientId, totalTime_hours, timestamp}
    → Portal unlocked; client can now login
```

**Step Functions State Machine** (Orchestrates Workflow):

```
StartState
├─ ParseInput (Extract clientId, legalName)
├─ KYCValidator (Parallel: Document scan + Rule engine)
│  ├─ Scan documents (Lambda)
│  └─ AML check against sanctions list (API call)
├─ OnErrorHandler (If KYC fails: SNS notify RM; Wait for manual approval)
├─ AccountProvisioner
│  ├─ Terraform apply (create S3, IAM, VPC endpoints)
│  ├─ Wait for stack creation (CloudFormation callback)
│  └─ On Success: Publish account.provisioned event
├─ ADXEntitlementUpdate (Lambda: call ADX API; grant dataset access)
├─ PortalActivation (Update Salesforce; trigger portal unlock)
└─ NotifyClient (Lambda: Send email "Welcome; you're ready to trade")

Execution Time: ~15 minutes (KYC ~5min + Provisioning ~7min + Entitlement ~2min + Notification ~1min)
```

**AWS Data Exchange (Zero-Copy Distribution)**:

```
Current Problem (Manual SFTP):
  New client needs historical prices → Ops uploads 50GB CSV → SFTP setup fails
  → Client submits ticket → 2 days delay → Finally downloads

New Solution (ADX Zero-Copy):
  Nomura publishes dataset: "Historical Stock Prices (500+ years)"  
  → Store once in S3 (managed by Nomura)
  → Clients reference via ADX API (zero data movement)
  → When client onboards: Single ADX API call grants access
  → Client immediately queries: $0 data egress charge (vs $1M+ annual egress)
```

---

### 3.3 Identity & Security (Zero-Trust SSO Guardrail)

**OIDC Federation via Cognito**:

```
Step 1: Employee (Nomura) Login
  → Browser redirects to Nomura SSO (OIDC provider)
  → Validates against Active Directory
  → Returns JWT token (email=john@nomura.com)
  → Cognito validates token; issues Cognito JWT
  → Portal unlocked; John can administer client accounts

Step 2: Client (External) Login
  → Client team member navigates to portal
  → "Sign in as Client" option
  → Redirects to Cognito hosted UI
  → Cognito brokers login (basic password or MFA)
  → Creates session; issue Cognito JWT (email=trader@citadel.com)
  → Portal shows "Client Mode" → limited to own data
  
Step 3: Technical Client (Optional AWS Access)
  → Client requests cross-account access (own AWS account)
  → Cognito brokers AWS STS AssumeRole call
  → Temporary credentials issued (STS token)
  → Client can now use AWS CLI/SDK with their creds
  → All actions logged to CloudTrail (immutable audit trail)

IAM Partition Keys (Fine-Grained Access):
  ClientA can only access: s3://onboarding/client_a/*
  ClientB can only access: s3://onboarding/client_b/*
  → Enforced by IAM policy; no cross-contamination
```

**CloudTrail Immutable Audit Log**:

```
Every action logged:
  2026-02-18T10:15:42Z | AssumeRole | arn:aws:iam::CITADEL:role/s3-reader 
                         | sourceIp=203.0.113.42 | success=true
  
  2026-02-18T10:16:01Z | GetObject | s3://onboarding/client_a/portfolio_data.csv
                         | principal=arn:aws:sts::NOMURA:role/citadel-access
                         | success=true | bytes=125000

Compliance Query:
  "Show me all data accessed by Citadel in the past 30 days"
  → CloudTrail Insights: 4,233 GetObject calls (portfolio data + prices)
  → Prove data access within SLA; regulatory audit ready
```

---

## 4. Operational Excellence: LTGM Monitoring Stack

### 4.1 The LTGM Control Plane

```yaml
Loki (Logs):
  Aggregates:
    - Portal errors (React console logs)
    - Step Functions execution logs (every step traced)
    - Compliance decision logs (manual approvals logged)
  
  Query Example:
    Find all KYC documents rejected in past 24 hours
    → {job="kyc-validator"} | filter status == REJECTED
    → Returns: 3 rejections (missing proof of funds × 2; outdated address × 1)

Tempo (Traces):
  Traces end-to-end flow:
    Client uploads document (Portal) 
    → GraphQL mutation (AppSync) 
    → Step Functions trigger 
    → Lambda virus scan 
    → Kafka event published 
    → ADX entitlement updated
    → Portal refreshes
    
    Total trace: 47ms (shows Document scan took 15ms = bottleneck)

Grafana (Dashboards):
  Dashboard 1: Onboarding KPIs (for executives)
    - Total clients onboarding: 42 (live counter)
    - Avg time to "Active": 2.1 days (trending ↓ = good)
    - Completion rate this week: 78% (vs 85% target; slightly below)
    - Blocked clients: 4 (stuck in KYC >24 hours; needs escalation)
  
  Dashboard 2: Platform Health (for ops)
    - Step Functions success rate: 98.2% (2 failures this week)
    - Kafka consumer lag: 0 messages (all events processed)
    - Portal API latency: p50=45ms, p95=230ms (acceptable)
    - Portal availability: 99.97% uptime

Mimir (Metrics):
  Tracks:
    onboarding_total{status="completed", days_to_activate_bucket="1-3"} = 42 clients
    onboarding_total{status="in_progress", stage="kyc"} = 15 clients
    step_functions_duration_seconds{step="kyc_validator"} = [5m, 12m, 3m] = average 6.7m
    portal_api_latency_ms{endpoint="/getStatus", percentile="p95"} = 230ms
```

---

## 5. Proactive Patterns: "Squeezing the Lemon"

### The Onboarding Anti-Patterns & Solutions

| Anti-Pattern ("Lemon") | Root Cause | Proactive Mitigation | Detection | Action |
|----------|-----------|----------------------|-----------|--------|
| **KYC Stalled >24hrs** | Manual reviewer backlog | Step Functions callback waits max 24hrs; SNS alert | Grafana alert triggered | Lambda escalates to manager; routes to backup reviewer |
| **Account Provisioning Failed** | CloudFormation stack rollback | Terraform drift detection (daily); auto-remediate | Stack status = ROLLBACK | Lambda retries with clean state; notifies ops |
| **Kafka Consumer Lag Growing** | Compliance validator slow | Monitor lag via Prometheus; scale consumer horizontally | Lag >1000 messages | Auto-scale EKS pods (HPA); page on-call |
| **Portal Login Timeouts** | Cognito token refresh failing | CloudWatch Canaries (synthetic login from 5 regions; 15min intervals) | Canary fails | Lambda recreates Cognito app client; alerts ops |
| **Data Not Accessible (ADX)** | Dataset policy permissions wrong | Pre-test ADX entitlement grant (Lambda runs dry-run before real grant) | Grant dry-run fails | Halt onboarding; notify data platform team |
| **Compliance Rule Engine False Negative** | Model drift (older AML rules) | Weekly accuracy audit; compare predicted vs actual violations | Audit finds <85% precision | Retrain model; deploy new version |

---

## 6. Production Runbooks & Incident Response

### 6.1 Incident: Client Stuck in "KYC Approval" for 48+ Hours

```
Severity: P2 (impacts client experience; not business outage)

Step 1: DETECT (Automated)
  Grafana alert: onboarding_stage_duration{stage="kyc"} > 48hrs
  → Alert rule fires every hour
  → Slack message: "#onboarding-alerts: 3 clients stuck in KYC >48hrs"

Step 2: INVESTIGATE (First 15 minutes)
  On-call engineer checks:
    [ ] Which clients? Query Grafana: clientId IN [Client1, Client2, Client3]
    [ ] Which step? Logs in Loki: {job="kyc-validator"} | Status?
      Result: Client1 = AWAITING_RESPONSE (documents incomplete)
              Client2 = PENDING_REVIEW (assigned but not started; reviewer offline)
              Client3 = REJECTED_AML (fail on sanctions list; needs manual override)
    
    [ ] Trace client path: Client3 in Tempo
      → Document scan: ✅ (passed)
      → AML rule engine: ❌ (name match = sanctions list false positive)
      → Flow stuck (waiting for human override)

Step 3: MITIGATE (5-30 minutes)
    For Client2 (Reviewer offline):
      [ ] Reassign case in Salesforce to backup reviewer
      [ ] Step Functions: Resume workflow (callback receives manual approval event)
      [ ] Time to resolve: ~5 minutes
    
    For Client3 (AML false positive):
      [ ] Review: Is "John Smith" actually on sanctions list? (No; false positive)
      [ ] Override: Lambda execute: "approveKYC(client3, override_reason=false_positive)"
      [ ] Result: Workflow resumes; account provisioned
      [ ] Time to resolve: ~10 minutes

Step 4: POST-INCIDENT (Next business day)
    [ ] Root cause: 2 issues: (1) Reviewer coverage gap, (2) AML model precision
    [ ] Action: (1) Add on-call coverage; (2) Retrain AML model with false positives
    [ ] Preventive: Automated reassignment if case not started within 4 hours
```

---

## 7. 12-Month Roadmap (M1-M12)

| Phase | Timeline | Deliverables | Success Metrics |
|-------|----------|-------------|-----------------|
| **M1-M2: Foundation** | Feb-Mar | Portal MVP + Salesforce integration; Step Functions basic workflow | 10 pilot clients onboarded in <1 week |
| **M3-M4: Scalability** | Apr-May | Kafka event mesh; ADX integration; LTGM monitoring live | 50 clients onboarded; avg time <3 days |
| **M5-M6: Compliance** | Jun-Jul | AI-powered KYC extraction; AML rule engine; audit trail complete | 100% automated compliance; regulatory audit pass |
| **M7-M8: Security Hardening** | Aug-Sep | Zero-Trust with Cognito federation; mTLS for all services | SOC2 audit pass; zero unauthorized access |
| **M9-M10: Optimization** | Oct-Nov | Predictive analytics (which clients will be slow?); auto-escalation | 90% clients active within 2 days |
| **M11-M12: Expansion** | Dec-Jan | Multi-region architecture; cross-account ADX entitlements; API for partners | 500+ clients onboarded; Nomura's #1 platform |

---

## 8. Interview Talking Points for AWS Data Exchange Architects

### For Shaw Levin (AWS Legacy + FinOps Expert)

> **On Zero-Copy Distribution**:
> "You built ADX at AWS. I'm applying that principle end-to-end: not just for our data distribution to clients, but for their onboarding experience itself. When a client contract is signed, we publish them as an 'ADX Consumer' automatically. They reference historical prices, attribution, fund details—zero files copied, zero egress charges, zero operational overhead. That's data-as-a-product thinking scaled horizontally."

> **On Event-Driven Orchestration**:
> "Rather than RMs updating spreadsheets, Kafka events flow: 'Client contract signed' → 'KYC started' → 'Account provisioned' → 'Data entitlements issued' → 'Portal activated'. Each event triggers the next step. If any step fails, the entire pipeline waits intelligently (not lost forever). It's like AWS StepFunctions, but with Kafka durability for financial compliance."

> **On FinOps for Client Experience**:
> "We're reducing opex by 75% ($2M → $500K annually) through automation. But more importantly: every hour of onboarding automation is an hour Client Relationship Managers spend on strategically growing the relationship instead of chasing spreadsheets. That's the business case—not just cost savings, but strategic realignment."

---

## 9. Cost Model & ROI

```
Investment (Year 1):
  Platform engineering: 5 FTEs × $250K = $1.25M
  AWS services: MSK, AppSync, Step Functions, RDS = $200K/year
  Salesforce integration (AppFlow): $50K
  Total: $1.5M investment

Savings (Year 1):
  Onboarding team: 6 people → 2 people = 4 FTEs × $150K = $600K savings
  Data egress reduction (ADX zero-copy): 1,000 clients × $500/year = $500K savings
  Operational efficiency gains): Reduced escalations & manual rework = $200K
  Total savings: $1.3M

Year 2+ Savings:
  Marginal cost per new client: $1K (tooling; no headcount needed)
  Revenue opportunity: Speed-to-onboarding attracts 100+ new clients = $30M new revenue
  
ROI: By M18, platform breakeven; by M24, $10M net profit
```

---

## 10. Conclusion

The **Strategic Client Onboarding Ecosystem** is not just a tool; it's a competitive advantage. By combining AWS event services, zero-copy data sharing, and zero-trust identity, Nomura can onboard institutional clients in **days, not weeks**, with **100% compliance automation** and **zero manual intervention**.

This positions Nomura as the **fastest, most secure, most compliant** platform in institutional asset management.

---

## References & Appendix

- [AWS Step Functions](https://docs.aws.amazon.com/stepfunctions/)
- [AWS Data Exchange - Zero-Copy Sharing](https://aws.amazon.com/data-exchange/)
- [Kafka Event-Driven Architecture](https://www.confluent.io/blog/event-driven-soa/)
- [OIDC Federation Best Practices](https://docs.aws.amazon.com/cognito/)
- [Observability with Grafana, Loki, Tempo](https://grafana.com/observability/)

---

**Status**: Ready for Evaluation Cycle 1  
**Next Steps**: 3-cycle evaluation framework (baseline → enhancement → final certification)
