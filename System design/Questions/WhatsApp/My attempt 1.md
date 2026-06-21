

![[Pasted image 20260622021525.png]]

---
### Feedbacks

1. Outbox with background service + READ DB with CDC is wrong

This approach is for reliability propagating events to other systems like downstream consumers, not for delivering a message to a person who's waiting for it. It adds latency.

WebSocket as a single pipe point is correct, but polling for non active chat is a weakness. If a user has 20 active convos, HTTP polling means their device is constantly sending HTTP requests which both drains user battery and create load on our servers.

Instead of tying WebSocket connection to a specific chat room, tie connection to user session. 

A dispatcher in front end code can look at events and update chats.


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
