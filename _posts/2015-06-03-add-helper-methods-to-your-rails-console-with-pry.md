---
layout: post
title: Add helper methods to your Rails console with Pry
excerpt: Add helper methods to your Rails console with Pry
date: 2015-06-03 20:12
excerpt: Having custom helper methods in your rails console is amazing, but if you're using Pry setting it up can get messy.
categories:
- pry
- rails
---

There is no doubt that the rails console is amazingly helpful. At [Ride][ride]
we use it every day on our production application, so we wanted to add some
custom helpers to make things easier for us.

**But we use Pry and things got messy.**

## How does it normally work?

Normally adding a method is quite simple, [Brandon Keepers has a **great
post**][bkpost] about it. I added helpers that way at [We Heart It][whi] where
we used the default Rails console (`IRB`), but Ride uses [Pry][pry].

For the default setup, you just `include` a module in the `Rails::ConsoleMethods`
module from a `console` block in your `config/application.rb` file.

This works because, rails [extends][rails] `IRB::ExtendCommandBundle` with that
module so everything in there is available by default.

This is how you get the `controller` method and other [useful methods][37signals]
in your console.

But this doesn't work with Pry! _Why?_ Because `Pry` only defines
`Pry::ExtendCommandBundle` as an (empty) module [to not break Rails][prymodule]
but never uses it.

## Making it work with Pry

After digging into the Rails and Pry's source code, I found a way
to add our methods to the console by default. Instead of including our module in
the rails module, we need to `extend` Pry's default bindings
(`TOPLEVEL_BINDING`) like this:

```ruby
# config/application.rb
console do
  require 'my_app/console'
  TOPLEVEL_BINDING.eval('self').extend MyApp::Console
end
```

This is [exactly what happens][pryrails] in the `pry-rails` gem to add the
`Rails::ConsoleMethods` methods, but since this also happens inside a `console`
block we can't add our methods to the Rails module before it gets added to the
binding, that's why we have to manually extend it.

I [opened an issue][issue] in `pry-rails` since extending the `TOPLEVEL_BINDING`
seems like a hack to me, but I could not find another way to do it. I had
spent a lot of time trying to make it work and that worked, so ¯\\_(ツ)_/¯.

I really hope this saves someone's time (and sanity), I got really frustrated for
a while since there was no information about how to make this work (until now).

[issue]: https://github.com/rweng/pry-rails/issues/74
[whi]: http://www.weheartit.com
[pry]: https://github.com/pry/pry
[bkpost]: http://opensoul.org/2012/11/08/add-helper-methods-to-your-rails-console/
[ride]: https://www.ride.com
[rails]: https://github.com/rails/rails/blob/7fb45a6130598703b3da8fe9ef29a5c7a1eb9d09/railties/lib/rails/commands/console.rb#L62-L64
[37signals]: https://signalvnoise.com/posts/3176-three-quick-rails-console-tips
[prymodule]: https://github.com/pry/pry/blob/ee2513bdb43ad272bba6895645ce04bb2c3d6377/lib/pry.rb#L121
[pryrails]: https://github.com/rweng/pry-rails/blob/69f5990f3a687e2be209ff4083ecfdf5e9a6b7d4/lib/pry-rails/railtie.rb#L25
