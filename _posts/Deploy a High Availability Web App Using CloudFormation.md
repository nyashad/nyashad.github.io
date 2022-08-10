---
title: Deploy a High Availability Web App Using CloudFormation
layout: post
post-image: "https://nyashad.github.io/assets/images/CLOUDFORMATION.jpg"
description: The github link has some screenshots of the progress of the project during the course of deployment
tags:
- webapp
- cloudformation
- devops
---

I made use of the AWS SSM Session Manager to provide access to the web application servers.
The reason why i chose this instead of the bastion is because the Bastion;
Like any other infrastructure host, it must be managed and patched.
It accrues a cost while it is running.
While the AWS SSM Session Manager reduces your system’s attack surface while also offering greater visibility into commands issued on your hosts. 

You can view the project scope [here](https://github.com/nyashad/ha-cloudformation-webapp){:target="blank"}
