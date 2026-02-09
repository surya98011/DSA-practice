# One-Page Interview Cheat Sheet (Java Backend, Banking)

**Snapshot Pitch (30 seconds)**
I have 8 years in Java backend in banking, delivering payment APIs on Java 21 and Spring Boot 3.2 with Oracle, MongoDB, Kafka, Docker, Kubernetes/OpenShift, Helm. I design for correctness, idempotency, and auditability, and I operate systems with Dynatrace, LogScale, and Day2Ops. I focus on resilient, event-driven payment flows with strong observability.

**Core Architecture (your domain)**
```
Client -> Akamai -> Apigee -> OpenShift Payment API -> Kafka (request)
Kafka (response) -> OpenShift Payment API -> Apigee -> Client
```
Key points: Apigee for auth/rate limiting/routing; APIs validate + log; Kafka for async downstream; consume response topic; return to client; strong correlation IDs.

**Must-Say Topics (System Design)**
1. Idempotency keys with unique constraints + TTL.
2. Outbox pattern to avoid dual writes.
3. Payment state machine with monotonic transitions.
4. SLIs/SLOs: P95 latency, error budget, Kafka lag.
5. Backpressure: rate limits + consumer autoscale.
6. Observability: tracing, structured logs, correlation IDs.
7. PCI/PII: tokenization, masking, least privilege.
8. Failure handling: timeouts, retries, circuit breakers, DLQ.

**Big Tech Angle**
1. Quantify scale early (TPS, peaks, data size).
2. Explicit trade-offs: consistency vs availability.
3. Multi-region DR and failover strategy.

**Fintech/Payments Angle**
1. Correctness > latency for money movement.
2. Reconciliation and audit trail are first-class.
3. Settlement cutoffs and batching awareness.

**Mid-Size Product Angle**
1. Keep it simple and evolvable.
2. Avoid over-engineering; scale when needed.
3. Prioritize maintainability and clear APIs.

**Coding Patterns I Use**
1. Two pointers / sliding window / monotonic stack.
2. Heap for top-k, quickselect for O(n) expected.
3. BFS/DFS, Dijkstra, and union-find for graphs.
4. Hash map + linked list for LRU.
5. Two heaps for streaming median.

**Concurrency & Reliability**
1. Use `ConcurrentHashMap` + `computeIfAbsent`.
2. Manual Kafka ack after successful processing.
3. Retry with exponential backoff + jitter.
4. Bounded queues with locks and conditions.

**High-Value Code Snippets (to cite)**
```java
// Idempotency: return same response for same key
Optional<PaymentResponse> existing = repo.findByIdempotencyKey(key);
if (existing.isPresent()) return existing.get();
```
```java
// Outbox for reliable event publishing
@Entity class OutboxEvent { @Id Long id; String type; @Lob String payload; Instant createdAt; }
```
```java
// Kafka consume with manual ack
@KafkaListener(topics = "payment-response", groupId = "payments")
public void onMessage(ConsumerRecord<String, String> r, Acknowledgment a) {
  process(r.value()); a.acknowledge();
}
```

**Common Follow-ups (prepare)**
1. How do you prevent double-charging? Idempotency + state machine.
2. What happens if Kafka is down? Buffer, DLQ, retry with backoff, degrade gracefully.
3. How do you handle schema changes? Registry + backward compatibility.
4. How do you ensure data integrity? Ledger pattern + append-only.
5. How do you measure success? SLOs, error budgets, and end-to-end latency.

**One-Minute System Design Template**
1. Clarify requirements + scale.
2. Propose high-level architecture.
3. Define data model + APIs.
4. Discuss consistency + reliability.
5. Observability + ops.
6. Trade-offs + future evolution.
