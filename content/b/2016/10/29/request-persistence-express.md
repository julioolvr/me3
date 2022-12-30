+++
title="Request-scoped variables in Express"
date=2016-10-29
+++

One common requirement in web applications is keeping track of the currently
logged in user throughout the execution of a request. While this is fairly
simple to do in web servers where each request has its own execution environment
(be it a thread or a process), Node's single-process single-thread model makes
it less straightforward - but it can be achieved with the help of
[`continuation-local-storage`](https://github.com/othiym23/node-continuation-local-storage).

Say we have a secure Express application. At some point, a middleware is going
to authenticate the user and save it somewhere for future use (again, I'm going
to refer to my
[earlier post about using JWT with Express](/b/2016/10/01/express-jwt) since I
had to implement this on the same app). In this case, after authentication is
done, we'll have the user available on `req.user`. Good so far, but the request
object is only available to middlewares and route handlers, and to use it
further down the chain we'd need to pass it all the way down through function
parameters. In some cases this might make sense, when conceptually each of those
functions is related to the user executing them. But here the user was needed to
make a permissions check on the opposite boundary of the application - before
making a request to an external system, to make sure they had the right
permissions to do it.

So if passing the user as a parameter is not conceptually sound, then how do we
get it all the way through to the code making the requests to the external
service? Here is where `continuation-local-storage` helps. Let's see first how
to use it and then try to understand exactly how it's doing it. First, we need
to create a namespaces in which to store our data. Even though the namespace
will be the same for every request, **the values will be scoped for each chain
of functions**.

```js
const { createNamespace } = require("continuation-local-storage");

const session = createNamespace("request");

// Assuming we have our express app in `app`
app.use((req, res, next) => {
  session.run(() => next());
});

// And once we have authenticated the user
app.use((req, res, next) => {
  session.set("currentUser", req.user);
  next();
});
```

See that I called the namespace `request`. `continuation-local-storage` isn't
really specific for a web server environment - **it works for any chain of
functions calling each other** - so I gave it a name representing exactly what
it was being used for in this case.

Then we have to define a couple of middlewares. For `continuation-local-storage`
to work, **it needs to wrap the chain of function calls on the namespace's
`#run` method**. That means that that first middleware has to be defined as
early as possible in the middleware chain. **It's only inside the call stack of
that method that values can be set or retrieved from the namespace**.

The second middleware, once the user is available somewhere (`req.user` in this
case), sets it in the namespace. And that's all that's needed for the setup.

The usage is way simpler since we'll already be inside a `#run` chain, so we
just have to get a reference to the namespace and fetch the value from it:

```js
const { getNamespace } = require("continuation-local-storage");

function getCurrentUser() {
  return getNamespace("request").get("currentUser");
}
```

As long as `getCurrentUser` can be traced all the way back to the `#run` method,
then everything put on the namespace will be available.

## How does it work?

Personally I was curious on how they achieved this behavior. How do they keep
variables across functions calling functions calling functions, many of those
asynchronous - going to the OS and back?

The answer is [`async-listener`](https://github.com/othiym23/async-listener).
`async-listener` is a package which allows us to set callbacks for the lifecycle
of asynchronous operations. **So when an asynchronous operation is queued, when
it fails, and right before or after our callbacks are called, we can add custom
behavior**. `continuation-local-storage` uses it to keep track of each execution
context and make the namespace values available again once we're back from the
asynchronous operation.

Then, how does `async-listener` does it? The answer is simple, although the
implementation is fairly complex: **wrap every asynchronous function in node to
be able to provide those callbacks**. You can
[take a look at the code to see how they did it on each case](https://github.com/othiym23/async-listener/blob/master/index.js),
with some of the solutions being quite involved (see how they wrap promises!).

## Links

- [`continuation-local-storage`](https://github.com/othiym23/node-continuation-local-storage)
- [`async-listener`](https://github.com/othiym23/async-listener)
