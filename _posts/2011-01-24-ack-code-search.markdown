--- 
layout: post
title: Ack! Effortlessly search your codebase
wordpress_id: 193
wordpress_url: http://object.io/site/?p=193
date: 2011-01-24 22:13:47 +01:00
categories: [tools, ack, search]
---
My favorite commandline tool for efficiently searching a codebase is <a href="http://betterthangrep.com/">Ack</a>. It shows file:line and highlights the matches in the output. When refactoring a solution it is a trusty companion to quickly find all uses of a method, for instance.

{% img /images/posts/2011-01-24-ack-code-search/ack-quick-demo.png %}

It is easy to install with DarwinPorts:

```
sudo port install p5-app-ack
```

This puts the <code>ack</code> command in your <code>PATH</code> and you're ready to go.

### Basic usage of Ack

A recurring task is to find all uses of a method or class in a project. As this is simply a substring search, you invoke Ack with:
```
ack inconvenience_late_payers
```
and ack quickly starts outputting all the places where this is used. If you add a <code>-i</code>, the search becomes case-insensitive. Use <code>ack --help</code> to see all the many available options.

### A bit of configuration goes a long way

Ack is fast because it skips uninteresting files. It is meant for searching for text, and skips version control folders, logfiles, images, and a lot of other <strong>stuff</strong> you are not interested in when searching through your source-code. However, the backside is you need to configure things once to have support for all your various kinds of source-code.

Adding this small bit of config means <code>.haml</code>, <code>.sass</code>, and Cucumber <code>.feature</code> files are also included when searching.

``` console ~/.arkrc:
--type-set=haml=.haml
--type-set=sass=.sass
--type-set=cucumber=.feature
```

With this in place, you are ready to rock HAML/SSS + Cucumber projects.

Know any extra must-have options or shortcuts I have left out?

<strong>Update:</strong> added link to official <a href="http://betterthangrep.com/">Ack homepage</a>.
