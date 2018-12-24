---
layout: default
title: Demo
nav_order: 40
---
# Demo

Demonstrates per-user (browser) server-event action using the following
packages.  Please refer to the following:

- [@aphorica/server-event-client](server-event-client.html) - a
  client-side package to instantiate an _EventSource_ object and
  initiate and connections for server-side events.

- [@aphorica/server-event-mgr](server-event-mgr.html) - a server-side
  package to register, maintain, and invoke server-side notifications.

## Caveats
- <em style="color:red">IN PROGRESS - STILL DEVELOPING</em>
  - This documentation describes what _should_ be happening.  If not,
    well, I'm working on it.
- (pull requests welcome, but we need to tightly coordinate any
  merges.)
- Read the document pages in this guide for each of the modules for more    detailed information on their use/implementation.
- Currently developing almost exclusively in _Chrome_, but have
  run it cursorily in _Firefox_.  Need to test in more browsers.

## To Set Up
This repository will set up a client/server development environment.

The server environment is created with _docker-compose_ to avoid
having to set up a separate virtual machine or actual machine for
the server side.

Out of the box, the client environment is a _vuejs_ source tree, but without any modules installed.

There are a couple of pre-requisites to setting up the environment:
- Docker ( https://docker.com ) needs to be installed and running.
- _npm_ or _yarn_ needs to be installed and available at the command-line   (I use _yarn_.)
- node needs to be run on the client side, since it is a _vuejs_
  application.  See https://vuejs.org for more information.
- An IDE is useful - I'm using _vscode_ ( https:visualstudio.com ).

The `docker-compose up` command will build an _NGINX_ server environment
under _alpine_.  It will install and run _nodemon_ at port 9229, so you can connect to a debugger (under _vscode,_ anyway.)

Since it is a development environment, you need to do the following
steps to get it up and running:

1. Clone this repository (presumably done.)
2. CD into the ./site directory of your clone.
3. Invoke `npm install` or `yarn install` from the command line.
4. Invoke `npm serve` or `yarn serve` from the command line.
4. Open another commandline window and cd to the demo root.
5. Invoke `docker-compose up` from the command line.

Once everything is up and running, you can open a browser and type:
```
http://localhost:8080
```
Next:

- Type in a name in the text field, and hit the _Set_ button.
  - The 'time' field at the top will start updating - this is an LT
    "listen" registration.
  - The _Submit_ button will be enabled.
- Hitting the _Submit_ button will submit a "task" registration 
   (timeout_test) - it will wait a few seconds and then signal completion.
   - When the task has been submitted, the _Trigger Server Response_ button and others will be enabled.
   - Note your id and registration timestamp will be reflected in the _Registrants_ window.
- You can, hit the _Trigger Server Response_ button,
   which will force the server to initiate an immediate _ad hoc_
   response (reflected by count in the _Response_ header.)
- Open more browsers and enter a different name in each, then hit
   the _Submit!_ and _Trigger Server Response_ buttons to see each
   client receives its own response.
- Open another browser (or more) and enter the same name as one
    of the other browsers.  Note that the responses are reflected
    for all clients registered in the same name.
- Hit the _Stop Listening_ button to remove the user's connection.
    The list will update in all the open browsers to show the user
    has been removed and the _State_ entry will show "Closed".

## Development
Because this is a client/server app, each side referencing it's own npm-module, the development environment is a bit complex.  Here is how I manage it:

### Demo Application
Open an IDE in each of the directories _./site_ and _./app.  This will
make the application source available in the IDE.

- On the client side _(./site)_, this will be a _vuejs_ application.
- On the server side _(./app)_, this will be a _node express_ application.

> Both are hot-served (not the server under Windows, I have discovered) - if you make a change to the source in either directory, they will update.  Unfortunately, this does not apply if you are changing source in either of the other modules.  See next.

### Modules
Clone each of the modules locally in separate directories.  You can either use the 'npm-link', 'yarn-link' or 'yalc' facilities to make them available in the _./site_ and _./app_ folders.

I personally like 'yalc', since the '*-link' facilities can be a bit flaky.

- Install yalc:
 - `npm install -g yalc`
 - `yarn global add yalc`

- cd into each of the module directories and type `yalc publish`.
- cd into the demo _./site_ directory and type `yalc add @aph/server-event-client`
- restart the server (ctrl-c if it's running and `npm serve` or `yarn serve`).
- cd into the demo _./app_ directory and type `yalc add @aph/server-event-mgr`
- restart the container from the demo root directory (`docker-compose down` and `docker-compose up`)

If you change source in either of the modules, you need to type `yalc update` in the module directory and then restart the applicable server, same as above.

If using _yalc_, when you're ready to commit, you need to clean the _yalc_ cruft out of any changed directories.  In each module directory, type `yalc remove`.

### Notes
It can be a bit difficult keeping everything straight.  I open an IDE in each of the _./site_ and _./app_ directories, as well as each of the module directories, with a terminal window in each, as well as a terminal window in the root demo directory.

A large monitor/multiple monitors helps.


