---
layout: post
title: Idiomatic shared state in RSpec
wordpress_id: 251
wordpress_url: http://object.io/site/?p=251
date: 2011-02-17 20:01:47 +01:00
categories: [rspec, ruby, bdd, api, rails]
---
RSpec is an extremely convenient tool for structuring and writing specs, and I use it on all my ruby-projects.

One set of built-in helpers I'd like to cheer on a bit are <code>let</code> and <code>let!</code>, as they can greatly simplify setting up state for multiple examples. They exist to provide a simple and convenient way to create and share state between examples. When not using these methods, specs with a bit of shared state typically look like this:

{% highlight ruby %}
context "with a bit of state" do
  before :each do
    @user = User.create!
  end

  # The following specs assume the user has already been created
  it "should be the latest user" do
    User.last.should == @user
  end

  # .. more examples reusing the @user
end
{% endhighlight %}

This is easy to get started with, and when there are only a few variables in the <code>before</code> block, things are fine. But if you have a typo in an example where you intended to use the <code>@user</code> instance variable in an example, but you accidentally typed <code>@usr</code>, you just get a <code>nil</code> value, instead of a helpful error message regarding an unknown indentifier. This minor problem can be avoided by using <code>let</code> instead, as you get an unknown indentifier error instead of a <code>nil</code> value. Another issue is that all variables in the <code>before</code> block are evaluated, whether or not they are actually used in the examples.

The excellent <a href="http://relishapp.com/rspec/rspec-core/v/2-5/dir/helper-methods/let-and-let">official RSpec 2</a> documentation has the following to say on <code>let</code>, before showing clear examples of use:

> ### <code>let</code> and <code>let!</code>
>
> Use <code>let</code> to define a memoized helper method. The value will be cached across multiple calls in the same example but not across examples.
>
> Note that <code>let</code> is lazy-evaluated: it is not evaluated until the first time the method it defines is invoked. You can use <code>let!</code> to force the method's invocation before each example.


The fact that results are both memoized and lazy-evaluated means no extra effort is expended if the value is not used in a given example. Performance win!

### Use <code>let</code> or <code>let!</code>?

Please use the correct helper method - I've seen examples like this:

{% highlight ruby %}
context "with a bit of state" do
  let(:user) { User.create! }

  before :each do
    user
  end

  # The following specs assume the user has already been created
  it "should be the latest user" do
    User.last.should == user
  end

  # .. more examples reusing the user
end
{% endhighlight %}

The exact same thing can be expressed directly and become more readable, just by adding a <code>!</code>:

{% highlight ruby %}
context "with a bit of state" do
  let!(:user) { User.create! }

  it "should be the latest user" do
    User.last.should == user
  end

  # .. more examples reusing the user
end
{% endhighlight %}

The <code>let</code> and <code>let!</code> helpers are available in from versions <code>rspec 1.3.1</code>, and <code>rspec-core 2.0.0</code> onwards. In a nutshell, this means they are available for use in Rails 2 and Rails 3 projects. Take them for a spin if you haven't already, and tell me how it goes.

I recommend having a quick look at the [Rspec 2.5](http://relishapp.com/rspec/rspec-core/v/2-5) documentation now, as you'll probably discover one or more neat new solutions to recurring issues when writing specs.
