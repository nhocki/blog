---
layout: post
title: "Using Liquid `include` with DB templates"
date: 2012-09-05 17:37
comments: false
categories:
  - rails
  - liquid
  - rails views
---

I am working on a new application that will use some [Liquid][1] templates to
allow our clients to edit some templates.

We will store these templates in the Database, load them, parse them and render
them. So far, no magic.

But, I needed to have some "partials" for these. I wanted some common elements
(like image galleries) to be available for the client to use in that specific
template.

The way you normally accomplish this is by using an `include` tag, but I needed
a little bit more control. Not all the clients will have the same partials and
I also wanted thee clients to be able to edit these partials.

So after reading the [Liquid source code][2], I found a really easy way to do
this.

You just need an object that responds to `read_template_file` and it can take
1 or 2 arguments (please, go for 2). This method must return the Liquid **source**
of the partial (not a parsed template).

So, here's a really simple one that will work:

{% gist 3645632 %}

## Default FileSystem

You could also set the default `FileSystem` with
`Liquid::Template.file_system = MyNewFileSystem.new`. This may be useful if you
want the partials to be shared on the database but not different for every user.

## Any other way?

Even though I find this to be *really* easy, another way may exist. If you know
of a better way to do this, please [let me know][3].


[1]: http://liquidmarkup.org/ "Liquid Markup"
[2]: https://github.com/Shopify/liquid/blob/master/lib/liquid/tags/include.rb#L49-61 "Liquid include source code"
[3]: https://twitter.com/nhocki