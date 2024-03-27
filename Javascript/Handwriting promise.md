```ts
const STATE = {
	FULFILED: 'fulfiled',
	REJECTED: 'rejected',
	'PENDING': 'pending',
}




class MyPromise {
	thenCallbacks = [];

	catchCallbacks = [];
	

	// whether the promise is rejected, resolved or pending
	state = STATE.PENDING;

	// resolved value
	value = '';


	// Recall the executor is executed right away, executor takes in two args
	// If you have error inside your promise, it will be rejected
	constructor(cb) {
		try {
			cb(this.onSuccess, this.onFail);
		} catch(e) {
			this.onFail(e);
		}
	}

	runCallbacks() {
	if (this.state === STATE.FULFILED) {
		// loop through then callbacks
		this.thenCallbacks.forEach(cb => {
			callback(this.value);
		})
		this.thenCallbacks = [];
	}


	if (this.state === STATE.REJECTED)
		// loop catch callbacks
		this.catchCallbacks.forEach(cb => {
			callback(this.value);
		})
		this.catchCallbacks = [];
	}

	// this should be private
	onSuccess(value) {
		// a promise only resolves once
		if (this.state != STATE.PENDING) {
			return;
		}
		this.value = value;
		this.state = STATE.FULFILED;
		runCallbacks();
	}

	// this should be private
	onFail(value) {
		// a promise only resolves once
		if (this.state != STATE.PENDING) {
			return;
		}
		this.value = value;
		this.state = STATE.REJECTED;
		runCallbacks();
	}

	// recall p.then(x => ...)
	// p.then().then()...
	// note callback is only ran onSuccess
	then(cb) {
		this.thenCallbacks.push(cb);

		runCallbacks();
	}
}


```