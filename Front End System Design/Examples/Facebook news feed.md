#### Identifying functional and non-functional requirements.

Infinite scrollable news feed - stories based on user subscriptions
User can share the story, can post a story.

We want feature for a range of devices, and be accessible for people with disabilities.

#### Draw a Mock UI
We can use it to understand the data flow in our app

![[Pasted image 20240608162939.png]]

Component Architecture

![[Pasted image 20240608163124.png]]

#### Data entities
```
type Story = {
	id: number,
	comments: Comment[],
	media: Media[];
	date: number;
	content: string;
	origin: { // can be from a user, or from a group
		id: string;
		type: OriginType;
		name: string;
		data:...
	}
}

type Comment = {
	id: number;
	authorId string;
	media: Media[];
	date: number;
	content: string;
}

type Media = {
	type: 'link' | 'video';
	url: string;
}
```


API 
``` 
// maybe we don't want the comments
// cursor is the timestamp, could be we want data from two years ago
getPosts(api_key, user_id, exclude_comments, cursor, page_size, min_id)


createPost(api_key, user_id, post_data)

createComment(api_key, user_id, post_id, comment_data)

```
We can go with REST api or GraphQL approach.
Talk about REST vs GraphQL.


How REST is inflexible, if you want dynamic data then you would have to build a lot of REST endpoints.

On the contrary GraphQL accepts both GET/POST requests, but GET request has no body, meaning your query have to be sent through query params, which can be problematic for larger queries (URL too long 414 status)
So best practice for GraphQL is to use POST requests with a `application/json` content type.




Q - Do they want to post with a single request? For request with medias, like image or video?


Data Store
We can simplify state management and reduce complexity by flattening our front end store.

Feed_store
User_store
Entity_store (origin)
Comments_store

![[Pasted image 20240608164159.png]]

For separation of concern, we can isolate presentation logic from data management and consolidate all actions that mutates the store in selective places. So the components interact with the presenter.



How can we efficiently load new stories?
Long polling - asking server for new data periodically, full request gets sent each time

Websocket - For real time, note it does not support HTTP2 protocol
More efficient than SSE and long polling, data can be transmitted back and forth without overhead of creating multiple HTTP requests.

A single websocket connection can be shared between browser tabs using a localStorage so not necessarily more expensive.

Also more flexible than SSE, WebSocket allows you to send JSON, binary data and custom protocols compared to just text.

Harder to load balance compared to REST API and SSE as both of latter are HTTP requests.

More complex due to being long lived and bidirectional. Specialsied lb like Nginx.


Trello uses WebSocket to exclusively receive info.
It uses HTTP request to send board modifications to server.

HTTP is okay because board modifications don't require immeidate real time feedback, they have less overhead compared to WebSocket messages, so more suitable for one off actions like creating, updating, deleting.

Also HTTP requests have built-in mechanisms for handling retries, which makes them more reliable for critical actions.

HTTP can leverage security mechanism like HTTPS. For WebSocket more complex.

WebSocket connections are persistent and consume server resources, so limiting their use to receiving updates can help reduce server load and improves scalability.



Server side events 
SSE - allows server to push real time updates to clients via HTTP. Clients initiates connection by sending a standard HTTP request requesting SSE stream.

Server establishes persistent HTTP connection, then sends data to client in text based events. Each event is a separate HTTP response. So longer latency compared to WebSocket.

If connection is lost between client and server, client will auto reconnect to server, ensuring continuous data streaming.

Unlike WebSocket, SSE only supports unidirectional communication, whereas WebSocket supports bidirectional communication.


Pagination for infinite scroll
Show more with Intersection Observer API

When user scrolls to intersection zone, we update the page by replacing the existing dom nodes with the new ones.

We can have a sliding window to show only a limited number of DOM elements instead more and more.



#### Optimisation
##### Network performance
Modern bundlers like Webpack don't handle compression directly.
Eg. gzip to compress Javascript. They encode files in a more compact format.

If browser supports webp, we can use webp images. They are lighter than jpg and png, therefore load faster.

We can also optimise image based on viewport, we can use some third party image optimisation service.

We can cache images and static contents on a CDN, so we don't need to connect to server.

CDNs are distributed, cache content. has optimised routing and better at handling high volumes of traffic therefore faster.

So static content ( styles, scripts, images, html, files that is the same every time it's delivered to the user ) should be hosted on CDN.

Host the app on a CDN - so your app is hosted on a distributed network of servers and the one closest to your user will send the data


Can switch to HTTP2 - multiplexing, which allows us to load resources in parallel - bundle splitting.

##### Rendering performance
SSR
Loading scripts using defer

##### Javascript performance
Do stuff async
Cache results
Memo
Web workers- for running scripts in the background, separate from main thread. For offloading heavy computational tasks, eg. complex calculations and data processing, prevents main thread from becoming blocked and keeps ui responsive.


Accessibility
Semantic HTML
Images have alt attributes
ARIA attributes
High colour contrast
Hot Keys for navigating card
REM fonts - instead of pixel for it to be adaptive to the scale of the system
cross device testing for compatibility

REM - relative to font size of root element
EM - relative to font size of their element

While fonts in px scale with zoom, if user change default font size in the browser and they visit a website with px font, then the websitefont will not change size.


For drag and drop can use react-beautiful-dnd

![[Pasted image 20240608194516.png]]

Security
CORS - Cross-Origin Resource Sharing
HTTPS -  HTTPS prevents websites from having their information broadcast in a way that’s easily viewed by anyone snooping on the network. When information is sent over regular HTTP, the information is broken into packets of data that can be easily “sniffed” using free software. This makes communication over the an unsecure medium, such as public Wi-Fi, highly vulnerable to interception.



