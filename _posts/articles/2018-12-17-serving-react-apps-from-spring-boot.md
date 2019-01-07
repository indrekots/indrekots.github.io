---
layout: post
title: "Serving React apps from Spring Boot"
excerpt:
modified: 2018-12-17 18:56:18 +0300
categories: articles
tags: [react, web, spring, spring boot, java, gradle]
image:
  path: /images/2019-01-05-spring-boot-reactjs/cover.jpg
  thumbnail: /images/2019-01-05-spring-boot-reactjs/cover_thumb.jpg
  caption: "Photo background by [Samuel Scrimshaw](https://unsplash.com/photos/fkS-me35j7I)"
comments: true
share: true
published: false
aging: true
---

Let's say you've decided to create a web app with [React](https://reactjs.org/ "A JavaScript library for building user interfaces").
You also want to build a back end so you've picked [Spring Boot](https://github.com/spring-projects/spring-boot) to do the job.
There are lots of ways to serve a React app in production.
In this post, we're going to take a look at how to bundle the React app inside a Spring Boot back end.

## Spring Boot app

Go to [start.spring.io](https://start.spring.io/ "Spring Initializr") and generate yourself a new Spring Boot app (or use an existing one if you wish).
Make sure you at least pick the Web dependency.
You should have the following folder structure:

// insert folder structure here

## Create React app

It's super easy to get started with React with [`create-react-app`](https://facebook.github.io/create-react-app/ "Create React App").
If you have the necessary tools installed (e.g. Node, npm), you can just execute this command

```
npx create-react-app enter-app-name-here
```

## Serve static content with Spring Boot

To serve your frontend web app from within the jar file, we need to first understand how Spring Boot serves [static content](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-developing-web-applications.html#boot-features-spring-mvc-static-content).

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
Now that we know how to serve static content, it should be relatively easy to place the React app in the same folder.

*But wait! Javascript source files have to be built first, CSS minified etc.*
That's correct.
We have to build the React app before we can move it into the `/static` directory.
Let's have a look at how to do that.

## Gradle build script

Before we can put our web app to production, [we must create a minified bundle with `npm run build`](https://github.com/facebook/create-react-app#quick-overview "Create React App Quick Overview").
To serve the minified bundle with Spring Boot, we have to move it to one of the directories where [Spring Boot serves static content](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-developing-web-applications.html#boot-features-spring-mvc-static-content).
I'm using [Gradle](https://gradle.org/ "Gradle Build Tool") in this example to build and package the Spring Boot application, but the same can be achieved with other build tools.
The key is to remember that in addition to building the Java code, we must also create the minified bundle of our web app and copy it to the correct directory.

*If the minified bundle will be placed in one of the static content directories, where will our React source files live?*
That's a good question.
The answer is really up to you.
In this example, I will put my React source files to `src/main/webapp` directory and use Gradle to create the minified bundle which is then copied to another directory.

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

    // Set the work directory for Yarn
    yarnWorkDir = file("${project.buildDir}/yarn")
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
Feel free to test run these tasks in isolation and see what happens.
For example, running `gradle appNpmInstall` should install all of our web app dependencies and place them in `src/main/webapp/node_modules/` directory.

## Development flow

During web app development, you should start the web app in development mode.
Therefore, instead of bundling the web app inside the spring app, let's serve it separately using the dev server.
Go to the `webapp` directory and run `npm start`.
This way, your web app will reload automatically if you make any changes in the web app source files.
At the same time, you should also start the Spring Boot application.

To make the development flow with Spring Boot a little more pleasant, you can also [configure `spring-boot-devtools` and enable automatic restarts](https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-devtools.html#using-boot-devtools-restart "Automatic Restart").
When configured, the Spring Boot app restarts whenever files on the classpath change.

## Build for production

To build our application for production, in addition to compiling and packaging our Java code, we must also create a minified bundle of the web app and place it inside the Jar file in the correct directory.
We already have our build script configured with tasks that can install web app dependencies, create the minified bundle and copy the bundle to the correct location.
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
The key is to configure the build script so that in addition to building and packaging the Java code, we also build and minify the web app and copy it to the Jar file.
Essentially, the same principles apply if you're using any other build tool (e.g. Maven) or you want to use something else than React (e.g. Angular).
