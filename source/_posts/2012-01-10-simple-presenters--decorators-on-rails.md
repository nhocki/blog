---
layout: post
title: Simple Presenters & Decorators on Rails
tags:
- rails
- presenters
- objects
- oop
---
There's been a [lot](http://blog.steveklabnik.com/posts/2011-09-06-the-secret-to-rails-oo-design) of [talk](http://blog.steveklabnik.com/posts/2011-09-09-better-ruby-presenters) [about](http://avdi.org/devblog/2011/11/15/early-access-beta-of-objects-on-rails-now-available-2/)[decorators](http://robots.thoughtbot.com/post/14825364877/evaluating-alternative-decorator-implementations-in) and [presenters](http://robots.thoughtbot.com/post/13641910701/tidy-views-and-beyond-with-decorators) ([which are almost the same](https://groups.google.com/forum/#!msg/objects-on-rails/htAopf3k_dM/qJMq6QAfMvsJ)) in Rails and how they help you with a "real" OO code. There are great gems and there's even a [Railscast](http://railscasts.com/episodes/287-presenters-from-scratch) (pro) on how to build a presenter from scratch. But, in the new project I'm working on, we (<- [he](https://github.com/febuiles)) has this **really** simple setup which I feel kind of cleaner.

What we wanted to have was a `present` method in the controller and build the view from its methods. So, we created an [ActiveSupport::Concern](http://api.rubyonrails.org/classes/ActiveSupport/Concern.html) called `Presentable` and included it in our application controller. Here's our (once again, his) code for that module.

{% gist 1592754 presentable.rb %}

Now, we just need to create each presenter. But, since there's a lot of shared logic between presenters we (he) just created a parent `Presenter class` with all that. Notice that our presenter takes an object as parameter, so we just delegated all the method calls to our record (thank you `method missing`) if it responded to that method. You could also send the view context to access Rails helpers inside your presenter. Here's our `Presenter base class`.

{% gist 1592754 presenter.rb %}

Then, we just need to inherit from the `Presenter` class and add our custom, clean methods. Using `delegate and method missing` will help us keep the class clean. I won't show you a presenter here, they'll look a lot like Ryan's one in his Railscasts so check that out first.

I will show you our controller though. Using the decent_exposure gem we managed to get super tiny controllers. Here's 1 of them:

{% gist 1592754 requests_controller.rb %}

Combining `presenters` and `form builders` will make your views even cleaner, giving a lot of pleasure to the future you! So please, just pretty-please, start using presenters and builders and leave helpers as simple as possible. Or, leave them for good.

You'll be amazed how different you'll feel when reading some old code or some co-worker's code.
