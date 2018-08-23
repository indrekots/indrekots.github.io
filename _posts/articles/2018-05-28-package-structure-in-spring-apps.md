---
layout: post
title: "Package structure in Spring apps"
excerpt:
modified: 2018-05-28 18:56:18 +0300
categories: articles
tags: [spring, java]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: false
aging: false
---

Multiple levels of organization:
1) methods
2) classes
3) packages

If you look at a typical Spring web application, most likely you see the following top level package names.

```
controller (or web, or rest)
service
repository (or dao)
```

This clearly indicates that a [three-tiered layered architecture is used](https://en.wikipedia.org/wiki/Multitier_architecture#Three-tier_architecture "Three-tier architecture")
We have a package for controllers that accept incoming HTTP requests.
`service` package includes classes that deal with application specific business logic.
And finally, `repository` contains data access functionality.
From a technical perspective, this separation makes sense.
This is a widely used approach that is familiar to developers.
Packages are organized in a fashion that makes it easy to understand where a specific class may be.
But is it a good approach to structure software?

## Quick recap of packages

In Java we can use [packages](https://docs.oracle.com/javase/tutorial/java/package/packages.html "Creating and Using Packages") to organize classes into logical groups.
Packages are also used to avoid naming conflicts.
If we have two types with the same name, we can put them into separate packages which gives them a distinct fully qualified name.
A third, and in my opinion a less used, function of packages is encapsulation.
Packages are not just folders on disk that organize classes the way you might organize your photo collection.
They can be used to define boundaries in code and hide classes from other parts of the system that don't need to know about them.

When designing a class, we need to consider what parts of it to hide and what to make `public`.
Essentially, we're are creating an [interface](https://en.wikipedia.org/wiki/Interface_(computing)) (in general OO sense) that describes how the class must be used.
Details of the inner workings of a class are [hidden from the outside world](https://en.wikipedia.org/wiki/Information_hiding "Information hiding").
Programming to the interface reduces coupling between classes because we're not depending on internal implementation details.
If we honor the interface by not changing it, class' implementation specifics can be changed without having to make modifications to the class' users.
But why not take this approach a step further and use it with packages?

low coupling and high cohesion

A layered software architecture defined with packages has one big drawback—all of our classes need to be public.
If a controller class inside the `controller` package wants to call a method in a service class that's in a `service` package, the latter needs to be public.
Again, a repository class needs to be public because we want to access it from the service layer.
What's bad about having a public repository class you might ask?
Having it publicly accessible can be beneficial, because there might be other service layer classes that want to operate on the same repository, right?
Unless you're dealing with a demo application, most of the time, accessing a repository means that other operations need to take place as well.
You might need to introduce logging, check permissions or start other business specific processes.
For example, whenever a new user is created, you might want to send them a welcome e-mail.
If `UserRepository` was publicly accessible, it's easy to bypass all of it without even knowing that you might have done something wrong.

Although it might not make sense to access the repository from anywhere else than a specific service class.
We have grown accustomed to starting every Java class with `public class ...`.
We could enforce this by having them in the same package and making the repository package-private.

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

* Layered approach to separate code is technical in nature and does not reflect the domain
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
