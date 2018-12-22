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
You also want a backend so you picked Spring Boot for the job.
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

## Bundle React app inside Spring Boot jar

To serve your frontend web app from within the jar file, we need to understand how Spring Boot serves static content.
https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-developing-web-applications.html#boot-features-spring-mvc-static-content

## Gradle build script

## Summary
