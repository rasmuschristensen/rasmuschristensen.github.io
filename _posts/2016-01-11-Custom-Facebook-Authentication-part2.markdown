---
layout: post
title:  "Facebook Auth with Xamarin Forms and WebAPI CustomGrant - part 2"
description:  "Facebook Auth with Xamarin Forms and WebAPI CustomGrant - part 2"
date:   2016-01-11
---

<p class="intro">
<span class="dropcap">I</span>n the previous post the mobile client was created. In this post we'll created the WebAPI backend part. We'll enable
the API to verify a Facebook access token and exhanged it with an access token to our backend, fully supported and integrated with the ASPNET WebAPI.
</p>

If you haven't read part1 you can find it [here](http://rasmustc.com/blog/Custom-Facebook-Authentication-with-webapi/)

### Token based authentication with ASPNET WebAPI
When working with a disconnected client like a mobile app, OAuth is one way to authenticate with a backend using tokens. Often when using ASPNET WebAPI there will be examples
using ASPNET Identity and entity framework. While this option is valid in many situations, it can in my opinion get a bit bloated with the different layers to support user auth for simple applications.
The goal of this post is to make a Token endpoint in the API, where we'll receive the Facebook access token, verify it and the exchange it for a valid access token to our system, area 2 and 3 in the figure below.
<img src="{{ '/assets/img/app-fb-api.png' | prepend: site.baseurl }}" alt="setup">

### Creating the project 
Start by creating a new Empty ASPNET project and only include the WebAPI references, we want to keep tings as simple as possible.
<img src="{{ '/assets/img/emptyapi.png' | prepend: site.baseurl }}" alt="newapi">

### Required nuget packages
Before adding any code, the following list of nuget packages must be installed.
It's very important the __Microsoft.Owin.Host.SystemWeb__ package is installed or else OWIN startup will not be executed.
{% highlight powershell %}
install-package Microsoft.Owin 
install-package microsoft.owin.cors
install-package Microsoft.Owin.Security.OAuth
install-package Microsoft.Owin.Host.SystemWeb
install-package Microsoft.AspNet.WebApi.Owin
install-package Facebook
{% endhighlight %}
For simplicity there is no use of Dependency Injection in this sample project, only the core server. The entrypoint is the __Startup__ class in the root of the solution.

### OWIN startup pipeline
The project will make use of the __OWIN__ pipeline, so we need some additional packages, compared to using Global.asax, these should already be added if all listed backages is installed. Note that Global.asax has been removed from the project.
<img src="{{ '/assets/img/slnOverview.png' | prepend: site.baseurl }}" alt="newapi">
#### Owin Start - Startup.cs
This is the main entrypoint to the API. In here we setup:

* __Route__ mapping to controllers is enabled.
* __Authentication__ is enabled.
* __CORS__ is enabled.

{% highlight csharp %}
[assembly: OwinStartup(typeof(Startup))]
namespace SimpleAPI
{
    public partial class Startup
    {
        public void Configuration(IAppBuilder app)
        {            
            var config = new HttpConfiguration();

            WebApiConfig.Register(config);
         
            ConfigureAuth(app);

            app.UseCors(Microsoft.Owin.Cors.CorsOptions.AllowAll);
            app.UseWebApi(config);
        }
    }
}
{% endhighlight%}

The __WebApiConfig.Register__ is just the default implementation in this example. Attributed routing is not included, we rely on default Controller and action mapping.

__ConfigureAuth__ is the most important part, implemented as partial on Startup named __Startup.Auth.cs__. It implements the core Token server setup and is added to the API just before the details to allow CORS (This is needed as or mobile clients is on another domain compared to our API)

#### Startup.Auth.cs - TokenServer setup

This class holds the configuration of the token server setup. First of all we tell the server to accept token requests on the __/Token__ endpoint.
If our API is hosted on __http://localhost:20000__, then it will be possible to request a token at __http://localhost:20000/token__. Meaning this is the endpoint our mobile client should request.
For now the access token expiration time is just set to a random value and there is no support to renew, beside manually send a request to the __/Token__ again.

{% highlight csharp %}
public partial class Startup
{
    public void ConfigureAuth(IAppBuilder app)
    {
        OAuthAuthorizationServerOptions OAuthServerOptions = new OAuthAuthorizationServerOptions()
        {
            AllowInsecureHttp = true,
            TokenEndpointPath = new PathString("/token"),
            AccessTokenExpireTimeSpan = TimeSpan.FromDays(14),  
            Provider = new SimpleAuthTokenProvider()
        };

        // Token Generation
        app.UseOAuthAuthorizationServer(OAuthServerOptions);
        app.UseOAuthBearerAuthentication(new OAuthBearerAuthenticationOptions());
    }        
}
{% endhighlight %}

The __Provider__ is where we can hook into the generation and validation of access tokens, implemented as SimpleAuthTokenProvider.

#### SimpleAuthTokenProvider.cs - CustomGrant 

The provider first of all needs to inherit __OAuthAuthorizationServerProvider__, this makes it possible to override the needed methods
__OAuthAuthorizationServerProvider__ and __OAuthAuthorizationServerProvider__ where the latter is where we want to put our custom logic.

{% highlight csharp %}
public override async Task GrantCustomExtension(OAuthGrantCustomExtensionContext context)
{
    if (context.GrantType.ToLower() == "facebook")
    {
        var fbClient = new FacebookClient(context.Parameters.Get("accesstoken"));

        dynamic response = await fbClient.GetTaskAsync("me", new { fields = "email, first_name, last_name" });
        
        string id = response.id;
        string email = response.email;
        string firstname = response.first_name;
        string lastname = response.last_name;
        // place your own logic to lookup and/or create users....

        // your choice of claims...
        var identity = new ClaimsIdentity(context.Options.AuthenticationType);
        identity.AddClaim(new Claim("sub", $"{firstname} {lastname}"));
        identity.AddClaim(new Claim("role", id));

        await base.GrantCustomExtension(context);
        context.Validated(identity);

    }            
    return;
}
{% endhighlight %}

The above code allows us to accept a custom __grant\_type__ from a caller and here we say the value should be __facebook__, a custom phrase we decide. This will also make it quite simple to extend the code
to allow other providers like google etc.
When we get a valid grant\_type, we fetch the __accesstoken__ from the request, again we decide what the key should be named. 

The key will contain a facebook access token as its value, the one we requested from Facebook in part1 and posted to the API.
In the beginning where we installed Nuget package, we also installed a Facebook package, this enables us to request Facebook, to verify the provided access token.
The fields we expect needs to match the setup in the app created in the Facebook developer portal.
When we get a response from Facebook, we can extract the expected values and in a real world scenario, we would then make some registration/lookup for the user in our system, but this part is up to you and the your system requirements.
All we do here is creating an identity and finally make the identity validated and that's it. no more, no less. Our server is now ready to run and provide access tokens to our system.

### Calling the Token endpoint from Xamarin
Now when we have our server created and running we can make the request from our mobile client created in [part1](http://rasmustc.com/blog/Custom-Facebook-Authentication-with-webapi/).

The code required to take the Facebook access token, send it to our API Token endpoint, and get an access token to our API in response is listed here. The full example is available on GitHub.

{% highlight csharp %}
using (var handler = new ModernHttpClient.NativeMessageHandler ()) 
{
    using (var client = new HttpClient (handler)) {
        var content = new FormUrlEncodedContent (new[] {
            new KeyValuePair<string, string> ("accesstoken", access),
            new KeyValuePair<string, string> ("provider", "facebook")
        });
            
        var authenticateResponse = await client.PostAsync (new Uri ("http://windows:8080/Token"), content);

        if (authenticateResponse.IsSuccessStatusCode) {

            var responseContent = await authenticateResponse.Content.ReadAsStringAsync ();
            var authenticationTicket = JsonConvert.DeserializeObject<AuthenticatedUser> (responseContent);

            if (authenticationTicket != null) {
                var apiAccessToken  = authenticationTicket.Access_Token;
                // Save it to App settings...
            }
        }
    }
}
{% endhighlight %}

If we want to take a look at the return data from the api, we can also make the request from a tool like __POSTMAN__. All that's required is a valid Facebook access token, which can be generated at __Facebook Graph API Explorer__.
<img src="{{ '/assets/img/requesttoken.png' | prepend: site.baseurl }}" alt="requesttoken">
As you can see we get a token of type __Bearer__ in response when we request the __/Token__ endpoint. We now have a complete setup to enable OAuth bearer tokens.
In the next and final part3 of this series we'll see how we can use the token to request data from our API.


Source code is available on [GitHub](https://github.com/rasmuschristensen/SimpleOAuth)

 
 