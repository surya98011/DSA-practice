# Part 2: Real Production Scenarios (7 YOE KILLER QUESTIONS)

For a 7 YOE candidate, interviewers are testing *battle scars*. You don't just know theories; you've survived production deployments.

---

### Q122: Application is slow in production – how will you debug?
**Structured Approach:**
1. **Identify the Scope**: Is it slow for all users (systemic issue) or specific users (data skew)? Is it a specific endpoint or the whole app?
2. **Observability tools (APM)**: Open Datadog / New Relic / Zipkin. Check Distributed Tracing to see EXACTLY which microservice or DB call is the bottleneck.
3. **If it's DB**: Look for slow queries, missing indexes, table locks, or exhausted HikariCP connection pools.
4. **If it's App layer**: Check metrics (Actuator/Prometheus). Is CPU maxed? Is there a memory leak causing continuous Major Garbage Collections (Stop-the-World)? Check thread pool exhaustion.
5. **Network**: Check if latency between regions has spiked or if a downstream 3rd-party API is slow and we lack circuit breakers.

### Q123: How to identify memory leaks?
1. **Symptoms**: Response times get progressively worse. CPU spikes due to continuous GC cycles. Eventually ends in `java.lang.OutOfMemoryError: Java heap space`.
2. **Verification**: Look at JVM Heap graphed over time in Prometheus/Grafana. If the "sawtooth" pattern never returns to the baseline after GC, memory is leaking.
3. **Debugging steps**:
   - Capture a Heap Dump when memory is high: `jcmd <pid> GC.heap_dump /tmp/heap.hprof`
   - Download the `.hprof` file and open it in **Eclipse MAT (Memory Analyzer Tool)**.
   - Run the "Leak Suspects" report. Look for collections (e.g., `HashMap`, `List`) growing infinitely without eviction, or unclosed DB/Stream resources, or ThreadLocal variables never cleared.

### Q124: Production exception occurred – what steps will you take?
1. **Acknowledge & Mitigate**: The immediate goal is MTTR (Mean Time To Recovery). If deploying V2 caused it, **rollback immediately** to V1. Do not debug on a bleeding production server.
2. **Analyze**: Go to Splunk/Kibana and search for the `Trace ID` or the Stack Trace.
3. **Reproduce**: Try to reproduce the issue locally or in the staging environment using the exact payload that failed.
4. **Root Cause Analysis (RCA)**: After fixing and deploying the hotfix, write a post-mortem documenting *why* it happened and *what* safeguards (tests/alerts) we are adding so it never happens again.

### Q125: Database is slow – what changes will you do?
1. **Query Analysis**: Use `EXPLAIN ANALYZE` on the slow query. Check if it's doing a Full Table Scan.
2. **Indexing**: Add appropriate B-Tree or Composite indexes. Ensure we aren't suffering from over-indexing (which slows writes).
3. **N+1 Problem**: Ensure JPA/Hibernate isn't firing hundreds of queries to fetch child entities. Use `@EntityGraph` or `JOIN FETCH`.
4. **Connection Pool**: Are we waiting for connections in Hikari? Increase pool size (carefully, matching `(core_count * 2) + effective_spindle_count`).
5. **Caching**: Introduce Redis to serve read-heavy, rarely changing data.
6. **Archiving/Partitioning**: If the table has 500 million rows, can we partition it by date, or archive old data?

### Q126: High CPU usage – how to debug?
1. Login to the server, use `top` or `htop` to identify the Java Process ID (PID).
2. Inside that PID, find which OS thread is using the most CPU: `top -H -p <pid>`
3. Convert the heavy Thread ID (decimal) to Hex.
4. Take a Thread Dump: `jstack <pid> > threaddump.txt`
5. Open `threaddump.txt` and search for the Hex Thread ID. This will point you to the EXACT line of Java code causing the spike. (Common causes: Infinite `while` loops, Heavy Regex matching, excessive GC activity, poor JSON serialization logic).

### Q127, Q128: Thread Pool & Connection Pool tuning?
- **HikariCP (Database)**: Setting max connections to 1000 is bad. It causes context switching and thrashing at the DB level. A common formula is `connections = ((core_count * 2) + effective_spindle_count)`. Usually, a pool of `10-20` is optimal for standard web loads.
- **Tomcat Thread Pool**: Default is `200`. If downstream APIs are slow (blocking IO), all 200 threads will get blocked, returning 500s. Solution: Implement non-blocking reactive stacks (WebFlux) OR tune threads higher, OR the preferred way – use Bulkheads (Resilience4j) to limit concurrency per downstream service.

### Q129, Q130: Timeout and Retry Design?
**Timeouts**: You must have read and connect timeouts on every network call. 
**Retries**:
- DO NOT retry blindly. Only retry on transient failures (e.g., `503 Service Unavailable`, `502 Bad Gateway`, or Network/Read timeouts). 
- Do NOT retry on a `400 Bad Request` (it will always fail).
- Use **Exponential Backoff and Jitter**: If an API is down, 50 microservices retrying simultaneously will cause a Thundering Herd effect. Adding jitter (randomness to the wait time) prevents synchronization.

```java
// Spring Retry or Resilience4j Backoff concept
@Retry(name = "paymentService", fallbackMethod = "fallback")
// Configuration will dictate: maxAttempts=3, waitDuration=1000ms, backoffMultiplier=2
```

### Q131: What are idempotent APIs?
An API is idempotent if making multiple identical requests has the same effect as making a single request. 
- `GET`, `PUT`, `DELETE` are idempotent by definition.
- `POST` is NOT idempotent. Creating an order twice causes duplicate credit card charges.
**Implementation**: The client sends a unique `Idempotency-Key` header with the `POST` request. The server checks Redis or a DB table: "Have I seen this key before?". If yes, simply return the cached response of the first successful processing without mutating state again.

### Q132: Data consistency in microservices?
Microservices usually have a Database-per-service pattern. You can't use traditional 2PC (Two-Phase Commit) distributed database transactions because they block and destroy availability.
Instead, use the **Saga Pattern**:
1. **Choreography**: Services publish Domain Events (Kafka), and other services react to them. If a step fails, compensation events are fired to undo previous steps.
2. **Orchestration**: A centralized coordinator service (like Temporal or AWS Step Functions) manages the workflow and invokes compensation logic if a failure occurs.

### Q136, Q137, Q138: Feature Toggle, Versioning, Backward Compatibility?
**To prevent breaking clients:** never modify an existing endpoint’s response payload by removing fields or changing types. Give it a new version: `/v1/api/users` vs `/v2/api/users`.
**Feature Toggles (LaunchDarkly/Unleash)**: 
Allows deploying code to production but keeping it turned "off". If a bug is caught, you just flip the toggle off instantly via an admin dashboard without restarting servers or doing rollbacks.

### Q140 - Q143: Cache Design (Redis Cache)
- **Cache-Aside Pattern**: App checks Redis. If miss, app fetches from DB, puts in Redis, and returns to client.
- **Eviction Strategies**: `LRU` (Least Recently Used), `TTL` (Time To Live). Give everything a TTL so data eventually expires.
- **Consistency Problems**: The hardest part. You must invalidate or update Redis *immediately* when a DB update happens. (See Cache Invalidation or Outbox Pattern).

### Q146, Q147, Q148: Kafka vs RabbitMQ? Eventual Consistency & DLQ?
- **RabbitMQ**: Smart broker, dumb consumer. Good for Point-to-Point, complex routing, traditional message queues.
- **Kafka**: Dumb broker, smart consumer. Good for massive event streaming, replayability (messages stay on disk), and log aggregation.
- **DLQ (Dead Letter Queue)**: If a message fails processing repeatedly (e.g., schema validation fails, or DB is permanently down), moving it back to the end of the queue is bad (poison pill). Instead, send it to a DLQ for manual inspection so the queue can keep processing healthy messages.
- **Eventual Consistency**: State won't be identical immediately across bounded contexts, but _eventually_ all systems will sync up seamlessly. (e.g., You place a YouTube comment, it might take 1 second for someone in another region to see it).

### Q151: Describe a real production issue you solved (To use in interview)
**Scenario (Adapt this to your resume):**
"During Black Friday peak traffic, our Checkout API latency spiked from 200ms to 4000ms. I checked Datadog and realized the thread pools in our Tomcat server were exhausted. Digging deeper via Zipkin distributed tracing, I found a downstream 3rd party fraud-detection API was responding slowly. Because we didn't have strict timeouts and circuit breakers, our threads were hung waiting for them. 
**Solution**: I implemented a Resilience4j Circuit Breaker on the Feign client with a 2-second timeout. If the 3rd party was slow, the circuit tripped and we bypassed the fraud check for smaller transactions, immediately recovering the API performance and saving the Black Friday sales."
