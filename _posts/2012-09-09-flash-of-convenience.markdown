---
layout: post
title: "Flash of Convenience"
date: 2012-09-09 14:05
comments: false
categories: [rails, i18n, workflow]
---

Flash messages in Rails applications need a bit of love. A lot of them tend to be identical within a given app, and it's a bit annoying to manage defaults and fallback for all these calls to `I18n.t()`. The following is a small helper-method and a simple structure to keep it all neatly organized.


For this to work, add the text you want shown to your `config/locale/en.yml` (by default):

```yaml
en:
  posts:
    create:
      flash:
        created: "Your post will now be reviewed before publishing"

  flash:
    created: "Successfully created"
    updated: "Successfully updated"
```

Here the `posts#create` action has a custom flash for the `:created` flash-message. All other controllers and actions in the application where a flash-message of `:created` is used the default message of `"Successfully created"` will be shown.

Next, add this to your `ApplicationController` (this is all of the magic):
```ruby
  protected
  def flash_message cause, args = {}
    primary_key = "#{controller_name}.#{action_name}.flash.#{cause}"
    default_key = "flash.#{cause}".to_sym
    I18n.t primary_key, args.merge(:default => default_key)
  end
```

Finally, this is how you use it within an action:
```ruby
  flash[:notice] = flash_message(:created)
  # and
  redirect_to :root, :notice => flash_message(:created)
```

Try to keep the actions somewhat generic like `:created`, `:deleted`, `:updated`, etc. This makes the default values easier to manage.


What just happened?
-------------------

The `I18n` system has some neat convenience shortcuts we're using here. First of all we build the primary flash lookup-key based on the current controller and action.

Second bit of functionality used here is the fallback message. Fallback messages can be given as an explicit string, or a symbol. When a symbol is used it will be used to lookup a new message in the `I18n` backend. We use that for the `:default` argument where the fallback is set to be a global flash message like `flash.created`.

Now you can customize all the flashes in your app without touching the code, you have all these small bits of user interface in a single file for easy overview. You can also easily vary the messages between languages, customize text where it is necessary, and have a convenient global fallback message.