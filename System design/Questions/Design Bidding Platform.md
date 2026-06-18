Based on Jordan has no life

Bidding platform
* Current bid
* Number of bids
* Highest bidder
* End time

Users can bid on items (highest price wins)
Auctions can have fixed end time or variable depending on last bid
Users looking at the item should get price update in real time
Needs to be able to support many incoming bids per second

### Approach

`Data Models`
Bid: bidId, auctionId, userId, price, server_timestamp (don't want to use timestamp from clients), status (is it the current highest bid?)

We want all bids per auction for auditing purpose ( e.g. primary buyer somehow leaves, then we want to give it to second highest bidder). We could put it into a time series database, and shard by AuctionId.

Auction State: AuctionId, currentHighestBidId, price, endtime

Each field in Auction state has 4 fields, so 8 bytes each and 32 in total.


#### Philosophy of problem
