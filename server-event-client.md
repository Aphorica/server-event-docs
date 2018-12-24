---
layout: default
title: "&nbsp;&nbsp;&nbsp;&nbspserver-event-client"
nav_order: 32
---
# @aphorica/server-event-client

Client-side utility for registering and maintaining _EventSource_ connections.

The npm package is here:
 - [@aphorica/server-event-mgr](https://www.npmjs.com/package/@aphorica/server-event-client) - server-side implentation of a server-event service.

## Overview
The EventSource object is provided by the browser (or a polyfill in the case of non-support (IE/Edge)) to manage long-lived _keep-alive_ connections with the server, and to keep them open until closed.

This utility provides an `EventSource` implementation that, in turn, is tuned to the server implementation provided by the @aphorica/server-event-mgr.

It also provides debugging invocations into the server to aid in development.

## Installation
`npm install @aphorica/server-event-client` _or_ `yarn add @aphorica/server-event-client`

## API
 The _EventSource_ object is created, retained, and
destroyed entirely within the `ServerEventClient` instance.  All interactions from the application are with the `ServerEventClient` instance.

Note also that the `ServerEventClient` instance is created via the `ServerEventClientFactory.create()` function.  The reason for this is that during the creation process, several async calls must be invoked with the server, and waiting for them within a constructor is not good practice, nor are errors detected and conveyed in a constructor easily.

<h3>Creation API</h3>
<blockquote>
<code>ServerEventClientFactory.create(name, appurl, cb, prefix)</code>
<blockquote>
<em>name</em> - a name unique to the caller<br/>
<em>appurl</em> - the base url for calls to the server<br/>
<em>cb</em> - an object containing requisite callbacks<br/>
<em>prefix</em> - prepended to api calls to the server<br/>
returns - a <code>ServerClient</code> instance.<br/><br/>
Creates the <code>ServerClient</code> instance.  This instance contains
the <code>EventSource</code> object and receives and sends requests to
and from the server.
<p>
The _cb_ object can be anything that implements the set of required
callbacks.  Callbacks are as follows:</p>
<code>sseOpened(readyState, readyStateText)</code>
<blockquote>
<p>
<em>readyState</em> - the readyState value of the <em>EventSource</em>
 object. Can be one of:</p>
<ul>
<li>EventSource.CONNECTING (0) ("Connecting")</li>
<li>EventSource.OPEN (1) ("Open")</li>
<li>EventSource.CLOSED(2) ("Closed")</li>
</ul>
<p>
<em>readyStateText</em> - text corresponding to the readyState<br/>
Called when the <em>EventSource</em> object establishes a
connection with the server.</p>
</blockquote>
<code>sseClosed()</code>
<blockquote>
Called when the <em>EventSource</em> object is closed.  At this
point, the object should remove itself as it is no longer
valid.</blockquote>
<code>sseError(readyState, readyStateText)</code>
<blockquote>
Called on error. (Actually haven't seen this.)</blockquote>
<code>sseListenersChanged()</code>
<blockquote>
Received on a <em>listeners-changed</em> notification from the server.</blockquote>
<code>sseTaskCompleted(id, taskid)</code>
<blockquote>
<em>id</em> - the unique id for this client<br/>
<em>taskid</em> - the taskid provided in the `submitTask` call.<br/>
Called when the specific task is completed.</blockquote>
<code>sseRegistered(id)</code>
<blockquote>
<em>id</em> - the unique id for this client<br/>
Called when the server registers the id.
</blockquote>
<code>sseAdHocResponse()</code>
<blockquote>
Called when the server sends an ad-hoc notification.</blockquote>
</blockquote>
</blockquote>
<h3>Operations API</h3>
<blockquote>
<code>submitTask(taskname)</code>
<blockquote>
<em>taskname</em> - name of the task.</em>
Submits a named task to the server to execute and then
send notification on completion.  The taskname will be provided in the 
<code>sseTaskCompleted</code> call.</blockquote>
<code>getSSEState()</code>
<blockquote>
Return the <em>EventSource.readyState</em> value.</blockquote>
<code>getSSEStateText()</code>
<blockquote>
Return the text corresponding to the <em>EventSource.readyState</em>
value.</blockquote>
<code>disconnect()</code>
<blockquote>
Disconnect the <em>EventSource</em> object from the server. This will remove the <em>EventSource</em> object and invoke the <em>sseClosed</em> callback (which should trigger deletion of the <em>ServerEventClient</em> class.)</blockquote>
</blockquote>

<h3>Debugging API</h3>
<blockquote>
<code>fetchRegistrants()</code>
<blockquote>
Fetch all the currently registered entries held by the server.  Returned as a JSON object.</blockquote>
<code>triggerAdHocServerResponse()</code>
<blockquote>
Invokes call to force the server to trigger an immediate reponse.</blockquote>
<code>triggerCleanup()</code>
<blockquote>
Invokes call to force the server to do an immediate cleanup pass.</blockquote>
</blockquote>