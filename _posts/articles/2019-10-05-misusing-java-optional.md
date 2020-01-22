---
layout: post
title: "Misusing Java's Optional type"
excerpt: "Java’s Optional type has been criticised for its flaws. It’s also being used for things it was perhaps not designed. This post shows some situations where I think Optional is being misused."
modified: 2019-10-05 18:56:18 +0300
categories: articles
tags: [java, optional, null reference, lambda]
image:
  path: /images/2019-08-12-misusing-java-optional/cover.jpg
  thumbnail: /images/2019-08-12-misusing-java-optional/cover_thumb.jpg
  caption: "Photo by [wu yi](https://unsplash.com/photos/mk7zhx5lFbc)"
comments: true
share: true
published: true
aging: false
amazon_links: true
---

[Optionals were introduced to Java](https://blog.indrek.io/articles/optionals-in-java-8/ "Optionals in Java 8") in version 8.
Prior to it, when we had no value to return from a method, we could for example return a `null` or throw an exception.
Both options have their drawbacks.
If the calling code forgets to check the return value for `null`, there's a high chance for a `NullPointerException` to happen down the line.
Throwing an exception also does not feel right.
The fact that a value is missing isn't always an exceptional situation.

Java's Optional has been [criticised](https://blog.developer.atlassian.com/optional-broken/) for its [flaws](https://medium.com/@bgourlie/java-8-s-new-optional-type-is-worthless-448a00fa672d) but despite that I personally find them tremendously useful.
Over the years, however, I've also seen code where I think the `Optional` type has been misused.
The following is a list of misuses of `Optional` and what you can do to avoid them.

## 1. Returning `null` from an Optional-returning method

When you see that a method returns an `Optional` of something, this is a clear indication that the method signature is trying to tell you that [the return value might be missing](https://blog.codefx.org/techniques/intention-revealing-code-java-8-optional/).
The good thing about Optionals is that they force you to handle the special case where there might be no value.
Since Optionals are designed so that we don't have to explicitly deal with `null` references anymore, returning a `null` from an Optional-returning method will definitely throw somebody off guard.
Most of us don't expect the `Optional` container itself to be `null`.
Returning a `null` from an Optional-returning method means that we have to handle two special cases - `null` reference and empty `Optional`.

In [Effective Java](https://amzn.to/31uiEnG "Effective Java 3rd Edition"), the author [Joshua Bloch](https://twitter.com/joshbloch) writes
> Never return a null value from an Optional-returning method: it defeats the entire purpose of the facility.

[Stuart marks](https://twitter.com/stuartmarks) gave a presentation about Optionals for [vJUG](https://virtualjug.com/) where he emphasised to never, ever, use `null` for an `Optional` variable or return value.

<figure class="align-center">
  <img src="{{ '/images/2019-08-12-misusing-java-optional/optional-null.png' | absolute_url }}" alt="">
  <figcaption>Slide from a <a href="https://virtualjug.com/">vJUG</a> presentation <a href="https://www.youtube.com/watch?v=4f9j0LtULiM">"Optional - The Mother of All Bikesheds by Stuart Marks"</a></figcaption>
</figure>

Make sure that the method responsible for returning the `Optional` *never* returns `null`.
Make use of `Optional.empty()` instead.

## 2. Checking whether an Optional is null

Checking whether a variable of type `Optional` is `null` is related to the previous misuse.
If we write code that ensures an `Optional` can never be `null`, then there's no need to check for `null` references.
It saves us from cluttering our code with unnecessary `null` checks.
If, however, you find yourself in a situation where an `Optional` for some reason is `null`, refer back to the previous paragraph.

Handling two special cases—`null` reference or an empty `Optional`—defeats the entire purpose of using Optionals.
To put it differently, adding an extra layer of indirection in the form of an `Optional` container but reaping none of its benefits is worse compared to not using Optionals at all.

If you use [SonarQube](https://www.sonarqube.org/) or have the [SonarLint](https://www.sonarlint.org/) plugin for your IDE, checking an Optional variable for `null` results in the following bug report.
> The concept of Optional is that it will be used when `null` could cause errors. In a way, it replaces `null`, and when Optional is in use, there should never be a question of returning or receiving `null` from a call.

To conclude, make sure a variable whose type is `Optional` is *never* `null`.

## 3. Using Optional as a method parameter

Java language doesn't have [optional method parameters](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/named-and-optional-arguments#optional-arguments) but one might think that we could implement it using `Optional`.
Let's have a look at the following method declaration.

```java
public void greet(Optional<String> name) {
  System.out.println("Hello " + name.orElse("world"));
}
```

Seems nice and succinct, right?
Well, the method body is a beautiful one-liner but let's quickly look at how callers should invoke the `greet` method.

```java
greet(Optional.of("neighbor"));
// prints "Hello neighbor"

greet(Optional.empty());
// prints "Hello world"
```

Callers must always wrap every argument with an `Optional` or pass an empty `Optional`.
A simpler approach would be to use overloaded methods instead.

```java
public void greet(String name) {
  System.out.println("Hello " + name);
}

public void greet() {
  greet("world");
}
```

Callers can simply pass in the string without having to do any additional wrapping.

```java
greet("neighbor");
// prints "Hello heighbor"
greet();
// prints "Hello world"
```

But let's say we were not trying to implement optional parameters.
We just have an `Optional` at hand and want to pass it to another method for further processing, would that be okay?

```java
Optional<Customer> customer = ... // get a customer object
process(customer);
```

There's nothing stopping you from doing that but perhaps we can do better?
If we think about, what would happen inside the `process` method?
Most likely it would try to unwrap the `Optional` and do something with the underlying `Customer`.
If that's the case, a more elegant solution would be to make use of the `ifPresent` method on the `Optional` class and pass it a [lambda expression]({{site.url}}/articles/java-8-lambda-expressions/ "Java 8: Lambda expressions") or a [method reference]({{site.url}}/articles/four-types-of-method-references-in-java-8/ "Four types of method references in Java 8").

```java
Optional<Customer> customer = ...
customer.ifPresent(this::process);
```

Java Language Architect Brian Goetz had the following to say [in an answer to a question about Optionals on Stack Overflow](https://stackoverflow.com/a/26328555/2928051).
> You should almost never use it as a field of something or a method parameter.

## 4. Getting a value from Optional without making sure it's present

Getting the value from the Optional container seems trivial.
Just use the `get()` method.
[But if we look more closely at the documentation](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html#get()), we can see that it throws a `NoSuchElementException` when the `Optional` is empty.
By just reading the method name it seemed like a safe operation but it's not.
Before calling `get()`, we *must* make sure the Optional has a value inside.

Brian Goetz wrote the following in a popular [Stack Overflow answer](https://stackoverflow.com/a/26328555/2928051).
> NEVER call `Optional.get` unless you can prove it will never be `null`; instead use one of the safe methods like `orElse` or `ifPresent`. In retrospect, we should have called get something like `getOrElseThrowNoSuchElementException` or something that made it far clearer that this was a highly dangerous method that undermined the whole purpose of Optional in the first place. Lesson learned.

There were even talks about deprecating `Optional::get` in [OpenJDK mailing list](http://mail.openjdk.java.net/pipermail/core-libs-dev/2016-April/040484.html) but the idea was not well received.
Java 10 introduced a new overloaded method for `Optional::orElseThrow` without any parameters.
The behaviour of this method is the same as for `Optional::get` but the name is more appropriate.

Therefore, before calling `get` you must prove that an Optional has a value with `isPresent`.

```java
Optional<String> optional = ...
if(optional.isPresent()) {
  String str = optional.get();
}
```
On the other hand, this approach is hardly an improvement compared to checking for nulls.
Compare it with the following code.

```java
String str = ...
if (str != null) {
  // do something with str
}
```

The structure is the same.
We first check, then act.
The problem with `Optional::get` is that it leads us to write the same style of code that we wrote when we explicitly checked for null references.
That invites an obvious question, what's the benefit of Optionals then?

Instead of using `Optional::get`, most of the time we can create a more elegant solution by using alternative methods on the `Optional` class (`map`, `filter`, `ifPresent`, `orElse`).
To put it differently, if possible, try to avoid using `isPresent`.

## 5. `orElse(null)`

`orElse` method on the `Optional` class can be used to provide a fallback value when the `Optional` is empty.
In some situations you might want to continue working with `null` references.
Let's say that you already have some code or use a library that expects to receive a nullable reference and can deal with the `null` case.
In these situations we can let the fallback value be `null`.

```java
// findUserById returns an Optional<User>
User user = findUserById(1L).orElse(null);
```

Other than that, I would try to avoid `orElse(null)`.
Usually there's a more elegant way to solve the problem.
Imagine that you used `orElse(null)` and now have a reference to a `User` object that might be null.
Whenever you start to call any methods on the user, you would first have to check that the reference is not null.

```java
User user = findUserById(1L).orElse(null);

if (user != null) {
  // do something with the user
}
```

In my humble opinion, using `ifPresent` in this case would be better.

```java
Optional<User> user = findUserById(1L));
user.ifPresent(/* lambda expression or method reference */);
```

## 6. Collections wrapped with an Optional

We've learned so far that Optionals can be used to model the absence of a result from a computation.
Let's say we have a method that returns a `List` of `User` objects from a database.
Let's also introduce a limitation that in some situations the database might not contain any users.
One might think that this special case can be modelled with an Optional.

```java
public Optional<List<User>> fetchUsersFromDatabase() {
  ...
}
```

Callers of the method can see from the return type that the value might be missing and can handle the special case.
*Pretty nifty, eh?*
No.
Not really.
Don't do that!

This is an example of going too far with Optionals.
Absence of values can also be represented with an empty collection.
Introducing Optionals to the mix only complicates things.
What you would end up is a container inside a container, like a [Matryoshka doll](https://en.wikipedia.org/wiki/Matryoshka_doll).

<figure class="align-center">
  <img src="{{ '/images/2019-08-12-misusing-java-optional/russian-doll.jpg' | absolute_url }}" alt="">
  <figcaption>Container inside another container. Image by <a href="https://pixabay.com/photos/russian-dolls-babuschka-matruschka-1861410/">Monika Schröder</a></figcaption>
</figure>

The book [Effective Java](https://amzn.to/31uiEnG "Effective Java 3rd Edition") mentions the following:
> Container types, including collections, maps, streams, arrays, and optionals should not be wrapped in optionals.

## 7. Optional as an element in a collection

We just looked at that there's no real benefit in wrapping a collection type with an `Optional`.
What about the other way around?
What if we have a collection of Optional types?

```java
List<Optional<Cat>> users = ... // fetch cats
```

When we have a list of cat objects, most likely we're going to iterate trough them and do something with each element.
But if each element is an `Optional`, we would first have to unwrap it and process it only when it is present.
A much simpler approach would be to not include Optionals in collection types at all and keep our collections clean of [Schrödinger's cats](https://en.wikipedia.org/wiki/Schr%C3%B6dinger%27s_cat)—cats that are simultaneously both there and not until we check for their presence.

<figure class="align-center">
  <img src="{{ '/images/2019-08-12-misusing-java-optional/cat.jpg' | absolute_url }}" alt="">
  <figcaption>Schrödinger's cat. Image by <a href="https://pixabay.com/photos/physics-schr%C3%B6dinger-s-cat-3864569/">Gerd Altmann</a></figcaption>
</figure>

I'm going to reference [Effective Java](https://amzn.to/31uiEnG "Effective Java 3rd Edition") for the last time in this post.
> ... it is almost never appropriate to use an optional as a key, value, or element in a collection or array

## Summary

The introduction of Optionals to Java [got some flak from the community](https://blog.developer.atlassian.com/optional-broken/).
[Some might think that `Optional` is a first-class option type](https://medium.com/@bgourlie/java-8-s-new-optional-type-is-worthless-448a00fa672d) integrated into the language.
It does not help that [during the design phase](https://blog.codefx.org/java/dev/design-optional) there were different opinions about what `Optional` should and shouldn't be.

[From JavaSE 11 documentation](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html) we can read that `Optional` is primarily intended for use as a method return type. In addition, Java language architect Brian Goetz wrote the following in a [Stack Overflow answer](https://stackoverflow.com/a/26328555/2928051).
> Of course, people will do what they want. But we did have a clear intention when adding this feature, and it was not to be a general purpose Maybe type, as much as many people would have liked us to do so. Our intention was to provide a limited mechanism for library method return types where there needed to be a clear way to represent "no result", and using null for such was overwhelmingly likely to cause errors.

The main takeaway form this post should be that Optionals don't replace nulls and [don't provide 100% null safety](https://www.codeproject.com/Articles/787668/Why-We-Should-Love-null).
Going far and beyond, trying to avoid nulls by using Optionals in places where it was not designed to be used could result in some cumbersome looking code.
Your milage may vary though.
