+++
title="Unexplained syntax error on PhantomJS with strict mode"
date=2015-07-24
+++

Short story: PhantomJS considers a syntax error when a named function expression
is called the same as some of its parameters, and won't be very explicit about
it.

Consider the following code:

```js
"use strict";
var something = function bug(bug) {};
```

The fact that the function is called `bug`, and the parameter is also called
`bug`, will trigger a syntax error in PhantomJS. Note that a function
declaration has no problems with it:

```js
"use strict";
function bug(bug) {}
```

No syntax error there! But when there _is_ a syntax error, PhantomJS will only
say so and won't specify what the error is. This can be kind of annoying when
your automated tests use Phantom and fail miserably while the app works fine on
your browser. One good way to find the cause for these kind of bugs is that
PhantomJS behaves very similar to Safari, and this specific bug is also present
there. So if you happen to develop on Safari then you'll see it before it gets
to Phantom, but if you don't it's a nice trick to remember the next time you
have to debug a Phantom-specific bug.
