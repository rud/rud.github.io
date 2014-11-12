---
layout: post
title: Easy local code-review with git
excerpt: When multiple developers contribute to a project, keeping on top of the constant flow changes can be a challenge. The following is a simple local workflow to stay on top of a tracked git repository where multiple developers share code.
wordpress_id: 54
wordpress_url: http://object.io/site/?p=54
date: 2011-01-07 10:45:51 +01:00
categories: [tools, git, workflow, code review]
---
When multiple developers contribute to a project, keeping on top of the constant flow changes can be a challenge. The following simple review workflow assumes a shared git-repository with a fairly <a href="{% post_url 2010-12-29-hack-and-ship %}">linear commit-history</a>, that is, not having too many merge-commits.

So, assuming a fairly linear history of commits from multiple developers, how do you easily keep track of what you have already read through and reviewed? Easy, use a local branch as a bookmark. This tiny script makes it trivial to add or update such a branch:

```sh
#!/bin/sh
NEW_BASE=${1?"Usage: $0 <treeish>"}

git branch --force reviewed $NEW_BASE || exit 1

echo Marked as reviewed: `git rev-parse --short reviewed`
```


Save this as a new file called <code>reviewed.sh</code> in your <code>PATH</code>.

Usage is extremely simple:

```sh
reviewed.sh 369b5cc
reviewed.sh master
reviewed.sh v1.0.7
reviewed.sh HEAD~5
```

Running one of these commands will mark the given <a href="http://book.git-scm.com/4_git_treeishes.html">treeish</a> as reviewed, and when you look at your commit-history in a visual tool such as ```git gui``` or <code><a href="http://gitx.frim.nl/">gitx</a></code>, the <code>reviewed</code> branch visually indicates how far you have gotten. Note how both commit-IDs, branch names, tags, and relative commit-IDs can be used as argument.

You can also utilize this review bookmark from the commandline. The following shows you all commits added to master since your last review:

```sh
git log reviewed..master --reverse
```

You can add a <code>--patch</code> to that command to see the full diff for each change. Adding <code>--format=oneline</code> just shows you the commit-IDs and first line of the commit-message.

Once you've read all the latest commits on master, simply do a
```sh
reviewed.sh master
```
and you're done.

### Why not use a tag?

I find it convenient to be able to do a push of all tags to the central repository with

```sh
git push --tags
```

and this would share such a private review-tag. As this is my private reminder of how far in the commit-history I have reviewed, sharing it is just confusing to other developers.

<strong>Notice:</strong> Any commits which are added only to the <code>reviewed</code> branch are unreferenced when you mark a new treeish as reviewed. Just something to keep in mind.

How do you keep track of the flow of changes?

