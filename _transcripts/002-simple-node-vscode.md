---
title: "#002 Transcript - Deploy a NodeJS site directly from VS Code"
layout: transcript
---

002 - A Simple Node Plan
 
In this video, we are going to deploy a simple Node application to Azure utilizing the App Service extension for VS Code. We’re going to start with a super simple app, and then build up to something more complicated so that we can see how Azure treats different kinds of applications. 

Just as Rob did in episode 1, we’re going to start with the the code from the Node “Getting Started” guide. Like I said, super simple. 

{% include dropbox.html img="https://d2mxuefqeaa7sj.cloudfront.net/s_250BBE10AD6C9355F608E73A8DC5B5251AA028E0CC4DBBE3DDA708DD0C5C1B22_1553281872672_image.png" %}


To run this, We can press `cmd/ctrl + ``  to open the terminal inside of VS Code. You can use your standard terminal or command line if you want, but the built-in terminal is super convenient. Then we just execute our file with node.

{% include dropbox.html img="https://d2mxuefqeaa7sj.cloudfront.net/s_250BBE10AD6C9355F608E73A8DC5B5251AA028E0CC4DBBE3DDA708DD0C5C1B22_1553281964083_image.png" %}


And let’s test it in the browser just to ensure everything is on the up and up. You can click on links in the VS Code terminal by pressing `Cmd/Ctrl` when you hover over them.

{% include dropbox.html img="https://d2mxuefqeaa7sj.cloudfront.net/s_250BBE10AD6C9355F608E73A8DC5B5251AA028E0CC4DBBE3DDA708DD0C5C1B22_1553282074691_image.png" %}


Super! Now that we have an app, let’s deploy it to Azure. 

There are several ways to deploy and host sites in Azure. You can do with with a virtual machine, you can do it with Docker, you can do it as just a static file, or you can do it with something called App Service.'

App Service is what is referred to as a “Platform as a Service”. Often shorted to just “PaaS”. Pronounced Pazzzz. I think.

The idea behind App Service is that you can just deploy your code, and Azure will provide the runtime. In our case that’s Node. The important thing to note here is that you never actually have to know anything about the operating system or the server that is running your app. You don’t care about that. That’s Azure’s problem. You just focus on deploying your app to Azure and someone else worries about the server.

Since we’re in VS Code, we can use the App Service extension. 

{% include dropbox.html img="https://d2mxuefqeaa7sj.cloudfront.net/s_250BBE10AD6C9355F608E73A8DC5B5251AA028E0CC4DBBE3DDA708DD0C5C1B22_1553282386060_image.png" %}


This extension is installed from the VS Code extension marketplace. It’s made by Microsoft, so that’s how you know you’ve got the right one. After you install the extension, you’ll see an Azure logo in the Action Bar inside of VS Code.

{% include dropbox.html img="https://d2mxuefqeaa7sj.cloudfront.net/s_250BBE10AD6C9355F608E73A8DC5B5251AA028E0CC4DBBE3DDA708DD0C5C1B22_1553282543150_image.png" %}


If you click on that icon, you’ll get a new explorer view for Azure App Service.

{% include dropbox.html img="https://d2mxuefqeaa7sj.cloudfront.net/s_250BBE10AD6C9355F608E73A8DC5B5251AA028E0CC4DBBE3DDA708DD0C5C1B22_1553282592171_image.png" %}


There are other Azure extensions that you can install, and they will all show up right here. Right where the Azure Icon is.

It wants me to sign in, so I’m going to go ahead and do that by clicking on the text that says “Sign in to Azure…”. When I do, it’s going to launch a browser window asking me to sign in.

{% include dropbox.html img="https://d2mxuefqeaa7sj.cloudfront.net/s_250BBE10AD6C9355F608E73A8DC5B5251AA028E0CC4DBBE3DDA708DD0C5C1B22_1553282677034_image.png" %}
{% include dropbox.html img="https://d2mxuefqeaa7sj.cloudfront.net/s_250BBE10AD6C9355F608E73A8DC5B5251AA028E0CC4DBBE3DDA708DD0C5C1B22_1553282729981_image.png" %}


If I go back to VS Code. I now see all the Azure Subscriptions that I have access to displayed with that little key icon.

{% include dropbox.html img="https://d2mxuefqeaa7sj.cloudfront.net/s_250BBE10AD6C9355F608E73A8DC5B5251AA028E0CC4DBBE3DDA708DD0C5C1B22_1553282782031_image.png" %}


In my case, that’s a lot! Too many in fact. It’s looking quite cluttered over here. Since I only want to work with my own subscription - that first one there at the top, I’m going to open the VS Code command pallette by pressing `Ctrl/Cmd + Shift + P` and I’m going to type “subscription”. This will filter the list of options down to “Azure: Select Subscriptions”.

{% include dropbox.html img="https://d2mxuefqeaa7sj.cloudfront.net/s_250BBE10AD6C9355F608E73A8DC5B5251AA028E0CC4DBBE3DDA708DD0C5C1B22_1553282912032_image.png" %}


I can click on that and then I get a list of all my available subscriptions. I can deselect them all, and then just pick the one I want to work with, and click the “OK” button.

{% include dropbox.html img="https://d2mxuefqeaa7sj.cloudfront.net/s_250BBE10AD6C9355F608E73A8DC5B5251AA028E0CC4DBBE3DDA708DD0C5C1B22_1553282966883_image.png" %}


Now I see just the subscription I selected.

{% include dropbox.html img="https://d2mxuefqeaa7sj.cloudfront.net/s_250BBE10AD6C9355F608E73A8DC5B5251AA028E0CC4DBBE3DDA708DD0C5C1B22_1553283018106_image.png" %}


If I expand that subscription, I see all of the current apps I have for this subscription listed out.

{% include dropbox.html img="https://d2mxuefqeaa7sj.cloudfront.net/s_250BBE10AD6C9355F608E73A8DC5B5251AA028E0CC4DBBE3DDA708DD0C5C1B22_1553283120342_image.png" %}


I’m going to create a new application here so that I can deploy this simple Node server. There are a few ways to do that. I can click the plus button up at the top of the Azure App Service extension explorer space. I could also right-click on the subscription and select “Create new web app”. Another way would be to open the command palette by pressing `Cmd/Ctrl + shift + p` and search for “create” which will give me the option “Azure App Service: Create New Web App”. I could also go back to the File Explorer where my index.js file is and right click it and select “Deploy to Web App” and then select “Create new web app”. So many ways to do the same thing. You can pick the one you like the most. I’m going to just click this little plus sign up here.

First we have to pick a name.

{% include dropbox.html img="https://d2mxuefqeaa7sj.cloudfront.net/s_250BBE10AD6C9355F608E73A8DC5B5251AA028E0CC4DBBE3DDA708DD0C5C1B22_1553283438530_image.png" %}


Note that the name you pick has to be unique across all of Azure. That’s because after you create this site, you are going to get a URL for it in the format <site name>.azurewebsites.net. And URL’s gotta be unique. So your name has to be unique. 

The next thing it asks us for is a Resource Group.

{% include dropbox.html img="https://d2mxuefqeaa7sj.cloudfront.net/s_250BBE10AD6C9355F608E73A8DC5B5251AA028E0CC4DBBE3DDA708DD0C5C1B22_1553283522523_image.png" %}


 
What’s a resource group? Resource Groups are a concept that was introduced in Azure a while back. They are simply a logical grouping for your resources. For instance, our app could have a database in Azure, maybe some Serverless Functions in Azure and we would probably want to group all of that stuff together in a new Resource Group. You can always move things between Resource Groups. The other nice thing about Resource Groups is that when you delete the group, everthing inside of it gets deleted. You’ll find that you end up with a lot of resources the most you use Azure and Resource Groups help you keep it all nice and tidy.

I’m going to create a new Resource Group for my application. We can just give it the same name as our app, plus an “rg” designator on the end. I like to do that just to make them stand out in the Azure Portal as Resource Groups.

{% include dropbox.html img="https://d2mxuefqeaa7sj.cloudfront.net/s_250BBE10AD6C9355F608E73A8DC5B5251AA028E0CC4DBBE3DDA708DD0C5C1B22_1553283764821_image.png" %}


Now it will ask us if we want Windows or Linux. This is exactly what you would think - what sort of operating system do yo want your application to run on. I said you don’t have to worry about the server - and that’s still true. But the way Windows and Linux app service works is different. Therefore this option is important. For this video, we’re going to focus on Linux hosting. So I’ll select Linux.

{% include dropbox.html img="https://d2mxuefqeaa7sj.cloudfront.net/s_250BBE10AD6C9355F608E73A8DC5B5251AA028E0CC4DBBE3DDA708DD0C5C1B22_1553283872639_image.png" %}


Now it wants to know what runtime we want. If you scroll through the list you’ll see Node, ASP.NET, Ruby, PHP and Tomcat (Java). We’re running a Node app, so we want a Node version. You’ll notice that it recommends that we use version 10.10 because that’s LTS (Long term support). That’s actually not accurate. The current LTS version at the time of this writing is 10.15.3. That means that the highest LTS version available in Azure (as I’m typing this) is 10.14. So that’s the one we’ll select.

The next prompt will ask you to create an App Service Plan. This is basically you choosing how much “compute” you want. Or rather, how much power. How much octane. Do you want lots of memory and processing power, or do you not need that much? Remember, you pay for compute, so this is where you save or spend money.

I’m going to create a new App Service Plan

{% include dropbox.html img="https://d2mxuefqeaa7sj.cloudfront.net/s_250BBE10AD6C9355F608E73A8DC5B5251AA028E0CC4DBBE3DDA708DD0C5C1B22_1553284104597_image.png" %}


Selecting a “tier” is where you select how much power you want. You see here that we have B1, B2, B3 and S1, S2, S3.  What do those numbers mean?

If we google “azure app service s1”, the first result is “pricing”.

{% include dropbox.html img="https://d2mxuefqeaa7sj.cloudfront.net/s_250BBE10AD6C9355F608E73A8DC5B5251AA028E0CC4DBBE3DDA708DD0C5C1B22_1553284323304_image.png" %}



That link will take us to another page, which looks a bit complex. If we change the operating system to “Linux” and scroll down a bit, things will start to make sense. 

Down in the section title “Basic Service Plan”, you’ll see those B1, B2, and B3 identifiers. These are levels of the “Basic” service plan. On each one of these you can see how many CPU cores you get, how much memory, how much storage and how much all of that costs.

{% include dropbox.html img="https://d2mxuefqeaa7sj.cloudfront.net/s_250BBE10AD6C9355F608E73A8DC5B5251AA028E0CC4DBBE3DDA708DD0C5C1B22_1553284498905_image.png" %}


For the B1 plan, we’re basically looking at approximately 38$ per month. There’s another way to figure this out besides doing math here. If you scroll back up and go to Calculator in the nav menu, you get a screen where you can select the product you want to price and then select the operating system and tier. Notice that for App Serivce, Windows is actually about $16 bucks more than Linux. Why is that? I don’t know, but cheaper is better and we’re going with Linux anyway.

{% include dropbox.html img="https://d2mxuefqeaa7sj.cloudfront.net/s_250BBE10AD6C9355F608E73A8DC5B5251AA028E0CC4DBBE3DDA708DD0C5C1B22_1553284714537_image.png" %}


Back in VS Code, we’ve got a simple HTTP Server, so I think B1 is all the beef we need for this hamburger.

{% include dropbox.html img="https://d2mxuefqeaa7sj.cloudfront.net/s_250BBE10AD6C9355F608E73A8DC5B5251AA028E0CC4DBBE3DDA708DD0C5C1B22_1553284791030_image.png" %}


Do note that the size of power of your server will affect your npm install. We’re not doing any npm installs here, so we don’t care about that. But if we had a big project with a big npm install, it could take quite some time on something like the B1 tier.

Now we need to select our data center. I’m going to pick the one that’s closest to me. You do you.

{% include dropbox.html img="https://d2mxuefqeaa7sj.cloudfront.net/s_250BBE10AD6C9355F608E73A8DC5B5251AA028E0CC4DBBE3DDA708DD0C5C1B22_1553284864164_image.png" %}


And now VS Code will start to create your new App Service site…

{% include dropbox.html img="https://d2mxuefqeaa7sj.cloudfront.net/s_250BBE10AD6C9355F608E73A8DC5B5251AA028E0CC4DBBE3DDA708DD0C5C1B22_1553284892471_image.png" %}


And when it’s done, it’s going to ask you if you want to deploy it….

{% include dropbox.html img="https://d2mxuefqeaa7sj.cloudfront.net/s_250BBE10AD6C9355F608E73A8DC5B5251AA028E0CC4DBBE3DDA708DD0C5C1B22_1553284914299_image.png" %}


Go ahead and say “Yes”. Then it asks which folder you want to deploy. Just select the current folder from the menu. After you do this once, it’s going to ask if you want to run npm install on the target server. We’re going to say “yes”, even though we don’t have any npm install to run. We’ll get to that in a minute. 

{% include dropbox.html img="https://d2mxuefqeaa7sj.cloudfront.net/s_250BBE10AD6C9355F608E73A8DC5B5251AA028E0CC4DBBE3DDA708DD0C5C1B22_1553285068888_image.png" %}


Then it’s going to ask you if you want to always deploy the current directory when doing a deployment. Say yes to that as well so it doesn’t ask us every single time we do a deploy.

{% include dropbox.html img="https://d2mxuefqeaa7sj.cloudfront.net/s_250BBE10AD6C9355F608E73A8DC5B5251AA028E0CC4DBBE3DDA708DD0C5C1B22_1553285097007_image.png" %}


And after a minute or so, your site will be deployed, and you can select “Browse Website”.

{% include dropbox.html img="https://d2mxuefqeaa7sj.cloudfront.net/s_250BBE10AD6C9355F608E73A8DC5B5251AA028E0CC4DBBE3DDA708DD0C5C1B22_1553285140765_image.png" %}


A browser window will load and…it takes a while to load….quite a long while actually….still loading. But eventually, we get this…

{% include dropbox.html img="https://d2mxuefqeaa7sj.cloudfront.net/s_250BBE10AD6C9355F608E73A8DC5B5251AA028E0CC4DBBE3DDA708DD0C5C1B22_1553285302808_image.png" %}


What happened?!? Why is our site not loading? It works locally so why doesn’t it work in Azure? Go ahead and click on that “diagnostic resources” link. This will open the Azure portal and go to the “diagnose and solve problems” section for this site.

Scroll down until you see an orange triagle with an exclamation mark. That probably means trouble. Those are our logs. If we look through this section - there are two logs - an application log and a docker log. A DOCKER LOG? What is going on here?

Underneath the hood, Azure App Service is loading your application into a Docker container. You may not know anything about Docker, and that’s ok. Just know that we have two log files here. The first one reports on our application, and the second reports on the docker container that our application is running in. If we look at the second log there is an error….


    2019-03-22 20:07:42.868 ERROR - Container simple-http-server_0 for site simple-http-server did not start within expected time limit. Elapsed time = 230.6462151 sec

It looks like what has happened is that the container that App Service is using to host my Node app didn’t start. And the reason for that is highlighted in yellow. Azure is trying to help us here.

{% include dropbox.html img="https://d2mxuefqeaa7sj.cloudfront.net/s_250BBE10AD6C9355F608E73A8DC5B5251AA028E0CC4DBBE3DDA708DD0C5C1B22_1553285777843_image.png" %}


See that port? There’s actually two of them, separated by a colon. This is called port publishing in Docker. What it means is that the port 57347 inside the container is published to port 8080 outside the container. When our application is accessed in Azure, we actually hit port 8080 on the container, which is mapped to port 57347 inside the container. That means that Azure expects our app to run on port 57347. But what port is our app running on? Do you remember? 

If we scroll up and look at the log section above, you can see what hostname and port App Service is trying to start our application on.

{% include dropbox.html img="https://d2mxuefqeaa7sj.cloudfront.net/s_250BBE10AD6C9355F608E73A8DC5B5251AA028E0CC4DBBE3DDA708DD0C5C1B22_1553529263345_image.png" %}


We’re trying to bind to local host and run on port 3000. But Azure is trying to start our application on port 8080. And we have a second and sort of common Docker problem. We can’t bind to the host 127.0.0.1 because we’re in a Docker container. So let’s fix both of these issues.

First, instead of binding to 127, we can bind to 0.0.0.0. 

{% include dropbox.html img="https://d2mxuefqeaa7sj.cloudfront.net/s_250BBE10AD6C9355F608E73A8DC5B5251AA028E0CC4DBBE3DDA708DD0C5C1B22_1553529758090_image.png" %}


Now you may be thinking - does this still run locally? YES. It does. Those 4 zeros basically mean “no ip address”, so the computer goes ahead and claims it as it’s own. Because it doesn’t exist. This is all part of the IP4 specification, which is great reading if you’re having trouble getting to sleep.

Now let’s also fix out port problem. One way would be to set our port to 8080. But a better way is to set the value of the port to the PORT environment variable. This value is set automatically by Azure, so we can just bind to that. And since we still want it to run locally on port 3000, we can fall back to that in case the environment variable isn’t found.

{% include dropbox.html img="https://d2mxuefqeaa7sj.cloudfront.net/s_250BBE10AD6C9355F608E73A8DC5B5251AA028E0CC4DBBE3DDA708DD0C5C1B22_1553529792390_image.png" %}


If we wanted to run this locally and pass in that variable, you can run the following from the terminal


    $> PORT=3001 node index.js

And our app will be running on port 3001.

{% include dropbox.html img="https://d2mxuefqeaa7sj.cloudfront.net/s_250BBE10AD6C9355F608E73A8DC5B5251AA028E0CC4DBBE3DDA708DD0C5C1B22_1553529928235_image.png" %}


So let’s deploy again. We can just right-click anywhere in the empty space on the sidebar and choose “Deploy to Web App”.

{% include dropbox.html img="https://d2mxuefqeaa7sj.cloudfront.net/s_250BBE10AD6C9355F608E73A8DC5B5251AA028E0CC4DBBE3DDA708DD0C5C1B22_1553530020665_image.png" %}


Now try and browse your site after the deployment finishes. VICTORY!

I hope you enjoyed this video. Make sure to check out the other Azure Casts, and I’ll see you again soon. 
