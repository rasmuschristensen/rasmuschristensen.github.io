---
layout: post
title:  "Facebook Auth with Xamarin Forms and WebAPI CustomGrant - part 1"
description:  "Facebook Auth with Xamarin Forms and WebAPI CustomGrant - part 1"
date:   2016-01-06
---

<p class="intro">
<span class="dropcap">S</span>ocial authention with providers is almost a standard in many of todays mobile apps. A lot of authentication is provided either 
directly from the mobile app itself or by an API. In this blog post I'll show how to do explicit authentication with Facebook in an app built with Xamarin and ASPNET WebAPI, to enable bearer token based authentication.
Part 1 will focus on the app and part 2 will focus on the API.
</p>

### A little background
Some applications have only the client itself and no backend. I often build apps where the client is supported by a backend API. My preferred API technology is ASPNET WebAPI.
When it comes to authentitation with social providers, you'll often find a lot about how the WebAPI can be setup to support clients like Facebook, Twitter etc. Often by redirecting the client
to some login page hosted by the API. While this is often fine and also for web applications, there are other solutions and also very simple solutions to support your own authentication/registration flow.
I've worked with both ThinkTecture IdentityServer and ASPNET WebAPI, but inspired by a lot of reading on [BitOfTech](http://bitoftech.net), I learned a lot more
of how to use WebAPI. With a background developing Single Page Applications with AngularJS, I got a lot of inspiration from especially this series [blog post](http://bitoftech.net/2014/06/01/token-based-authentication-asp-net-web-api-2-owin-asp-net-identity/).
Just like Single Page Web applications, token based authentication is a perfect fit for mobile applications.

### The mission
We'll build a mobile Xamarin app, where the app requests Facebook to get an facebook access token __(1)__. Afterwards we'll verify __(2)__ and exchange this token with an access token (bearer token) to our system __(3)__.
<img src="{{ '/assets/img/app-fb-api.png' | prepend: site.baseurl }}" alt="setup">

### Register the app with facebook
First we need an app where we can authenticate with Facebook. The app will request Facebook for a loginpage provided by Facebook.
 This requires the app to be registered inside the [Facebook Developer Portal](https://developers.facebook.com/). We just need to register it as a simple 
 web application with a valid  __OAuth redirect URIs__ to be used when authentication succeeds. When your app is registered, you'll end up with a __ClientId__,__ClientSecret__ and a __OAuth redirect URIs__. The latter needs to be a valid
 endpoint to the API we'll be creating in part 2.
 
### Building the app
The app will be created using Xamarin Forms. During this post we'll only focus on the authentication, making a simple app that will redirect to authentication as soon as it starts. I a realworld application you might want to check settings
to see if the user already did sign in.
Because we're using Xamarin Forms we'll create a __CustomRenderer__ for each platform IOS/Android. The PageRenderer for IOS is displayed here and the source is available from [GitHub](https://github.com/rasmuschristensen/SimpleOAuth)

{% highlight csharp %}
public override void ViewDidAppear (bool animated)
{
    base.ViewDidAppear (animated);
    OAuth2Authenticator auth = null;
              
    auth = new OAuth2Authenticator (
        clientId      : "1519399498360864",
        scope: "email",
        authorizeUrl: new Uri ("https://www.facebook.com/dialog/oauth"),
        redirectUrl: new Uri ("http://windows:8080/login_success.html"));

    // we do this to be able to control the cancel flow outself...
    auth.AllowCancel = false;

    auth.Completed += async (sender, e) => {

        if (!e.IsAuthenticated)
            return;
        else {
            var access = e.Account.Properties ["access_token"];

            var client = new HttpClient (new ModernHttpClient.NativeMessageHandler ());

            var content = new FormUrlEncodedContent (new[] {
                new KeyValuePair<string, string> ("accesstoken", access),
                new KeyValuePair<string, string> ("grant_type", "facebook")
            });


            var authenticateResponse = await client.PostAsync (new Uri ("http://windows:8080/Token"), content);


            if (authenticateResponse.IsSuccessStatusCode) {

                //get api access bearer token from api
                // store user is logged in, request additional info...
            }

            ((App)App.Current).PresentMain ();
        }
    };			
        
    UIViewController vc = auth.GetUI ();

    ViewController.AddChildViewController (vc);
    ViewController.View.Add (vc.View);

    // add out custom cancel button, to be able to navigate back
    vc.ChildViewControllers [0].NavigationItem.LeftBarButtonItem = new UIBarButtonItem (
        UIBarButtonSystemItem.Cancel, async (o, eargs) => await App.Current.MainPage.Navigation.PopModalAsync ()
    );
}
{% endhighlight %} 
 
 The code above is very explicit, all the auth settings might be stored inside app settings, but in this sample app we don't hide anything. The clientId is the FaceBook clientId we got earlier when we registered the app on facebook
 It's important that the __redirectUrl__ matches the one also registered on facebook as __OAuth redirect URIs__.
 
If everything is setup and the above app is started in either emulator or device, the app should start and immediately redirect to a webview with the facebook login page. This happens because we create the OAuth2Authenticator by using the 
Xamarin.Auth component and make a request to FaceBook OAuth Url. When you enter your username and password for Facebook, the __Completed__ event will be fired and we'll get a status of authenticated or Unauthenticated. Here we expect it to authenticated.
Authenticated also means we'll receive the users facebook access token inside the event arguments, which we'll use to register/signup in our own backend. Before we finally accept the user as being fully authenticated we need to register with the API. As shown above, we do this by making a simple http POST request to our API _(This API will be created in part 2, for now we just expect it to be there)_. Finally when we get the api access token of type bearer from the API request and we are now ready to proceed with our app.

Source code is available on [GitHub](https://github.com/rasmuschristensen/SimpleOAuth)

 
 