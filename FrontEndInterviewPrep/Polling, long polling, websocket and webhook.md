[[#What is polling?]]
[[#What is long polling?]]
[[#What is WebSocket and how does it differ from long polling?]]
[[#What is a webhook?]]


---



#### What is polling?
This is a variation of HTTP request, where a client keeps sending a request to the server at a regular interval.

Unnecessary network calls as most time response will be empty, so waste resource on both client and server. Also can cause delay since there maybe new messages between the last http request and the next http request.


#### What is long polling?

![[Pasted image 20240602114306.png]]

Long polling is a way to keep an HTTP connection open, so you would need to implement this on the server side.

Unlike regular polling, the connection continues and server keeps a connection open until there is new data to send or a timeout occurs. Upon a timeout (502) or after data is received, the browser then makes a new request immediately. 

In this case, only when a message is delivered or a timeout happens, the connection is closed and re-established.

Long polling reduces number of requests sent by the client. But still needs to poll periodically just less.

Long polling is generally much more taxing on the server than websockets.

#### What is WebSocket and how does it differ from long polling?
Client sends a HTTP request to the server to ask for opening a connection with websocket.
Sever will send a 101 response if agrees.
A single, persistent TCP/IP connection is open, which allows bi-direction messages to pass between client and server with very low latency at will (unlike typical http requests with request response model where repeated attempts maybe required and each request requiring a new connection), until one party drops out.

E.g. Chat room app, where you need to push messages to multiple clients.

Scalability issues - Websockets are stateful, meaning that once a connection is established, both client and server will maintain the connection. 

This can lead to increased resource usage on the server, as the websockets must remain open for an extended periods.

Websocket can introduce challenges when scaling because all servers will need to be aware of each state of each websocket connection. We can handle this by making sure websocket connections coming from a particular client that consistently routed to the same server. This ensures the server handling a websocket connection has access to the connection's state.

Instead of sending messages to websocket client right away, we can publish them on a channel like Kafka and then let Kafka handle them. Upon receiving messages from Kakfa we then send back to websocket clients.

This helps by decoupling. We decouple websocket server from message processing logic and can independently scale both. We can scale websocket servers to handle incoming connections without worrying about processing


##### A few word about using WebSocket in React
We can use libraries like `useWebSocket` and `Socket.IO` for managing connections with WebSocket.

Some nice things about using `useWebSocket` is we don't have to worry about cleanup, as the hook auto cleanups after component unmount.

We want to subscribe to WebSocket events from top level components, and update top level state in response to new events, and pass the state down as a prop.

Alternatively we can pass it down by using React Context.












#### What is a webhook?
To put simply, it's just an API endpoint you expose to another service you are interested in so it will send a request to you when something interesting happens.

Client register with Webhook api provider. Client register the event it's interested in, and the callback url for Webhook api provider to send callback to. The callback url is an endpoint ( really just an API endpoint like REST ) the client exposes for the webhook api provider.

When an event happens, eg. a new order placed, the system sends a HTTP request to the webhook url with info about that event. The receiving system then process this info and perform some actions.

For instance, if you're working with Stripe, and you send your customers to a Stripe run service to make a payment, you would want to know when a payment has succeeded or failed, because that affects how they're treated when they return to your website.

You can setup webhook with the address of your endpoint, so you're notified when such events occur. If your client app doesn't have its own endpoint ( think of a simple front-end only react app ), then webhook wouldn't work.

So webhook is used more for event driven communication between system to system, while long polling and websocket are used for real-time communication between end users and systems.














#### Bundle splitting


ARIA attributes



REM font


Keyboard navigation with tab index



#### Observaility
Tracking user interactions
Logging for performance tracking, TTI and TTL





