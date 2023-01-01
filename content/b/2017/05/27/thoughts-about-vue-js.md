+++
title="Thoughts about Vue.js"
date=2017-05-27
+++

I recently attended a workshop about Vue.js, organized by
[@workshopsjs](https://twitter.com/workshopsjs) and given by
[@ianaya89](https://twitter.com/ianaya89). This post is an attempt to organize
my thoughts regarding the framework, as basic as they might be by having such
little exposure to it.

First of all, the exercises are very good and Ignacio made a really good job
making them simple to follow along, so I recommend to anyone wanting to give Vue
a try (at least those of you that understand Spanish) to take a look at it at
[github.com/ianaya89/workshop-vuejs/](https://github.com/ianaya89/workshop-vuejs/).

My immediate thought after seeing some first code samples is that **the
resemblance to Angular 1 is evident**. I've worked with Google's framework for a
couple of years and being familiar with it certainly eased the learning curve,
as some of the first concepts such as the binding and directives are similar.

That being said, Vue was created in a different context - in a world where we
are starting to see the huge benefits of component-oriented frontend
architectures. While it's possible to develop that way in Angular 1, the
framework doesn't naturally lead you that way (at least before Angular 1.5).
Directives rule, and they double as components and as DOM manipulation tools.
Angular 2 seems to have made the distinction between components and
DOM-manipulating directives more obvious, and so does Vue. This seems to **lead
to a better, more natural separation of the UI in components**, and that's
something I consider crucial after working with Angular pre-1.5 for a while.

**The colocation of HTML, CSS and JS for each component is also something I like
and that is present in Vue.js' .vue files**. It seems to be possible to
preprocess each part as well, and there are some ways to integrate JSX into it,
but I haven't looked into these two.

**Vue as a library is architectured in layers** - the core includes the
rendering and component systems, and everything else is built on top of that.
Much is supported officially, but it does mean that **the resulting application
will only be as complex as it needs be**. Routing, state management and others
can be added if necessary and left out if they aren't, resulting not only in a
reduced size overhead but also on a smaller cognitive load. **You only need to
learn the concepts you need**.

There's a CLI which I used in the workshop to create the project,
[`vue-cli`](https://github.com/vuejs/vue-cli), that was very useful to generate
a basic setup which uses Webpack to transpile `.vue` files into plain JS and
CSS. It seems to do its job well, although I haven't used it more than that so I
have yet to see what other types of projects it generates. It is possible to
generate projects according to different levels of complexity, making use of the
layers I mentioned before. While I created the simplest one, there are others
which already includes things like state management and a more opinionated
directory structure.

In conclusion, Vue seems to be Angular 1 done right - even though Angular's
newest versions have improved on top of the initial release, the API surface is
still huge and using it without TypeScript is not that supported (even though it
should be technically possible). **Vue shows up as smaller, simpler alternative
to Angular, improving on top of Angular 1 while at the same time staying
conceptually lean**. This allows some immediate benefits (a flatter learning
curve) and some not so obvious: for instance, the fact that state management is
not baked in means that Vue can benefit from new practices in this area (e.g.,
[`vuex`](https://github.com/vuejs/vuex) is a Flux-like state management solution
for Vue). It should be possible to implement other patterns into Vue as long as
they don't impact the core rendering/component systems.

Even though I'm not sure if I'd switch from React to Vue, I wouldn't mind
working with it and I'll certainly keep an eye on the library to see where it
goes.
