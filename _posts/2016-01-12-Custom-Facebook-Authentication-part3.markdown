---
layout: post
title:  "Facebook Auth with Xamarin Forms and WebAPI CustomGrant - part 3"
date:   2016-01-12
---

<p class="intro">
<span class="dropcap">B</span>ased on the two previous posts, we have a complete setup by now, with a mobile client and an secured API. Now we'll see how we can consume the API with the access tokens we generated in the previous post and finally display the 
data in the mobile client.
</p>

You can find part1 _Building the mobile client_ [here](http://rasmustc.com/blog/Custom-Facebook-Authentication-with-webapi/) and part2 _WebAPI CustomGrant setup_ [here](http://rasmustc.com/blog/Custom-Facebook-Authentication-part2/). 


### Extend the API 
Before we can consume any data from the API, we need to have a controller with an action. We'll add a __GadgetsController__. In this sample we'll add a single HTTP GET endpoint to get a list of gadgets. 
{% highlight csharp %}
[Authorize]
public class GadgetsController : ApiController
{
    public List<string> Get()
    {
        return new List<string> {"MBP", "IPad mini", "Nexus 5x", "Tesla Model X"};
    }
}
{% endhighlight %}

The important part to notice, is the __[Authorize]__ attribute on top of the class. This ensures the controller and all of it's actions can only be consumed by authenticated clients. It is possible to deviate from this 
on specific actions.
With this controller in place we're ready to test the API. Again we do this by consuming the API using POSTMAN, and we do it unauthenticated to ensure authentication works as expected __<font color='red'>RED</font>/<font color='green'>GREEN</font>__ testing.
As  expected we get a HTTP 401 Unauthorized because we didn't provide any access token.
<img src="{{ '/assets/img/postmanUnAuth.png' | prepend: site.baseurl }}" alt="postman unauth">

### Extend the mobile client
In our mobile client we ended [part2](http://rasmustc.com/blog/Custom-Facebook-Authentication-part2/) with an access token to consume our API. Now we'll extend the client with a simple view where we'll consume the gadgets with
the api access token.

We start by adding a view to the client, one we can navigate to upon successful login. To persist the access token on the client, we'll use the __Settings__ [plugin](https://github.com/jamesmontemagno/Xamarin.Plugins/tree/master/Settings) by 
[James Montemagno](https://twitter.com/JamesMontemagno). Add the package __Xam.Plugins.Settings__ to the solution. The package must be added both to the PCL and Platform specific project. But the Settings implementation will only be in the PCL.
<img src="{{ '/assets/img/settingsplugin.png' | prepend: site.baseurl }}" alt="settings plugin">

 With this package installed we'll create a simple setting __ApiAccesstoken__ to the pregenerated Settings class, also added by the plugin during package installation. 
{% highlight csharp %}
private const string ApiAccessTokenKey = "apiAccessToken_key";
private static readonly string ApiAccessTokenDefault = string.Empty;

public static string ApiAccessToken {
    get {
        return AppSettings.GetValueOrDefault<string> (ApiAccessTokenKey, ApiAccessTokenDefault);
    }
    set {
        AppSettings.AddOrUpdateValue<string> (ApiAccessTokenKey, value);
    }
}
{% endhighlight %}
With this in place, we can store the API access token like this when we authenticate. After login the app navigate to a __GadgetsView__
{% highlight csharp %}
if (authenticationTicket != null)
{
    var apiAccessToken = authenticationTicket.Access_Token;
    Settings.ApiAccessToken = apiAccessToken;
}
{% endhighlight %}

### Consuming and listing API data
To consume the APIs Gadgets endpoint we will add a __GadgetsViewmodel__ as the __GadgetsView__ will later bind to and display the data we consume in the viewmodel.
The viewmodel will have a method __LoadGadgets__ where we securely load the gadgets. To improve performance we use the nuget package [ModenHttpClient](https://www.nuget.org/packages/modernhttpclient/)
{% highlight csharp %}
public async Task<List<Gadget>> LoadGadgets ()
{			
    using (var handler = new NativeMessageHandler ()) {
        using (var client = new HttpClient (handler)) {

            var requestMessage = new HttpRequestMessage {
                Method = HttpMethod.Get,
                RequestUri = new Uri ("http://windows:8080/api/gadgets")						
            };

            requestMessage.Headers.Add ("Accept", "application/json");
            requestMessage.Headers.Authorization = new AuthenticationHeaderValue ("Bearer", Settings.ApiAccessToken);

            var response = await client.SendAsync (requestMessage);
            var content = await response.Content.ReadAsStringAsync ();

            return JsonConvert.DeserializeObject<List<Gadget>> (content);
        }
    }
}
{% endhighlight %}

We build a __HttpRequestMessage__ of type __GET__ and and specify the api endpoint we want to consume. To the Header of the request we add __application/json__ to ensure the response is formatted as json and finally we add
an __authorization header__ where we specify our API access token, this is the _key_ to access our protected resource.

The rest of the implementation to bind the view and the viewmodel can be found in the source code at [GitHub](https://github.com/rasmuschristensen/SimpleOAuth).
When the app is started we are first asked to authenticate with facebook and after successfull login, redirected to the main screen implemented by the GadgetsView, where all the gadgets are loaded and displayed.
<img src="{{ '/assets/img/fblogin.png' | prepend: site.baseurl }}" alt="postman unauth"><img src="{{ '/assets/img/gadgetsview.png' | prepend: site.baseurl }}" alt="postman unauth"> 

Source code is available on [GitHub](https://github.com/rasmuschristensen/SimpleOAuth)

 
 