
Note thread takes memory and takes time to start thread.

Async programming is ideal for tasks not CPU bound. Accessing a DB, file server etc

With synchronous programming, imagine your app is currently querying the database, your caller thread will actively wait (thus blocking the cpu) for the response, it will not be busy, but merely waiting for the result from database ( non- cpu bounded tasks ). During that time ,we are also blocking the cpu to do anything else (eg. user interface rendering). Our app will appear frozen.

With async programming, we can query the database, let it do its job in the background. Do something else. And when the database query is finished, it will trigger another piece of async code that does the processing. The important thing is during the query time, our cpu and ui thread is free.

#### Async programming on server side
