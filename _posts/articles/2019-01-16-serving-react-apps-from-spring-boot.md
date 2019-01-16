---
layout: post
title: "Serving React apps from Spring Boot"
excerpt: You've decided to build a wep app with React and want to use Spring Boot for the back end. This post will show you how to serve a React app from Spring Boot.
modified: 2019-01-16 18:56:18 +0300
categories: articles
tags: [react, web, spring, spring boot, java, gradle]
image:
  path: /images/2019-01-05-spring-boot-reactjs/cover.jpg
  thumbnail: /images/2019-01-05-spring-boot-reactjs/cover_thumb.jpg
  caption: "Photo background by [Samuel Scrimshaw](https://unsplash.com/photos/fkS-me35j7I)"
comments: true
share: true
published: true
aging: false
---

Let's say you've decided to create a web app with [React](https://reactjs.org/ "A JavaScript library for building user interfaces").
You also want to build a back end so you've picked [Spring Boot](https://github.com/spring-projects/spring-boot) to do the job.
There are lots of ways to serve a React app in production.
In this post, we're going to take a look at how to bundle the React app inside a Spring Boot back end.

## Create a Spring Boot app

To get started, go to [start.spring.io](https://start.spring.io/ "Spring Initializr") and generate yourself a new Spring Boot app (or use an existing one if you wish).
Make sure you at least pick the Web dependency.
In this example, I'm using Gradle as my build tool.
You should have the following folder structure (or similar):

```
.
├── build.gradle
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew
├── gradlew.bat
├── settings.gradle
└── src
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── example
    │   │           └── demo
    │   │               └── DemoApplication.java
    │   └── resources
    │       ├── application.properties
    │       ├── static
    │       └── templates
    └── test
        └── java
            └── com
                └── example
                    └── demo
                        └── DemoApplicationTests.java
```

## Serving static content with Spring Boot

To serve our front end web app from a Spring Boot jar file, we need to first understand how Spring Boot handles [static content](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-developing-web-applications.html#boot-features-spring-mvc-static-content).

> By default, Spring Boot serves static content from a directory called `/static` (or `/public` or `/resources` or `/META-INF/resources`) in the classpath or from the root of the `ServletContext`

For example, the following is an example `index.html` that we will put into `src/main/resources/static/` folder.

```html
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Static content test</title>
</head>
<body>
    <h1>Hello World</h1>
</body>
</html>
```

When we start our Spring Boot app and point our web browser to it (with default configuration the URL should be `localhost:8080`), *Hello World* should be displayed to us.
Everything in `src/main/resources/static/` is moved to the correct location in the packaged jar file.
Now that we know how to serve static content, it should be relatively easy to serve the React app in a similar manner.

*But wait! Javascript source files have to be built first, CSS minified etc.
We can't just put our React source files to `src/main/resources` and expect everything to work.*

That's correct.
We have to build the React app before we can move it into the `/static` directory.
Let's have a look at how to do that.
But first, we need to generate a new React app.

## Create a React app

It's super easy to get started with React by using [`create-react-app`](https://facebook.github.io/create-react-app/ "Create React App").
If you have the necessary tools installed (e.g. `node`, `npm`), you can just execute this command

```
npx create-react-app enter-app-name-here
```

You're free to choose where you would like to place the generated web app.
In this example, I'm going to copy it to `src/main/webapp`.

## Gradle build script

Before we can put our web app to production, [we must create a minified bundle with `npm run build`](https://github.com/facebook/create-react-app#quick-overview "Create React App Quick Overview").
To serve the minified bundle with Spring Boot, we have to move it to one of the directories where [Spring Boot serves static content](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-developing-web-applications.html#boot-features-spring-mvc-static-content).
I'm using [Gradle](https://gradle.org/ "Gradle Build Tool") in this example to build and package the Spring Boot application, but the same can be achieved with other build tools.
The key is to remember that in addition to building the Java code, we must also create the minified bundle of our web app and copy it to the correct directory.

To integrate a Gradle build with Node, I'm using the [`gradle-node-plugin`](https://github.com/srs/gradle-node-plugin "Gradle plugin for integrating NodeJS in your build").
The following are the most interesting parts of my Gradle build script.

```groovy
// Read more about how to configure the plugin from
// https://github.com/srs/gradle-node-plugin/blob/master/docs/node.md
node {
  download = true

  // Set the work directory for unpacking node
  workDir = file("${project.buildDir}/nodejs")

  // Set the work directory for NPM
  npmWorkDir = file("${project.buildDir}/npm")
}

task appNpmInstall(type: NpmTask) {
  description = "Installs all dependencies from package.json"
  workingDir = file("${project.projectDir}/src/main/webapp")
  args = ["install"]
}

task appNpmBuild(type: NpmTask) {
  description = "Builds production version of the webapp"
  workingDir = file("${project.projectDir}/src/main/webapp")
  args = ["run", "build"]
}

task copyWebApp(type: Copy) {
  from 'src/main/webapp/build'
  into 'build/resources/main/static/.'
}
```

`appNpmInstall` is a Gradle task that runs `npm install` in the `webapp` directory.
Similar to `appNpmInstall`, the build script declares the `appNpmBuild` task that runs `npm run build` to create the minified bundle of the web app.
Finally, `copyWebApp` is a simple [`Copy`](https://docs.gradle.org/current/dsl/org.gradle.api.tasks.Copy.html) task to copy the minified bundle to the static content directory.
Feel free to test run these tasks in isolation and see how they work.
For example, running `gradle appNpmInstall` should install all of our web app dependencies and place them in `src/main/webapp/node_modules/` directory.

## Development flow

During web app development, you should start the web app in development mode.
Therefore, instead of bundling the web app inside the Spring app, let's serve it separately using the dev server.
Go to the `webapp` directory and run `npm start`.
This way, your web app will reload automatically if you make any changes in the source files.
At the same time, you should also start the Spring Boot application.

To make the development flow with Spring Boot a little more pleasant, you can also [configure `spring-boot-devtools` and enable automatic restarts](https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-devtools.html#using-boot-devtools-restart "Automatic Restart").
When configured, the Spring Boot app restarts whenever files on the classpath change.

## Build for production

To build our application for production, in addition to compiling and packaging our Java code, we must also create a minified bundle of the web app and place it inside the Jar file to one of the directories where Spring Boot serves static content.
We already have our build script configured with tasks that can install our dependencies, create the minified bundle and copy it to the correct location.
To make the packaging a bit more simpler, we could define some dependencies between the build tasks in our build script.

```groovy
appNpmBuild.dependsOn appNpmInstall
copyWebApp.dependsOn appNpmBuild
compileJava.dependsOn copyWebApp
```

This makes sure that whenever Java code is compiled, web app dependencies are also installed, minified bundle is created and the bundle is copied to a static content directory.
When you run `gradle clean build`, you don't have explicitly run any web app specific Gradle tasks.
Once the build has finished, you can start the application via command-line.

```
java -jar path/to/web-app.jar
```

Point your browser to it and you should see the newly created React app running.

## Summary

We went through how to create a basic React web app and serve it with Spring Boot.
The key is to configure our Gradle build script so that in addition to building and packaging our Java code, we also build and minify the web app and copy it to the Jar file.
Essentially, the same principles apply if you're using any other build tool (e.g. [Maven](https://maven.apache.org/)) or you want to use something else than React (e.g. [Angular](https://angular.io/)).

The example code presented in this post can be found on [Github](https://github.com/indrekots/static-content-demo).
