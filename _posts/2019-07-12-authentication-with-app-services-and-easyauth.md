---
layout: video
title: "Authentication with App Services and EasyAuth"
number: "010" #set this to your episode
categories:
tags:
summary: "In this episode Rob sets up Azure's Authentication and Authorization service, otherwise known as EasyAuth, using an Express application, Google, and Twitter. Don't want to use an auth service or library? This is for you."
video: https://www.youtube.com/embed/2EPvfeIf4u4
author: Rob Conery
github: https://github.com/azurecasts/010-easyauth
transcript: 010-easyauth
minutes: "13:30"
twitter: robconery
---

Yes, yes I know I can "just use Passport" with a Node/Express application, but I wanted to explore what Azure's Auth service, informally known as EasyAuth, has to offer. In this video I turn the service on and setup login, logout, personalization and finally lockdown of protected resources. I had fun!

## Resource Links

You can read more about the Azure resources in this video here:

 - [Material Theme from Creative Tim](https://www.creative-tim.com/product/material-kit)
 - [Authentication and Authorization](https://docs.microsoft.com/en-us/azure/app-service/overview-authentication-authorization?WT.mc_id=azurecasts-website-robcon)
 - [Authentication and Authorization - Advanced Topics](https://docs.microsoft.com/en-us/azure/app-service/app-service-authentication-how-to?WT.mc_id=azurecasts-website-robcon)
 - [Setting Up Google](https://docs.microsoft.com/en-us/azure/app-service/configure-authentication-provider-google?WT.mc_id=azurecasts-website-robcon)
 - [Setting Up Twitter](https://docs.microsoft.com/en-us/azure/app-service/configure-authentication-provider-twitter?WT.mc_id=azurecasts-website-robcon)

## Transcript

I'm building a NodeJS web application using the Express Web Framework. I've downloaded and set up a lovely template from [Creative Tim](http://creative-tim.com) that you see here and I think it looks pretty swell - but it doesn't do anything! I'm going to change that today by doing the next logical thing: setting up user authentication.

{% include screenshot.html img="screenshot_304" %}

Now I could go the obvious route: using a prebuilt solution such as Passport, which is a super popular NodeJS authentication library. But I want to try something different - so today I'm going to plug in Azure's authentication and authorization service for App Services. That's a mouthful - so most people just call it EasyAuth.

It takes a bit to get used to - but it's a lot of fun so let's jump right in.

## Getting Setup

If you want to play along - which I hope you do - you'll have to get an Express app up and running on Azure. You can download the source for this video from GitHub and then build your Azure Resources using the Makefile in the azure directory. I go over what this means in [episode #005](https://azurecasts.com/2019/04/11/005-using-make-to-orchestrate-shell-scripts/)

{% include screenshot.html img="screenshot_303" %}

If you're building an Express app on your own please note that I did need to install the `express-session` package so that I could use sessions. Here's my `app.js` setup:

{% include screenshot.html img="screenshot_305" %}

## Turning Authentication On

Every App Service on Azure has access to Authentication and Authorization - _EasyAuth_ - all you have to do is to turn it on which you can do by heading to the portal, clicking on the "Authentication and Authoriation" link and then turning it on, as you see here:

{% include screenshot.html img="screenshot_306" %}

As you can see, this service is still in preview which means you can expect things to change and that we might also have a few bumps in the road. Hopefully not... but just in case... you've been warned.

Turning on EasyAuth allows users to log in to your site using a variety of different services, including Azure's Active Directory, Facebook, Google, Twitter, or your own custom provider. I'll just be using Google and Twitter today and, as you can see, I've already configured them for use.

I'm going to assume you've used OAuth before and know the drill. If you don't, there are instructions linked in the show notes for how to create and register applications with Google, Twitter and Facebook. This takes just a few minutes and when you're done these services will give you authentication keys that you'll need to set in the Azure portal. As I mention - I've done this already so I'm just going to skip over that part in the interest of time.

Now that our Google and Twitter authentication keys are all set, we need to save those settings and stop/restart our App Service.

{% include screenshot.html img="screenshot_307" %}

## Dot Auth

Believe it or not, that's all we have to do as far as Azure is concerned, and this is where things get a bit strange. What just happened?

When we "flipped the switch", Azure added a few URLs to our site that we can now access. Specifically:

{% include screenshot.html img="screenshot_308" %}

When our users navigate to these URLs EasyAuth kicks in. For instance: navigating to `/.auth/login/twitter` will log them into Twitter and...

_Wait a minute_. This is confusing - at least I was super confused until I read this sentence in the Azure documentation. I tried to study that graphic but that... really didn't help much.

{% include screenshot.html img="screenshot_309" %}

Here's the deal: your App Service runs inside of a container which runs inside of an App Service Web Worker, which you can think of as a VM. When you turn on EasyAuth you pipe all of your requests through it, _before they even hit your app_. It's kind of like having Nginx run in front of an application server like PM2. In fact it's _exactly like that_.

The `/.auth` functionality sits completely outside your application - managed by Azure - which I think is kind of neat. The obvious question, however, is "how in the world do I get access to my user's information?" Good question - I'm getting there.

If I navigate to `/.auth/login/twitter` from my app, the call gets intercepted, as you can see, and I'm redirected off to Twitter. Azure handles the structuring of the call - there's nothing I need to do. 

{% include screenshot.html img="screenshot_310" %}

Once I click the blue button here and authorize the app I'm sent back ... to this screen ... which is _not_ my app ... but it looks like I'm logged in so hooray. I'll fix this in a second:

{% include screenshot.html img="screenshot_311" %}

I can logout by navigating to `/.auth/logout` and boom. I'm logged out! Or so this page is telling me:

{% include screenshot.html img="screenshot_312" %}

## Auth Me

OK so we can login and logout, but where is this user information and how can I access it? This gets tricky fast - but the short answer is that you can see your login information at `/.auth/me`:

{% include screenshot.html img="screenshot_313" %}

That's a big dump of JSON and it contains all the stuff that Twitter forwarded to Azure - or rather to EasyAuth. Non of this is stored _anywhere_ outside your App Service web worker - it's all in memory and has an expiration date. You can save this, if you want, but it involves a small hack we'll see later on.

## Setting up Redirects

We obviously don't want users seeing the default login page to let's fix that by appending a `post_login_redirect_url` parameter to our login links. As you can see, I've added the `/.auth/login/` url to my social login links - now I just need to tell them where to redirect to. I'm using an absolute URL here but you can also use a relative URL since the redirect is coming from within your domain... nifty!

{% include screenshot.html img="screenshot_315" %}

To see the changes I'll need to redeploy, and this is when I realize the one big downside of using a hosted auth system like this: _I need to redeploy on any changes I make_. This can be a bummer, but the system is so simple and deployment is so fast that it doesn't really bother me. There are probably ways to intercept this stuff but for now I'll just deploy a lot.

{% include screenshot.html img="screenshot_314" %}

OK - refreshing and logging in again... success.

### A Better Redirect

Right now I'm redirecting my users to the root of my site, but there's a better way to do this which will allow me to track user information coming in, and that is to set up a `/login/success` route:

{% include screenshot.html img="screenshot_316" %}

As I mentioned before: when EasyAuth is turned on it handles all incoming requests. It'll look for a session cookie that it has planted on the client and if the user is logged in, using _any_ of the auth service providers, it will forward a set of headers to your application. These headers are never seen by the client - they are internal only.

There are two in particular that I'm interested in: the `x-ms-client-principle-id` and `x-ms-client-principle-name`. The `id` header will tell me the unique ID that the service uses to identify my user. The name is the common name, if any. 

{% include screenshot.html img="screenshot_317" %}

I can use these values in my session and use them to identify this user in my database. For isntance: if they buy an order, I can save the principle id and that will tell me who bought it. I can gather other information at checkout if I need.

{% include screenshot.html img="screenshot_319" %}

I'll save all of this information in the server's session for now and then I'll redirect off to the root of my site - I'll change this logic later on. If I don't have a header for some reason I'll send them off to the login page.

Now I just need to update my links to redirect to this new URL:

{% include screenshot.html img="screenshot_318" %}

## Setting a Current User

Now that I'm saving user information to the session I'll want to make it available to my views. I don't want to have to send it in every time, so I'll add a piece of custom middleware right above my routes - which is important! If you put this below your routes they won't have access to it.

{% include screenshot.html img="screenshot_320" %}

I like being explicity about this kind of thing, so I'll make sure there's _some kind_ of user exposed, with a `loggedIn` property I can check.

I can now update my site's navigation to be a bit more personalized. I can check if the user is logged in and if there are I can say hello and show a logout link. Otherwise the login button will remain.

{% include screenshot.html img="screenshot_321" %}

The only way we'll get to see if this works is by deploying again. Yes, this can be a pain but honestly this is pretty straightforward stuff so hopefully we won't have too many redeploys.

Trying it out - and it works!

{% include screenshot.html img="screenshot_322" %}

## Logging Out and Killing the Session

Sort of. If I click "Logout" I still see this page. Worse yet - I know I've been logged out by the system but my information is still showing! That's not good - but it's easy to fix as the problem is that my session is still alive in Express.

I can fix that by adding a querystring parameter here, which is oddly called `post_logout_redirect_uri` (as opposed to url) and I'll do the same kind of thing I did with logging a user in: I'll redirect to a special URL that kills the session:

{% include screenshot.html img="screenshot_323" %}

Redeploying one more time - you get used to this after a while - and hey, look at that it works.

## Personalization

When I login to the site I have a nice greeting (with crappy formatting) - but it would be nice if it said my name, not my Twitter handle or Gmail address. There's so much lovely information available in `/.auth/me` that I can use - so let's use it!

{% include screenshot.html img="screenshot_324" %}

The neat thing is that I can use JavaScript to personalize my site - I don't need to change any server code. Some see this as a major strength of EasyAuth, other's like to use server-side code whenever they can. I'll show you how to do that in a bit, for now let's take the path least resistance.

### Client Side

I have this lovely free template from Creative Tim - the Material Template free version - and it has a groovy chunk of HTML I can use for displaying user information. I'll plug that in instead of my ugly greeting. Notice that I've added a class name to the user's avatar as well as given an id to the name heading. I'll be setting that in just a minute.

{% include screenshot.html img="screenshot_325" %}

I'm going to keep this as simple as I can by not using a framework other than Axios, which I like a lot for JSON requests. If you're a Vue or React fan - go for it.

Because I'm a bit of a diehard server-side guy I'm going to wrap this JavaScript in an `if` statement because I don't want it going off if the user isn't logged in. That's me, and I'm not sorry.

{% include screenshot.html img="screenshot_326" %}

I'll start by creating an `async` function that pings `/.auth/me` and pulls down some JSON. The returned JSON is, oddly, an array so I need to work with that and save the first element (which is the data I want) to a variable:

{% include screenshot.html img="screenshot_327" %}

{% include screenshot.html img="screenshot_328" %}

Now it's just a matter of turning that array into a usable object - which I'll call `user`. I'll pull `provider` and `user_id` off of the root, but I already have the `user_id` in my session code, so neither of these things is going to help me personalize my site. I need access to the _claim data_.

This is another weird data formatting choice. Each user claim is returned as an array element which is an object with a `typ` key and a `val` key. Not sure why this can't be its own object but... whatever.

{% include screenshot.html img="screenshot_329" %}

This claim information will vary based on the authentication provider too. If I logout here and log back in using Google, for instance, you can see that the claim information (my name, email, etc) is different. It would be nice if EasyAuth could formalize a return object - providing an id, name, email and avatar url with a `data` key that has the raw return attached to it. I'll recommend that to the team.

For now, I'll loop the array and build the object myself.

{% include screenshot.html img="screenshot_330" %}

To complete the task I need to invoke my function, using the return information to update the DOM. Notice here that I'm using `document.getElementsByClassName` for the avatars? The reason is that I might have multiple places where I want to show the user their lovely mug - such as on the profile page.

{% include screenshot.html img="screenshot_331" %}

You know what we get to do next! That's right... deploy! Logging in once again and hey! It's my Google avatar! This looks much better. Clicking my profile and I can see my daughter's happy screaming face - I call that success.

{% include screenshot.html img="screenshot_332" %}

### Server-side

Let's see if we can do all of this server-side, shall we? The short answer is that _we really can't_, we still need some client code but that's OK because it's super hacky and we can feel good about being bad.

I'll create a new view which I'll call `login-success.ejs` and I'll copy/paste, like a good front-end dev, the code I just wrote. At the top I'll output a message that says the login was successful and that the user is about to be redirected.

{% include screenshot.html img="screenshot_333" %}

Instead of updating the DOM, however, we'll just redirect the user to a new route, which I'll call `login-remember`. I'll send along the name and avatar - the two bits of information I need currently - and then save that information to the session. Ideally this would go into a database so I could access it later, but this works fine.

{% include screenshot.html img="screenshot_334" %}

{% include screenshot.html img="screenshot_335" %}

Personally, I like the client-side approach. Redirects after a login feel icky, but this is an option if you want to use it.

## Locking Things Down

OK, one last thing to do! I need a way to make sure that certain routes are not reachable unless a user is logged in. The standard way to do this with Express is to have an `ensureLoggedIn` middleware function that you can plug into your route.

{% include screenshot.html img="screenshot_336" %}

All I need to do here is to check for that special header. If it's present, we're good and I'll fire `next()`. If not I'll redirect to the login page:

{% include screenshot.html img="screenshot_337" %}

But wait! That's hacky! Let's be nice and remember where the user was trying to go before we redirected them - I can do that using the `originalUrl` property on the request. I'll save that in the session and in my `/login/success` route I'll check for it.

{% include screenshot.html img="screenshot_338" %}

Great let's try it out! After we deploy, of course...

Navigating to the profile page and I'm redirected, as I would expect. Logging in... and boom! I'm on my way. That's pretty seamless!

{% include screenshot.html img="screenshot_338" %}

Well that's it for me - thanks for watching and I hope you enjoyed! See you soon.