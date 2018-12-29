---
layout: post
title: "Serving React apps from Spring Boot"
excerpt:
modified: 2018-12-17 18:56:18 +0300
categories: articles
tags: [react, web, spring, spring boot, java, gradle]
image:
  path:
  thumbnail:
  caption:
comments: true
share: true
published: true
aging: true
---

You want to create a web app with React.
You also want a backend so you picked Spring Boot to do the job.
It's very easy to start the backend Spring Boot app and the React app separately.
Spring Boot app's can be easily deployed with a single executable jar file.
In this post, we're going to take a look how to bundle the React app inside the jar file.

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

I'm using Gradle in this example, but the same can be achieved with other build tools.
The key is to remember to copy the static content to a specific folder in the jar file.

## Dev flow

During web app development, we benefit from hot reloading.
Therefore, instead of bundling the web app inside the spring app, let's serve it separately using the dev server.

## Build for production

When building your back-end for production, build the final jar file and make sure the static content is bundled inside the jar file.
The static content for the web app is served from the Spring Boot app when you start it with `java -jar`

## Summary
