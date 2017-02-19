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
aging: true
---

* used to use field injection with autowired, did not even consider other approaches
* service classes with 10+ dependencies seemed to be "normal", this is what everybody does right? one service class per domain object and all domain object related business code is in the service class, grows super large over time, but it's okay right? (https://www.petrikainulainen.net/software-development/design/the-biggest-flaw-of-spring-web-applications/)
* tests used either the spring container (kind of slow) or reflection (kind of a workaround) to inject mocks
* cargo cult programming, did what everybody else did, without considering if it is good or bad
* classes start out small, 1 or 2 dependencies injected, but it is super easy to add another dependency, just autowire it
* classes grew larger, just add another dependency, easy
* classes grow to a monolith
* violation of single responsibility principle (separation of concerns), but I did not think that would be a problem, did not see it (if constructor is messy, you can feel the pain and want to refactor it)
* a counter argument might be that it does not violate SRP because the class does everything related to the domain object, right? wrong. I'm pretty sure the the class touches other domain entities as well, e.g. save different domain entities in a single transaction

* controversial topic, I'm sure there's many who use field injection and are quite OK with it

* field injection -> NPE about to happen
* invalid state when dependency not in constructor
* force clients to provide the required dependencies
* if you have many dependencies, constructor gets awkwardly big, that's a clear indication that your class/component is doing way to much (Single Responsibility Principle)

> You want to really cure the pain, not blindly apply pain killers to it, don’t you?

* immutability, field injection would require dependencies to be non final (verify it)

* messy constructor is good, it's a sign that it needs refactoring

* Dependencies need to be communicated publically, Oliver Gierke argues

* Testability, easy to inject mocks via constructor
* Alternative would be to use reflection but it kind of seems like a workaround

* cons, too verbose maybe? java is very verbose as is, constructor injection increases it
* constructors become messy if many dependencies exist (maybe you should break the class apart?)

* maybe it's my confirmation bias at play here but i see this popping up in a lot of places (IntelliJ IDEA warning, JHipster removed field injection, blog posts)

* http://olivergierke.de/2013/11/why-field-injection-is-evil/
* https://www.petrikainulainen.net/software-development/design/why-i-changed-my-mind-about-field-injection/
* http://vojtechruzicka.com/field-dependency-injection-considered-harmful/


* don't be a cargo cult programmer, consider the pros and cons and make up your mind
* following constructor injection might be a "purist" way of thinking, and real world is not pure,


## to read

* https://twitter.com/olivergierke/status/314704198908403713
* http://blog.schauderhaft.de/2012/01/01/the-one-correct-way-to-do-dependency-injection/
* http://blog.schauderhaft.de/2012/01/01/the-one-correct-way-to-do-dependency-injection/#comment-2825180814
* https://spring.io/blog/2016/03/04/core-container-refinements-in-spring-framework-4-3
* https://stackoverflow.com/questions/39890849/what-exactly-is-field-injection-and-how-to-avoid-it
* https://stackoverflow.com/questions/5433143/why-doesnt-spring-support-direct-field-dependency-injection-except-for-autowir

## from spring docs

http://docs.spring.io/spring/docs/4.2.x/spring-framework-reference/html/beans.html#beans-constructor-injection

> The Spring team generally advocates constructor injection as it enables one to implement application components as immutable objects and to ensure that required dependencies are not null. Furthermore constructor-injected components are always returned to client (calling) code in a fully initialized state. As a side note, a large number of constructor arguments is a bad code smell, implying that the class likely has too many responsibilities and should be refactored to better address proper separation of concerns.
Setter injection should primarily only be used for optional dependencies that can be assigned reasonable default values within the class. Otherwise, not-null checks must be performed everywhere the code uses the dependency. One benefit of setter injection is that setter methods make objects of that class amenable to reconfiguration or re-injection later.

## other topics

* field injection in spring configuration classes?
* field injection in JUnit tests with Spring?
