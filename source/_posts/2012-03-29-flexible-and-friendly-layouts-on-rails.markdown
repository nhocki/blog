---
layout: post
title: "Flexible and friendly layouts on rails"
date: 2012-03-29 23:16
comments: true
categories: 
  - rails
  - rails views
  - html
---

I've been working on a "new" project (I'm new to the project, the project itself is 'old') and at the beginning I had to edit a bunch of stuff in the views.

A friend needed to change the header of the page on **one** action of the `UsersController`. He is new to rails and was struggling with this, so he asked me for help.

It's pretty common to have a `header` partial rendered on the view. If you're a lovely developer you would have `render "header"` and not something like `render "layouts/header"`. The reason behind this is simple. [Template Inheritance](http://railscasts.com/episodes/269-template-inheritance "Railscasts Template Inheritance"). This means that if you're working on the `UsersController`, when rails looks for the `header` partial, it will first look for `users/header` moving up the parents chain, ending in `application/header`. So, next time, put all your *layout* stuff in `application/_partial` so you can easily override it!

But this is a _controller wide_ solution, since all the actions from that controller will render the same partial. So, how do you achieve this on a single action? My solution is a 3 line helper:

{% gist 2246251 application_helper.rb %}

Now all you have to do is fill your layouts with "content boxes" that can be changed on an controller or action-specific need, giving great flexibility.

How would you get around this?
