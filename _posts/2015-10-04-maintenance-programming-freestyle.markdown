---
layout: post
title: "Maintenance development - freestyle"
date: 2015-10-04 13:30
comments: false
---

How do you get started doing effective maintenance development? This topic is near and dear to me, as I've spent most of my career writing new functionality, but also to a large degree getting good at cleaning up and extending code written by others. There are some knacks to doing it well, and I'm in the process of writing write a practical guide to help you learn it faster.

Style guides are topic with the potential for endless discussion. The ruby community has some [good](https://github.com/bbatsov/ruby-style-guide) [ones](https://github.com/styleguide/ruby), and you are of course free to write your very own as well. One thing you will typically find taking over a project that has lived for a while, maybe with multiple maintainers, is inconsistent code styles. This is obviously not the end of the world, but it can sidetrack you when trying to reason about the current behavior of a piece of non-obvious code.

Fortunately it is easy these days to setup [rubocop](https://github.com/bbatsov/rubocop) and ensure a consistent style is used. It is even partially possible to automatically clean up most of it to conform. Consistent hash style, yay.

Some large codebases are a beautiful and valuable mess. They often have a large gap between the current state and an ideal state of syntax consistency and well-managed complexity. All is fortunately not lost, The Rewrite is typically not the answer. Instead, a processes is needed to break things down and attacking the problem in smaller chunks over a period of time, without halting all other progress. It is such a process I'm describing in the guide.

## Update

The guide is ready and now available as a free article on getting started with [Rubocop for Large Projects]({{ site.baseurl }}{% post_url 2017-04-11-rubocop-for-large-projects %}). Enjoy!
