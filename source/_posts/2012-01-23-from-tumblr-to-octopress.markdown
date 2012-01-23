---
layout: post
title: "From Tumblr to Octopress"
date: 2012-01-23
comments: true
categories: [Tumblr, Octopress]
---

I wrote an article yesterday about URL design and I spent about an hour fighting with [Tumblr](http://tumblr.com/) to format my `code inside a pre (or code) tag`. I knew I was done with them. Today, I'm giving [Octopress](http://octopress.org/) a try. I honestly wanted to try [Jekyll](http://jekyllrb.com/) but I was really lazy to code my HTML. So, for now, I'm welcoming Octopress.

I really love the fact that I can write in [Markdown](http://daringfireball.net/projects/markdown/), it's really powerful yet simple that makes me focus on writing and not formatting. I know I can use Markdown in Tumblr but I don't have Textmate there, so this is a big +1 for Jekyll/Octopress.

Moving Posts From Tumblr
---

There's a [great script](https://github.com/mojombo/jekyll/wiki/Blog-Migrations) to move posts from Tumblr. They'll even be downloaded as markdown, but I still had to edit them to add the Gist snippets. Thankfully I don't have a lot of posts so it didn't took a lot.

Deploying
---

I've deployed my blog to [Heroku](http://heroku.com) now. It was supposed to be really simple. It was, after I realized that the `public` folder is ignored by the git. I spent an hour of my life there :-( !

<pre>Remove the public folder from your .gitignore</pre>

I'll probably post more about Octopress when I get to know it better. There are a bunch of plugins and stuff that may be useful, or maybe, it'll suck and I'll have to go and find something else!

