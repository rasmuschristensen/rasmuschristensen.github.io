---
layout: post
title:  "Consume an IISExpress hosted WebAPI from OSX"
date:   2015-12-06
---

<p class="intro">
<span class="dropcap">D</span>eveloping a mobile application with Xamarin on OSX consuming a WebAPI hosted on windows, can be a bit complicated if you want to run everything local and keep things as simple
as possible. The goal of this post is a step by step to consume an IISExpress hosted WebAPI running in a VMWare Fusion virtual machine with Windows 10 from OSX. Most of the steps should be similar if you are running
 Parallels instead of VMWare Fusion. 
</p>

<p>When running a basic WebAPI locally, the default setup in Visual Studio is to use IIS Express, you can use Local IIS but my goal is to use IIS Express to keep things as simple as possible.</p>

## Creating the API in Windows
Creating the WebAPI is out of scope for this post, but basically follows the description [here](http://www.asp.net/web-api/overview/getting-started-with-aspnet-web-api/tutorial-your-first-web-api). 
Whether you want to use OWIN is not important, but I would recommend it. Ensure you have a simple controller __ValuesController__ with no authentication applied and a single action to consume a value.
For the purpose of this post we just need the action to return a simple value. 

{% highlight csharp %}
public class ValuesController : ApiController
{
    [HttpGet]
    public int Version()
    {
        return 1;
    }
}
{% endhighlight %}

Press F5 to ensure the API can run and try to consume the value using POSTMAN chromeextension e.g.
We are still on the windows host and should get the following result based on the request.
<img src="{{ '/assets/img/consume_api.png' | prepend: site.baseurl }}" alt="consumeapi">

## Bridge mode network
The first step to allow access is to ensure the virtual machine is running with network in bridge mode. For VMWare Fusion I've written about it here
[ATS and local dev](http://rasmustc.com/blog/ATS-and-local-dev/)

## Expose the URL of the API
To allow an incoming request to the host to be directed to IISExpress, execute the following command __ipconfig__ from powershell running as administrator to get your current ip
<img src="{{ '/assets/img/getipv4.png' | prepend: site.baseurl }}" alt="get ip">
With the IP in hand, execute the following command
{% highlight powershell %}
 netsh http add urlacl url=http://192.168.1.234:1586/ user=everyone
{% endhighlight %}
A small note to executing the command specified above. The __user__ part must be localized according to your current windows settings or else you'll get an error. You could also use your computername instead of ip.

## Add a binding to IISExpress
To ensure our API is resolved when we make a request and the _API is running_, we need to add an additional binding to the API projects applicationhost.config file. This file is located at:
__PathToYourProject\\.vs\config\applicationhost.config__. Edit the file and locate the __\<Bindings\>__ section under your projectname. My config looks like the configuration below where I've added both a specific binding to my IP on port 1586
and a binding to the computername __windows__ on port 8080. Remember to specify different port numbers or else the API will fail during startup.

{% highlight xml %}
<site name="ExposeAPI" id="2">
    <application path="/" applicationPool="Clr4IntegratedAppPool">
        <virtualDirectory path="/" physicalPath="C:\dev\exposeapi" />
    </application>
    <bindings>
    <binding protocol="http" bindingInformation="*:1585:localhost" />
    <binding protocol="http" bindingInformation="*:1586:192.168.1.234" />
    <binding protocol="http" bindingInformation="*:8080:windows" />
    </bindings>
</site>
{% endhighlight %}

## Firewall private access rule
The last thing to configure is the firewall. Be sure to only configure a rule for your private network. You can either use the graphical firewall client on windows and specify the path to IISExpress.
Or use this command from Powershell. Note the portnumber at the end os the command.
{% highlight powershell %}
netsh advfirewall firewall add rule name="IISExpressWeb" dir=in action=allow protocol=TCP localport=8080
{% endhighlight %}

## Making a request from OSX
Finally it's time to test the API from OSX. As with windows I'm using POSTMAN to test the API. Testing the command this way would result in the same as consuming the API from a mobile client using httpclient
<img src="{{ '/assets/img/osx2win.png' | prepend: site.baseurl }}" alt="osx2win">