# Half-Page Flash Card (Java Backend, Banking)

**30s Pitch**  
I have 8 years in Java backend in banking. I build payment APIs on Java 21 + Spring Boot 3.2 with Oracle, MongoDB, Kafka, Docker, Kubernetes/OpenShift, and Helm. I design for idempotency, auditability, and resilience, and I operate services with Dynatrace, LogScale, and Day2Ops.

**Core Flow**  
Client -> Akamai -> Apigee -> OpenShift API -> Kafka (req)  
Kafka (res) -> OpenShift API -> Apigee -> Client

**System Design Must-Say**
1. Idempotency keys + unique constraint + TTL.
2. Outbox pattern to avoid dual writes.
3. Monotonic payment state machine.
4. SLIs/SLOs: P95 latency, error rate, Kafka lag.
5. Backpressure + DLQ + retries with jitter.
6. PCI/PII: tokenization, masking, least privilege.

**Coding Must-Say**
1. Two pointers / sliding window / monotonic stack.
2. Heap for top-k, quickselect trade-off.
3. BFS/DFS/Dijkstra for graphs.
4. LRU via hashmap + DLL.
5. Two heaps for streaming median.

**Concurrency/Resilience**
1. `ConcurrentHashMap` + `computeIfAbsent`.
2. Manual Kafka ack after successful processing.
3. Timeouts, circuit breakers, bulkheads.

**One-Minute Design Template**
1. Clarify requirements + scale.
2. High-level architecture.
3. Data model + APIs.
4. Consistency + reliability.
5. Observability + ops.
6. Trade-offs + evolution.
