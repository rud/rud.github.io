--- 
layout: post
title: Modernize Your Git Workflow
wordpress_id: 382
wordpress_url: http://object.io/site/?p=382
date: 2011-05-09 11:00:39 +02:00
categories: [tools, git]
---
Years ago I read [Streamed Lines: Branching Patterns for Parallel Software Development](http://www.cmcrossroads.com/bradapp/acme/branching/), and many of the thoughts have stayed me. Think of it as a catalogue of [Design Patterns](http://en.wikipedia.org/wiki/Design_pattern_\(computer_science\)) for version control workflows. There are so many ways to efficiently use branches to both collaborate and isolate work. Lively discussions can be had on how to best organize a repository, and having a great overview of the options makes for more informed discussions.

I have previously described how I've been using small <a href="{% post_url 2010-12-29-hack-and-ship %}">hack &amp;&amp; ship</a> commands to simplify and streamline my personal git workflow. Recently I've switched to using the [Git Flow](http://nvie.com/posts/a-successful-git-branching-model/) setup as my branching and release workflow.

### Setting up git-flow

Installing the [git-flow](https://github.com/nvie/gitflow) toolset on OS X is trivial using the <a href="http://mxcl.github.com/homebrew/">homebrew</a> installer:

```
$ brew install git-flow
```

The [git-flow project](https://github.com/nvie/gitflow) also has instructions for installation on Linux and Windows.

Installing git-flow adds a few helpful commands to the git environment for creating and managing branches for features and releases. A fresh git repository is born with a <code>master</code> branch. In the default git-flow setup, this is where the current <strong>production</strong> release lives. Furthermore, a branch called <code>develop</code> is created, this is where development takes place. Note that git-flow is just a series of shortcuts to having a development branch and a production branch with sensible ways of shuttling changes back and forth. After installing the git-flow package, configure your local repository for git-flow use with:

```
$ git flow init
```

You can most likely accept the defaults by pressing enter at each question - this also makes it easier for others to initialize git-flow in their working copy, as all are using the same defaults. The only change from running the <code>init</code> command is two <code>[gitflow]</code> sections in your <code>.git/config</code> file looking like this:

```
[gitflow "branch"]
	master = master
	develop = develop
[gitflow "prefix"]
	feature = feature/
	release = release/
	hotfix = hotfix/
	support = support/
	versiontag = 
```

It's no more magic than that. Consult the built-in help with: 

```
$ git flow
```

If you've got the <code>bash-completion</code> package installed, you can use tab-completion for git-flow commands and arguments. Highly recommended.

### Working on a feature

Start work on a feature - say feature 77 from your issue-tracker:

```
$ git flow feature start 77-speedup-yak-shaving
```

This creates a new regular local git branch called <code>feature/77-speedup-yak-shaving</code> based on the current <code>develop</code> and places you on it.  Want to share your work in progress with others? Use:

```
$ git flow feature publish 77-speedup-yak-shaving
```

This pushes the branch, and sets up your local branch to track the remote branch in one easy step. The regular <code>git push</code> and <code>git pull --rebase</code> work as you would expect here. <span class="highlighted">There is nothing special about branches created by git-flow, they are just conveniently named and managed.</span>

All done with a feature? Rebase your feature on the current <code>develop</code> branch, then merge it in:

```
$ git flow feature rebase
$ git flow feature finish 77-speedup-yak-shaving
```

With these two steps, you end up on <code>develop</code> with your feature merged in. Run your test-suite, and push to a central repository as appropriate.

### Releasing to production version and hotfixes

Production releases are handled quite nicely in git-flow:

```
$ git flow release start 2011_year_of_the_yak
```

This creates a new branch called <code>release/2011_year_of_the_yak</code>, based on the current <code>develop</code> branch. Here you can fix any HISTORY or VERSION file, commit, and the finish up the release with:

```
$ git flow release finish 2011_year_of_the_yak</pre>
```

By finishing the release, you now get a tag called <code>2011_year_of_the_yak</code>, and the temporary release-branch is removed. You end up back at the <code>master</code> branch.

A hotfix is simply a feature branch which is based on the latest production release, and which is automatically merged to both <code>develop</code> and <code>master</code>. Extremely handy. 

### Rebase considered harmful?

Some developers prefer to only ever merge, but I'm a big fan of [rebasing local work before publishing](http://darwinweb.net/articles/the-case-for-git-rebase). I'll leave you with this:

> If you treat rebasing as a way to clean up and work with commits before you push them, and if you only rebase commits 
> that have never been available publicly, then you’ll be fine. If you rebase commits that have already been pushed 
> publicly, and people may have based work on those commits, then you may be in for some frustrating trouble.
<p style="text-align: right;">From <a href="http://progit.org/book/ch3-6.html">Pro Git</a>, chapter 3</p>
