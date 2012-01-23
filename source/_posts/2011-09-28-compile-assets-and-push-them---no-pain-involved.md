---
layout: post
title: Compile assets and push them - no pain involved!
date: 2011-09-28
comments: true
categories:
- heroku
- asset-pipeline
- rake
- rails
---
This week, I've been working on a side project with [Andrés
Mejía](http://andr.esmejia.com/) ([@andmej](http://twitter.com/andmej)) and
for some really really sad reason [Heroku](http://heroku.com) is not working
as expected (a simple git push should do the trick, but we're having a really
[weird issue](https://github.com/ddollar/rails_log_stdout/issues/4)).

At first, I thought it had something to do with the assets. Even-though
[Heroku](http://heroku.com) has [asset compiling tasks on their Cedar stack](h
ttp://devcenter.heroku.com/articles/rails31_heroku_cedar#the_asset_pipeline),
it didn't really worked for us at first. So I just decided to create a small
rake task to compile the assets and push them to Github.

The good thing about this rake task is that it will only push the assets. This
means, that **you'll need a clean git tree to run this task or it will die**.
This ensures that the automatic push **won't commit any non-related change to
the compilation of the assets**.

Now, you only have to run **rake assets:compile** and sleep well at night.
Here's the code. Hope it helps.

{% gist 1241135 %}
