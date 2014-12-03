---
layout: post
title: Colorize your rake output
date: 2011-11-30
excerpt: Color your ruby scripts output with no pain.
categories:
- rake
- ruby
- rails
---
I've been writing a lot of rake tasks lately. I like to have a lot of output (feedback) from my tasks, at least while I'm writing them. I have colors in my tests, but my output is really ugly, so I took [some code](https://github.com/andmej/chatte/blob/0b5509f800ed514ee302a8019c0f8cd410bf40da/client.rb#L15-24) from [Chatte](https://github.com/andmej/chatte), - a simple chat application a [friend](http://andr.esmejia.com/) (and awesome [developer](https://github.com/andmej/)) wrote for college- and made some modifications for it. Here's my module right now...

```ruby
# https://gist.github.com/nhocki/1410582
module Colors
  def colorize(text, color_code)
    "\033[#{color_code}m#{text}\033[0m"
  end

  {
    :black    => 30,
    :red      => 31,
    :green    => 32,
    :yellow   => 33,
    :blue     => 34,
    :magenta  => 35,
    :cyan     => 36,
    :white    => 37
  }.each do |key, color_code|
    define_method key do |text|
      colorize(text, color_code)
    end
  end
end

# green "Hello"
#   => "\e[32mHello\e[0m"
```

Now, you just need to `include Colors` in your tasks and enjoy your life.

I know there's a [colorize](https://github.com/fazibear/colorize) gem out there, but I don't like their syntax. I prefer Andres' approach much more, so maybe I'll turn this into a really simple gem. Do you think it's worth it?
