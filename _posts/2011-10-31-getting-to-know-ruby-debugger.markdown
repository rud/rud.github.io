---
layout: post
title: Getting to know Ruby debugger
wordpress_id: 460
wordpress_url: http://object.io/site/?p=460
date: 2011-10-31 14:07:09 +01:00
author: Cameron Dykes
categories: [ruby, debug, workflow]
---
<em>This is a guest-post by <a href="https://github.com/yellow5">Cameron Dykes</a>, about getting started with the ruby debugger.</em>

A key step to debugging any program is replicating the environment to ensure you can consistently produce the bug. In my early <a href="http://ruby-lang.org">Ruby</a> days, to inspect the environment, I used a primitive method: placing <code>puts</code> lines in my code to print values to the console (let's call them "inspection puts").

It may have looked something like this:

```ruby
class Buggy
  # assuming perform_operation and special_options exist...
  def buggy_method(param=nil, options={})
    puts "\n\n\nDEBUG"
    puts "param: #{param}"
    puts "options: #{options}"
    @instance_var = perform_operation(special_options(param, options))
  end
end
```

<h3>The case for ruby-debug</h3>

This method very easily gives me the information I need, but it has some downsides:

1. To inspect the return value of <code>special_options</code> I have to add another inspection puts.
1. Every addition of a new <code>puts</code> requires that I restart the application to inspect the results.
1. To inspect how <code>special_options</code> and <code>perform_operation</code> are handling the data, I have to add inspection puts inside of them.
1. I must remember to remove all of the inspection puts before I push the code.

If only there was a better way to do this.

<a href="https://rubygems.org/gems/ruby-debug19">ruby-debug</a> to the rescue! By putting a breakpoint in our code, we have the ability to inspect environment state, check the return value of any methods, and step through our code one line at a time. This is much more versatile than inspection puts because the full environment is available to us. The "I wonder what the value of this is" problem is gone since we can inspect whatever we want. The step functionality the debugger gives us is useful as well, allowing us to step inside of a called method while maintaining the interactive environment.

<h3>Setting up ruby-debug</h3>

To get set up with the debugger, we'll need to install the gem:

```sh
# Using MRI-1.9.2
gem install ruby-debug19

# Using MRI-1.8.7 or Ruby Enterprise Edition
gem install ruby-debug
```

<h3>ruby-debug in action</h3>

Let's update the example from above using a debugger breakpoint instead of inspection puts:

```ruby
class Buggy
  # assuming perform_operation and special_options exist...
  def buggy_method(param=nil, options={})
    require 'ruby-debug'; debugger
    @instance_var = perform_operation(special_options(param, options))
  end
end
```

The next time the method is called, the debugger will stop and give us an interactive shell. We can inspect the values of each variable with the <code>eval</code> command:

``` ruby
eval param
eval options
```

We can also see what the return value is of the invoked methods:

``` ruby
eval special_options(param, options)
eval perform_operation(special_options(param, options))
```

We can even use the step command to enter inside of <code>special_options</code> and <code>perform_operation</code> to see what they do.

Here are the various debugger commands that I most commonly use:

* <code>list, l</code> - show the code for the current breakpoint
* <code>eval, e</code> - evaluate expression and print the value
* <code>step, s</code> - next line of code, moving within methods
* <code>continue, c</code> - continue in the program until the program ends or reaches another breakpoint
* <code>quit, q</code> - abort the program

Many more commands are available, which can be seen by entering <code>help</code> in the debugger.

<h3>Better debugging ftw!</h3>

With the <a href="https://rubygems.org/gems/ruby-debug19">ruby-debug</a> gem, we have a better tool for diving in to our code than inspection puts. Using a debugger breakpoint, we can interactively step through our code with less cleanup.

Happy debugging!

<h3>Debugging references</h3>


* <a href="http://guides.rubyonrails.org/debugging_rails_applications.html#debugging-with-ruby-debug">Debugging with ruby-debug in the Debugging Rails Applications guide</a>
* <a href="https://rubygems.org/gems/ruby-debug19">ruby-debug19 gem</a>
* <a href="https://gist.github.com/1290190">Example code from this post</a>

<h3>Who am I?</h3>

<ul>
<li><a href="http://twitter.com/yellow5">@yellow5</a></li>
<li><a href="http://github.com/yellow5">Github</a></li>
</ul>
