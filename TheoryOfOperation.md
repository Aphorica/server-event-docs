---
layout: default
title: Theory of Operation
nav_order: 25
---
# Theory of Operation
The fundamental premise is that each connection needs to have a unique id, so that the server can associate a connection with a client.

The server maintains all connections in a map, keyed by the unique client id.  The unique client id is obtained _from the server_, since it knows all current ids and can guarantee uniqueness.

So, the first thing the client does is request that id from the server.  It then opens a connection (instantiates an EventSource object) to the server, which REST endpoint is 'register-listener'.  The server responds
with an acknowledgement ('registered': id), creates a resObj as follows:
```
resObj = {
  notifyRes: res,   // the response object
  'registered-ts': Date.now(),  // the registered timestamp
  listenKeys: []    // identifier for to-be-registered keys
}
```
The object is stored in a module-scoped 'connections' object, keyed by the id.

On the client side, the client registers a callback object to receive various notifications.

## Two Types of Listen Requests
The client can request one or more instances of listen requests:
<dl>
<dt>Long-termed (LT)</dt>
<dd>
These are requests that are globally sent to all listeners of the specified name.  Things like timed loops, listeners-changed, etc
are messages that can be sent by the server at any time.

The connection is typically held open for the duration of the app
session.
</dd>
<dt>Task</dt>
<dd>
A task is defined on the server that may take a long time to complete -
longe than the http-request timeout. The request is made to the server to initiate the task and then the client continues (not unlike normal asynchronous requests, except they may take longer to complete - like
minutes, hours or days.)

When the task is complete, the server notifies the client it is complete by id, and the client can handle it.
</dd>
</dl>

To handle the id and connections on the client side, the client instantiates a 'server-event-client' object, which in turn does the following:
- requests a unique id from the server
- instantiates an `EventSource` object
- requests the 'register-listener' endpoint from the server using the `EventSource` constructor.
- assigns a callback object to the `EventSource` object.

This establishes the contact with the server.  From here, the client register to general LT listeners, or submit a task.  Multiple listeners and tasks can be registered/submitted on a single connection.

## General LT Notification
To register for general LT notification, the client calls
```
<sseObj>.listen(listenKey);
```
supplying the key for the events it wants notification (note these must be active processes defined on the server.)  When the server sends a notification, it is via the callback object provided to the `server-event-client` object on instantiation:
```
<cbObj>.sseNotify(listenKey);
```

## Task Notification
To submit a task, the client calls
```
<sseObj>.submitTask(taskname);
```
Once again, the 'taskname' is an identifier to a specific task implemented on the server.

When the task is completed, the notification is recieved via the callback object provided to the `server-event-client` object on instantiation:
```
<cbObj>.sseCompleted(taskname);
```
After the task is completed, the client may elect to close the connection and remove the `server-event-client` object.
## Closing connection
Making and closing connections are controlled from the client, always.  The server will not close a connection on its own.

To close a connection, the client calls:
```
<sseObj>.disconnect();
```
This will notify the server of the disconnection (so it can clean up), call close() on the `EventSource` object, and then delete it when it responds with a 'closed' callback.

Finally, it will the 'closed' callback to the client-registered callback object, which can (should) delete the `server-event-client` object.

## Other Notifications
There are other maintenance/book-keeping/debugging notifications provided by the server and the `server-event-client` object - they are detailed in their respective documentation pages:

- [server-event-client](server-event-client.html) - the client-side object.
- [server-event-mgr](server-event-mgr.html) - the server-side object.


