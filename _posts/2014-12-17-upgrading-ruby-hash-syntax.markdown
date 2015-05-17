---
layout: post
title: "Upgrading ruby hash syntax"
date: 2014-12-17 18:56
comments: false
categories: [rails]
---

Consistency makes code easier to read. Long-lived projects can be a wildly inconsistent in their ruby hash syntax, so here is a quick way of upgrading all symbol-based hash-keys to the new leaner format. Simply put: perform the below operation to convert `:thing => 'value'` to `thing: 'value'` in one easy step.

Please remember to *commit everything first* so you have a clean slate to start out from, and undo is easy.

In [Sublime Text](http://www.sublimetext.com), open `Find | Find in Files...` and enter:

{% highlight ruby %}
Search:   ([^:]):([^:\s=]+)\b\s*=>\s*
Replace:  \1\2:
{% endhighlight %}

Ensure the Regular Expression mode is active (the button looks like: `.*`). Now is a good time to use the `File | Save All` action.

Remember to double-check everything looks just right. I recommend using `git add --patch .` for this, because that way you can easily skip any changes you might not agree with.

This is just a minor detail in the overall readability and consistency of a project, but even minor details matter when it comes to long-term ease of maintainance.

**Update 2015:** [Rubocop](https://github.com/bbatsov/rubocop) can easily fix this minor issue and also automatically fix quite a few other inconsistencies. Highly recommended.
