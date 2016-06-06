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

