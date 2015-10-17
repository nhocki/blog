---
layout: post
title: Queue Based Architectures at RubyConf Colombia
excerpt: Today I talked about Queue Based Architectures in the first ever RubyConf in Colombia. These are my slides.
date: 2015-10-16 21:36
---

For the first time, we have a [RubyConf in Colombia][rbconf] and I had the
privilege to be a speaker. I think the event was amazing. **Thanks a lot to all
the organizers**.

I talked about **Queue Based Architectures** and how they can help you develop
and maintain your application in the modern days.

If there's something I wanted people to take away from my talk was that

> Asynchronous messaging is fundamentally a pragmatic reaction to the problems
of distributed systems. Sending a message does not require both systems to be up
and ready at the same time.

> -- Enterprise Integration Patterns

That quote gives a short summary of the biggest advantage of having a reliable
queue handling the communication between services: Your users will feel that your
application is working even if some parts of it are not.

Here are my slides both in English and Spanish. I'd really appreciate any
feedback, so ping me [on twitter](https://twitter.com/nhocki).

## English Slides

<script async class="speakerdeck-embed" data-id="4d51d7e680104baabffd726874b540f1" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

## Spanish Slides

<script async class="speakerdeck-embed" data-id="7b29ce2d85b74fba93c8816c841d23a8" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

[rbconf]: http://www.rubyconf.co/
