From Hello Interview



Instagram Auction Post
 - ability to auction it off on IG
 - auction has fixed end time - highest bid wins


In this video, we first started with analysing requirements.

---
#### Functional Requirements
1. Users can create auction post (clarified just sell one item)
2. Users can submit bids
3. Determine and notify auction winner at end
4. Users viewing items should see bids coming in (real-time updates)

##### Non-Functional Requirements
1. Strong consistency for bidding
2. Scalability (Asked for scales, interviewer mentioned mils of auction bids, and also most users will just view and not bid, so much more reads than writes)
3. Low latency for bid updates


---
#### Core entities
- User
- Auction
- Auction Item
- Bid

---

API (inferring from functional requirements)
* POST auction, mentions auth through header via JWT
* POST bid
* Mentioned that determining Auction winner is a server side thing, so no endpoint needed
* GET auction

This guy missed GET APIs for live subscription

---

* Note this guy missed talking about CDNs like Cloudflare often sits in front of API Gateway which handles global load balancing, CDN caching etc


#### Initial design

Client -> API Gateway (authenticate, rate limiting, request/response transformation, routing to different services) -> Auction Service -> DB  <- cron job figuring out winner


