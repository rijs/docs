# API Reference

---

#### Instantiation

<a name="ripple" href="#ripple">#</a> **`ripple`**

default global ripple instance

<a name="require" href="#require">#</a> **`require('ripple')`**`(opts | server)`

both parameters optional. `server` is the `http.Server` instance (express app) to connect to clients on. if object (`opts`) is passed in, it can have the following:

* `server` — as above
* `secret` — [secret used to sign session ID cookie](https://github.com/expressjs/session#secret)
* `name` — [name of the session ID cookie](https://github.com/expressjs/session#name)

---

#### General

<a name="ripple-1" href="#ripple-1">#</a> **`ripple`**`('name')`

return the named resource, creating one if it doesn't exist

<a name="ripple-2" href="#ripple-2">#</a> **`ripple`**`('name', body)`
 
create or overwrite the named resource with the specified body

<a name="ripple-3" href="#ripple-3">#</a> **`ripple`**`('name', body, headers)`

create or overwrite the named resource with the specified body and extra metadata

<a name="ripple-4" href="#ripple-4">#</a> **`ripple`**`({ 'name', body, headers })`

create or overwrite the named resource with the specified body and extra metadata

<a name="ripple-5" href="#ripple-5">#</a> **`ripple`**`([ .. ])`

register multiple resources

<a name="ripple-6" href="#ripple-6">#</a> **`ripple`**`(ripple2)`

import resources from another ripple node

<a name="resource" href="#resource">#</a> **`ripple.resource`**`('name'[, `_`body`_`[, `_`headers`_`]])`

alias for ripple as above that allows method chaining for registering multiple resources

<a name="on" href="#on">#</a> **`ripple.on`**`('change', function)`

react to all changes 

<a name="on" href="#on">#</a> **`ripple('name').on`**`('change', function)`

react to changes on the named resource

<a name="once" href="#once">#</a> **`ripple('name').once`**`('change', function)`

react once to a change on the named resource

---

#### Render

<a name="draw-1" href="#draw-1">#</a> **`ripple.draw`**`()` 

redraw all components on page

<a name="draw-2" href="#draw-2">#</a> **`ripple.draw`**`(element)`

redraw specific element

<a name="draw-3" href="#draw-3">#</a> **`ripple.draw`**`.call(element)`

redraw specific element

<a name="draw-4" href="#draw-4">#</a> **`ripple.draw`**`('name')` 

redraw elements that depend on named resource

<a name="draw-5" href="#draw-5">#</a> `MutationObserver(`**`ripple.draw`**`)`

redraws element being observed

---

#### Components

```html
<component-name is="component-name" data="name(s)" template="name" css="name" inert delay="ms">
```

* `component-name` — name of registered component resource
* `is` — name of registered component resource, extends another element
* `data` — name of registered data resource, multiple values separated by space
* `template` — apply a template before invoking the component function
* `css` — apply the css before invoking the component function
* `inert` — signals to ripple to not touch this element
* `delay` — time in ms to postpone rendering a component

Examples:
```html
<time is="relative-time">
<fast-grid data="prices instruments">
<event-item data="events" id="519" template="event-item.html">
```

---

#### Streaming

<a name="io" href="#io">#</a> **`ripple.io`**

the underlying socket.io instance

<a name="sync" href="#sync">#</a> **`ripple.sync(sockets)(resources)`**

emit all or some resources, to all or some clients. 

values: 
* `sockets` can be a nothing (means send to all sockets), a socket, or a sessionID string
* `resources` can be nothing (means send all resources) or name of resource to sync

---

#### Database

<a name="db" href="#db">#</a> **`ripple.db`**`('type://user:password@host:port/database')`

connects ripple to something else, synchronishing any changes. `type` must exist in `ripple.adaptors`

<a name="adaptors" href="#adaptors">#</a> **`ripple.adaptors`**

array of services ripple knows how to connect to. mysql only by default.

each new adaptor must be a function that takes the destructured connection string `{ type, user, password, host, port, database }` and returns an object with four crud functions `{ push, update, remove, load }` - these functions will be called when the corresponding event occurs.

<a name="connections" href="#connections">#</a> **`ripple.connections`**

array of active connections to other services

---

#### Versioning

<a name="version-1" href="#version-1">#</a> **`ripple.version`**`('name')`

retrieves the current version index for the named resource

<a name="version-2" href="#version-2">#</a> **`ripple.version`**`('name', i)`

rollbacks the named resource to version `i` and returns its value at that time

<a name="version-3" href="#version-3">#</a> **`ripple.version`**`()`

retrieves the current historical index for the entire application

<a name="version-4" href="#version-4">#</a> **`ripple.version`**`(i)`

rollbacks entire application state to version `i`

--- 

#### Headers

<a name="content-type" href="#content-type">#</a> `[header]`**`content-type`**

resource type is interpreted based on the body type - it is not necessary to explicitly set this value.

values: `application/data (default)` | `application/javascript` | `text/html` | `text/css`. 

Example: `ripple('tweets', [], { 'content-type': 'application/data' })`

<a name="cache-control" href="#cache-control">#</a> `[header]`**`cache-control`**

caching behaviouring for a resource. values: `no-store | undefined (default)`

Example: `ripple('user', {}, { 'cache-control': 'no-store' })`

<a name="table" href="#table">#</a> `[header]`**`table`**

specify which database table/collection to populate the resource with and sync changes with 

Example: `ripple('users', [], { 'table': 'members' })`

<a name="to" href="#to">#</a> `[header]`**`proxy-to | to`**

function applied to body before being sent to clients 

Example: `ripple('users', [], { to: reduceToNumber })`

<a name="from" href="#from">#</a> `[header]`**`proxy-from | from`**

function applied to changes received from client 

Example: `ripple('read-only-data', [], { from: ignoreChages })`

<a name="listeners" href="#listeners">#</a> `[header]`**`listeners`**

array of listeners listening for changes on this resource

Example: `header('listeners')(resource).length`

<a name="max-versions" href="#max-versions">#</a> `[header]`**`max-versions`**

maximum number of versions to store for this resource. values: `Infinity (default) | 0` 

Example: `ripple('frequently-updated-data', [], { 'max-versions': 0 })`

<a name="extends" href="#extends">#</a> `[header]`**`extends`**

specifies which element this component extends 

Example: `ripple('color-picker', require('./picker'), { 'extends': 'input' })`