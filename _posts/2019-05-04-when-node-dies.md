---
layout: video
title: "When Node Dies"
number: "007" #set this to your episode
categories:
tags:
summary: "Node is single-threaded and when the process dies, *everything dies*. Frameworks like Express and Koa have safeguards built in, but most of the time it's a good idea to use a process manager, like PM2. In this video, Burke blows his site up to show us the details."
video: https://www.youtube.com/embed/hYdSfH-0tVI
author: Burke Holland
github: https://github.com/azurecasts/007-when-node-dies
transcript: 
minutes: "12:11"
twitter: burkeholland
---

An unhandled exception in a Node application can crash the whole process, taking your site offline completely. _Even if_ you try to trap the error in a `try/catch` block, it can still find a way to die. The best way to handle this is to, as Erlanger's like to say, _let it crash_ and restart it.

In this video Burke Holland will show you how Azure restarts a dead Node application using Docker container orchestration. There's a cheaper way to do that, however. **Azure uses PM2** to run Node applications, which means you can tweak the restart rules and even create multiple Node processes in a cluster!

## Resource Links

You can read more about the Azure resources in this video here:

 - [PM2](http://pm2.keymetrics.io)
 - [Azure Scaling](https://docs.microsoft.com/en-us/azure/app-service/web-sites-scale?WT.mc_id=azurecasts-website-buhollan)
 - [VS Code Azure Extenstions](https://code.visualstudio.com/docs/azure/extensions?WT.mc_id=azurecasts-website-buhollan)
 - [Design your app to scale out](https://docs.microsoft.com/azure/architecture/guide/design-principles/scale-out?WT.mc_id=azurecasts-website-buhollan)
 - [Deploying to Azure using AppService](https://code.visualstudio.com/tutorials/app-service-extension/getting-started?WT.mc_id=azurecasts-website-buhollan)
