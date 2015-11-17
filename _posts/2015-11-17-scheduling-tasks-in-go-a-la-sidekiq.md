---
layout: post
title: Scheduling tasks in Go à la Sidekiq
date: 2015-11-17 11:30
excerpt: Have you ever wondered how does Sidekiq schedule jobs to be perfomed in the future? Here's how it works behind the scenes and a small example using Go & Redis.
categories:
- redis
- golang

---

[Sidekiq][sk] is an amazing project. It let's you process things in the background
and it's really easy to use. One of its most useful features is to
[schedule a task to be queued in the future][schedule].

For example, if we want to send an email to a user 24 hours after they've
registered, we can do it with:

```ruby
WelcomeEmailWorker.perform_in(24.hours, user_id: '123')
# or
WelcomeEmailWorker.perform_at(24.hours.from_now, user_id: '123')
```

Have you wondered how does Sidekiq do that? The answer is **Redis Sorted Sets**!

## Sorted Sets 101

Redis has a data structure called [sorted sets][szet] (or _**ZSET**_ as Redis
calls them). They are “collections of non repeating *strings* where each one is
associated with a score”.

`ZSET`s are **very** powerful, you can use then to [get trending content][ps],
build dashboards or leaderboards and much much more.

To schedule tasks we only need a few operations:

```sh
$ redis-cli

# adding 'Hello' with score 1.0 to `sortedset`
127.0.0.1:6379> ZADD sortedset 1.0 "Hello"
(integer) 1

# adding 'World' with score 2.0 to `sortedset`
127.0.0.1:6379> ZADD sortedset 2.0 "World"
(integer) 1

# get members from `sortedset` with scores [-inf, +inf]
127.0.0.1:6379> ZRANGEBYSCORE sortedset -inf +inf
1) "Hello"
2) "World"

# delete the 'Hello' element from `sortedset`
ZREM sortedset "Hello"
(integer) 1
```

While `ZADD`, `ZRANGEBYSCORE` and `ZREM` functions are all we need, you should
read the documentation of [all zset functions][zsetf], chances are you'll find
something you need there.

## Scheduling with ZSETs

To schedule a task, we add it to a set, retrieve it when the time is right and
then queue it for a worker to process it. Once we queue it, we remove it from
the set.

But how does Sidekiq use _zsets_ to schedule data? It stores the task using
**the timestamp of *when* you want the task to be queued at as the *score*
for that task.**

With that in mind, we can write the `PerformIn` and `PerformAt` functions. Let's
assume we're storing them in the `jobs:scheduled` set.

```go
// scheduler.go

package main

import (
	"fmt"
	"os"
	"os/signal"
	"time"

	"github.com/fzzy/radix/redis"
)

var redisConn *redis.Client

func init() {
	var err error
	if redisConn, err = redis.Dial("tcp", "127.0.0.1:6379"); err != nil {
		panic(err)
	}
}

// PerformIn schedules a given task to be executed in the given duration.
func PerformIn(in time.Duration, task string) {
	at := time.Now().Add(in)
	PerformAt(at, task)
}

// PerformAt schedules a task to be executed at a given time.
func PerformAt(at time.Time, task string) {
	fmt.Printf("> Scheduling %s with score %d\n", task, at.Unix())
	redisConn.Cmd("zadd", "jobs:scheduled", float64(at.Unix()), task)
}

func main() {
	at, _ := time.Parse("2006-Jan-02", "2016-Jan-01")
	PerformAt(at, "Happy New Year!")
	PerformIn(2 * time.Minute, "Snooze wakeup alarm!")
}
```

Running this gives us the following output:

```
$ go run scheduler.go

Scheduling Happy New Year! with score 1451606400
Scheduling Snooze wakeup alarm! with score 1447653988
```

We can confirm that the tasks were scheduled (and in the right order) in Redis:

```sh
$ redis-cli

127.0.0.1:6379> ZRANGEBYSCORE jobs:scheduled -inf +inf
1) "Snooze wakeup alarm!"
2) "Happy New Year!"
```

## Getting Tasks Out

Now that we're getting tasks on a sorted set, we can poll it every few seconds
(Sidekiq's default is 15 seconds), looking for tasks that should run.

For this, we get all the tasks with `score <= time.Now()` and queue them where
they belong so workers can pick them up as any other task.

This is really simple to do in `Go` with a `time.Ticker`:

```go
// poller.go

package main

import (
	"fmt"
	"os"
	"os/signal"
	"time"

	"github.com/fzzy/radix/redis"
)

var redisConn *redis.Client

func init() {
	var err error
	if redisConn, err = redis.Dial("tcp", "127.0.0.1:6379"); err != nil {
		panic(err)
	}
}

// Poll checks Redis to determine whether scheduled tasks need to be run or not.
func Poll(interval time.Duration, done <-chan os.Signal) {
	ticker := time.NewTicker(interval)
	for {
		select {
		case <-ticker.C:
			fmt.Print("> Checking for scheduled tasks... ")
			nowUnix := time.Now().Unix()
			response := redisConn.Cmd("ZRANGEBYSCORE", "jobs:scheduled", "-inf", float64(nowUnix))
			tasks, _ := response.List()

			if len(tasks) == 0 {
				fmt.Println("No tasks found.")
			}

			for _, task := range tasks {
				fmt.Printf("Queue task: %s\n", task)
				redisConn.Cmd("ZREM", "jobs:scheduled", task)
			}
		case <-done:
			fmt.Println("Shutting down poller")
			return
		}
	}
}

func main() {
	c := make(chan os.Signal)
	signal.Notify(c, os.Interrupt)
	fmt.Println("Polling Redis every 15 seconds for scheduled tasks...")
	Poll(15 * time.Second, c)
}
```

Running `poller.go` gives us the following output:

```
$ go run poller.go

Polling Redis every 15 seconds for scheduled tasks...
> Checking for scheduled tasks... Queue task: Snooze wakeup alarm!
> Checking for scheduled tasks... No tasks found.
> Checking for scheduled tasks... No tasks found.
```

It's worth mentioning that Sidekiq does the polling operation a little bit
differently to prevent a lot of tasks being returned by `ZRANGEBYSCORE`. This
way the amount of memory used in the operation is significantly smaller.
More info about this approach can be found in [mperham/sidekiq#843][pr843].

If you're interested in how Sidekiq implements scheduling tasks, take a look
at the [scheduled.rb file in the source code][scheduled].

It is also really important to queue the task instead of handling it inline. The
poller's responsibility is just to poll the queue, not processing jobs.


## Process Sidekiq jobs from Go

This is a simplified explanation of how a small part of Sidekiq works, if you want
to process sidekiq jobs from Go, take a look at [Go Workers][gworkers].
The project looks pretty complete and handles a lot of things.

### Sidekiq's Payload

Below you can find the payload that Sidekiq adds to the set as a JSON string:

```json
{
   "args" : [ 123 ],
   "retry" : 3,
   "created_at" : 1447658966.98817,
   "jid" : "72c60a524d7d3230741d1540",
   "queue" : "some_queue",
   "backtrace" : true,
   "class" : "SomeWorker"
}
```

You can check it with `ZRANGE schedule 0 100`.

## Conclusion

So that's how Sidekiq manages to schedule tasks. This is a simplified version
and only talks about the _scheduling_ and not queuing jobs for Workers to consume.

Sidekiq takes scheduling to the next level by having random query intervals
for each process polling the queue and other options giving you a lot of flexibility.

However understanding the underlying concept is simply polling for a sorted
set (that acts like a heap) allows you to build something similar in other
languages.

[The whole example can be downloaded here][gist]. If you've got any questions
about it, feel free to ping me on [twitter][twitter].

[sk]: http://sidekiq.org/
[szet]: http://redis.io/topics/data-types#sorted-sets
[zsetf]: http://redis.io/commands#sorted_set
[ps]: http://blog.nhocki.com/2014/12/02/trending-content-with-ruby-and-redis/
[gist]: https://gist.github.com/nhocki/a7e5c55ff4387020eac3
[gworkers]: https://github.com/jrallison/go-workers
[pr843]: https://github.com/mperham/sidekiq/pull/843
[schedule]: https://github.com/mperham/sidekiq/wiki/Scheduled-Jobs
[scheduled]: https://github.com/mperham/sidekiq/blob/fe43e1cbcd0498a37a69581ace523bc36630cd3f/lib/sidekiq/scheduled.rb
[twitter]: https://twitter.com/nhocki
