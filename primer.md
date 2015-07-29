# Tractatus Ripple[*](#1)

Guide to building applications with Ripple

#### 1 Setup

`1.0 -----` `npm i rijs`

`1.1 -----` On the server, `ripple = require('rijs')(server)`

`1.2 -----` On the client, `<script src='/ripple.min.js'></script>`

#### 2 Components

`2.0 -----` **A Ripple application is built up of microviews**

`2.0.0 ---` Each microview is declared as a Custom Element `<twitter-feed>`

`2.0.1 ---` Each microview may depend on zero, one or many data resources `<twitter-feed data="tweets profile">`

`2.0.2 ---` Each microview may have its own styles `<twitter-feed css="blue-twitter-feed.css">`

`2.0.3 ---` Each microview is prerendered with any styles being encapsulated and prepended once

`2.0.4 ---` Each microview is rendered with the invocation `component.call(<el>, data)`

`2.1 -----` **Each component function must be an idempotent transformation function of data `component(data)`**

`2.1.0 ---` The component function is the tag name, or the value of the `is` attribute `<button is="crazy-button">`

`2.1.1 ---` The component function and all declared dependencies must exist in Ripple to render a microview

`2.1.2 ---` The component function will be called with `this` being the DOM Element it is upgrading

`2.1.3 ---` The component function will be called with the data dependencies injected as arguments

`2.1.4 ---` For multiple dependencies, the component will be injected a map `twitterFeed({ tweets, profile })`

`2.1.5 ---` The component is expected to always produce the correct output for the data it is given

`2.2 -----` **All microviews will always be up to date**

`2.2.0 ---` A change in a component or dependencies will result in a rerender of the affected microviews

`2.2.1 ---` This means you will experience reactive views if you change some data

`2.2.2 ---` This means you will experience hot code pushes if you change a component implementation

`2.2.3 ---` This means you will experience the styles being automatically updated if you change a stylesheet

`2.3 -----` **Write cohesive styles**

`2.3.0 ---` Component and styles are decoupled, but one component usually has one set of related styles

`2.3.1 ---` Conventially, call it the same name suffixed with `.css`, `<twitter-feed css="twitter-feed.css">`

`2.4 -----` **Write data-driven components**

`2.4.0 ---` Data-drive both variable and structural parts of your component with [utilise/once](https://github.com/utilise/utilise#--once)

`2.4.1 ---` You can use templating, but this is an anti-pattern `<custom-element template="custom.jade">`

#### 3 Application

`3.0 -----` **Your entire application is a pure function of data**

`3.0.1 ---` Your entire application is actually a collection of microviews, each a pure function of data

`3.1 -----` **Ripple maximises realtime interactivity and optimistic rendering for perceived performance**

`3.1.0 ---` When your application first loads, it will load resources, if any, from `localStorage` and render

`3.1.1 ---` When your application first loads, it will stream resources from the server and rerender any changes

`3.1.2 ---` When your application changes any resource, it will be sent to the server to process the change

`3.1.3 ---` When your application changes any resource, it will synchronise the change with all (other) clients

`3.1.4 ---` When your application changes any resource, it will synchronise connected adaptors (db, redis, etc)

#### 4 Change

`4.0 -----` **There is only one event: `change`**

`4.0.1 ---` You can listen for change on all resources `ripple.on('change', fn)`

`4.0.2 ---` You can listen for change on a particular resource `ripple('name').on('change', fn)`

`4.0.3 ---` You can use `once` instead of `on`

`4.0.4 ---` You can force listener updates (on all or for specific resources) `ripple.emit('change')`

`4.0.5 ---` You can register and emit your own event `.on('event', fn)`

`4.0.6 ---` You can namespace events. There is only one listener for namespaced event.

#### 5 Sync

`5.0 -----` **Ripple uses per-resource declarative proxy functions to define the flow of data between server-client**

`5.0.0 ---` The concept is similar to, but more generic than, "request-response" as that is HTTP implementation detail 

`5.0.1 ---` Consider receiving a response ("push") without a request, or making a request that does not have a response

`5.0.2 ---` Register proxies as a header `ripple('resource', [], { from, to })`

`5.1 -----` **`proxy-to`**: Whenever a resource is sent from the server **to** a client, it will be passed through this first

`5.1.0 ---` `this` is the socket being sent to, first argument is the current resource body to transform

`5.1.1 ---` This is used to send a different representation(s) to a client, if at all. For example:

`5.1.1.0 -` `(..) => false` returning false will not send the resource at all, useful for privatising some resources

`5.1.1.1 -` `(likes) => likes.length` this will collapse and just send the total number of likes, rather than all like records

`5.1.1.2 -` `(..) => users[this.sessionID] ? everything : somethings` useful for varying representations based on auth

`5.2 -----` **`proxy-from`**: Whenever a resource is received **from** the client it will be passed through this first

`5.2.0 ---` `this` is the socket being reeiving the resource from

`5.2.1 ---` first argument (`item`) - item in object or array that was changed

`5.2.2 ---` second argument (`body`) - the full collection (resource body)

`5.2.3 ---` third argument (`index`) - where the item appears in the body (key/index)

`5.2.4 ---` fourth argument (`type`) - type of change that happened to the item in the body (`push`/`update`/`remove`)

`5.2.5 ---` fifth argument (`name`) - the name of the resource being changed

`5.2.6 ---` This is used to process a change from a client before Ripple stores it in-memory and ripples it elsewhere:

`5.2.6.0 -` `(..) => false` returning false will ignore all changes from the client for a resource

`5.2.6.1 -` `(..) => type != 'push'` ignore all type of changes, except adding new items

`5.2.6.2 -` `(..) => likes.push(this.ip)` this will ignore the integer increment, and push a new record instead

#### 6 Pipeline

`6.0 -----` **Mix and match your own pipeline**

`6.0.0 ---` pemrouz/ripple is one particular pipeline, which you are unlikely to need as-is.

`6.0.1 ---` See the particular modules used [here](https://github.com/pemrouz/ripple/blob/master/src/index.js#L27-L45) and pick/create your own in a similar fashion.

#### 7 Adding Resources

`7.0 -----` **For convenience, all JS and CSS files under `/resources` will be auto imported on startup**

`7.0.0 ---` Styles (CSS) will registered under the name matching their filename (with `.css`)

`7.0.1 ---` Components (JS) will registered under the name matching their filename (without `.js`)

`7.0.2 ---` Components will be `require`d, so write as `module.exports = function component(data){ .. }`

`7.1 -----` **Break out resources into separate modules where possible**

`7.1.0 ---` You can import resources from other modules using `ripple(..)` or `ripple.resource(..)`

`7.1.2 ---` You can export/import a resource, an array of them or another Ripple node with its own resources

#### 8 Server Side Rendering (SSR)

`8.0 -----` **SSR is not one thing**

`8.0.0 ---` Ripple gives the user the chance to pick different strategies per-component:

`8.0.0.0 -` Fallback Content (e.g. "Your browser is not good enough")

`8.0.0.1 -` Static Render (e.g. SVG Chart as PNG)

`8.0.0.2 -` Semantic Content (e.g. "`<table>`" vs full markup for complex data grid)

`8.0.0.3 -` Full Prerender (e.g. SVG Chart or full markup for complex data grid) <br>

`8.1 -----` **SSR is enabled as middleware**

`8.1.0 ---` HTML files with `<custom-elements>` are expanded (as on the client) before being sent

`8.1.1 ---` This means it will result in a full prerender by default for microviews

`8.1.2 ---` You can force a different strategy in your component `if (process.ua < acceptable) return fallback`

`8.1.3 ---` Microviews are seamlessly updated on the client as and when subsequent changes happen over streaming

<a name="1">[1]</a>A reference that this follows Wittgenstein's style in [_Tractatus Logico-Philosophicus_](https://www.gutenberg.org/files/5740/5740-pdf.pdf), as a series of short statements