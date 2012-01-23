---
layout: post
title: Transfer data from one Heroku app to another
tags: 
---
Today I needed to transfer all the data from one application (production
environment) to another (staging environment).

I'm pretty aware that the [taps gem](https://rubygems.org/gems/taps) exists
(which is great btw, and I use to pull the data from the production
environment to my development environment), but the problem with the taps gem
is that a table with a lot of data and some indexes will take a while to
migrate.

I guess it was my lucky day because [Thoughtbot](http://thoughtbot.com/)
[tweeted](https://twitter.com/thoughtbot/status/92957355330904064) with
exactly what I needed. Turns out that using the [PG Backups
addon](http://addons.heroku.com/pgbackups) it's really easy and fast!

{% gist 1089598 %}

That's all you need! Those 4 lines (actually, the last 2) will backup your
production database and then restore that backup in your staging database.

Here is the [original gist](https://gist.github.com/1089598).
