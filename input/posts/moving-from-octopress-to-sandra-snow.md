Published: 2014-01-21
Title: Moving from Octopress to Sandra.Snow
Lead: I have had a blog since April 2012. In this post I will cover how I migrated my blog from Octopress to Sandra.Snow.
Author: richtasker
permalink: /2014/01/21/moving-from-octopress-to-sandra-snow
Tags:
  - Sandra.Snow
  - Blogging
---
This is a slightly belated post, but better late than never.
I have had a blog since April 2012, I was not a massive blogger before this year I had post about 4 times. Near the end of last year I migrated my blog from Octopress to Sandra.Snow.

When I originally created my blog, I created it on Posterous before it was acquired and shut-down by Twitter, I then migrated to [Octopress](http://octopress.org/). Created by Brandon Mathis, it is built on top of 
[Jekyll](http://jekyllrb.com/). With Jekyll you have to setup all your templates, CSS, etc. before you can start blogging. Octopress gives you a default template out of the box, simply clone from the [github repo](https://github.com/imathis/octopress), install some dependencies and your ready to go.

Octopress/Jekyll is written in Ruby and works by parsing markdown text files and rendering it as static HTML.
This means there is no longer a need for databases, no more comment moderation or updates. The only downsides I had were that I had to setup Ruby (I did this on a linux VM) and my Ruby is not very strong but it did get better. To write a post I had to be infront of my desktop where I had the VM. I did eventually find out about the [Cloud 9 IDE](https://c9.io/) (<<< I think this is great, why can't all IDEs be like this.) which got around this issue.

## Then along came Sandra

Then a few of the guys behind NancyFX, a web framework inspired by Sinatra, released Sandra.Snow. Sandra is inspired by Jekyll but for .Net. As soon as I found out about Sandra, I was very quick to start investigating. There was virtually no hold ups migrating my existing posts, the markdown files had almost the same format. The only thing that took the time was converting all my page templates to Razor views.

Octopress came with a rake file which had a load of tasks for adding new pages, post, generating the HTML output and more. I missed this... :^(. 
To get this functionality back I add a PSake script. [PSake](https://github.com/psake/psake), if you haven't heard about it, is used for build automation similar to Nant and MSbuild scripts but with no nasty XML.
Initially I created 3 tasks `Compile`, `Deploy` and `New-Post`, a recent release of Sandra has added the ability to self host your site, so I added the `Preview` task.

+ `Compile` - simpily runs the Snow.exe with a path to my config file. 
+ `Deploy` - depends on `Compile` running then commits and pushes the output to github.
+ `New-Post` - is slightly more complicated. Firstly the full command is `psake .\generate.ps1 New-Post -parameters "@{ title = 'My Post Title' }"`.
This will take the title, convert it to lowercase replacing all spaces and punctuation with dashes (-). Then concatinates the title with that days date to generate
a filename along the lines of `yyyy-mm-dd-my-post-title.markdown`. The markdown file also includes a default header, this is the header for this post.

<pre><code>---
layout: post
title: Moving from Octopress to Sandra.Snow
date: 2014-01-21 06:07
comments: true
categories:
published: Private
---
</code></pre>

## Where Next

I would really like to convert the PSake script into a full blown PowerShell plugin/module and add a `New-Blog` command to setup a new default site.
