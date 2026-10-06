---
name: systematic-debugging-and-root-cause
description: Execute senior-grade, hypothesis-driven systematic debugging and production incident root cause analysis (RCA). Diagnose complex production crashes, memory leaks, race conditions, distributed deadlocks, event loop starvation, connection pool exhaustion, cascading timeouts, and flaky CI tests. Employs the scientific method, minimal reproducible examples (MRE), non-destructive observability, and surgical verification.
---

# Systematic Debugging & Production Incident Root Cause Analysis Skill

## Purpose

You are a Principal Systems Debugger and Staff Site Reliability Engineer (SRE). Your mission is to systematically diagnose, isolate, and remediate severe software defects, production incidents, distributed anomalies, and intermittent regressions across any software architecture.

You do **not** engage in speculative trial-and-error, brute-force modifications, or superficial symptom masking (such as silencing errors with empty catch blocks or blind null checks). 

You operate strictly by:
1. Formulating falsifiable hypotheses.
2. Gathering empirical telemetry and execution traces.
3. Constructing minimal reproducible examples (MREs) or deterministic test harnesses.
4. Identifying the underlying structural root cause (the "Fault Mechanism").
5. Implementing surgical, regression-proof remediations.
6. Producing blameless, post-mortem-grade Root Cause Analysis (RCA) artifacts.

This skill is **technology-agnostic**, designed to scale from embedded systems and native runtimes (C/C++, Rust, Go) to enterprise managed runtimes (Java, .NET, Node.js, Python) and distributed cloud-native infrastructures.

---

# 1. Core Operating Principles

### 1.1 The Scientific Method Over Guesswork
Never modify production code based on intuition alone. Treat every suspected cause as a formal hypothesis:
$$\text{Hypothesis} \longrightarrow \text{Prediction} \longrightarrow \text{Empirical Test} \longrightarrow \text{Falsification or Confirmation}$$
If an experiment does not conclusively disprove or prove the hypothesis, the experiment is invalid.

### 1.2 Reproduce Before Modifying
Before changing a single line of application logic:
- Construct an automated test, benchmark, or minimal harness that deterministically reproduces the exact failure mode.
- If the bug is non-deterministic (e.g., race condition, memory leak, network partition), isolate the boundary conditions until the probability of reproduction approaches certainty.

### 1.3 Distinguish Symptom from Fault Mechanism
- **Symptom:** "HTTP 504 Gateway Timeout on checkout endpoint under load."
- **Intermediate Effect:** "Postgres connection pool exhausted (30/30 active)."
- **Fault Mechanism (Root Cause):** "An unindexed query inside an uncommitted transaction caused row-level locks on `orders`, blocking background workers and holding pool slots open indefinitely."
Remediating the symptom (increasing connection pool size) merely postpones system collapse. You must eradicate the Fault Mechanism.

### 1.4 Non-Destructive Observability
During live incidents or investigations, your diagnostic actions must never exacerbate system instability:
- Never execute unbounded queries (`SELECT * FROM table` without limits) on production replicas.
- Never attach blocking debuggers (e.g., `gdb`, `jdb`) to live production traffic without traffic shedding.
- Employ asynchronous sampling, dynamic log-level escalation, and read-only diagnostic tooling.

### 1.5 Preserve Forensic State
Before restarting services, cycling containers, or modifying state:
- Capture thread dumps, core dumps, heap dumps, or memory snapshots.
- Preserve the exact commit hash, environment variables (with secrets redacted), dependency lockfile states, and OS kernel metrics.

---

# 2. Phase 1 — Incident Triage & Blast Radius Assessment

When summoned to an incident or defect, rapidly establish the operational context:

### 2.1 Establish Incident Severity & Trajectory
1. **Severity Classification:**
   - **SEV-0 / Critical:** Complete service outage, data corruption, active security breach, or total revenue-path blockage.
   - **SEV-1 / High:** Severe degradation for a major subset of users; no viable workaround available.
   - **SEV-2 / Medium:** Partial degradation, localized failures, or performance regressions with existing workarounds.
   - **SEV-3 / Low:** Cosmetic, minor functional defect, or non-blocking edge case.
2. **Trajectory Assessment:**
   - Is the failure expanding (cascading failure, resource exhaustion)?
   - Is it stable/intermittent?
   - Is immediate mitigation (traffic shedding, feature flag toggle, rollback) required before deep root cause investigation?

### 2.2 Define Blast Radius & Differential Timeline
Map the exact coordinates of the failure:
- **Who:** All users, specific tenants, specific geographic regions, or specific client versions?
- **When:** Exact timestamp of inception (UTC). Correlate with:
  - Deployments, canary releases, or rollouts.
  - Infrastructure changes (DNS, routing, firewall, cert rotation).
  - External dependency disruptions (third-party APIs, cloud providers).
  - Data migration or scheduled cron jobs.
- **Where:** Specific microservices, container pods, database shards, or network hops.

---

# 3. Phase 2 — Forensic Telemetry Ingestion & Log Archaeology

Extract facts from the system runtime using structured log archaeology and metrics analysis.

### 3.1 Metrics Triangulation (USE & RED Methods)
- **RED Method (Requests, Errors, Duration):**
  - **Rate:** Has traffic spiked (DDoS, thundering herd, retry storm) or plummeted (upstream drop)?
  - **Errors:** What is the precise distribution of HTTP 4xx vs 5xx, gRPC error codes, or application exception classes?
  - **Duration:** Where did the latency shift? Check P50, P90, P99, and max. A widening gap between P50 and P99 indicates queueing delays or lock contention.
- **USE Method (Utilization, Saturation, Errors):**
  - **CPU:** High user CPU (runaway loops, regex catastrophism) vs high system CPU (excessive context switching, thread thrashing, kernel I/O contention).
  - **Memory:** Steadily climbing RSS (leak) vs sawtooth pattern with high GC pause times (allocation churn).
  - **Disk / Network I/O:** Queue lengths, socket buffer drops, disk IOPS saturation.

### 3.2 Distributed Tracing & Span Deconstruction
Inspect distributed trace timelines (OpenTelemetry, Jaeger, Datadog):
1. Identify the **critical path**: Where is time spent across parent and child spans?
2. Detect **unintended serialization**: Are sequential outbound calls being made inside loops instead of concurrent fan-out?
3. Detect **clock skew**: Are negative span durations or impossible causal sequences present due to unsynchronized NTP servers?

### 3.3 Log Archaeology & Correlation
- Group log occurrences by exact stack trace fingerprint.
- Correlate client-facing request IDs (`X-Correlation-ID`, `traceparent`) from edge gateway down through services to database logs.
- Search for the "First Sin": The first warning or error emitted before the deluge of cascading secondary errors.

---

# 4. Phase 3 — Deep Diagnostic Archetypes

Classify the defect into one of the following primary engineering archetypes and follow the dedicated diagnostic protocol:

---

### Archetype A: Concurrency, Race Conditions & Deadlocks

#### Symptoms
- Intermittent test failures in CI ("flakes").
- Thread deadlocks, hanging requests, thread pool exhaustion.
- State inconsistencies or double-spend anomalies under high concurrency.

#### Investigation Checklist
1. **Shared Mutable State:**
   - Trace all access paths to variables, in-memory caches, or global singletons accessed by multiple threads/coroutines.
   - Verify if reads and writes are guarded by identical synchronization primitives (mutex, lock, atomic).
2. **Lock Hierarchy & Inversion:**
   - Map the acquisition order of multiple locks:
     ```text
     Thread A: Acquires Lock 1 -> Wants Lock 2
     Thread B: Acquires Lock 2 -> Wants Lock 1  ==> DEADLOCK
     ```
   - Enforce a strict global lock acquisition hierarchy or transition to lock-free structures.
3. **Check-Then-Act Race Conditions (TOCTOU):**
   ```python
   # Anti-pattern: Non-atomic check-then-act
   if not cache.exists(key):
       cache.set(key, compute_heavy_value()) # Multiple workers compute concurrently
   ```
4. **Thread Dumps & Profiling:**
   - JVM: Inspect `jstack` output for `BLOCKED` threads waiting on object monitors or `java.util.concurrent.locks`.
   - Go: Trigger `pprof/goroutine` to detect leaked goroutines blocked on unbuffered channels.
   - Rust: Audit `unsafe` blocks, cross-thread message channels, and `Arc<Mutex<T>>` cycles.

---

### Archetype B: Memory Leaks, Buffer Overflows & GC Thrashing

#### Symptoms
- Process crashes with Out Of Memory (OOM) killed by OS kernel (`dmesg: Out of memory: Kill process`).
- Application throughput progressively degrades as garbage collection pauses grow longer.

#### Investigation Checklist
1. **Differentiate Leak Types:**
   - **Managed Runtimes (Java, Node, Python, Go):** Unintentional object retention (listeners never unregistered, unbounded in-memory caches, circular closures, module-level collections).
   - **Native Runtimes (C, C++, Rust):** Unfreed heap allocations, dangling pointers, buffer overflows, double frees.
2. **Heap Snapshot Differencing:**
   - Capture Heap Snapshot A (at baseline steady state).
   - Induce synthetic load or wait for duration $T$.
   - Capture Heap Snapshot B.
   - Perform a **retained size diff**: Identify which class or object type accounts for the delta in retained memory.
3. **Common Leak Vectors:**
   - Event listeners attached repeatedly without corresponding cleanup.
   - Caches lacking eviction policies (no LRU, TTL, or max bounds).
   - ThreadLocal variables never cleared across pooled worker threads.
   - Unclosed streams, database result sets, or file descriptors.

---

### Archetype C: Asynchronous Runtimes & Event Loop Starvation

#### Symptoms
- High response latency despite low CPU utilization.
- Healthcheck endpoints timing out even when the system is processing minimal traffic.
- Node.js, Python AsyncIO, or Twisted servers freezing intermittently.

#### Investigation Checklist
1. **Event Loop Lag Monitoring:**
   - Check event loop delay metrics (e.g., `libuv` lag in Node.js, `asyncio` debug mode).
2. **Synchronous Blocking on the Main Loop:**
   - Search for CPU-intensive synchronous operations executed inside the event loop:
     - `JSON.parse` / `JSON.stringify` on multi-megabyte payloads.
     - Synchronous crypto (`bcrypt.hashSync`, `crypto.pbkdf2Sync`).
     - Synchronous filesystem operations (`fs.readFileSync`).
     - Catastrophic Regular Expression backtracking (ReDoS).
3. **Unhandled Promise Rejections & Dangling Futures:**
   - Are async operations initiated without `await` or `.catch()`, causing silent failures or background resource exhaustion?
4. **Backpressure Neglect:**
   - Is a producer streaming data into an async pipe faster than the consumer can process, buffering unbounded chunks in RAM?

---

### Archetype D: Database Contention, Locking & Pool Starvation

#### Symptoms
- "Connection pool timeout: could not acquire connection within 5000ms."
- Database CPU spikes to 100% or transaction queues balloon.
- Cascading timeouts across all dependent backend services.

#### Investigation Checklist
1. **Connection Pool Leak vs Pool Starvation:**
   - Is every connection checked out explicitly closed/returned in a `finally` block or RAII guard?
   - Or are all connections actively executing long-running queries?
2. **Transaction Scope & Long-Running Transactions:**
   - Are network calls, email sends, or heavy computations performed *inside* active database transactions?
   - Rule: Keep database transactions as short as physically possible. Never perform external network I/O while holding a database transaction.
3. **Query Execution Plan Analysis (`EXPLAIN ANALYZE`):**
   - Sequential Scans on tables with millions of rows.
   - Missing composite indexes on multi-predicate WHERE clauses.
   - Bad cardinality estimates leading to nested loop joins instead of hash joins.
4. **Lock Contention:**
   - Postgres: Query `pg_stat_activity` and `pg_locks` for `granted = false`. Identify the PID holding the blocking lock.
   - MySQL: Inspect `INFORMATION_SCHEMA.INNODB_TRX` and `sys.innodb_lock_waits`.

---

### Archetype E: Distributed Systems Failures & Cascading Collapse

#### Symptoms
- Upstream failure in Service C triggers simultaneous outages in Service B and Service A.
- Recovered services instantly crash again upon restarting ("Thundering Herd").

#### Investigation Checklist
1. **Timeouts and Deadlines:**
   - Are client timeouts explicitly configured on all HTTP/gRPC/RPC clients?
   - Rule: Client timeouts must be shorter than upstream SLA thresholds.
   - Are deadlines propagated across service boundaries (OpenTelemetry Baggage / gRPC context)?
2. **Retry Storms & Exponential Backoff:**
   - Are retries executing immediately without jitter?
   - Anti-pattern: Fixed-interval retries that multiply traffic during an incident ($100 \times \text{clients} \times 5 \text{ retries} = 500\times \text{load}$).
   - Fix: Exponential backoff with full jitter and circuit breakers.
3. **Thundering Herd / Cache Stampede:**
   - When a popular cache key expires, do thousands of concurrent requests all hit the primary database simultaneously?
   - Fix: Mutex locking (single-flight), probabilistic early expiration, or stale-while-revalidate background refresh.
4. **Split-Brain & Partitioning:**
   - Verify distributed consensus state (Raft, Paxos, Zookeeper, etcd).
   - Are quorum rules strictly observed?

---

# 5. Phase 4 — Construction of the Minimal Reproducible Example (MRE)

A fix without a reproducible test is merely an assumption.

### 5.1 Rules for Constructing an MRE
1. **Strip Extraneous Variables:**
   - Eliminate unnecessary middleware, UI components, unrelated database tables, and third-party APIs.
   - Reduce the reproduction to the absolute minimal code required to exhibit the failure.
2. **Make it Deterministic:**
   - If the bug is timing-dependent, use artificial time-dilation, thread synchronization barriers (e.g., `CountDownLatch`, `sync.WaitGroup`), or mock virtual clocks.
   - If data-dependent, extract the exact minimal input payload that triggers the defect.
3. **Automate as a Test:**
   - Embed the reproduction in a standalone test case (unit or integration test).
   - The test must FAIL before the fix is applied.
   - The test must PASS after the fix is applied.

---

# 6. Phase 5 — Surgical Fix & Defensive Hardening

When writing the remediation, adhere to senior-grade software engineering standards:

### 6.1 Avoid Anti-Patterns in Fixes
- ❌ **Do not swallow errors:**
  ```javascript
  // BANNED: Symptom masking
  try { doSomething(); } catch (e) { /* ignore */ }
  ```
- ❌ **Do not apply blanket null checks without fixing the invariant:**
  ```typescript
  // BANNED: If user cannot be null according to domain invariants,
  // do not just return empty; find why an invalid state was permitted.
  if (!user) return;
  ```
- ❌ **Do not increase timeouts as a primary fix:** Increasing a timeout from 5s to 30s usually just creates a 30s queue before failure.

### 6.2 Defensive Hardening Guidelines
1. **Restore and Enforce Invariants:** Validate inputs at trust and domain boundaries using strict schemas.
2. **Fail Fast:** If an irrecoverable state occurs, fail immediately with informative context rather than propagating corrupt state downstream.
3. **Graceful Degradation:** When an auxiliary service fails, ensure the core application degrades gracefully (e.g., recommendations fallback to default popular items).
4. **Idempotency:** Ensure operations can be safely retried without side effects.

---

# 7. Phase 6 — Verification & Stress Testing

Prove that the remediation works under real-world conditions:

### 7.1 Multi-Dimensional Verification Matrix
1. **Functional Verification:** Run the automated MRE test; confirm it transitions from Red to Green.
2. **Regression Suite:** Run the entire existing test suite to ensure no regressions were introduced.
3. **Load / Concurrency Stress Test:** Subject the modified path to concurrent requests (e.g., via `k6`, `wrk`, or concurrent test runners).
4. **Fault Injection / Chaos Check:** Inject network delays, packet drops, or upstream errors to verify that error handling operates gracefully.

---

# 8. Phase 7 — The Blameless Post-Mortem & Incident Artifact

Document the findings in an executive, engineering-grade Root Cause Analysis report:

```markdown
# Incident Post-Mortem & Root Cause Analysis

## Incident Overview
- **Incident ID / Ticket:** INC-XXXXX
- **Severity Level:** SEV-X
- **Date & Time:** YYYY-MM-DD HH:MM UTC
- **Duration / MTTR:** XX minutes
- **Services Affected:** [Service Name, Endpoints, Databases]
- **Customer / Business Impact:** [Percentage of traffic impacted, failed transactions, financial/reputational impact]

## Executive Summary
A concise, non-jargon explanation of what occurred, why it occurred, how it was resolved, and what prevents recurrence.

## Incident Timeline (UTC)
- **HH:MM** - Anomaly introduced (e.g., release v1.4.2 deployed).
- **HH:MM** - First anomalous telemetry observed in APM.
- **HH:MM** - Automated alert fires; Incident Commander paged.
- **HH:MM** - Triage initiated; blast radius contained (e.g., traffic rolled back / circuit breaker tripped).
- **HH:MM** - Root cause identified via forensic tracing.
- **HH:MM** - Surgical patch deployed and verified.
- **HH:MM** - Incident terminated; steady-state metrics restored.

## The 5 Whys (Causal Chain)
1. **Why did the checkout service return HTTP 504?** Because all database connections in the pool were saturated.
2. **Why was the connection pool saturated?** Because queries against `inventory_reservations` took 8.5 seconds instead of 4 milliseconds.
3. **Why did the inventory query take 8.5 seconds?** Because it performed a sequential table scan across 12,000,000 rows.
4. **Why was a sequential scan performed?** Because the recent migration added a `WHERE tenant_id = ? AND sku = ?` filter, but the existing index was only on `sku`.
5. **Why was the index missing in production?** Because staging test databases had only 1,000 rows, allowing the query to pass CI performance budgets undetected.

## Root Cause (The Fault Mechanism)
Detailed technical description of the exact mechanical failure mode, including code snippets and architectural diagrams.

## Remediation & Changes Implemented
- [File/Component 1]: Description of change.
- [File/Component 2]: New index / lock restructuring / architectural change.

## Verification & Proof
- Reproduction test details.
- Benchmark comparisons before vs after.

## Corrective & Preventive Action Items (CAPA)
| Action Item | Type (Prevent / Detect / Mitigate) | Owner | Priority | Target Date |
|---|---|---|---|---|
| Add missing composite index | Mitigate | @engineer | P0 | DONE |
| Add CI query plan check for unindexed foreign keys | Prevent | @infra | P1 | YYYY-MM-DD |
| Enforce statement timeout of 2000ms on web pool | Detect | @dbre | P1 | YYYY-MM-DD |
```

---

# 9. Agent Operational Rules

When executing this skill:
- **Never hypothesize without verifying:** If you claim a function causes a deadlock, show the exact line numbers and conflicting lock sequence.
- **Never claim a bug is fixed without a test:** Always present the before/after execution result of a reproducible test.
- **Never paste unredacted credentials or user PII** from logs into the analysis output.
- **Preserve git history cleanliness:** Atomic commits for the fix and regression test.
