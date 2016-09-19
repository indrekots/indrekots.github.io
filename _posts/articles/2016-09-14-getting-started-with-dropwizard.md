---
layout: post
title: "Getting started with Dropwizard"
excerpt:
modified: 2016-09-14 10:36:38 +0300
categories: articles
tags: [dropwizard, rest, java]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: false
aging: true
---

[Dropwizard](http://www.dropwizard.io) is a library/framework for building [RESTful](https://www.sitepoint.com/what-does-restful-really-mean/) web services. In this post we're going to have a look at how to get started with developing a Dropwizard application by building a new service from scratch. We're going to go beyond the *hello world* level and explore features that are usually not covered in a basic tutorial. Overall, the article should still be easy for newcomers to follow along.

The sample project will be on Github so you can follow along. Key commits will be tagged and I'll be referencing them throughout this post.

## What is Dropwizard

* Glue code, including multiple popular libraries
  - Jersey
  - Jackson
  - Jetty
  - Metrics
  - etc.
* Deployable JAR file
* Good for microservices

## Introduce demo application domain

* Describe what it can do

## Project structure

* Generate project with Maven archetype
* Describe folder structure

## Application configuration

* Describe configuration class
  - create application configuration in a .yml file
  - use Hibernate Validator

## Application class

* Contains main

## Create a representation of domain objects

* What are representations
* Serialization

## Create a simple resource

* Introduce Jersey

## Running the application

* Demonstrate how to run the app
* Deployable (fat) JAR
* Run from command line

## Database

* Liquibase out of the box
* Hibernate

## Implement error handling

* Returning an error
* Override exception mappers

## Testing

## Add custom banner

## Admin interface

* running on a separate port
* health checks
* tasks
* metrics

## Add custom banner

## Summary
