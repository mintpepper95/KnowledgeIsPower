### Identifying functional and non-functional requirements.

Infinite scrollable news feed - stories based on user subscriptions
User can share and post a story.

Story can contain text and image.

There are comments and likes.

We want feature for a range of devices, and be accessible for people with disabilities.

#### Draw a Mock UI
We can use it to understand the data flow in our app

![[Pasted image 20240608162939.png]]

---

### Component Architecture

![[Pasted image 20240608163124.png]]

---
### Data entities

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

---

### API 
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

---
### State management
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

##### What happens when user visits a react app
1. Browser sends HTTP request
2. Server sends a single html, index.html for a react app
3. The index file includes script tags that points to JS bundles
4. As browser is parsing index.html, `<link rel='stylesheet>` triggers request for css, `<script src='...'>` triggers request for js. So now more requests.
5. Server returns compiled/minifield JS, CSS, images, fonts etc. These live in `dist` or `build` folder.
6. Downloaded JS, `main.js` initialise the React app, and begins to render its components.


---

#### CDN to cache static files

In this setup:

- Your assets **live on your origin server** (e.g., Vite dev server, Express, or even Netlify).
    
- The CDN **sits in front**, and **caches files on-demand** as users request them.
    

This is how **Vercel**, **Netlify**, **Cloudflare Pages**, etc., work by default.

**How it works:**

1. User requests `main.js`
    
2. CDN checks if it has a cached copy.
    
3. If not, it fetches it from your origin (your app server), stores it, and serves it.
    
4. Next time someone else requests it, it's served directly from the CDN.

## What About JS, HTML, and CSS?

Yes — they are also **static files**, just like images.

When you build your React app (`vite build` or `npm run build`), you generate:

- `index.html`
    
- `main.[hash].js`
    
- `style.[hash].css`
    
- `/assets/...` (fonts, images, icons, etc.)
    

You deploy **all of this** to the CDN (either manually or through automated deployment).

The CDN then:

- Caches them.
    
- Serves them globally.
    
- Can apply optimizations like Brotli compression, HTTP/2, etc.


If you're using **Azure Static Web Apps** → **YES**, static files (JS, HTML, CSS, images) are **automatically deployed to a global CDN** behind the scenes.

- You run `vite build` → this outputs files to `dist/`.
    
- You configure Azure (via `azure-static-web-apps.yml`) to upload `dist/` as the static site root.

After vite build, the output files in `dist/` is all you need to serve your react app in production.

It's cause when you build, vite compiles your source code, jsx, tsx, ts, css etc, and minifies and optimises everything.

Hash filenames for caching.

And outputs a production ready static site to `/dist`.


Note vite does auto code splitting for third party libraries. 
For react component level code splitting, we need React.lazy().


When you `vite build`, Vite uses **Rollup** under the hood. It will:

- Split **vendor code** (e.g., `react`, `react-dom`, etc.) into separate chunks.
    
- Generate a `manifest` to track dependencies and optimize load order.

* It doesn’t split your **own components*** without react.lazy


We hash filenames because browser cache static files. If main.js changes, we want a new main.xxxx.js to distinguish it from main.yyyy.js. So browser sees it as a fresh file and refetch it.


Client side production react app is considered a `static` site, as it's just a bundle of html, css, js and assets. It doesn't need server side runtime, it's just files that can be served directly via web servers or CDNs.


When you visit a url with web app, browser sends a request to your origin server/hosting platform, for example azure static web app.

For a hosting platform like azure static web app that supports CDNs, the request will first hits the CDN, which act as a reverse proxy. CDN cache HTML, JS, CSS at edge nodes (close to user)

Checks if request files is in its file, if yes, serve it, if not fetch it from origin server. 

From browser perspective, it's talking to domain, `myapp.com`, but response may be served by a CDN node near the user.

Once CDN delivers the `index.html`, browser parses it and see links and scripts, and thus request for more content. 

Again these requests go to your domain, CDN intercepts request and respond with cache if available


Note for large media, like large videos and high quality images, it's better to store them in things like blob storage and get them in the app.


---

##### Network performance

Modern bundlers improve FCP and TTI via tree shaking ( smaller bundle ), code splitting (code splits vendor code only, so initial load is smaller), asset optimisation (minifying, compress images to webp), faster parsing JS.


Modern bundlers like Webpack don't handle compression directly.
Eg. gzip to compress Javascript. They encode files in a more compact format.

Brotli is an alternative compression algorithm that enables smaller sizes compared to gzip. It improves performance by reducing size of assets served to the client.

React app often ships with large JS bundles, CSS files and other static assets. Brotli compresses them for smaller bundle sizes.

Reducing time to first paint (FCP) and time to interactive (TTI), as it makes app load faster.

They require custom compression plugins to enable them.

There are some trade offs, with more resources and increased build time.


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



