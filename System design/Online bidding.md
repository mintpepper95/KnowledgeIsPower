Design a bidding system
* User can post an item for auction/bidding anytime
* Bidders can bid on any existing item any number of times
* 10M new auction items daily, 100M new bids daily
* User wins if no higher bids in next 1 hour
* User has to pay the item within 10 min of winning
* User can only bid for 1 count of one item at any time, so if 10 cars in the system. User can only bid 1 car at a time.

#### Functional Requirements
* User can put items for bid
* Bidders can search for items they are interested in
* Bidders can bid on items.

#### Non-functional Requirements
* Low latency
* High availability
* High consistency - bidding part should be highly consistent. Bidders shouldn't be able to bid on un-available items. It's okay for new items to take some time before they become available for bidders.

Talk about entities and API
