
### Dead Letter Queue

#### Queue vs Topic
Queue is one producer one consumer.

Topic is one producer many consumers. Each consumer gets their own copy of the message.
#### What is a Dead Letter Queue
Special queue/topic where messages go when they can't be processed successfully after retries, so messages aren't lost. It's setup on the consumer side. 

Can think of DLQ as a temp holding area for hold things like poison pills (malformed payload), messages that trigger business logic exceptions, messages with bad schema etc.

```
Normal Flow:
─────────────────────────────────────────────────────────
  Broker          Consumer        Downstream
  [event]   ──→   [process]  ──→  [success ✅]


Failure + Retry Flow:
─────────────────────────────────────────────────────────
  Broker          Consumer
  [event] ──→  [process] ──→ fails ❌
               [retry 1]  ──→ fails ❌  (backoff: 1s)
               [retry 2]  ──→ fails ❌  (backoff: 2s)
               [retry 3]  ──→ fails ❌  (backoff: 4s)
                               ↓
                          [DLQ] 🗄️   ← parked here for inspection
```

#### When to use a DLQ and when not to use a DLQ

* You use it for any consumer consuming business critical events. Without a DLQ, bad message is either discarded to allow messages to keep flowing in or blocks other messages

* It also helps investigating why message processing failed

* Don't use for non-critical messages like metrics, telemetry, hear-beats etc

* Don't use it if ordering is critical. Replay from a DLQ means event will arrive out of order. Naively replay from DLQ may corrupt state


#### DLQ Best Practices

- **Always include metadata** — original topic, retry count, failure reason, timestamp, stack trace. You need this to debug.
- **Alert on DLQ depth** — a growing DLQ is a production incident waiting to happen. Monitor it.
- **Build a replay mechanism** — a DLQ with no way to replay is a graveyard, not a safety net.
- **Set a retention policy** — DLQ messages shouldn't live forever; define when to discard vs escalate.
- **Consider per-service DLQs** — one global DLQ becomes a mess. Each consumer/service should own its own.

