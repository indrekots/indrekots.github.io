---
layout: post
title: "Version control and deployment practices for SugarCRM"
excerpt:
modified: 2016-01-14 19:47:25 +0200
categories: articles
tags: [sugarcrm, vcs, version control, deployment, configuration management, ansible, php]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: false
---

* intro, not an expert
* problems with vc, changes in source code and database, both need to be in sync
* git to the rescue, .gitinore
* git hooks for database export
* deployment -> git pull, apply database scripts
* automate -> ansible
* silentInstall, repair scripts -> no gui interaction
* other tables that need to be kept in sync -> workflows and expressions

SugarCRM is a web based customer relationship management system written in PHP. I'm fairly new to this piece of software and started to figure out how to have multiple environments for SugarCRM. My plan was to have my own local development environment, a staging and a production environment. In SugarCRM, users are allowed to define their own custom modules. There are essentially new tables in the database. They can define data types and add constraints to new fields. In addition to changes in the database, SugarCRM generates new php files as well. In order to keep 2 environments in sync, both file system level changes and database changes need to be transferred.

Source code should be kept in a version control system. I decided to use Git. But not all SugarCRM files need to be version controlled. For example the `/cache` directory can be excluded. This is the directory where SugarCRM keeps its runtime code.

Concept of modules, table and fields metadata table. Original table is generated from vardefs file. This is a php file.

Not all database need to be kept in sync.
