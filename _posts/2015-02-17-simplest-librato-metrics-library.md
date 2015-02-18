---
layout: post
title: Simplest Librato Metrics library
excerpt: How to <em>easily</em> track metrics to Librato from Heroku using log drains. Without dependencies and in less than 30 lines of code.
date: 2015-02-17 11:27
---

[Librato][librato] is a great platform to track metrics and has a really simple
gem to track your [rack based apps][lr], but what if yours is not one? And if
you're on Heroku you can't use [StatsD][sd]... So what do you do?

It turns out that **Librato can get metrics from the application's log** so by
[adding a drain][drain] to your Heroku app you'll have the simplest metrics
tracking library ever:

```ruby
class Metrics
  def self.increment(metric, amount = 1)
    logger.info "count##{metric}=#{amount}"
  end

  def self.time(metric, **options, &block)
    if block_given?
      start_time = Time.now
      block.call
      total = (Time.now - start_time) * 1000
    else
      total = options[:time]
    end
    logger.info "measure##{metric}=#{total.round}ms" if total
  end

  def self.gauge(metric, amount)
    logger.info "sample##{metric}=#{amount}"
  end

  def self.logger
    @logger ||= Logger.new(STDOUT)
  end
  private_class_method :logger
end
```

And now getting metrics from your application is really, really simple:

```ruby
Metrics.increment('shares')
Metrics.gauge('queue_size', 1)
Metrics.time('expensive_operation') { # do some work }
```

You can check out all the things you can do with Librato and log metrics at the
[Heroku DevCenter][devcenter].

## Metrics in your logs

Probably the thing I like the most about this approach is having your metrics
in your application log waiting for them to be can be consumed and analyzed
from any service... **Your application is a producer, Librato is
_just_ a consumer**.

You could add [Kibana][kibana] with its powerful queries to get stuff out from
the metrics (and other data), or have multiple teams consume the metrics
and build different applications.

Setting up alerts based on metrics becomes trivial with services like
[Papertrail][papertrail]

Bottom line, [logs are awesome][logs], and **you should take advantage of that**!

[librato]: https://www.librato.com/
[lr]: https://github.com/librato/librato-rack
[sd]: https://github.com/etsy/statsd/
[drain]: http://support.metrics.librato.com/knowledgebase/articles/265391-heroku-native-and-custom-metrics-without-the-libra
[kibana]: http://www.elasticsearch.org/overview/kibana/
[logs]: http://engineering.linkedin.com/distributed-systems/log-what-every-software-engineer-should-know-about-real-time-datas-unifying
[papertrail]: https://papertrailapp.com/
[devcenter]: https://devcenter.heroku.com/articles/librato#custom-log-based-metrics
