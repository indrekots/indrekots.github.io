---
layout: post
title: "Wrong version of Ant in path"
modified: 2013-04-18 18:26:17 +0300
tags: [ant, path, windows, java]
image:
  feature: 
  credit: 
  creditlink: 
comments: true
share: true
---

Recently, working on a Windows box, I had a problem where my `ANT_HOME` and `ant -version` result did not match. 
Problem resided in the order of path variables. `ANT_HOME` was declared last but when the system looks for a command in the path, it uses the first result it finds.