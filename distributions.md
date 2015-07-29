# Distributions

On the client, there are four flavours of ripple you can use depending on how you plan to package your application. 


* `ripple.js` - all-in-one, unminified
* `ripple.min.js` - all-in-one, minified
* `ripple.pure.js` - does not include external dependencies (socket.io and utilise), unminified
* `ripple.pure.min.js` - does not include external dependencies (socket.io and utilise), minified

These are all also exposed as endpoints on your server.

Bundling is done with Browserify and UglifyJS. If you wish to produce your own bundle, you can start by taking a look at the "npm run build" command.