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
A [version control system](https://en.wikipedia.org/wiki/Version_control "Version Control") is an essential collaboration tool used in software engineering. I think for most developers, it is a tool for sharing
work in a collaborative manner. For the most part, this is correct. But VCSs also play an important role—something
that cannot be understated—in tracking the history of changes in a codebase. This leads me to the topic of this
post—commit messages and how to write a good one.

## How to write good commit messages
- what you should write
- 50/72 rule
- git tooling benefiting from 50/72 rule
- bad examples
- good examples

To understand what's a good commit message, let's start with a few examples of bad ones.
I dug through some old git repositories and found some gems that I have written.
I'm sure you've seen similar messages.

* started working on feature X
* some testing stuff
* little fix
* TICKET-66
* very big commit but a short commit message

All of them would receive a high ranking in [the commit messages hall of shame](http://www.codelord.net/2015/03/16/bad-commit-messages-hall-of-shame/ "Bad Commit Messages Hall of Shame").
They're vague and don't say anything about why the particular change was introduced. Looking at them now,
they're almost as good as having no commit messages at all. Let's turn our attention to what to look for
in a good commit message.

## The story of a good commit message

To understand how to write a good commit message, let's put ourselves in the place of a maintainer or
a future developer of the project. After all, we should write commit messages not for our pleasure but
for others to read.

Imagine yourself as a developer who is tasked to maintain a relatively large codebase. You need to hunt down
a bug and while doing that, you come across a line of code that looks interesting, to say the least. You start to wonder
why the original author implemented it like that. Maybe there was a very specific and rational reason.
Or maybe, just maybe, this is the source of that dreaded bug that you've been trying to catch for hours.

As a curious code detective, you whip out your handy Swiss Army knife—Git—like MacGyver and proceed
to unravel the mystery. Using `git blame` you try to understand who was the author of the line of code in question
and what was the original intent. After some time spent on [reading up on how `git blame` works](https://git-scm.com/docs/git-blame "Git blame documentation") you read the commit message
of the revision the line of code was part of.

If this were a *choose your own adventure* book, you would be presented with the following options:

1. The commit message is vague. You look at other changes in the same commit in hopes of understanding
what the original author tried to achieve but they're all over the place. It seems that the original author
bundled many logical changes into one commit.

2. The commit message is vague. Luckily the commit is fairly small with only 2 files being changed in total.
Although the message is not useful, other changes in the commit help you deduce the original intent of the
author.

3. The commit message is well structured and describes clearly the intent of the author. The message
includes why the change was introduced and also explains the peculiarities of the line of code that prompted you to take this journey.

If you're like me, you'd choose the third option. The main takeaway is that commit messages should be
written for other developers—or for the future you—so they could understand you and why you introduced these changes. Writing code is a team effort. Software projects can last years and see many developers joining and leaving. A lot of times it is impossible for a new developer to go and ask a previous developer what was on his/her mind when a piece of code was written. Good commit messages can help here. They're like a trail of breadcrumbs you leave behind for future developers who might stumble on the same twisted path you once took; hopefully leading them out of the dark forest.

//Commit history is a collaborative journal.  
//saves time to review/re-establish context

## What makes up a good commit message?

Hopefully by now I have convinced you of the importance of good commit messages. Without further ado, let's move on and look at what a commit message should contain.  

Overall, a commit message should explain what was changed and why. There's no need for a detailed description on how a patch was implemented. That's what source code is for. It's also good to compare the current implementation to how things were before the change. This usually gives enough context so the reader can understand why the change was necessary.

[Peter Hutterer writes](https://who-t.blogspot.com.ee/2009/12/on-commit-messages.html "On commit messages") that a good commit message should answer three questions about a patch:

* **Why is it necessary?** It may fix a bug, it may add a feature, it may improve performance, reliabilty, stability, or just be a change for the sake of correctness.

* **How does it address the issue?** For short obvious patches this part can be omitted, but it should be a high level description of what the approach was.

* **What effects does the patch have?** (In addition to the obvious ones, this may include benchmarks, side effects, etc.)

Put yourself in the position of a future developer or maintainer and try to think of what they should know about your patch. In addition, try to avoid large commits. Checking in a day's worth of work at the end of the day is usually not a good practice. If you don't know what to write, your commit might be too large. [Try to keep your commits atomic](https://www.freshconsulting.com/atomic-commits/).

## Technical details of a commit message

- 50/72 rule
- blank line
- present tense

## Benefits of good commit messages
- speeds up code reviews
- project maintainability, you can understand why a developer has done the given change
- possible source for release notes

## Summary

I like how Peter Hutterer put in a blog post https://who-t.blogspot.com.ee/2009/12/on-commit-messages.html
> A commit message shows whether a developer is a good collaborator.

## Rules on how to write

* summary 50 chars, then blank line, this is useful if you use git tools, git log --oneline for example. If it is difficult to use no more than 50 chars, could it be that your commiting too much in a single commit? Keeping commits fairly small benefits the reviewer of the commit and the ones who might be interested later to understand what this particular commit did, why was it introduced. Having too much going on in a single commit, is difficult to understand, even though you can see everything that had been changed.

Begin the summary line with a Capital letter and do not end it with a period. This can be considered a title. You would not want to have a heading with a small letter and ending with a period, do you?

Summary is also used when sending patches with e-mails, then the summary becomes the email subject.

Use imperative style for commit summary, e.g. merge..., add something, fix something else..., Git's generated commits use it, so should you. Think of the subject line not as what you did, but what this commit will to if this is applied to my history. E.g. this commit will *add support for markdown*. When reviewing a pull request or merging branches, you can have a look at the summary of the commits and read them as follows: If I accept commit bc43a2d, then this will *remove unused variables in controller*. The same is true if you're cherry picking commits. This would not work if the commit summary was written as something that has happened in the past.

wrap commit body lines at the 72 char mark. Git commands add 4 chars of padding, so when you follow the 72 char rule, commits can be easily viewed in a 80char wide terminal window.

Commit body should include, what what changed and why this change was introduced (my boss told me to do it is not a reason). It's not usually needed to explain how the patch was implemented. Hopefully your code is self explanatory, if not, read clean code. Contrast the current implementation with the way thing were before your commit and what was wrong with the previous implementation (e.g. this caused an unexpected exception when doing ... ).

## How
''
First line is your title, it can include your ticket name as well (50 chars) https://git-scm.com/book/en/v2/Distributed-Git-Contributing-to-a-Project. Then leave a blank line and starting from the third line, you can write additional text.

// show examples how this is displayed in git log

50/72 rule https://medium.com/@preslavrachev/what-s-with-the-50-72-rule-8a906f61f09c
from git commit manpage -> subject 50 chars
super small commits probably do not need a body, because they're self explanatory

write good commit message with a proper text editor.

## Other benefits

* Basis for release notes
* Speed up code reviews

If it seems that it is difficult to summarize your commit, maybe your commit is too big?

http://www.commitlogsfromlastnight.com/

Don't be too verbose
Writing a good commit summary is difficult. Keeping it short and meaningful can be a challenge. It's like naming a variable, you don't want it to be too long, but you want to convey a meaning that's easy to understand.

A diff will show you what changed, a commit message will tell you why this change was done.

Good commit messages contribute to the overall project log. Writing good commit messages makes it easier to decipher why a change was introduced in the first place (hopefully) a month/year after it was introduced.

As developers, we agree that the projects code should follow a similar style. Consistency is better than having each participant write using their own style. https://chris.beams.io/posts/git-commit/.

Commit log should be the same. Commit log can be a valuable project artifact, so let's treat it as we treat our code.

// show good commit messages from open source projects, and bad ones.

Pro git, read it, it's good stuff.
http://whatthecommit.com/
