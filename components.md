# Guide to Writing Components
 
Ripple is itself completely agnostic to how you structure your components. In most examples, you will see `.innerHTML =` for brevity. However, given the importance of the issue this article aims to bring together all the best practices and presents a practical developer guide to writing components that achieves a range of goals:

* Framework Agnostic (Widely Reusable)
* Performant (avoid DOM churn)
* Universal/Isomorphic
* Concise and Declarative
* Easy to Test
* Easy to Extend and Maintain

There have been huge changes in this area, so there is a short background to put some of the ideas into context for those who haven't been following the latest changes from the UI trenches. 
 
## String Replace
 
```html
el.innerHTML = '<div>{change-me}</div>'.replace('change-me', 'content')
```

Perhaps the simplest approach that could be envisaged towards rendering is loading a HTML template as a string and replacing parts of it or otherwise by directly manipulating innerHTML. This can obviously become very cumbersome, and doesn't scale well if you need to recursively update templates.
 
## jQuery
 
```js
$(el).append($('<div>').html('content'))
```

jQuery made raw DOM manipulation easier through a finer-grained element API and method chaining. You could point at an element and poke it till it looked like you wanted. The problem with hacking components with jQuery was that it would eventually lead to "spaghetti code". This was at least partly a probem with the jQuery philosophy itself. It was very imperative. You couldn't run a sequence of rendering steps (e.g. appending elements) more than once, which meant spawning an exponentional number of code paths based on different possible states and executing similar but slightly different rendering steps. The intricately stateful components would often become difficult to maintain in large applications.
 
## Templating
 
```html
<div ng-repeat="(key, value) in myObj"> ... </div>
```

One development on raw string processing is to use templating. This provides a dedicated syntax for common use cases and the pro is that it is declarative. The downside with templating tends to be that:
 
* It always reinvents an inferior and non-standard subset of JavaScript (e.g. a syntax for looping)
 
* There is no extension point. It is a complete dead-end if you want to do something not catered for by the syntax and you need to fallback to a different approach.
 
* It mostly ends up being server-side only. You are unlikely to use templating on the client for subsequent interactions as you have to recalculate and replace a huge HTML chunk, more than is necessary (JSX and Glimmer are exception to this since they have another layer (React and Ember) to commit only the diffs to the DOM - but more on that later).
 
## D3
 
The contribution of D3 was that you could now write **declarative JavaScript** components. You don't have to care what state your component is in and embark on a different code path - the problem with jQuery. If you need to blend in advanced customisations, you are not limited by the syntax of your declarative markup. Given some data, it would always produce the correct representation on the DOM. Consider changing the rows variable and rerunning the following example:
 
```js
var join = d3.select(this)
  .selectAll('tr')
  .data(rows)
 
join.enter()
    .append('tr')
 
join.text(String)
 
join.exit()
    .remove()
```

This effectively has the performance benefits as a Virtual DOM. The concept can be explained as "rubber stamping" - if you try to rubber stamp the DOM with the same component twice, it won't append any new elements.

The paradigm shift to **transformation functions** (or unidirectionality in React) was huge and is similar to the container revolution in shipping or deployments. The constraint on the remit on which a component could operate meant you could stack them together much more robustly to build an application.
 
## Data Drive Everything
 
D3 was revolutionary, but there were few rough edges to iron out. First, doing the join dance can be quite verbose for almost every element. This led to a few half-measure implementations, using a more primitive approach for "structural elements" that don't change often (e.g. table headers) and data-driving the "variable parts" (e.g. table rows). The solution to this was to roll your own micro-utility that simplifies the boilerplate (enter/exit/remove) to lower the barrier to always using the declarative approach. I use [once](https://github.com/utilise/utilise#--once), which can be summarised as an **idempotent data-driven microlibrary**. It's incredibly versatile, using lots of convenient defaults as well as providing breakout capacity by returning the raw D3 selection in case you need to do any advanced customisations:
 
```js
once('tr', [1,2,3]).text(String) // same code as above, generates:
```
```html
<tr>1</tr><tr>2</tr><tr>3</tr>
```

In less trivial examples, it has a composable and terse lispy syntax:

```js
once(node)                        // limit to this node
  ('div', { results: [1, 2, 3] }) // creates one div (with the specified datum)
    ('li', key('results'))        // creates three li (with datum 1, 2, 3 respectively)
      ('a', inherit)              // creates anchor in each li (with parent datum)
        .text(String)             // sets the text in anchor to the datum
```

A few unexpected benefits of this approach:
 
* We need to prioritise being agile with respect to dealing with changing requirements. Favouring stronger lower-level grammer, `once` makes it cheaper to write smaller, even **disposable components/features**, rather than continually modifying a super-component that solves all the problems in one. You can only extend a generic reusable component so much with flags, before it becomes brittle, a maintenance nightmare and snaps. 
 
* The other cool thing about this was that it seamlessly works with server-side rendering (SSR). You could run that line on the server to prerender a view, then rerun it on the client and it would only touch the DOM where it needs to. See [The Perfect Render](https://rijs.io/hybrid) for an overview of SSR approaches.
 
## Web Components
 
The second issue with the original D3 Components was that they had a JS API and still stored some state inside a closure (via the accessor functions). This is limiting since it can define an arbitary API surface, thus forcing the application developer to write integration glue between components and keep hold of JS references. It was also impossible to listen to events from another module unless you had acess to that JS reference (see Twitter Flight which embraced the DOM for events). The idea is to be able to change data/state and redraw like a game loop. This is where components as pure functions dovetail with the new Web Components very well. You can declare a Custom Element in markup, and then let the browser (or polyfill) upgrade them with the appropiate transformation function (`twitter-feed`), dependency injecting any data it needs (`tweets`):
 
```html
<twitter-feed data="tweets">
```

The other issue with a separate JS instance is now you have two things competing with each other: The DOM Element and the JS Instance. You don't need both. An `<input>` is a component, with API like `.value`. Likewise, Custom Elements can have their own API `<input is="crazy-input">` and a higher-level `.crazyValue`. If you need to interact with a custom element, you can just select it (e.g. `<twitter-feed>`) and it use its own public API for JS interaction.

## Data

Loading data is not the concern of your component internals. In Ripple, data dependencies have always been expressed per view via the data attribute and dependency injected into the component for you. Falcor and Relay are recent approaches in this area that tackle the same problem (see the [hypermedia module]() for further comparisons on this).

## Local State

Transformation functions should be stateless - they should not hold onto anything inside the closure. Data is available via the arguments, but besides this you will need local state. This should be persisted on the element itself as D3 does with it's data:

```js
var state = this.state = this.state || { scroll: 10, focused: false }

// rendering sequence
sel(this).classed('is-focused', focused)

// later, perhaps in event handler
state.focused = true
ripple.draw(this)
```

The choice of whether something is data or state depends on your requirements. For example, having the scroll position stored in local state means it will default to a value on the first render. If you wanted to drive this via the URL for example, you would move this out of local state to the global data.

## Event Handlers

Since a view is just an element you can publish/subscribe events via the DOM API. However, Ripple also [emitterifies](https://github.com/utilise/utilise#--emitterify) them by default giving you light-weight cross-browser compatible synthetic events that are a bit more concise (`on/emit`):

```js
// subscribe
this.on('change', d => {
  state.count += 1
  ripple.draw(this)
})

// publish
this.emit('change')

// or, via DOM:
sel(this).on('click', ...)
```

The most important thing is that you should never perform any ad-hoc DOM operations in event handlers. This may be tempting to begin with, but you should adapt to the declarative mindset: there should be be only one rendering sequence. Your event handlers change either local state (`state.title = 'meh'`) or global data (`ripple('tweets').push('tweet')`) and then redraw themselves (`ripple.draw(this)`) or emit a relevant change event (`ripple('tweets').emit('change')`) which will redraw all affected views.

## FRP

You can also use the [frp module]() which enhances elements with the `.events` API that allows you to respond to a stream of events in a more FRP-style:

```js
this
  .events('click')
  .filter(..)
  .map(..)
  .reduce(..)
  .map(d => ripple.draw(this))
}
```

## Summary

Like most of the JavaScript community, the range of options and techniques when it comes to rendering are quite large and diverse. Given the current state affairs, the conclusions can be condensed into the following three practical recommendations:

* Write your components as pure functions of data for greater flexibility and so they can be used in a greater range of contexts beyond your immediate application:

```js
export default function twitterFeed(tweets){ .. }
```

* Write your components as idempotent functions by internally using `once` so they will be efficient:

```js
once(this)
  ('li', tweets)
    .text(String)
```

* Favour the standard HTML syntax for instantiating components rather some proprietary syntax:

```html
<twitter-feed data="tweets">
```

## Feedback

If you agree/disagree or have any feedback on this topic/article, please let me know on twitter (@pemrouz) or via GitHub issues!

---

# Addendum 

I took these sections out of the main article, but thought they may be of interest for any one thinking how this compares with other approaches currently in fashion:

## React
 
The Facebook team, similarly working on a large JS projects, discovered some similar UI ideals and can be largely credited with bringing this idea from the fringe to the mainstream. However, whereas React takes over your entire view layer, the components espoused here are independent, loosely decoupled and framework agnostic. React is 40kB minified and gzipped. Where React is strongly OOP, the components here are the functional equivalent and can be compared to just being the `render` function, or [the new simpler stateless functional syntax](http://facebook.github.io/react/blog/2015/10/07/react-v0.14.html#stateless-functional-components). 99% of the DOM churn is avoided with D3 Joins, and the Virtual DOM diffing aspect can be bolted onto the end of your rendering pipeline in Ripple as a module extension. JSX only makes it less painful to generate HTML markup in JS, but once/D3 gives you three architecturally useful selections (enter/update/exit) upon which you can selectively apply operations, or create further semantic subselections from. It also does this without requiring any compilation.

## Virtual DOM
 
More modular Virtual DOM libraries are great and avoid most of the above issues with React, but they are basically a one-to-one mapping with the DOM and so operate at a lower level than `once`, which is useful for driving your views based on your data. 

```js
once('ul')
  ('li', [1, 2, 3])
```

```js
h('ul', [
    h('li', { data: 1 }),
    h('li', { data: 2 }),
    h('li', { data: 3 }),
])
```

```html
<ul>
  <li></li>
  <li></li>
  <li></li>
</ul>
```