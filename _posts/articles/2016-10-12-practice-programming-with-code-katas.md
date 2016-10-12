---
layout: post
title: "Practice programming with code katas"
excerpt:
modified: 2016-10-12 14:05:22 +0200
categories: articles
tags: [kata, javascript, binary search, binary tree, algorithm, data structure]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: false
aging: true
---

Dave Thomas @PragDave proposes the idea of code katas.

> [Kata](https://en.wikipedia.org/wiki/Kata "Kata Wikipedia page") (Japanese for form or pattern) are an exercise where the novice repeatedly tries to emulate a master. In karate, these kata are a sequence of basic moves (kicks, blocks, punches, and so on), strung together in a way that makes sense. You’ll never be attacked in such a way that you could repeat a kata to defend yourself: that isn’t the idea. Instead the idea is to practice the feel and to internalize the moves. (Interestingly, kata are not just used in the martial arts. Calligraphers also learn using kata, copying their masters’ brush strokes.)

The idea is that just like in martial arts, we should practice programming just for the sake of writing code.

He has a website (codekata) which lists a number of example katas. I'm going to be looking the one which involves implementing a binary search algorithm (something you should be familiar with if you have attended an algorithms and data structures course) 5 days in a row. The caveat is that every day it should be implemented differently. At the same time you should observe what kind of common errors you encounter and do you improve over the period of 5 days.

## Binary search in short

## Specification

I needed to write a function that accepts an integer and a sorted array. It should return -1 if the integer is not present in the array. Otherwise it should return the index of the integer in the array.

## Day 1

The first day was easy because I had at least two techniques in mind—loops and recursion. I decided to implement a binary search algorithm with a while loop. The program would take the middle element of the array and at each iteration choose if the target element is to the left or to the right of the selected element, essentially halving the search space. The next iteration would only consider the sub-array that was either to the left or to the right of the initial middle element. This loop continues until the selected middle element of a sub-array is the target element or the sub-array is empty, in which case -1 is returned.

{% highlight javascript %}
function chop(int, array) {
  var l = 0;
  var r = array.length - 1;

  while (l <= r) {
    var m = Math.floor((l+r)/2);
    if (array[m] < int) {
      l = m + 1;
    }
    else if (array[m] > int) {
      r = m - 1;
    }
    else {
      return m;
    }
  }

  return -1;
}
{% endhighlight %}

Beware of infinite loops. It's easy to use a wrong boolean expression for the while loop. Additionally off by one errors are common I guess.
