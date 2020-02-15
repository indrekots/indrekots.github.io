---
layout: post
title: "Java 14 Records"
excerpt: "Java 14 introduces a new preview feature to the Java language—Records. It's goal is to provide a compact syntax to declare containers of shallowly immutable data."
modified: 2020-02-15 18:56:18 +0300
categories: articles
tags: [java, records]
image:
  path: /images/2020-02-15-java-14-records/cover.jpg
  thumbnail: /images/2020-02-15-java-14-records/cover_thumb.jpg
  caption: "Photo by [Steve Harvey](https://unsplash.com/photos/LUMT0EeNnjE)"
comments: true
share: true
published: true
aging: false
---

Java 14 introduces a [new preview feature](https://openjdk.java.net/jeps/359 "JEP 359: Records (Preview)") to the Java language—Records.
It's goal is to provide a compact syntax to declare containers of shallowly immutable data.

## Records in Action

It's common to declare classes in Java that hold only data and no behavior.
Sometimes they're referred to as data transfer objects or value objects.
For example, if we would like to represent a point on a Cartesian plane, we could create the following class.

```java
class Point {
    private double x;
    private double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    // accessors
    // toString
    // equals and hashCode
}
```

To express a pair of coordinates on a Cartesian plane, we have to create a new class, declare a constructor, provide accessors and also write `toString`, `equals` and `hashCode` methods.
That's a lot of code where the only important bits are the x and y coordinates.
Modern IDEs can generate this code for us.
Users of [Lombok](https://projectlombok.org/ "Project Lombok") are well aware of the `@Data` and `@Value` annotations that generate the boilerplate code during build time.

Records, on the other hand, provide a syntax to declare only the parts that are important for the reader.

```java
record Point(double x, double y) {}
```

No need to declare a constructor and worry about whether `equals` and `hashCode` are implemented.
That's all done for you.

## It's not only about boilerplate

- data container, immutable
- modelling data as data
