---
layout: post
title: The sneakernet in git
wordpress_id: 334
wordpress_url: http://object.io/site/?p=334
date: 2011-02-27 19:51:21 +01:00
categories: [tools, git, workflow]
redirect_from:
  - /site/2011/02/sneakernet-git/
---
There are many ways to backup a git repository and shuttle sets of changes around. If you just want an offline copy for safe-keeping, this method is very simple and convenient:

{% highlight shell-session %}
$ git bundle create blog-dump.bundle master
{% endhighlight %}

This creates a file <code>blog-dump.bundle</code> with the full history for <code>master</code>, including all parent commits. You now have a single file, which can effectively be mailed, downloaded via SCP or FTP, or simply moved to [Dropbox](http://db.tt/cHU9N1d) (affiliate link) for safe keeping, or otherwise shared with others.

Single files are easy to archive and backup.

### Thawing a bundle

When you want to restore from such a bundle backup, you simply clone from it. The steps include creating a new repository, and then pulling from the bundle, like you would a remote repository:

{% highlight shell-session %}
$ git init
$ git pull blog-dump.bundle master
{% endhighlight %}

Your repository now holds all commits that were stored in the bundle. For more usage-scenarios see the
[official git bundle help](http://www.kernel.org/pub/software/scm/git/docs/git-bundle.html) or just type:

{% highlight shell-session %}
$ git bundle --help
{% endhighlight %}

