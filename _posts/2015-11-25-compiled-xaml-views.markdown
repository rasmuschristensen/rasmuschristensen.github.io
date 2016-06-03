---
layout: post
title:  "Compile XAML with Xamarin"
description: "How to enable compilation of XAML in Xamarin"
date:   2015-11-25
---

<p class="intro">
<span class="dropcap">O</span>ne of the new features in Xamarin 4 and Xamarin Forms 2.0 is the ability to compile Xaml views.
 This feature should reduce view load time, the compiled app size, the catch might be an increase in compilation time, so let's see how
 this works.
</p>

## Xaml runtime error
First we created a basic Xamarin cross platform project based on Xamarin Forms. Ensure to update the Xamarin.Forms nuget packages to version 2.x.
Add a Xaml view to the project with a simple Button control and two properties Id and Text (...Yes the Id is not valid, let's verify it.)
<img src="{{ '/assets/img/InvalidXaml.png' | prepend: site.baseurl }}" alt="xamarin invalid attr">
Before Xamarin Forms 2.0 this would compile just fine, and if we didn't do anything to the current project in Xamarin 2.0, compilation will also complete without any errors.
If we start the app either in the simulator or on a device, we would get a runtime error.
<img src="{{ '/assets/img/xamlruntimeerror.png' | prepend: site.baseurl }}" alt="xamarin runtime error">   
As expected the "Id" property is invalid. 
As this is a very simple example, this could however be something deep inside our app and not that easy to locate.
##Enabling Xaml Compilation
The new feature to enable the XamlC, Xaml Compiler - can be enabled at either assembly level or class level or in a mix of both. It's located in the namespace
__Xamarin.Forms.Xaml__. 
Locate the App class and above the namespace definition, add the following codesnippet.
´´´ 

using Xamarin.Forms.Xaml
[assembly: XamlCompilation (XamlCompilationOptions.Compile)]
´´´

Build the project again and now the code won't compile, with the following error, but luckily this time, we caught it during compilation.
<img src="{{ '/assets/img/compileerror.png' | prepend: site.baseurl }}" alt="xamarin compile error">
It's also possible to add Xaml compilation to a specific view, simply by added the same attribute as before without __assembly:__ to at the class level of a view.
The result is the same as before, except this time Xaml compilation is only enabled for a specific view.
<img src="{{ '/assets/img/xamlcompilationclass.png' | prepend: site.baseurl }}" alt="xamarin compile error">

Finally it's possible to either add and mix multiple class level Xaml compilations or if you enabled it at assembly level, you can exclude it for specific viewsby selecting
__.Skip__ instead of __.Compile__ at class level
 <img src="{{ '/assets/img/xamlcompileskip.png' | prepend: site.baseurl }}" alt="xamarin compile skip">


