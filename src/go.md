# Go

## Context


Context is a concurrency pattern in Go. It allows request handlers in Go servers to pass cancellation signals to any goroutine that has started from this request.

For example, in a web server, the each request is typically handled by a different goroutine, which in turn may start multiple goroutines to execute database queries
or send requests to other services.

Using the context pattern, a struct called the `context` is passed from one function to the other containing a `Done` channel, which any function starting goroutines
should wait on, typically with `select`. If the `Done` channel is closed, any goroutine should do a best effort to stop any operation and free resources used. 

Without context, an http request may be cancelled, but gorotoutines pointlessly continue their operations. The resources spared in this way are typically small but in 
situations of heavy load, requests may become slow and with clients resending their requests when aborted, performance issues may compound.

* You can add values to it like a map using withValue but it's somewhat of an antipattern
* The main use is for canceling goroutines
* Let's say there is an http request coming, and you start an expensive operation
    * In Node.js, you'd execute this operation, which may even take 10 min.
    * But the client has left, not waiting for a response.
    * You would continue processing it, and at the end put it in the request, which will not be sent anywhere.
* Using native promises, there is no way, for the one who started it, to cancel it.
* In Go, your goroutine can listen in the done channel, to know when to cancel.
* Also, you can define a context deadline as a time duration.
* If you switch to channels, it's good practice to always accept a context and always listen in the done channel.
* Also, for every function that does IO operations you can pass context and call the libraries by passing the context, expecting them to accept it. Eg. in database calls.
    * In this way, your can separate between IO and pure functions too.
