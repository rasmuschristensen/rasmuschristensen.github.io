---
layout: post
title:  "Explicit Facebook Auth with Xamarin Forms and WebAPI CustomGrant part 1"
date:   2016-01-04
---

<p class="intro">
<span class="dropcap">S</span>ocial authention with providers is always a standard in many mobile applications today. A lot of authentication is provided either 
directly from the mobile app itself or by an API. In this blog post I'll show how to do explicit authentication with Facebook in an app built with Xamarin and ASPNET WebAPI to enable token based authentication.
Part 1 will focus on the app and part 2 will focus on the API.
</p>

### A little background
Some applications have only the client itself and no backend. I often build apps where the client is supported by a backend API. My preferred API solution is based on ASPNET WebAPI.
When it comes to authentitation with social providers, you'll often find a lot about how the WebAPI can be setup to support clients like Facebook, Twitter etc. Often by redirecting the client
to some login page. While this is often fine and also for web applications, there are other solutions and also very simple solutions.
I've worked with both ThinkTecture IdentityServer and WebAPI, but inspired by a lot of reading on [BitOfTech](http://bitoftech.net) I learned a lot more
of how to use WebAPI. With a background developing Single Page Web Applications using tokens, I got a lot of inspiration from this [blog post](http://bitoftech.net/2014/06/01/token-based-authentication-asp-net-web-api-2-owin-asp-net-identity/).

When it comes to mobile applications, token based authentiation is also a great fit.

### The mission
We'll build a mobile Xamarin app, where the app requests Facebook to get an facebook access token __(1)__. Afterwards we'll verify __(2)__ and exchange this token with an access token (bearer token) to our system __(3)__.
<img src="{{ '/assets/img/app-fb-api.png' | prepend: site.baseurl }}" alt="setup">

### Register the app with facebook
First we need an app where we can authenticate with Facebook. The app will request Facebook for a loginpage provided by Facebook.
 This requires the app to be registered inside the [Facebook Developer Portal](https://developers.facebook.com/). We just need to register it as a simple 
 webapp with a OAuth redirect URIs to be used when authentication succeeds. When your app is registered, you'll end up with a __ClientId__,__ClientSecret__ and a OAuth redirect URIs. The latter needs to be a valid
 endpoint to the API.
 
### Build the app
The app will be created using Xamarin Forms. During this post we'll only focus on the authentication, making a simple app that will redirect to authentication as soon as it starts.
Because we're using Xamarin Forms we'll create a __CustomRenderer__ for each platform. 
 
 
 
 