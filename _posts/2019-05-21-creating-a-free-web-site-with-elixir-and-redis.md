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

## Transcript

In episode #006 I told you that we couldn't use Azure's free tier, _F1_, unless we used the Windows operating system. 

{% include screenshot.html img="screenshot_155" %}

Not a big deal if we're using Node and are happy to use Azure's canned hosting engine. But what if we want to do our own thing with Docker? Unfortunately that meant we were out of luck because Docker requires the Linux operating system, which wasn't free.

Until 2 weeks ago!

## Announcing Linux Free Tier for Azure App Services

I was at //Build 2019 watching my old boss's boss, Scott Gu, who is now my boss's boss's boss's, boss's, boss's boss - announce all the fun things coming for Azure. 

{% include screenshot.html img="screenshot_212" %}

About 30 or so minutes into his keynote, Scott Guthrie announced a "perpetual free tier for Azure App Services on Linux". _That is big news_!

{% include screenshot.html img="screenshot_211" %}

Why is this big news? Because we can now run dockerized applications on Azure _for free_ in Azure's free tier. This is _outstanding news_ for developers who want to be sure their clients can see their work. The free (or F1) App Service tier is the same as with Windows:

 - You have 1G total disk space
 - 1G of RAM
 - Only one instance allowed, so no scaling possible
 - No custom domains

{% include screenshot.html img="screenshot_209" %}

The idea is that you can use this free tier to try things out and show your client or boss what you're up to when you're building something.

This is the best part: _you don't need to worry about using file-based databases or fake services either_. App Services on Linux supports containerized web applications, **including multi-container applications using Docker Compose or Kubernetes**. 

That's what I built for this week's AzureCast, a super simple web application in Elixir, connected to Redis:

{% include screenshot.html img="screenshot_213" %}

Let's see how I did it.

## Some Things to Prepare

I hope you like playing along; that's kind of how I make these things after all! If you want to, there are a few things you'll need to have rolling before we start:

 - An Azure subscription and an authenticated CLI with a default sub picked.
 - Docker running and the knowledge of how to use it
 - A familiarity with Docker Compose
 - General familiarity with Make, which I cover in [episode #005](https://azurecasts.com/2019/04/11/005-using-make-to-orchestrate-shell-scripts/).

## What We'll Build

This is my groovy Elixir app running locally here on my machine. I'm simply outputting a nice hello message with the result of a Redis `PING` command, which replies `PONG`.

The Elixir app is just like the Node app I created in [episode #001](https://azurecasts.com/2019/03/29/001-simple-docker-deployment/) - as simple as I can possibly make it. There are no frameworks involved here, just the sample code from [this tutorial up at Elixir School](https://elixirschool.com/en/lessons/specifics/plug/).

The code is as simple as I could make it, and uses Elixir's Plug module to generate a response:

```elixir
defmodule Example.HelloWorldPlug do
  import Plug.Conn

  def init(options), do: options

  def call(conn, _opts) do
    conn
    |> put_resp_content_type("text/plain")
    |> send_resp(200, "Hello World!\n")
  end
end
```

You can review this code in [our GitHub repo](https://github.com/azurecasts/009-freedom-linux) if you want know more, but I won't be explaining much more about Elixir.


### My Local Docker Files

I decided to use Docker Compose to create the Redis instance as well as run my Elixir app, which is what you see here. There's nothing overly complicated about it - which is exactly the way I want it for right now. 

{% include screenshot.html img="screenshot_214" %}

I am well aware that this is _not_ an optimal development setup, but as I keep saying in these screencasts I want to **keep everything as simple as I can** so that, in the event of an error with Azure, I can be as sure as possible that it's not coming from my code.

This is the Dockerfile that is being built for my Elixir app:

{% include screenshot.html img="screenshot_215" %}

Again - nothing terribly complicated here if you know Elixir _however_, there is one thing to draw your attention to which you might have noticed in my `docker-compose.yml` file: I _need_ to use port 8080. I'll explain more about this later on.

Right - let's fire it up and make sure everything works, and it does. Hey with enough practice you, too, can make screencasts!

{% include screenshot.html img="screenshot_216" %}

The final thing to do is to push this docker image up to DockerHub. I'll build it with the tag `robconery/009-elixir` and then do a `docker push`... and up it goes.

{% include screenshot.html img="screenshot_217" %}

But why did I need to do that?

## Azurizing My Docker Setup

If you recall from [episode #001](https://azurecasts.com/2019/03/29/001-simple-docker-deployment/), using Docker with Azure requires an accessibly Docker image. I used DockerHub for that, but I could have used a custom registry if I wanted to.

That presents two issues for me right now:

  1. My docker-compose.yml file is building my application's Docker image for me. This is nice and all, but Azure won't do this because it won't have access to my code.
  1. I need to have a way to tell docker-compose (and therefore Azure) how/where to get my application's Docker image.

The good news is that I can replace this line right here:

{% include screenshot.html img="screenshot_218" %}

With an `image` specification, pointing to DockerHub. That will work for Azure, but it won't work for me locally, so to get around that I'll create a dedicated `docker-compose.yml` just for Azure, copying the file I use locally:

{% include screenshot.html img="screenshot_220" %}

I'll reset the `build` command to `image` and I'm all set.

## Pushing to Azure

I'm a huge fan of the CLI, primarily because I can program it using shell scripts. I'm using Bash on my Mac, but you can do the same thing with Powershell if you're a Windows fan.

Here's a script that should look familiar if you've seen some of my other AzureCast episodes. I have a variable block set at the top, specifying my resource group, application name, location and SKU name, which is `F1` for FREEEEEE:

{% include screenshot.html img="screenshot_221" %}

We could use this script if we wanted - in fact I've added it to the source code for this episode - but I find that using Make is much more fun and useful, so let's flip over to a Makefile:

{% include screenshot.html img="screenshot_224" %}

This should look very familiar to you if you've seen episode #006. The main differences with this Makefile, however, is the free SKU and  that I'm sending up a Docker Compose file - specifically the `docker-compose-azure.yml` file that I just created. I'm also telling Azure this file is for use with Compose and not Kubernetes.

How did I know how to do this? Hopefully you know by now! I used `az webapp create --help` to see what kind of Docker settings I could use, and there it was!

{% include screenshot.html img="screenshot_231" %}

Notice that it's _Linux only_. Which is cool, man, cause it's free now.

Because I'm using Docker, I'll want to make sure that container logging is enabled and I can do that with the `--docker-container-logging` setting, set to "filesystem". This will also push the log contents to STDOUT, which will help us troubleshoot messed up deployments.

{% include screenshot.html img="screenshot_225" %}

Let's see if it works! I'll execute this file using `make || make rollback`, which is why I like using Make for this kind of thing. If *any* of the targets errors out, the `rollback` target will be fired, dropping the entire resource group - I talk about all of this in episode #005.

{% include screenshot.html img="screenshot_226" %}


## Waiting. And Waiting. And.... Waiting.....

The resources were built without a problem, however we now have to wait. For people (like me) who are familiar with Azure's non-Docker App Services (which I showed in episode #006) or services like Heroku or Zeit - this is going to seem incredibly _slow_. In fact, this whole process took about 9 minutes from start to finish!

{% include screenshot.html img="screenshot_227" %}

The thing is, though, that there are a lot of pieces involved here. Azure has to create our resources and then pull down and run each image we need - all on a service that has roughly 1G of RAM and very limited storage space. This is a one-time setup and it's also free - updates typically roll in within minutes. If you need something faster, there's always the regular App Service that I showed in episode 006.

Oh hey! Look at that:

{% include screenshot.html img="screenshot_228" %}

We're greeted with the familiar 502 error because our container has been built, but the lag between the start and our site coming up causes this problem. Let's wait until the logs finish, showing our containers running and then we can refresh... boom!

{% include screenshot.html img="screenshot_229" %}

Victory! I think that's pretty neat. In a matter of minutes we have an Elixir application hooked up to Redis running in the cloud for free. There are a few more things we can do here, but I'm going to save that for a later episode because right now I feel like celebrating and taking the rest of the day off...

## So... What's Happening Here?

Before I do that, however, I want to show you a few things you should be aware of. Let's jump over to the portal and have a look around. If you click on "Container Settings" you'll see that we have a Docker Compose setup, which is in Preview currently.

That's important.

{% include screenshot.html img="screenshot_230" %}

We're playing around with things that could very well change and while this is fun, it means we'll need to be flexible. It also means that things might improve a lot. And that some other things might be confusing or broken... either way: _patience_.

One setting that is super important is this one right here:

{% include screenshot.html img="screenshot_232" %}

If you're creating an application for work or a client, it's likely they won't want it findable on DockerHub, so you'll make it private (good job)! You can set those credentials right here, or you can pass them in via the CLI - I'll let you figure out how. Hopefully you have the hang of that by now!

The next setting that's important is the Webhook URL. This is the URL that your container registry uses whenever you update your image. For me, DockerHub will ping this URL and Azure will pull down the updated image. You'll also have to turn on "Continuous Deployment" for this to work.

{% include screenshot.html img="screenshot_232" %}

Finally, you can see the Docker logs right here in this window, which can sometimes be useful.

### Kudu and SSH

We also have access to a Linux version of Kudu. It shows most of the same things, but in our case it's not receving Git pushes or synchronizing files. What it _is_ doing is handling the webhook url pings from DockerHub and kicking off a pull and restart. This isn't obvious, however, as you click around.

{% include screenshot.html img="screenshot_235" %}

We can also startup a Bash emulator, as you see here, but there are no files in our `/site` directory. This, again, makes sense because we're not dealing with files, we're dealing with Docker!

{% include screenshot.html img="screenshot_236" %}

What happens if we use a volume with our code, however? What if we install SSH into our container? These are all great questions - ones that I think I'll answer in a later episode. Or maybe I'll make Burke do it.

## One Last Thing

Oh yeah - one last thing. I told you that I needed to use port 8080 earlier and I mentioned that I would tell you why _later_. I suppose I should do that! The multi-container preview has some limitations, specifically:

{% include screenshot.html img="screenshot_233" %}

Certain commands, like `build` and `depends_on` are completely ignored. Port settings, other than 80 or 8080 are _also ignored_. I ran into this a few months back and didn't solve it until yesterday when I read this page.

Well that's it for me! Thanks so much for watching.