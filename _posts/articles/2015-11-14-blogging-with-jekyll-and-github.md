---
layout: post
title: "Blogging with Jekyll and Github"
excerpt: It is possible to host your blog at Github and version control your blog posts with Git. Here's how.
modified: 2015-11-14 19:55:19 +0300
categories: articles
tags: [jekyll, github, git, blog, blogging, markdown, ruby]
image:
  feature: 2015-11-14-blogging-with-jekyll-and-github/cover.jpg
  credit:
  creditlink:
comments: true
share: true
published: false
---

Starting a blog is easy. Find a blogging service provider or host it yourself. But did you know you can use Github to host your blog and make use of Git to version control your blog posts.

Github Pages helps you host a static webpage for your project. Technically you can host any static website there, including your blog. But static you say? That's so 90's. How can I edit my blog post? Write it in HTML?

##Introducing Jekyll

Jekyll is a static site generator designed for blogs and written in Ruby. You can edit your content in markdown and let Jekyll generate HTML pages from it. This is ideal if you're only producing static content. No need to set up a database and no need to install updates

##Version control with Git

Since Github Pages are regular Git repositories, all you're content is version controlled. Creating or editing a post includes writing Markdown (or other supported formats) and committing them to your repository.

##Writing in Markdown

Blog posts can are written in Markdown. Each markdown file has a header which includes some metadata about the post (e.g.title, excerpt, published status).  

##Having your own domain
