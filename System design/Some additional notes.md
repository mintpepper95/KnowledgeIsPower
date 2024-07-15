
### Message queue vs LB


#### LB
With LB we can evenly distribute requests among different servers to avoid overload.

A web service like Amazon Elastic Compute Cloud (EC2) can scale up and down number of services dynamically.

Here the API Gateway is used to authenticate and authorise requests.
![[https-api-gateway-load-balancer-service-registry.png]]
LB is more useful for communication in real time, they are generally easier to implement compared to message queues.


##### How does WebSockets work together with LB?
Recall WebSockets opens a stateful long lived duplex connection for bi-direction communication. Which requires the server to tie its resources to that particular request. 

This is contrast to stateless protocol like HTTP where connections are short-lived, less taxing on servers.

If you have a lot of WebSockets connections, how do we scale them?

To scale servers, we can do it two ways.
* Vertical scaling - better hardware, limited benefits
* Horizontal scaling

Imagine a Chat application, client connected to one server WILL NOT receive a message routed to another server. How can the client gets notified when there's a new message?

The solution is to store the connection state out of process, using redis.
This way, any server behind loader balancer can access the same session info. This ensures regardless which server the client connects to, they can resume their session seamlessly.

Basically server publishes session data to a redis channel, use session id as the key.
For subsequent requests, client include session id as part of the request.

When a client makes a new request, LB may route it to a different server. This new server will retrieve session data from client request. This allows this new server to access same session info as the initial server.

In the case of a chat app, Redis pub/sub can broadcast message to all subscribed servers, in this case all subscribed servers maybe all servers with the same room id.
And each of these servers will then send the message to all its connected clients.


![[Pasted image 20240715224537.png]]

If client is connected to an API Gateway service, the Gateway service can redirect to actual services via HTTP or gRPC.

- f there are real-time updates (e.g., a new highest bid in the case of a bidding service), the backend service publishes these updates to Redis.
- The API Gateway subscribes to relevant Redis channels and forwards these updates to the connected clients via WebSockets as needed.
#### Main challenges of scaling WebSockets
We also need to sync connection state, if user A goes offline. User B can be notified. 

###### What happens when one of the subscribed servers is reaching its hardware limit?
We need a mechanism to detect overload and reject incoming connections, and also do load shedding, disconnect some clients.




---
#### Message queue
Message queue acts as a centralised hub for communication.
Services publish message to the queue.

When adopting message queue, the need for LB is eliminated. The queue will distribute the messages evenly among the instances.

Message queue is for async communication and acts as a buffer, so when front end is down we can still process the events on the queue. Useful for when there's no need for real time communication.

It's not good for real time communication because there is a potential delay between when message is sent and when it's processed.


![[message-queue-as-load-balancer.png]]


---

API Gateway vs Load Balancer

![[https___dev-to-uploads.s3.amazonaws.com_uploads_articles_hbtqnb9vajxpq8acbgx2.avif]]

![[https___dev-to-uploads.s3.amazonaws.com_uploads_articles_loxqc71ei86bovy7bjis.avif]]
