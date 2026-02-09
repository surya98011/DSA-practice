# Java Backend Coding Interview Q&A (50)

Context: 8 years Java backend (banking), Java 21, Spring Boot 3.2, Oracle, MongoDB, Kafka, Docker, Kubernetes/Helm, Dynatrace, LogScale, Day2Ops.

1. Q: Two Sum in O(n)?  
   A: I use a hash map from value to index. For each value `v`, I check if `target - v` exists; if yes I return indices. This is O(n) time, O(n) space.

2. Q: Detect if a linked list has a cycle.  
   A: I use Floyd’s tortoise-hare. If slow and fast meet, there is a cycle; otherwise no cycle. This is O(n) time and O(1) space.

3. Q: Merge two sorted lists.  
   A: I use a dummy head and stitch nodes in ascending order, then append the remainder. O(n+m) time, O(1) extra space.

4. Q: LRU cache implementation.  
   A: I combine `LinkedHashMap` with access-order or a custom doubly linked list + hash map. I evict on capacity overflow. O(1) get/put.

5. Q: Implement a thread-safe singleton.  
   A: I prefer the enum singleton in Java. If I need lazy init, I use the static holder pattern for safe, lazy, and lock-free init.

6. Q: Find k-th largest element.  
   A: I use a min-heap of size k; push elements and pop when size exceeds k. O(n log k).

7. Q: Reverse a string without extra space.  
   A: I convert to char array and swap from both ends. O(n).

8. Q: Validate parentheses.  
   A: I use a stack; push openers and match with closers. Empty stack at end means valid. O(n).

9. Q: Binary search in rotated sorted array.  
   A: I identify the sorted half and narrow the search accordingly. O(log n).

10. Q: Top K frequent elements.  
    A: I count frequencies in a map, then use a min-heap of size k or bucket sort. O(n log k) or O(n).

11. Q: Detect a palindrome string.  
    A: I compare from both ends with two pointers, skipping non-alphanumerics as needed. O(n).

12. Q: Convert Roman numeral to integer.  
    A: I scan left to right; if current < next, I subtract; else add. O(n).

13. Q: Merge intervals.  
    A: I sort by start and merge overlapping intervals in one pass. O(n log n).

14. Q: Longest common prefix.  
    A: I compare characters column-wise across all strings and stop on mismatch. O(n * m).

15. Q: Unique paths in a grid.  
    A: I use DP with `dp[i][j] = dp[i-1][j] + dp[i][j-1]`. O(mn).

16. Q: Climbing stairs.  
    A: It’s Fibonacci; I compute iteratively with O(1) space. O(n).

17. Q: Best time to buy/sell stock.  
    A: I track min price so far and update max profit. O(n).

18. Q: Meeting rooms (min rooms).  
    A: I sort start/end times and sweep with two pointers to track max overlap. O(n log n).

19. Q: Check if a tree is a BST.  
    A: I validate with min/max bounds during DFS. O(n).

20. Q: Lowest common ancestor in a BST.  
    A: I walk down from the root; if both targets are smaller go left, bigger go right, else current is LCA. O(h).

21. Q: Serialize/deserialize binary tree.  
    A: I use preorder with null markers and rebuild via recursion. O(n).

22. Q: Level order traversal.  
    A: I use a queue and process level by level. O(n).

23. Q: Kth smallest in BST.  
    A: I do in-order traversal and count nodes. O(h+k).

24. Q: Find duplicate in array (1..n).  
    A: I use Floyd’s cycle detection on indices. O(n), O(1).

25. Q: Product of array except self.  
    A: I compute prefix and suffix products and multiply; no division. O(n).

26. Q: Rotate array by k.  
    A: I reverse the full array, then reverse two parts. O(n), O(1).

27. Q: Search a 2D matrix.  
    A: I binary search treating it as a 1D array. O(log(mn)).

28. Q: Find all anagrams in a string.  
    A: I use a sliding window with character counts. O(n).

29. Q: Maximum subarray (Kadane).  
    A: I keep running max and global max. O(n).

30. Q: Implement a rate limiter.  
    A: I use a fixed window or sliding window with Redis; locally, I use `ConcurrentHashMap` + `AtomicInteger` with expiry.

31. Q: Design a thread-safe bounded queue.  
    A: I use a `ReentrantLock` with `Condition` for notEmpty and notFull. O(1) ops.

32. Q: Count distinct in sliding window.  
    A: I use a map of counts; add/remove as window slides. O(n).

33. Q: Check graph is bipartite.  
    A: I color via BFS/DFS and check for conflicts. O(V+E).

34. Q: Shortest path in unweighted graph.  
    A: I use BFS. O(V+E).

35. Q: Dijkstra on weighted graph.  
    A: I use a priority queue and relax edges. O(E log V).

36. Q: Detect cycle in directed graph.  
    A: I use DFS with visited and recursion stack. O(V+E).

37. Q: Trie insert/search.  
    A: I model nodes with an array/map of children and a boolean end flag. O(L).

38. Q: Word break.  
    A: I use DP on prefixes: `dp[i]=true` if any `dp[j]` and substring `j..i` in dict. O(n^2).

39. Q: Min stack.  
    A: I keep a stack of values and a stack of mins. O(1) getMin.

40. Q: Implement a JSON parser (basic).  
    A: I implement a recursive descent with a tokenizer for objects, arrays, strings, numbers, and literals.

41. Q: Concurrency safe map usage.  
    A: I use `ConcurrentHashMap` and atomic `computeIfAbsent` for initialization to avoid races.

42. Q: Idempotency key handling.  
    A: I store keys with responses in Redis/DB and return the same response for duplicates within TTL.

43. Q: Kafka consumer exactly-once processing.  
    A: I use transactions or idempotent writes with dedupe key; I commit offsets after a successful write.

44. Q: Implement exponential backoff with jitter.  
    A: I use `base * 2^attempt` plus random jitter and cap at max delay.

45. Q: Oracle pagination query.  
    A: I use `OFFSET ... FETCH NEXT` for stable pagination with indexed ordering.

46. Q: MongoDB aggregation for totals.  
    A: I use `$match` then `$group` with `$sum` and `$cond`.

47. Q: How do I compute moving average?  
    A: I maintain a fixed-size queue and running sum for O(1) updates.

48. Q: Find median from data stream.  
    A: I use two heaps (max-heap for lower half, min-heap for upper) to keep balance. O(log n) insert.

49. Q: Implement token bucket.  
    A: I store last refill timestamp and current tokens; refill based on elapsed time, then consume if available.

50. Q: Build a simple Spring Boot REST endpoint.  
    A: I expose `@RestController` with `@GetMapping` or `@PostMapping`, validate with `@Valid`, and return DTOs.

## Sample code snippets (Java 21 / Spring Boot 3.2)

```java
// LRU cache with LinkedHashMap
class LruCache<K, V> extends LinkedHashMap<K, V> {
    private final int capacity;
    LruCache(int capacity) {
        super(capacity, 0.75f, true);
        this.capacity = capacity;
    }
    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > capacity;
    }
}
```

```java
// Thread-safe bounded queue
class BoundedQueue<T> {
    private final Object[] items;
    private int head, tail, count;
    private final ReentrantLock lock = new ReentrantLock();
    private final Condition notEmpty = lock.newCondition();
    private final Condition notFull = lock.newCondition();
    BoundedQueue(int capacity) { this.items = new Object[capacity]; }
    public void put(T item) throws InterruptedException {
        lock.lock();
        try {
            while (count == items.length) notFull.await();
            items[tail] = item;
            tail = (tail + 1) % items.length;
            count++;
            notEmpty.signal();
        } finally {
            lock.unlock();
        }
    }
    @SuppressWarnings("unchecked")
    public T take() throws InterruptedException {
        lock.lock();
        try {
            while (count == 0) notEmpty.await();
            T item = (T) items[head];
            items[head] = null;
            head = (head + 1) % items.length;
            count--;
            notFull.signal();
            return item;
        } finally {
            lock.unlock();
        }
    }
}
```

```java
// Kafka consumer with manual ack (Spring for Apache Kafka)
@KafkaListener(topics = "payment-response", groupId = "payments")
public void onMessage(ConsumerRecord<String, String> record, Acknowledgment ack) {
    process(record.value());
    ack.acknowledge(); // commit offset after successful processing
}
```

```java
// Spring Boot REST endpoint
@RestController
@RequestMapping("/payments")
class PaymentController {
    @PostMapping
    public ResponseEntity<PaymentResponse> pay(@Valid @RequestBody PaymentRequest req) {
        PaymentResponse res = service.process(req);
        return ResponseEntity.ok(res);
    }
}
```

**Tier Tailoring**

**Big Tech Focus (FAANG-level)**
1. I emphasize optimal time/space and prove complexity with tight bounds.
2. I call out edge cases first, then test with 2-3 examples before coding.
3. I write clean invariants for two-pointer, sliding-window, and monotonic-stack problems.
4. I include trade-offs between heaps, quickselect, and bucket techniques for top-k.
5. I explain iterative vs recursive memory usage and stack overflow risks.
6. I prepare for follow-ups that remove or change constraints.
7. I keep code minimal, with clear naming and no extra allocations.
8. I explain how I would unit test tricky boundaries.

**High-Scale Fintech/Payments Focus**
1. I connect algorithms to real problems: idempotency, dedupe, ordering, and rate limits.
2. I explain exactly-once vs at-least-once semantics in code patterns.
3. I practice concurrency questions and lock-free or contention-reducing designs.
4. I add latency budgets and throughput considerations to solutions.
5. I focus on robustness: retries, timeouts, and backpressure-friendly structures.
6. I show how data structures support SLAs and audit trails.
7. I highlight safe handling of money values (BigDecimal, rounding).
8. I show how I keep operations deterministic and auditable.

**Mid-Size Product Focus**
1. I prioritize clarity and correctness over micro-optimizations unless asked.
2. I show practical trade-offs and explain why they are good enough.
3. I keep solutions readable and easy to maintain.
4. I include basic tests and explain how I would monitor in production.
5. I discuss data validation and error handling as part of the solution.
6. I show that I can extend or refactor if requirements change.
7. I focus on clean APIs and modular code boundaries.
8. I avoid over-engineering unless scale requires it.

**Hard Variants to Drill**
1. LRU cache with TTL and concurrent access.
2. Sliding window with dynamic constraints (variable window size).
3. Streaming median with deletions.
4. Top-k with frequent updates and bounded memory.
5. Graph shortest path with time-dependent edges.
6. Kth smallest in a sorted matrix with memory constraints.
7. Interval scheduling with weighted profits.
8. Deduplication with limited RAM using bloom filters.
