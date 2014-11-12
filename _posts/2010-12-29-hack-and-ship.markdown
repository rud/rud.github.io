---
layout: post
title: Simple git workflow with hack and ship
excerpt: Local feature-branches are quick and easy in git, but when collaborating using a shared central repository, some often repeated steps are part of the local workflow. The hack and ship commands simplify this common scenario.
wordpress_id: 38
wordpress_url: http://object.io/site/?p=38
date: 2010-12-29 15:25:26 +01:00
tags: [git]
categories: [tools, git, workflow]
comments: true

---
I use git as my primary version control tool for all internal development, configuration files, and collaborative development. As branches are virtually free with git, it makes a lot of sense to create short-lived feature-branches for each new thing you start working on. This does mean a bit of shuffling back and forth to integrate changes from others in your local work, but this "pull changes and rebase my work" workflow can be greatly eased by these small scripts.

For more than a year I've been using two small shell-scripts called <code>hack</code> and <code>ship</code> to manage local feature-branches. Hat-tip goes to <a href="http://reinh.com/blog/2008/08/27/hack-and-and-ship.html">ReinH</a> for the original version of these. Notice these shortcuts are <strong>only usable when you work on a feature branch based on master</strong>, not remote branches in general.

<code>hack</code> pulls down the latest changes from the central <code>origin/master</code> branch, and rebases your local feature-branch on this new <code>master</code>. The end result is all the latest changes are integrated, and you will be able to push your commits without adding an unnecessary merge-commit to the shared history.

```sh
#!/bin/sh -x
# Exit if any error is encountered:
set -o errexit
# git name-rev is fail
CURRENT=`git branch | grep '\*' | awk '{print $2}'`
git checkout master
git pull --rebase origin master
git checkout ${CURRENT}
git rebase master
```

<code>ship</code> is a quick way to merge your current branch to <code>master</code>, and push the result to the central repository branch called <code>origin/master</code>.

```sh
#!/bin/sh -x
# Exit if any error is encountered:
set -o errexit
# git name-rev is fail
CURRENT=`git branch | grep '\*' | awk '{print $2}'`
git checkout master
git merge ${CURRENT}
git push origin master
git checkout ${CURRENT}
```

Usually when a feature is completed, I run <code>hack</code>, run all code-tests for the project, the run <code>ship</code>. Taken together, the process is automated and looks like this:
```bash
hack && rake && ship
```
where <code>rake</code> runs all relevant tests, and exits with a non-zero error-code. If one or more tests fail, the changes are not shipped (due to the nature of <code>&amp;&amp;</code> between the commands), and a fix can be committed before sharing the changes with other developers.

What is your process for managing feature branches in git?

**Update:**
See also the <a href="{% post_url 2011-01-18-meet-chop%}">chop</a> script.
