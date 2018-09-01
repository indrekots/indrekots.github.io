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
├── controller
├── service
└── repository
{% endhighlight %}

This clearly indicates that a [layered architecture is used](https://en.wikipedia.org/wiki/Multitier_architecture#Three-tier_architecture "Three-tier architecture").
We have a package for controllers that accept incoming HTTP requests. `service` package includes classes that deal with application specific business logic.
And finally, `repository` contains data access functionality.

<figure class="align-center">
  <img src="{{ '/images/2018-05-28-package-structure/groups.jpg' | absolute_url }}" alt="">
  <figcaption>Classes are grouped like cutlery on a tray. <a href="https://unsplash.com/photos/yw3UaP-5ybM">Image by Jarosław Ceborski</a></figcaption>
</figure>

From a technical perspective, this separation makes sense.
It's a widely used approach that's familiar to developers.
Packages are grouped in a fashion that makes it easy to understand where a specific class may be.
But is it a good approach to structure software?

## Quick recap of packages

In Java [packages](https://docs.oracle.com/javase/tutorial/java/package/packages.html "Creating and Using Packages") can be used for the following

1. Oranizing classes into logical groups
2. Avoiding naming conflicts. If we have two types with the same name, we can put them into separate packages which gives them a distinct fully qualified name.
3. Encapsulation

In my opininion, using packages as a tool to achieve encapsulation is the least used function of the three.
Packages are more than just folders on disk that organize classes the way you might organize your photo collection.
They can be used to define boundaries in code and hide classes from other parts of the system that don't need to know about them.
More on that later.

## Package by layer

Designing packages by layer is technical in nature and does not reflect the underlying domain.

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">When developers write novels.<br><br>Chapter 1 - Characters<br>Chapter 2 - Locations<br>Chapter 3 - Vehicles<br>Chapter 4 - Relationships<br>Chapter 5 - Plots<br>Chapter 6 - Conclusions<br>Appendix A - Twists <a href="https://t.co/8lcc27WeoA">https://t.co/8lcc27WeoA</a></p>&mdash; Richard Dalton 🇪🇺 (@richardadalton) <a href="https://twitter.com/richardadalton/status/936228404084559872?ref_src=twsrc%5Etfw">November 30, 2017</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Whenever a layer above calls a layer below, we're crossing package boundaries.
To do that, the callee needs to be `public`.
For example, if a service class inside the `service` package wants to call a method in a repository class that's in the `repository` package, the latter needs to be public.
What's bad about having a type declared as public you might ask?
Having it publicly accessible can be beneficial, because there might be other service layer classes that want to operate on the same repository, right?

I'd like to take a step back and look at this from another angle.
When we design a class, we need to consider what parts of it to hide and what to make `public`.
Essentially, we're using access modifiers to [hide the inner workings of a class from the outside world](https://en.wikipedia.org/wiki/Information_hiding "Information hiding").

Unless we're dealing with a demo application, most of the time, accessing a repository can be considered *the inner workings* of a larger feature.
In addition to, say, storing data to a database, you might need to introduce logging, check permissions or start other business specific processes.
For example, whenever a new user is created, you might want to send them a welcome e-mail.
If a hypothetical `UserRepository` was publicly accessible to all other classes, it's easy to bypass other domain related checks and processes without even knowing that you might have done something wrong.

Designing packages by layer means that the classes that are used together the most are in different packages.
This is referred to as [low cohesion](https://en.wikipedia.org/wiki/Cohesion_(computer_science) "Cohesion").
Our classes *need* to be public, otherwise the layer above cannot call them.
And when something is public, every other class can call it, meaning that it's easy to introduce architecture violations.
It's easy to declare a dependency on a class that might just be an *implementation detail*.
There's a higher risk that the system evolves into something that's difficult to change.

## Package by feature

* Tightly coupled classes belong in the same package

We use public access modifier all the time because we package our classes by layers.
We have to possibility to encapsulate classes inside packages.
Using a layered approach, we intentionally give this benefit away.

When we have everything public, we lose the benefit of encapsulation.
It becomes hard to enforce what classes can use what other classes.
The more dependents you have for a class, the harder it becomes to change it.
There's a higher likelihood that we break something.

## Whoops, where did my architecture go
* http://olivergierke.de/2013/01/whoops-where-did-my-architecture-go/
* https://www.youtube.com/watch?v=tEm0USdF-70/

* On the other hand, this is widely used and understood
* It's easy to introduce an architecture violation in a monolith with layers, you call some class that you should not have access to
* Because of layers first approach, our classes need to be public. And because they're public, It's very easy to introduce a dependency we'd like to avoid
* Slices first approach - only types that need to be public can be kept public, others can be package private
* Default visibility for classes generated by IDE's is `public`, although in the Java language the default is `package-private`

* Layers are widely understood but less important to business (DDD)
* Slices hardly understood but key for business requirements

* layer.slice, slice.layer, slice?
* layer first - types have to be public so they could be accessed over package boundaries, leaks slice internals, lower layers visible to everyone
* slice first - also types have to be public
* slice only - layering becomes slice internal, you can keep slice internals private to the package and only expose the API
This means that nobody can directly access code that should not be accessed directly.
E.g. you can keep a repository class private to the package.
It might not be good to let others access the repository directly.
There might be business specific processes present that need to be done before something is stored to the database.

> Start with less packages and the least visibility possible

slices can become bounded context, extracted to a separate maven module, microservices anyone?



## Software architecture as code (https://www.youtube.com/watch?v=oDpdaXt0HQI)

Abstraction helps us to reason about big/complex software systems (modules, components, layers etc.).
