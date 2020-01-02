---
layout: post
title: "Running Spring Boot apps as GraalVM native images"
excerpt: ""
modified: 2019-12-21 18:56:18 +0300
categories: articles
tags: [java, graalvm, native image, containers]
image:
  path: /images/2019-12-28-graalvm-native-image/cover.jpg
  thumbnail: /images/2019-12-28-graalvm-native-image/cover_thumb.jpg
  caption: "Photo by [Alexander Sinn](https://unsplash.com/photos/KgLtFCgfC28)"
comments: true
share: true
published: true
aging: true
---

[GraalVM](https://www.graalvm.org/ "High-performance polyglot VM") is a high-performance polyglot virtual machine for running applications written in JavaScript, JVM-based languages, LLVM-based languages and [others](https://www.graalvm.org/docs/ "GraalVM Docs").
For Java workloads we can use it as a regular JVM.
However, running applications inside a JVM [comes with a cost](https://aboullaite.me/understanding-jit-compiler-just-in-time-compiler/ "Understanding JIT compiler").
Another interesting feature GraalVM brings to the table is its ability to create *ahead-of-time* (AOT) compiled native images of JVM applications which promise faster startup times and lower memory footprint.
In this post we're going to focus on how to create native binaries of Spring applications.

## GraalVM Native Image 101

Java applications are compiled into bytecode using `javac`.
During application runtime, the JVM loads class files into memory and analyzes the program's performance for [hot spots](https://en.wikipedia.org/wiki/Hot_spot_(computer_programming)); hence the name "[HotSpot JVM](https://en.wikipedia.org/wiki/HotSpot)".
The *just-in-time* (JIT) compiler kicks in and compiles parts of the application which are executed repeatedly into native machine code.
[JIT compilation, however, requires processor time and memory](https://aboullaite.me/understanding-jit-compiler-just-in-time-compiler/ "Understanding JIT compiler") which affects the startup time of the application.

GraalVM native image allows us to *ahead-of-time* compile our JVM application into machine code.
It statically analyzes application's bytecode, finds all classes and methods that are ever reachable and compiles them into a native executable.
The output is a platform specific executable binary of your application.

For instance, let's build a native image from the following "Hello World" program.
```java
class HelloWorld {
  public static void main(String[] args) {
    System.out.println("Hello World");
  }
}
```

First, we need to compile the Java code with `javac` and then use `native-image` to build the binary from the class file.
```
javac HelloWorld.java
native-image HelloWorld
```

`native-image` builds an executable binary.
```
$ ./helloworld
Hello World
```

## Native Image Java Limitations

GraalVM native image static analysis requires a closed-world assumption.
It needs to know all classes and bytecode that is ever reachable ahead of time during image generation.
Thus, [not all Java features are supported](https://github.com/oracle/graal/blob/master/substratevm/LIMITATIONS.md "Native Image Java Limitations") or they require configuration.

For example, dynamic class loading/unloading is not supported.
Reflection requires configuration.
CGLIB proxies don't work with native images.
On the other hand, JDK proxies are supported but require configuration.
Additionally, you need to tell native image about all resource accesses.

Configuration is supplied in a form of a JSON document.
For instance, to configure reflection, you create the following file and use the `-H:ReflectionConfigurationFiles` command line flag to specify the file location to the `native-image` command.

```json
[
  { "name":"java.lang.Object" },
  { "name":"org.apache.naming.factory.ResourceFactory", "methods" : [{"name": "<init>","parameterTypes":[]}] },
  ...
]
```

Similar files have to be created to configure dynamic proxies, JNI and resource accesses.
Doing all this by hand is a lot of work though, especially when we're dealing with a large application.
Fortunately, there's a [Java agent that can generate the configuration](Introducing the Tracing Agent: Simplifying GraalVM Native Image Configuration "Introducing the Tracing Agent: Simplifying GraalVM Native Image Configuration").
It observes the behavior of the application running in a JVM and produces configuration files needed for native image generation.

To get a complete set of configuration files, you would need to exercise all code paths in your application.
A tests suite with 100% coverage would do the trick but in reality, test suites never test all paths.
Therefore, manual modification of these configuration files might be needed as well.

## Spring and GraalVM Native Image

Starting from Spring Framework 5.1, [initial support for GraalVM native images was provided](https://github.com/spring-projects/spring-framework/issues/21529 "Initial GraalVM native images support").
[The 5.2 development cycle is focusing on improving the integration](https://github.com/spring-projects/spring-framework/wiki/GraalVM-native-image-support#support-of-native-images-at-spring-framework-level "Support of native images at Spring Framework level") and full support without requiring extra configuration or workarounds is [one of the themes of the upcoming Spring Framework 5.3 release](https://twitter.com/devoxx/status/1128937256180473856).

<blockquote class="twitter-tweet"><p lang="en" dir="ltr"><a href="https://twitter.com/hashtag/Spring?src=hash&amp;ref_src=twsrc%5Etfw">#Spring</a> framework 5.3 (Q2 2020) will support <a href="https://twitter.com/hashtag/GraalVM?src=hash&amp;ref_src=twsrc%5Etfw">#GraalVM</a> native images and much more! <a href="https://twitter.com/hashtag/SpringBoot?src=hash&amp;ref_src=twsrc%5Etfw">#SpringBoot</a> <a href="https://t.co/GqsfknZ6gc">pic.twitter.com/GqsfknZ6gc</a></p>&mdash; Devoxx (@Devoxx) <a href="https://twitter.com/Devoxx/status/1128937256180473856?ref_src=twsrc%5Etfw">May 16, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## Example Spring Boot Application

[The `spring-graal-native` Github repository](https://github.com/spring-projects-experimental/spring-graal-native "Graal feature for building native images of Spring applications") contains examples of how to build a native image from a Spring Boot application.
The project implements a [GraalVM `Feature`](https://www.graalvm.org/sdk/javadoc/index.html?org/graalvm/nativeimage/hosted/Feature.html) which does the heavy lifting when it comes to configuring reflection, proxies etc.

> Features allow clients to intercept the native image generation and run custom initialization code at various stages. All code within feature classes is executed during native image generation, and never at run time.

Let's focus on a *hello-world-level* Spring Boot example—[Spring MVC with Tomcat](https://github.com/spring-projects-experimental/spring-graal-native/tree/master/spring-graal-native-samples/springmvc-tomcat).
Keep in mind, as of writing this, the example expects that you're using GraalVM 19.2.1 and you have the `native-image` plugin installed.

// example commands
// first, let's build the Feature
// then build image
// takes time, longer than a regular maven build
// additionally, it likes to eat all the RAM

// compare startup times

## Summary

## Reflection

When doing reflection, you need to warn native-image that you're doing it.
Static configuration file via JSON.
Otherwise you'll get method/class not found during runtime.

## Resource access

e.g. application.properties, application.yml

When calling Class.getResource(), native-image needs to know about it ahead of time.
Resources need to be put into the image.
Otherwise, during runtime you'll find yourself with a missing resource.
Supply another json file.

## Dynamic proxies

CGLIB not supported. JDK proxies are supported.
We need to provide ahead of time the proxies that need to be baked into the image.

Spring Boot 2.2 can work without CGLIB proxies (`@SpringBootApplication(proxyBeanMethods=false)`)

## Class initialization

Static initializers can be run during image build time, or runtime.
If you had an expensive operation that was done during class initialization, it could be done during image build time.
This would improve the startup time of your application.

Crafting json files is painful. Build run test loop.
native image agent, perhaps you have to run all your test cases as well to exercise your app through all possible code paths.

## Dynamic configuration with Spring Boot GraalVM feature
native-image finds your `@AutomaticFeature` from class path during image build time.

Github repo spring-graal-native

JIT vs AOC

Download GraalVM, download native-image command

fully self contained executable

Future of container workloads
Comparison of startup time, comparison of container size, comparison of memory usage

Faster startup time, lots of work is done during image build time, no longer needed to scan classpath during startup for configuration etc.
Startup times comparable to native languages (e.g. Go).

Spring Boot roadmap, what does not work yet
