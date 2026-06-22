# Interview Tracker

| Field   | Detail            |
| ------- | ----------------- |
| Company | GlobalLogics      |
| Date    | 2026-06-17        |
| Time    | 3:00 PM           |

---

## Index

1. [Builder Design Pattern](#1-builder-design-pattern)
2. [BFS (Breadth-First Search)](#2-bfs-breadth-first-search)
3. [DFS (Depth-First Search)](#3-dfs-depth-first-search)
4. [SAGA Pattern – Distributed Transactions](#4-saga-pattern--distributed-transactions)
5. [Handling 1000 Requests When a Third-Party Service Is Down](#5-how-will-you-handle-1000-requests-if-a-third-party-service-is-down)
6. [NoSQL vs SQL](#6-nosql-vs-sql)
7. [Redis](#7-redis)
8. [Kubernetes HPA](#8-hpa-horizontal-pod-autoscaler)
9. [Circular Dependency](#9-circular-dependency)
10. [HashMap Internal Implementation](#10-hashmap-internal-implementation)
11. [CompletableFuture](#11-completablefuture)

---

## Interview Questions & Answers

### Design Patterns

#### 1. Builder Design Pattern
A **creational** pattern that constructs a complex object step-by-step, separating construction from representation. Useful when an object has many optional fields (avoids telescoping constructors).

- **When to use:** many constructor parameters, optional fields, immutable objects.
- **Benefits:** readable fluent API, immutability, no need for multiple constructors.

```java
Pizza pizza = new Pizza.Builder()
        .size("Large")
        .cheese(true)
        .pepperoni(true)
        .build();
```
In Java, `StringBuilder` and Lombok's `@Builder` are real-world examples.

---

### Search Algorithms

#### 2. BFS (Breadth-First Search)
Explores level by level using a **Queue (FIFO)**. Visits all neighbors at the current depth before moving deeper.

- **Data structure:** Queue + visited set.
- **Time:** O(V + E), **Space:** O(V).
- **Use cases:** shortest path in an **unweighted** graph, level-order traversal, finding nearest nodes.

```java
void bfs(Node start) {
    Queue<Node> q = new LinkedList<>();
    Set<Node> visited = new HashSet<>();
    q.add(start); visited.add(start);
    while (!q.isEmpty()) {
        Node n = q.poll();
        for (Node nb : n.neighbors)
            if (visited.add(nb)) q.add(nb);
    }
}
```

#### 3. DFS (Depth-First Search)
Explores as deep as possible before backtracking, using a **Stack** (or recursion).

- **Data structure:** Stack / recursion + visited set.
- **Time:** O(V + E), **Space:** O(V).
- **Use cases:** cycle detection, topological sort, connected components, path existence, maze solving.

```java
void dfs(Node n, Set<Node> visited) {
    if (!visited.add(n)) return;
    for (Node nb : n.neighbors) dfs(nb, visited);
}
```

**BFS vs DFS:** BFS finds shortest path in unweighted graphs and uses more memory (wide). DFS uses less memory (deep), better for exhaustive path/structure problems.

---

### Distributed Systems

#### 4. SAGA Pattern – Distributed Transactions
Manages data consistency across microservices **without** a distributed (2PC) transaction. A saga is a sequence of **local transactions**; if one step fails, **compensating transactions** undo the previous steps.

Two coordination styles:
- **Choreography:** each service publishes events that trigger the next service. No central coordinator; good for few services, can get complex.
- **Orchestration:** a central **orchestrator** tells each service what to do and handles compensation. Easier to manage and monitor for complex flows.

**Example (Order):** Order → Payment → Inventory → Shipping. If Inventory fails, run compensations: refund Payment, cancel Order.

**Key point:** provides **eventual consistency**, not ACID. Operations must be idempotent and compensations must be defined.

#### 5. How will you handle 1000 requests if a third-party service is down?
Layered resilience strategy:

1. **Circuit Breaker** (Hystrix / Resilience4j) – stop hammering the dead service; fail fast once the failure threshold is crossed.
2. **Fallback** – return cached data, a default response, or a queued acknowledgment instead of an error.
3. **Async + Queue** – push requests to a message queue (Kafka/RabbitMQ) and process them when the service recovers (decoupling, buffering).
4. **Retry with exponential backoff + jitter** – for transient failures only; cap retries to avoid overload.
5. **Timeouts & bulkheads** – isolate thread pools so the failing dependency doesn't exhaust resources for the whole app.
6. **Rate limiting / load shedding** – protect the system and the recovering service.
7. **Graceful degradation** – serve partial functionality and notify users.

> Summary: *Fail fast (circuit breaker) → fall back → queue for later → retry with backoff → degrade gracefully.*

---

### Databases

#### 6. NoSQL vs SQL

| Aspect        | SQL (RDBMS)                          | NoSQL                                   |
| ------------- | ------------------------------------ | --------------------------------------- |
| Schema        | Fixed, predefined                    | Flexible / schema-less                  |
| Structure     | Tables (rows & columns)              | Document, key-value, column, graph      |
| Scaling       | Vertical (scale-up)                  | Horizontal (scale-out)                  |
| Transactions  | Strong ACID                          | Mostly BASE / eventual consistency      |
| Relationships | Joins, foreign keys                  | Denormalized, embedded                  |
| Examples      | MySQL, PostgreSQL, Oracle            | MongoDB, Cassandra, Redis, DynamoDB     |

- **Use SQL when:** strong consistency, complex queries/joins, structured data, transactions (banking, ERP).
- **Use NoSQL when:** huge scale, high write throughput, flexible/evolving schema, unstructured data (logs, IoT, real-time analytics, caching).
- **CAP theorem:** SQL typically favors **CA/CP**; many NoSQL stores favor **AP**.

#### 7. Redis
In-memory **key-value** data store, extremely fast (sub-millisecond), single-threaded core.

- **Data types:** String, Hash, List, Set, Sorted Set, Bitmap, HyperLogLog, Streams.
- **Use cases:** caching, session store, rate limiting, leaderboards (sorted sets), pub/sub, distributed locks.
- **Persistence:** **RDB** (snapshots) and **AOF** (append-only log) — can combine.
- **Scaling/HA:** Redis Sentinel (failover), Redis Cluster (sharding).
- **Eviction policies:** LRU, LFU, TTL-based — important for cache sizing.

---

### Kubernetes

#### 8. HPA (Horizontal Pod Autoscaler)
Automatically scales the **number of pods** in a Deployment/ReplicaSet based on observed metrics.

- **Metrics:** CPU/memory utilization (via Metrics Server) or custom/external metrics.
- **How it works:** controller checks metrics on an interval and adjusts replicas toward a target (e.g., keep CPU at 50%).
- **Formula:** `desiredReplicas = ceil(currentReplicas × currentMetric / targetMetric)`.
- **Horizontal (HPA)** = more pods; **Vertical (VPA)** = bigger pods (more CPU/RAM).

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
spec:
  scaleTargetRef:
    kind: Deployment
    name: my-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50
```

---

### Spring

#### 9. Circular Dependency
Occurs when two or more beans depend on each other, forming a cycle: **A → B → A**. Neither can be fully constructed before the other, so the container can't resolve the order.

**Example:**
```java
@Component
class A { A(B b) { } }   // A needs B

@Component
class B { B(A a) { } }   // B needs A  -> cycle
```

- **Constructor injection:** Spring **cannot** resolve the cycle and throws `BeanCurrentlyInCreationException` at startup (fail-fast — actually a good thing, it surfaces a design smell).
- **Field / setter injection:** Spring **can** resolve it because it creates the bean first (early reference) and injects dependencies afterward.

**How to fix (best → workaround):**
1. **Refactor** — usually the cycle signals poor design. Extract the shared logic into a third class/service that both depend on.
2. **`@Lazy`** — inject a lazy proxy so the dependent bean is created only when first used, breaking the eager cycle.
3. **Setter / field injection** — defers wiring until after construction.
4. **`@PostConstruct`** — wire the dependency manually after beans are built.
5. (Spring Boot 2.6+) `spring.main.allow-circular-references=true` — a last-resort flag, not recommended.

> Best practice: prefer **constructor injection** and **refactor** away the cycle rather than masking it.

---

### Java Collections

#### 10. HashMap Internal Implementation
A `HashMap` stores key-value pairs in an **array of buckets** (`Node<K,V>[] table`). Each entry is a `Node` holding `hash`, `key`, `value`, and a `next` pointer.

**put(key, value) flow:**
1. Compute `hashCode()` of the key, then apply a **perturbation**: `hash = h ^ (h >>> 16)` to spread high bits and reduce collisions.
2. Find the bucket index: `index = (n - 1) & hash` (works because capacity `n` is always a power of 2).
3. If the bucket is empty → insert the node.
4. If occupied (**collision**) → compare with `equals()`:
   - Same key → overwrite the value.
   - Different key → append to the bucket's chain.

**Collision handling:**
- **Before Java 8:** collisions stored as a **linked list** → worst case lookup O(n).
- **Java 8+:** when a bucket's chain exceeds **TREEIFY_THRESHOLD = 8** (and table capacity ≥ 64), it converts to a **balanced Red-Black Tree** → worst case O(log n). It reverts to a list when size drops below **UNTREEIFY_THRESHOLD = 6**.

**Resizing (rehashing):**
- Default initial capacity **16**, load factor **0.75**.
- When `size > capacity × loadFactor` (e.g., 12 of 16), capacity **doubles** and entries are redistributed.

**Complexity:** average **O(1)** for get/put; worst case **O(log n)** (Java 8+ with treeified buckets).

**Key contracts & notes:**
- Keys must implement consistent `hashCode()` **and** `equals()`; equal objects must return equal hash codes.
- Allows **one null key** (stored in bucket 0) and multiple null values.
- **Not thread-safe** — use `ConcurrentHashMap` for concurrency (`Collections.synchronizedMap` is a coarse-grained alternative).
- Prefer **immutable keys** (e.g., `String`, `Integer`); mutating a key after insertion breaks lookups.

---

### Java Concurrency

#### 11. CompletableFuture
Introduced in **Java 8**, `CompletableFuture<T>` represents an **asynchronous computation** that may complete in the future. It improves on the old `Future` by supporting **non-blocking chaining, composition, and callbacks** — you don't have to call a blocking `get()`.

**Future vs CompletableFuture:**

| `Future`                          | `CompletableFuture`                          |
| --------------------------------- | -------------------------------------------- |
| Blocking `get()` only             | Non-blocking callbacks (`thenApply`, etc.)   |
| No chaining/composition           | Chain & combine multiple futures             |
| Can't be completed manually       | Can complete manually (`complete()`)         |
| No exception handling pipeline    | `exceptionally`, `handle`, `whenComplete`    |

**Creating:**
```java
// async with return value (uses ForkJoinPool.commonPool by default)
CompletableFuture<String> cf = CompletableFuture.supplyAsync(() -> "result");

// async with no return value
CompletableFuture<Void> run = CompletableFuture.runAsync(() -> doWork());

// already completed
CompletableFuture<String> done = CompletableFuture.completedFuture("value");
```

**Chaining / transforming:**
- `thenApply(fn)` – transform the result (sync), returns a value.
- `thenAccept(consumer)` – consume the result, returns void.
- `thenRun(runnable)` – run after completion, ignores result.
- `thenCompose(fn)` – **flatMap**: chain another future (avoids nested `CompletableFuture<CompletableFuture<T>>`).
- `thenCombine(other, fn)` – combine results of **two independent** futures.
- Add the **`...Async`** suffix (e.g., `thenApplyAsync`) to run the step on a different thread / custom executor.

```java
CompletableFuture.supplyAsync(() -> fetchUser(id))
    .thenApply(user -> user.getName())          // transform
    .thenCompose(name -> fetchOrders(name))     // chain dependent call
    .thenAccept(System.out::println);           // consume
```

**Combining many:**
```java
CompletableFuture.allOf(f1, f2, f3).join();   // wait for ALL
CompletableFuture.anyOf(f1, f2, f3).join();   // first to complete
```

**Exception handling:**
- `exceptionally(ex -> fallback)` – recover with a fallback value.
- `handle((res, ex) -> ...)` – runs on both success and failure.
- `whenComplete((res, ex) -> ...)` – side-effect (logging), doesn't alter the result.

```java
CompletableFuture.supplyAsync(() -> riskyCall())
    .exceptionally(ex -> "default")
    .thenAccept(System.out::println);
```

**Key points:**
- Prefer a **custom `ExecutorService`** for blocking/IO tasks instead of the default common pool (which is sized for CPU-bound work).
- `get()` is blocking and throws checked exceptions; `join()` is blocking but throws unchecked.
- Great for parallel I/O calls (e.g., aggregating multiple microservice responses) and ties into the *"handle 1000 requests"* async strategy above.

---

## Notes
_(Add follow-ups and interviewer feedback here)_
