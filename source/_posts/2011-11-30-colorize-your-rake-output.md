---
layout: post
title: Colorize your rake output
date: 2011-11-30
comments: true
categories:
- rake
- ruby
- rails
---
I've been writing a lot of rake tasks lately. I like to have a lot of output
(feedback) from my tasks, at least while I'm writing them. I have colors in my
tests, but my output is really ugly, so I took [some
code](https://github.com/andmej/chatte/blob/master/client.rb#L15-24) from
[Chatte](https://github.com/andmej/chatte/), - a simple chat application a
[friend](http://andr.esmejia.com/) (and awesome
[developer](https://github.com/andmej/), btw) wrote for college- and made some
modifications for it. Here's my module right now…

{% gist 1410582 %}

Now, you just need to `include Colors` in your tasks.

I know there's a [colorize](https://github.com/fazibear/colorize) gem out
there, but I don't like their syntax. I prefer Andres' approach much more, so
maybe I'll turn this into a really simple gem. Do you think it's worth it?
