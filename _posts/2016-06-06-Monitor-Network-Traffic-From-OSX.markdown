---
layout: post
title:  "Monitor Mobile Traffic On OSX"
description:  "How to monitor your mobile device network traffic with Charles on OSX"
date:   2016-06-06
---

<p class="intro">
<span class="dropcap">D</span>eveloping a mobile app on OSX with Xamarin or any other tooling, you might run into the situation when you want to see what's being transfered over the wire. As a long time Windows developer I've been use to Fiddler, but what do you do when you're on OSX? 
</p>

### Charles, the simple tool
Maybe you have a situation where you want to see what's beeing sent across the wire in an http request, like a header. Maybe as in my case you add an authentication token or an IF-MODIFIED-SINCE header and want to check the timestamp in the request. In either case, we need a tool to monitor the traffic. Further more we also need a tool that can act as a proxy to the mobile device. Just as your devlopment machine, the device itself has its own separate network connection. To monitor the mobile device' traffic on the development machine, we need to route the traffic to it, we need a proxy.

After some searching I found a tool called [Charles](https://www.charlesproxy.com/). 
_"Charles is an HTTP proxy / HTTP monitor / Reverse Proxy that enables a developer to view all of the HTTP and SSL / HTTPS traffic between their machine and the Internet. This includes requests, responses and the HTTP headers (which contain the cookies and caching information)"_

### Charles setup
Charles is a paid product, not expensieve ~50$, and there is a trial you can use for some days and 30 min at a time.
To use Charles as a proxy, you first need to locate your own ip. Either goto __System preferences -> Network__ and locate the IP
<img src="{{ '/assets/img/webproxy/systemnetwork.png' | prepend: site.baseurl }}"   alt="find ip">
Or you can use the command __ifconfig__ from a terminal. 
<img src="{{ '/assets/img/webproxy/ifconfig.png' | prepend: site.baseurl }}"   alt="find ip">

### Configure your mobile device to use the proxy.
When charles is running, we need to route the mobile device to the proxy. On iOS go to __Settings -> WiFi__ Select your current network connection and tap the __"i"__ information icon to the right of the network. This brigs you to some details of the network. At the bottom, locate the Http-Proxy and select __Manual__. Type in the IP we located before and port 8888, by default Charles is setup to act as a proxy on port 8888. Disable authentication. Now all network traffice from the mobiel device will be routed through the proxy.
<img src="{{ '/assets/img/webproxy/ios.png' | prepend: site.baseurl }}"   alt="setup ios">

### Ready to monitor
When you make some interaction with an app, you can now follow the communication in __Charles__. You might see a lot of "noise", as other apps also use the network and it can be hard to follow just your requests to a specific endpoint. Charles has a simple feature called __Focus__. You simple select __Structure__ at the main layout and right click one of your requests and to __Focus__ from the menu. Now your requests will be grouped at the top of the treeview and all other traffic will be moved into a node called __Other hosts__. Simple and to the point. As with other similar monitoring tools, you can now see and browse all request/response your app is making.
<img src="{{ '/assets/img/webproxy/charles.png' | prepend: site.baseurl }}"   alt="charles">

### Monitor data from your app
As I mentioned in the beginning, Authentication and IF-MODIFIED-SINCE are two of the values I used Charles to investigate, so let's see how this can be done i real life with Xamarin Forms.
Let's say we have a car API and want to fetch all cars, the code could be something like this.
{% highlight csharp %}
public void GetAllCars(string baseUri)
{
	var httpClient = new HttpClient(new ModernHttpClient.NativeMessageHandler());

	httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", Settings.AccessToken);
	if (httpClient.DefaultRequestHeaders.CacheControl == null)
					httpClient.DefaultRequestHeaders.CacheControl = new CacheControlHeaderValue();

	var lastModifiedDate = Settings.GetLastModified(ifModifiedSinceKey);

	httpClient.DefaultRequestHeaders.CacheControl.NoCache = true;
	httpClient.DefaultRequestHeaders.IfModifiedSince = lastModifiedDate;
	httpClient.DefaultRequestHeaders.CacheControl.NoStore = true;

	var httpResponse = await httpClient.GetAsync(new Uri(baseUri, "api/Cars/All" ));
	......
}
{% endhighlight %}
First we create and instance of the HttpClient using ModernHttpClient. Next up we add an __AuthenticationHeader__ setting a value we stored in Settings. Finally we set the __IfModifiedSince__ with a value stored in settings. Note I'm using this awesome crossplatform [plugin](https://github.com/jamesmontemagno/Xamarin.Plugins/tree/master/Settings). The CacheControl will not be of focus for this post. Now let's see how the data is visible in Charles.
First we can take a look at the __AuthenticationHeader__. As you can see it's pretty easy to read the data now.
<img src="{{ '/assets/img/webproxy/authheader.png' | prepend: site.baseurl }}"   alt="authheader">
Next we have the IF-MODIFIED-SINCE. As in my case I had a timezone issue and needed to compare the value from my code, to the actual value being transfered, and I didn't have access to debug the API. 
<img src="{{ '/assets/img/webproxy/ifmodifiedsince.png' | prepend: site.baseurl }}"   alt="idmodifiedsince">
You may notice I change the view in charles, and we have some more details here, but as you can see once again, we get the data we're looking for very easy.
