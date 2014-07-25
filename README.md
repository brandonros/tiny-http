## tiny-http

[![Build Status](https://travis-ci.org/tomaka/tiny-http.svg?branch=master)](https://travis-ci.org/tomaka/tiny-http)

Tiny but strong HTTP server in Rust.
Its main objectives are to be 100% compliant with the HTTP standard and to provide an easy way to create an HTTP server.

What does **tiny-http** handle?
 - Accepting and managing connections to the clients
 - Parsing requests
 - Requests pipelining
 - Transfer-Encoding and Content-Encoding (**not fully implemented yet**)
 - Turning user input (eg. POST input) into a contiguous UTF-8 string (**not implemented yet**)
 - Ranges (**not implemented yet**)
 - HTTPS (**not implemented yet**)
 - `Connection: upgrade` (used by websockets)

Tiny-http handles everything that is related to client connections and data transfers and encoding.

Everything else (parsing the values of the headers, multipart data, routing, etags, cache-control, HTML templates, etc.) must be handled by your code.
If you want to create a website in Rust, I strongly recommend using a framework instead of this library.

[**Link to the documentation**](http://www.rust-ci.org/tomaka/tiny-http/doc/tiny-http/index.html)

### Installation

Add this to the `Cargo.toml` file of your project:

```toml
[dependencies.tiny-http]
git = "https://github.com/tomaka/tiny-http"
```

Don't forget to add the external crate:

```rust
extern crate httpd = "tiny-http"
```

### [Usage](http://www.rust-ci.org/tomaka/tiny-http/doc/tiny-http/index.html)

```rust
let server = httpd::Server::new().unwrap();

for request in server.incoming_requests() {
    println!("received request! method: {}, url: {}, headers: {}",
        request.get_method(),
        request.get_url(),
        request.get_headers()
    );

    let response = httpd::Response::from_string("hello world".to_string());
    request.respond(response);
}
```

### Speed

Tiny-http was designed with speed in mind:
 - Each client connection will be dispatched to a thread pool. Each thread will handle one client.
 If there is no thread available when a client connects, a new one is created. Threads that are idle
 for a long time (currently 5 seconds) will automatically die.
 - If multiple requests from the same client are being pipelined (ie. multiple requests
 are sent without waiting for the answer), tiny-http will read them all at once and they will
 all be available via `server.recv()`. Tiny-http will automatically rearrange the responses
 so that they are sent in the right order.
 - One exception to the previous statement exists when a request has a large body (currently > 1kB),
 in which case the request handler will read the body directly from the stream and tiny-http
 will wait for it to be read before processing the next request. Tiny-http will never wait for
 a request to be answered to read the next one.
 - When a client connection has sent its last request (by sending `Connection: close` header),
 the thread will immediatly stop reading from this client and can be reclaimed, even when the
 request has not yet been answered. The reading part of the socket will also be immediatly closed.
 - Decoding the client's request is done lazily. If you don't read the request's body, it will not
 be decoded.
