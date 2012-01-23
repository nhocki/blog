---
layout: post
title: Simple scrollbar synchronization with jQuery
tags:
- coffeescript
- jquery
- collatzio
---
I'm working on a really cool project on my University. It's a web application
to compare book editions and help professors build what is know as the
Collatzio. The Collatzio is simply the "original" version of the book. That
means, to have the book the author wanted to write and not the one that the
editors wanted to publish.

I needed to have two versions of the book aligned side by side. So, the
HTML/CSS was pretty simple. Just have two simple divs with **overflow-y:
auto;** to make the scrollbar appears on each div.

The coffeescript code I used is really simple, here it is:

{% gist 1233545 %}

Hope it helps someone. I struggled a little bit with the "common" way of
scrolling things. People usually use the `animate` method to scroll the body,
but I couldn't use it because it kept triggering the scroll event on the
elements, so it was an infinite loop.

``` javascript
$("html, body").animate({ scrollTop: 0 }, "slow");
```

  
