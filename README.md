# spirit-express
Wraps a Express (or Connect) middleware to make it compatible with spirit.

Wrapped Express (or Connect) middleware and spirit middleware can be used in conjunction with each other, it's not one or the other.

It is _not_ meant to bring full Express API compatibility, but enough for most Express middleware to work, thus reducing extra overhead from Express API that normally are never used.

[![Build Status](https://travis-ci.org/spirit-js/spirit-express.svg?branch=master)](https://travis-ci.org/spirit-js/spirit-express)
[![Coverage Status](https://coveralls.io/repos/github/spirit-js/spirit-express/badge.svg?branch=master)](https://coveralls.io/github/spirit-js/spirit-express?branch=master)

## Usage
`npm install spirit-express`

```js
const compat = require("spirit-express")
compat(express_middleware) // use anywhere that supports spirit middleware
```

For specific examples, see the [examples dir](./examples)

NOTE: it is almost always better to use a "native" spirit middleware over using this module to wrap a Express middleware (if one exists).

## Supported Express middleware
Note, this list is not exhaustive, but the following have been tested:
- passport
- body-parser
- cookie-parser
- multer
- express-session
- webpack-dev-middleware & webpack-hot-middleware

Express error handling middleware `(err, req, res, next)` are not supported on purpose. Instead handle errors the conventional way by using Promise `catch`. Note that `next(err)` is supported and will just throw the err.


## Doc
This module wraps a Express middleware so it runs like a spirit middleware.

It also makes changes to how a request and response are handled.

#### request
The normal request object in spirit is copied over to the original `req` object from node (http.IncomingRequest).
This "req" object is what is passed to Express middleware _and_ spirit middleware.

So the `req` object replaces the spirit `request` throughout once a Express middleware is used.

Note that the `req` does __not__ support any Express related req API, which in most cases is not needed. Most Express middleware avoid using the Express req API so they can be compatible with Connect.

#### response
The `res` object passed to Express middleware is not a node `res` object. But instead, it's a object that mocks certain common properties and methods that exist on a node and Express `res` object.

In spirit, middleware flow once on input, and once on output (bidirectional). This is in contrast to the Express model where middleware only go one way. To make it compatible with the spirit model, all changes a Express middleware would normally do on the `res` object are stored and accumulated. 

The stored changes will be set __once__ by the first Express middleware that is encountered on flow back (which would be the last Express middleware, depending on how you think about it).

The above is only true when a Express middleware makes partial changes to the `res` object. If a Express middleware terminates (because of `res.end` or `res.writeHead`) then it would not continue moving forward, but instead immediately return a response.

The following Express methods are supported:
- res.redirect
- res.send
