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
published: false
aging: false
---

I think I'm late to the party and understand this is a somewhat controversial topic (add link mabye?) but recently I have switched to favor constructor injection over field injection. I understand that this topic has been discussed many times before (links to previous articles). Nevertheless, I'd like to go over the arguments that made me prefer constructor injection.

* used to use field injection with autowired, did not even consider other approaches, although I was well aware of other approaches

I used to use Spring's field injection all the time. Although I was well aware, that Spring framework supports constructor and setter injection, I did not consider using them. There were multiple reasons. But mainly I liked the conciseness of adding `@Autowired` or `@Inject` annotation to a private field. The Spring framework would do the heavy lifting. Required dependencies were injected to classes without me having to do much work at all. That's what frameworks are for, right? To save us from doing the ugly work.

New classes started out small. Maybe they had one or two dependencies. But over time, like in many code bases, they grew larger. I needed to add support for a new feature. No problem. Let's take the existing class, and add another dependency to it. Spring makes it quite convenient. Just *autowire* it. This process was repeated multiple times until, lo and behold, we have created a monolith. But that's normal? Right? Raise your hand if you've seen a service class with ten or more dependencies.

At least I thought this was normal. Maybe it was cargo cult programming (link), but I did what everybody else did, without considering whether it was good or bad.

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

* field injection -> NPE about to happen
* invalid state when dependency not in constructor
* force clients to provide the required dependencies
* if you have many dependencies, constructor gets awkwardly big, that's a clear indication that your class/component is doing way to much (Single Responsibility Principle)

> You want to really cure the pain, not blindly apply pain killers to it, don’t you?

* immutability, field injection would require dependencies to be non final (verify it)
* if you want to enforce that dependencies are never changed, make the fields final, setter injection does not work here because final fields can only be set in the constructor (find reference)

* messy constructor is good, it's a sign that it needs refactoring

* [Dependencies need to be communicated publically](http://blog.schauderhaft.de/2012/01/01/the-one-correct-way-to-do-dependency-injection/#comment-2825180814), Oliver Gierke argues
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

## drawbacks of field injection

* cannot create immutable objects, fields cannot be final (verify this)
* classes are tightly coupled to a DI container and cannot be used outside of it, only the DI container knows how to inject fields into your bean, you might think that I'm never going to use my application classes outside of my DI container, I'm not writing a general purpose library, this is some internal application for an enterprise, but what about testing then? you do write tests, right? do you use reflection to inject beans? or set up a spring container for tests?
* what's wrong with setting up a spring container during unit tests? I would argue that these are not unit tests then, you're not testing a single focused unit of code
* dependencies are hidden from the public
* it's easy to violate the single responsibility principle, super easy to add a new dependency, suddenly you discover you have 10+ dependencies injected to a single class, I'm quite sure you're class has more than one responsibility then, you have created a [god object](https://en.wikipedia.org/wiki/God_object)

## reducing boilerplate

* [as of Spring 4.3](https://spring.io/blog/2016/03/04/core-container-refinements-in-spring-framework-4-3), implicit constructor injection for single-constructor classes is available, meaning that no need to provide `@Autowired`/`@Inject` annotations (`@Configuration` classes support constructor injection as well) (demo, verify)
* Lombok to reduce boilerplate

* maybe it's my confirmation bias at play here but i see this popping up in a lot of places (IntelliJ IDEA warning, JHipster removed field injection, blog posts)

* http://olivergierke.de/2013/11/why-field-injection-is-evil/
* https://www.petrikainulainen.net/software-development/design/why-i-changed-my-mind-about-field-injection/
* http://vojtechruzicka.com/field-dependency-injection-considered-harmful/


* don't be a cargo cult programmer, consider the pros and cons and make up your mind
* following constructor injection might be a "purist" way of thinking, and real world is not pure,

## to read

* https://www.martinfowler.com/articles/injection.html
* https://softwareengineering.stackexchange.com/questions/300706/dependency-injection-field-injection-vs-constructor-injection
* https://github.com/olivergierke/ninjector

## from spring docs

http://docs.spring.io/spring/docs/4.2.x/spring-framework-reference/html/beans.html#beans-constructor-injection

> The Spring team generally advocates constructor injection as it enables one to implement application components as immutable objects and to ensure that required dependencies are not null. Furthermore constructor-injected components are always returned to client (calling) code in a fully initialized state. As a side note, a large number of constructor arguments is a bad code smell, implying that the class likely has too many responsibilities and should be refactored to better address proper separation of concerns.
Setter injection should primarily only be used for optional dependencies that can be assigned reasonable default values within the class. Otherwise, not-null checks must be performed everywhere the code uses the dependency. One benefit of setter injection is that setter methods make objects of that class amenable to reconfiguration or re-injection later.

* for mandatory dependencies use constructor injection
* for optional dependencies, use setter injection
* avoid field injection in most cases

## other topics

* field injection in spring configuration classes?
* field injection in JUnit tests with Spring?

## further reading
