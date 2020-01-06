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
For Java workloads it's used as a regular JVM [but with performance advantages](https://www.youtube.com/watch?v=pR5NDkIZBOA "Twitter's quest for a wholly Graal runtime").
An interesting feature that GraalVM brings to the table is its ability to create *ahead-of-time* (AOT) compiled native images of JVM applications which promise faster startup times and lower memory footprint.
In this post we're going to focus on how to create native binaries from Spring applications.

## GraalVM Native Image 101

Java applications are compiled into bytecode using `javac`.
During application runtime, the JVM loads class files into memory and analyzes the program's performance for [hot spots](https://en.wikipedia.org/wiki/Hot_spot_(computer_programming)); hence the name "[HotSpot JVM](https://en.wikipedia.org/wiki/HotSpot)".
The *just-in-time* (JIT) compiler compiles parts of the application which are executed repeatedly into native machine code.
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

First, we need to compile the Java code with `javac`
```
$ javac HelloWorld.java
```

and then use `native-image` to build an executable binary from the class file.
```
$ native-image HelloWorld
```

To start the application, simply
```
$ ./helloworld
Hello World
```

## Native Image Java Limitations

GraalVM native image static analysis requires a closed-world assumption.
It needs to know all the bytecode that's ever reachable ahead of time during image generation.
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
It observes the behavior of an application running in a JVM and produces configuration files needed for native image generation.

To get a complete set of configuration files, you would need to exercise all code paths in your application.
A tests suite with 100% coverage would do the trick but in reality, test suites never test all paths.
Therefore, manual modification of these configuration files might be needed as well.

## Spring and GraalVM Native Image

Starting from Spring Framework 5.1, [initial support for GraalVM native images was provided](https://github.com/spring-projects/spring-framework/issues/21529 "Initial GraalVM native images support").
[The 5.2 development cycle is focusing on improving the integration](https://github.com/spring-projects/spring-framework/wiki/GraalVM-native-image-support#support-of-native-images-at-spring-framework-level "Support of native images at Spring Framework level") and full support without requiring extra configuration or workarounds is [one of the themes of the upcoming Spring Framework 5.3 release](https://twitter.com/devoxx/status/1128937256180473856).

<blockquote class="twitter-tweet"><p lang="en" dir="ltr"><a href="https://twitter.com/hashtag/Spring?src=hash&amp;ref_src=twsrc%5Etfw">#Spring</a> framework 5.3 (Q2 2020) will support <a href="https://twitter.com/hashtag/GraalVM?src=hash&amp;ref_src=twsrc%5Etfw">#GraalVM</a> native images and much more! <a href="https://twitter.com/hashtag/SpringBoot?src=hash&amp;ref_src=twsrc%5Etfw">#SpringBoot</a> <a href="https://t.co/GqsfknZ6gc">pic.twitter.com/GqsfknZ6gc</a></p>&mdash; Devoxx (@Devoxx) <a href="https://twitter.com/Devoxx/status/1128937256180473856?ref_src=twsrc%5Etfw">May 16, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## Example Spring Boot Application

[The `spring-graal-native` Github repository](https://github.com/spring-projects-experimental/spring-graal-native "Graal feature for building native images of Spring applications") has examples of how to build a native image from a Spring Boot application.
The project implements a [Graal `Feature`](https://www.graalvm.org/sdk/javadoc/index.html?org/graalvm/nativeimage/hosted/Feature.html) which does the heavy lifting when it comes to configuring reflection, proxies etc.

> Features allow clients to intercept the native image generation and run custom initialization code at various stages. All code within feature classes is executed during native image generation, and never at run time.

Let's focus on a hello-world-level Spring Boot example—[Spring MVC with Tomcat](https://github.com/indrekots/spring-graal-native/tree/master/spring-graal-native-samples/springmvc-tomcat).
Keep in mind, as of writing this, the example expects you're using GraalVM 19.2.1 and you have the `native-image` plugin installed.

Before building the example, we need to compile the Spring Graal Feature.
The [root of the repository](https://github.com/indrekots/spring-graal-native) has a bash script to do that.

```
$ ./build-feature.sh
```

Once that's finished, let's move to the [Spring MVC example folder](https://github.com/indrekots/spring-graal-native/tree/master/spring-graal-native-samples/springmvc-tomcat) and execute [`compile.sh`](https://github.com/indrekots/spring-graal-native/blob/master/spring-graal-native-samples/springmvc-tomcat/compile.sh).
It builds the Spring app using Maven and then generates a GraalVM native image.
The `native-image` command is supplied with the location of the Spring Graal Feature and various configuration files.
Be warned that native image generation takes considerably longer than a regular Maven build.
Also, the process likes to use a lot of RAM.
When finished, navigate to the `target` folder and start the app.

```
$ ./springmvc-tomcat
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::

...

INFO: Started TomcatApplication in 0.054 seconds (JVM running for 0.057)
```

Notice the fast startup time of 0.054 seconds.
For comparison, when running the app in a JVM, the reported startup time for me was 1.455 seconds.

## Summary

GraalVM native image enables us to build ahead-of-time compiled JVM applications that start very fast and use less memory.
That's definitely useful for short lived processes, especially in the serverless scene where you're billed by the millisecond.

Due to classpath scanning and auto-configuration, Spring Boot apps are [very CPU hungry during startup](https://stackoverflow.com/q/47270059/2928051).
When starting multiple Spring Boot apps simultaneously on a shared host, they start to compete for CPU and the startup time increases.
Orchestration tools could even kill the processes [because they didn't start fast enough](https://github.com/kubernetes/kubernetes/issues/3312).
Fast-starting ahead-of-time compiled Spring Boot apps could be the answer to the problem.

Containerized Spring Boot applications have something to gain as well.
Since a native binary has everything it needs, there's no need to bake a JRE into the container anymore. We can build [smaller Docker images](https://blog.softwaremill.com/small-fast-docker-images-using-graalvms-native-image-99c0bc92e70b "Small & fast Docker images using GraalVM’s native-image").

Several microservice-focused frameworks have already made use of the native image feature (e.g. [Quarkus](https://quarkus.io/), [Micronaut](https://micronaut.io/), [Helidon](https://helidon.io/)).
Although, Spring Boot does not yet fully support native image generation, I think it will be a significant addition to the framework.
