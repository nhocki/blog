---
layout: post
title: Getting my Emacs config playing nice with Boxen
excerpt: How I installed my Emacs configuration through Boxen after a lot of pain.
date: 2014-12-11 23:28
---

I'm moving all my development environment to [Boxen][boxen] and it's been great,
but I have to admit I've had some problems configuring some of my stuff [^1].

Here is how I got my [emacs config][emacsdotd] working (after fighting it for
over 2 hours!).

## Cloning my config

Cloning the [repository][emacsdotd] was probably the easiest part, there are a lot
of manifests showing how to do this so I got this working pretty quickly:

```puppet
$emacs = "${home}/src/emacsdotd"

file { $emacs:
  ensure => directory
}

repository { $emacs:
  source  => 'nhocki/dotemacs',
  require => File[$emacs]
}

exec { "pull emacs config":
  command => 'git pull origin master',
  cwd => $emacs
}
```

This will create a directory at `~/src/emacsdotd` and clone my [repo][emacsdotd]
there.

The `require => File[$emacs]` attribute will let Puppet know that this block
can only be excecuted when the `$emacs` one has finished (makes sure the directory
is created before cloning the repo).

## Installing Cask

I use [Cask][cask] to manage my emacs dependencies. The easiest way to install
it is [homebrew][brew], so to install it within Boxen, I just needed to:

```puppet
package { 'cask':
  ensure => present
}
```


With Cask installed, I needed to make it work with my configuration.

My `init.el` file looks for the `cask.el` file at `~/.cask/cask.el` (the
default location) but since I installed Cask and Homebrew with Boxen it gets
was at `/opt/boxen/homebrew/opt/cask/` which is kind of a weird place.

So I needed to create a symlink between that folder and `~/.cask`. For this
I added the following `file` block:

```puppet
$homebrew = "${boxen::config::home}/homebrew/opt"

file { "${home}/.cask":
  ensure  => 'link',
  target  => "${homebrew}/cask",
  require => Package['cask']
}
```

Notice that I needed the `require` attribute to point to the package installation
so **this block will only be executed when the Cask installation is finished**.

The `ensure => 'link'` makes this `file` block a symlink... Let's talk about
this for a second.

## Puppet Symlinks

Let's be clear, [Puppet Symlinks][psymlinks] are **really fucked up**.

If you're going to create a symbolic link between `<source_file>` and
`<target_file>` [^2], you would normally do `ln -s <source_file> <target_file>`,
here's the usage message from `ln`:

```
$ ln
usage: ln [-Ffhinsv] source_file [target_file]
       ln [-Ffhinsv] source_file ... target_dir
       link source_file target_file
```

But for some reason **in Puppet the `target` attribute is the `source_file`**,
which honestly WHAT THE FUCK!

We could use the *long* syntax (where the block's title is *not* the `path`) and it
is still **very** confussing:

```puppet
file { 'create symlink':
  path   => "target_file",
  target => "source_file",
  ensure => 'link'
}
```

I don't really know *why* they named the `source_file` *target* when `target` is the
name for the symlink in the actual `ln` command. This is really confussing and I
lost a lot of time on this!

Now that we know how to symlink stuff in Puppet, we can create the link for the
emacs configuration:

```puppet
file { 'create symlink':
  path   => "${home}/.emacs.d",
  target => $emacs,
  ensure => 'link'
}
```

This will create the `~/.emacs.d` directory and point to the project (`~/src/emacsdotd`).
Emacs will look for the `~/.emacs.d/init.el` file by default.

## Installing the dependencies

Now that we have a *kind-of-working* installation and everything is in the right
place, we can install all the dependencies in the `Cask` file.

We need to make sure that the repo has been cloned, and for some reason this
command (`cask install`) sometimes fails (can't find some file - happens
randomly AFAIK), so we'll need to be prepared for that:

```puppet
$emacsIns = "pull emacs config"

exec { 'install emacs dependencies with Cask':
  command => 'cask install',
  cwd => $emacs,
  require => Exec[$emacsIns],
  tries => 3,
  try_sleep => 1
}
```

This block will run `cask install` in our repo directory and try at most 3 times.

It will only run it once the `pull` action has happened in the repo directory. This is
really important because if we run `cask install` without a `Cask` file, the command
will fail.

It will also install or update automatically the dependencies if they ever change
in the Emacs repository, which is awesome.

With all this I got my installation working and now I can run `boxen` and have
everything configured.

## Hope this helps

First of all, you can find the whole file [here][gist].

I wrote this because I spent over 2 hours trying to figure out how to make this
works, hopefully you'll find this and don't go through the same pain I did,
specially with the symlinks.

If for any reason you know **why** Puppet chose to name `source` as `target` I
would really like to know, so please ping me on [Twitter][twitter].

[^1]: Probably because I have no idea how to use Puppet well.
[^2]: I did not came up with those names, those are the ones in the `ln` documentation!

[cask]:  https://github.com/cask/cask
[brew]:  http://brew.sh/
[gist]:  https://gist.github.com/nhocki/7744a6f939abd71880a9
[boxen]: https://boxen.github.com
[twitter]: https://twitter.com/nhocki
[emacsdotd]: https://github.com/nhocki/dotemacs
[psymlinks]: http://www.puppetcookbook.com/posts/creating-a-symlink.html
