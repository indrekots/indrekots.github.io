---
layout: post
title: "Accessing EC2 Instances with AWS SSM"
excerpt: "Discover how to access your EC2 instances with AWS Systems Manager (SSM), eliminating the need for complex networking setups or SSH key management."
modified: 2024-04-07 12:05:18 +0200
categories: articles
tags: [aws, ec2, ssh]
image:
  path: /images/2024-04-07-access-ec2-instance-with-aws-ssm/cover.jpg
  thumbnail: /images/2024-04-07-access-ec2-instance-with-aws-ssm/cover_thumb.jpg
  caption: "Photo by [Kaffeebart](https://unsplash.com/photos/a-close-up-of-a-padlock-on-a-door-KrPulSdUetk)"
comments: true
share: true
published: true
aging: false
amazon_links: false
---

The traditional way of accessing servers is to use SSH.
This, however, comes with configuration overhead.
First of all, the server has to be accessible over the network.
You need to carve out a path for SSH to work.
This can bring extra challenges, especially in an environment where you want to keep the server in a private network.
Secondly, you're faced with the overhead of managing SSH keys.
This post will show you how to use AWS Systems Mananger (SSM) to start a remote session to an EC2 instance, eliminating the need to configure SSH.

## IAM Instance Profile

To be able to start an SSM session to an EC2 instance, Systems Manger needs to have access to your instances.
We can do that with IAM roles.

1. Go to the IAM Console and create a new Role
2. For the trusted entity type, select **"AWS service"** and select EC2 for the use case
3. When configuring permissions, select the AWS managed **"AmazonSSMManagedInstanceCore"** policy
4. Finally, give the role a name, say, **"SSMAccess"**

## Create AWS EC2 Instance

When you launch a new EC2 instance, don't forget attach the **SSMAccess** IAM instance profile.

<p class="notice--warning">
<i class="fas fa-exclamation-triangle"></i>
<strong>Keep in mind</strong> that the EC2 instance needs to have the SSM agent installed. <a href="https://docs.aws.amazon.com/systems-manager/latest/userguide/ami-preinstalled-agent.html">It's preinstalled on some Amazon Machine Images (AMIs) provided by AWS</a>.</p>

And that's it.
The newly launched EC2 instance should be accessible once it's fired up.
However, if you wish to access an EC2 instance that doesn't have a public IP or doesn't have a route to the [Internet Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html), additional configuration is required.

## Configure Access to Private EC2 Instances

We need to create VPC endpoints to be able to access private EC2 instances via SSM.

1. Go to the VPC console and select Endpoints
2. Create a new Endpoint and give it a name
3. Select AWS services from the service category
4. Select the `com.amazonaws.[region].ssm` service
5. Select your VPC and associate the endpoint with the subnets you're using
6. In security groups, you need to select a security group that allows outbound HTTPS (port 443) traffic to Systems Manager endpoints. Create one if it doesn't already exist. Keep in mind that this is the same security group that should be associated with the EC2 instance.
7. Finally, create the endpoint
8. Repeat the steps to create another endpoint, but this time select the `com.amazonaws.[region].ec2messages` service
9. Repeat the steps again to create an additional endpoint, but this time select the `com.amazonaws.[region].ssmmessages` service

## Start Session via `awscli`

`awscli` can be used to start a session with the EC2 instance.
Before you do so, make sure you have [installed the Session Manager plugin for AWS CLI](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html).
Once that's done, you should be good to go.

```bash
aws ssm start-session --target <ec2-instance-id>
```

If everything is configured correctly, you should have a newly established terinal session to your EC2 instance.

Alternatively, you can also start a session using the AWS EC2 console.
Select the EC2 instance and click the "Connect" option.
Select "Session Manager" and click "Connect" again.
This will open a terminal session in your browser.