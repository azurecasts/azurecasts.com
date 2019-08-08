---
number: "001"
title: A Simple Docker Deployment
summary: In this video we’ll kick the tires on Azure and see if it will work as the production home of project I’ll be working on. I don’t have an application just yet, but one is in the planning. 
video: https://www.youtube.com/embed/z5Kdrw2KsIg
author: Rob Conery
layout: video
minutes: "13:39"
transcript: 001-simple-node-deploy
github: https://github.com/azurecasts/001-a-simple-plan
twitter: robconery
---

In this video we’ll kick the tires on Azure and see if it will work as the production home of project I’ll be working on. I don’t have an application just yet, but one is in the planning. 

I want to start by doing the simplest thing I can think of: deploying the sample code from the Node “Getting Started” guide. I don’t want to deal with dependencies, build processes, or unit tests just yet. I want to do the least possible thing so I can isolate any problems I run into.

## Resources

 - [The Node Getting Started Guid](https://nodejs.org/es/docs/guides/getting-started-guide/)
 - [The Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest&WT.mc_id=azurecasts-website-robcon)
 - [Docker](https://docs.docker.com/)
 - [az webapp](https://docs.microsoft.com/cli/azure/webapp?view=azure-cli-latest&WT.mc_id=azurecasts-website-robcon)
 - [App Service Pricing](https://azure.microsoft.com/pricing/details/app-service/windows/?WT.mc_id=azurecasts-website-robcon)
 - [az appservice](https://docs.microsoft.com/cli/azure/appservice?view=azure-cli-latest&WT.mc_id=azurecasts-website-robcon)
 - [az group](https://docs.microsoft.com/cli/azure/group?view=azure-cli-latest&WT.mc_id=azurecasts-website-robcon)


## Transcript


## Step 1: Setting Up The App
Here’s the app I’ll work with, the canonical “Hello World” web application from the NodeJS website. I’ll copy this code directly into VS Code without changing a single thing.

{% include screenshot.html img="001" %}

Next, I’ll run it to be sure everything works:

{% include screenshot.html img="002" %}

Not much of a surprise here, it works as expected:

{% include screenshot.html img="003" %}

## Step 2: The Tools

My friend K. Scott Allen suggested that developers think of Azure as a “programmable data center”, which I quite like. I’m a big fan of working with command line interfaces and, if I’m being honest, the Azure portal can be a little difficult to figure out at times.

For this video I’ll be leaning completely on the command line interface, but rather than rattle off commands from memory, I’ll show you the process I used to discover how to use this powerful tool.

The first step is to be sure you have the CLI tools installed. You can download them online easily.

{% include screenshot.html img="004" %}

The next step is to set your default subscription. Most people I know only have a single Azure subscription, but it’s possible to have more than one tied to your Azure account. If you do, then be sure to set your default subscription using this command.

{% include screenshot.html img="005" %}

Finally, I’ll be working on Docker today. If you’re new to Docker, don’t worry as I won’t be doing too many crazy things. However you will need it installed if you plan on following along.

{% include screenshot.html img="006" %}

## Step 3: Containerizing Our Application

If you’ve never worked with Docker, this might look a bit weird. I would encourage you to pause the video and watch a quick introduction on YouTube if you need.

The first thing that I typically do when starting with a web application is to create a Dockerfile and also a Makefile. You don’t need to use Make, I like it simply because it helps with repetitive tasks - like entering Docker commands.

{% include screenshot.html img="007" %}

I’ll fill out the Dockerfile using the simplest possible commands, pulling the image for NodeJS, copying my code file into the container, exposing port 3000 which is the port under which my application will run, and finally creating the default entry command.

{% include screenshot.html img="008" %}

It’s important to me that I have the least possible amount of code and orchestration at this point because I want to get the “feel” of Azure as clearly as I can.

I’m a big fan of using shell scripts, but typing them is something I’m not terribly fond of. That’s why I like to use Make for common tasks that need to happen in a particular order. I’ll add two targets: _build_ and _run_:

{% include screenshot.html img="009" %}

I’m going to name my application “velzy”, after the great Dale Velzy, one of the pioneers of surfing that used to live right down the road from me.

I’ll build the application by executing `make build`:

{% include screenshot.html img="010" %}

Once that’s done, I’ll run it with `make run`:

{% include screenshot.html img="011" %}

Let’s see if that worked! Oops:

{% include screenshot.html img="012" %}

This is a very common issue when you move your code into a Docker container: understanding the difference between ports on the virtual machine running your containers and the host machine, running Docker.

The problem I’m having is that Node is binding my app to `127.0.0.1` over port 80. This, unfortunately, is not translating correctly to the host machine. To fix this, I need to bind to all IP addresses within the virtual machine. To do that, I simply need to change my code to specify the host as `0.0.0.0`:

{% include screenshot.html img="013" %}

Great. Now I just need to stop my container, rebuild the image, and then run it once again. It works!

{% include screenshot.html img="014" %}

The final step is to publish the container to a public repository. I’ll use DockerHub for this for now, just to test things out. Later on, I’ll want to be sure this image is private, either at DockerHub or using a custom registry. We’ll deal with that in a later episode.

{% include screenshot.html img="015" %}

## Step 4: Figuring Out the CLI

I have the Azure command line interface - the CLI - installed and I now get to figure out how to use it. If you’re used to visual tooling, the text-based programming interface is going to seem a little weird.

So let’s start with the simplest thing: we’ll execute the command `az` and see what Azure has to say to us:

{% include screenshot.html img="016" %}

Most CLIs will list off all of the commands that you can execute, as well as a description. If the CLI is small you’ll also typically see various options and flags that you can set. 
There are a lot of commands here, but the key is to not get overwhelmed, which is easy to do. I’m trying to find out how to create a web application so let’s look for the word “web” - and here it is right at the bottom:

{% include screenshot.html img="017" %}

It would be nice if the description was a bit more detailed, but for now it looks like this is what I need. Let’s run the command `az webapp` and see what happens:

{% include screenshot.html img="018" %}

We get an error message telling us that a subcommand was required. This is how the Azure CLI is constructed: there are parent commands that each have a series of subcommands to them. Reading the list here, we can see that just about every task we can think of is covered with regards to web applications.

We can get into those later. Right now I want to create a web application so I’ll use `az webapp create` and follow the CLI’s suggestion of using `-h` (a standard way of asking for help with a CLI):

{% include screenshot.html img="019" %}

The result is *very* useful. I can see at the very bottom what a successful command might look like. The options, however, don’t mention anything about deploying a container. Scrolling up… and I can see two things that are very useful:

{% include screenshot.html img="020" %}

The first is that an application name is required, which makes sense. I also need  a “plan”, which appears to be an “app service plan” according to the details. The final thing I need is to create a resource group. If this is my first time working with Azure, I have no idea what any of these things are. 

At this point I could quickly search the internet for answers to these questions, but I’d rather see how far I can go with the CLI before I get lost down any rabbit holes.

## Step 5: Create a Resource Group

The things you create inside of Azure are called “resources”, so it seems pretty straightforward that a resource *group* is a way to keep things tidy within Azure. We can find out how to create one by executing `az` one more time and looking for a command with the words “resource” or “group”. Scanning down the list, I see the `group` command right here, and it appears to be what I need:

{% include screenshot.html img="021" %}

I’ll ask the CLI for help with the `group` command, and it shows me a list of all of the subcommands and what they do:

{% include screenshot.html img="022" %}

It looks like `create` will be what I need, let’s see what’s required:

{% include screenshot.html img="023" %}

I need to give `az group create` two bits of information: a name, which makes sense and also which Azure data center this group, and all its resources, will live in. The details also tell me to run `az account list-locations` to find out the possible names I can choose from:

{% include screenshot.html img="024" %}

The result is a pretty big splash of JSON and there are ways to make this read a bit better - but I’ll deal with that at a later time. Scanning over this list is pretty easy, and I’ll pick West US 2 as my data center.

I’ll call the resource group “velzy”, after the name of my application - and with that I’m ready to create my new resource group:

{% include screenshot.html img="025" %}

I once again see a JSON response from Azure telling me that my resource has been provisioned. First job is done!

## Step 6: Create an AppService Plan

The `az webapp create -h` help output said that a `--plan` argument was required. The detail of that mentioned that we could create one with `az appservice plan create` but that’s about it.

The obvious question here is: *what the heck is an App Service Plan*? This can be a little confusing, because there are resources in Azure called “App Services” and there are other resources called “App Service Plans”. Reading through the documentation can add to the confusion, unfortunately. For the answer, I turned to my colleague Scott Hanselman, who told me that I could think of an App Service Plan as the VM my app will be running on - even though an App Service Plan has nothing to do with VMs, they are conceptually equivalent. 

An App Service plan is measured in terms of CPU cores, RAM and storage capabilities, much like a virtual machine might be. You can search the internet for “App Service Plan Pricing” and you’ll end up at the table below:

{% include screenshot.html img="026" %}

One other thing to note about App Service Plans is that they can be reused for multiple resources. I could deploy 10 web applications to a single App Service Plan, or create multiple instances of a given database.
Now that we know all of this, let’s create our plan. I’ll add a `-h` flag to the `az plan create` command so I can understand what I need to do next.

The only required arguments for this command are a resource group name as well as the name for our plan. I can also add a few other options, which in my case are very important.

The first is the `--is-linux` flag. If I don’t set this, I’ll get a Windows App Service Plan, which won’t work with deploying docker containers. I’ve learned this the hard way:

{% include screenshot.html img="028" %}

The next argument I want to pay attention to is `--sku` because this will dictate how much I’ll be paying each month. Pricing varies due to a number of factors, including data center and subscription plan. Head over to the App Service Plan Pricing page to see which plan works for you.

I’ve decided that a Standard Plan, or S1, will do nicely for this application. I can see the list of possible SKUs by scrolling up and reading the details:

{% include screenshot.html img="029" %}

Note that a free plan won’t work if you’re planning on deploying a Docker container, as I am.
Now that I have all of that, I’m ready to create my App Service Plan:

{% include screenshot.html img="030" %}

## Step 7: Create a Web App for Containers

We’re almost there! I have my resource group and my app service plan, now all I need to do is to come up with a name for my application and set the deployment image.

If you recall, I’ve deployed the simplest Node app I could come up with to DockerHub under the tag `robconery/velzy`. That’s the last piece of this puzzle: *how do I tell Azure where to get the image?*

If you recall, executing the command `az webapp create -h` lists out all of the arguments that you can send in, including one for deploying a Docker container, which I can specify using `-i`:

{% include screenshot.html img="031" %}

With that, my command is completed! I simply need to use the command:
```bash
az webapp create -g velzy \
                 -p velzyplan \
                 -n velzyapp
                 -i robconery/velzy
```

The `-g` flag specifies my resource group, the `-p` flag specifies the App Service Plan. The `-n` flag is the name of my app and `-i` is the DockerHub image tag. This a pretty simple naming scheme and your needs may vary; go with whatever works for you and your organization.

As usual, we get a splash of JSON back. If you get an error, read the messages carefully as they are typically quite detailed. My most common error is usually due to spelling.

There’s one bit of JSON here that I care about most, the URL to my new site:

{% include screenshot.html img="032" %}

Let’s go have a look!

## Step 8: Wait

Unfortunately the startup process isn’t very fast. All we’ve done up to this point is to tell Azure all of the settings that go into running this web application - we haven’t actually told Azure to run it. The good news is that we don’t have to do that explicitly, we can just visit the URL:

{% include screenshot.html img="033" %}

This experience is underwhelming as it gives no sense of what’s happening. The spin up time for a containerized web application can be as short as 30 seconds or as long as 10 minutes, depending on numerous factors. Each resource that will power this web application needs to be started up and, once that’s done, the image needs to be pulled from DockerHub and a container started. This can take a while!

To better understand what’s going on, let’s head to the Azure Portal and turn on Diagnostics and Logging. You can do this through the CLI as well, but right now this is the easiest thing to do for us. We’ll head over to our velzyapp resource and scroll down, clicking on “Diagnostic logs”, which is turned off by default. We’ll select “File System” and click save.
Now, let’s click on “Log stream”. This will tail the logs for us so we can see, realtime, what’s happening in the background. As you can see - not a lot right now.

While we’re waiting for logs to appear, let’s have a look at a very common problem that bites a lot of people when they first try to deploy a containerized web application to Azure. I’ve had this happen to me more than I care to say.
If your application uses a port other than the default web port (80), you have to make sure you set the `WEBSITES_PORT` application setting. You can do this in the Application Settings blade on the portal:

{% include screenshot.html img="034" %}

The other thing I could have (and probably should have) done is to use the `PORT` environment variable in my code:

{% include screenshot.html img="035" %}

This is such a common stumbling block for Node developers, however, that the Azure App Service team decided to handle port 3000 by default.

OK, about 3 minutes have gone by, let’s see if our logs are telling us anything:

{% include screenshot.html img="036" %}

The output shows that Azure is pulling our image from DockerHub and is attempting to start it. Waiting for another 30 seconds and we see something that isn’t terribly exciting, a *502 server error*:

{% include screenshot.html img="037" %}

This can be confusing, especially for .NET developers who recognize this as an ASP.NET error page. Our App Service is supposed to be Linux! What’s this page doing there?

The quick answer is that this is the server that sits in front of our container and routes traffic to it, which runs on Windows and uses Microsoft IIS. It’s telling us that it can’t reach our application, and it expected to. When this happens after a long wait it typically means one of two things:

 1. Our container hasn’t started/wasn’t able to start due to an error
 2. Our ports aren’t setup correctly

The first thing I typically do when this happens is to wait for another minute or two and then refresh the page. If that doesn’t work, I’ll go through and make sure that all my ports are correctly set.
In this case we get lucky and refreshing the page does the trick:

{% include screenshot.html img="038" %}

The URL indicates that the page is not secure, which we can change manually by using `https`:

{% include screenshot.html img="039" %}

Great!

## Epilog
When we’re all done playing around, we can make sure that don’t have to pay for resources we’re not using. This is one of the primary benefits of using resource groups - we can delete everything we’ve created with a simple command:

```bash
az group delete -n velzy
```

We’ll be asked to confirm our choice, and then we’re all done!
That’s it for this first AzureCast. I hope you enjoyed it!
