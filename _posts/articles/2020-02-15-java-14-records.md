---
layout: post
title: "Java 14 Records"
excerpt: "Java 14 introduces a new preview feature to the language—Records—with a goal to provide a compact syntax for declaring containers of shallowly immutable data."
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

Java 14 introduces a [new preview feature](https://openjdk.java.net/jeps/359 "JEP 359: Records (Preview)") to the language—Records.
Its goal is to offer a compact syntax for declaring containers of shallowly immutable data.

## Records in Action

It's common to declare classes in Java that hold nothing more than data.
Sometimes they're referred to as *data transfer objects* or *value objects* but you can think of them as just a bag of getters and setters.
For example, to represent a point on a [Cartesian plane](https://en.wikipedia.org/wiki/Cartesian_coordinate_system "Cartesian coordinate system"), the following class could do the job.

```java
class Point {
    private double x;
    private double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    // getters and setters
    // toString
    // equals and hashCode
}
```

The bits of information that make the class interesting are the x and y coordinates.
However, in order for the class to behave correctly, a constructor has to be declared, getters and possibly setters have to be provided, and also `toString`, `equals` and `hashCode` methods have to be implemented.

Modern IDEs can generate this code for us.
Users of [Lombok](https://projectlombok.org/ "Project Lombok") are well aware of the `@Data` and `@Value` annotations that generate the boilerplate code during build time.
Records, on the other hand, introduce a syntax to declare *only* the parts that are important for the reader.

```java
record Point(double x, double y) {}
```

No need to declare a constructor or worry whether `equals` and `hashCode` are implemented correctly.
Records need a name and a *state description*, that's it.

## It's not only about boilerplate

At first glance, records seem to address the "*java is too verbose*" issue.
While that might be true to some extent, the description of [JEP 359: Records](https://openjdk.java.net/jeps/359) hints that records have a semantic goal, to model data as data.

> IDEs will help write most of the code in a data carrier class, but don't do anything to help the reader distill the design intent of "I'm a data carrier for x, y, and z" from the dozens of lines of boilerplate. Writing Java code that models simple aggregates should be easier -- to write, to read, and to verify as correct.
While it is superficially tempting to treat records as primarily being about boilerplate reduction, we instead choose a more semantic goal: modeling data as data. (If the semantics are right, the boilerplate will take care of itself.) It should be easy, clear, and concise to declare shallowly-immutable, well-behaved nominal data aggregates.

## Constructor

A constructor for a record is automatically generated.
It accepts the *components* of a record in the order they were declared.
Continuing with our Cartesian point example, to create a new `Point`, the syntax is the same as creating any other object.

```java
var point = new Point(1.0, 2.9);
```

In cases where we want to validate constructor arguments, we have to explicitly declare the constructor.
It may be declared without a formal parameter list though, since it is assumed to be identical to the *state description*.
Parameters that are left unassigned in the constructor body are implicitly initialised.
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
Declared fields are implicitly `final` and they can be accessed via accessor methods.

```java
var point = new Point(2.9, 5.2);
System.out.println(point.x()); // prints out 2.9
```

The new syntax provides an easy way to create what [DDD](https://en.wikipedia.org/wiki/Domain-driven_design "Domain-driven design") practitioners call [value objects](https://www.martinfowler.com/bliki/ValueObject.html "ValueObject")
These are immutable objects that model a value that's only distinguishable by its state.
If you need to edit a record, you need to create a new one.

{: .notice}
📖You can read more about value objects and DDD form the book that started it all: [Domain-Driven Design: Tackling Complexity in the Heart of Software](https://amzn.to/320l3Z5 "Domain-Driven Design: Tackling Complexity in the Heart of Software")

## Static methods

Records can contain static methods.
For example, we could include a static factory method to the `Point` record that creates a new point at the origin.

```java
record Point(double x, double y) {
  public static Point origin() {
    return new Point(0, 0);
  }
}
```

## Instance Methods

Besides static methods, we can also include instance methods to records.
Let's say, we would like to know the distance to another point on a Cartesian plane.
The formula for finding the distance between two points is:

\\[ \sqrt{(x_2 - x_1)^2 + (y_2 - y_1)^2} \\]

Implementing it as an instance method on the `Point` record looks as follows.

```java
record Point(double x, double y) {
  public double distanceTo(Point point) {
    return Math.sqrt(Math.pow(point.x - this.x, 2) + Math.pow(point.y - this.y, 2));
  }
}
```

## Limitations

Due to the intended purpose of records, there are limitations we have to take into account.
Records cannot extend any other class and cannot declare instance fields that aren't part of its state description.
This is to make sure the state description of a record alone defines its representation.
Since immutability is at the core of a record, the *components* of a record are implicitly final.

## Summary

Records introduce a new syntax to the Java language, allowing programmers to declare immutable data classes.
While on the outset, the biggest benefit seems to be the reduction of boilerplate code, there's also a semantic difference.
Records *model data as data* and provide a clear sign to the reader that they're dealing with a data class.

Keep in mind, records in Java 14 are a [preview feature](https://openjdk.java.net/jeps/12 "JEP 12: Preview Language and VM Features").
The implementation could change and evolve in the upcoming releases.
To unlock records, use the `--enable-preview` command line flag for `javac` and `java`.
