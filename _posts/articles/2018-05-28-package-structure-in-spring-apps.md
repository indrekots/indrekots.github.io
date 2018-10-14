---
layout: post
title: "Package structure in Spring apps"
excerpt:
modified: 2018-05-28 18:56:18 +0300
categories: articles
tags: [spring, java]
image:
  feature: 2018-05-28-package-structure/cover.jpg
  credit: Rubén Bagüés
  creditlink: https://unsplash.com/photos/ZzApzgh5lxo
comments: true
share: true
published: false
aging: false
---

If you've ever worked on a typical [Spring web application](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html "Spring Web MVC"), most likely you have seen the following top level packages (or something similar).

{% highlight text %}
.
├── controller - web stuff
├── service - "business" stuff
└── repository - database stuff
{% endhighlight %}

This clearly indicates that a [layered architecture is used](https://en.wikipedia.org/wiki/Multitier_architecture#Three-tier_architecture "Three-tier architecture").
We have a package for controllers that accept incoming HTTP requests. `service` package includes classes that deal with application specific business logic.
And finally, `repository` contains data access functionality.

<figure class="align-center">
  <img src="{{ '/images/2018-05-28-package-structure/groups.jpg' | absolute_url }}" alt="">
  <figcaption>Classes are grouped like cutlery on a tray. <a href="https://unsplash.com/photos/yw3UaP-5ybM">Image by Jarosław Ceborski</a></figcaption>
</figure>

From a technical perspective, this separation makes sense.
It's a widely used approach that's familiar to many software developers.
Packages are grouped like cutlery on a tray.
Controllers, services and repositories are in separate packages the same way knives, forks and spoons are in their respective containers.
But is it a good approach to structure software?
Should layers in Java be modelled with packages?
Are there any alternatives?

## Quick recap of packages

In Java, [packages](https://docs.oracle.com/javase/tutorial/java/package/packages.html "Creating and Using Packages") can be used for the following

1. Oranizing classes into logical groups
2. Namespace classes to avoiding naming conflicts. If we have two types with the same name, we can put them into separate packages which gives them a distinct fully qualified name.
3. Encapsulation

In my opinion, using packages as a tool to achieve encapsulation is the least used function of the three.
Packages are more than just folders on disk that organize classes the way you might organize your photo collection.
They can be used to define boundaries in code and hide classes from other parts of the system that don't need to know about them.
More on that later.

## Package by layer

The first chapter of [Patterns of Enterprise Application Architecture by Martin Fowler](https://www.amazon.com/Patterns-Enterprise-Application-Architecture-Martin/dp/0321127420) says that layering is one of the most common techniques that software designers use to break apart a complicated software system.
They make a complex system easier to reason about.
In Java, we usually see layers implemented using packages.
If you have ever worked on a Java based web application, you have probably seen it yourself.

<figure class="align-center">
  <img src="{{ '/images/2018-05-28-package-structure/package_by_layer.png' | absolute_url }}" alt="Layered architecture modelled with packages.">
  <figcaption>Layered architecture modelled with packages. Types in each layer have to be public.</figcaption>
</figure>

Designing packages by layer is technical in nature and does not reflect the underlying domain.
Whenever a layer above calls a layer below, we're crossing package boundaries.
To do that, the callee needs to be `public`.
For example, if a service class inside the `service` package wants to call a method in a repository class that's in the `repository` package, the latter needs to be public.
What's bad about having a type declared as public you might ask?
Having it publicly accessible can be beneficial, because there might be other service layer classes that want to operate on the same repository, right?

### Encapsulation

I'd like to take a step back and look at this from another angle.
When we design a class, we need to consider what parts of it to hide and what to make `public`.
Essentially, we're using [access modifiers](https://docs.oracle.com/javase/tutorial/java/javaOO/accesscontrol.html "Controlling Access to Members of a Class") to [hide the inner workings of a class from the outside world](https://en.wikipedia.org/wiki/Information_hiding "Information hiding").
We don't want anybody to freely access the private parts of our class because (a) we want to hide complexity and (b) have the freedom the change implementation details without having the entire world know about it.
Using packages for encapsulation, it is possible to achieve to same goals but on a higher level of abstraction.

Unless we're dealing with a demo application, most of the time, accessing a repository can be considered *an implementation detail* of a larger feature.
In addition to, say, storing data to a database, you might need to introduce logging, check permissions or start other business specific processes.
For example, whenever a new user is created in your [SaaS](https://en.wikipedia.org/wiki/Software_as_a_service "Software as a service") product, you might want to send them a welcome e-mail.
If a hypothetical `UserRepository` was publicly accessible to all other classes, it's easy to bypass important domain related checks and processes without even knowing that you might have done something wrong.

### Everything public

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">When developers write novels.<br><br>Chapter 1 - Characters<br>Chapter 2 - Locations<br>Chapter 3 - Vehicles<br>Chapter 4 - Relationships<br>Chapter 5 - Plots<br>Chapter 6 - Conclusions<br>Appendix A - Twists <a href="https://t.co/8lcc27WeoA">https://t.co/8lcc27WeoA</a></p>&mdash; Richard Dalton 🇪🇺 (@richardadalton) <a href="https://twitter.com/richardadalton/status/936228404084559872?ref_src=twsrc%5Etfw">November 30, 2017</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Designing packages by layer means that the classes that are used together the most are in different packages.
This leads to [low cohesion](https://en.wikipedia.org/wiki/Cohesion_(computer_science) "Cohesion").
Our classes *need* to be public, otherwise the layer above cannot call them.
When you think about it, we've hardwired ourselves to always use the `public` access modifier when creating a new class.
It does not help that `public` is usually the default option when generating a new class via an IDE.
And when something is public, every other class can call it, meaning that it's easy to introduce architecture violations.
Without discipline, it's easy to declare a dependency on a class that might just be an *implementation detail*.
There's a higher risk that the system evolves into something that's difficult to change.
Using a *package-by-layer* approach, we intentionally give away the benefits of encapsulation.

> If all types are public, Java packages are about organisation of code rather than encapsulation
>
> <footer><strong>Simon Brown</strong> &mdash; <a href="https://www.youtube.com/watch?v=kbKxmEeuvc4">Modular monoliths</a></footer>

## Package by feature

*If we stop doing package by layer, how should we structure our code?*
Instead of creating a package for each layer (a horizontal slice), what if the top-most level of organization in code was a feature (a vertical slice)?

<figure class="align-center">
  <img src="{{ '/images/2018-05-28-package-structure/package_by_feature.png' | absolute_url }}" alt="Packages representing a domain concept">
  <figcaption>Slicing software vertically with domain specific packages. Layered architecture is still present but hidden from the top level view.</figcaption>
</figure>

Compared to the *package-by-layer* approach, classes that are used together the most are now in the same package, allowing us to design more cohesive modules.
There's no benefit in making all classes public anymore.
As a matter of fact, we gain more from starting to create [package-private](https://docs.oracle.com/javase/tutorial/java/javaOO/accesscontrol.html "Controlling Access to Members of a Class") classes.
Types that are specific to a feature can be made package-private so they don't leak outside of their domain.
On the other hand, public classes define the API of the package their part of.

When slicing software vertically, we have better control over how classes call each other over package boundaries.
For example, a repository class could be made package-private so that nobody from any other package could call it directly.
Access to a repository can be encapsulated and callers can be forced to go through the public API.
Therefore, implementation details of a feature are hidden from the outside world.
Layered architecture could still exist inside a package but [layering in general becomes an implementation detail](http://olivergierke.de/2013/01/whoops-where-did-my-architecture-go/ "Whoops! Where did my architecture go").

> So what does the architecture of your application scream? When you look at the top level directory structure, and the source files in the highest level package; do they scream: **Health Care System**, or **Accounting System**, or **Inventory Management System**? Or do they scream: **Rails**, or **Spring/Hibernate**, or **ASP**?
>
> <footer><strong>Uncle Bob</strong> &mdash; <a href="https://8thlight.com/blog/uncle-bob/2011/09/30/Screaming-Architecture.html">Screaming Architecture</a></footer>

Packages could be designed around [Domain Driven Design](https://www.amazon.com/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215 "Domain-Driven Design: Tackling Complexity in the Heart of Software") [aggregates](https://martinfowler.com/bliki/DDD_Aggregate.html "DDD_Aggregate").
When it makes sense, vertical slices can become [bounded contexts](https://martinfowler.com/bliki/BoundedContext.html "BoundedContext").
If you see a need, they could be extracted into a separate [Maven module](https://maven.apache.org/guides/mini/guide-multiple-modules.html "Guide to Working with Multiple Modules").
With the advent of [the Java Module System](https://blog.codefx.org/java/java-module-system-tutorial/ "Code-First Java Module System Tutorial"), it is possible to think about domain boundaries at a higher level of abstraction.
Theoretically it should be relatively painless to extract a vertical slice into a separate application when we have well defined APIs in place between modules.

## Summary

Abstraction helps us to reason about big and complex software systems (modules, components, layers etc.).

// hexagonal architecture, ports and adapters, even if you have all classes public, they all look the same, basically no architecture

Where do I start?
// Stop making every class public - Simon Brown

Let software design come to life using software cells - Ralf Westphal -> compared the OSI network 7 layers with layered software architecture, OSI layering, each layer adds an additional level of abstraction, software layers are completely separate

Don't try to decompose your system along technical lines, decompose it along the usage/domain

coming up with module boundaries is hard, layered architecture is a simple starting point (M. Fowler)
Java 9 modules
