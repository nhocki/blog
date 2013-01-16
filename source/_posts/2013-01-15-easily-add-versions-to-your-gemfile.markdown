---
layout: post
title: "Easily add versions to your Gemfile"
date: 2013-01-15 22:31
comments: false
categories: 
  - ruby
  - bundler
  
---

Since [bundler][1] came out, managing dependencies
on ruby applications has been amazing. Combined with [some][2] [bootstrap][3] [script][4],
it's really easy to download some source code, install, and run an application.

There is still some room for improvement though. I've always supported the use
of [SemVer][5] and pessimistic locking (the `~>`). This will allow you to
update applications without pain. And, with all this rails vulnerabilities,
you need just that.

But, since people don't always have the versions on their `Gemfile`, this can get
a little messy. So, I've created a simple gem to help you with that.

### Welcome version_gemfile

* 1. Simply install it: `$ gem install version_gemfile`
* 2. Go to your rails application: `$ cd ~/my/ruby/application`
* 3. And add the versions to your *Gemfile*: `$ version_gemfile`


## How does it work?

This gem will go through each line in your `Gemfile` looking for dependencies
that have no versions. Then, will look into your `Gemfile.lock` for the version
you are currently using and add a pessimistic lock on that version.

The code can be found on [Github][6]. If you have any problems, open an issue there
or ping me on [Twitter][7].

[1]: http://gembundler.com/ "Bundler"
[2]: http://37signals.com/svn/posts/2998-setting-up-a-new-machine-for-ruby-development "37 Signals bootstrap script"
[3]: http://zachholman.com/talk/unsucking-your-teams-development-environment/ "Zach Holman's talk on development environment"
[4]: http://ngauthier.com/2012/08/one-click-development.html "One click development"
[5]: http://semver.org/ "SemVer: Semantic Versioning"
[6]: https://github.com/nhocki/version_gemfile "Version Gemfile source code"
[7]: https://twitter.com/nhocki "My twitter profile"