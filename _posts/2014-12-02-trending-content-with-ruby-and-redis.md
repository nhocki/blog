---
layout: post
title: Trending Content with Ruby and Redis
date: 2014-12-02 22:28
---

I've released a new gem called [Popular Streams][1], with this gem you can easily
get "trending" content based on events. It is based on a really nice [blog post][2]
by Micah Fivecoate on Heyzap. There are several implementations in that post, you
should check it out.

The gem is really easy to use and has a nice README on how to do it, so I won't get
into a lot of details on **how** to use it here. This post is mostly about **why**
the gem works (and a little bit of the math behind it). The source is on
[GitHub][1] if you want to look at it.

## How it works? And why?

So, what's popular *right now*?. For example, at [We Heart It][whi] people *heart*
images that inspire them, and they do that *a lot*. We get around 13 million hearts
per day. The trick about popularity is that it will [decay over time][3]. This means
that an image that was *hearted* 10 times week ago is less popular than an image
that was hearted 6 times today. Imagine each heart being a vote on an image.

The way the algorithm works is that, every day, we'll divide the weight for each
vote by half. Going through all the hearts is obviously impossible for us, but
luckily there's a formula to know the score of a specific vote on a specific time.

![Half Life]({{ site.url }}/assets/half_life.png)

Now the tricky part is knowing what time `t` we're at. But if we fixate the starting
time (`epoc`) and we compare everything against that, we can just get the value of
the vote at `epoc`:

![Half Life at Epoc]({{ site.url }}/assets/half_life_epoc.png)

That value is what this specific vote will be worth at `epoc`. So now our problem
becomes really easy! Every time an image gets a vote, we add that value to the
score of that image (at `epoc`).

At first we will add **really** small values, but it doesn't matter because all
values will be small. And they will get bigger and bigger over time, but they will
all getbigger in the same proportion. This is pretty awesome and that's what made
me chose Micha's solution.

One problem with the gem is when `epoc` is much smaller than `t`, so the value we
will add is really big and we might get an overflow, but we can control that a
little bit by selecting a nice `epoc` and a nice `half_life`. With the ones that
the gem has by default, it should be OK for about 5 years, so I'm not really worried
about that right now.

## How to I use it?

Here's a simple example on how to use the gem to get the popular tags of a site.

```ruby
stream = PopularStream.new("popular_tags")

stream.vote(field: 'rubygems')

# You can also add an optional `weight` param.
stream.vote(field: 'ruby', weight: 2)

# And you can even specify when the event happened.
# Notice that time is a number, meaning seconds since Epoc.
time = Date.yesterday.to_time
stream.vote(field: 'rubygems', time: time.to_i)

stream.get # => ['rubygems', 'ruby']

# You can pass `limit` and `offset`
stream.get(limit: 1, offset: 1) # => ['ruby']
stream.get(offset: 10) # => []

# You can ask to get the scores as well
stream.get(with_scores: true) # => [['ruby', 0.00001]]
```

Really easy, right? So go, checkout the gem and let me know what content you
start surfacing!

[1]: https://github.com/nhocki/popular_streams
[2]: http://stdout.heyzap.com/2013/04/08/surfacing-interesting-content/
[3]: http://en.wikipedia.org/wiki/Exponential_decay
[whi]: http://weheartit.com
