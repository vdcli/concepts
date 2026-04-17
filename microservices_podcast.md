# 🎙️ Small Services, Big Systems
## The Microservices Podcast — Breaking the Monolith the Right Way

> **Episode Runtime:** ~60 Minutes | **Format:** Conversational Two-Host
>
> **ALEX** — The developer: enthusiastic, asks great newbie questions, builds things hands-on.
> **PRIYA** — The architect: experienced, explains concepts with analogies, connects the dots.

---

## 📋 Speaker Guide

- **[ALEX]** — The developer voice. Asks the questions a curious engineer would ask.
- **[PRIYA]** — The architect voice. Explains with depth, analogies, and real-world context.
- `🎵` — Sound/music direction for the audio producer.
- `📝` — Production notes.

---

## Topics at a Glance

| Time | Topic | Key Concepts |
|------|-------|--------------|
| 0:00–7:00 | What Are Microservices? | Monolith vs microservices, when to split, the tradeoffs |
| 7:00–15:00 | Service Design | Bounded context, single responsibility, API contracts, data ownership |
| 15:00–24:00 | Communication Patterns | Synchronous REST/gRPC, async messaging, event-driven, choreography vs orchestration |
| 24:00–33:00 | Data Management | Database per service, sagas, eventual consistency, CQRS |
| 33:00–41:00 | Resilience Patterns | Circuit breaker, retry, timeout, bulkhead, fallback |
| 41:00–48:00 | API Gateway & Service Discovery | API Gateway patterns, client-side vs server-side discovery, service mesh |
| 48:00–54:00 | Observability | Distributed tracing, correlation IDs, structured logging, health checks |
| 54:00–58:00 | Deployment Patterns | Independent deployability, feature flags, contract testing |
| 58:00–62:00 | When Not to Use Microservices | The distributed monolith, operational complexity, starting points |

---

# SEGMENT 1: Cold Open & What Are Microservices? `(0:00 – 7:00)`

🎵 *Upbeat tech-themed intro music fades in, plays for 5 seconds, then fades under dialogue*

**[ALEX]:** Welcome to Small Services, Big Systems — where we tackle the architecture pattern that runs every major tech company, and also the one that has caused more engineering nightmares than perhaps any other. I'm Alex.

**[PRIYA]:** And I'm Priya. Microservices. The word that gets said in every architecture meeting, put on every resume, and misunderstood in roughly 80% of those contexts.

**[ALEX]:** Strong opener. So what actually are microservices?

**[PRIYA]:** Microservices is an architectural style where you build an application as a collection of small, independently deployable services, each with a single responsibility, communicating over well-defined APIs. The opposite is a monolith — one large application with all the code in one deployable unit.

**[ALEX]:** What's wrong with a monolith?

**[PRIYA]:** Nothing, initially! This is the first thing to get right. A monolith is often the correct choice when you're starting out. The problems emerge as the system grows. The deploy-everything-to-change-anything problem — one team's change forces a full redeploy. The can't-scale-parts-independently problem — you have to scale the whole thing even if only the search feature is under load. The organizational scaling problem — 50 engineers working in one codebase becomes a coordination nightmare.

**[ALEX]:** So microservices solve the scaling problem — both technical and organizational.

**[PRIYA]:** Exactly. Conway's Law says your system architecture will mirror your organizational communication structure. If you have a monolith, your teams are coupled — you need to coordinate every change. With microservices, the Orders team owns the Orders service, deploys it independently, picks their own tech stack. The teams can move fast without stepping on each other.

**[ALEX]:** Companies like Amazon and Netflix are the famous examples.

**[PRIYA]:** Amazon started as a monolith and broke it apart around 2002. Jeff Bezos issued the famous "API mandate" — every team must expose their data through service interfaces, no exceptions. That became the foundation of AWS itself. Netflix moved to microservices around 2008 after a database corruption incident brought down their entire service for three days.

**[ALEX]:** The high cost of tight coupling.

**[PRIYA]:** That incident is why Netflix open-sourced Hystrix, Eureka, and the rest of the Netflix OSS stack — the resilience and service discovery tools they built to make microservices survivable.

🎵 *Short musical sting — segment transition*

---

# SEGMENT 2: Service Design — Drawing the Boundaries `(7:00 – 15:00)`

**[ALEX]:** How do you decide what becomes a service? Where do you draw the lines?

**[PRIYA]:** This is the hardest part. The wrong service boundaries are worse than a monolith. The right framework is **Bounded Context** from Domain-Driven Design — DDD. A bounded context is a clear conceptual boundary within which a particular model of the domain is defined and applicable.

**[ALEX]:** Can you make that concrete?

**[PRIYA]:** Let's take an e-commerce system. "Product" means different things to different teams. To the catalog team, a product has a name, description, images, and categories. To the inventory team, a product has a count, warehouse location, and reorder threshold. To the pricing team, a product has a base price, discount rules, and margin targets. These are three different bounded contexts — each has its own model of "product."

**[ALEX]:** So the service boundary aligns with the business capability, not the data model.

**[PRIYA]:** Right. Not "one service per database table" — that's a common mistake. Services should be organized around business capabilities: Order Management, Product Catalog, Inventory, Pricing, User Accounts, Shipping, Notifications. Each is a cohesive business capability with clear inputs, outputs, and responsibilities.

**[ALEX]:** The "micro" in microservices — how small is small?

**[PRIYA]:** Ignore "micro." Sam Newman, who wrote the definitive book on microservices, says: as small as it needs to be, no smaller. The guideline I use: a service should be fully owned by one team (no shared ownership), deployable in under 10 minutes, understandable by a new engineer in under a day.

**[ALEX]:** Data ownership is non-negotiable?

**[PRIYA]:** This is the hardest rule and the most important. Each service owns its own database — other services cannot directly query it. If Service A needs data that Service B owns, it must go through B's API. Think of it like each department in a company having locked filing cabinets. HR can't walk into Finance and read salary data directly — Finance provides a payroll API.

**[ALEX]:** But that means you can't do a JOIN across services.

**[PRIYA]:** Correct. And this changes how you think about data. You accept eventual consistency. You denormalize. You maintain local copies of data you frequently need from other services. This is uncomfortable at first, but it's what makes services truly independent.

**[ALEX]:** API contracts — how do you keep services from breaking each other?

**[PRIYA]:** Define APIs explicitly — OpenAPI specs, Protobuf for gRPC, AsyncAPI for event schemas — and treat them as a contract. Breaking changes require versioning: `/v1/orders`, `/v2/orders`. Never remove fields from a response — only add. Additive changes are backward compatible. **Consumer-Driven Contract Testing** — tools like Pact — let consumers define what they expect from an API, and CI validates that providers don't break those expectations.

🎵 *Transition music sting*

---

# SEGMENT 3: Communication Patterns — Sync vs Async `(15:00 – 24:00)`

**[ALEX]:** Services need to talk to each other. What are the options?

**[PRIYA]:** Two fundamental patterns: **synchronous** and **asynchronous**. With synchronous communication — REST or gRPC — Service A calls Service B and waits for a response. Simple to understand, easy to debug, immediate consistency.

**[ALEX]:** REST — HTTP/JSON — is the default for most teams.

**[PRIYA]:** Right, and for good reason. Simple, human-readable, ubiquitous client support. gRPC is the alternative — uses Protobuf binary format, much faster, supports streaming, strongly typed contracts. Better for internal service-to-service calls where performance matters. Worse for browser clients that don't natively support HTTP/2 streaming.

**[ALEX]:** The downside of synchronous communication?

**[PRIYA]:** Temporal coupling. Service A can't function if Service B is down. Cascade failures. If the Payment service is slow, every checkout request is slow. If it's down, checkouts are down. You're trading simplicity for brittleness.

**[ALEX]:** And asynchronous communication solves that?

**[PRIYA]:** At the cost of complexity. With async — event-driven messaging — Service A publishes an event ("OrderPlaced") to a message broker — SQS, Kafka, RabbitMQ, SNS. Service B subscribes and processes the event when it can. A doesn't wait for B. A doesn't even know B exists.

**[ALEX]:** The producer and consumer are decoupled.

**[PRIYA]:** Like dropping a letter in a mailbox. You don't wait for the recipient to open it. You don't even know exactly when they'll read it. The postal system is the broker. If the recipient is on vacation (service is down), the letter waits in the mailbox (queue) until they return.

**[ALEX]:** Choreography versus orchestration. Let's go through that.

**[PRIYA]:** These are two approaches to coordinating multi-service workflows. **Choreography** — each service reacts to events and publishes its own events. No central coordinator. An order is placed → Order service publishes `OrderPlaced` → Inventory service subscribes, reserves stock, publishes `StockReserved` → Payment service subscribes, charges card, publishes `PaymentConfirmed` → Fulfillment subscribes and ships.

**[ALEX]:** Like a flash mob — everyone knows the dance, no one's giving orders.

**[PRIYA]:** Beautiful analogy. **Orchestration** is the opposite — a central orchestrator tells each service what to do and waits for confirmation. AWS Step Functions is a great orchestrator. It gives you visibility into exactly where in the workflow you are, handles retries, and provides a visual representation of the flow.

**[ALEX]:** When do you choose choreography vs orchestration?

**[PRIYA]:** Choreography: when services are truly independent, when you want low coupling, when flows are simple. Orchestration: when you need visibility into long-running workflows, when you need complex error handling and compensation, when the workflow has conditional branches. Most real systems use both.

🎵 *Transition music sting*

---

# SEGMENT 4: Data Management — The Hardest Part `(24:00 – 33:00)`

**[ALEX]:** Database per service is the rule. But then how do you handle operations that span multiple services?

**[PRIYA]:** This is genuinely hard. The classic example: placing an order requires debiting inventory AND charging the customer. These are in two separate services with two separate databases. You can't use a database transaction across them.

**[ALEX]:** The distributed transaction problem.

**[PRIYA]:** Welcome to distributed systems. The solution is **Sagas**. A saga is a sequence of local transactions, each publishing events that trigger the next step. If any step fails, compensating transactions undo the previous steps.

**[ALEX]:** Like a chain reaction of local commits and undos.

**[PRIYA]:** Exactly. Order Saga for checkout:

```
1. Create order (pending) → publishes OrderCreated
2. Reserve inventory → publishes InventoryReserved
   OR releases order if stock unavailable → publishes OrderCancelled
3. Process payment → publishes PaymentProcessed
   OR releases inventory AND cancels order if payment fails
4. Confirm order → publishes OrderConfirmed
5. Trigger fulfillment
```

Each step is a local transaction. If payment fails at step 3, compensating transactions run steps in reverse: release the reserved inventory, cancel the order.

**[ALEX]:** This is eventual consistency — the system is consistent eventually but not immediately.

**[PRIYA]:** And you have to accept that. The customer might see "order processing" for a few seconds before it transitions to "confirmed" or "cancelled." Design your UX around this — optimistic UI, clear status messages. Don't show instant confirmation if the system can't truly guarantee it instantly.

**[ALEX]:** CQRS — Command Query Responsibility Segregation. Where does that fit?

**[PRIYA]:** CQRS separates read operations (queries) from write operations (commands). You have separate models optimized for each. Writes go to the authoritative database — normalized, transactionally consistent. Reads go to a separate read model — denormalized, optimized for query patterns, potentially in a different technology (Elasticsearch for full-text search, Redis for fast lookups).

**[ALEX]:** The read model is updated from events?

**[PRIYA]:** Right — **Event Sourcing**. Instead of storing current state, you store the sequence of events that led to the state. Current state is derived by replaying events. Like accounting: you don't store the current balance, you store every debit and credit, and the balance is the sum. This gives you complete audit history, the ability to replay history with new logic, and a natural event stream to update read models.

🎵 *Transition music sting*

---

# SEGMENT 5: Resilience Patterns — Building for Failure `(33:00 – 41:00)`

**[ALEX]:** In a microservices system, services fail. How do you stop one failure from cascading?

**[PRIYA]:** Resilience is non-negotiable. Let's go through the key patterns. Start with **Circuit Breaker** — named after electrical circuit breakers. If Service B starts failing, instead of letting Service A retry forever (creating a thundering herd), a circuit breaker trips after a threshold of failures.

**[ALEX]:** Like the circuit breaker in your house that trips when there's a power surge.

**[PRIYA]:** Exactly. Three states: Closed — normal operation, requests flow through. Open — the circuit has tripped, requests fail immediately without hitting the downstream service (fail fast). Half-Open — after a cooldown period, try one request. If it succeeds, close the circuit. If it fails, reopen.

```python
# Using the circuit-breaker library (Python)
from circuitbreaker import circuit

@circuit(failure_threshold=5, recovery_timeout=30)
def call_payment_service(order_id):
    return requests.post('http://payment-service/charge', json={'orderId': order_id})
```

**[ALEX]:** What does the caller do when the circuit is open?

**[PRIYA]:** **Fallback** — a degraded response when the normal path is unavailable. An e-commerce site where the recommendations service is down shows static popular items instead of personalized ones. A news feed returns cached results. The key: define what "acceptable degradation" looks like for each service before you need it.

**[ALEX]:** Retry with backoff.

**[PRIYA]:** Always retry with **exponential backoff and jitter**. Retry immediately, then wait 1 second, then 2, then 4, then 8 — doubling each time. Add random jitter — a random 0–1 second — to prevent all retrying clients from hitting the recovering service at the same moment.

```python
import random
import time

def retry_with_backoff(func, max_retries=5):
    for attempt in range(max_retries):
        try:
            return func()
        except Exception:
            if attempt == max_retries - 1:
                raise
            wait = (2 ** attempt) + random.uniform(0, 1)
            time.sleep(wait)
```

**[ALEX]:** Timeouts.

**[PRIYA]:** Every outgoing service call must have a timeout. No timeout means a single slow downstream service can hold all your threads and bring your service down. Set timeouts at the P99 of expected response time — if 99% of calls complete in 200ms, set a 500ms timeout. Requests that take longer are failing anyway.

**[ALEX]:** Bulkhead — I've heard this term.

**[PRIYA]:** From ship design — hull compartments that contain flooding. In software: isolate different downstream calls in separate thread pools or connection pools. If payment service calls are slow and exhaust one thread pool, it doesn't exhaust the pool used for inventory calls. The flood stays in one compartment.

🎵 *Transition music sting*

---

# SEGMENT 6: API Gateway & Service Discovery `(41:00 – 48:00)`

**[ALEX]:** External clients — like a mobile app — how do they talk to a microservices backend?

**[PRIYA]:** Through an **API Gateway**. The gateway is the single entry point for all clients. It handles: routing requests to the appropriate service, authentication and authorization, rate limiting, request/response transformation, SSL termination, and sometimes aggregation — combining multiple service responses into one.

**[ALEX]:** The Backend for Frontend pattern?

**[PRIYA]:** A refinement of API Gateway — instead of one gateway for all clients, you have a specific gateway per client type. The mobile BFF (Backend for Frontend) aggregates and transforms data for mobile's limited bandwidth and screen size. The web BFF does the same for the web app. The BFF team lives with the frontend teams, moving fast without coupling to internal service contracts.

**[ALEX]:** Service discovery — how does Service A find Service B's address?

**[PRIYA]:** Services in dynamic environments — Kubernetes, ECS — have constantly changing IP addresses. Service discovery solves this.

**Client-side discovery**: Services register themselves in a service registry — Consul, Eureka. Clients query the registry to find available instances and load-balance themselves.

**Server-side discovery**: The load balancer or DNS handles it. Kubernetes Services with CoreDNS — `http://order-service.default.svc.cluster.local` — always routes to healthy instances. The client is blissfully unaware of individual pod IPs.

**[ALEX]:** Kubernetes basically does server-side discovery for you.

**[PRIYA]:** Which is one of the big reasons Kubernetes adoption took off. On AWS with ECS, Cloud Map does the same. For the modern stack, you rarely implement service discovery yourself.

**[ALEX]:** Service mesh — Istio, Linkerd. Where does that fit?

**[PRIYA]:** A service mesh moves the resilience patterns — circuit breaking, retry, timeout, mTLS, traffic splitting — out of your application code and into a sidecar proxy injected into every pod. Your application code makes a plain HTTP call. The sidecar handles the retry policy, the circuit breaker, encrypts the connection with mTLS, records a trace.

**[ALEX]:** Separation of concerns — business logic in the app, infrastructure concerns in the sidecar.

**[PRIYA]:** And it works across every language. Your Node.js service and your Java service get the same retry and circuit-breaking behavior without any library differences. The tradeoff: operational complexity of managing the mesh control plane. Istio in particular has a steep learning curve. Start without a mesh, add it when you feel the pain of inconsistent resilience implementations across services.

🎵 *Transition music sting*

---

# SEGMENT 7: Observability — You Can't Avoid Distributed Tracing `(48:00 – 54:00)`

**[ALEX]:** Debugging a monolith is hard. Debugging microservices must be a nightmare.

**[PRIYA]:** It is, without proper observability. The three pillars: logs, metrics, traces. In a monolith, a request has one log file, one process. In microservices, one user request might touch 8 services. Without tooling, good luck correlating what happened.

**[ALEX]:** Correlation IDs.

**[PRIYA]:** The foundation. Every incoming request gets a unique correlation ID — a UUID — at the edge (API Gateway or first service). Every service logs this ID with every log line. Every downstream call includes the ID in a header — `X-Correlation-ID`. Now you can search your log aggregator for that ID and see the complete journey across all services.

```python
import uuid
from flask import Flask, request, g

app = Flask(__name__)

@app.before_request
def set_correlation_id():
    g.correlation_id = request.headers.get('X-Correlation-ID', str(uuid.uuid4()))

@app.after_request
def add_correlation_header(response):
    response.headers['X-Correlation-ID'] = g.correlation_id
    return response
```

**[ALEX]:** Distributed tracing goes further than correlation IDs?

**[PRIYA]:** Yes — distributed tracing, with tools like Jaeger, Zipkin, or AWS X-Ray, records the full call graph. Each service creates a **span** — a named unit of work with start time, duration, metadata. Spans are connected in a **trace** that shows the complete tree: API Gateway (50ms) → Order Service (30ms) → [Inventory Service (8ms), Payment Service (20ms) in parallel] → Notification Service (5ms).

**[ALEX]:** You can see exactly where latency is coming from.

**[PRIYA]:** That's the power. Without tracing, "checkout is slow" is a 3-hour debugging session. With tracing, you open the trace, see Payment Service is taking 400ms, drill in, see it's a slow database query — 10 minutes.

**[ALEX]:** Structured logging — JSON logs — is non-negotiable.

**[PRIYA]:** Every service should log JSON. Every log line should have: timestamp, severity, service name, correlation ID, and context-specific fields. This makes logs queryable in CloudWatch Insights, Elasticsearch, Datadog — any log aggregation system.

```json
{
  "timestamp": "2024-01-15T10:23:45Z",
  "level": "INFO",
  "service": "order-service",
  "version": "v2.3.1",
  "correlationId": "abc-123-xyz",
  "action": "create_order",
  "userId": "user-456",
  "orderId": "order-789",
  "durationMs": 23
}
```

**[ALEX]:** Health checks at every service.

**[PRIYA]:** Two types. **Liveness**: is the service alive? If no, restart it. A simple HTTP endpoint returning 200. **Readiness**: is the service ready to serve traffic? Checks database connectivity, dependency availability. If not ready, don't route traffic to it. This distinction is critical — a service can be alive but not ready during startup or while recovering from a dependency outage.

🎵 *Transition music sting*

---

# SEGMENT 8: Deployment Patterns — Independent Deployability `(54:00 – 58:00)`

**[ALEX]:** The whole point of microservices is being able to deploy services independently. How do you enable that?

**[PRIYA]:** Independent deployability is the goal, and several practices enable it. First: **each service has its own CI/CD pipeline**. Push to the order-service repo → tests run → if they pass, order-service is deployed. The payment-service deployment is completely separate. No shared release train.

**[ALEX]:** What about breaking changes to shared APIs?

**[PRIYA]:** **Expand-Contract** pattern — also called parallel change. When changing an API: first expand (add the new field/endpoint while keeping the old). Wait until all consumers are updated. Then contract (remove the old). Never break consumers by removing things they're using.

**[ALEX]:** Feature flags for safer deployments.

**[PRIYA]:** Feature flags decouple deployment from release. You deploy new code to production with the feature disabled. Enable it for 1% of traffic. Watch metrics. Expand to 10%, 50%, 100%. Or instantly disable it if something goes wrong — no redeploy needed.

**[ALEX]:** Consumer-Driven Contract Testing — you mentioned Pact earlier.

**[PRIYA]:** This is underused and incredibly powerful. Consumers write tests that specify what they expect from a provider API. Those tests are shared with the provider team. The provider's CI runs these consumer tests — before merging, the provider must satisfy all consumer contracts. If the payment-service team wants to change a response schema, they run all consumer contract tests first. Catches breaking changes before they reach production.

🎵 *Closing music begins to fade in softly*

---

# SEGMENT 9: When NOT to Use Microservices `(58:00 – 62:00)`

**[PRIYA]:** The most important segment. When should you NOT use microservices?

**[ALEX]:** Let's hear the honest take.

**[PRIYA]:** The **distributed monolith** is the worst outcome — you broke up your application but services are still tightly coupled. They share a database. One service failing cascades to all. You have all the complexity of microservices with none of the benefits. This happens when you split prematurely without clear boundaries.

**[ALEX]:** What are the signs you should stick with a monolith?

**[PRIYA]:** Small team — fewer than 8 to 10 engineers. Unclear domain boundaries — you don't understand the business well enough yet to draw lines. Early-stage product — you're still figuring out what you're building. Lack of operational maturity — no CI/CD, no observability, no on-call rotation. A monolith with clean internal modules is dramatically better than a prematurely split microservices mess.

**[ALEX]:** The famous "Majestic Monolith" — DHH's term.

**[PRIYA]:** Basecamp, Stack Overflow, Shopify initially — massive, successful products on well-maintained monoliths. The point is not that microservices are bad. The point is that they solve specific problems that you may not have yet.

**[ALEX]:** Martin Fowler's recommendation?

**[PRIYA]:** Start as a monolith. Understand your domain. Find the natural seams. Extract services surgically when you feel the pain that microservices solve — independent deployment bottlenecks, scaling specific components, team coordination overhead. Don't start with microservices unless you're rebuilding a system you already deeply understand.

**[ALEX]:** OK — what should people build to practice microservices patterns?

**[PRIYA]:** Four projects:

1. **E-Commerce Backend** — Order, Inventory, Payment, Notification services communicating via async events. Saga pattern for checkout flow. Each service has its own database.
2. **Event-Driven Pipeline** — Services communicate exclusively via Kafka or SQS events. Practice CQRS — write to one service, query from a read-optimized Redis projection.
3. **Resilience Workshop** — Use Chaos Engineering (kill random services) and validate your circuit breakers, retries, and fallbacks work as designed. Chaos Monkey for your learning project.
4. **Strangler Fig Migration** — Take a small monolith and extract one service using the Strangler Fig pattern — route traffic to the new service while the monolith still exists, gradually transferring ownership.

**[ALEX]:** That's a complete curriculum. Today we covered service design, bounded contexts, synchronous vs async communication, sagas, CQRS, resilience patterns, API gateways, observability, and when to avoid microservices entirely.

**[PRIYA]:** All the code samples, Saga diagrams, and recommended reading are in the show notes. Start simple. Extract carefully. Observe everything.

**[ALEX]:** Until next time —

**[PRIYA]:** Stay loosely coupled. 🔗

🎵 *Outro music swells and fades out over 5 seconds*

---

## 🎯 Key Analogies Reference Card

| Concept | Analogy |
|---------|---------|
| Monolith vs Microservices | One giant Swiss Army knife vs a specialized toolbox — each tool purpose-built |
| Bounded Context | "Product" means different things to Catalog, Inventory, Pricing — same word, different worlds |
| Database per Service | Locked filing cabinets per department — Finance can't read HR files directly |
| Choreography | Flash mob — everyone knows the dance, no one gives orders |
| Orchestration | Dance instructor (Step Functions) — tells each dancer what to do and waits for confirmation |
| Async Messaging | Dropping a letter in a mailbox — you don't wait for the recipient to read it |
| Circuit Breaker | Electrical circuit breaker — trips on overload to prevent worse damage |
| Fallback | Backup generator — degraded power when the grid fails, not total darkness |
| Retry with Backoff | Knocking on a door — knock once, wait, knock again louder, then wait longer |
| Bulkhead | Ship hull compartments — flooding in one compartment doesn't sink the whole ship |
| Correlation ID | Tracking number on a package — follow it through every hand it passes through |
| Distributed Trace | Flight itinerary with layovers — see every leg, see where delays happened |
| Feature Flag | Light dimmer — gradually increase exposure rather than all-on or all-off |
| Strangler Fig | Vine growing around a tree — gradually replaces the host as it matures |

---

## 📚 Patterns Quick Reference

```
┌─────────────────────────────────────────────────────────────┐
│ COMMUNICATION PATTERNS                                      │
├─────────────────────────────────────────────────────────────┤
│ REST (HTTP/JSON)   Sync, simple, universal client support   │
│ gRPC (Protobuf)    Sync, fast, strongly typed, streaming    │
│ Message Queue      Async, decoupled, buffered (SQS, RabbitMQ│
│ Event Streaming    Async, ordered, replayable (Kafka, Kinesis│
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ RESILIENCE PATTERNS                                         │
├─────────────────────────────────────────────────────────────┤
│ Circuit Breaker    Stop calling failed services immediately  │
│ Retry + Backoff    Retry with exponential wait + jitter     │
│ Timeout            Every outbound call must have a deadline │
│ Bulkhead           Separate thread pools per dependency     │
│ Fallback           Degraded response when service is down   │
│ Idempotency        Safe to process the same message twice   │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ DATA PATTERNS                                               │
├─────────────────────────────────────────────────────────────┤
│ DB per Service     No shared databases between services     │
│ Saga               Multi-step workflow with compensations   │
│ CQRS               Separate read and write models          │
│ Event Sourcing     Store events, derive state by replay     │
│ Eventual Consistency  Accept temporary inconsistency        │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ DEPLOYMENT PATTERNS                                         │
├─────────────────────────────────────────────────────────────┤
│ Independent CI/CD  Each service deploys on its own         │
│ Feature Flags      Decouple deploy from release             │
│ Expand-Contract    Add new, migrate consumers, remove old   │
│ Consumer Contracts Test that providers don't break consumers│
│ Strangler Fig      Gradually extract from monolith          │
│ Blue/Green         Switch all traffic at once, instant undo │
│ Canary             Gradual traffic shift with auto-rollback │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ OBSERVABILITY CHECKLIST                                     │
├─────────────────────────────────────────────────────────────┤
│ ✓ Correlation IDs propagated through every service call    │
│ ✓ Structured JSON logging (not plaintext)                  │
│ ✓ Distributed tracing (Jaeger / X-Ray / Zipkin)            │
│ ✓ Liveness + Readiness health check endpoints              │
│ ✓ Service-level SLI/SLO metrics (latency p99, error rate)  │
│ ✓ Alerts on error rate, latency, and queue depth           │
│ ✓ Dashboards showing service dependency health             │
└─────────────────────────────────────────────────────────────┘
```

---

## 📚 Recommended Reading

- **Building Microservices** — Sam Newman (the definitive book)
- **Designing Distributed Systems** — Brendan Burns (O'Reilly)
- **Release It!** — Michael Nygard (resilience patterns)
- **Domain-Driven Design** — Eric Evans (bounded contexts)
- **Microservices.io** — Chris Richardson's pattern catalog (free, comprehensive)

---

*Small Services, Big Systems is produced independently. All trademarks belong to their respective owners.*
