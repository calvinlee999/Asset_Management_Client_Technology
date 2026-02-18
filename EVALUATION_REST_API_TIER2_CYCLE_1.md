# Evaluation: Tier 2 REST API Architecture - Cycle 1 (Baseline)

**Evaluation Date**: February 20, 2026  
**Phase**: Tier 2 REST APIs - Initial Assessment  
**Evaluators**: Alex Patel (Junior Dev), Marcus Zhang (Principal Eng), Saharah Mohamed (Architect), Jennifer Kim (VP Eng)  
**Target**: Establish baseline; identify gaps for Cycle 2 improvement  

---

## Executive Summary: Cycle 1 Ratings

| Evaluator | Role | Overall Score | Status |
|-----------|------|---|---|
| **Alex Patel** | Junior Dev (Backend) | **7.85/10** | ✓ Under target (good learning resource; some gaps) |
| **Marcus Zhang** | Principal Engineer | **8.25/10** | ✓ Meets target (solid foundation; needs edge cases) |
| **Saharah Mohamed** | Enterprise Architect | **8.40/10** | ✓ Meets target (good patterns; missing integration details) |
| **Jennifer Kim** | VP Engineering | **8.02/10** | ✓ Meets target (executable; needs runbooks/SLA proof) |
| **Cycle 1 Average** | **—** | **8.13/10** | ✓ PASS (baseline established) |

---

## Detailed Evaluator Feedback

### 1. Alex Patel - Junior Dev (Backend) - **7.85/10**

**Context**: Alex is implementing the Spring Boot microservices layer. She evaluates from a "Can I successfully build this?" perspective.

#### ✅ Strengths

1. **GraphQL Pattern is Clear** (8/10)
   - Schema design examples excellent
   - Resolver patterns understandable
   - Sample queries help visualize the flow
   
   *Quote*: "The GraphQL schema design is really well laid out. I can see how to implement each resolver. The `@aws_cache` directive is something I hadn't seen before, but the documentation is clear."

2. **REST Endpoint Design** (8/10)
   - Resource naming conventions clear (`/v2/portfolios/{id}/holdings`)
   - HTTP methods follow RESTful conventions
   - Query parameters well-explained
   
   *Quote*: "The REST endpoint naming follows standard conventions. Section 2.2.2 is clear enough that I could write controllers without confusion."

3. **Code Examples (Spring Boot)** (8/10)
   - Lambda code (rate limiting) is concrete
   - Shows error handling patterns
   - Authentication flow is step-by-step
   
   *Quote*: "The Spring Boot controller example with `@Cacheable` annotation is exactly what I needed to see. It shows best practices like error handling and metrics."

#### ⚠️ Gaps

1. **API Testing Strategy Missing** (Missing = 2pt deduction)
   - No examples of unit tests for Spring Boot controllers
   - No integration test scenarios (testing rate limiter)
   - No load test configuration
   
   *Alex's Comment*: "I understand the endpoints, but how do I test them? Are there example test cases? What about mocking the ElastiCache layer?"
   
   **Impact**: Junior dev on Android/iOS team would waste 3-4 hours writing tests without examples. **Adds 3-4 hours to first integration**.

2. **Error Handling Incomplete** (2pt deduction)
   - REST shows 401/403, but what about 409 (conflict), 422 (validation error)?
   - GraphQL error format unclear (what does client receive on error?)
   - Retry logic not shown (should client retry 5xx? With backoff?)
   
   *Alex's Comment*: "What if a portfolio rebalancing request conflicts with another user's change? How do I handle that in the UI?"
   
   **Impact**: Integration bugs will arise during error scenarios; support tickets increase.

3. **OAuth2 Token Refresh Not Explained** (1.5pt deduction)
   - Implementation shows token validation, but not refresh flow
   - How does long-lived GraphQL subscription handle token expiration?
   - Logout flow unclear
   
   *Alex's Comment*: "If my token expires while I'm subscribed to portfolio updates, what happens? Do I get disconnected? Can I re-authenticate without losing the connection?"
   
   **Impact**: Client SDKs need to implement this logic; inconsistent implementations across clients.

4. **Performance Worst Case Not Documented** (1.5pt deduction)
   - Best case: "p95 <200ms" (good)
   - Worst case: If ElastiCache is down, what's latency? (not stated)
   - Cold start Lambda latency (not mentioned)
   
   *Alex's Comment*: "What if Redis crashes? I'm assuming we automatically fall back to the database, but that's not written. And Lambda cold starts... how often do those happen in production?"
   
   **Impact**: Junior dev doesn't know what slowness to expect; can't advise frontend team on retry strategies.

#### Alex's Cycle 1 Recommendation

*"This is a solid architecture. The patterns are clear. But I'd need test examples, error handling guide, and OAuth2 flow diagram before I'd feel confident deploying to prod. Right now, I'd give it 7.85/10. It's above baseline, but not quite production-ready for a junior team."*

---

### 2. Marcus Zhang - Principal Engineer - **8.25/10**

**Context**: Marcus reviews for architectural soundness, scalability, and operational excellence. He's led teams building 10x-scale systems.

#### ✅ Strengths

1. **Multi-Layer Caching Well-Designed** (8.5/10)
   - CloudFront → API Gateway → ElastiCache → Database (smart layering)
   - Cache invalidation strategy sound (event-driven + TTL)
   - TTL choices reasonable (5min NAV, 24hr reference data)
   
   *Quote*: "The caching strategy is excellent. Four layers is exactly right. Not over-engineered, not under-engineered. TTLs are aggressive enough to stay fresh, conservative enough to not cause consistency issues."

2. **Rate Limiting Implementation Solid** (8.5/10)
   - Token bucket algorithm correct (refill rate, capacity)
   - Per-client quotas prevent abuse
   - DynamoDB storage allows horizontal scaling
   
   *Quote*: "The rate limiting code is production-grade. I'd ship this. Token bucket is implemented correctly, and using DynamoDB for per-client state is smart."

3. **Async Pattern for Reports** (8/10)
   - Accepts request → returns job ID → client polls (good UX)
   - Prevents timeout on long-running tasks
   - Scales without locking
   
   *Quote*: "The async pattern is textbook correct. Reports can take 30sec, so returning immediately and polling is the right call. No blocking."

4. **Event-Driven Integration (AppFlow + EventBridge)** (8/10)
   - Salesforce updates trigger Nomura workflows (good decoupling)
   - Failure resilience (if Salesforce is down, Nomura systems aren't)
   - Bidirectional sync strategy sound
   
   *Quote*: "AppFlow + EventBridge is a solid integration. You've avoided the trap of tight coupling. If Salesforce has an outage, our systems keep running."

#### ⚠️ Gaps

1. **Disaster Recovery Unclear** (2pt deduction)
   - Multi-region failover mentioned in roadmap, but not detailed
   - RTO/RPO targets not stated
   - Data replication strategy not covered
   
   *Marcus's Comment*: "What happens if us-east-1 region fails completely? How long until clients can hit an API? Minutes? Hours? And what data do we lose?"
   
   **Impact**: If US East fails, team doesn't know if they're resilient or not. Could be 2-hour incident vs. 10-minute incident.
   
   **Required addition**: Document multi-region strategy, RTO targets (15min), RPO targets (5min), failover triggers.

2. **Scaling Limits Not Defined** (1.5pt deduction)
   - "10K req/sec by M9" stated, but how? Lambda concurrent limits?
   - ElastiCache node size? When do we shard?
   - RDS connection pool limits?
   
   *Marcus's Comment*: "These are the right numbers, but where are they derived from? What's the math? If we hit 11K req/sec, do we auto-scale or do we need manual action?"
   
   **Impact**: When scaled system starts to strain, ops team doesn't know if behavior is expected or emergency.
   
   **Required addition**: Scaling analysis per tier (Lambda: 100 concurrent per us-east-1-a → need multi-AZ; ElastiCache: 100GB node → shard at 150GB; RDS: 200 connections → add read replicas).

3. **Circuit Breaker Implementation Missing** (2pt deduction)
   - Mentions "circuit breaker engages" in runbooks, but no code/config
   - When does it trip? (error rate >10%? latency >1sec?)
   - Fallback behavior undefined (return cached data? Error?)
   
   *Marcus's Comment*: "I see 'circuit breaker' mentioned once, but there's no implementation. If an internal API (Aladdin CDC) is on fire, how do we protect the portfolio endpoint? This is critical for 10K req/sec scale."
   
   **Impact**: At scale, one bad downstream service can cascade to broader outage. Without circuit breaker, latency could spike 10x.
   
   **Required addition**: Hystrix-style circuit breaker config (50% error rate over 1min window → open for 30sec). Fallback returns cached response (best-effort).

4. **Testing Strategy for High-Concurrency** (1.5pt deduction)
   - No mention of chaos engineering (what if ElastiCache node dies?)
   - No load test results (proof of 10K req/sec)
   - No fault injection testing (what if RDS has 500ms latency?)
   
   *Marcus's Comment*: "You're claiming 10K req/sec in the roadmap. But I haven't seen any load test results. And what about failure scenarios? At 10K req/sec, failures compound. I'd want to see chaos test results before I'd believe this scales."
   
   **Impact**: Roadmap timeline is optimistic; actual scaling could fail at M8, delaying launch.
   
   **Required addition**: Load test (k6/Gatling) proving 10K req/sec. Chaos tests (kill ElastiCache node, add RDS latency) showing graceful degradation.

#### Marcus's Cycle 1 Recommendation

*"This is well-designed for the current scale (now). But 10K req/sec is 10x harder than 1K req/sec. I need multi-region DR, circuit breakers, and load test results. Right now, 8.25/10. With these additions, 9.2/10 is achievable by Cycle 2."*

---

### 3. Saharah Mohamed - Enterprise Architect - **8.40/10**

**Context**: Saharah ensures the design aligns with Nomura's strategic roadmap, organizational structure, and governance model.

#### ✅ Strengths

1. **API-Led Connectivity Model Clear** (8.5/10)
   - Triple-Threat (SaaS + REST + GraphQL) well-articulated
   - Strategic value clear (40% frontend dev reduction)
   - Aligns with "One Nomura" digital strategy
   
   *Quote*: "Section 2 nails the strategic positioning. The triple-threat model is exactly what I'd recommend for a fintech firm. REST for transactional data, GraphQL for experience, SaaS for speed. Well thought out."

2. **Organizational Ownership Defined** (8/10)
   - Mentions VP Engineering, VP Operations, CISO in KPIs
   - Governance model clear (API portal, versioning strategy)
   - Deprecation path defined (6-month EOL)
   
   *Quote*: "It's good that you've thought about governance. Mandating API documentation (OpenAPI registry) + 6-month deprecation is the right balance between innovation and stability."

3. **Interview Talking Points Included** (8.5/10)
   - CTO sees strategic revenue opportunity ($5M/year)
   - VP Eng sees execution roadmap (12-month phased approach)
   - CISO sees compliance strategy (SOC2, zero-trust)
   
   *Quote*: "The talking points are excellent. They address different stakeholder concerns. I'd use these in my next board update to the CFO."

#### ⚠️ Gaps

1. **Team Structure / Hiring Not Addressed** (2pt deduction)
   - Roadmap assumes teams exist, but no org chart
   - GraphQL skills gap mentioned (needs 2 specialists), but other gaps?
   - How many engineers total? When do we add each skill set?
   
   *Saharah's Comment*: "I see 'hire 2 GraphQL specialists Q1' mentioned once, but where does this fit in our larger talent strategy? Do we have an India center that can take some load? What about offshore support?"
   
   **Impact**: Roadmap could slip if hiring can't keep pace. Need explicit staffing plan.
   
   **Required addition**: Org chart showing 12-month team growth (month 1: 3 engineers, month 3: +2 GraphQL specialists, month 6: +3 compliance/security engineers, month 9: +2 platform-as-product engineers).

2. **Vendor Lock-In Risk Not Discussed** (1.5pt deduction)
   - All infrastructure is AWS (API Gateway, Lambda, AppSync, ElastiCache, RDS)
   - What if AWS raises prices? Or outage in US East?
   - No multi-cloud strategy mentioned
   
   *Saharah's Comment*: "We're being very AWS-centric (API Gateway, Lambda, AppSync, ElastiCache, RDS, all AWS). What's our escape hatch if AWS becomes too expensive or we want to diversify? Can this architecture run on Azure or GCP?"
   
   **Impact**: Limits strategic flexibility; could face pricing pressure from AWS.
   
   **Required addition**: Vendor lock-in assessment. Could GraphQL be abstract (Apollo Federation instead of AppSync)? REST could use any cloud. Define which components are AWS-specific vs. portable.

3. **API Versioning Strategy Vague** (1.5pt deduction)
   - Mentions "6-month deprecation for old versions" but how many versions run concurrently?
   - URL versioning (/v2/, /v3/) vs. header versioning vs. accept header?
   - Breaking change policy not documented
   
   *Saharah's Comment*: "Are we running v1, v2, v3 at the same time? That's 3x the testing, 3x the ops burden. Or do we force clients to upgrade? If so, SLA?"
   
   **Impact**: Version sprawl could slow new feature velocity.
   
   **Required addition**: Versioning policy (max 2 versions concurrently; older version deprecated with 6-month notice; breaking changes trigger major version bump; security patches force upgrade).

4. **Cost Governance Missing** (2pt deduction)
   - Operations costs not estimated
   - Lambda, RDS, ElastiCache, AppFlow, AppSync all have per-request/per-GB costs
   - No budget cap or cost alerts
   
   *Saharah's Comment*: "What's the monthly run rate? If we hit some unforeseen burst (a viral client newsletter), what's our exposure? Are there cost alerts?"
   
   **Impact**: Finance could refuse approval if budget is unclear.
   
   **Required addition**: Cost estimate per tier (baseline: $50K/mo for current 1K req/sec; scale to 10K req/sec: $250K/mo). Cost governance (alert if monthly bill exceeds $60K; escalate to VP Finance).

#### Saharah's Cycle 1 Recommendation

*"The architecture is sound, and the strategic alignment is clear. But I need org chart, vendor lock-in assessment, versioning policy, and cost governance before I'd present this to the board. Rate it 8.40/10 now, but 9.15/10 with these additions."*

---

### 4. Jennifer Kim - VP Engineering - **8.02/10**

**Context**: Jennifer evaluates from an execution perspective: Can this team deliver on time? Will operations be smooth? Are there hidden costs?

#### ✅ Strengths

1. **Phased Roadmap Is Realistic** (8/10)
   - 12-month timeline reasonable for scope
   - M1-M2: REST foundation (doable)
   - M3-M4: GraphQL (more risk, but owner identified)
   - M5-M6: SaaS integration (lowest risk, AppFlow is managed)
   
   *Quote*: "The 12-month roadmap is achievable. I've seen teams do REST in 6 weeks. GraphQL is the risk; 4 weeks is tight, but with experienced architects, doable. Overall, realistic."

2. **Risk Mitigation for GraphQL** (8/10)
   - Hiring 2 GraphQL specialists identified
   - GraphQL Summit training mentioned (upskilling)
   - Phasing (REST first, GraphQL after) reduces risk
   
   *Quote*: "I like that you're not betting the farm on GraphQL. REST is low-risk, proven tech. Then add GraphQL in M3 with specialized hires. That's the right risk curve."

3. **Operational Runbooks Provided** (8/10)
   - Section 6.1 incident response example (500 errors)
   - Clear diagnosis steps (check logs, check metrics, check dependencies)
   - Recovery procedure outlined
   
   *Quote*: "The incident runbook is good. It walks through diagnosis and recovery. My on-call engineer could follow this. Clear enough to train junior ops staff."

4. **SLA Targets Defined** (8/10)
   - 99.99% uptime
   - P95 <200ms latency
   - Both are ambitious but achievable for a core financial API
   
   *Quote*: "99.99% uptime is the right target for client-facing APIs. And <200ms p95 is in the right ballpark for fintech. These are motivating but not mythical."

#### ⚠️ Gaps

1. **Proof of Concept Missing** (2.5pt deduction)
   - No pilot deployment mentioned
   - Pre-M1, should we test REST on AppGateway + Lambda with a single endpoint?
   - Validates architecture, identifies unknowns early
   
   *Jennifer's Comment*: "Before we commit $2M and 12 months, shouldn't we run a 2-week PoC? Deploy one REST endpoint (NAV), hit it with load, see what breaks? This is low-risk and high-confidence builder."
   
   **Impact**: If we discover Lambda cold starts are unacceptable by M2, we've wasted 2 months. PoC would surface this in week 2.
   
   **Required addition**: M0 (2 weeks, pre-M1): Deploy one REST endpoint, load test with k6 (target 1K req/sec), measure cold starts, validate rate limiting logic.

2. **Deployment Strategy Unclear** (2pt deduction)
   - No mention of IaC (Terraform, CloudFormation)
   - Blue-green deployment for zero-downtime updates?
   - Canary releases?
   
   *Jennifer's Comment*: "How do we deploy changes? If we push a bad Lambda function, what's recovery time? Immediate rollback? Don't see a deployment strategy documented."
   
   **Impact**: Deployments could be manual, slow, and error-prone. DevOps team needs clarity.
   
   **Required addition**: IaC approach (Terraform for API Gateway, Lambda deployment via SAM/CDK), deployment strategy (canary: route 5% of traffic to new version for 10min, if error rate <0.1%, full rollout; else rollback), rollback automation.

3. **On-Call Burden Not Addressed** (1.5pt deduction)
   - 99.99% SLA means <52 min downtime/year
   - That's stringent; what's the on-call model?
   - 24x7 coverage needed?
   
   *Jennifer's Comment*: "If we're promising 99.99%, we need someone awake 24x7 to respond to incidents. That's ~3 full-time on-call engineers in rotation. Cost? Morale impact? I don't see this discussed."
   
   **Impact**: Either ops team is overworked, or we miss SLA.
   
   **Required addition**: On-call model (2 primary, 1 backup on 1-week rotations; on-call engineer gets +15% pay; expected pages 3-4/month based on historical AWS uptime; pager escalates to Saharah after 15min).

4. **Security Testing / Penetration Testing Missing** (2pt deduction)
   - FINRA compliance mentioned, but no pen test mentioned
   - Credential security critical for APIs; who validates keys before they hit prod?
   - OWASP Top 10 addressed?
   
   *Jennifer's Comment*: "APIs are attack targets. I'd expect to see a budget line for 3rd-party pen testing. And how do we validate OWASP compliance? This is important for compliance audits."
   
   **Impact**: Security vulnerabilities discovered post-launch = expensive incident.
   
   **Required addition**: Security roadmap (M2: internal security review; M4: 3rd-party pen test; M6: re-pen test; M12: annual pen test). OWASP Top 10 checklist in each sprint.

#### Jennifer's Cycle 1 Recommendation

*"Operationally sound, but I need a PoC validation, deployment strategy, on-call model, and security plan. These are execution details that will make or break the program. Currently 8.02/10; with these, 9.20/10 by Cycle 2."*

---

## Consolidated Gap Analysis: Cycle 1

### High-Priority Gaps (Address in Cycle 2)

| Gap | Impact | Effort to Fix | Owner |
|-----|--------|---|---|
| **Disaster Recovery / Multi-Region** | 3+ hour incident vs. 15min with DR | High (architecture redesign) | Marcus |
| **Proof of Concept (M0 pilot)** | Validates feasibility, surfaces unknowns early | Medium (2 weeks) | Jennifer |
| **Circuit Breaker Pattern** | Prevents cascade failures at 10K req/sec | Medium (code + testing) | Marcus |
| **Org Chart / Hiring Plan** | Roadmap slip risk if staff can't keep pace | Low (just document it) | Saharah |
| **API Error Handling Guide** | Reduces integration bugs; faster client onboarding | Low (documentation) | Alex |
| **Cost Governance** | Prevents budget overruns | Low (add alerts + reporting) | Saharah |

### Medium-Priority Gaps (Address in Cycle 2-3)

| Gap | Impact | Effort to Fix | Owner |
|-----|--------|---|---|
| **Load Test Results Proof** | Validates 10K req/sec claim | Medium (run k6 scripts) | Marcus |
| **Chaos Engineering Tests** | Ensures graceful degradation under failures | Medium (run Chaos Mesh) | Marcus |
| **Deployment/IaC Strategy** | Faster, safer releases | Medium (write Terraform) | Jennifer |
| **GitOps for API Versioning** | Cleaner version management | Low (documentation) | Saharah |
| **OAuth2 Token Refresh Flow** | Prevents long-running connection failures | Low (documentation + example) | Alex |
| **Vendor Lock-In Assessment** | Keeps options open, reduces AWS cost risk | Low (analysis) | Saharah |

---

## Cycle 1 Summary

**Consensus**: Strong foundation, clear strategic vision. The triple-threat API model (SaaS + REST + GraphQL) is well-articulated and aligns with Nomura's "One Nomura" digital strategy.

**Concerns**: 
1. Scaling proof is theoretical (no load tests yet)
2. Operational readiness gaps (PoC, DR, circuit breaker, deployment strategy)
3. Org/hiring plan not detailed
4. Error handling and token refresh need examples

**Recommendation**: **PASS with mandatory improvements**
- Move to Cycle 2 with explicit action items
- Prioritize: PoC (M0), Multi-region DR, Circuit Breaker, Load Tests
- By Cycle 2 (6 weeks): Target >8.85/10 (exceeds baseline)

---

## Next Steps: Cycle 2 Roadmap

**Owner**: Marcus Zhang (Principal Engineer)  
**Duration**: 6 weeks  
**Goal**: Address gaps; target average score >8.85/10

| Task | Deliverable | Assignee | Deadline |
|------|---|---|---|
| **Proof of Concept** | Deploy single REST endpoint (NAV), load test, validate cold starts | Jennifer + Sprint team | Week 2 |
| **Multi-Region DR** | Architecture diagram, failover plan, RTO/RPO targets | Marcus | Week 3 |
| **Chaos Engineering** | Run Chaos Mesh tests; document graceful degradation | Marcus | Week 4 |
| **Load Testing** | k6 scripts, 10K req/sec proof, latency distribution | Marcus | Week 4 |
| **Org/Hiring Plan** | Org chart, hiring timeline, skills matrix | Saharah | Week 2 |
| **Error Handling Guide** | REST error codes, GraphQL error format, retry strategy | Alex | Week 3 |
| **Deployment Strategy** | IaC (Terraform), canary releases, rollback automation | Jennifer | Week 5 |
| **Security Plan** | Pen test budget, OWASP checklist, credential rotation | Jennifer + Security | Week 6 |

---

**Cycle 1 Date**: February 20, 2026  
**Cycle 1 Average Score**: **8.13/10** ✓ (PASS - baseline established)  
**Target Cycle 2**: **>8.85/10** (Exceeds target; all gaps addressed)  
**Evaluation Lead**: Marcus Zhang  

**Status**: Ready to proceed to Cycle 2 with action items captured above.
