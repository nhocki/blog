---
layout: post
title: "Fix Xcode Path"
date: 2012-05-09 19:38
comments: true
categories: 
  - xcode
  - lion
---

I updated my Xcode version this week and to my surprise they moved it from `/Developer` to `/Applications/Xcode`, the problem is that now all the developer tools went missing. Here's the easy solution for this:

`sudo /usr/bin/xcode-select -switch /Applications/Xcode.app/Contents/Developer/`

That's it. Just let Xcode know where to look for the developer tools again.