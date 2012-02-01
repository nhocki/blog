---
layout: post
title: "Thoughts on Decorators & Presenters"
date: 2012-01-31 18:12
comments: true
categories: 
  - decorators
  - presenters
  - rails
  - oop
---

I have been developing a new app using a lot of [decorators and presenters](http://blog.nhocki.com/2012/01/10/simple-presenters--decorators-on-rails/). I've been highly influenced by [Avdi](http://about.avdi.org/) (both his [blog](http://avdi.org/devblog/) and his [Objects in Rails book](http://avdi.org/devblog/2011/11/15/early-access-beta-of-objects-on-rails-now-available-2/)) and [Steve Klabnik](http://steveklabnik.com), but now I have a problem. My decorators are starting to get big. I'll have to use `concerns` for that. Expect another post about it (maybe).

**I want to warn you that I am (by any means) an expert on the subject. So I'm more than open to any feedback you can give me.**

> All presenters are decorators, but not all decorators are presenters.

> A decorator is a class that adds some sort of functionality to another
class. A presenter is a class that adds some sort of presentation
formatting functionality to another class.

(Steve Klabnik [here](https://groups.google.com/forum/#!msg/objects-on-rails/htAopf3k_dM/qJMq6QAfMvsJ))

I think that what a decorator *should* do is give a standard API for the processed data stored somewhere (probably a database). I don't think a decorator should be in charge of the markup of the presented data. For me, it makes no sense that the decorator (that is processed data for me), knows *how* it is being shown to the user.

That said, I think the **same** decorator should be used to build the JSON response and the HTML response. The HTML markup and JSON structure are *just* \'markup\'. So, they can be treated as templates.

### So, where do the logic in the template goes?

I had this discussion with a [friend](http://mheroin.com/), and he asked me that, with this approach, how would he add elements to a view depending on a condition, for example, the famous \'***You are logged in as nhocki***\' if there's a user logged in.

I think this should be created inside helper methods that takes a block. Inside that block you can use the **decorator** methods to show the needed data. You can use those methods in your JSON builder too (try out the new [Jbuilder](https://github.com/rails/jbuilder)). You could bundle up these methods and re-use them in your projects too.  

Here's an example:

{% gist 1714913 %}

### Maybe I have the names wrong

As I mentioned before, I'm no expert on design patterns. And I'm pretty *new* to presenters and decorators. So, maybe my approach has a different name. I don't really care how this is called, I feel it's a more natural approach and it keeps the template and the processed data in different parts.

### Please provide feedback

It is also possible that I have it all wrong, that's why I'm really interested in your opinion. I'm eager to learn about everything from anyone, so I would love to have some feedback about this. You can post a comment below or send me a message on [twitter](http://twitter.com/nhocki).