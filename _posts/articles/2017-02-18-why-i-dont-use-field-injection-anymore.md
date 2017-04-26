---
layout: post
title: "Why I don't use field injection anymore"
excerpt:
modified: 2017-02-18 22:22:57 +0200
categories: articles
tags: [java, spring, autowire, inject, dependency injection]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: true
aging: false
---

I think I'm late to the party and understand this is a somewhat controversial topic (add link mabye?) but recently I have switched to favor constructor injection over field injection. I understand that this topic has been discussed many times before (links to previous articles). Nevertheless, I'd like to go over the arguments that made me prefer constructor injection.

I used to use Spring's field injection all the time. Although I was well aware, that Spring framework supports constructor and setter injection, I did not consider using them. There were multiple reasons. But mainly I liked the conciseness of adding `@Autowired` or `@Inject` annotation to a private field. The Spring framework would do the heavy lifting. Required dependencies were injected to classes without me having to do much work at all. That's what frameworks are for, right? To save us from doing the ugly work.

New classes started out small. Maybe they had one or two dependencies. But over time, like in many code bases, they grew larger. I needed to add support for a new feature. No problem. Let's take the existing class, and add another dependency to it. Spring makes it quite convenient. Just *autowire* it. This process was repeated multiple times until, lo and behold, I had created a monolith. But that's normal? Right? Raise your hand if you've seen a service class with ten or more dependencies and line count approaching to several hundred.

At least I thought this was normal. Maybe it was [cargo cult programming](https://en.wikipedia.org/wiki/Cargo_cult_programming "Cargo Cult Programming"), but I did what everybody else did, without considering whether it was good or bad.

Today I would argue that field injection encourages classes to become god objects. It's so easy to add a new dependency.


* service classes with 10+ dependencies seemed to be "normal", this is what everybody does right? one service class per domain object and all domain object related business code is in the service class, grows super large over time, but it's okay right? (https://www.petrikainulainen.net/software-development/design/the-biggest-flaw-of-spring-web-applications/)
* tests used either the spring container (kind of slow) or reflection (kind of a workaround) to inject mocks
* cargo cult programming, did what everybody else did, without considering if it is good or bad
* I'm kind of late to the game I think, always did what others around me did
* classes start out small, 1 or 2 dependencies injected, but it is super easy to add another dependency, just autowire it
* classes grew larger, just add another dependency, easy
* classes grow to a monolith
* violation of single responsibility principle (separation of concerns), but I did not think that would be a problem, did not see it (if constructor is messy, you can feel the pain and want to refactor it)
* a counter argument might be that it does not violate SRP because the class does everything related to the domain object, right? wrong. I'm pretty sure the the class touches other domain entities as well, e.g. save different domain entities in a single transaction

* controversial topic, I'm sure there's many who use field injection and are quite OK with it
* as I understand, this can create a never ending discussion/argument/flame war (has created)
* major influencers in the industry (e.g. Spring team) are moving towards discouraging field injection

## What's wrong with field injection?

As [Oliver Gierke put in his blog](http://olivergierke.de/2013/11/why-field-injection-is-evil/), it's NullPointerExceptions beginning to happen. Say you create an instance of your class without a DI container. You don't know, without looking at the source, which dependencies are needed for the class to behave correctly. So it's just a matter of time until you call a method which in turn calls a dependency that is not initialized. He argues that [dependencies need to be communicated publically](http://blog.schauderhaft.de/2012/01/01/the-one-correct-way-to-do-dependency-injection/#comment-2825180814).

The problem with field injection is that you are allowed to instantiate a class in an invalid state. The class does not force invariants. I personally feel that objects should be ready to be worked with after construction. Everything the object needs to do its job should be provided via the constructor. If the object needs a dependency to behave correctly, it seems only logical that it publicly advertises it as one of the required constructor parameters. It's like making a promise. As long as you provide me the required tools to work with, I make sure the job is done.

Admittedly when using Spring IoC, this isn't a problem since in application code you won't be instantiating objects yourself. But you will do it in unit tests.

## DI container will inject the required dependencies, not me

//smooth transition from the previous paragraph

When will I be ever instantiating my classes myself? My DI container will take care of it. Frameworks should make our lives easier. I don't need the public interface of my class telling me what pieces it needs to work correctly. As long as the DI container does the job, I'm good with that.

But what about testing? How will you be providing the required dependencies during unit tests for example?

## Constructor gets awkwardly big

I have a class with 10+ dependencies. Refactoring to use constructor injection would create a considerable amount of boilerplate code and now my constructor would so big that it is unusable. I'll never remember the order of the parameters that need to be passed to the constructor. And above all, it's ugly to look at a constructor with so many parameters.

But that's actually a good thing. This is a [clear indication that your class has probably too many responsibilities](http://vojtechruzicka.com/field-dependency-injection-considered-harmful/). [Robert C. Martin (a.k.a Uncle Bob)](https://en.wikipedia.org/wiki/Robert_Cecil_Martin) has said the following

> A class should have only one reason to change

Meaning that if a class as multiple responsibilities, it also has multiple reasons to change. And that is clear a violation of the [Single Responsibility Principle](https://en.wikipedia.org/wiki/Single_responsibility_principle). Field injection makes it very easy to add new dependencies. At the same time, it makes it very easy to grow your class until it becomes a [god object](https://en.wikipedia.org/wiki/God_object "God Object").

> [You want to really cure the pain, not blindly apply pain killers to it, don’t you?](http://olivergierke.de/2013/11/why-field-injection-is-evil/)

[Steve McConnell](https://en.wikipedia.org/wiki/Steve_McConnell "Steve McConnell"), in his book [Code Complete](https://www.goodreads.com/book/show/4845.Code_Complete "Code Complete"), mentions low-to-medium [fan-out](http://it.toolbox.com/blogs/enterprise-solutions/design-principles-fanin-vs-fanout-16088 "Design Principles: Fan-In vs Fan-Out") as a desirable characteristic of a design.

> Low-to-medium fan-out means having a given class use a low-to-medium number of other classes. High fan-out (more than about seven) indicates that a class uses a large number of other classes and may therefore be overly complex.

Seeing a big constructor is a good opportunity to think about splitting the class into smaller pieces.

## Immutability

In his book [Effective Java](https://www.goodreads.com/book/show/105099.Effective_Java_Programming_Language_Guide "Effective Java 2nd Edition"), [Joshua Bloch](https://twitter.com/joshbloch) recommends to favor immutable classes.

> Classes should be immutable unless there's a very good reason to make them mutable....If a class cannot be made immutable, limit its mutability as much as possible.

When we use field injection, we are required that our classes are mutable. We cannot declare our private fields to be `final` since this would break field injection (verify). If you want to enforce that the dependencies are never changed, you need to use the `final` keyword and initialize the fields in the constructor.

## To hide or expose dependencies

* using field injection is [hiding dependencies](https://twitter.com/olivergierke/status/314704198908403713), you don't know what your class depends on when you instantiate it, you only rely on the DI container
* you need to check the source code for the class to see what its dependencies are
* and to set them, you must use reflection
* constructor injection exposes dependencies to the outside world, the client of the class knows what the target depends on

* setter injection for optional dependencies
* optional dependencies - if not provided, the class can do its work without them, maybe has a reasonable default behavior

* Testability, easy to inject mocks via constructor
* Alternative would be to use reflection but it kind of seems like a workaround

* cons, too verbose maybe? Java is very verbose as is, constructor injection increases it
* constructors become messy if many dependencies exist (maybe you should break the class apart?)
* [Very little boilerplate code to use field injection](http://blog.schauderhaft.de/2012/01/01/the-one-correct-way-to-do-dependency-injection/). All you need is a simple annotation on the field.
* super convenient and boilerplate-less to add a new dependency via field injection

## Does constructor injection brake encapsulation

https://stackoverflow.com/questions/1005473/must-dependency-injection-come-at-the-expense-of-encapsulation

## Formalize class contracts

From code complete by Steve McConnell on class contracts.

> At a more detailed level, thinking of each class's interface as a contract with the rest of the program can yield good insights. Typically, the contract is something like "If you promise to provide data x, y and z and you promise they'll have characteristics a, b and c, I promise to perform operations 1, 2 and 3 within constraints 8, 9 and 10." The promises the clients of the class make to the class are typically called "preconditions," and the promises the object makes to its clients are called the "postconditions."

## drawbacks of field injection

* cannot create immutable objects, fields cannot be final (verify this)
* classes are tightly coupled to a DI container and cannot be used outside of it, only the DI container knows how to inject fields into your bean, you might think that I'm never going to use my application classes outside of my DI container, I'm not writing a general purpose library, this is some internal application for an enterprise, but what about testing then? you do write tests, right? do you use reflection to inject beans? or set up a spring container for tests?
* what's wrong with setting up a spring container during unit tests? I would argue that these are not unit tests then, you're not testing a single focused unit of code
* dependencies are hidden from the public
* it's easy to violate the single responsibility principle, super easy to add a new dependency, suddenly you discover you have 10+ dependencies injected to a single class, I'm quite sure you're class has more than one responsibility then, you have created a [god object](https://en.wikipedia.org/wiki/God_object)

## reducing boilerplate

* [as of Spring 4.3](https://spring.io/blog/2016/03/04/core-container-refinements-in-spring-framework-4-3), implicit constructor injection for single-constructor classes is available, meaning that no need to provide `@Autowired`/`@Inject` annotations (`@Configuration` classes support constructor injection as well) (demo, verify)
* Lombok to reduce boilerplate

* http://olivergierke.de/2013/11/why-field-injection-is-evil/
* https://www.petrikainulainen.net/software-development/design/why-i-changed-my-mind-about-field-injection/
* http://vojtechruzicka.com/field-dependency-injection-considered-harmful/


* don't be a cargo cult programmer, consider the pros and cons and make up your mind
* following constructor injection might be a "purist" way of thinking, and real world is not pure, pragmatics over theoretical purity, getting things done is more important than arguing about theoretical correctness, don’t want to argue endlessly about academic points of theory, I want to get stuff done, which is a valid point

## to read

* https://www.martinfowler.com/articles/injection.html
* https://softwareengineering.stackexchange.com/questions/300706/dependency-injection-field-injection-vs-constructor-injection
* https://github.com/olivergierke/ninjector

* for mandatory dependencies use constructor injection
* for optional dependencies, use setter injection
* avoid field injection in most cases

## other topics

* field injection in spring configuration classes?
* field injection in JUnit tests with Spring?

## Where the industry is moving?

Maybe it's my [confirmation bias](https://en.wikipedia.org/wiki/Confirmation_bias) at play here but I have seen the theme of favoring constructor injection popping up in many places. At first I started to notice that IntelliJ IDEA begin to display warnings if I was using field injection.

//image of a warning

It says that the Spring team recommends to use constructor injection instead. Out of curiosity, I scanned through [Spring's reference manual](http://docs.spring.io/spring/docs/4.2.x/spring-framework-reference/html/beans.html#beans-constructor-injection "Spring's reference manual") and found the following section.

> The Spring team generally advocates constructor injection as it enables one to implement application components as immutable objects and to ensure that required dependencies are not `null`. Furthermore constructor-injected components are always returned to client (calling) code in a fully initialized state. As a side note, a large number of constructor arguments is a bad code smell, implying that the class likely has too many responsibilities and should be refactored to better address proper separation of concerns.
Setter injection should primarily only be used for optional dependencies that can be assigned reasonable default values within the class. Otherwise, not-null checks must be performed everywhere the code uses the dependency. One benefit of setter injection is that setter methods make objects of that class amenable to reconfiguration or re-injection later.

In the release notes of [JHipster 4.0.0](https://jhipster.github.io/2017/02/02/jhipster-release-4.0.0.html), an application generator for creating Spring Boot + Angular projects, we can find that they have moved away from field-based injection as well.

>JHipster is a complete upgrade of Spring libraries, with some major refactoring. The most important one is our switch from field-based injection to constructor-based injection.

One of the reasons for the change is pointed out to be the following.

>Constructor-based injection is considered cleaner by many people, in particular as it eases testing

## further reading
