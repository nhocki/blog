---
layout: post
title: Displaying deploys with Capistrano and Graphite
excerpt: How a simple Capistrano task can make your Graphite charts much better.
date: 2014-12-06 22:11
---

At [We Heart It][whi], we collect a lot of data in several graphs & dashboars.
But the graph data alone might not always be enough, sometimes you need context about
what actually changed to know why the graph changed (like a big change on response times).

Thankfuly, Graphite now supports *[events][1]* (although is not very well documented),
so it's actually really easy to display when the code was deployed and add some information
about the deploy itself.

We use [Graphana](http://grafana.org/) for most of our dashboards. It is an
amazing graphing tool, that gets data from [Graphite](http://graphite.wikidot.com/)
and displays it in a really nice way. You should really try it.

## The Basics

First we need to create an event. This is easily done by `POST`ing some simple JSON:

```sh
curl -X POST http://graphite.example.com/events -d \
'{"what": "Code deploy", "tags": "rails-deploy", data: "commit information"}'
```

Then we need to enable [Graphana annotations][annotations] and display the `rails-deploy`
tag on our dashboard. The result will look like this:

![Deploy Events]({{ site.url }}/assets/deploy_events.png)

## Using Capistrano

Since we use Capistrano to deploy our apps, we can automatically create deploy
events every time a deploy finishes, here's our task:

```ruby
after "deploy", "graphite:mark_deploy"

namespace :graphite do
  task :mark_deploy do
    commit_information = `git log -n 1 --pretty=format:'%h - %s - %an' --abbrev-commit`

    auth = { username: "foo", password: "bar" }
    body = {
      what: "rails code deployed.",
      tags: "rails-deploy",
      data: "#{fetch(:branch)} deployed at #{Time.now} ( #{commit_information} )"
    }

    HTTParty.post("https://graphite.example.com/events/", basic_auth: auth, body: body.to_json )
    puts "Deploy sent to Graphite."
  end
end
```

The `commit_information` information will give you something like this:

```
68305ff - Allow wildcard increment/decrement on stats counters. - Nicolás Hock Isaza
```

You can use this information as a starting point when things go wrong with a
deploy.

This is a very easy thing to add and gives you a lot of extra context around your
graphs, I found it really useful when we added it for the first time (and use it
a lot today as well).

[1]: https://code.launchpad.net/~lucio.torre/graphite/add-events/+merge/69142
[annotations]: http://grafana.org/docs/features/annotations/
[whi]: http://weheartit.com
