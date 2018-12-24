---
layout: default
title: Overview
nav_order: 20
---
# Overview
Server-side events are useful for situations where
 - You (the client app) want to receive events sent spontaneously by the    server.
 - You (the client app) trigger some request on the server that will    
   exceed an http request timeout (and don't want to block the client
   waiting for it.)
 - The communication is one way.

## VS WebSockets
Websockets are good for brief, two-way communication.  But they are
resource intensive. A simple google search reveals that 10,000 or so
simultaneously open sockets becomes an issue and can heavily load a server.

For one-way communication from the server, you already have an https
connection - why not use that to receive responses from the server?

## Implementation
Ostensibly, server-side events are comparatively simple.  On the
client side, you create an `EventSource` object with a URL that
the server know you're initiating.  The server then responds with
these headers:
```
 'Content-Type':'text/event-stream'
 'Connection':'keep-alive'
 ```
 And that's about it.  When the server wants to send a message,
 it simply does a `res.write()` with the message.

 But, there are devils in the details:

 <h3>Server Complications</h3>
  - The server has to keep the response object around, so that it
    can use it to send messages.  Logically, that means it has to store it
    somewhere.  Easy enough, but...
  - If the client goes down, the server doesn't get any notification
    that it's gone.  So, saved responses
    hang around, indefinitely.  Cleanup of stale connections is
    an issue.
  - The `EventSource` object in the client re-calls the initial
    url now and again, which generates a fresh response object.
    The old response object is now dead and must be replaced
    with the new.

One way to address these issues (the way that is employed here)
is for the server to associate the responses with the requestor, somehow, along with a timestamp.  It can periodically check how long it's been since it has been contacted by that requestor. If the duration exceeds some threshold, it can assume the connection is stale and remove it.

That introduces yet another issue: if you have 10,000 or 100,000
listeners, you don't want to block the server while you do these
checks - or global notifications.  So, the loops have to be
constructed in such a way as to yield control back to the server
after every (successful) iteration.

### Client Complications
There are complications on the client-side, too:
  - When the client creates the `EventSource` object, it passes
    a listener object. The listener can listen
    for messages, but can't trigger them until the `open`
    callback is received.
    
    This introduces timing issues if,
    for instance, you want to trigger a task with a _Submit_
    button or some other event.
  - If the server is going to associate a response with a connection
    it needs a unique identifier to do so.  The client has to
    retain the unique identifier for subsequent requests, and for
    shutting down the connection.
  - Operations are somewhat arcane.


