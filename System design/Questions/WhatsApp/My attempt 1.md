

![[Pasted image 20260622021525.png]]

---
### Feedbacks


#### Outbox with background service + READ DB with CDC is wrong

This approach is for reliability propagating events to other systems like downstream consumers, not for delivering a message to a person who's waiting for it. It adds latency.

WebSocket multiplexing as a single pipe point is correct, but polling for non active chat is a weakness. If a user has 20 active convos, HTTP polling means their device is constantly sending HTTP requests which both drains user battery and create load on our servers.

A dispatcher in front end code can look at events and update chats.
##### WebSocket multiplex
WebSocket multiplexing is running multiple independent streams of data over a single WebSocket connection. Reduced overhead compare to multiple WebSocket connection as HTTP handshake is performed just once. Clients can subscribe/unsubscribe to specific topics.


#### Inefficiency in retrieving unread message
In my design, to find unread messages, we have to scan db looking for `is_read` flag.
If a user recieves 500 messages, we would need to do 500 updates. Cause massive locks on db.

We can instead use a `watermark` for each user in chat. 
* maintain small table, [user_id, chat_id, last_read_message_id]
* When user opens the chat, we do just one db update, update last_read_message_id


#### Raw timestamps
In my design, we use server timestamps to order messages. In a distributed system, we have hundreds of servers processing messages. There can be clock skew between different servers, eg. Server A's clock maybe slightly faster than Server B. Message maybe appear out of order.

Yes we can send all messages for a chat to the same server.
However there will be a hot partition problem handling large traffic.
Also if server goes down, then room is dead. System must detect the crash, elect a new server and reroute the traffic and the chat state must be loaded.

We must separate acting of receiving messages from act of delivering it.
When client sends a message, they hit a LB, LB routes request to any available server.
Server persist message to db and drops it to kafka.

Kafka use  chat_id to route message to a specific partition. A bg worker can process that partition.


#### Session routing & Presence Service
Distributed chat system can't function without knowing where users are connected and if they are online.

Because WebSockets are stateful, long lived connections. A user is tethered to one WebSocket node out of many.
* Session routing - When a message arrives in backend, backend must ask a fast store like Redis: Which specific WebSocket server holds User B's connection now? So need to map user_id to WS node to route message to the right machine so it can be pushed to the client.
  
  We can have a Presence Service for listening to heartbeats from client.

### Sql vs NoSql
They are pretty similar these days. Even NoSql can do transactions now.
So instead we could talk about data access pattern.

Chat append to a data partition, read recent N by sort key, occasional scan back.

So NoSql works well, however many chat systems also works well on Sql. Discord started with Sql and moved to NoSql.

#### CAP and Idempotency

CAP only applies during a partition.
Messafe delivery is about deliver now, reconcile order later.

Message ordering within a chat and idempotetency push you towards needing per partition consistency.


#### Do we need event sourcing?
We don't need event sourcing pattenrn. But chat message log is append only, features like edit/delete history, read receipts and multi-device sync can be moddeled as events over time.

#### Idempotency
Without a client generated message id/idempotency key, retrying failing messages will create dupes.

#### Fan-out for group chats
Do you fan out on write (push to everyone's inbox) of fan out on read (everyone reads shared chat partition).


#### Hot partition
A 50k member group keyed to one partition is a hotspot. Can say chat throughput per group is bounded.



Send diagram

![[Pasted image 20260622104635.png]]
Offline/multi-device reconciliation

![[Pasted image 20260622104715.png]]



The first is the **idempotency + ordering pair**. The script is roughly: "Client generates a `client_msg_id`. Chat Service dedups on `(chat_id, client_msg_id)` so network-blip retries don't duplicate. It then assigns a monotonic per-chat sequence — that's my ordering authority, not wall-clock time, which dodges clock skew across nodes. Kafka partitioning by `chat_id` preserves that order through the broker, and the client uses the sequence to detect gaps and trigger backfill." If you say that unprompted, you've answered idempotency, ordering, indexing, and clock-skew in one breath, and the interviewer mentally upgrades you.

---

### Ordering, clock skew, Snowflake ID and sequence


