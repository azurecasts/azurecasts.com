---
title: "#006 - Deploying Things for Free"
number: "006"
layout: transcript
---

I've been enjoying working with Azure so far, but testing things can be quite expensive unless you're familiar with the billing process. Rather than risk a surprise at the end of the month, I'm going to turn my attention to creating a web application on Azure that's completely free - including the database!

## The Problem
I've been deploying my site using a Docker container which is working well, but the S1 pricing tier is a bit more than I want to pay during development. I'd like to explore some cheaper options, and nothing beats free. 

{% include screenshot.html img="screenshot_153" %}

It turns out that Azure does have a free tier, but to use it you have to _akamai_ as they say here in Hawaii - "be smart" and do a little research. So let's do the research!

### Pricing Basics
Pricing varies widely based on region, subscription type, resources and, inevitably, time. Long story made as short as I can make it: we _can indeed_ deploy our current site for free, *with a database*, but we have to adjust how we're doing things.

{% include screenshot.html img="screenshot_154" %}

First: we need to make sure we're using the right kind of service plan. Right now I'm using `S1` to accomodate my docker container, but there's also the `F1`, or "free" tier, which I can use BUT - and this is critical - it doesn't support linux just yet. 

{% include screenshot.html img="screenshot_155" %}

This might change in the near future, but what this means for me is that I can't use docker any more because using Docker means I need to use Linux which I can't if I want things to be free.

Believe it or not, this actually makes things a bit easier on me. There are more details to discover, of course, but let's do that as we move forward.

## Updating Our Deployment

Here's the Makefile I've been using to push my Dockerized site to Azure. I like Make a lot and I show you how I build things with it in episode 005. As you can see here, I need to use a Standard tier App Service plan in order to use both Linux and Docker. 

{% include screenshot.html img="screenshot_190" %}

The problem is that I want this all to be free, which means no more image and a SKU reset to F1, the Azure free tier.

{% include screenshot.html img="screenshot_191" %}

I'll remove the image in the webapp creation command and I'll also trim back the logging as I don't need all of these settings if it's a hosted webapp.

The last thing I'll do is to rename the app - I'll call it `velzyappfree` with resource group `velzyfree` and the plan `velzyplanfree`. I don't _have_ to do this - it just makes me feel better.

{% include screenshot.html img="screenshot_158" %}

### Create the Deploy User

Now that I'm deploying directly to Azure, I need to use deployment credentials. This was handled with DockerHub using an authentication key - but I need to identify myself if I'm going to push directly to the app.

I'll set my credentials here using the app name and a deploy suffix. For the password I like to use pass phrases as they're easy to remember but difficult to brute force.

{% include screenshot.html img="screenshot_159" %}


### The Git Remote URL

Azure is going to be running this site for me so I need to tell it how to do that. I won't be running on Linux which means that a Windows virtual server in a container is going to be running my app. I'm not used to using Windows as the remote repository for Git, but the good news is that this experience is seamless.

But how do we know where to push? This, once again, is a job for the CLI help system. Here are the commands that I used to figure this all out:

{% include screenshot.html img="screenshot_193" %}

The very last one is telling me I need to execute the `config-local-git` command, doing that I get a URL back:

{% include screenshot.html img="screenshot_194" %}

Examining the structure of this URL we can see that it's entirely guessable given the information we have. We can build the hostname using the name of our app and adding a `.scm.azurewebsites.net/appname.git`. We're using basic authentication to login (over HTTPS so it's secure), which means we can pop our credentials right before the host name as you see here.

Great - that's set but now I need to select a runtime - something that works on Windows. How can I figure out what that is?

{% include screenshot.html img="screenshot_160" %}

### Choose the Runtime

I can find that out by asking the CLI - specifically `az webapp create --help`. There _should_ be some kind of thing in there about runtimes... and there is: `az webapp list-runtimes`.

{% include screenshot.html img="screenshot_161" %}

That's a lot of runtimes! At least it looks that way - there are only four total languages to choose from, however:

 - node
 - php
 - java
 - python

Since I'm using Node I'll pick the latest runtime, which is version 10.6 and I'll specify that right here in my runtime variable.

{% include screenshot.html img="screenshot_162" %}

### Tweak the Web App Create

Now that I have the runtime set it's time to tweak the `webapp create` command. I need to specify the runtime I just chose and I _also_ need to tell it that I want to deploy my source using a `git push`.

To find this out I'll run `az webapp create --help` once again and look for an argument that specifies a `git push` or something. But I hit paydirt right off the bat! At the very bottom here is an example command for deploying a site using Node _and_ a `git push`! I just need to add the argument `--deployment-local-git`.

{% include screenshot.html img="screenshot_163" %}

Just for the sake of being complete - let's take a look at the other deployment options that use Git. The one I'm using is the first selection and I can specify a specific branch if I want using `-b`. I can also set things up to pull from a remote Git repository, like GitHub.

That's for another episode. For now, I'll copy and paste the example command right into my Makefile, resetting the runtime specification to use my shell variable.

{% include screenshot.html img="screenshot_164" %}

### Setting the Deployment and Pushing

We're almost ready! I've setup my `DEPLOY_USER` and `DEPLOY_PASSWORD` variables but now I need to tell Azure about them. To that, I execute `az webapp deployment user set --user-name $(DEPLOY_USER) --password $(DEPLOY_PASSWORD)`. I figured that out by using the CLI help system, which I think you should have the hang of by now. Once I've done that I'll add the `remote` branch to my Git repository which will use the `GIT_DEPLOY_URL` I created previously.

{% include screenshot.html img="screenshot_165" %}

When you add the `--deployment-local-git` argument to the `webapp create` command, you're given a remote Git repository to push to. It follows a standard naming convention, just like our webapp does. Since we're pushing over HTTPS, I can pass the user name and password right in the URL too, which makes things very easy.

The final step is to create an actual `deploy` target that will push my code up to Azure. This is a simple `git push azure master`.

{% include screenshot.html img="screenshot_166" %}

I'll reset the dependencies for each target so that these commands go off in the correct order and we're done!

## Trying It Out

Let's give it a try! I'll use my typical `make || make rollback` command and... oops. 

{% include screenshot.html img="screenshot_167" %}

Looks like I have a small error which is ... ha! The perils of copy/paste! I don't have a proper line continuation.

{% include screenshot.html img="screenshot_168" %}

Fixing that and... dang another error! The good thing about this is that my `rollback` target is called which wipes out the entire resource group. 

{% include screenshot.html img="screenshot_169" %}

While that's happening I'll see if I can find the error and... there it is. I accidentally put `DEPLOY_PASS` in my deployment URL instead of `DEPLOY_PASSWORD`.

{% include screenshot.html img="screenshot_170" %}

Let's execute this again and... skipping ahead a few minutes... it looks like it's working! Our `git push` went off just fine and the post-receive hook up on Azure is sending us back some interesting information. The most interesting of which is this line, right here:

{% include screenshot.html img="screenshot_171" %}

Why is this so interesting? Let's take a quick tangent.

### Kudu and Windows

We already know we're deploying to Windows but this just confirms it for us: _the path specification is using backslashes and is referencing the D drive_. The file structure is _also_ fascinating to me!

It looks like we're inside a Windows container with an attached `D:\` drive, which is a standard data drive for Windows servers. Basically: _we have an entire Windows machine_ at our disposal... for free!

I can also see that there's a thing called "Kudu" which is running some kind of synchronization between the repository I'm pushing to and the place where my site is actually hosted. I'm going to dig into that a bit later.

## Boom, It Worked!

OK, back from our tangent and our site pops up - this time it's _a lot faster_ than my previous experience with Linux containers. What's happening here, and why is this experience so much different?

{% include screenshot.html img="screenshot_172" %}


### Digging into Kudu

Let's find out more about Kudu. If you head over to the Azure Portal you'll see a non-descriptive link under "Development Tools" called "Advanced Tools". See that "K" - looking icon? That's "K" for "Kudu". 

{% include screenshot.html img="screenshot_148" %}

How do I know this? The short answer is that I was having trouble one day and called Scott Hanselman (which is a fun thing that I can do because I work here again) and he said "oh - go checkout Kudu - I bet you'll find out the answer".

> Kudu? WTF is that?

"Dude - go to your portal and..." and he proceeded to show me this weird-looking website. Most people don't even know it's here, which is a shame because it's pretty fascinating. I'll show you why right now.

Click on that link and you see this ever-so-helpful page. None of this says _anything_ about Kudu; you just have to be persistent. Once you click on this link you're taken to _another_ login screen which might seem a bit gratuitous on Azure's part BUT! There's a good reason for it which I'll talk about _right after_ I talk about this screen:

{% include screenshot.html img="screenshot_149" %}

The terrifically bland, boilerplate Bootstrap-styled Kudu front end. What... the heck is happening? I'll provide a quick summary now but will dive into these advanced tools more in a future episode.

{% include screenshot.html img="screenshot_151" %}

The short story is that Kudu is a project created by Microsoft that will receive a `git push` and, using a series of commands in a `post-receive` hook, it will synchronize a live web site with the updated changes. It also has web front end that tells you all about what's happening in the background.

Kudu _is not Azure_, it's an open source tool created by Microsoft that is used on Azure. This is an important distinction to make - that's why this website looks this way. In fact, if you examine the URL you can see that it's pointing to the same host as our remote Git repository.

{% include screenshot.html img="screenshot_147" %}

You can use Kudu anywhere - it doesn't have to run on Azure specifically. Microsoft just put this tool in place for us so we could do a remote push using Git to deploy our website. You could put Kudu on your own Windows server and have the exact same experience.

OK - so why do we care about this? For a simple reason: _this is super powerful information about your deployed site_. We have access to environment information and a REST API that tells us everything we need to know about our deployed app. My favorite is this online CLI that emulates a Unix CLI or Powershell. I can browse my virtual Windows machine right in here, looking at the deployed code and the Git repository too.

I've come to depend on Kudu so much that I've taken to creating a shortcut to it in my Makefile, using the target `sense`:

{% include screenshot.html img="screenshot_173" %}

I can open Kudu whenever I'm frustrated or confused by typing `make sense` in the command line... get it? I'm soooo clever. One of my favorite tools is the command line tool, which uses a Unix emulator or Powershell, depending on what you're comfortable with. Right above the emulator is an HTML file tree - this is crazy!

{% include screenshot.html img="screenshot_187" %}

I can click the links in the file tree or I can navigate in the CLI below it, with the file tree synchronizing to whatever the present working directory, or `PWD` is. We'll be coming back to this in a bit - right now I want to see if I can create a file-based (or embedded) database on my live site _without_ worrying about an overwrite.

## Using a Database

Up front: this probably is _not_ the optimal solution for a production system! For testing things out, however, it's perfect because it's all _free_. I'm going to use NeDB, which is kind of like SQLite for document databases. It emulates the MongoDB API, which means you can get rolling with a document system easily and graduate to Mongo later on.

{% include screenshot.html img="screenshot_174" %}

Setting this thing up is easy - just install it using NPM and declare what type of data store you want. I'll choose an auto-loading persistent one, storing my data in a single file.

{% include screenshot.html img="screenshot_175" %}

I've added this code to my Express index route, which you see here. 

{% include screenshot.html img="screenshot_176" %}

I want to store the data right in the root of my application (yeah, I know, _danger danger_!) so I'll create the directory, called `/data` and make sure to add that to my `.gitignore` file.

{% include screenshot.html img="screenshot_177" %}

Next - I've added a simply query to my index route which displays the data and I have a `post` handler which adds the data. I'm trying to keep this as simple as I possibly can because my goal is to see if I can use an embedded database without fear of overwrite when I update my site.

{% include screenshot.html img="screenshot_178" %}

Let's test it out and see if it works locally... and it does. It's almost as if I practice this stuff beforehand!

{% include screenshot.html img="screenshot_179" %}

Before I push the changes to Azure, let's see which files have changed locally. This is critical to me! I want to be sure that Kudu will _only_ synchronize changed files - not completely overwrite! These are the ones I've changed - so let's push.

{% include screenshot.html img="screenshot_180" %}

## Kudu Synchronization

My site has deployed and Kudu's `post-receive` hook has taken over. I can see the same message I saw before, but this time _only the changed files are synchronized_! That's great! But we still need to know if we can create a directory in the root of our app and have it stay there between pushes.

{% include screenshot.html img="screenshot_181" %}

Refreshing the site after deployment - there's our new home page. I'll add Burke to my friends list... though I don't know why... and it works. Let's add Emily too.

{% include screenshot.html img="screenshot_183" %}

Let's head back over to the Kudu command line tool and ... hey look! It's updated in the background - there's my `/data` folder right in the root of my site. 

{% include screenshot.html img="screenshot_184" %}

If I click on it, I can see the JSON file that NeDB uses for data storage. In fact, I can edit it directly if I'm feeling evil.

{% include screenshot.html img="screenshot_185" %}

Let's be sure this data will survive a full stop and restart. I'll use the CLI tool to do the start and stop.... verifying that the app is completely stopped... 

{% include screenshot.html img="screenshot_188" %}

Restarting - great! The data is still there.

{% include screenshot.html img="screenshot_189" %}

You can read more about how Kudu synchronizes files in their wiki - but right here is the relevant line:

> It's also smart enough to delete files that were removed from the repo, while not deleting files that were created at runtime by the site.

Perfect. That's exactly what I want to see. 

## Is This Really a Good Idea?

I know what you're thinking: _Rob, you're kind of crazy. I like crazy... but this is a bit too much crazy..._ and I hear you. This will do fine when testing things out or staging things for your client during development, but there are a number of reasons this won't work for anything more than that:

 - You can't use a custom domain
 - You get 1G of RAM and 1G storage, but the app will shut down after a while, creating a "cold-start lag" if you haven't visited it in a while
 - There's no scaling plan

In a later episode we'll have a look at a better storage solution that's virtually free, removing the need to stress out about deleting your database from disk.

That's it for me - thanks for watching.