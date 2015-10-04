---
layout: post
title: "Manitenance programming - freestyle"
date: 2015-10-02 20:20
comments: false
---

How do you get started doing effective maintenance programming? This topic is near and dear to me, as I've spent most of my career both writing new code, but also to a large degree getting good at cleaning up and extending code written by others. There are some knacks to doing it well, and I'm in the process of writing write a practical guide to help you learn it faster.

A topic with the potential for endless discussion is style guides. The ruby community has some [good](https://github.com/bbatsov/ruby-style-guide) [ones](https://github.com/styleguide/ruby), and you are of course free to write your very own as well. One thing you will typically find taking over a project that has lived for a while, maybe with multiple maintainers, is inconsistent code styles. This is obviously not the end of the world, but it can sidetrack you when trying to reason about the current behavior of a piece of code.

Fortunately it is easy these days to setup [rubocop](https://github.com/bbatsov/rubocop) and ensure a consistent style is used. It is even partially possible to automatically clean up most of it to conform. Consistent hash style, yay.

Some codebases are huge, with a large gap between the current state and a codebase that conforms to an ideal state of syntax consistency and manageable complexity. A processes is needed to break things down and attacking the problem in smaller chunks over a period of time, without halting all other progress. It is such a process I'm describing in the guide.

I'll be happy to notify you when the guide [Effective Maintenance: taming messy code (Ruby edition)](https://gumroad.com/rud/) is ready - simply enter your email below, and I'll keepy you informed.
