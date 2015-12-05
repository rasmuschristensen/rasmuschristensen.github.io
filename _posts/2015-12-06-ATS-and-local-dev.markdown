---
layout: post
title:  "ATS and local development with Xamarin"
date:   2015-12-06
---

<p class="intro">
<span class="dropcap">A</span>fter working with Xamarin for a while, one of the things I spent a lot of time on, was how to cosume an API running on Windows and hosted in VmWare Fusion from
a mobile Xamarin client running in either simulator or on a device.
</p>

As a .NET developer, my primary development IDE has been Visual Studio for many years. When I started doing Xamarin development, I went with 
Xamarin Studio, primarily because of the licensesing cost, but also to challenge myself with OSX and Xamarin Studio.
Based on this setup I'm hosting windows in a virtual machine using Vmware Fusion, with Visual Studio. And most of the time this will be where my mobile backend will be running during development.

## Consuming a windows hosted API from OSX
The basic key to consume a window hosted API from OSX is the network setup. With VmWare Fusion, It's pretty easy, using a <a href="https://en.wikipedia.org/wiki/Bridging_(networking)"> network bridge</a>.
From VmWare select __Virtual Machine__ -> __Settings__ -> __Network Adapter__
<img src="{{ '/assets/img/bridgenetwork.png' | prepend: site.baseurl }}" alt="Bridge network"> 
Inside this menu select __Bridged Networking__ and the __Wi-fi__ option. With this enabled it's now possible to consume a VmWare Fusion hosted Windows WebAPI from Xamarin running the client 	on OSX.

## Http for local dev...the ATS issue on IOS
While it's pretty easy to make the network bridge between OSX and Windows, the next bump on the road during development is <a href="https://developer.apple.com/library/prerelease/ios/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html#//apple_ref/doc/uid/TP40009251-SW33">Application Transport Security</a>, (ATS - NSAppTransportSecurity).
Since IOS 9.0, Apple by default requre all communication to be secure, meaning https. As all projects starts small, this would be a bit of pain, and luckily there is an option to avoid this while developing on your local machine.

If you run a vanilla Xamarin Forms project and try to consume a Windows API as described above, you'll get this message,
but only if you taget IOS 9.0 or above.

<img src="{{ '/assets/img/iosATS.png' | prepend: site.baseurl }}" alt="IOS ATS"> 

## Optional ATS using info.plist
To workaround this issue during local development (in production you should always use secure communication) you can add a new entry to the info.plist
in the Xmarin.ios project in your solution.
Locate the file and select the __source__ tab in the bottom of the file. Add a new entry to the file, either directly in Xamarin Studio, or in another text editor, this is just a plain xml file.
Add the entry __NSAppTransportSecurity__
{% highlight xml %}
<key>NSAppTransportSecurity</key>
	<dict>
		<key>NSAllowsArbitraryLoads</key>
		<true/>
	</dict>
{% endhighlight %}

Afterwards your infp.plist will should look like this
<img src="{{ '/assets/img/infoplistchange.png' | prepend: site.baseurl }}" alt="info plist change"> 

This will enable you to switch ATS on/off. Now turn it off and run the same code as before. This time you'll be able to consume the API without being blocked by 
the lack of insecure communication.


