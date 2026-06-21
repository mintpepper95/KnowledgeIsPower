From Jordan has no life


Real time chat
Deterministic message ordering
Add/remove members from chat
Receive chats from all chats you are in - whether online or coming back from offline

Chat
- Chat Id
- User Id

Message
- Sender Id
- Chat Id
- Message content
- server timestamp
- metadata

---
#### Initial approach

![[Pasted image 20260622005505.png]]

WebSocket for real time updates from server to clients.
However if a user is in many conversations, they would need to open many connections to different servers which is expensive.

#### Second approach
Each receiver user should just maintain a single connection to the active convo.

We also don't want senders to send requests to many different servers. As it puts load on the sender device to send many api requests for large group chats.


![[Pasted image 20260622005920.png]]

---
#### Third approach

![[Pasted image 20260622010220.png]]

Server can do the routing.

We want to report 'sent' to the sender but async ensure all parties receive the message.
Sender needs to publish message to a durable location.

#### Fourth approach

![[Pasted image 20260622010411.png]]

Kafka to act as buffer for servers. We can send message to Kafka and return success for sender.

Problem here, what if a receiver device is offline, then we need to send messages when it comes back online.

We can sync the message to our message db. When receiver comes online, it can poll message it missed. DB will be bottlenecked. 

#### Fifth approach

![[Pasted image 20260622010720.png]]


We can introduce a UserMessages table.
User polls to get recent messages.
Some kind of eviction policy so table doesn't grow too alrge
If scrolling up in a chat, poll Message table.
Message table can be shared by chat id.

We 


