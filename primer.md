# Guide to Building Applications with Ripple

Table of Contents
* [1 Setup](https://github.com/rijs/docs/blob/master/primer.md#1-setup)
* [2 Basic API](https://github.com/rijs/docs/blob/master/primer.md#2-basic-api)
* [3 Components](https://github.com/rijs/docs/blob/master/primer.md#3-components)
* [4 Application](https://github.com/rijs/docs/blob/master/primer.md#4-application)
* [5 Sync](https://github.com/rijs/docs/blob/master/primer.md#5-sync)
* [6 Events](https://github.com/rijs/docs/blob/master/primer.md#6-events)
* [7 Conventions](https://github.com/rijs/docs/blob/master/primer.md#7-conventions)
* [8 Hot Reloading](https://github.com/rijs/docs/blob/master/primer.md#8-hot-reloading)
* [9 Server Side Rendering](https://github.com/rijs/docs/blob/master/primer.md#9-server-side-rendering)
* [10 Compatibility](https://github.com/rijs/docs/blob/master/primer.md#10-compatibility)

<br>
<br>
## 1 Setup

Install with npm:

```sh
npm i rijs
```

On the server:

```js
ripple = require('rijs')(server)
```

The HTTP(S) server instance is primarily used to synchronise with clients in realtime. If you don't specify one, you can create a server-only node which doesn't connect to any clients. There are also some [other options](https://github.com/rijs/docs/blob/master/api.md#require) you can specify.

On the client: 

```html
<script src='/ripple.min.js'></script>
```

The [client endpoints](https://github.com/rijs/docs/blob/master/distributions.md) are exposed on the same server instance for ease of use. This is done by the [serve module](https://github.com/rijs/serve), so if you want to serve the client yourself, you can just comment this module out of your Ripple pipeline.

<br>
<br>
## 2 Basic API

To register a resource:

```js
ripple('key', value)
```

To retreive a resource:

```js
ripple('key')  // returns value
```

To subscribe to updates for a resource:

```js
ripple('key').on('change', function(r){ .. })
ripple('key').once('change', function(r){ .. })
```

To subscribe to all updates globally:

```js
ripple.on('change', function(r){ .. })
```

The idea is to keep it simple! All [other modules](https://github.com/rijs/docs/blob/master/architecture.md) build on this simple API, by reactively taking action when something changes.

<br>
<br>
## 3 Components

A Ripple application is built up of microviews. Each microview is declared as a Custom Element:

```html
<html>
  <body>
    <twitter-feed>
  </body>
</html>
```

Each Custom Element will be upgraded by the corresponding transformation function in Ripple's web of resources if you have registered one:

```js
ripple('twitter-feed', function(){
  this.innerHTML = 'Hello World!'
})
```

Each microview may depend on zero, one or many data resources:

```html
<twitter-feed data="tweets">
```

If an element depends on any data, the value of these data resources will be dependency injected into the transformation function:

```js
ripple('twitter-feed', function(tweets){
  this.innerHTML  = 'Hello World! You have ' + tweets.length + ' tweets'
})
```

Multiple data dependencies are passed in as a map:

```html
<twitter-feed data="tweets profile">
```

```js
ripple('twitter-feed', function({ tweets, profile }){
  this.innerHTML  = 'Hello, ' + profile.name + '! You have ' + tweets.length + ' tweets'
})
```

Whenever there is a change in the component (`twitter-feed`) or any data dependencies (`tweets`), the element will be redrawn. This is accomplished by the [components module](https://github.com/rijs/components) (in case you'd like to use a different view layer), but in a more generic way:

```js
ripple('tweets').on('change', r => all('[data=tweets]').map(ripple.draw))
```

This declarative paradigm - writing components as idempotent functions of data - enables ensuring you're views will always be up to date and react to any changes in data.

Each microview may have its own styles: 

```html
<twitter-feed css="twitter-feed.css">
```

This allows you to write cohesive styles for each component, but decoupled enough to also allow skinning your component with different sets of styles. 

The styles from the resource `twitter-feed.css` will inserted once in a `<style>` tag at the start of your component root, and will be automatically updated if those styles change. 

You can write your styles as if all your views have [Shadow DOM encapsulation](https://github.com/rijs/shadow). For browsers that cannot create a Shadow DOM, or if you choose not to use that module in your rendering pipeline, the styles will be [translated and scoped](https://github.com/rijs/precss) to the same effect.

<br>
<br>
## 4 Application

Since each view is a function of data, your entire application is also a function of data. For any particular set of resources, there will be one output. This determinism enables many benefits, such as better testing, time travel debugging, hot reloading, universal apps, etc.

The lifecycle of an application is follows:

* HTML page is rendered on server and sent to client
* HTML page opens, then two things happen:
  * Ripple immediately renders the application with the last-known good set of resources from localStorage
  * A WebSocket connections is established with the server and resources start streaming
* As resources start coming in, the parts of your application that are affected will be rerendered

The philosophy here is to maximise optimistic rendering for better perceived performance.

When your application makes any changes, this will be synchornised with the server, then synchronised with any other clients:

```js
ripple('tweets').push({ text: 'Hello World!' })
```

This is what makes Ripple an extension of Flux. Not only is the current page up to date, but all other clients will be kept up to date in realtime too. Everything is "pushed" rather than "pulled".

When the server receives updates, this change will also be broadcast to any other connected services - updating databases like MySQL, caches like Redis, or synchronising in-memory state of other servers over TCP. For example, the above action on the client may result in the following SQL:

```SQL
INSERT INTO tweets (text) VALUES ('Hello World!'');
```

To pipe updates to other services, you just need to pass a connection string when you first configure Ripple:

```js
ripple
  .db('mysql://user:password@host:port/database')
  .db(...)
  .db(...)
```

The [DB module](https://github.com/rijs/db) simply deconstructs the connection string and passes it as an object to the adaptor specified by the protocol (`proto://..`). That module should then initialise the connection and return four CRUD functions which will be invoked whenver the respective changes happen. In this way, you could write your own adaptor for a wide range of services.

<br>
<br>
## 5 Sync

Ripple uses per-resource declarative transformation functions to define the flow of data between server-client. This enables "realtime REST" or "REST over WS". All changes flow through these functions which may return a different representation. The concept is analogous to, but more generic than, "request-response" in HTTP. In request-response, you have to make a request to get a response. If you decouple this, consider that you could receive a response without a request, or make a request with multiple responses, or make a request that does not have a response.

There are two proxy functions (`from` and `to`) which you can define in the headers section:

```js
ripple('tweets', [], { from, to })
```

#### **`to`**

Whenever a resource is sent from the server **to** a client, it will be passed through this first.

`this` is the socket being sent to, first argument is the current resource body to transform. This function is used to send a different representation to a client, if at all. 

For example, returning `false` will not send the resource at all, useful for privatising some resources:

```js
tweets => false
```

The following will collapse and just send the total number of tweets. Whenever there is a change to the `tweets` resource, a push will still be triggered to broadcast to all clients so they are still always up to date, but instead of transferring an array of all tweets, they will just get the total count representation now:

```js
tweets => tweets.length
```

You can vary representations based on authentication. Each socket has a `sessionID` which you could use to lookup whether that user has logged in or not, and send a different representation if so:

```js
tweets => users[this.sessionID] ? tweets : tweets.filter(limit)
```

#### **`from`**

Whenever a resource is received **from** the client it will be passed through this first

`this` is the socket you are receiving the resource from. The arguments are:

* `item` - item in object or array that was changed
* `body` - the full collection (resource body)
* `index` - where the item appears in the body (key/index)
* `type` - type of change that happened to the item in the body (`push`/`update`/`remove`)
* `name` - the name of the resource being changed

This function is used to process a change from a client before Ripple commits it in-memory and broadcasts it to other clients.

Returning false will ignore all changes from the client for a resource:

```js
(..) => false
```

Ignore all type of changes, except adding new items:

```js
(..) => type != 'push'
```

You could choose to ignore whatever change the user made, and take another action instead (Ripple populates the `ip` property on all sockets). Manually updating another resource instead of returning `true` would also trigger a wave of updates to any interested clients/services:

```js
(..) => audit.push(this.ip)` 
```

If a user successfully logins, you could force a refresh of _all_ resources for that user since they may now have access to more resources:

```js
({ username, password }) => login(username, password).then(ripple.sync(this))` 
```

These declarative transformation functions have a very high power-to-weight ratio and the above examples are just a few illustrative examples.

The imperative API for sending all or some resources, to all or some clients is:

```js
ripple.sync(sockets)(resources)
```

The first parameter (`sockets`) could be one socket, an array of sockets, a `sessionID` string identifying a socket, or nothing which would imply all connected sockets. The second parameter (`resources`) could be the name of a resource to send, or nothing which would imply sending all resources.

<br>
<br>
## 6 Events

There is only one event: `change`. Emitterification is achieved using [utilise/emitterify](https://github.com/utilise/utilise#--emitterify) 

* You can listen for changes on a particular resource or all resources (see [Basic API](https://github.com/rijs/docs/blob/master/primer.md#2-basic-api)), using either `on` or `once`. 
* You can namespace events - there is only one listener for each namespaced event.
* You can force listener updates using `.emit('change')`. 

In general, the [reactive module](https://github.com/rijs/reactive) means you never have to manually emit changes. Instead of writing:

```js
ripple('tweets').push('Hi')
ripple('tweets').emit('change')
```

You can just write:

```js
ripple('tweets').push('Hi')
```

Which will implicitly call the `.emit('change')`. This is basically achieved via `Object.observe` for upto two-levels or dirty checking in browsers that don't support that:

```js
Object.observe(resource, c => resource.emit('change'))
```

You can opt-out of this module and manually invoke `emit` after making a change. The only other time you may need this is if you wish to immediately (i.e. synchronously) flush any changes and notify other observers.

The idea is that your data are your first-class citizens, and the focus is on writing declarative business logic rather than the plumbing required to update views, other clients, services, etc. Having the `emit` triggered as an epiphenomenon of a data change operation rather than an explicit command is another step in this direction.

<br>
<br>
#### 7 Conventions

So far, this guide demonstrates registering resources by hand: 

```js
ripple('twitter-feed', function(){ .. })
ripple('twitter-feed.css', ':host { color: red }')
```

But obviously you would want to break these out into different files:

```js
ripple('twitter-feed', require('./twitter-feed.js'))
ripple('twitter-feed.css', file('./twitter-feed.css'))
```

We can take this even further:

For convenience, all JS and CSS files under `/resources` will be auto imported on startup by the [resdir module](https://github.com/rijs/resdir) (ignoring any files with `test`). Once loaded into server, they will be streamed to clients. This means at no point do you actually have to manually register any resources.

Components will registered under the name matching their filename (without `.js`). You can write these files as follows:

```js
// resources/twitter-feed.js
export default function myComponent(data) { .. }
```

Styles will registered under the name matching their filename (with `.css`)

```css
/* resources/twitter-feed.css */
:host { color: red }
```

It is recommend to group relevant component files together into one folder i.e:

```
/resources
  /twitter-feed
    twitter-feed.js
    twitter-feed.css
    test.js
```

You can also break out data resources, writing them as follows:

```js
export default {
  name: 'tweets'
, body: []
, headers: { from, to }
}
```

For each data, you will often just need to define the `from`/`to` functions in that file.

You can also import from other modules which export an array of resources in the same way:

```js
markdown = require('@pemrouz/markdown-editor')

ripple
  .resource(markdown)
```

<br>
<br>
## 8 Hot Reloading

In non-production environments, [resdir](https://github.com/rijs/resdir) loads each file and also watches it for changes. When a change occurs, that resource is simply re-registered. This reregistration emits a change which causes the latest version to be streamed to clients. Your application is then instantly udpated with the latest version of the new component/styles.

<br>
<br>
## 9 Server Side Rendering

Server Side Rendering (SSR) is not one thing. Ripple gives the user the chance to pick different strategies per-component.

* Fallback Content (e.g. "Your browser is not good enough")
* Static Render (e.g. SVG Chart as PNG)
* FOUC Prevention (e.g. children not rendered, but for example widths/heights fixed to content size)
* Semantic Content (e.g. "`<table>`" vs full markup for complex data grid)
* Full Prerender (e.g. SVG Chart or full markup for complex data grid) <br>

SSR is enabled as an express middleware on your server.

HTML files with `<custom-elements>` are expanded (as on the client) with the latest component/data available before being sent.

This means it will result in a full prerender by default for all views.

You can force a different strategy in your component, for example:

```js
if (process.ua < acceptable) return fallback
```

Microviews are seamlessly updated on the client as and when subsequent changes happen and are streamed to clients.

On the client, if there is anything in the light DOM it is switched over to the Shadow DOM before re-rendering so elements inserted into the server are not recreated.

<br>
<br>
## 10 Compatibility

Officially, Ripple v0.3 supports the latest version of each browsers. However, most modules run on any platform. This is the idea behind publicising compatibility of different modules with popper to allow users to knowingly opt-in and out of features to suit their requirements. The only module that does not work on older browsers is the [components module](https://github.com/rijs/components). The Web Component affordances (attached, detached, attribute changed callbacks) are currently polyfilled by `MutationObservers`. This could be solved in a more generic way like the [reactive module](https://github.com/rijs/reactive) does with `Object.observe`. However, it might be better to just slightly adjust your development pattern. Whenever you update a Custom Element, just remember to subsequently invoke a `ripple.draw` on that node. This is the same as if you weren't using the reactive module and you manually had to invoke `.emit('change')` after making a change.

![image](https://cloud.githubusercontent.com/assets/2184177/8728132/6a211df0-2bdb-11e5-8295-cb2e9f836203.png)
_Snapshot of Test Results for Ripple v0.3 on latest Chrome, Firefox, IE, Android and iOS_
