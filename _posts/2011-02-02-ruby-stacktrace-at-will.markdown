---
layout: post
title: Identify what your Ruby process is doing
wordpress_id: 219
wordpress_url: http://object.io/site/?p=219
date: 2011-02-02 08:43:19 +01:00
categories: [ruby, debug, rails, optimize]
---
Quick diagnostics of production issues become a lot easier with a direct way to inspect the running process. Ever wondered exactly what the program is doing right now, without adding a lot of slow logging? With this gem in place, you can get full stacktraces for all threads in the system, while the process keeps running, in production. This means you can do this a few of these dumps, and then look for recurring patterns to identify the currently largest bottlenecks. It's much easier to fix a problem when you know exactly where to look.

### Soon you'll be back on the road to awesome

Simply install and load the <code><a href="https://github.com/ph7/xray">xray</a></code> gem, then add this to your ```Gemfile``` (or similar):

{% highlight ruby %}
gem "xray", require: "xray/thread_dump_signal_handler"
{% endhighlight %}

Next, you need to start the ruby-process, and find the PID (process identifier):

{% highlight shell-session %}
$ ps ax | grep ruby
{% endhighlight %}

You can now invoke:

{% highlight shell-session %}
$ kill -3 1234
{% endhighlight %}

where <code>1234</code> is the PID of the ruby process. <strong>Note this does not terminate the process</strong>. Instead, you immediately see a full stacktrace for <strong>current thread</strong> written to standard out of the process. If you're using the <a href="http://www.rubyenterpriseedition.com/">Ruby Enterprise Edition</a> runtime, you will get a stacktrace for <strong>all threads</strong>, a big win. If you are running the process interactively, you see the output directly in the terminal. For service-processes in production, you will need to check the appropriate logfile.

## Thread dumps for Phusion Passenger with RVM

You can see all running <a href="http://www.modrails.com/">Phusion Passenger</a> instances, including their individual PIDs with this:

{% highlight shell-session %}
$ rvmsudo passenger-status
{% endhighlight %}

Then move in for the <code>kill -3</code>, and have a look at standard out for Passenger. By default on OS X 10.6, this ends up in the file <code>/private/var/log/apache2/error_log</code>, so have a look there. Notice the use of <code>rvmsudo</code> instead of regular <code>sudo</code> - this is because I'm using <a href="http://rvm.beginrescueend.com/">RVM</a> to manage my ruby-versions.

I hope you found this useful -- what is your stacktrace-aided war-story?
