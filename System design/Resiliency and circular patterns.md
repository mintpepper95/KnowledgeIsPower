
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

#### Explain main idea of Circuit Breaker Pattern and its three states?
**Stop trying to call something that's failing**, give it time to recover, then cautiously retry.

**CLOSED** — normal operation, requests pass through. Monitors success and failure.

**OPEN** — failure threshold exceeded. All requests blocked from reaching failed service/resource (no network call made). A timer starts.

**HALF-OPEN** — timer expires, a small number of probe requests are allowed through. If they succeed → back to CLOSED. If they fail → back to OPEN.

---
#### Circuit Breaker in details

Polly is a resilience library with multiple policies: retry, circuit breaker, timeout, bulkhead isolation etc.

##### Difference between retry and circuit breaker
**Retry:** "Request failed - try again" 
It assumes the failure is transient. But this is dangerous on its own, because if the downstream service is truly broken, retries pile up, amplify load, and can cascade failures across your whole system.

**Circuit breaker:** "Service failed too many times - stop trying" 
It watches the _pattern_ of failures, and when a threshold is exceeded, it starts _rejecting requests immediately_ without even attempting the call. It protects both the caller and the downstream service.

Imagine Service A calls Service B, and Service B is down. Without a circuit breaker:
- Every request to A tries to call B -> A use up all of its threads trying to call B, A's threads wait for B's timeout
- A eventually has no free thread to handle incoming request
- A becomes slow, causing cascade effects to services C, D, E that depend on A

With a circuit breaker, once B trips the breaker, A's calls fail fast (in microseconds, not seconds). A's threads stay free, its queue stays clear, it can still serve other requests. The damage is _contained to the B→A boundary_ rather than cascading upward. That's blast radius containment — not retry/delay, but _fail-fast isolation_.

##### Dotnet example

Here circuit breaker wraps the retry. This means: the breaker monitors _logical outcomes_ (did the user's intent succeed or fail?), not individual attempt counts. Three retries that all fail = one failure from the breaker's perspective. If you reverse the order, every retry attempt is counted separately, making the breaker trip too eagerly on a single transient hiccup.

`MinimumThroughput` prevents flapping. If the service just started and 2 of 2 calls failed, that's 100% failure ratio but statistically meaningless. The threshold ensures you have enough signal before tripping.

```cs
// Recommended: retry inside circuit breaker
pipeline.AddCircuitBreaker(new CircuitBreakerStrategyOptions {
    MinimumThroughput = 5, // for 5 calls 
    FailureRatio = 0.6, // 60% of 5 calls fail 
    SamplingDuration = TimeSpan.FromSeconds(10), // in 10 secs
    BreakDuration = TimeSpan.FromSeconds(30), // open for 30 secs
    ShouldHandle = new PredicateBuilder() // what count has failure
    .Handle<HttpRequestException>()
});

// Retry sits inside the breaker
pipeline.AddRetry(new RetryStrategyOptions {
	MaxRetryAttempts = 3,
    Delay = TimeSpan.FromMilliseconds(200),
    BackoffType = DelayBackoffType.Exponential,
    UseJitter = true,
    ShouldHandle = new PredicateBuilder()
    .Handle<HttpRequestException>()
});

// ┌─ CircuitBreaker ────────────────┐
// │  ┌─ Retry ─────────────────┐    │
// │  │   actual HTTP call      │    │
// │  └─────────────────────────┘    │
// └─────────────────────────────────┘
```

Use retry alone when failures are almost always transient — a DNS blip, a brief network partition, a rate-limit response where you should back off. Typically fine for idempotent operations like reads.

Use a circuit breaker when the downstream service can be genuinely broken for extended periods, or when protecting your own throughput matters. Essential for writes and payment flows where a degraded experience is better than a hung request.

Use both together for most production HTTP calls to external services — the retry handles transient noise, the breaker handles sustained outages.

#### Key Configuration Knobs

These are the things you'd actually tune in production, and interviewers love to probe this:

| Parameter                 | What it controls                     | Tradeoff                                     |
| ------------------------- | ------------------------------------ | -------------------------------------------- |
| **Failure threshold**     | How many failures before opening     | Too low = flapping; too high = slow to react |
| **Time window**           | Rolling window for counting failures | Short = reactive; long = stable              |
| **Open timeout**          | How long to stay open before probing | Too short = hammering a sick service         |
| **Half-open probe count** | How many requests to test with       | Too many = risk re-overloading               |
| **Failure definition**    | Timeout? 5xx? Both?                  | Misconfigured = wrong signal                 |


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