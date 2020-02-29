---
layout: post
title: "Helpful NullPointerExceptions in Java 14"
excerpt: "Helpful NullPointerExceptions in Java 14 improve NPE error messages by describing precisely which variable was null."
modified: 2020-02-29 18:56:18 +0300
categories: articles
tags: [java, npe]
image:
  path: /images/2020-02-26-java-14-helpful-nullpointerexceptions/cover.jpg
  thumbnail: /images/2020-02-26-java-14-helpful-nullpointerexceptions/cover_thumb.jpg
  caption: "Photo by [Jonas Jacobsson](https://unsplash.com/photos/MLSS52p0ze4)"
comments: true
share: true
published: true
aging: false
---



Being [the most common exception in Java](https://blog.overops.com/the-top-10-exceptions-types-in-production-java-applications-based-on-1b-events/ "The Top 10 Exception Types in Production Java Applications"), the infamous `NullPointerException` is probably the first exception a new Java developer experiences.
To offer helpful information to developers and improve the usability of the language, Java 14 [introduces improved `NullPointerException`](https://openjdk.java.net/jeps/358 "JEP 358: Helpful NullPointerExceptions") messages which point out precisely the variable that was `null`.

## NPEs before Java 14

Let's say we have the following classes.

```java
class Person {
  Name name;
}

class Name {
  String firstName;
  String lastName;
}
```

A `NullPointerException` occurs when we try to dereference a variable that points to nothing.

```java
Person person = null;
System.out.println(person.name);

// Exception in thread "main" java.lang.NullPointerException
//   at mypackage.MyClass.main(MyClass.java:7)
```

Reading the error message, it's fairly obvious that `person` is `null`.
However, when the code is more complex, the error message is not that helpful anymore.
Imagine we get a reference to a `Person` object and then try to access its fields.

```java
var person = getPersonFromDatabase();
System.out.println(person.name.firstName);

// Exception in thread "main" java.lang.NullPointerException
//   at mypackage.MyClass.main(MyClass.java:7)
```

From the error message we can infer that either `person` or `name` was `null`.
If the issue is reproducible and we have access to the source code we could debug the problem by attaching a debugger or creating intermediate local variables.
However, when the error is found from logs from a production server, we don't have a clear indication which variable was `null`.

## New and improved NPE error messages

Java 14 introduces [Helpful NullPointerExceptions](https://openjdk.java.net/jeps/358 "JEP 358: Helpful NullPointerExceptions") that improve NPE error messages by describing precisely which variable was `null`.
If we were to run the previous example in Java 14, we would get a different error message.

```java
var person = getPersonFromDatabase();
System.out.println(person.name.firstName);

// Exception in thread "main" java.lang.NullPointerException: Cannot read field "firstName" because "person.name" is null
//   at mypackage.MyClass.main(MyClass.java:7)
```

This time, the error message is more usable and there's no need to hunt down which variable is `null`.

## Summary

By pointing out which variable was `null` in the error message greatly improves the usability of the language.
Keep in mind that in Java 14, helpful `NullPointerException` messages are turned off by default.
You need to start the JVM with the `-XX:+ShowCodeDetailsInExceptionMessages` command line flag.
Helpful error messages will be enabled by default in Java 15.
