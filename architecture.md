# Architecture

Ripple is a reactive, resource-oriented, standards embracing, progressively enhancing, isomorphic, productive, functional, modular, realtime application architecture. The philosophy is that _all_ changes ripple across the network to all other connected servers, clients and databases synchronising them in realtime where possible.

It is built as a pipeline of independent modules, to abstract the plumbing of using modules directly in your application code, and at the same time future-proofing by not being locked into yet another framework. It starts off with a trivial, but extensible, [core](https://github.com/rijs/core) module. This is essentially an event emitter with one event (change). Every module takes the existing ripple instance and layers new behaviours and affordances.

All modules live under the [rijs](https://github.com/rijs/) org. This document explains the particular set of modules connected together to make the [pemrouz/ripple](https://github.com/pemrouz/ripple/blob/master/src/index.js#L28-L45) nervous system (which is what I use in side projects), but you will likely change some modules to build your own pipeline to suit your development patterns or specific application (e.g. Popper uses only a [connection of three modules on test agents](https://github.com/pemrouz/popper/blob/master/client.js#L1-L4) since the agents could be very old browsers). For more info on any module, click to drill down into its own repo.

For learning how to build applications with Ripple, see the [Primer]() instead.

[![Coverage Status](https://coveralls.io/repos/rijs/sync/badge.svg?branch=master&service=github)](https://coveralls.io/github/rijs/sync?branch=master)
[![Build Status](https://travis-ci.org/rijs/sync.svg)](https://travis-ci.org/rijs/sync)
* [**Sync**](https://github.com/rijs/sync): When a resource changes on the client, it is emitted back to the server. When a resource changes on the server it is broadcast to all clients. Hooks provided for private resources (or transforming them in general) before broadcast/receive.

[![Coverage Status](https://coveralls.io/repos/rijs/data/badge.svg?branch=master&service=github)](https://coveralls.io/github/rijs/data?branch=master)
[![Build Status](https://travis-ci.org/rijs/data.svg)](https://travis-ci.org/rijs/data)
* [**Type - Data**](https://github.com/rijs/data): This extends core to register objects and arrays. It also enables per-resource change listeners on those resources.

[![Coverage Status](https://coveralls.io/repos/rijs/fn/badge.svg?branch=master&service=github)](https://coveralls.io/github/rijs/fn?branch=master)
[![Build Status](https://travis-ci.org/rijs/fn.svg)](https://travis-ci.org/rijs/fn)
* [**Type - Functions**](https://github.com/rijs/fn): This extends core to register functions. For cases when a function resource is transferred to a client or loaded from cache as a string, this converts it into a real function before storing.

[![Coverage Status](https://coveralls.io/repos/rijs/components/badge.svg?branch=master&service=github)](https://coveralls.io/github/rijs/components?branch=master)
[![Build Status](https://travis-ci.org/rijs/components.svg)](https://travis-ci.org/rijs/components)
* [**Web Components**](https://github.com/rijs/components): This redraws any custom elements on the page when either the component definition changes or any of it's dependencies.

[![Coverage Status](https://coveralls.io/repos/rijs/shadow/badge.svg?branch=master&service=github)](https://coveralls.io/github/rijs/shadow?branch=master)
[![Build Status](https://travis-ci.org/rijs/shadow.svg)](https://travis-ci.org/rijs/shadow)
* [**Shadow DOM**:](https://github.com/rijs/shadow) This extends the rendering pipeline to append a shadow root before rendering a custom element. If the browser does not support shadow roots, it sets the `host`/`shadowRoot` pointers so that a component implementation depending on them works both in the context of a shadow root or without.

[![Coverage Status](https://coveralls.io/repos/rijs/precss/badge.svg?branch=master&service=github)](https://coveralls.io/github/rijs/precss?branch=master)
[![Build Status](https://travis-ci.org/rijs/precss.svg)](https://travis-ci.org/rijs/precss)
* [**Pre-apply CSS**:](https://github.com/rijs/precss) If a custom element has a CSS dependency, it will be added at the start of the shadow root if one exists, or scoped and added once in the head if there is no shadow root.

[![Coverage Status](https://coveralls.io/repos/rijs/prehtml/badge.svg?branch=master&service=github)](https://coveralls.io/github/rijs/prehtml?branch=master)
[![Build Status](https://travis-ci.org/rijs/prehtml.svg)](https://travis-ci.org/rijs/prehtml)
* [**Pre-apply Template/HTML**:](https://github.com/rijs/prehtml) In case you like to pre-apply HTML templates before operating on it with JS. I've long moved away from this, but kept it as it serves an example of how modules for other templating types could be supported.

[![Coverage Status](https://coveralls.io/repos/rijs/reactive/badge.svg?branch=master&service=github)](https://coveralls.io/github/rijs/reactive?branch=master)
[![Build Status](https://travis-ci.org/rijs/reactive.svg)](https://travis-ci.org/rijs/reactive)
* [**Reactive**:](https://github.com/rijs/reactive) Watches any data resource for changes and emits a change event when it does, to avoid the repetitive boilerplate in manually dispatching events. Uses `Object.observe`, or fallback to polling, 

[![Coverage Status](https://coveralls.io/repos/rijs/offline/badge.svg?branch=master&service=github)](https://coveralls.io/github/rijs/offline?branch=master)
[![Build Status](https://travis-ci.org/rijs/offline.svg)](https://travis-ci.org/rijs/offline)
* [**Offline**:](https://github.com/rijs/offline) When a resource changes, this caches all resources to `localStorage`. When the application starts it loads everything from cache, which has _massive_ impact on how fast your application is perceived - as opposed to waiting for subsequent network events to render things.

[![Coverage Status](https://coveralls.io/repos/rijs/db/badge.svg?branch=master&service=github)](https://coveralls.io/github/rijs/db?branch=master)
[![Build Status](https://travis-ci.org/rijs/db.svg)](https://travis-ci.org/rijs/db)
* [**Database**:](https://github.com/rijs/db) This allows connecting the server node to other things. For example, when a resource changes, it could update a database, synchronise with other instances over AMQP, or pump to Redis. Each adaptor is just an object with the four crud functions, which will be invoked when the respective event happens. 

[![Coverage Status](https://coveralls.io/repos/rijs/mysql/badge.svg?branch=master&service=github)](https://coveralls.io/github/rijs/mysql?branch=master)
[![Build Status](https://travis-ci.org/rijs/mysql.svg)](https://travis-ci.org/rijs/mysql)
* [**Adaptor - MySQL**:](https://github.com/rijs/mysql) This will tell database how to deal with a MySQL connection

[![Coverage Status](https://coveralls.io/repos/rijs/pgsql/badge.svg?branch=master&service=github)](https://coveralls.io/github/rijs/pgsql?branch=master)
[![Build Status](https://travis-ci.org/rijs/pgsql.svg)](https://travis-ci.org/rijs/pgsql)
* [**Adaptor - PostgreSQL**:](https://github.com/rijs/pgsql) This will tell database how to deal with a PostgreSQL connection

[![Coverage Status](https://coveralls.io/repos/rijs/redis/badge.svg?branch=master&service=github)](https://coveralls.io/github/rijs/redis?branch=master)
[![Build Status](https://travis-ci.org/rijs/redis.svg)](https://travis-ci.org/rijs/redis)
* [**Adaptor - Redis**:](https://github.com/rijs/redis) This will tell database how to deal with a Redis connection

[![Coverage Status](https://coveralls.io/repos/rijs/hateoas/badge.svg?branch=master&service=github)](https://coveralls.io/github/rijs/hateoas?branch=master)
[![Build Status](https://travis-ci.org/rijs/hateoas.svg)](https://travis-ci.org/rijs/hateoas)
* [**Adaptor - HATEOAS**:](https://github.com/rijs/hateoas) - This will tell database how to deal with consuming a HATEOAS source, and also enriches data resources with links the user can follow to other resources.

[![Coverage Status](https://coveralls.io/repos/rijs/sessions/badge.svg?branch=master&service=github)](https://coveralls.io/github/rijs/sessions?branch=master)
[![Build Status](https://travis-ci.org/rijs/sessions.svg)](https://travis-ci.org/rijs/sessions)
* [**Sessions**:](https://github.com/rijs/sessions) This will enrich each socket with a matching `sessionID`, so you could for example not broadcast resources to any users who are not logged in.

[![Coverage Status](https://coveralls.io/repos/rijs/serve/badge.svg?branch=master&service=github)](https://coveralls.io/github/rijs/serve?branch=master)
[![Build Status](https://travis-ci.org/rijs/serve.svg)](https://travis-ci.org/rijs/serve)
* [**Auto Serve Client**:](https://github.com/rijs/serve) Exposes the distribution files as endpoints on your server.

[![Coverage Status](https://coveralls.io/repos/rijs/singleton/badge.svg?branch=master&service=github)](https://coveralls.io/github/rijs/singleton?branch=master)
[![Build Status](https://travis-ci.org/rijs/singleton.svg)](https://travis-ci.org/rijs/singleton)
* [**Singleton**:](https://github.com/rijs/singleton) Exposes the instance globally on window/global.

[![Coverage Status](https://coveralls.io/repos/rijs/rest/badge.svg?branch=master&service=github)](https://coveralls.io/github/rijs/rest?branch=master)
[![Build Status](https://travis-ci.org/rijs/rest.svg)](https://travis-ci.org/rijs/rest)
* [**REST Endpoints**:](https://github.com/rijs/rest) Ripple prefers "REST over WebSockets", but if you need a traditional HTTP API for interop, this will allow you to generate endpoints for specific resources from a selection of content-types and replete with hypermedia links to other resources.

[![Coverage Status](https://coveralls.io/repos/rijs/versioned/badge.svg?branch=master&service=github)](https://coveralls.io/github/rijs/versioned?branch=master)
[![Build Status](https://travis-ci.org/rijs/versioned.svg)](https://travis-ci.org/rijs/versioned)
* [**Versioned**:](https://github.com/rijs/versioned) - When a data resource changes, it stores a history of all the past versions using immutable data structures. This is used by time travel debuggers to capture the events that happened before an error occured, so that a developer can replay the application as the user saw it. 

[![Coverage Status](https://coveralls.io/repos/rijs/url/badge.svg?branch=master&service=github)](https://coveralls.io/github/rijs/url?branch=master)
[![Build Status](https://travis-ci.org/rijs/url.svg)](https://travis-ci.org/rijs/url)
* [**Parameterised Resources**:](https://github.com/rijs/url) Currently all resources can be globally uniquely referenced by a name. This allows you to register parameterised resources (names as URL routes), which can be useful for example to deliver a subset of the resource for memory/network constrained devices.

[![Coverage Status](https://coveralls.io/repos/rijs/ssr/badge.svg?branch=master&service=github)](https://coveralls.io/github/rijs/ssr?branch=master)
[![Build Status](https://travis-ci.org/rijs/ssr.svg)](https://travis-ci.org/rijs/ssr)
* [**Server Side Rendering**:](https://github.com/rijs/ssr) This registers a middleware on your server to expand any Custom Elements using the same components logic and available resources at the time before sending the page to the client.

[![Coverage Status](https://coveralls.io/repos/rijs/resdir/badge.svg?branch=master&service=github)](https://coveralls.io/github/rijs/resdir?branch=master)
[![Build Status](https://travis-ci.org/rijs/resdir.svg)](https://travis-ci.org/rijs/resdir)
* [**Resources Directory**:](https://github.com/rijs/resdir) On startup, this will load resources in your `/resources` directory so you do not have to require and register each file manually.

[![Coverage Status](https://coveralls.io/repos/rijs/live/badge.svg?branch=master&service=github)](https://coveralls.io/github/rijs/live?branch=master)
[![Build Status](https://travis-ci.org/rijs/live.svg)](https://travis-ci.org/rijs/live)
* [**Live Reload**:](https://github.com/rijs/live) During development, this will watch for any changes in your resources folder and reregister it on change. So if you change a component implementation, or a component CSS, it will automatically be reflected in the client without any refreshes.

[![Coverage Status](https://coveralls.io/repos/rijs/virtual/badge.svg?branch=master&service=github)](https://coveralls.io/github/rijs/virtual?branch=master)
[![Build Status](https://travis-ci.org/rijs/virtual.svg)](https://travis-ci.org/rijs/virtual)
* [**Virtual DOM**:](https://github.com/rijs/virtual) This does not acutally exist, but I just added it as an example of how trivial it would be to tack a module on the end of your rendering pipeline that captured the component output, diffed it, then applied the changes. Note that I have no need or intention of actually building this because I think HTML churn can be mostly solved by writing idempotent components with micro-libraries like [once](https://github.com/utilise/utilise#--once).