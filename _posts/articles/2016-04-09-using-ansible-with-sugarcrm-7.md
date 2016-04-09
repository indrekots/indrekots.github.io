---
layout: post
title: "Using Ansible to manage SugarCRM 7 environments and deployments"
excerpt:
modified: 2016-04-09 21:44:00 +0300
categories: articles
tags: [ansible, sugarcrm, configuration management, deployment]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: false
---

SugarCRM is a web based customer relationship management system written in PHP. Due to how it is designed there are some difficulties getting it into source control. Additionally there are some things to keep in mind when managing multiple environments and doing deployments. I've decided to use Ansible to manage SugarCRM environments and deployments.
## Getting started

Firstly you should clone [sugarcrm-ansible](https://github.com/indrekots/sugarcrm-ansible) repository which contains playbooks which this blog post is going to refer to.

### Install Vagrant
Vagrant will be used to create a virtual machine.

* Make sure vagrant is installed
* Install the latest version of VirtualBox (or any other supported provider)

### Set up folder structure

Vagrant mounts SugarCRM from your local filesystem. The following diagram shows where Vagrant expects you to have placed Sugar's source code relative to `Vagrantfile`.

```
.
├── sugarcrm-ansible/
│   └── Vagrantfile
└── sugarcrm/

```

### Required Vagrant plugins
* [vagrant-hostsupdater](https://github.com/cogitatio/vagrant-hostsupdater)
* [vagrant-hosts](https://github.com/oscar-stack/vagrant-hosts)

If plugins are missing, vagrant commands will fail and prompt you to install any missing plugins.

### Configure variables
To configure your install, make changes in `inventory/development/group_vars/all/common.yml` and `group_vars/all/common.yml`.

### vagrant up

After setting up your folder structure, you should run `vagrant up`. A new VM is created and Ansible is used for provisioning.
You will be prompted for your sudo password since *vagrant-hostsupdater* will make changes in your `/etc/hosts` file and map a domain name to point to the newly created VM.

If Ansible finished successfully, then point your browser to the domain name you specified in your inventory and you should see a fresh install of SugarCRM.

## Deployment

### Create a deploy key
For deployments to work you should host your Sugar source code in a Git repository. Then generate a deploy key which allows you to access the repository.
Look at `inventory/development/group_vars/deploy_key.yml` for example and replace it with your key.

Configuration management should be kept in source control. You don't want to expose any secrets. Therefore you should encrypt your key.

```
ansible-vault encrypt inventory/development/group_vars/deploy_key.yml
```

When running a playbook, add the `--ask-vault-pass` flag for Ansible to prompt you for your vault password.


### Run Ansible deploy playbook
Running `ansible-playbook -i inventory/development/hosts deploy.yml -k -K --ask-vault-pass` will start a deployment to your local environment.

### Create new inventory entry
Creating a new inventory entry for a production environment is easy. Create a new folder `inventory/prod` and copy the contents of `inventory/development` into the newly created directory.
Replace all variables with values for the new environment. Make sure the `hosts` file contains correct addresses.
