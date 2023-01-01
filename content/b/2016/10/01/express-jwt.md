+++
title="Adding JWT authentication to an Express API"
date=2016-10-01
+++

One of my most recents tasks was to add some sort of user authentication to an
API built with [Express](https://expressjs.com/) running on an embedded device.
I'm used to working either with Rails, where these kind of things are mostly
provided by gems that make all the decisions for you; or on small Node projects
which didn't require authentication at all. So this was new and required me to
do some research about alternatives and implementation details.

## Stateless and stateful authentication

An important distinction in authentication methods for this particular case is
whether they need to keep track of state or not. With stateful authentication,
upon confirming the identity of the user the server generates a key to identify
them, provides this key to the user and **saves internally which keys are
assigned to which users**. This way, subsequent requests which include that key
can be traced to the corresponding user to know who's making them.

Stateless authentication, on the other hand, relies on the user providing _all_
the information about their own identity. **The server doesn't need to keep
track of any data about users that have "logged in"** as in the previous case.
Everything the server know about the current user is sent by the users
themselves. Obviously, this implies that the server has to somehow trust that
the information the user is sending is reliable.

I was particularly interested in the lack of tracking on the server side when
going stateless. While the device the API was running on wasn't particularly
restrictive, the less its memory, storage and processing requirements were, the
better. In stateless authentication, we save in storage (no need to keep track
of the sessions being stablished for each user) and processing (no need to look
up a user from a database, all the information needed comes in the token sent
from the client).

Authentication methods are way more complex than what I've described here, and
I'm by no means an expert. I can make a recommendation though:
[Auth0's blog](https://auth0.com/blog/) is full of resources from people who
make a living out of the authentication of applications.

## Implementation

[JSON Web Tokens](https://jwt.io/) are a standard for securely transmitting data
and can be used to allow a server to provide stateless authentication to its
clients. The process is fairly simple:

1. The client provides the server with its credentials.
2. If the credentials are valid, the server returns a token which is signed with
   a secret key known only to the server, and the client stores it in any way it
   wants.
3. On each subsequent request, the client has to send the token to the server.
   This token includes any data the server wanted to provide, which should be
   enough to avoid roundtrips to the database so it'll probably include stuff
   like the username and the user's roles.

Just a quick note - I've mentioned that only the server knows the secret key
used to sign the payload. I've read that there's a variation using an asymmetric
key on which both server and client know one part of it. I didn't need this so I
don't know the details, but it's something that's out there too and might be
useful for other use cases.

We can see that the server has two responsibilities: **generating the tokens**
(i.e. signing a payload) and **decoding the tokens back into the original
payload**. This means that the server also needs to handle users somehow, but in
principle the way it does that makes no difference (although if you're using
something like [Passport](https://www.npmjs.com/package/passport) there might be
better ways to add JWT).

### Generating tokens

In my case I'm using a very simple file-based database where the users are
stored. For the JWT part I'm going to use a very simple package called
[`jsonwebtoken`](https://www.npmjs.com/package/jsonwebtoken). It allows encoding
and decoding JWTs but in this case we'll use it only for encoding.

Let's start with creating an endpoint for generating the token, which would be
the API's version of a user signing in. We'll setup a route in Express similar
to the following:

```js
const router = express.Router();
router.post("/token", generateToken);
```

The handler for the route will simply take the username and the password from
the payload and pass it to the controller.

```js
function generateToken(req, res, next) {
  const username = req.body.username;
  const password = req.body.password;

  return controller
    .generateToken(username, password)
    .then((token) => res.status(200).send(token))
    .catch(next);
}
```

Now let's dive into that `generateToken` method in the controller:

```js
const jwt = require("jsonwebtoken");

/* ... */

function generateToken(username, password) {
  // First we try to find our user.
  const user = User.findByUsername(username);

  if (!user || !user.passwordMatches(password)) {
    // We use the same error either if the user is not found or if the password doesn't match.
    // This way, if someone is trying to list users by bruteforcing the authentication endpoint,
    // they won't know whether they found an existing username or not.
    throw new Error("User not found");
  }

  return new Promise((resolve, reject) => {
    jwt.sign(
      {
        id: user.id,
        username: user.username,
        role: user.role,
      },
      process.env.AUTHENTICATION_SECRET,
      { expiresIn: "7d" },
      (err, token) => {
        if (err) {
          reject(err);
        } else {
          resolve(token);
        }
      }
    );
  });
}
```

Notice that you can send anything you want in the payload being signed. Try to
keep it as small as possible, since this will be sent back on every request from
the client, but add any information you need so that DB lookups for that user
are rare.

Also, you can see that the secret key coming from an environment variable in
`process.env.AUTHENTICATION_SECRET`. Always load your secret keys from
environment variables or, at the very least, make sure they're not checked in in
version control.

The rest of the code just sets an arbitrary value for the expiration of the
token and makes sure that the promise works as expected. Since the server is not
keeping track of the generated tokens, an existing token can't be "revoked" by
deleting it from anywhere. **Revoking tokens is harder than in stateful
authentication**, so make sure to add a reasonable expiration date for your
tokens to automatically become invalid over time.

Having authenticated a user and given them their token, let's see how to
validate that token when it comes back and extract the payload from it.

### Decoding tokens

Before, I mentioned that if you're using a package for managing authentication
then probably that package already has a way to use JWT instead of doing it as
"manually" as above. For decoding tokens, we're going to take advantage of the
fact that the API is an Express application and use the
[`express-jwt`](https://github.com/auth0/express-jwt) package which provides JWT
validation and decoding as an Express middleware. Once installed, it can be used
as any other middleware. For instance, we can use it wherever we define the
application's middleware stack:

```js
const jwt = require("express-jwt");

app.use(someMiddleware);
app.use(someOtherMiddleware);
app.use(
  jwt({ secret: process.env.AUTHENTICATION_SECRET }).unless({
    path: ["/token"],
  })
);
```

Make sure that you're using the same secret you used for signing the tokens.
`unless` is useful to make some routes accessible without authentication, which
is needed for the endpoint used to generate the token, but it could include
other endpoints depending on your business rules.

The middleware will (by default) look for the token in the `Authorization`
header - it will expect the requests to have a header looking like
`Authorization: Bearer token1234` where `token1234` is the actual token, and
will store the user in `req.user`. If the token can't be decoded, or is invalid
according to the app's secret, the request will fail. You'll want this
middleware to execute as early as possible for each request so that the user is
available as soon as possible or, if the token is invalid, so that the request
will fail quickly.

---

With that done, we already provided users with a way to get authentication
tokens, and to use those tokens to make requests to our API. I want to stress
again that I'm not an expert on authentication by any means, and this is just
what I've learned while doing some research for implementing a specific use
case - but I hope it can be useful to someone else at least as a first take on
how such a feature can be implemented.
