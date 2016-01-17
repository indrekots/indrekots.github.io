---
layout: post
title: "Version control and deployment practices for SugarCRM"
excerpt:
modified: 2016-01-14 19:47:25 +0200
categories: articles
tags: [sugarcrm, vcs, version control, deployment, configuration management, ansible, php, vagrant]
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

Handling database changes is more difficult. You need to know which database tables are changed and which ones need to be kept in sync. Sugar does not generate migration files. After doing some testing and googling, I came to a conclusion that at first, I should track the changes in the `fields_meta_data` table. When you create a new custom module and deploy it, Sugar creates a new database table for it and is able to recreate from the generated PHP files. But when you decide to edit a deployed module, the changes are added to the `fields_meta_data` table.

I created a `database` folder in the root of the SugarCRM folder and I'm going to add database dumps there. To make sure I'm not going to forget to to a database dump before a commit, I created a git pre-commit hook which does it for me.

{% highlight bash %}
//TODO: insert git pre-commit hook
{% endhighlight %}

Before even starting to investigate how to manage multiple environments of Sugar, I decided to first automate the creation of a clean environment. I reckoned I was going to mess up the installation at some point. So having a quick and easy way to recreate the environment was a must for me. There are many configuration management tools out there that can do the job. I decided to use [Ansible](https://github.com/ansible/ansible "Ansible github page") since I'm most familiar with it. But you can achieve the same results with [Chef](https://www.chef.io/chef/ "Chef's homepage"), [Puppet](https://puppetlabs.com/ "Puppet's homepage") or [Salt](https://github.com/saltstack/salt "Salt's github page"). In addition, I'm using Vagrant to create my local development environment. Later I'm going to show how to use Ansible for deployment of SugarCRM.

If you have ever installed an instance of SugarCRM you have probably used the web based installer. The downside is that it requires user interaction. My goal was to run a command and let Ansible to everything for me. In addition, the web based installer makes it harder to set up and deploy SugarCRM to multiple servers. You would have to go through each installer session separately and this is time consuming. Fortunately I was able to find [a handy SugarCRM silent installer script](https://gist.github.com/sadekbaroudi/f0f3c759df00ce1094f9 "Github gist of a silent installer script") which can be used to automate the installation process.

Having Ansible in place helps me to deploy changes to new environments. Previously I showed a pre-commit hook which takes a database dump of some important tables. I created an Ansible deploy playbook which checks out the latest commit from my git repoistory and applies it to the server I specify. The commit might include source code changes and database changes. If you've ever edited a custom module you know that you must click the repair button in the SugarCRM Administration menu. I don't like human interaction which gets in the way of automation. Luckily I found a [repair script](https://gist.github.com/chicks/6084088#file-repair-php "Github gist of a repair script") which can be used to automate the deployment step.

To conclude, I use Ansible to create a fully functioning SugarCRM environment. I can create new servers with ease by calling my ansible playbook. For my local environment I use Ansible together with Vagrant to create a virtual machine. In addition, Ansible is used to to deployments as well. i keep my SugarCRM files in source control using Git. A database dump of important tables is taken before each commit by a pre-commit hook. Ansible deploy playbook checks out the latest commit, applies it to the server I specify and calls the repair script. No human interaction with SugarCRM's web interface is needed during installation and deployment.
