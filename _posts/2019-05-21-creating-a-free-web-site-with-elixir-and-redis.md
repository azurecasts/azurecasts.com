---
layout: video
title: "Creating a Free Web Site with Elixir and Redis using Docker Compose"
number: "008" #set this to your episode
categories:
tags:
summary: "In this video Rob shows you how to create a multi-container web application using Docker Compose - all for Free! We'll create a site using Elixir and then wire it up to Redis, all containerized."
video: https://www.youtube.com/embed/xaTAOWMZEYA
author: Rob Conery
github: https://github.com/azurecasts/008-freedom-linux
transcript: 008-freedom-linux 
minutes: "12:42"
twitter: robconery
---

In episode #006 I told you that we couldn't use Azure's free tier, _F1_, unless we used the Windows operating system. Not a big deal if we're using Node and are happy to use Azure's canned hosting engine. But what if we want to do our own thing with Docker? Unfortunately that meant we were out of luck because Docker requires the Linux operating system, which wasn't free.

Until 2 weeks ago! All of that changed when Scott Guthrie announced a free tier for Linux. This is great for us because we can now push Docker containers to Azure for free!

## Resource Links

 - [Multi-container App Services](https://docs.microsoft.com/en-us/azure/app-service/containers/quickstart-multi-container?WT.mc_id=docs-azurecasts-robcon)
 - [Configuring an App Container](https://docs.microsoft.com/en-us/azure/app-service/containers/configure-custom-container?WT.mc_id=docs-azurecasts-robcon)
 - [What is Kudu?](https://azure.microsoft.com/en-us/resources/videos/what-is-kudu-with-david-ebbo?WT.mc_id=docs-azurecasts-robcon)
 - [AzureCasts on Free Web Apps](https://azurecasts.com/2019/05/03/creating-a-web-app-for-free/)
 - [AzureCasts on Makefiles](https://azurecasts.com/2019/04/11/005-using-make-to-orchestrate-shell-scripts/)