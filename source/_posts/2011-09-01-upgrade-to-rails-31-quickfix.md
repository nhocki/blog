---
layout: post
title: Upgrade to Rails 3.1 Quickfix
tags: 
---
In case you missed it, [Rails 3.1 is out](http://guides.rubyonrails.org/3_1_release_notes.html)! So please, stop using the release candidates and update now!

**Short Version; TL;DR**

**If you're can't see your images in your production environment, add this to your `production.rb` file**

{% gist 1185431 %}

Yesterday, I was updating [Timehub](http://timehub.net) to Rails 3.1, we had a rails3.1 branch and we were just waiting for the official version to change it.

As soon as I got home from the [Coffee Grid](http://coffeegrid.org/) meetup (where we talked about Timehub), I decided to update it. What could possibly go wrong? We've been using a release candidate with **no more than 3 days old** and it was working just fine.

We changed our `Gemfile` and started to use Rails 3.1. We tested the app on development and everything was working as expected, so I just relaxed while deploying.

I **always** look at the server after a deploy, and to my surprise, **no images where shown in the production environment**. WTF? We were compiling our assets (with a small Capistrano task).

We watched the source of the page and instantly found the problem. The server was not looking for the images with the MD5 digest at the end. This was REALLY weird because we were using `image_tag` everywhere so we shouldn't care about it!

This means that the server was looking for `/assets/rails.png` instead of
`/assets/rails-3289328982JAKSDJK.png`

I looked to the awesome [Asset Pipeline Guide](http://guides.rubyonrails.org/asset_pipeline.html), I asked around with Twitter, I restarted the server several times. I did everything and nothing worked until [@guilleiguaran ](https://twitter.com/#!/guilleiguaran)gave me the answer on twitter!

You simply need to add those 2 lines to fix your assets (look up if you missed it!)

The problem is that it wasn't really documented and that the release candidates didn't have that! So it was **REALLY** hard to guess. Actually, I think @guilleiguaran knew because he made those changes!

After that, we could easily update and now we're 100% on Rails 3.1!

Hope this helps you too!
