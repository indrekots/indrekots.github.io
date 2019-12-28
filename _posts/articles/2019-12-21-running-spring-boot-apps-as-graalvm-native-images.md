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

## GraalVM native image 101

Java applications are compiled into bytecode using `javac`.
During application runtime, the JVM loads class files into memory and analyzes the program's performance for [hot spots](https://en.wikipedia.org/wiki/Hot_spot_(computer_programming)); hence the name "[HotSpot JVM](https://en.wikipedia.org/wiki/HotSpot)".
The *just-in-time* (JIT) compiler kicks in and compiles parts of the application which are executed repeatedly into native machine code.
[JIT compilation, however, requires processor time and memory](https://aboullaite.me/understanding-jit-compiler-just-in-time-compiler/ "Understanding JIT compiler") which affects the startup time of the application.

GraalVM native image allows us to *ahead-of-time* compile our JVM application into machine code.
It statically analyzes application's bytecode, finds all classes and methods that will be called and compiles them into a native executable.
The output is a platform specific executable binary of your application.

```
# start my application
./my-application
```

// example of hello world

## What's supported and what doesn't work

The inherent nature of JVM is that the runtime is dynamic.
Applications can load new classes at runtime, classes which perhaps were not available during compilation.
We can reflectively access classes and execute methods unknown to static analyzers.
Spring applications make heavy use of dynamic proxies for example.
GraalVM native images need to know everything about the running application ahead of time.
Thus, not all Java applications are suitable for native image generation.
GraalVM native image does support reflection but we need to give it a hand by supplying some metadata.

-- table

Problem with Spring Boot in containers, slow startup, high memory usage, high CPU usage.

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
