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
published: true
---

* intro, not an expert
* problems with vc, changes in source code and database, both need to be in sync
* git to the rescue, .gitinore
* git hooks for database export
* deployment -> git pull, apply database scripts
* automate -> ansible
* silentInstall, repair scripts -> no gui interaction
* other tables that need to be kept in sync -> workflows and expressions

SugarCRM is a web based customer relationship management system written in PHP. I'm fairly new to this piece of software and started to figure out how to manage changes and multiple environments of SugarCRM. My plan was to have my own local development environment, a staging and a production environment. In SugarCRM, users are allowed to define their own custom modules. There are essentially new tables in the database. They can define data types and add constraints to new fields. In addition to changes in the database, SugarCRM generates new PHP files as well. In order to keep 2 environments in sync, both file system level changes and database changes need to be transferred. While being very flexible towards its users, SugarCRM is at first glance difficult to handle by operations.

I'm going to rephrase to problem a bit more clearly. Users are able to basically create new database tables and generate new PHP files without them knowing what is happening in the background. If I have my local Sugar environment and create a new module, I need to have this module be present in the staging environment for testing as well. This means the new PHP files and database changes need to be applied to the staging environment. Without knowing much about SugarCRM, this seems to be an easy task. Source code changes can be handled by version control and database changes could be handled by some kind of database migration tool (e.g. [Flyway](http://flywaydb.org/ "Flyway homepage")). But as usual, life is not as easy as it seems.

When it comes to source code changes, yes, version control is the way to go. But there are some things to keep in mind. You should not add all SugarCRM's files to source control or you'll end up in merge conflict As far as I understand, Sugar does some magic and generates new source files in the `/cache` directory which is used during runtime. So that folder can be excluded from version control. But there are others as well.

![Merge conflict]({{ site.url }}/images/2016-01-14-version-control-and-deployment-practices-for-sugarcrm/meme-merge.jpg "Merge conflict")

I'm using [Git](https://git-scm.com/ "Git source code management homepage") for version control. The following is my `.gitignore` files which instructs Git from not adding the specified files and folders to version control. As you can see, a lot of files can be ignored.

{% highlight bash %}
//TODO: insert .gitignore
{% endhighlight %}

Handling database changes is more difficult. You need to know which database tables are changed and which ones need to be kept in sync. Sugar does not generate migration files. After doing some testing and googling, I came to a conclusion that at first, I should track the changes in the `fields_meta_data` table. When you create a new custom module and deploy it, Sugar creates a new database table for it. But when you decide to edit a deployd module, the changes are added to the `fields_meta_data` table.

git hook

Concept of modules, table and fields metadata table. Original table is generated from vardefs file. This is a php file.

Not all database need to be kept in sync.
