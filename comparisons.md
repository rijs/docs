## Rationale

The Ripple pattern evolved naturally from developing web apps as "game-loops" that were recursively composed of [idempotent components](http://ag.svbtle.com/on-d3-components) (i.e. components as simple transformation functions of data - `html = f(data)`). Reinvoking the function with new data would always produce the latest representation of that component, so rather than each component setting up it's own plumbing, listening to data change events to receive/dispatch data, the concern of _streaming data to components_ can be effectively abstracted out. This makes realtime components the default behaviour, not an afterthought. Most alternatives in this area also did not make any attempt to be a lightweight library, embracing standards and native behaviour but went down the road of being a heavyweight framework, inventing concepts and large API surfaces. Applying RESTful principles in the client - the concept of organising applications from _resources and their different representations_ - is also a key part of the philosophy, which currently seems relatively unexplored in other JavaScript projects.

### Ripple vs Flux

Ripple shares key architectural concepts with Flux, such as the single dispatcher, data-flow programming (unidirectional), and views updating when the associated data changes, etc. However, **Ripple is an extension to the Flux paradigm**, in that the dispatcher will not update only the data/views on the current page, but on all other clients too. Ripple introduces much less proprietary concepts (everything is a resource) and the API aims to embrace standards rather than invent new ones.

### Ripple vs Meteor

Ripple and Meteor share some key benefits, such as reactive programming, hot code push, database everywhere, etc. The key difference however is that **Ripple is a _library_ whereas Meteor is a _framework_** - one that takes over your entire development fabric and locks you in. There is no [inversion of control](http://martinfowler.com/bliki/InversionOfControl.html) with Ripple and you can use it many different ways. Besides that, the implementation of the aforementioned benefits is more powerful in Ripple: For example, not being tied to a particular templating engine (Spacebars) and the Meteor 'database everywhere' is just a 'prototyping concept' [removed for production apps](https://www.meteor.com/try/11) requiring [additional wiring that defeats the original friction-reducing purpose of having it](https://www.meteor.com/try/10). In contrast, Ripple's [generic proxies](#rippleresourcename-body-opts) allows you to filter out 'privacy-sensitive data' and the overall architecture gives you 'latency compensation' for free (changes to data update dependent views immediately, and if the change was unsuccessful there will be another invocation to put it in the correct state).

### Ripple vs Compoxure

Ripple and [Compoxure](https://medium.com/@clifcunn/nodeconf-eu-29dd3ed500ec) are very similar in decomposing applications in terms of independent resources. Like Compoxure, Ripple can call out for resources from separate micro-services, avoiding the monolith criticism (1), it can pre-render views avoiding the SEO-incompatibility criticism (2) and it's doesn't lock you in to using a particular hybrid-approach like React or rendr (i.e. you could call a Java service to generate the resource) (3). The key difference is that whereas Compoxure is only concerned with the first render and requires manually wiring up event listeners and AJAX calls for updates, **resources in Ripple are long-lived and continue to send/receive updates after the first render**. Ripple does not currently cache resources in Redis (but that's on the roadmap).

### Ripple vs Basket

Ripple and Basket both using localStorage for storing and loading from. Basket does this on a script-level however, whereas Ripple does this on a resource-level. Basket uses localStorage as an alternative to the browser cache, whereas Ripple uses it for the initial page render and then re-renders relevant parts when there is new information available sent from the server.

### Ripple vs Polymer 

Ripple and Polymer both embrace Web Components for composing applications, but beyond auto-generating Shadow DOM roots for upgraded Custom Elements and transparently encapsulating styles for non-Shadow DOM users, Ripple does not provide anywhere near the same level of sugar as Polymer on top of Web Components.

### Ripple vs Redux

The Ripple core is comparable to but lighter than Redux, whilst achieving the same design goals in a more modular and efficient manner (developer experience, hot reloading, time travel, universal apps, record and replay). Despite Redux trying to eliminate boilerplate in concepts from orthodox React/Flux, it is still strongly entrenched with some legacy concepts and a particular way of doing things (reducers, action creators, etc). The basic Ripple API allows the user to choose more options (e.g. immutability) and opt-in to modules.

### Ripple vs loadCSS

The Ripple architecture results in the same behaviour as loadCSS offers. CSS modules (in fact all resources) are loaded asynchronously (streamed) and appended in order to preserve cascading. If you are using Shadow DOM, they will be added at the start of the shadow root and to the head of the document if not (see [PreCSS](https://github.com/rijs/precss) for more info). In addition (i) only the modules you are currently using will be appended (ii) this is not something you need to manually manage at all but is inferred from a declarative syntax.

### [Ripple vs React](https://github.com/rijs/docs/blob/master/components.md#react)

### [Ripple vs Virtual DOM](https://github.com/rijs/docs/blob/master/components.md#virtual-dom)