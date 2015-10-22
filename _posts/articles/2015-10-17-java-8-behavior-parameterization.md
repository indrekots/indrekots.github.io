---
layout: post
title: "Java 8: Behavior parameterization - overcoming verbosity"
excerpt:
modified: 2015-10-17 21:55:09 +0300
categories: articles
tags: [java8, java]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: false
---

Behavior parameterization is essentially taking a block of code and making it available without executing it. It can be passed to a method for example. Since Java 8 introduced lambdas (finally), it is now possible to parameterize method's behavior. If you're familiar with Javascript, Scala, Groovy or any other language that has lambdas built in, then you probably have been using behavior parameterization all along.

It might not come to you as a surprise that in software development user requirements can change. Passing behavior as a parameter can help relieve the pain of change.

##Example domain

Let's look at an example of filtering Java objects. More specifically, I'm going to use Java 7 to filter a list of `book` objects without using any external libraries. The book class has 2 fields: `name` and `pageCount`. Imagine we have a library application and according to requirements it should be possible to find books that have more than 200 pages.

{% highlight java %}
public class Book {
    private String name;
    private int pageCount;

    //getters and setters omitted...
}
{% endhighlight %}

The easiest way to find long novels (in our case a long novel is a book with more than 200 pages) is to loop over the list of books, use an if clause to check if it has more than the specified amount of pages, add the book to the result list and finally return the result list.

{% highlight java %}
public static List<Book> findLongNovels(List<Book> books) {
    List<Book> result = new ArrayList<>();
    for (Book book : books) {
        if (book.getPageCount() > 200) {
            result.add(book);
        }
    }
    return result;
}
{% endhighlight %}

Everything's fine and dandy, right? As is customary, requirements change and new requirements are added. Now the library application should be able to filter books by author. Pretty simple to accomplish. Just use the same general layout as before.

{% highlight java %}
public static List<Book> filterBooksByAuthor(List<Book> books, String author) {
    List<Book> result = new ArrayList<>();
    for (Book book : books) {
        if (author.equals(book.getAuthor())) {
            result.add(book);
        }
    }
    return result;
}
{% endhighlight %}

If you compare this to the previous `findLongNovels` method, you can clearly see that they're very similar. This is a [WET](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself "Wikipedia DRY vs WET") solution. Let's [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself "Wikipedia DRY vs WET") it up. The overall structure is the same. Code iterates over a list of books and applies a filtering clause. The goal is to keep the iteration and filtering separate. When using Java 7, we could create a `BookPredicate` interface which could define the logic for filtering. A [predicate](https://en.wikipedia.org/wiki/Predicate_(mathematical_logic) "Wikipedia: Predicate (mathematical logic)") is essentially a boolean-valued function. Since Java 7 does not have lambdas, we're going to wrap the predicate in a class.

{% highlight java %}
public interface BookPredicate {
    boolean test(Book book);
}
{% endhighlight %}

Filtering logic can be moved to a separate class which implements the `BookPredicate` interface.

{% highlight java %}
public class LengthPredicate implements BookPredicate {
    private int length;

    public LengthPredicate(int length) {
        this.length = length;
    }

    @Override
    public boolean test(Book book) {
        return book.getPageCount() > length;
    }
}

public class AuthorPredicate implements BookPredicate {
    private String author;

    public AuthorPredicate(String author) {
        this.author = author;
    }

    @Override
    public boolean test(Book book) {
        return author.equals(book.getAuthor());
    }
}
{% endhighlight %}

After doing some refactoring, it is possible to reuse the method that iterates over the list of books.

{% highlight java %}
public static List<Book> filterBooks(List<Book> books, BookPredicate p) {
    List<Book> result = new ArrayList<>();
    for (Book book : books) {
        if (p.test(book)) {
            result.add(book);
        }
    }
    return result;
}

//somewhere in main
BookPredicate lengthPredicate = new LengthPredicate(200);
BookPredicate authorPredicate = new AuthorPredicate("Lewis Carrol");
filterBooks(books, lengthPredicate);
filterBooks(books, authorPredicate);
{% endhighlight %}

Now we're not repeating ourselves but hey, that's a lot of code to write. As they say, **Java is verbose**. Initially there were 2 methods that filtered books. That's about 16 lines of code. After removing duplicate code and moving filtering logic to separate classes there's about 33 lines. Although this is not much for a small project, with a large project the lines add up. Is there anything that can be done to write more concise code?

##anonymous inner classes

##lambdaj

##Filter with loop and if clause
###show boilerplate code

##Introduce predicates, filter clause in a separate Object
###Multiple implementations
###anonymous classes

##Replace predicate function with a lambda

##Use streams and filter method (more functional style)
###parallelization, no loops

##useful for flexible apis
