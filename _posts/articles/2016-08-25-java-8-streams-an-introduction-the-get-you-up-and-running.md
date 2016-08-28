---
layout: post
title: "Java 8 Streams: an introduction the get you up and running"
excerpt:
modified: 2016-08-25 14:39:19 +0300
categories: articles
tags: [java, streams]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: true
aging: true
---

Even the simplest programs use some kind of collection of elements. They are an essential part in programming. Be it arrays, lists, dictionaries or maps, they're used to store data so it could be easily accessed. Depending on the use case, different data structures are chosen. Arrays are good for storing a sequence of elements. Key-value data can be stored in dictionaries or maps (some programming languages call them associative arrays). Up until Java 8, processing Collections in Java has been inconvenient.

I must admit that some operations are relatively simple to achieve but if you're faced with a task to filter and then group data, you're probably going to write several lines of code with multiple intermediary data containers. It would be better to declaratively define what operations you want to be applied to the collection. Think of SQL. You can select, filter and group data from a table declaratively. Java designers introduced the Streams API which allows to manipulate collections in a declarative and functional manner.

## Streams

Streams are a new API in Java 8 which allows developers to declaratively manipulate collections of data. It promotes the usage of functional style programming. Together with lambda expressions, Streams make the code more concise and readable. Additionally Streams allow to pipe multiple operations one after another. If you're familiar with Unix command-line pipes then you might find composing stream operations simple to understand. All in all, it helps the code to be more composable.

I think the best way to demonstrate the conciseness of the Streams API is to show the following comparison. The first snippet uses Java 7 and the next uses Java 8 Streams to accomplish the same task.

{% highlight java %}
List<Book> library = new ArrayList<>();
//initialize a library of books
List<Book> library = new ArrayList<>();
library.add(new Book("Alice's Adventures in Wonderland", 300, "Lewis Carrol"));
library.add(new Book("Through the Looking-Glass, and What Alice Found There", 228, "Lewis Carrol"));
library.add(new Book("The War of the Worlds", 192, "H.G. Wells"));
library.add(new Book("1984", 268, "George Orwell"));
library.add(new Book("Animal Farm", 102, "George Orwell"));
library.add(new Book("The Neverending Story", 396, "Michael Ende"));

//group books by author using Java 7
Map<String, List<Book>> byAuthor = new HashMap<>();
for (Book book : library) {
    if (byAuthor.get(book.getAuthor()) == null) {
        List<Book> books = new ArrayList<>();
        books.add(book);
        byAuthor.put(book.getAuthor(), books);
    }
    else {
        List<Book> books = byAuthor.get(book.getAuthor());
        books.add(book);  
    }
}

System.out.println(byAuthor);

//group books by author using Java 8 Streams API
Map<String, List<Book>> byAuthor2 = library.stream().
        collect(groupingBy(Book::getAuthor));

System.out.println(byAuthor2);

//prints out:
//{Michael Ende=[Name: The Neverending Story, page count: 396, author: Michael Ende], H.G. Wells=[Name: The War of the Worlds, page count: 192, author: H.G. Wells], George Orwell=[Name: 1984, page count: 268, author: George Orwell, Name: Animal Farm, page count: 102, author: George Orwell], Lewis Carrol=[Name: Alice's Adventures in Wonderland, page count: 300, author: Lewis Carrol, Name: Through the Looking-Glass, and What Alice Found There, page count: 228, author: Lewis Carrol]}
//{Michael Ende=[Name: The Neverending Story, page count: 396, author: Michael Ende], H.G. Wells=[Name: The War of the Worlds, page count: 192, author: H.G. Wells], George Orwell=[Name: 1984, page count: 268, author: George Orwell, Name: Animal Farm, page count: 102, author: George Orwell], Lewis Carrol=[Name: Alice's Adventures in Wonderland, page count: 300, author: Lewis Carrol, Name: Through the Looking-Glass, and What Alice Found There, page count: 228, author: Lewis Carrol]}
{% endhighlight %}

Although they produce the same result (they group books by authors), using Streams is easier to understand and required less work from the developer. Don't worry if you don't understand the code. I'll go over the basics in the following paragraphs.

## Creating a stream

creating a stream from collections, streams from values, streams from arrays, streams from files, streams from functions

## Stream operators

intermediate operations vs terminal operations
data source -> intermediate operations -> terminal operation

## Examples of Stream operators

functional -> map, reduce, filter, other stream usages
truncating, limit()
skipping, skip()
finding
max min

## Pipelining

simple diagram of stream pipelining

## Specialized Streams

numeric streams, specialized streams (intStream)

## Parallel Streams

## Other libraries

Guava, apache, lambdaj

## Other

traversable only once
lazy

## Summary
