

### Why Resiliency Matters

In a distributed system, **partial failure is the norm, not the exception**. Any network call can:

- Fail outright
- Hang indefinitely
- Succeed slowly enough to cascade

The goal of resiliency patterns is to **contain blast radius** — stop one failing service from taking down everything that depends on it.

---

### The Core Problem: Cascading Failure

```
User → Service A → Service B → Service C (slow/down)
                                    ↑
                             threads pile up here
                             waiting for timeout
```

Service A's thread pool fills up waiting on B. B's fills up waiting on C. The whole chain becomes unresponsive even though only C is degraded. This is called a **cascading failure** or **thundering herd**.

---

### The Circuit Breaker Pattern

Borrowed from electrical engineering. The idea: **stop trying to call something that's failing**, give it time to recover, then cautiously retry.

#### The Three States

```
              failure threshold                   probe request
  CLOSED ──────────────────────► OPEN ──────────────────────► HALF-OPEN
    ▲                                                               │
    │                        success                               │
    └───────────────────────────────────────────────────────────── ┘
                             fail → back to OPEN
```

**CLOSED** — normal operation, requests pass through. Failures are counted.

**OPEN** — failure threshold exceeded. All requests **fail immediately** (no network call made). A timer starts.

**HALF-OPEN** — timer expires, a small number of probe requests are allowed through. If they succeed → back to CLOSED. If they fail → back to OPEN.

---

#### Key Configuration Knobs

These are the things you'd actually tune in production, and interviewers love to probe this:

|Parameter|What it controls|Tradeoff|
|---|---|---|
|**Failure threshold**|How many failures before opening|Too low = flapping; too high = slow to react|
|**Time window**|Rolling window for counting failures|Short = reactive; long = stable|
|**Open timeout**|How long to stay open before probing|Too short = hammering a sick service|
|**Half-open probe count**|How many requests to test with|Too many = risk re-overloading|
|**Failure definition**|Timeout? 5xx? Both?|Misconfigured = wrong signal|

---

### Related Resiliency Patterns

Circuit breaker rarely works alone. At staff level you should know the **full toolkit** and when to reach for each:

#### Retry with Backoff

Automatically retry failed requests, with increasing delay.

- **Exponential backoff**: wait 1s, 2s, 4s, 8s...
- **Jitter**: add randomness to prevent **retry storms** (all clients retrying in sync)
- Critical rule: **only retry idempotent operations**. Retrying a payment = double charge.

#### Timeout

Set a hard deadline on every outbound call. Without it, a slow dependency holds your threads forever.

- Set at **multiple levels**: connection timeout, read timeout, overall request timeout
- Should be **lower than your caller's timeout** — otherwise your caller gives up before you do, orphaning work

#### Bulkhead

Isolate resources so one consumer can't exhaust them all. Named after ship hull compartments.

- Give Service B calls their **own thread pool**, separate from Service C calls
- If C hangs and fills its pool, B's pool is unaffected
- In practice: separate connection pools, separate executor services, separate queue limits

#### Fallback

Define what to do when the primary path fails:

- Return **cached data** (stale but available)
- Return a **default/degraded response** ("recommendations unavailable, showing popular items")
- **Shed load** gracefully with a meaningful error

#### Rate Limiting / Load Shedding

Protect yourself from being the bottleneck for others. Shed excess load deliberately rather than falling over.

---

### How They Compose

At staff level, the key insight is these patterns **layer together**:

```
Incoming request
       │
  [Rate Limiter]          ← shed load before it hits you
       │
  [Bulkhead]              ← isolate downstream calls
       │
  [Circuit Breaker]       ← stop calling dead services
       │
  [Timeout]               ← bound how long you'll wait
       │
  [Retry + Jitter]        ← handle transient failures
       │
  [Fallback]              ← graceful degradation
```

Each layer handles a different failure class. Missing any one creates a gap.

---

### Staff-Level Depth: What Interviewers Actually Want

Beyond the mechanics, be ready to discuss:

**Observability** — A circuit breaker you can't observe is dangerous. You need metrics on state transitions, failure rates, latency percentiles (p99, not just averages). Knowing _when_ a breaker opened and _why_ is essential.

**The "thundering herd" on recovery** — When a circuit closes after an outage, every backed-up client retries at once. Half-open with limited probes + jitter on retry helps, but this is still a real operational risk.

**Distributed circuit breakers** — If you have 50 instances of Service A, each has its own in-memory breaker. They don't share state. A service might be open on 10 instances and closed on 40. Solutions: centralized state (Redis), or accept the inconsistency as a feature (gradual rollout of recovery).

**Testing resilience** — **Chaos engineering** (Chaos Monkey, Gremlin) — deliberately inject failures in production (or staging) to verify your resiliency mechanisms actually work. Breakers that have never been tested are assumptions, not guarantees.

**Business logic vs. infrastructure concerns** — Circuit breakers should live in infrastructure/middleware, not scattered in business logic. Frameworks like **Hystrix** (legacy), **Resilience4j** (JVM), **Polly** (.NET), or service meshes like **Istio** handle this at the platform level.

---

### The Key Tradeoff to Articulate

> **Resiliency patterns trade availability for consistency.**

A fallback returning stale data means the system stays _up_ but returns _possibly wrong_ answers. Whether that's acceptable is a **product and business decision**, not just a technical one. Staff engineers are expected to drive that conversation, not just implement the pattern.