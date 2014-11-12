---
layout: post
title: Quick data import and linking in Rails
wordpress_id: 116
wordpress_url: http://object.io/site/?p=116
date: 2011-01-17 11:35:34 +01:00
categories: [ruby, rails]
---
Some web-applications have to ingest an enormous amount of new data on a regular basis. Import scripts easily become an ever-growing procedural mess, annoying to maintain. In this post I show a bit of code which can be used to simplify and unify such import scripts.

Assume you have a pipeline of post-import steps to run. This can be organized in numerous ways. Simplest is to just have a bunch of methods called one after the other once you have the data loaded:

{% highlight ruby %}
link_frobnitz
spin_really_fast_around_z_axis
reticulate_splines
deploy_hamsters
{% endhighlight %}

Now, assume once in a while one of the steps fail for an unexpected reason. You know, it's rare data from external sources is as clean as we'd like. So you need to fix a few things and retry the import. However, as datasizes grow and with that the running time of the import, it can be a huge waste redoing all the work because of a misplaced comma made the final <code>deploy_hamsters</code> step fail.

Exceptions are the obvious way to report fatal data-errors, and implicit or explicit transactions to ensure consistency of the import. But how can this easily be combined for a resume-friendly import mechanism?

Enter the bulk importer step runner with trivial progress reporting:
{% highlight ruby %}
def import_updaters
  all_steps.each do |step_name|
    run_import_step(step_name)
  end
end

private

def run_import_step step_name
  puts "Running #{step_name}"

  ImportModel.transaction do
    self.send(step_name)
  end

rescue => e
  STDERR.print "\nImport error:  #{e.inspect}\n#{e.backtrace.join("\n")}"
  STDERR.print "Please resume at step #{step_name}"
  exit 1
end

protected

def all_steps
  [
    :link_frobnitz,
    :spin_really_fast_around_z_axis,
    :reticulate_splines,
    :deploy_hamsters
  ]
end
{% endhighlight %}

Notice you obviously have to change the model-name (<code>ImportModel</code> above) and provide the actual implementation for these individual steps. <code>all_steps</code> returns the list of methods to run, <code>run_import_step</code> runs a single step with error-handling, and <code>import_updaters</code> runs all the relevant updaters.
<h3>Easy performance statistics</h3>
As a bit of bonus-functionality, the following can be used for reporting import progress with timing-statistics after each step completes:

{% highlight ruby %}
def report_progress message, &block
  STDERR.print message

  if block_given?
    time = Benchmark.measure { yield }
    formatted_time = "%.2fs" % time.real

    STDERR.puts " - #{formatted_time}"
  else
    STDERR.puts
  end
end
{% endhighlight %}

Usage is simple - just call <code>report_progress</code> with a comment to print and a block of code, like this:

{% highlight ruby %}
def run_import_step step_name
  report_progress "Running #{step_name}" do
    ImportModel.transaction do
      self.send(step_name)
    end
  end
  //...
{% endhighlight %}

What do you use to make data-imports easier to manage?
