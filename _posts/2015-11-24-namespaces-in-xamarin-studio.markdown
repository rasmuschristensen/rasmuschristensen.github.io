---
layout: post
title:  "Namespaces in Xamarin Studio"
description: "A simple guide to structure code in a namespace style"
date:   2015-11-24
---

<p class="intro">
<span class="dropcap">I</span>f you like me have a background using Visual Studio, one of the first things you'll notice after a little
coding in Xamarin Studio is the default namespace handling. In Visual Studio namespaces per default follows the folder structure hierarchical.
In Xamarin Studio is's just the opposite, a flat namespace stucture.
</p>

To get the same behaviour in Xamarin Studio as in Visual Studio, goto 
__Preferences -> Source Code -> .NET Naming Policies__. and select __Hierarchical__


<img src="{{ '/assets/img/xamarinnamespaces.png' | prepend: site.baseurl }}" alt="xamarin studio"> 

Please note this is best done on a new project. If not, you have to manually ensure all namespaces are set correctly.