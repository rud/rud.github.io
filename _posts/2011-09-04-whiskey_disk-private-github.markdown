---
layout: post
title: Deploy a private Github repository with whiskey_disk
wordpress_id: 420
wordpress_url: http://object.io/site/?p=420
date: 2011-09-04 01:00:04 +02:00
categories: [deploy, ruby, git, workflow]
---

[`whiskey_disk`](https://github.com/flogic/whiskey_disk) makes it very easy to quickly deploy a new version of your site to one or more servers. It's very efficient, as files are copied directly from your git repository to the server. This means you can very quickly make large deployments even when you are on a slow internet connection. You do need a secure way for your server to access the git repository while the deploy is going on, read on to see how this is easily done.

For this example, I'm going to deploy a small static site from a private <a href="https://github.com">Github</a> repository to a server via ssh using ```whiskey_disk```.  There are a few steps in getting this all setup, but once you have the ssh infrastructure playing nicely, a complete deployment of a small static site takes on the order of 2-5 seconds. Stay with me, and I'll show how all the pieces fit together.

To begin with, I'll assume you have a site or application already pushed to a private Github reposity that you now want to do a deploy of. You also have ssh access to the server you are deploying to for this to work. For this example, I'm deploying the <code>master</code> branch of the following private git repository:

`git@github.com:rud/whiskey_disk-demo.git`

## Adding the configuration file

Make sure you have your appropriate code pushed to Github, then we'll add the whiskey_disk config file <code>config/deploy.yml</code> to the project:

{% highlight yaml %}
demo:
  domain:     "deploy_user@example.com"
  deploy_to:  "/home/deploy_user/domains/useful_site.com"
  repository: "git@github.com:rud/whiskey_disk-demo.git"
{% endhighlight %}

This is a simple YAML file (indentation matters), you will need to adjust the domain and paths as appropriate. I use the name <code>demo</code> for the environment, other common examples would be <code>staging</code>, <code>production</code>, etc. A lot more options are available, see the official <a href="https://github.com/flogic/whiskey_disk#readme">whiskey_disk readme</a>, but this all we need to get started.

## Running whiskey_disk setup

whiskey_disk has two commands: a <code>setup</code> command, for doing the initial repository clone, and the <code>deploy</code> command used for all subsequent deploys.

With the <code>config/deploy.yml</code> file in place inside the project, we're going to just try and run the <code>setup</code> command from the local computer, seeing what happens. For now, this errors out:

{% highlight sh %}
$ wd setup -t demo
Initialized empty Git repository in /home/deploy_user/domains/useful_site.com/.git/
Host key verification failed.
fatal: The remote end hung up unexpectedly
fetch-pack from 'git@github.com:rud/whiskey_disk-demo.git' failed.
error: pathspec 'origin/master' did not match any file(s) known to git.
Did you forget to 'git add'?
{% endhighlight %}

Okay, we're not quite ready to do deployments just yet.

### Resolving all ssh issues

Github hangs up on us - what's happening? Testing from my local machine as described in <a href="http://help.github.com/ssh-issues/">the github ssh guide</a>:

{% highlight sh %}
$ ssh -T git@github.com
Hi rud! You've successfully authenticated, but GitHub does not provide shell access.
{% endhighlight %}

That's the kind of success we're looking for on the server as well. Ssh to the server as the deploy user, and try the same command there:

{% highlight sh %}
$ ssh -T git@github.com
The authenticity of host 'github.com (207.97.227.239)' can't be established.
RSA key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'github.com,207.97.227.239' (RSA) to the list of known hosts.
Permission denied (publickey).
{% endhighlight %}

Try the command again, now that the host key is trusted:

{% highlight sh %}
$ ssh -T git@github.com
Permission denied (publickey).
{% endhighlight %}

Notice how we get a new error later in the connection process, a <code>Permission denied (publickey)</code> instead of <code>Host key verification failed</code>.

For the sake of completeness, this is what this situation looks like from whiskey_disk:

{% highlight sh %}
$ wd setup -t demo
Initialized empty Git repository in /home/deploy_user/domains/useful_site.com/.git/
Permission denied (publickey).
fatal: The remote end hung up unexpectedly
fetch-pack from 'git@github.com:rud/whiskey_disk-demo.git' failed.
error: pathspec 'origin/master' did not match any file(s) known to git.
{% endhighlight %}

### ssh-agent to the rescue

The root cause for this difference is Github knows my public key and has access to the corresponding private key, but this key is not available to my server, so it cannot authenticate against Github. The solution is fortunately simple. Do this on your local machine:

{% highlight sh %}
$ ssh-add
{% endhighlight %}

This prompts you for the password for your private key, then keeps the private key unlocked in memory for automated access. <strong>You need to do this after each time you restart</strong> before you can do new deployments, as the unlocked key is only stored in memory. We also need to enable ssh-agent support for the connection to the server - as ssh-agent gives a server the possibility to use our private key without further prompting, you need to trust the administrator of servers you deploy to. With that caveat out of the way, open your <code>~/.ssh/config</code> file, and add a config block like this:

{% highlight sh %}
Host example.com
    ForwardAgent yes  # This enables ssh-agent functionality, making whiskey_disk happy
{% endhighlight %}

Adjust the hostname to taste, obviously. You can also add port-number here, if your admin moved ssh access from the default port. While you have that file open, also check out these extremely handy [ssh productivity tips](http://blogs.perl.org/users/smylers/2011/08/ssh-productivity-tips.html) for a lot of extra ssh goodness.

Close the previous connection to your server, open a new one (so ssh-agent can do its magic), then re-run the test:

{% highlight sh %}
$ ssh -T git@github.com
Hi rud! You've successfully authenticated, but GitHub does not provide shell access.
{% endhighlight %}

We have ssh-win!

## Deploying

Time to re-try the deployment <code>setup</code> from the local machine:

{% highlight sh %}
$ wd setup -t demo
Results:
deploy_user@example.com => succeeded.
Total: 1 deployment, 1 success, 0 failures.
{% endhighlight %}

This means the repository was successfully cloned into place. We only have to do this once, from now on deployments are just a matter of:

{% highlight sh %}
$ wd deploy -t demo
Results:
deploy_user@example.com => succeeded.
Total: 1 deployment, 1 success, 0 failures.
{% endhighlight %}

I'm seeing a total time taken for a deployment in the area of 2-5 seconds. Remember to push to Github before running your deployment, as it is from Github your server is pulling versions.

This method for deployment is extremely awesome, and the amount of awesome grows with the size of your deployments, as server-server connections are most likely a lot faster than your own internet connection.

## Next steps

There is a lot more to whiskey_disk that I've shown here. In particular I'd like to highlight the "Configuration Repository" section of the [official whiskey_disk readme](https://github.com/flogic/whiskey_disk#readme). Editing per-application configuration files on individual servers is effectively a thing of the past.
