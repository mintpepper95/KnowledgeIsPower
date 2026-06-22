# WhatsApp / Chat — Attempt 1

My design for a senior/staff-level chat system, followed by feedback organised by theme.

## My design

This is the design I actually drew:

![[Pasted image 20260622021525.png]]


---

## TL;DR — the headline critiques

In rough priority order, this is what a staff interviewer flags about the design:

1. **Live delivery is serialised through durable storage.** An online user with an open socket should get a message in tens of ms, but the diagram makes delivery wait on Kafka → processor → DB. Real-time delivery and durable persistence are two different latency budgets — **split them**.
2. **There's no explicit push-to-online-user path.** Nothing in the diagram shows *which* component pushes down the socket, or how a message finds the connection server holding the receiver. The delivery/connection tier is missing.
3. **The Kafka / Notification arrows are tangled and there are two Kafkas.** Collapse to one durable log partitioned by chat ID that both persistence and delivery consume from.

Everything below expands on these and the supporting concepts.

---

## 1. Separate live delivery from durable persistence

**This is the big one.** For a user who is *online with an open WebSocket in the active chat*, a message should arrive in tens of milliseconds. In the current design, delivery to that online user depends on the message first being written to Kafka, picked up by the processor, written to the DB, and only then surfaced — i.e. real-time delivery is coupled to durable persistence. The diagram reads as *persist-then-deliver, serially*.

The fix is the **dual-path** pattern. The moment the Send Service accepts a message, it does two things in parallel:

- **Live path (fast, best-effort):** publish directly to the pub/sub → connection-server layer so anyone online in that chat gets it in ~ms. A drop here is fine — the client reconnects and replays from the durable log.
- **Durable path (correct, guaranteed):** write the message + an outbox row to the DB; CDC → Kafka → persistence / offline-notification / read-model. This is the source of truth and the backfill mechanism an offline user reconciles against on reconnect (via last-seen message ID).

> **Why this matters:** *"Losing a message is bad; showing one twice is recoverable."* The live path optimises latency, the durable path optimises correctness, and idempotency makes the overlap between them harmless.

*Corrected send path (from the feedback):*

![[Pasted image 20260622104635.png]]

*Corrected offline / multi-device reconciliation (from the feedback) — how a returning device catches up against the durable log:*

![[Pasted image 20260622104715.png]]

### Where the Read DB / CQRS fits

The separate Read DB (via CDC) is **defensible but misplaced if it sits in the live path**.

- **What CQRS-via-CDC buys you:** a read-optimised replica (denormalised for "fetch my unread", "load room history", with different sharding/indexing) so heavy reads don't contend with the write path. Legitimate for read-heavy systems.
- **The cost:** CDC is asynchronous, so the Read DB is **eventually consistent** — it lags the write DB by however long CDC takes (tens to low-hundreds of ms).
  - For **offline/history backfill** ("show unread since last login"), that lag is invisible — fine.
  - For **live delivery to the active chat**, that lag is unacceptable.

So: keep the Read DB for the **offline-history and unread-backfill path only**, explicitly labelled as eventually-consistent and *off* the live path. Deliver live messages out-of-band via pub/sub before they ever hit the read replica.

> The original "outbox + background service + Read DB via CDC" relay is built for reliably propagating events to *downstream systems*, **not** for delivering a message to a person who is waiting for it — it adds latency on the live path.

---

## 2. The delivery/connection tier

The diagram doesn't show which component pushes down the socket, or how a message knows which connection server holds the receiver. Because WebSockets are **stateful, long-lived connections**, a user is tethered to one connection node out of many. You need a way to answer: *"which connection server holds receiver 4567 right now?"*

There are **two mechanisms** — pick one as primary; the original design reached for both at once.

### Approach A — deterministic placement (hash/partition by ID)
The connection server is *computed*, not looked up: `hash(userId)` or `hash(chatId)` → server N. Anyone who needs to reach the user runs the same hash and knows where to push. No registry needed — the location is *derivable*. Weakness: rebalancing on node add/remove, and hot nodes for hot chats.

### Approach B — registry / directory (the Redis map)
Connections land on *whatever* server (e.g. via a load balancer); each server records "user 4567 is on me, subscribed to rooms X, Y" into a shared store (Redis). To reach a user you *look them up*. Weakness: extra lookup hop, and the registry must be kept fresh.

### Presence & session routing
- **Session routing:** when a message arrives, the backend asks a fast store (Redis) which connection node holds User B, so it can route the push to the right machine.
- **Presence Service:** listens to client heartbeats to track online/offline, and (in Approach B) maintains the user→node map. A distributed chat system can't function without knowing where users are and whether they're online.

### WebSocket multiplexing
Run multiple independent streams over a **single** WebSocket connection rather than one connection per conversation. The HTTP handshake happens once → lower overhead. Clients subscribe/unsubscribe to specific topics, and a front-end dispatcher routes incoming events to the right chat view.

> Multiplexing over one socket is the right call. The weakness in the original design was **HTTP polling for non-active chats** — a user in 20 conversations would have their device constantly firing HTTP requests, draining battery and loading the servers. Prefer push over the multiplexed socket.

---

## 3. Message ordering & IDs

With distributed servers each having their own clock, **clock skew** means two servers can disagree by tens to hundreds of ms. If we stamp messages with `DateTime.UtcNow`, order can be wrong — and timestamps aren't guaranteed unique anyway.

Key realisation: for chat we only need **correct order *within* a single conversation**, not globally.

- **Route a chat to one partition** (`hash(chatId)` → partition/shard). All messages for that chat land in order on the same partition — the partition *is* the ordered log.
- **Per-conversation sequence number** (1, 2, 3…) gives a definitive intra-chat order and lets the client **detect gaps** and trigger backfill. This is the ordering authority, *not* wall-clock time.

> **Why not just pin a chat to one server?** You can, but a single server per room causes a **hot-partition** problem under load, and if it dies the room is dead until the system detects the crash, elects a new owner, reroutes traffic, and reloads chat state. Partitioning the durable log is the more robust version of the same idea.

### Snowflake ID — what it does and doesn't solve

- **Does NOT solve clock skew across servers.** It solves the narrower problem of *two messages in the same millisecond on the same server* colliding, by adding a per-machine sequence counter.
- **Time-prefixed, so roughly sortable.** Unlike a random GUID, a Snowflake/ULID embeds a timestamp (ms since an epoch). Without it you'd need *both* a `uuid` for uniqueness *and* a per-conversation counter for ordering.
- **The biggest practical win is physical write performance.** The `messages` table holds all messages from all conversations, keyed by message ID. A random GUID (UUIDv4) PK lands every insert at a random spot in the B-tree → page splits, fragmentation, poor cache behaviour. A time-prefixed ID makes inserts *roughly monotonic* (append near the end of the index). This is a real, measurable win — and it's about the *physical table*, not user-visible order. It's the single biggest reason people pick Snowflake/ULID over random GUIDs.

---

## 4. Idempotency & at-least-once delivery

**An operation is idempotent if doing it multiple times produces the same result as doing it once.** Pressing an elevator button five times → it still comes once (idempotent). Withdrawing $100 five times → you're out $500 (not idempotent).

This matters because **the network is unreliable and things get retried**. A sender often can't distinguish "my request failed" from "it succeeded but the ack was lost", so the safe-for-delivery choice is to retry — which means operations sometimes run twice. This is exactly **at-least-once** delivery: *"I guarantee it arrives, but maybe more than once."* The price of that guarantee is duplicates; **idempotency is how you pay that price without the user seeing double.**

> Note on **CAP**: CAP only applies *during a partition*. Message delivery is "deliver now, reconcile order later" — but message *ordering within a chat* and *idempotency* push you toward needing **per-partition consistency**.

### Where duplicates arise in *this* design

It's not one place — every hop is a retry point:

- **Client → server send.** Network drops the *response*; the client's "failed to send" logic retries → server receives the same message twice.
- **Kafka consumers (at-least-once).** A processor reads, processes, then crashes *before committing its offset* → on restart it reprocesses → written to DB / published downstream twice.
- **Outbox + CDC relay.** The relay can publish a row and crash before marking it done → republishes on restart. At-least-once by construction.
- **Live delivery push.** A message pushed live, then the client reconnects and *also* backfills from the durable log by last-seen ID → it sees the same message via both paths.

So duplicates can enter at send, persistence, relay, and delivery. **Idempotency is the cross-cutting defence at each layer** — it's not a missing box, it's a missing *property* the whole pipeline needs.

### The core tool: two IDs with distinct jobs

The crucial move people miss: **the client mints a stable ID *before* the first send and reuses it across retries.** If the *server* generates the ID on arrival, a retry looks brand-new → duplicate. So:

- **Client-generated idempotency key** (UUID, made before first send) → for **deduplication** across retries. Stable across attempts.
- **Server-generated message ID** (Snowflake/UUIDv7, assigned on first acceptance) → for **ordering, storage PK, pagination cursor**. Assigned once, after dedup.

On a retry, the server looks up the idempotency key, finds it processed, and returns the *original* server message ID — same outcome, no duplicate stored.

### Mechanism at each layer

- **Send (client → server):** keep a store of recently-seen idempotency keys (Redis with TTL) *or* — the bulletproof version — a **unique constraint** on `client_message_id` in the DB. On send: check key → return original result if seen, else process and record. Under a race, the second insert simply *fails the constraint* (insert; on unique-violation, fetch the original).
- **Kafka consumers (processor / persistence):** make the *write* idempotent. Because the message carries a stable ID, "insert with this ID" via **unique constraint / upsert** makes reprocessing a no-op. So even though Kafka is at-least-once, the *effect* on the DB is **exactly-once**. → *"You can't get exactly-once delivery, but you get exactly-once **effect** via idempotent writes."*
- **Delivery (server → client):** the client dedupes on display by tracking rendered message IDs. If the same ID arrives twice (live + backfill), it's shown once. This is what lets live delivery stay best-effort.

### The interview one-liner

> *"I chose at-least-once delivery throughout, so I need idempotency to make it safe: a client-generated idempotency key per message that's stable across retries, server-side dedup on that key, idempotent (unique-constraint/upsert) writes so reprocessing is a no-op, and client-side dedup on message ID for display. That gives exactly-once **effect** on top of at-least-once **delivery**."*

And the combined **idempotency + ordering** script (answers idempotency, ordering, indexing, and clock skew in one breath):

> *"Client generates a `client_msg_id`. Chat Service dedups on `(chat_id, client_msg_id)` so network-blip retries don't duplicate. It then assigns a monotonic per-chat sequence — that's my ordering authority, not wall-clock time, which dodges clock skew. Kafka partitioning by `chat_id` preserves that order through the broker, and the client uses the sequence to detect gaps and trigger backfill."*

---

## 5. Persistence & event flow: Outbox, CDC, Kafka

### Outbox vs CDC — not the same thing

- **Outbox** is a *data-modelling* pattern: write your business data **and** the event you want to emit in **one atomic DB transaction** (both rows in the same DB). Solves the dual-write problem.
- **CDC** is a *mechanism* for reading changes out of a DB — typically by tailing its internal log (MySQL binlog / WAL) — and emitting a stream of updates.

They compose:

- **Outbox + CDC:** CDC emits new outbox rows to Kafka instead of polling. CDC is a more efficient relay than polling, and the outbox lets you write *exactly the event you want to emit*, decoupled from table layout (raw CDC emits everything).
- **CDC without Outbox:** point CDC straight at the DB tables and treat each row change as an event. Simpler, but you emit your raw schema.

### The simplification: drop the message processor

The current diagram does **Kafka → processor → DB**, meaning Kafka receives messages that *aren't yet durably stored* — a processor crash between Kafka and DB risks loss. Flip it:

- Send Service writes message **+ outbox row** to the Message DB in one transaction (atomic, no dual-write).
- **CDC tails the DB** and relays committed outbox events to Kafka. The message processor *disappears* because CDC **is** the relay now — and the "write to Kafka then re-read and write to DB" round-trip goes away.
- Kafka (partitioned by chat ID) is consumed by whatever delivers / notifies / builds the read model.

This is the **transactional outbox + CDC** pattern done properly: DB commit is the single source of truth, events are *derived* from the commit, at-least-once downstream with idempotent (dedupe-by-ID) consumers.

> **The one gap it leaves:** CDC adds latency, so if live delivery is driven *solely* by CDC→Kafka you've reintroduced critique #1 in milder form. Pair it with the fast live path from §1.

### Consolidate the two Kafkas

The diagram has two ("Kafka" under Send Service and "kafka message queue" up top) with tangled arrows between API gateway, Notification Service, and the queue. Collapse to **one logical durable log, partitioned by chat ID**, that both persistence and delivery consume from.

> **Worth saying out loud (you got this right):** shard Send/Notification by chat ID, one chat → one Kafka partition for in-conversation ordering, competing consumers per partition, DB sharded by chat ID. Partition-per-chat *also* gives per-conversation ordering for free — a deliberate two-birds choice.

---

## 6. Fan-out, hot partitions & thundering herd (group chats)

A popular room with 100k members: one message arrives and must be delivered 100k times.

### Fan-out strategy
- **Fan-out on write** — push the message into each recipient's inbox/message table. Good for small rooms; cheap to read, expensive to write.
- **Fan-out on read** — store the message once and let clients pull it when they load the room. Cheap to write, more work on read. Better for large rooms.

### Hot partition
A 50k-member group keyed to one partition is a hotspot. Mitigate by bounding per-group chat throughput.

### Connection layering (don't push 100k sockets from one box)
Have the message hit a tier of **gateway servers**, each owning a subset of connections, fed by a **pub/sub bus** (Redis) — each edge server delivers only to its own connected clients. (See §7 for the flow.)

### Connect storms
When many clients connect at once (e.g. a popular stream goes live):
- **Jittered exponential backoff** — clients retry with randomness.
- **Load shedding** — server queues / rejects excess connection attempts.

### Serving popular rooms from cache
CDNs cache static/public assets, but chat history changes constantly and is per-user authorised, so we cache it ourselves (Redis sorted sets). On room open, serve recent history from Redis instead of the DB.
- **Cache-aside on miss:** if `room:1234:recent` is absent, load last N from DB, populate Redis, return.
- **Single-flight on a stampede:** if the cache expires and 50k requests miss at once, only the first rebuilds the cache; the rest wait for it.

---

## 7. Live delivery transport: Pub/Sub vs Redis Streams vs Kafka

### Pub/Sub flow

```
Client A ──ws──> Gateway 3 ──> [publish msg to room:1234] ──> PUB/SUB BUS
                                                                  │
                          ┌───────────────────┬─────────────────┤
                          ▼                   ▼                 ▼
                      Gateway 1           Gateway 7         Gateway 42
                  (subscribed to       (subscribed)       (subscribed)
                   room:1234)              │                  │
                      │                    │                  │
              push to its local      push to its       push to its
              sockets in that room   local sockets     local sockets
```

**With Redis Pub/Sub** (for live delivery to connected clients):
- Each gateway runs `SUBSCRIBE room:1234` when a client joins that room.
- The sending gateway does `PUBLISH room:1234 "<message json>"`.
- Redis delivers the payload to every subscribed gateway; each loops over *its own* local sockets for that room and writes to them.
- One publish → Redis fans out to the ~50 gateways → each does its local fan-out. The sender never knew how many gateways/clients existed.

**With Kafka:** same shape, different trade-off — the room (or a hash of it) maps to a topic/partition and gateways are consumers. The big difference: **Kafka persists the log** (append-only on disk); Redis Pub/Sub does not.

### Fault tolerance — Pub/Sub is fire-and-forget
If a gateway has a network blip during a publish, it **never sees that message**, and Redis snapshots don't replay missed Pub/Sub messages. So:
- **Pub/Sub is fine for live push but can NOT be the source of truth.** Correctness comes from the DB on reconnect — live delivery is best-effort.
- For durability + replay, use **Redis Streams** (persist and support replay) or Kafka.

### Cache ↔ DB consistency on the hot path
- **Write-through:** when a message arrives you already persist to DB and push to the room — add a third step: append to the room's recent-cache (`LPUSH room:1234:recent`, trim to N). Because chat is **append-only**, the cache is just "the tail of the log", so appending keeps it correct *without* re-reading the DB. Append-driven caches don't drift the way caches of mutable rows do.
  - They barely need a TTL — use TTL only to **evict cold rooms** no one is reading. Add **probabilistic refresh before expiry** so a batch of entries doesn't all expire on the same tick.
- If the DB write succeeds but the cache write fails, you just miss one most-recent message — fine, the next write fixes the cache.

### Redis Streams vs Kafka — choosing
A **stream partitioned by chat ID** collapses three concerns into one:
- durability + replay (an offline consumer resumes from its last offset),
- per-conversation ordering (the partition *is* the ordered log; the offset is the order),
- decoupling the sender from delivery fan-out.

A common combo: **Redis Streams as source of truth + ordering, Pub/Sub for the final fan-out** to WebSocket servers — if a Pub/Sub message drops, the client reconnects and replays from the stream.

Caveat: with very many rooms, one stream per chat is a lot of keys, and Redis (and its replay history) is **memory-bound**. Kafka has fixed partitions, so no partition-per-room explosion.

| Scenario | Choice |
|---|---|
| Small/medium scale, want simplicity, short replay window | **Redis Streams**, partition by chat (or hash chats to a set of streams) |
| Large scale, long retention, many rooms | **Kafka**, `hash(chatId)` → partition, ordering preserved within partition |

---

## 8. Data model choices

### SQL vs NoSQL
They're pretty similar these days — even NoSQL does transactions now — so talk about the **access pattern** instead: *append to a partition, read recent N by sort key, occasional scan back.* NoSQL fits that well, but plenty of chat systems run fine on SQL (Discord started on SQL and later moved to NoSQL).

### Unread messages — use a watermark
Scanning the DB for an `is_read` flag is inefficient: 500 received messages → 500 updates → massive locks. Instead keep a small **watermark** table `[user_id, chat_id, last_read_message_id]`. When the user opens a chat, do **one** update to `last_read_message_id`. Unread = everything after the watermark.

### Do we need event sourcing?
Not required. But the chat message log is **append-only**, and features like edit/delete history, read receipts, and multi-device sync can be naturally **modelled as events over time** — so it's a reasonable lens even without going full event sourcing.

---

## What I'd change on the diagram (staff-level pass)

1. Add an explicit **connection/delivery tier** (the WS-holding gateways + a pub/sub layer or connection registry) and show the arrow that actually *pushes* to the online receiver.
2. Make Send Service publish to the **live path and durable path in parallel**, not serially through DB-then-CDC.
3. Collapse the two Kafkas into **one durable log partitioned by chat ID**, and replace the message processor with **outbox + CDC**.
4. Keep the **Read DB / CQRS for offline-history and unread-backfill only**, explicitly labelled eventually-consistent and off the live path.



Send path
![[Pasted image 20260623021707.png]]


Receive path
![[Pasted image 20260623021728.png]]

![[Pasted image 20260623022639.png]]