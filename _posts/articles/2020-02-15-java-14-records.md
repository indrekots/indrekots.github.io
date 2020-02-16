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
amazon_links: true
---

Java 14 introduces a [new preview feature](https://openjdk.java.net/jeps/359 "JEP 359: Records (Preview)") to the Java language—Records.
It's goal is to provide a compact syntax to declare containers of shallowly immutable data.

## Records in Action

It's common to declare classes in Java that hold only data and implement no behaviour.
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

No need to declare a constructor and worry about whether `equals` and `hashCode` are implemented correctly.
That's all done for you.

## It's not only about boilerplate

At first glance, records seem to address the "*java is too verbose*" issue.
While that might be true to some extent, the description of [JEP 359](https://openjdk.java.net/jeps/359) hints that records have a semantic goal, to model data as data.

> IDEs will help write most of the code in a data carrier class, but don't do anything to help the reader distill the design intent of "I'm a data carrier for x, y, and z" from the dozens of lines of boilerplate. Writing Java code that models simple aggregates should be easier -- to write, to read, and to verify as correct.
While it is superficially tempting to treat records as primarily being about boilerplate reduction, we instead choose a more semantic goal: modeling data as data. (If the semantics are right, the boilerplate will take care of itself.) It should be easy, clear, and concise to declare shallowly-immutable, well-behaved nominal data aggregates.

## Constructor

Once a record has been declared, a constructor is automatically generated as well.
It accepts all the properties of a record in the order they were declared.
Continuing with our Cartesian point example, to create a new `Point`, the syntax is the same as creating any other object.

```java
var point = new Point(1.0, 2.9);
```

In cases where we want to validate constructor arguments, we have to explicitly declare the constructor.
It may be declared without a formal parameter list though, since it is assumed to be identical to the state description.
Parameters that are left unassigned in the constructor body are implicitly initialized.
This means that we don't need to assign every constructor parameter to a field (`this.x = x`).

```java
record Point(double x, double y) {
  public Point {
    // to illustrate, let's imagine that the value 3.5
    // is not allowed for the x coordinate
    if (x == 3.5) {
      throw new IllegalArgumentException();
    }
  }
}
```

## Immutability

Records are shallowly immutable.
Declared fields are `final` and you're only provided with accessors.

```java
var point = new Point(2.9, 5.2);
System.out.println(point.x()); // prints out 2.9
```

The new syntax provides an easy way to create what [DDD practitioners](https://amzn.to/320l3Z5 "Domain-Driven Design: Tackling Complexity in the Heart of Software") call value objects.
These are immutable objects that model a value that's only distinguishable by its state.
If you have a need to modify a record, you need to create a new one.

## Static methods

## Methods

## limitations

## Summary

Records in other languages
Preview feature, could change
Enable preview feature
