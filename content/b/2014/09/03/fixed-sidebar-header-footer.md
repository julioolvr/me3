+++
title="Fixed sidebar with fixed header and footer"
date=2014-09-03
+++

These are a couple of ways of constructing a layout that consist of a fixed
footer, header, and sidebar. The application we were building had a preview to
the left and the fixed sidebar to the right to tweak that preview. The problem
to solve is how to give the sidebar the proper height so it will scroll its
content correctly.

The first solution makes use of CSS's
[`calc`](https://developer.mozilla.org/en-US/docs/Web/CSS/calc). We want our
sidebar's height to be 100% MINUS the header and footer's height. It's not
uncommon for the header's height to be fixed; and given that the footer in this
case is fixed too, it probably will also have a fixed height. For this example
they are 40px each, so it's as simple as giving the sidebar a
`height: calc(100% - 80px)`.

<Codepen id="klvdG" />

One important advantage of this approach is that, since the sidebar's real
height is what it's supposed to be (and not bigger as in the following
approaches). This is nice because that way, you see the whole scrollbar.

If `calc` can't be used, another approach is add a padding to the sidebar.
Again, very simple, just add a top and bottom padding that are equal to the
header and footer's height, 40px in this example:

<Codepen id="fLjBy" />

This works in _most_ browsers, although you can see the cropped scrollbar.

Now, let's say you need it to work on another browser. Let's say that browser
is, for instance, IE8. I'd usually rant about IE, but in this case they are not
THAT wrong. As mentioned in [this](http://stackoverflow.com/a/5804795)
StackOverflow answer,
[the spec](http://www.w3.org/TR/CSS21/visufx.html#overflow) only says that the
browser should scroll to show the clipped CONTENT. It makes sense not to
consider a padding as content. The result? IE8 won't scroll to show the
"padding" (that would end up behind the footer anyway), and the last of the
sidebar's content ends up hidden by the footer.

To solve that problem, the third and final solution involves adding `:before`
and `:after` elements with the height of the header and footer, and THOSE will
be the ones that end up hidden. This has the same issue with the scrollbar as
the previous solution, but supports IE8:

<Codepen id="dtqxE" />

And that's it, I hope this saves someone some time :)
