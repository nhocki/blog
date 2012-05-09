---
layout: post
title: "Mixing Presenters and Helpers"
date: 2012-05-08 18:28
comments: true
categories: 
  - rails
  - oop
  - presenters
---

I just finished [The Rails View](http://pragprog.com/book/warv/the-rails-view) and the best chapter for me was about presenters. I've been [working with this](http://blog.nhocki.com/2012/01/31/thoughts-on-decorators-and-presenters/) for a while and I have to admit I love the approach presented on the view.

All the presenters I've done were [instantiated in the controller](http://blog.nhocki.com/2012/01/10/simple-presenters--decorators-on-rails/). In the book they suggest that **if the presenter doesn't depend on params or session variables it's the view responsibility to create it**. This is extremely flexible specially when we're presenting various objects on the same view. Here's an example to present users in our application, let's use the same presenter for the _index_ and _show_ actions, but let's make it in a really simple way!

### The Helper & View

Let's create the helper method first. It's really simple and will take a block that we can use to build a custom markup "on the fly".

{% gist 2640442 user_helpers.rb %}

Now, let's take a look at the views. In the **show** action, we're going to present only one user, and we can easily build the custom view for this. In the **index** action, we'll present all a "basic info" for the each user. We have that in a partial because this is used for all the collections of users in our application. Here's the view code:

{% gist 2640442 user_views.html.erb %}

### The Presenter, A.K.A What's happening here?

What makes this **really** clean is that we can actually use `<%=  %>` directly on our helper method. This works because we'll define the `to_s` method in our presenter! In this method we **render our partial in the context of the helper**, which is amazing. Here's the code for that:

{% gist 2640442 user_basic_presenter.rb %}

Is **that** simple! We have a unique interface (the helper) that will work great in both places! We could obviously make the presenter more complex, otherwise, why use it?

Even though I haven't used this approach a lot, I'm really happy with it. I felt the difference instantly and it made a lot of sense to me. Being able to mix this with a [flexible layout](http://blog.nhocki.com/2012/03/29/flexible-and-friendly-layouts-on-rails/) is really powerful!

### Feedback

I'm also really interested in how people are using presenters. I'd love to gain more feedbacks and points of view about this. Feel free to post a comment (here on in the gist) or ping me on Twitter, [@nhocki](https://twitter.com/nhocki).
