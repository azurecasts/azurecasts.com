---
layout: video
title: "Using Make to Orchestrate Shell Scripts"
number: "005" #set this to your episode
categories:
tags:
summary: "In this video I'll take control of the shell commands that I've been using in episodes #001 and #003, first adding them to a script file and then orchestrating things a bit more carefully with Make"
video: https://www.youtube.com/embed/bal-GLCAb5s
author: Rob Conery
github: https://github.com/azurecasts/005-make
transcript: 005-orchestration-make
minutes: "11:50"
twitter: robconery
---

In this video I'll take control of the shell commands that I've been using in episodes #001 and #003, first adding them to a script file and then orchestrating things a bit more carefully with Make. You don't have to understand Make to benefit from this video and yes, I might be abusing it just a little bit... but it's for a good cause!

## Resource Links

You can read more about the Azure resources in this video here:

 - [What the heck is Make?](https://www.youtube.com/watch?v=_r7i5X0rXJk)
 - [The az webapp container command](https://docs.microsoft.com/cli/azure/webapp/config/container?view=azure-cli-latest?WT.mc_id=azurecasts-website-robcon)
 - [Install the CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest?WT.mc_id=azurecasts-website-robcon)

## Transcript


I like working with the Azure Command Line Interface - also known as the CLI. I don’t know Azure very well and I find the help feature extremely useful when trying to create new Azure resources as I’ve been doing in the previous episodes.

There is a drawback, however, and that is that you can end up with a bunch of cryptic text strewn around your application or, worse yet, you forget what command you used to create a given resource and have to go through the discovery process again.

The good news, however, is that this is programming! As my friend K. Scott Allen says: the Azure CLI allows you to treat Azure like a programmable data center - I think that’s great. We can treat the commands that we write in the same way we would treat any code: we'll add it to a file, comment it, and organize it so it makes sense and executes correctly.

Today I'm going to organize the commands that I’ve been working with in the previous episodes. I’ll walk you through how I do things, from simple script files all the way to orchestrating things with Make.

If you’re not a CLI fan but are curious - hold on tight as this episode is for you!

{% include screenshot.html img="screenshot_115" %}


## The Simple Script File

It goes without saying that this episode will devoted to the CLI. I'll be using the terminal app on my Mac but any Unix-based system can follow along. If you're on Windows I would encourage you to use the Ubuntu subsystem if you want to follow along there.

I love thinking about the Azure CLI as a text-browser. All of the UI is stripped off the portal and each command I give using the `-h` argument is just like browsing a web page, but a lot faster. To get to the screen you see here, I'm just running `az`.

{% include screenshot.html img="screenshot_115" %}

I'll be using the ExpressJS website that I created in Episode #003 - the one with continuous deployment and logging. It's a barebone Express site created from the generator; I haven't customized it at all. 

{% include screenshot.html img="screenshot_116" %}

### One File to Rule The Commands

In the previous episodes I've been hunting down the commands I needed and then running them right in place through the shell. This is about as efficient as opening a REPL and testing your app by hand vs. using some form of automated testing.

We'll want to be sure our Azure environment can be recreated easily, so let's take a second and put our commands into a script. I'll do what I normally do: start sloppily and then slowly improve.

Shell scripts are executed top-down and in order, so I'll need to be sure that I start by creating the resource group, then the appservice plan, the webapp, etc. I won't make you watch my type all of this out... but here's what that script looks like:

```sh
az group create -n velzy -l westus2

az appservice plan create -g velzy \
                          -n velzyplan \
                          --is-linux
                          --sku S1

az webapp create -n velzyapp \
                  -g velzy
                  -p velzyplan \
                  -i robconery/velzy

az webapp log config --application-logging true \
              --web-server-logging filesystem \
              --detailed-error-messages true \
              --docker-container-logging filesystem \
              --failed-request-tracing true \
              --level information \
              --name velzyapp \
              --resource-group velzy

```

As you can see, a shell script is just a series of commands executed in exactly the same way as if I typed it directly into the terminal.

Let's run this and make sure it works.

{% include screenshot.html img="screenshot_118" %}

Oops. It didn't work and a bunch of not-good things happened:

 - I have a syntax error somewhere in my script which happened _after_ my resource group was built
 - The error didn't stop the script from executing, so other resources may or may not be living up on Azure right now
 - I have to go through and delete each resource that was created or somehow figure out how to bypass them if they exist so I don't get even more errors!


{% include screenshot.html img="screenshot_119" %}

This is the frustrating thing about shell scripts. If you create them as I did here they'll just keep running until they reach the end! There is good news, however: _we can fix this_ problem - and we will later on! We can also batch-delete everything we just created by deleting our resource group:

```sh
az group delete -n velzy
```

_Note: it goes without saying that dropping a resource group can be dangerous, especially if you mispell the name and manage to drop your production system. You might also drop work that people are doing that you didn't know of - so make sure dropping this group isn't going to get you fired. I'll talk about protecting resources in another episode_.

OK, our resource group is deleted now so let's fix the problem, which is silliness on my part: I'm missing two line-continuation characters `\` in the webapp command and the appservice command. I'll quickly fix that:

```sh
az group create -n velzy -l westus2

az appservice plan create -g velzy \
                          -n velzyplan \
                          --is-linux \
                          --sku S1

az webapp create -n velzyapp \
                  -g velzy \
                  -p velzyplan \
                  -i robconery/velzy

az webapp log config --application-logging true \
              --web-server-logging filesystem \
              --detailed-error-messages true \
              --docker-container-logging filesystem \
              --failed-request-tracing true \
              --level information \
              --name velzyapp \
              --resource-group velzy

```

... and now I can rerun this command... and after 30 seconds or so I can see that everything is built as I hoped!

## Adding Clarity With Comments

A shell script without comments can be frustrating, especially if you don't know the binaries being used. This is where traditional programming and shell scripting differ: we're orchestrating the building of resources here, not executing logic. It's OK to be super obvious!

Also: a personal preference of mine is to be sure you keep the context in mind. My comments are here to help you understand the script I wrote in the context of this video, so I'm adding a few hyperlinks and other things.

{% include screenshot.html img="screenshot_120" %}

Much better. These comments will help people who aren't familiar with shell scripts and those who aren't familiar with the Azure CLI's commands. It's a small thing, but your team (not to mention your future self) will really appreciate it.

For instance: this comment contains the log configuration settings right here, copy/pasted from the CLI documentation. This is extremely helpful if you want to tweak the scripts later on.

{% include screenshot.html img="screenshot_121" %}

## Refactoring Variables

Let's keep rolling with the improvements by making this script file a bit more idiomatic. It's a good idea to use variables to avoid hard-coding values in your script and to reduce repetition. My application name is repeated a few times in here, as is the resource group name. 

Let's fix that by creating a set of variables at the top of the script. Variables in bash are upper case and are always strings. You don't need quotes, but if your value contains special characters (like quotes, stars, back slashes etc) you'll want to wrap them in single or double quotes.

I'll set the application name, resource group, plan name and location right here at the top. To use these values, I simply prepend a `$` to the name of the variable. The value is then "expanded" when the script is run.

{% include screenshot.html img="screenshot_122" %}

Great - this reads a lot better. In fact, it's starting to feel a bit more like programming and a bit less like orchestration. We can turn this script into something a bit more general-purpose if we move _every_ value (that's not part of the command itself) into a variable. This includes the `SKU` and my DockerHub image tag.

There we go! This is now a generalized script we can rerun whenever we need a container-based web application!

```sh
APPNAME=velzyapp
RG=velzy
PLAN=velzyplan
LOCATION=westus
SKU=S1
IMAGE=robconery/velzy

#create the resource group; replace with your resource name
#you can list locations with az account list-locations
az group create -n $RG -l $LOCATION

#create the appservice
#for help: az appservice plan create -h
az appservice plan create -g $RG \
                          -n $PLAN \
                          --is-linux \
                          --sku $SKU
#create the webapp and pass in the image name
#replace the names below with whatever works for you
#for help: az webapp create -h
az webapp create -n $APPNAME \
                  -g $RG \
                  -p $PLAN \
                  -i $IMAGE

#setup logging
# az webapp log config [--application-logging {false, true}]
# [--detailed-error-messages {false, true}]
# [--docker-container-logging {filesystem, off}]
# [--failed-request-tracing {false, true}]
# [--ids]
# [--level {error, information, verbose, warning}]
# [--name]
# [--resource-group]
# [--slot]
# [--subscription]
# [--web-server-logging {filesystem, off}]
az webapp log config --application-logging true \
              --web-server-logging filesystem \
              --detailed-error-messages true \
              --docker-container-logging filesystem \
              --failed-request-tracing true \
              --level information \
              --name $APPNAME \
              --resource-group $RG
```

I feel good about this, but there's one extra step we can take to improve this script.

## Using a .env File 

I use Z-shell instead of bash because I find the conveniences to be extra useful. The main reason I use Z-shell, however, is because of the [Oh My Zsh](https://github.com/robbyrussell/oh-my-zsh) project from Robby Russel. I mentioned this in a previous episode, but it's worth mentioning here one more time.

Oh My Zsh is a "shell framework" which offers all kinds of useful shortcuts and plugins that make programming (if you're a CLI person) a lot more efficient. One of the plugins I absolutely love is the [dotenv plugin](https://github.com/robbyrussell/oh-my-zsh/tree/master/plugins/dotenv), which will load the variables, functions and aliases in a `.env` file right into the shell whenever you navigate into a directory.

Using a `.env` file for this project will work out well, especially if I'm going to end up with additional shell scripts later on. My Node project can also read from this file using the `dotenv` module:

```sh
npm install dotenv --save
#require("dotenv").config() in your code
```

I have this plugin installed so let's try it out. I'll move my variables into the `.env` file and then use the `printenv` command to verify they're not in my current environment. I'll `cd` one directory up and then back down and I'll run `printenv` again and there they are.

{% include screenshot.html img="screenshot_123" %}

Incidentally, you might have noticed these environment variables for Azure. This is a test I was running that unfortunately didn't work out as I had hoped - but I wanted to show you anyway because it might be useful.

During the recording of this episode I left a suggestion on the GitHub repo for the Azure CLI. I recommended that the CLI set the values for `--resource group` and `--name` by checking to see if an environment variable was present.

{% include screenshot.html img="screenshot_123" %}

This is the fun part of my job - using the tools that the product team builds and sharing my experiences with them!

One of the standard ways to configure binaries at runtime is to use a "runtime configuration" file - or an "rc file" which is typically the name of the binary prefixed with a `.` and suffixed with `rc`. For instance, you can set Vim's runtime configuration using a `.vimrc` file. 

This is another suggestion I made for the team: _allow for a `.azrc`, which would be extremely helpful as I could then run my commands without having to repeat the `-g` and `-n` settings.

This is when I found out about the `AZURE_DEFAULTS_` variables, which is great!

{% include screenshot.html img="screenshot_125" %}

I tried out a few commands with them and they do, indeed, work for the most part. If I set `AZURE_DEFAULTS_GROUP` I can run commands like `az group show` without the `-n` argument and it works. Unfortunately, the `az group create` command will throw an error if the required variables are not passed in.

I followed up with the team on this - that they should 1) consider allowing required variables to be set by environment variabls and 2) they should allow for a `.azrc` file that might contain those settings.

The good news? It looks like this feature might be coming soon!

{% include screenshot.html img="screenshot_126" %}

OK - let's verify that our script works using the `.env` file. I deleted the resource group off camera and I'll skip ahead here as well... and yes! It works.

## Orchestrating Commands with Make

I'm a huge fan of Make because it helps to orchestrate shell commands, and I think we could use some of that right here. 

As you might have noticed, I have a Makefile in the root of my application which builds and runs my site - I created that in episode #003. I don't want to confuse the two things so I'm going to create an `/azure` directory and put my Makefile in there.

{% include screenshot.html img="screenshot_128" %}

### What is Make Again?

If you've never used Make, it's kind of a dinosaur from the prehistoric days of C programming. In technical terms it's a build tool that orchestrates code files and turns them into binary objects. If you're familiar with build tools of _any kind_ (Rake, Grunt, MS Build, whatever) then Make will be kind of familiar. 

The bottom line is this: _Make turns one file into another_ following some rules. I'll discuss those rules as we go along.

## The Make Skeleton

With a tool this old you're bound to run into a lot of different schools of thought and I'm going to show you mine. The first thing I tend to do is to create an `all:` target and a `.PHONY:` target which I spelled wrong in the video (`.PHONEY` - sorry about that).

{% include screenshot.html img="screenshot_127" %}

That's what each of these labels is called in a Makefile: a _target_. They're called that because they're typically associated with building or transforming a file of some kind whether it's code or binary.

Each target does one thing and that one thing is described by a shell command. We have a nice set of commands so far, so let's add each one as a target.

{% include screenshot.html img="screenshot_129" %}

Great. Now I'll copy/paste each of my shell commands under each of my targets, making sure to indent using a single tab. You'll get an error, by the way, if you try to use any other kind of indentation.

{% include screenshot.html img="screenshot_130" %}

You'll notice the variable names look weird, and that's because a Makefile is _not_ a script file, even though you can pop shell scripts directly in as you see I'm doing here. Let's take a quick tangent.

### Anatomy of a Makefile

When you work with Makefiles you create targets, rules and recipes. We'll be doing each of these things today, and they look like this:

```make
VARIABLE=VALUE

target: prerequisite target #together this is a rul
  recipe $(VARIABLE) #some form of command
```

The variables go at the very top and use, roughly, the same syntax style as a shell command. There are differences, but that's beyond what I want to get into for this episode.

Next we have our targets. Indented under the target is a _recipe_ which is the shell command to run to fulfill that target. A target can also have a _dependency_ or prerequisite, which is another target in the same Makefile, that needs to be completed before continuing.

Each target is run in _its own process_, which is a key thing to remember. This allows Make to know if a target has succeeded or not as the process will return a value of `1` if it has failed. That's all it can return, a `0` for success or a `1` for failure.

Now that we understand that, let's continue on.

## Fixing Up the Makefile

I'll copy and paste my variables up top here and I don't need to do anything special with them, but I _do_ need to change the reference to them within each recipe. This syntax: `$(RG)`, tells Make to substitute the variable value before handing it off for execution.

Great, all the commands and variables are set. Now comes the fun part! _Orchestration_. I can orchestrate what happens when by simply defining dependencies. I'll go top to bottom here and specify that my `plan` target requires `rg` to have completed. The `webapp` needs a `plan` and logging requires `webapp`.

Make will execute the very first target in the Makefile by default, which is `all:`. To get everything to work properly I need to execute the _last_ target in the chain, which is `logging`. 

{% include screenshot.html img="screenshot_131" %}

There is another way I could have done this and some Make fans might thing it's clearer. I could have avoided prerequisites on each target and just put one big prerequisite chain on `all`. Both work, I suppose it's a matter of style.

```
all: rg plan webapp logging
```

Let's make sure it works. I'll run this file using the command `make`, which will execute my `all` target. As you can see, each command is output into the terminal so I can see what's going on. The orchestration is working great!

{% include screenshot.html img="screenshot_131" %}

This works, but we're missing the best part - our _rollback feature_. I'll delete the resource group once again so we can amp out this Makefile and make it shine!

```sh
az group delete -n $RG
```

## Handling Errors

While the resource group is deleting I'll change things in my Makefile by using a resource group name, right here, that doesn't exist:

```make
plan: rg
	az appservice plan create -g BLARG \
														-n $(PLAN) \
														--is-linux \
														--sku $(SKU)
```

Right, this will throw an error that comes from the Azure CLI. Now let's run our Makefile... and BOOM!

{% include screenshot.html img="screenshot_133" %}

This is GREAT! Notice that execution _stopped_. We still have a resource group as that was completed with the previous target. That's OK, we can clean things up with a special target devoted to just that.

### Rolling Things Back

A typical Makefile has a dedicated target to clean up the build artifacts in the event of an error of if the user just wants to rebuild everything entirely. Not surprisingly, that target is called `clean`.

I would use that here, but I'd rather be ultra clear for the context of this project (which is a video for people who might not be completely familiar with Make), so I'm going to call my target `rollback`. This target is simply going to drop the resource group entirely, as you've seen me do:

```make
rollback:
  az group delete -n $(RG) -y
```

Great. To use this we can leverage the return value of Make which, like each of it's targets, is either `1` or `0` depending on if there was a failure. That means we can call Make and evaluate its result, executing the `rollback` if there was an error:

```
make || make rollback
```

Nice and simple. Let's see if it works by executing the exact same Makefile with an error:

{% include screenshot.html img="screenshot_134" %}

Yes! Victory!

## Final Touches

Let's do just a few more things for convenience. We'll want to see our site after we deploy it, so let's pop in a new target here called `open`. I also like to look at the logs while my site is loading up (so I know what's going on) so I'll add a `logs` target as well.

Here's the entire Makefile:

```make 
APPNAME=velzyapp
RG=velzy
PLAN=velzyplan
LOCATION=westus2
SKU=S1
IMAGE=robconery/velzy

all: logs

rg:
	az group create -n $(RG) -l $(LOCATION)

plan: rg
	az appservice plan create -g $(RG) \
														-n $(PLAN) \
														--is-linux \
														--sku $(SKU)
webapp: plan
	az webapp create -n $(APPNAME) \
										-g $(RG) \
										-p $(PLAN) \
										-i $(IMAGE)
logging: webapp
	az webapp log config --application-logging true \
								--web-server-logging filesystem \
								--detailed-error-messages true \
								--docker-container-logging filesystem \
								--failed-request-tracing true \
								--level information \
								--name $(APPNAME) \
								--resource-group $(RG)

open: logging
	open https://$(APPNAME).azurewebsites.net

logs: open
	az webapp log tail -n $(APPNAME) -g $(RG)

rollback:
	az group delete -n $(RG) -y

.PHONY: logs rollback open webapp rg plan all
```

The final thing I want to do is to spell `.PHONY` correctly. This is another special target that tells Make that no files are actually produced by the following targets. This is another feature of Make - it will check the timestamps of the files produced by a target to see if a given target even needs to run. If the source of a target hasn't changed since it was last built, it will be skipped.

We can omit that check and speed things up by specifying targets that don't produce any files. We do that with `.PHONY`.

OK, the last thing I need to do is to reset the `all` target to call `logs` which will start the prerequisite chain! 

Running the command and waiting for just a bit... Ha! There's my browser open to my site's URL and the familiar spinning icon. If you recall from previous episodes, container deployments can take a while.

{% include screenshot.html img="screenshot_135" %}

I'll skip ahead about 4 minutes and show you both screens: my terminal, which is tailing my site's logs and my browser. Look at that! Something's happening!

{% include screenshot.html img="screenshot_136" %}

Azure is pulling down my container image and building everything, which again, takes a while so I'll skip ahead to the interesting part, which I froze right here:

{% include screenshot.html img="screenshot_137" %}

The image has been pulled and the container started. I froze it right here because a split second later...

{% include screenshot.html img="screenshot_138" %}

BOOM! The same 502 server error I've seen in previous episodes. In fact, I've seen this error _every single time_ I've tried to deploy using an image from DockerHub. I have a note into the team about this because it seems to happen in conjunction with the container starting, which isn't instant.

Anyway - we know what to do next: _refresh_! Doing that and I can see my site is up!

{% include screenshot.html img="screenshot_139" %}

I hope this was helpful and you can check out the code for this episode in our repository, the link is below. Thanks for watching!