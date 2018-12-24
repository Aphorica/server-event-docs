---
layout: default
title: "&nbsp;&nbsp;&nbsp;&nbsp;server-event-mgr"
nav_order: 34
---
# @aphorica/server-event-mgr

Implements a server-event service in node _Express_.

the npm package is here:
 - [@aphorica/server-event-mgr](https://www.npmjs.com/package/@aphorica/server-event-mgr)

Implements a server-event service in node express.

## Features:
 - provides rest endpoints for operations
 - provides a singleton 'manager' object that actually performs
   the operations.  You can either allow the express
   rest calls provided (which will use the object, internally), 
   or you can provide your own rest calls and invoke the operations
   on the manager object.
 - keeps a list of connections - these are used to send
   messages when a task is completed.
 - the list is periodically scanned for dormant connections.  Those
   inactive for a specific timeout threshold are cleaned up.
 - you can provide a prefix to the built-in rest points.  The
   default prefix is '/sse/'

## Caveats:
 - <em style="color:red">IN PROGRESS - STILL DEVELOPING</em>
 - (pull requests welcome, but we need to tightly coordinate any
    merges.)
 - Currently does not support multiple managers. Could think
   about that, if required.
 - If the client drops, there is no way for the server to know that
   happened.  Consequently, when the client comes back, its id is
   invalid.  Active re-connection by the client (get a new id and
   re-register) will resume any currently registered tasks.

## Implementation Notes:
### Instantiated Router
The manager creates a router via the `createRouter()` function,
which returns the router.  The router needs to be added
to the _Express_ app:
```
const ServerEventMgr = require('@aph/server-event-mgr');
app.use(ServerEventMgr.createRouter());
```
If you provide your own rest handlers, you do not need to
create the router (untested).
### Express Global Request Handlers
A typical _Express_ app will have a global request handler to
set things like response headers, especially CORS headers.

SSE responses require their own headers.  If you have implemented
a global request handler to provide headers, you must qualify the
request to not send headers when the rest path includes
`/register-listener/`.

An example from the test/demo package follow (I use semi-colons):

```
var allowCrossDomain = function(req, res, next) {
  console.log('allowXDomain: ' + req.method +
  ' ' + req.path);
  if (req.path.indexOf('/register-listener/') === -1) {
            // do NOT do following if registering listener
    res.header('Access-Control-Allow-Origin', '*');
    res.header('Access-Control-Allow-Methods', 'GET,PUT,POST,DELETE,OPTIONS');
    res.header('Access-Control-Allow-Headers', 'Content-Type, Accept, Authorization, Content-Length, X-Requested-With');
    // intercept OPTIONS method
    if (req.method === 'OPTIONS') {
      res.sendStatus(204);
      return;
    }
  }

  next();
};
```

SSE responses include a CORS header for '*' (Might could make this
a settable feature, if desired);
### "/submit-task" REST Endpoint
The `/submit-task` REST endpoint is provided by you.  In this
endpoint, you will perform the task necessary, and at the end of
the task, you will invoke the manager `notifyCompletions()` function
with the client id (provided earlier) and a task id.

The method of the endpoint is entirely up to the you - it can
be a __PUT__, __GET__, __POST__, or whatever.

### "/listen" REST Endpoint
The `/listen` REST endpoint is provided by you.

## Rest API
The provided rest endpoints are listed, along with their corresponding
manager calls.  See the _ServerEventManager_ API for operational details.

### Main API
 - `(pfx)/make-id/:name` => `getUniqueID(name)`
 - `(pfx)/register-listener/:id` => `registerListener(id, response)`
 - `(pfx)/disconnect-registrant/:id` => `unregisterListener(id)`

### Provided By Implementor (you)
 - `/submit-task/:id/:taskid` => (ultimately) `notifyCompletions(id, taskid)`

### Debugging 
 - `(pfx)/list-registrants` => `getListenersJSON()`
 - `(pfx)/clear-registrants` => `unregisterAllListeners()`
 - `(pfx)/trigger-adhoc/:id` => `triggerAdHocResponse(id)`
 - `(pfx)/trigger-cleanup` => `triggerCleanup()`

## Manager API
### Setup API
<blockquote>
<code>createRouter(prefix)</code>
<blockquote>
<em>prefix</em> - the prefix to apply (default: '/sse/')<br/>
returns - a new router<br/><br/>
The prefix will be prepended to all publicly exposed urls
in the router.  If not provided, the default ('/sse/') will
be used.

The returned router must be 'added' to the _Express_ app by the
caller.
</blockquote>
<code>setVerbose(flag)</code>
<blockquote>
<em>flag</em> - true/false<br/><br/>
If set, will log diagnostic messages.  Used for debugging.</blockquote>
<code>setNotifyListenersChanged(flag) - (experimental)</code>
<blockquote>
<em>flag</em> - true/false<br/><br/>
If set, when users are added or removed, all listeners will be
notified of the change.  Mainly for debugging, but might be
useful in other contexts.</blockquote>
</blockquote>

### Main API
<blockquote>
<code>getUniqueID(name)</code>
<blockquote>
<em>name</em> - typically a username<br/>
returns - the unique id<br/><br/>
Constructs a unique id based on the provided name, in the form
`name_xxxxx` where 'x' represents an alphanum character.  The
resultant id is used for all subsequent operations that require
an id.</blockquote>
<code>isRegistered(id)</code>
<blockquote>
<em>id</em> - the unique id for this listener<br/>
returns - whether or not the id is registered<br/><br/>
Query if the id is in the list of active connections.</blockquote>
<code>registerListener(id, response)</code>
<blockquote>
<em>id</em> - the unique id for this listener<br/>
<em>response</em> - the original response object passed in the
      request.<br/><br/>
The response object is associated with this id and held.  It is
used to transmit subsequent notifications to the client(s)
associated with this ID.</blockquote>
<code>unregisterListener(id)</code>
<blockquote>
<em>id</em> - the unique id for this listener<br/><br/>
The listener is removed from the list of active connections and
the response is nulled deleted.  No other actions occur.</blockquote>
<code>notifyCompletions(id, taskid)</code>
<blockquote>
<em>id</em> - the unique id for this listener<br/>
<em>taskid</em> - a task-specific id provided by the client<br/><br/>
Called by the implementor when a specific task is completed after
submission (from the client.)  The taskid is returned to the client in the notification.</blockquote>
</blockquote>

### Debugging API
<blockquote>
<code>getListenersJSON()</code>
<blockquote>
returns - a JSON of all the listeners as a map<br/><br/>
Used mainly for debugging</blockquote>
<code>unregisterAllListeners()</code>
<blockquote>
Removes and deletes all listener entries.  Also
turns off the cleanupInterval mechanism.</blockquote>
<code>notifyListenersChanged() - (experimental)</code>
<blockquote>
Will send change notification to all listeners.</blockquote>
<code>triggerAdHocResponse(id)</code>
<blockquote>
<em>id</em> - the unique id for this listener<br/><br/>
Forces the server to send a notification.</blockquote>
<blockquote>
<code>triggerCleanup()</code>
<blockquote>
Forces an immediate invocation of the cleanup mechanism.</blockquote>
</blockquote>

