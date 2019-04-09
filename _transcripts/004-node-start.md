---
title: "#004 - When NodeJS Refuses to Start"
number: "003"
layout: transcript
---

Hey Friends

In this video, we’re going to look at how Azure actually runs Node applications, and the different ways that you can instruct it to run yours. 

I’m a huge VS Code fan, so I use the App Service extension for VS Code when I”m working with Azure. You can get that from the VS Code extension gallery.

I’ve got an extremely simple Node app that is actually just the Node.js getting started guide code. I’ve got it deployed on Azure and it’s running like a champ. You can check out episode 2 if you want to see how I did this miracle of coding simplicity.


{% include dropbox.html img="https://paper-attachments.dropbox.com/s_1E9A65461FFC5351E51720438BDB10923512EFD2EF76EFE2694A6BD756F61F4F_1554474629033_image.png" %}


Let’s upgrade this app a bit by adding in the Express web server package and serving up an actual HTML file. I’m going to create a new Git branch to do that since this is a pretty big change.


{% include dropbox.html img="https://paper-attachments.dropbox.com/s_1E9A65461FFC5351E51720438BDB10923512EFD2EF76EFE2694A6BD756F61F4F_1554474877468_image.png" %}


We can install the Express package from npm. Before we do that, let’s create a package.json file. Every Node project should have one of those. We can do that with npm init. And here’s a neat trick that Elijah Manor taught me - if you don’t want have to answer all these questions here, just pass the -y flag


    npm init -y


{% include dropbox.html img="https://paper-attachments.dropbox.com/s_1E9A65461FFC5351E51720438BDB10923512EFD2EF76EFE2694A6BD756F61F4F_1554474948775_image.png" %}


Now let’s install Express…


    npm install express

You can also just use the “i” shortcut and do…


    npm i express

I’m going to change out the code in the `index.js` file to be a simple Express server that just returns an `index.html` file.

```js
const express = require("express");
const app = express();
const path = require("path");
const port = process.env.PORT || 3000;

// viewed at http://localhost:3000
app.get("/", function(req, res) {
  res.sendFile(path.join(__dirname + "/index.html"));
});

app.listen(port, () => console.log(`App listening on port ${port}!`));
```

And let’s add an `index.html` file for Express to return. You can use the built-in Emmet support in VS Code to quickly scaffold out an HTML file by just typing an exclamation mark - or a bang if you will and then hitting tab. Then you can tab through all of the tab stops until you get to the body. Nifty. Emmet is a super quick way to create HTML. Let’s add an H1 and return a title. And let’s create a p tag with some lorem ipsum text. And emmet helps us create that lorem ipsum text too. Emmet is so neat.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>My App</title>
  </head>
  <body>
    <h1>My Express App</h1>
    <p>
      Lorem ipsum dolor sit amet consectetur adipisicing elit. Quod aperiam
      minima earum veritatis rem asperiores nam maxime tempore aut? Excepturi
      ipsam nisi laudantium eius deserunt eaque vel hic mollitia nemo.
    </p>
  </body>
</html>
```

Now we’re starting to get a lot of files in this project. Let’s move the `index.js` and `index.html` files into an `src` folder.


{% include dropbox.html img="https://paper-attachments.dropbox.com/s_1E9A65461FFC5351E51720438BDB10923512EFD2EF76EFE2694A6BD756F61F4F_1554475996973_image.png" %}


Let’s run it locally with `node src/index.js`. Lovely. Doesn’t it feel good when things work the way you want? I love it.


{% include dropbox.html img="https://paper-attachments.dropbox.com/s_1E9A65461FFC5351E51720438BDB10923512EFD2EF76EFE2694A6BD756F61F4F_1554475968186_image.png" %}



Let’s go ahead and deploy this. It tells us to check the output window for details, so let’s do that. Open up the bottom panel in VS Code with `Cmd/Ctrl + J`. Select the “Output” tab and then select “App Service” from the dropdown. Here we can watch App Service chug along. You can see that it’s creating a zip file here in the background and then uploading it. 


{% include dropbox.html img="https://paper-attachments.dropbox.com/s_1E9A65461FFC5351E51720438BDB10923512EFD2EF76EFE2694A6BD756F61F4F_1554478496855_image.png" %}


Another thing that you can see here is that it’s zipping up the entire node_modules folder and then uploading it.


{% include dropbox.html img="https://paper-attachments.dropbox.com/s_1E9A65461FFC5351E51720438BDB10923512EFD2EF76EFE2694A6BD756F61F4F_1554478542502_image.png" %}


Yikes. We don’t want that. That folder will get enormous as our app gets bigger and Azure is going to run an `npm install`  as part of this Oryx build command anyway. Uploading the `node_modules` folder is super inefficient.  Let’s instruct VS Code to please *not* do that.

Open the settings.json file in the .vscode folder.  This setting here for `appService.zipIgnorePattern` looks promising. If we’re not sure, we can just put our mouse over the settings and it will show us in a tooltip what this settings does.


{% include dropbox.html img="https://paper-attachments.dropbox.com/s_1E9A65461FFC5351E51720438BDB10923512EFD2EF76EFE2694A6BD756F61F4F_1554476424773_image.png" %}


That looks right. Let’s add a pattern to ignore the `node_modules` folder. I don’t know how to write ignore patterns, but I know how to copy this line here and change it to “node_modules”. I can copy/paste code with the best of them.

```js
{
  "appService.zipIgnorePattern": [
    ".vscode{,/**}",
    "node_modules{,/**}"
  ],
  ...
}
```

If you open that `.deployment` file that was created by VS Code when we first uploaded this site, you can see a setting. This is the setting that tells AppService to do an `npm install` and pull in all of our dependencies.


    [config]
    SCM_DO_BUILD_DURING_DEPLOYMENT=true

Now let’s do our deployment again.

And let’s open up the output panel and see what Azure is up to this time…


{% include dropbox.html img="https://paper-attachments.dropbox.com/s_1E9A65461FFC5351E51720438BDB10923512EFD2EF76EFE2694A6BD756F61F4F_1554479032980_image.png" %}


Ah - ok - so it’s deleting everything and redeploying, and this time we don’t get those logs about it copying over all of our node_modules because we didn’t upload any. Instead it just does the npm install - you know - the way god intended.

And let’s check it out on Azure to make sure it works…


{% include dropbox.html img="https://paper-attachments.dropbox.com/s_1E9A65461FFC5351E51720438BDB10923512EFD2EF76EFE2694A6BD756F61F4F_1554479097525_image.png" %}


Wait. What? This is the default page you get BEFORE you deploy anything. What is going on? Did I screw up or did Azure screw up?

First, let’s just make sure that our files are actually there. This page here makes it seem like they aren’t, but we can know for sure by going to the AppService extension for VS Code and expanding the site and then going to files.


{% include dropbox.html img="https://paper-attachments.dropbox.com/s_1E9A65461FFC5351E51720438BDB10923512EFD2EF76EFE2694A6BD756F61F4F_1554479270204_image.png" %}


My files are definitely there. I just had this working. Let’s roll back to where we started with the simple “index.js” file that was working and see what happens. I’m going to do that by just switching back to the master branch.


{% include dropbox.html img="https://paper-attachments.dropbox.com/s_1E9A65461FFC5351E51720438BDB10923512EFD2EF76EFE2694A6BD756F61F4F_1554480719773_image.png" %}


Let’s deploy this again and see what we get - just to make sure we aren’t going crazy here. This did work before, right?


{% include dropbox.html img="https://paper-attachments.dropbox.com/s_1E9A65461FFC5351E51720438BDB10923512EFD2EF76EFE2694A6BD756F61F4F_1554480774726_image.png" %}


I’m officially confused. Let’s go back to the Express project by switching branches and deploying. We know that this doesn’t work, so let’s go to the log files. We can get there from the AppService extension. We’ve got a few options in here. One of them is to connect to the Log Stream. This will stream the application logs right into VS Code. Another one is to just look at the log files themselves. There is only one right now with this date format in front of it and it says “Docker”


{% include dropbox.html img="https://paper-attachments.dropbox.com/s_1E9A65461FFC5351E51720438BDB10923512EFD2EF76EFE2694A6BD756F61F4F_1554481782169_image.png" %}


Why is there a “Docker” log? Behind the scenes AppService is running our app in a Docker Container. That’s how AppService works - it’s all based on containers. You can bring your own, or you can just deploy files, but they will end up in a container once they’re in Azure.

We don’t have any application logs yet, and that’s because they aren’t on be default. If we right-click the “Logs” folder, we can select “Enable File Logging” and that will restart the app and enable the file logs.


{% include dropbox.html img="https://paper-attachments.dropbox.com/s_1E9A65461FFC5351E51720438BDB10923512EFD2EF76EFE2694A6BD756F61F4F_1554481924768_image.png" %}


Now click the refresh icon in the extension taskbar, and we see a second file. 


{% include dropbox.html img="https://paper-attachments.dropbox.com/s_1E9A65461FFC5351E51720438BDB10923512EFD2EF76EFE2694A6BD756F61F4F_1554482481085_image.png" %}


This file is the application log. Let’s have a peek.

Alright. As we scroll down here we can see up at the top that Azure is reporting that it can’t find a startup command or autodetected startup script. And that’s it’s running the static default site. So I guess that explains why we’re seeing the static default site.


{% include dropbox.html img="https://paper-attachments.dropbox.com/s_1E9A65461FFC5351E51720438BDB10923512EFD2EF76EFE2694A6BD756F61F4F_1554482588476_image.png" %}


Below that we’ve got some ASCII art that says “PM2”. Let’s Google it. Below that it says it’s running pm2 start to start the application. What the heck is PM2? Let’s Google it.


{% include dropbox.html img="https://paper-attachments.dropbox.com/s_1E9A65461FFC5351E51720438BDB10923512EFD2EF76EFE2694A6BD756F61F4F_1554482701068_image.png" %}


It’s a Node process manager. It must be baked into our container behind the scenes.

OK, so Azure can’t start our site. But it *could* start it when we just had an “index.js” file.  So lets go back and move our index.js and public.html files back to the root and deploy.


{% include dropbox.html img="https://paper-attachments.dropbox.com/s_1E9A65461FFC5351E51720438BDB10923512EFD2EF76EFE2694A6BD756F61F4F_1554483038566_image.png" %}


And let’s see if that works.


{% include dropbox.html img="https://paper-attachments.dropbox.com/s_1E9A65461FFC5351E51720438BDB10923512EFD2EF76EFE2694A6BD756F61F4F_1554483175386_image.png" %}


That works! Azure follows a convention here. If you don’t tell it how to start your app, it tries to guess. It will look for “index.js”, “server.js”, “main.js”, “app.js” or even “bin/www” which is the starting point for an Express application if you scaffold it with the Express CLI.

But we should really be telling Azure how to start this site. We can do that by adding a startup script to our `package.json` file. So let’s put our files back in the “src” folder. Then we’ll add a startup script. And change our index.html so we’re sure our change is picked up.

OK. That works. Azure can now start our site no matter how we structure it or what we name our files.


{% include dropbox.html img="https://paper-attachments.dropbox.com/s_1E9A65461FFC5351E51720438BDB10923512EFD2EF76EFE2694A6BD756F61F4F_1554483514704_image.png" %}


Now it’s important to note that Aure using PM2 to start our app is kind of important here. Let me show you why.

When you are running a Node application, you run the risk of it crashing if you have an unhandled error. We can make this happen pretty easily. Let’s add another route here that tries to read a file that doesn’t exist into a stream.

```js
const express = require("express");
const app = express();
const path = require("path");
const fs = require("fs");
const port = process.env.PORT || 3000;

// viewed at http://localhost:3000
app.get("/", function(req, res) {
  res.sendFile(path.join(__dirname + "/index.html"));
});

app.get("/read", function(req, res) {
  const stream = fs.createReadStream("does-not-exist.txt");
});

app.listen(port, () => console.log(`Example app listening on port ${port}!`));
```

If you visit this site in the browser. Everything looks all hunky dory.


{% include dropbox.html img="https://paper-attachments.dropbox.com/s_1E9A65461FFC5351E51720438BDB10923512EFD2EF76EFE2694A6BD756F61F4F_1554484469781_image.png" %}


But if you go to /read and execute that bad line of code….


{% include dropbox.html img="https://paper-attachments.dropbox.com/s_1E9A65461FFC5351E51720438BDB10923512EFD2EF76EFE2694A6BD756F61F4F_1554484510412_image.png" %}


Uh oh. That didn’t work. And here’s the really bad part. If you go back to the root URL. That one isn’t working anymore either.


{% include dropbox.html img="https://paper-attachments.dropbox.com/s_1E9A65461FFC5351E51720438BDB10923512EFD2EF76EFE2694A6BD756F61F4F_1554484543363_image.png" %}


This whole app is dead in the water. Have a look in the VS Code terminal and you’ll see that Node has just crashed. This app is down for everyone. Not. Good.


{% include dropbox.html img="https://paper-attachments.dropbox.com/s_1E9A65461FFC5351E51720438BDB10923512EFD2EF76EFE2694A6BD756F61F4F_1554484583564_image.png" %}


PM2 (and other Node process managers) will watch your Node process, and if it goes down, they restart it. Automatically.

PM2 is just installed from `npm`  like any other Node package.


    npm i -g pm2

PM2 works off the concept of configuration files. We could create one of those by running `pm2 init`. Then we could specify our startup point there. But we already have a startup point in our package.json. So instead, we can just tell PM2 to read our start script by running…'


    pm2 start npm -- start

Notice the space between the dashes here - that’s important.


{% include dropbox.html img="https://paper-attachments.dropbox.com/s_1E9A65461FFC5351E51720438BDB10923512EFD2EF76EFE2694A6BD756F61F4F_1554485083824_image.png" %}


Now our app is running. It’s name is “npm” because that’s the process that PM2 executed. If we want to see the logs from our app, we can run `pm2 logs` and then pass in 0 - the ID of the process or “npm”, the name of the process.

Our app is still running on port 3000. If we go to the”read” endpoint to crash it, it will crash….


{% include dropbox.html img="https://paper-attachments.dropbox.com/s_1E9A65461FFC5351E51720438BDB10923512EFD2EF76EFE2694A6BD756F61F4F_1554485194100_image.png" %}


BUT! But but but. This time, go back to VS Code and you can see that the app did crash and PM2 just restarted it for us.


{% include dropbox.html img="https://paper-attachments.dropbox.com/s_1E9A65461FFC5351E51720438BDB10923512EFD2EF76EFE2694A6BD756F61F4F_1554485228263_image.png" %}


So if we go back to the root of our app - it’s still up!


{% include dropbox.html img="https://paper-attachments.dropbox.com/s_1E9A65461FFC5351E51720438BDB10923512EFD2EF76EFE2694A6BD756F61F4F_1554485263142_image.png" %}


Azure is using PM2 as well, so if we deploy this site to Azure - broken as it is, our app won’t crash and stay down. Azure will use PM2 to make sure that it recovers. That’s a very nice feature.

I hope you enjoyed this video. Please check out the other Azure Casts, and I’ll see you again soon.