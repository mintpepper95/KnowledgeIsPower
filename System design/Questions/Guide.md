# WebSockets in System Design

How real-time connections are handled at scale (Discord / WhatsApp / Slack style), top to bottom: why they're special → where they sit → what the servers do → auth → routing → rate limiting.

> **The one idea behind all of it:** HTTP is stateless (any server can answer any request), but a WebSocket is a **stateful, long-lived TCP connection pinned to one specific server**. Almost every design decision below exists to work around that single fact.

---

## 1. Why WebSockets are different from HTTP

- **Standard HTTP is stateless.** A request arrives, gets forwarded to *any* available backend, returns a response, and is forgotten.
- **WebSockets are persistent.** They begin as an HTTP request with an `Upgrade: websocket` header, then become a long-lived TCP connection that must be held open in **one server's memory** for minutes or hours.

Because the connection lives in a specific server's memory, that user is physically *pinned* to that one instance — which creates routing problem.

---

## 2. Topology: where WebSocket servers sit

The architecture branches at the **Edge Load Balancer**, which sits at the very front of your network (a single public IP). The API Gateway and the WebSocket fleet sit **parallel** to each other — the LB is *not* behind the gateway.

```
                 ┌──────────────────────┐
   Client ──────▶│  Edge Load Balancer  │  (AWS LB, CLoudFlare, GCP LB)
                 └──────────┬───────────┘
              /api/*        │        /ws/*
        ┌─────────────┐     │     ┌──────────────────────┐
        │ API Gateway │◀────┴────▶│ WebSocket Server Fleet│
        └──────┬──────┘           └──────────────────────┘
               │ auth, rate limit,
               ▼ forward
        Stateless microservices
```

- **HTTP (`/api/*`)** → API Gateway → handles auth, rate limiting, forwarding to stateless microservices.
- **WebSocket (`/ws/*`)** → routed *directly* to the WebSocket fleet, **bypassing the API Gateway**.

**Why bypass the gateway?** Modern managed gateways (AWS API Gateway, Apigee X) *can* do WebSockets, but at massive scale, forcing millions of idle, long-lived connections through a heavy, feature-rich gateway is prohibitively expensive and limits your control over connection lifecycles, load balancing, and broadcasting. Bypassing it for real-time traffic is a standard staff-level optimisation. Instead you deploy a dedicated fleet of **WebSocket Servers** (a.k.a. Connection Handlers / Edge Servers).

---

## 3. The WebSocket server's job

These servers are deliberately **lightweight**. They don't do heavy business logic, DB writes, or user auth. Their sole job is to:

- hold hundreds of thousands of idle TCP connections open efficiently (non-blocking I/O), and
- act as the persistent bridge between client devices and your internal backend services.

---

## 4. Authentication: the handshake upgrade

Because the initial handshake is **pure HTTP** (before the permanent TCP connection exists), you can validate the user *before* upgrading.

### Ticket-based authentication (industry standard)

Rather than make the WebSocket server do heavy crypto (parsing JWTs, hitting an Auth DB) while juggling millions of connections, use a short-lived "ticket":

1. **Request a ticket:** client makes a normal HTTP POST to an Auth Service **via the API Gateway**.
2. **Issue ticket:** Auth Service verifies credentials, generates a random, single-use token, stores it in Redis with a ~60s TTL (`ticket:xyz → user_id:123`), and returns it.
3. **Connect with ticket:** client connects to the Edge LB (bypassing the gateway) and hits the WS server with the ticket in the query string: `wss://ws.example.com/?ticket=xyz`.
4. **Validate & consume:** WS server checks Redis. If valid → upgrade the connection, **delete the ticket** (single-use), and establish the mappings. If invalid → reject the upgrade with `401 Unauthorized`.

---

## 5. Routing a message between two users

### The problem

A connection is pinned to one instance, so a server only knows about *its own* clients:

- **User A** connects → assigned to **WS Server #1**.
- **User B** connects → assigned to **WS Server #99**.
- User A sends a message for User B → it arrives at **#1**, which has no idea where (or whether) User B is online.

### The fix: a Redis directory + internal broker

A global, low-latency directory tracks **which user is on which server**. The end-to-end flow:

1. **Register on connect:** when User B connects to #99, the server writes `user_id:B → ws_server:99` to Redis.
2. **Ingest:** User A sends "Hello!" through their socket to #1.
3. **Look up:** #1 queries Redis — *"which server holds User B?"*
4. **Route internally:** Redis replies `ws_server:99`. #1 forwards the payload to #99 over an **internal broker** (Redis Pub/Sub, Kafka, or direct gRPC) — *not* over the public internet.
5. **Deliver:** #99 finds User B's socket in its local memory and pushes the message down the pipe.
6. **Clean up on disconnect:** when User B drops, #99 detects the closed socket and deletes `user_id:B` from Redis.

### Network-layer vs application-layer routing

Once a WebSocket is open, it does **not** use path-based routing — every payload (chat, typing indicator, read receipt) travels the same TCP pipe. So routing happens at two levels:

- **Initial connection (network layer):** the Edge LB routes the connection URL (`wss://example.com/connect`) to any healthy node, typically **least-connections**.
- **In-flight messages (application layer):** the WS server reads an action/type field in the payload (e.g. `{"action": "send_message", "to": "user_B"}`) and uses the Redis mapping / internal broker to route it (steps 3–5 above).

---

## 6. Rate limiting: two tiers

The attack vectors differ before and after the connection is established, so rate limiting splits in two.

| | **Tier 1 — Connection** | **Tier 2 — Message (in-flight)** |
|---|---|---|
| **Protects against** | Thundering herd / DDoS of connection attempts | A client spamming messages down an open pipe |
| **Where it lives** | Edge Load Balancer / CDN (Cloudflare, AWS Shield/ALB) | Inside the WebSocket server's memory |
| **How it works** | Limits HTTP Upgrade requests per IP per second; rogue clients are dropped before reaching the fleet | Token Bucket / Leaky Bucket as lightweight middleware in the connection event loop |
| **Why there** | Stop the flood at the edge | Connection is pinned to this server, so it tracks frequency **locally** — zero network hops to Redis/DB, so it's extremely fast |

---

