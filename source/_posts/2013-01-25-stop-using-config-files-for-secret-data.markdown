---
layout: post
title: "Stop using config files for secret data"
date: 2013-01-25 11:02
comments: false
categories: 
---

Seriously, it's **your** fault that Github exposed your keys. If you want to
add everything to your repo (not that this is a good idea though), at least
make it private.

If you want to have **per-environment variables, use the environment variables**.
*But my development env would be full of crap!* some of you may cry... well,
there's a simple solution for that. Use [Dotenv](https://github.com/bkeepers/dotenv).

With *Dotenv* you only need to add a `.env` file to your application root directory
and you're done. Here's a quick example:

```sh
# .env
S3_BUCKET=some_development_bucket
S3_SECRET=a_secret_key
```

Then in your code, simply use `ENV[]` and be happy (and safe, but mostly happy)

```ruby
# some config file
config.fog_directory  = ENV['S3_BUCKET']
```

And just like that, you can setup your server's config environments and not
worry about *supposed-to-be-private* public information.

One really cool thing about this is that this variables will only be loaded
in that specific application. It won't change **your** environment. So when
you stop the servers, your environment won't have the `.env` variables.

You can read more about [Dotenv here](http://opensoul.org/blog/archives/2012/07/24/dotenv/).