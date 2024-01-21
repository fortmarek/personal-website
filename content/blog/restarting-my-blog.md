---
title: Restarting my blog
date: "2024-01-21"
tags:
  - "Web"
description: "Why I'm restarting my blog with Eleventy."
---


I got really motivated lately to start blogging again, mainly inspired by all the people blogging (and posting about blogging) on Mastodon. Seeing multiple services where people share their thoughts and ideas turn into fascist hell sites, people's content being hidden behind paywalls (looking at you Medium), or the whole rise of AI-generated content, I value more and more reading personal content created by smart and interesting individuals hosted in the free, open web, outside the reach of the world's billionaires.

And while I do already have my own personal page with my blog posts when I wanted to add a new post last week after two years not adding anything, I ran into an issue – `npm install` would throw up an error. I spent a couple of minutes of fixing it but after that I decided to rewrite my personal page from scratch.

When I created my personal page in 2021, I decided to use Gatsby. I started with a neat template, did some smaller changes, and that was it. It was quite easy to start with Gatsby, I have to say. But my issue with it is that it's React-based – and for something simple as my personal site, I really don't want to reach for that.

### Choosing a new technology

My requirements when choosing my new stack for personal site were:
- Blog posts written in markdown
- Easy to migrate off to a new stack down the line

The latter point has a bunch of implications – once you choose React, it's hard to move elsewhere. So, what will be still relevant in 10, 20 years? Yup, HTML and CSS. I love the current renaissance of going back to the Web platform principles.

I did consider using _only_ HTML and CSS – but I still wanted to write my posts in markdown as that's for me the natural way to write. I also wanted to spend as little time as possible on creating my new site.

So, instead of completely pure HTML and CSS site, I went with [Eleventy](https://www.11ty.dev/). This tool is one of the most popular ones, as highlighted in [this](https://buttondown.email/ownyourweb/archive/issue-04/) "Own Your Web" post (I highly recommend this newsletter btw!)

What I love about it is that most of your site can still be written in HTML and CSS, but then the tool takes care of turning markdowns into HTML, creating an RSS feed, and you have a basic layer for bits of business logic using the [Nunjucks](https://mozilla.github.io/nunjucks/templating.html) format.

If Eleventy goes unmaintained at any point, moving to a different static site generator (or to HTML & CSS) will be easy as the majority of the website is already written with Web's primitives.

You can always check out the source code of my [website on my GitHub](https://github.com/fortmarek/personal-website).

