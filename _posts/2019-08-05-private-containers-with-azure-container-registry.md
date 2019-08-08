---
layout: video
title: "Private Containers with Azure Container Registry"
number: "011" #set this to your episode
categories:
tags:
summary: "In this episode we move our docker image to a private docker repository hosted by Azure. We'll make sure to keep the registry private as well. Finally, we'll ask Azure to build our image for us using ACR Tasks."
video: https://www.youtube.com/embed/Prrik2jgYyg
author: Rob Conery
github: 
minutes: "14:13"
twitter: robconery
---

Put the long form description of the show here

## Resource Links

You can read more about the Azure resources in this video here:

 - [Azure Container Registry](https://azure.microsoft.com/services/container-registry/?WT.mc_id=azurecasts-website-robcon)
 - [Creating a Private Container Registry](https://docs.microsoft.com/azure/container-registry/container-registry-get-started-azure-cli?WT.mc_id=azurecasts-website-robcon)
 - [Build and Run a Container in the ACR](https://docs.microsoft.com/azure/container-registry/container-registry-quickstart-task-cli?WT.mc_id=azurecasts-website-robcon)

## Transcript

I've been working with Azure's free App Service tier for Linux, which has been a lot of fun cause I like free things. All I've had to do is to send up a docker-compose file, which pulls an image from Docker Hub containing my Elixir app and also pulls a Redis image, which is my app's database.

{% include screenshot.html img="screenshot_102" %}

This works, but the repository is public, which isn't ideal. In the real world we probably don't want to allow Joe or Jill Public to download and run our app - so I need to fix that today by making my Docker Repo private.

I can do this on Docker Hub easily by flipping a switch on my repo to make it private. As you can see, I'm allowed 1 private repo with my current free plan. If I want more repositories I need to pay - which is fine by me as the price is reasonable, starting at $7/mo for 5 private repositories.

{% include screenshot.html img="screenshot_103" %}

{% include screenshot.html img="screenshot_104" %}

But I like alternatives and I especially like the idea of keeping all my application assets in a single place - the Azure data centers... I also want to continue with my job-continuation program so today I'm going to dive in and take a look at Azure's Docker Hub alternative: The Azure Container Registry, or ACR. 

ACR pricing is similar to Docker Hub, depending on which tier I choose. The Basic tier comes out to right around $5/mo with 10G of storage size. I can bump that to the Standard tier which is roughly $20/mo - which I think is pretty reasonable.

{% include screenshot.html img="screenshot_105" %}

So, in today's episode, I'll create my very own Docker registry on Azure. I'll build my app's image, which is a super simple Elixir web app that connect's to Redis, and then push it to my new Azure repository. I'll also get funky with Azure Container Registry's tasks - making it build my code for me.

Lot's to do, let's go!

## Creating a Registry
Here's my application which I pushed to Azure in episode 008 - deploying a free web app with Linux. This is a multi-container site which is running Elixir as well as Redis. Here's my docker-compose file and here's how I pushed it all to Azure: using _Make_.

{% include screenshot.html img="screenshot_106" %}

I want to use ACR to host my container, so the first thing I need to do is investigate how that's done. I'm not a portal person - I prefer the Azure CLI as I find it to be, basically, a text browser. If I type in `az` here on the command line I can see all kinds of commands. The one I'm looking for is for the container registry - and as I expected, there it is.

{% include screenshot.html img="screenshot_107" %}

Let's investigate how to use the `acr` subcommand, and I'll do what I typically do by accessing the Azure CLI help system using `--help` following any command. As always, there are a bunch of commands that I could use to distract myself, but I want to stay on target - and that target is creating an ACR registry. As I've come to expect, there's a `create` command right here.

{% include screenshot.html img="screenshot_108" %}

I'll use the help system once again to see what's needed to use the `az acr create` command and right at the bottom I can see a very helpful example. I think this is all I need - I just need to give the registry a name, resource group and a SKU:

{% include screenshot.html img="screenshot_109" %}

Before I do any of that I'll want to create a resource group for my registry. I'll call mine `011` for this episode:

```
az group create -n 011
```

Before we go any further I think it's a good idea to point out that having your container registry _in the same resource group_ as your app code might not be a good idea. In previous episodes I've wholesale dropped a resource group to clean up everything in a failed deployment. I might not want to do that with a container registry! Exercise caution here.

OK - let's move on to creating our registry. I want to run the `az acr create` command, but what should the SKU argument be? I can see "Standard" in the example here and I think I saw "Basic" as a choice before... but I want to be sure. I'll have a look at the help command one more time to be sure I'm not missing anything:

{% include screenshot.html img="screenshot_110" %}

OK - I see the SKUs - Basic, Classic, Premium and Standard. I'll choose Basic as the price works for me. But what's this? It seems like `--admin-enabled` might be something I need to understand before moving forward.

For this, let's head to the documentation. I got here by doing a quick Google search on "az cli admin-enabled". Looks like the idea is to allow simple access rather than going through the more complex option of using Azure's Active Directory and Identity Principle. For larger customers, having more control over access is a good idea. For my purposes, simple access is better, so I'll set `--admin-enabled true`.

This looks like a good command:

```
az acr create -n 011 -g 011 --sku Basic --admin-enabled true
```

Uh oh...

{% include screenshot.html img="screenshot_111" %}

Looks like my name is a bit off. The name `011` for a container registry is a bit silly anyway - I should add some details. How about...

```
az acr create -n 011-registry -g 011 --sku Basic --admin-enabled true
```

{% include screenshot.html img="screenshot_112" %}

Crap. Another bad name - looks like I violated a regex naming policy that wants me to use only alphanumeric characters. That makes sense - your container _registry_ is a place that will hold all of your Docker repositories and therefore should have some degree of organization. A good naming strategy might be something like `company-department-group`, or perhaps a project name.

I think, for me, it would be nice to have a registry where I could push the images for AzureCasts, so that's what I'll call mine:

```
az acr create -n azurecasts -g 011 --sku Basic --admin-enabled true
```

{% include screenshot.html img="screenshot_113" %}

## Registry Authentication

Alright! It worked that time - we have ourselves a registry. But how do we log in? Reading the docs again - all we have to do is to tell the Azure CLI to show us the credentials:

{% include screenshot.html img="screenshot_114" %}

Perfect. I'll copy paste that command and pop in the name of my registry and there they are: my username and 2 passwords:

{% include screenshot.html img="screenshot_115" %}

By the way, yes, I rerolled these passwords in fact I did it just now using `az acr credential renew`.

OK, before I forget - I need to store this password so I can access my registry. The best place for this is my `.env` file, which will add the variable `ACRPW` to my environment variables because I have the dotenv plugin for z-shell.

{% include screenshot.html img="screenshot_116" %}

There's one more step for us, and I was tempted to let us find out the hard way but this video is already pretty long so I'll get right to it. We created a private container registry on Azure with simple administrative access. We need to tell Docker about that access, otherwise we'll get an error. I originally thought that Azure would just know who I am because I'm authenticated, so everything would just work. Unfortunately, Docker (and the Docker CLI) doesn't care about my Azure credentials - it's a completely separate tool.

That means I need to tell Docker about my new registry and how to authenticate with it. I can, if I want, do a `docker login` and give it the `username` and `password` along with the registry URL. OR I could have the Azure CLI do it for me using:

```
az acr login -n azurecasts
```

{% include screenshot.html img="screenshot_117" %}

Great! Azure told the Docker CLI about my repo and also created an authentication token good for one hour. If you want to read more about this, head over to the documentation.

{% include screenshot.html img="screenshot_118" %}

## Pushing Our First Image

Now we get to build our project and push it. For this, I'll need some more information from Azure - specifically the URL of the server we're going to push to. If I run `az acr show -n azurecasts` I'll get all the information I need:

{% include screenshot.html img="screenshot_119" %}

Docker uses a conventional naming strategy to know where to push your image that basically resembles a URL. The default repository is up at Docker Hub - so if I use "robconery/011-elixir" as the tag for my project image, the Docker CLI will assume I want to push to Docker Hub - specifically to robconery's account.

If, however, I use the `loginServer` setting here, which is a qualified hostname, the Docker CLI will try to push there. To see how this works, let's build our image:

```
docker build -t azurecasts.azurecr.io/011-elixir .
```

Make sure you spell the name correctly, if you don't your push will fail and you'll probably have no idea why. Also - as always, don't forget the ".".

{% include screenshot.html img="screenshot_120" %}

Great. Our project is built, now we just need to push and I can do that using:

```
docker push azurecasts.azurecr.io/011-eilxir:latest
```

And up it goes!

{% include screenshot.html img="screenshot_121" %}

Let's be sure it's there. If I run `az acr repository list -n azurecasts` I can see our repo... great!

{% include screenshot.html img="screenshot_122" %}

## Fixing Up Deployment

We have a private Docker registry... yay us! Our dev image is safe and secure behind a private repo which makes life better for us. Now I need to update my deployment strategy.

The first thing to do is simple: I just change where my image is being pulled from in my `docker-compose-azure.yml` file, resetting the repository to my Azure registry.

{% include screenshot.html img="screenshot_123" %}

I also need to update my Makefile. I'll add a variable with my container registry name at the top here, and use it along with my `APPNAME` to create the full tag of my image. 

{% include screenshot.html img="screenshot_124" %}

Now comes the fun part - I need to do two things:

 1. I need to give Azure the permissions it needs to pull the image during deployment and
 1. I need to enable continuous deployment so whenever the image changes, my app is updated

The first thing I'll do is use the help system for `az webapp config` as I'm guessing the changes I need are in the configuration somewhere. I can see right here there's a subcommand for containers:

{% include screenshot.html img="screenshot_125" %}

How do I use this? I don't know! Let's ask the help system. I'll ask for help with the `az webapp config container set` command and here we go! We've seen this before, back in episode 008 when I deployed multiple contianers for free. This time I'm most interested in the `docker-registry` arguments:

{% include screenshot.html img="screenshot_125" %}

Looks like I can use these to identify the server, username and password. Perfect. The password is the easiest one - if you recall I saved that in my `.env` file which pops it into my environment variables. That means I can just set `$(ACRPW)` here and be done with it. The server URL needs to be fully qualified, which means it needs to start with `https://`. The rest of the domain is just my username plus "azurecr.io". Finally: the user name is simple `azurecasts`, the name of my registry, which I can specify using `$(ACR)`.

{% include screenshot.html img="screenshot_128" %}

The last thing to do is to make sure I specify the resource group using `-g` and we're good to go. Almost. While I'm here let's tidy a few things up by fixing some indentation to make the commands a bit more readable. I'll also add a log target so we can see what's going on.

Outstanding! Well... let's see if this works...

## Deployment

I'll execute my Makefile by running `make` in my shell, which seems to have gone off OK:

{% include screenshot.html img="screenshot_129" %}

The initial load of the site is going to take some time, as it normally does, because Azure is pulling down the image and setting things up for us. If we take a look at the logs, we can see our image is being pulled down and the container created.

{% include screenshot.html img="screenshot_131" %}

After a couple of minutes - we're up!

{% include screenshot.html img="screenshot_132" %}

This is a multi-container deployment running on the Linux free tier, pulling from the Azure Container Registry (which, to be clear, is _not_ free, but is still worth every penny). OK this is neat and all but I need to do the second thing we came here to do: _set up continuous deployment_. I could make life easy on myself and just flip this here switch, but hey! Let's have a script command do it for us!

Quickly: I need to get some help with deployment of my webapp, so I'll ask Azure:

{% include screenshot.html img="screenshot_133" %}

I start with `az webapp deployment --help` and then move down the subcommand chain until I end up with `az webapp deployment container config --enable-cd true`. This definitely _could be easier_ to find, but hey it didn't take me that long.

{% include screenshot.html img="screenshot_135" %}

Now I need to add this target to my Makefile, which I'll call `enablecd`, setting `config` as a prerequisite and then resetting the `logging` target to have `enablecd` as a prereq.

{% include screenshot.html img="screenshot_136" %}

Rather than drop everything and run again, I'll just execute this command from the command line:

```
az webapp deployment container config --enable-cd true -n 011-elixir -g 011
```

That works just fine and what we get back is interesting:

{% include screenshot.html img="screenshot_137" %}

We have a `CI_CD_URL`, which sort of looks like a webhook doesn't it? I'll get to that in a second. If we head back to the portal and refresh, we can see that we're now set up for continuous deployment.

## Updates

Let's make sure updates go off as we expect. Here in my Elixir app I'll change the greeting on the home page to a different message. Next, because I'm lazy and I love Makefiles I'll add two targets - the first called `build` which builds our image locally and the next called `push` which does a `docker push`.

Let's run those and push our updated image to Azure:

{% include screenshot.html img="screenshot_139" %}

{% include screenshot.html img="screenshot_140" %}

Uh oh. Looks like I've been talking too much and we've been logged out. This is kind of hard to see, don't you think? We can get around this easily by reauthenticating ourselves. Whenever we do this we have a one hour window to push our new images to our registry.

```
az acr login -n azurecasts
```

Pushing again and off we go. While that's happening, let's pop over to the portal once again and checkout our registry's webhooks. You can find them by clicking on "Webhooks" in the menu on the left. On the right we can see "webapp011elixir", which obviously seems like my web app. 

{% include screenshot.html img="screenshot_141" %}

If I click on that I can see two actions down below, representing the two pushes I've made to the repository. You can click through to see details if you're bored...

{% include screenshot.html img="screenshot_142" %}

Neat. Let's go see if our app updated yet. It's been a total of 2.5 minutes or so and yes! It has:

{% include screenshot.html img="screenshot_143" %}

## ACR Tasks

The ACR has the notion of "tasks" - little Docker jobs it can perform for you. I'm not going to jump into those, aside form this one here, which is incredibly useful:

{% include screenshot.html img="screenshot_144" %}

The build tasks will take your code, load it to Azure, and then execute a `docker build` command _using Azure resources_. That's kind of neat, and opens the door to the wonderful, exciting, and never-ending world of DevOps. 

We can ask for help with this command and see all kinds of interesting responses using

```
az acr build --help
```

There are a lot of choices, but I want to keep things simple and on-target so I'm going to ask Azure to build my local directory using my "azurecasts" registry and an image name (which is also a repository name) of `011-elixir`. Once again: don't forget the dot! That tells the command where to find your files. Wouldn't it be nice if that was the dang default?

{% include screenshot.html img="screenshot_145" %}

The Azure CLI zips up our code and sends it up to Azure - including our Dockerfile, which is the most important part. The _exact same_ build process that happened on our local drive is now going off in Azure somewhere, on one of its many servers. I don't need to devote resources to building this thing, which is nice.

{% include screenshot.html img="screenshot_146" %}

After a few minutes it's all finished, and our webhook fires letting our App Service know to pull the updated image and deploy. 

{% include screenshot.html img="screenshot_147" %}

I can replace my push command with the `az acr build` command if I want to - it's up to you and what makes the most sense for your project.

Well that's it for me! Hope you've enjoyed this episode and I'll see you soon.
