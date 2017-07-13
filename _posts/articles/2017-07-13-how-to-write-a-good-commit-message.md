---
layout: post
title: "How to write a good commit message"
excerpt:
modified: 2017-07-13 19:47:17 +0300
categories: articles
tags: [vcs, git, version control]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: false
aging: false
---

* Commit message, what it is and why you should bother
* Put yourself in place of a maintainer/future developer
  * you see a line of code and start to wonder why this was implemented or why it is implemented like that
  * One of tree things can happen, 1) firstly, you see a vague commit message, you look other changes in the same
  commit and they're all over the place, making it even harder to figure out what the original committer tried to achieve. 2) Secondly, you see a small commit, with a vague message. Although the message is not useful, other changes in the commit might help you to deduce the original intent. 3) Thirdly, and this is probably the best scenario, you see a small commit with a well structured and clearly written commit message.
  This will hopefully make it super clear why something was done the way it was.
* Bad examples: short descriptions, only ticket numbers, big commit but a single line commit message, you've probably seen them. Now imagine you're about to review one of these commits. E.g. "fixed small bug" or "work on FEATURE-123" http://www.codelord.net/2015/03/16/bad-commit-messages-hall-of-shame/

Writing code is a team effort. Software projects can last years and see many developers joining and leaving. A lot of times it is impossible for a new developer to go and ask a previous developer what was on his/her mind when a piece of code was written. Good commit messages can help here. They're like a trail of breadcrumbs you leave behind for future developers who might stumble on the same twisted path you once took; hopefully leading them to clarity.

I like how Peter Hutterer put in a blog post https://who-t.blogspot.com.ee/2009/12/on-commit-messages.html
> A commit message shows whether a developer is a good collaborator.

## What should I write in a commit message?

First and foremost, I think the most important piece of information is why this commit was introduced. Describe your motivation for the change. For example, a commit might add a new feature, fix a bug, improve readability etc.

Secondly, for commits that might be difficult to follow, you can give a brief overview what approach was taken to implement a feature, fix a bug etc. Here it's useful to contrast your implementation with the previous behavior.

Other useful info, e.g. side effects that might not be obvious, performance related ...

Put yourself in the position of a future developer and try to think what they should know about your change.

## How

First line is your title, it can include your ticket name as well (50 chars) https://git-scm.com/book/en/v2/Distributed-Git-Contributing-to-a-Project. Then leave a blank line and starting from the third line, you can write additional text.

// show examples how this is displayed in git log

## Other benefits

* Basis for release notes
* Speed up code reviews

If it seems that it is difficult to summarize your commit, maybe your commit is too big?

http://www.commitlogsfromlastnight.com/
