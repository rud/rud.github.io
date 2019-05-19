---
layout: post
title: "Spelunking an ActionCable Error"
date: 2018-06-11 14:15:42
comments: false
categories: [ruby, rails, actioncable, pry]
---

Upgraded an app to Rails 5.0.7, and after following the official migrations steps, the app completely failed to load.

The exception itself was rather puzzling: `undefined method 'logger' for nil:NilClass (NoMethodError)`, raised deep inside of ActionCable at [`lib/action_cable/engine.rb:20`](https://github.com/rails/rails/blob/v5.0.7/actioncable/lib/action_cable/engine.rb#L19-L21):

{% highlight ruby %}
initializer "action_cable.logger" do
  ActiveSupport.on_load(:action_cable) { self.logger ||= ::Rails.logger }
end
{% endhighlight %}


So, how can this seemingly simple code go wrong?
Let's read backward from what is calling the above `on_load` code-block.

According to the [documentation](http://api.rubyonrails.org/v5.0.7/classes/ActiveSupport/LazyLoadHooks.html) for `ActiveSupport::LazyLoadHooks`, the code added to `on_load(:action_cable)` is triggered when `run_load_hooks(:action_cable, context)` is called at a later time, with a context object passed in.
In this case, the `context` passed to `on_load` was unexpected a `nil` value.

Let's look closer to what is actually passed to this [`run_load_hooks`](https://github.com/rails/rails/blob/v5.0.7/actioncable/lib/action_cable/server/base.rb#L85) call:

{% highlight ruby %}
ActiveSupport.run_load_hooks(:action_cable, Base.config)
{% endhighlight %}

Let's check the [`ActionCable::Server::Base.config`](https://github.com/rails/rails/blob/v5.0.7/actioncable/lib/action_cable/server/base.rb#L13) method definition:

{% highlight ruby %}
cattr_accessor(:config, instance_accessor: true) {
  ActionCable::Server::Configuration.new
}
{% endhighlight %}

The intent here is the `Base.config` method lazy-initializes an `ActionCable::Server::Configuration` instance and re-uses that for subsequent calls.

Somehow that initialization fails to run in our case, and we get `nil` value returned instead.

## Inspecting the weirdness

So, here's how I finally figured out what was going wrong, with the above code reading context in mind:
I used `bundle open actioncable` and edited the end of the `lib/action_cable/server/base.rb` file locally like so:

{% highlight ruby %}
  require 'pry'; binding.pry  # <-- ADDED THIS LINE
  ActiveSupport.run_load_hooks(:action_cable, Base.config)
end
{% endhighlight %}
[Source](https://github.com/rails/rails/blob/v5.0.7/actioncable/lib/action_cable/server/base.rb#L85)

I added `gem 'pry-rails'` to the project `Gemfile`, and started a Rails console.

Time to show where the `Base.config` method is actually defined:

``` shell-interaction
[1] pry(ActionCable::Server)> show-method Base.config

From: ~/.rbenv/versions/2.5.1/lib/ruby/gems/2.5.0/gems/table_print-1.1.4/lib/table_print/cattr.rb @ line 9:
Owner: #<Class:ActionCable::Server::Base>
Visibility: public
Number of lines: 1
````

Surprise! Hello `table_print-1.1.4`, didn't expect to see you here.
That most likely explains why the lazy initialization was skipped, an old monkey-patch gumming up the works.

Removing `table_print` from the project `Gemfile`, and re-running the console, we get:

``` shell-interaction
[1] pry(ActionCable::Server)> show-method Base.config

From: ~/.rbenv/versions/2.5.1/lib/ruby/gems/2.5.0/gems/activesupport-5.0.7/lib/active_support/core_ext/module/attribute_accessors.rb @ line 60:
Owner: #<Class:ActionCable::Server::Base>
Visibility: public
Number of lines: 3

def self.#{sym}
  @@#{sym}
end
```

With the removal of the old gem, the Rails app succeeds in booting the Rails console.
There was much rejoicing.

As we're done debugging here, let's clean up the edited `actioncable` gem:

``` shell-interaction
$ gem pristine actioncable
Restoring gems to pristine condition...
Restored actioncable-5.0.7
```

## A conclusion of sorts

Reading, understanding and messing with the Rails source are sometimes the only way forward when things behave unexpectedly.
`binding.pry` is a good friend and `show-method` can really save the day, especially when the observed behavior doesn't match the source-code.

Got any weird puzzling bugs holding your project back?
I'd be happy to take a look.
Contact details below.
