# Architecture

Ripple is a reactive, resource-oriented, standards embracing, progressively enhancing, isomorphic, productive, functional, modular, realtime application architecture. The philosophy is that _all_ changes ripple across the network to all other connected servers, clients and databases synchronising them in realtime where possible.

It is built as a pipeline of independent modules, to abstract the plumbing of using modules directly in your application code, and at the same time future-proofing by not being locked into yet another framework. It starts off with a trivial, but extensible, [core](https://github.com/rijs/core) module. This is essentially an event emitter with one event (change). Every module takes the existing ripple instance and layers new behaviours and affordances.

All modules live under the [rijs](https://github.com/rijs/) org. This document explains the particular set of modules connected together to make the [pemrouz/ripple](https://github.com/pemrouz/ripple/blob/master/src/index.js#L28-L45) nervous system (which is what I use in side projects), but you will likely change some modules to build your own pipeline to suit your development patterns or specific application (e.g. Popper uses only a [connection of three modules on test agents](https://github.com/pemrouz/popper/blob/master/client.js#L1-L4) since the agents could be very old browsers). For more info on any module, click to drill down into its own repo.

For learning how to build applications with Ripple, see the [Primer]() instead.

* **Sync**: When a resource changes on the client, it is emitted back to the server. When a resource changes on the server it is broadcast to all clients. Hooks provided for private resources (or transforming them in general) before broadcast/receive.

* **Type - Data**: This extends core to register objects and arrays. It also enables per-resource change listeners on those resources.

* **Type - Functions**: This extends core to register functions. For cases when a function resource is transferred to a client or loaded from cache as a string, this converts it into a real function before storing.

* **Web Components**: This redraws any custom elements on the page when either the component definition changes or any of it's dependencies.

* **Shadow DOM**: This extends the rendering pipeline to append a shadow root before rendering a custom element. If the browser does not support shadow roots, it sets the `host`/`shadowRoot` pointers so that a component implementation depending on them works both in the context of a shadow root or without.

* **Pre-apply CSS**: If a custom element has a CSS dependency, it will be added at the start of the shadow root if one exists, or scoped and added once in the head if there is no shadow root.

* **Pre-apply Template/HTML:** In case you like to pre-apply HTML templates before operating on it with JS. I've long moved away from this, but kept it as it serves an example of how modules for other templating types could be supported.

* **Reactive:** Watches any data resource for changes and emits a change event when it does, to avoid the repetitive boilerplate in manually dispatching events. Uses `Object.observe`, or fallback to polling, 

* **Offline:** When a resource changes, this caches all resources to `localStorage`. When the application starts it loads everything from cache, which has _massive_ impact on how fast your application is perceived - as opposed to waiting for subsequent network events to render things.

* **Database:** This allows connecting the server node to other things. For example, when a resource changes, it could update a database, synchronise with other instances over AMQP, or pump to Redis. Each adaptor is just an object with the four crud functions, which will be invoked when the respective event happens. 

* **Adaptor - MySQL:** This will tell database how to deal with a MySQL connection

* **Adaptor - PostgreSQL:** This will tell database how to deal with a PostgreSQL connection

* **Adaptor - Redis:** This will tell database how to deal with a Redis connection

* **Adaptor - HATEOAS:** - This will tell database how to deal with consuming a HATEOAS source, and also enriches data resources with links the user can follow to other resources.

* **Sessions:** This will enrich each socket with a matching `sessionID`, so you could for example not broadcast resources to any users who are not logged in.

* **Auto Serve Client:** Exposes the distribution files as endpoints on your server.

* **Singleton:** Exposes the instance globally on window/global.

* **REST Endpoints:** Ripple prefers "REST over WebSockets", but if you need a traditional HTTP API for interop, this will allow you to generate endpoints for specific resources from a selection of content-types and replete with hypermedia links to other resources.

* **Versioned:** - When a data resource changes, it stores a history of all the past versions using immutable data structures. This is used by time travel debuggers to capture the events that happened before an error occured, so that a developer can replay the application as the user saw it. 

* **Parameterised Resources:** Currently all resources can be globally uniquely referenced by a name. This allows you to register parameterised resources (names as URL routes), which can be useful for example to deliver a subset of the resource for memory/network constrained devices.

* **Server Side Rendering:** This registers a middleware on your server to expand any Custom Elements using the same components logic and available resources at the time before sending the page to the client.

* **Resources Directory:** On startup, this will load resources in your `/resources` directory so you do not have to require and register each file manually.

* **Live Reload:** During development, this will watch for any changes in your resources folder and reregister it on change. So if you change a component implementation, or a component CSS, it will automatically be reflected in the client without any refreshes.

* **Virtual DOM:** This does not acutally exist, but I just added it as an example of how trivial it would be to tack a module on the end of your rendering pipeline that captured the component output, diffed it, then applied the changes. Note that I have no need or intention of actually building this because I think HTML churn can be mostly solved by writing idempotent components with micro-libraries like [once](https://github.com/utilise/utilise#--once).