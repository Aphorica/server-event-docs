---
layout: default
title: Requirements
nav_order: 27
---
# Requirements
## Client-side: _server-event-client_
On the client side, the only requirement is a javascript client implementation (the demo client is written with _vue.js_, but that is not a requirement.)

## Server-side: _server-event-mgr_
On the server side, the implementation is expected to be a _node express_ implementation, since the `server-event-mgr` object is effectively an _express_ module.

It could be implemented in another server-side language - _java_, _php_, _python_,_ruby_, whatever.  That's just not my current need.

If this is deemed useful enough that you want it in another server-side language, it shouldn't be too difficult to port.

