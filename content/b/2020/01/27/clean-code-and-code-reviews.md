+++
title="Clean Code and Code Reviews"
date=2020-01-27
+++

[Dan Abramov](https://twitter.com/dan_abramov) recently published a blog post
called ["Goodbye, Clean Code"](https://overreacted.io/goodbye-clean-code/),
which looks back into an old work story and reflects on two things:

- The pursuit of "clean" code just for the sake of it (and how it can actually
  backfire).
- The team dynamics around that episode.

I'd like to focus on the second part because it's something I've been thinking
about a lot.

To keep it brief (do read Dan's post, it's a short one and really drives the
point home), the story is about a specific implementation by a coworker where
Dan identifies duplication as something undesirable. He makes some changes, and
then:

> I checked in my refactoring to master and went to bed, proud of how I
> untangled my colleague’s messy code.

Lately, I've been thinking about code reviews and their purpose. I want to
identify ways we can be better at them and, in order to do that, we need to
understand _why_ do we have a code review process in the first place. I like
this story because it highlights a couple of places where code reviews might be
particularly useful.

Quoting from the article again, something I wholeheartedly agree with:

> A healthy engineering team is constantly building trust. Rewriting your
> teammate’s code without a discussion is a huge blow to your ability to
> effectively collaborate on a codebase together.

Even if the code review process might be skipped in some exceptional cases, it
most likely won't be for pure refactoring work. This means that, **just by
following the process, a situation where one rewrites a teammate's code without
any discussion won't happen.**

Another aspect that's often mentioned as an advantage of going through a code
review process is that of learning. Sometimes that learning implies learning
something new - some new API, or a new approach or even design pattern that can
help tackle a problem. In this case, the post reflects about how reducing
duplication doesn't necessarily lead to objectively _better_ code, and how it's
instead a series of trade-offs. While it's possible that this wouldn't have come
up during the review, if it did, it would've been a kind of learning where we
question our own assumptions. Instead of only learning something new, we shed
new light on something that we thought was clear. **What initially might have
been a routine comment, after being challenged becomes an opportunity to
reevaluate and learn.**
