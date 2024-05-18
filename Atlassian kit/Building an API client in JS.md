









What do we want to test for an API client?
We want to test return data.





#### What if I Need to Call Two Different APIs With the Same Client?

API Client should be **very opinionated**, you might be wondering how you would build an API client that needs to call two different APIs. The answer is: **you don't**. Instead, you should be building _two different clients_ - each one tailored specifically to the given API. The moment you start to think, "how can I make this more generic", you've already lost. You start down that slippery slope and you end-up just re-building the `fetch()` API on to of the `fetch()` API. And then all you have is incidental complexity without any value.