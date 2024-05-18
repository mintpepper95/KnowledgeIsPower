Problem statement - create a `cachedApiCall()` that takes in the cache time in ms, returns a function which takes in a url and a config object (optional), when executed this function will return a promise.

```ts
const cachedApiCall = (time) => {
	// utilising closure, mapping url+config to time_expires
	const cache = {};

	return async (url, config={}) => {
		const key = `${url}${JSON.stringify(config)}`;
		// do we have data in cache
		const entry = cache[key];
		// if no entry or entry expired
		if (!entry) || Date.now() > entry.expiry) {
			console.log('making api call');
			try {
				// now can use async/await
				let resp = await fetch(url, config);
				resp = await resp.json();
				cache[key] = {
					value: resp,
					// now + time in ms when it expires
					// note if you want human readable format need to pass it into a new Date object
					expiry: Date.now() + time
				}
			} catch(e) {
				console.log(e);
			}
		}
		// This will be wrapped in a promise
		return cache[key].value;	
	}
}

```