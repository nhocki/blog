---
layout: post
title: Managing Docker Machines with Ease
excerpt: Having multiple Docker Machines can be a pain, but there's nothing a few aliases won't fix.
date: 2015-12-23 16:51
---

[Docker Machine][dm] is amazing. It lets you create Docker hosts virtually
anywhere. Mix it with [Docker Compose][dc] and you'll be able to setup all your
systems pretty much anywhere.

Together, they are a great option for automated end-to-end testing. You can also
distribute development builds of features without deploying them to production servers.

At [Ride][ride], we can create servers in [Digital Ocean][do] and have the whole
backend running there. We have a repo with a `docker-compose` file that sets up
all systems **(thanks [Sergio][sergio]!)**.

But working with many machines can get a little bit confusing and long to type.

## Multiple Machines

Imagine we have two machines configured. `development` will be a local machine
and `json-api-10` will be hosted in Digital Ocean.

```
$ docker-machine create --driver virtualbox development

Running pre-create checks...
Creating machine...
(development) Creating VirtualBox VM...
...
```

```
$ docker-machine create --driver digitalocean \
      --digitalocean-size 2gb --digitalocean-access-token \
      $TOKEN json-api-10

Running pre-create checks...
Creating machine...
(json-api-10) Creating SSH key...
...
```

To make our docker client point to a specific machine, we need to set up some
environment variables...

But who wants to type (or remembers) `eval "$(docker-machine env <name>)"` anyway?!

## Use a Machine

We created the `use` function that will do just that:

```
use () {
	eval "$(docker-machine env $1)"
}
```

Now changing is really easy `use json-api-10` will point to the remote machine!

We can check that everything's working with `docker-machine ls`:

```
$ use json-api-10

$ docker-machine ls
NAME           ACTIVE   DRIVER         STATE     URL                      
json-api-10    *        digitalocean   Running   tcp://1.1.1.1:1234       
development    -        virtualbox     Running   tcp://192.168.99.100:2376
```

But this is not enough, when you have a lot of machines it's hard to remember
the names, so we need autocomplete...

## Adding Autocomplete

To add autocomplete (in zsh) we need to have a file called `_<command>` in a
directory that's in `$FPATH`.

In my case I have `~/.zsh/completion` there, so I added the following file `~/.zsh/completion/_use`:

```
#compdef use

_arguments "1: :(`docker-machine ls -q`)"
```

This makes everything better now! You can now `use <TAB>` and you'll get all
your machines there!

```
[~] use
api-json-api-10  development
```

I hope this saves you some time and helps you work with Docker even more. It's
totally worth it to give it a try!

<div class='big-skip'> </div>

#### Extra Aliases

I also have the following aliases because `docker-machine` and `docker-compose`
is pretty long and I'm very lazy:

```
alias d='docker'
alias dc='docker-compose'
alias dm='docker-machine'
```

If you miss `dc` (the polish calculator), you can still access it with `\dc`.

[do]: https://www.digitalocean.com/
[dm]: https://docs.docker.com/machine/
[dc]: https://docs.docker.com/compose/
[ride]: https://www.ride.com/
[sergio]: https://github.com/sergiobuj
