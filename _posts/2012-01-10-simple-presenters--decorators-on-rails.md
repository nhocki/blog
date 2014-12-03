---
layout: post
title: Simple Presenters & Decorators on Rails
date: 2012-01-10
excerpt: How a simple approach on presenters & decorators went a long way on a project.
categories:
- rails
- presenters
- objects
- oop
---
There's been a [lot](http://blog.steveklabnik.com/posts/2011-09-06-the-secret-to-rails-oo-design) of [talk](http://blog.steveklabnik.com/posts/2011-09-09-better-ruby-presenters) [about](http://avdi.org/devblog/2011/11/15/early-access-beta-of-objects-on-rails-now-available-2/) [decorators](http://robots.thoughtbot.com/post/14825364877/evaluating-alternative-decorator-implementations-in) and [presenters](http://robots.thoughtbot.com/post/13641910701/tidy-views-and-beyond-with-decorators) ([which are almost the same](https://groups.google.com/forum/#!msg/objects-on-rails/htAopf3k_dM/qJMq6QAfMvsJ)) in Rails and how they help you with a "real" OO code. There are great gems and there's even a [Railscast](http://railscasts.com/episodes/287-presenters-from-scratch) (pro) on how to build a presenter from scratch. But, in the new project I'm working on, we ([he](https://github.com/febuiles)) have this **very** simple setup which I feel kind of cleaner.

What we wanted to have was a `present` method in the controller and build the view from its methods. So, we created an [ActiveSupport::Concern](http://api.rubyonrails.org/classes/ActiveSupport/Concern.html) called `Presentable` and included it in our application controller. Here's our (once again, his) code for that module.

```ruby
# https://gist.github.com/nhocki/1592754
# app/concerns/controllers/presentable.rb

# Presentable Concern
# Include this module in the controllers where
# you need the `present` method.
module Presentable
  # This method returns a Presenter Object to work
  # with in the view.
  #
  # Attributes:
  #   object:   The object to present
  #
  # Supported Options
  #    as:   The presenter class. If this is not suplied
  #          the method will try to guess the class
  #          e.g: UserPresenter for a User object.
  def present(object, options = { })
    if object.respond_to?(:first)
      return [] if object.empty?
      presenter = "#{object.first.class.to_s.pluralize}Presenter"
    else
      presenter = "#{object.class}Presenter"
    end

    presenter = options.fetch(:as) { eval(presenter) }
    presenter.new(object)
  end
end
```

Now, we just need to create each presenter. But, since there's a lot of shared logic between presenters we (he) just created a parent `Presenter class` with all that. Notice that our presenter takes an object as parameter, so we just delegated all the method calls to our record (thank you `method missing`) if it responded to that method. You could also send the view context to access Rails helpers inside your presenter. Here's our `Presenter base class`.

```ruby
# Base presenter class
# All presenters should inherit from this one.
class Presenter
  # Include all the modules you need or
  # send the view context to access rails helpers.
  include Rails.application.routes.url_helpers
  # ...

  include Presentable

  attr_reader :record

  def initialize(record)
    @record = record
    @klass = record.class
  end

  def method_missing(name, *arguments, &block)
    super unless record.respond_to?(name)
    record.send(name, *arguments, &block)
  end

  def translate_field_name(name)
    @klass.human_attribute_name(name)
  end

  # UrlHelper was being an ass, easier to go commando.
  def link_to(name, url, options={})
    href = { :href => url }
    content_tag(:a, name, options.merge(href))
  end
end
```

Then, we just need to inherit from the `Presenter` class and add our custom, clean methods. Using `delegate and method missing` will help us keep the class clean. I won't show you a presenter here, they'll look a lot like Ryan's one in his Railscasts so check that out first.

I will show you our controller though. Using the decent_exposure gem we managed to get super tiny controllers. Here's 1 of them:

```ruby
class RequestsController < ApplicationController
  expose(:filter) { params.fetch(:filter) { :all} }
  expose(:filtered_requests) { Request.filter(filter) }
  expose(:requests) { present(filtered_requests) }

  before_filter :find_request, :only => [:show, :destroy]

  def index
  end

  def show
    extend(RequestRedirector) # Another concern
    redirect_to_request
  end

  def destroy
    @request.destroy
    redirect_to requests_path(:filter => "new_requests"), :notice => "Se ha eliminado la solicitud"
  end

  private
  def find_request
    @request = Request.find(params[:id])
  end
end
```

Combining `presenters` and `form builders` will make your views even cleaner, giving a lot of pleasure to the future you! So please, just pretty-please, start using presenters and builders and leave helpers as simple as possible. Or, leave them for good.

You'll be amazed how different you'll feel when reading some old code or some co-worker's code.
