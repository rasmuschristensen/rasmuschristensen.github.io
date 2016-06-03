---
layout: post
title:  "Orientation in Xamarin Forms"
description: "How to handle Orientation in Xamarin Forms"
date:   2016-06-02
---

<p class="intro">
<span class="dropcap">W</span>hen my device is rotated I want the screen to rotate, but for this screen I only want to allow portrait mode? Sounds pretty simple and so it should be. When it comes to Xamarin Forms, you might run into a wall, let's break it. 
</p>

Before writing this post I searched for a solution and found a lot of issues around Xamarin Forms and orientation. In the future this might be fixed but for now this is the solution I found to solve the problem.

### Device orientation IOS
In one of my projects, I ran into the situation described above. Some views should be allowed to rotate while others shouldn't. Targeting IOS, the way you allow rotation is to define it inside __info.plist__. Here you can define either specific allowed orientation or all, all pretty simple. 
<img src="{{ '/assets/img/orientation/infoplist.png' | prepend: site.baseurl }}"   alt="infoplist">
But when it comes to Xamarin Forms, you can't really define within a specific view, that you want it only in a specific orientation. Imagine you had a view with a signature control and you want __Landscape__ orientation to be enabled, but for the rest of the application only __Portrait__.


### The basic of orientation on IOS
When you have the allowed orientations defined inside __info.plist__, you can set the supported orientations of the current view by overriding __GetSupportedInterfaceOrientations__ in __AppDelegate.cs__. This method returns an __UIInterfaceOrientationMask__ where the allowed orientation(s) for a specific view can be overridden, exactly what we want to achieve. 

### View orientation responsibility 
Before we dive deeper into the implementation of __GetSupportedInterfaceOrientations__, we first need a way to let each view communicate its supported orientations. To do this we introduce the __ISupportOrientation__ interface.
{% highlight csharp %}
public interface ISupportOrientation
{
	DeviceOrientation[] SupportedOrientation { get; }
}

public enum DeviceOrientation
{
	Undefined,
	Landscape,
	Portrait
}
{% endhighlight %}
A pretty simple interface, in my case I added it to the PCL, as that's the common setup of my projects, but it should work fine with __Shared__ projects too. Together with the interface, we also add an enum __DeviceOrientation__. Feel free to add additional orientations as you need them. 
An implementation of a view that implements __ISupportOrientation__ might look like this.
{% highlight csharp %}
public partial class SignatureView : ContentPage, ISupportOrientation
{
 public SignatureView()
  {
   InitializeComponent();
  }

 public DeviceOrientation[] SupportedOrientation
 {
  get
  {
   return new[] { DeviceOrientation.Landscape, DeviceOrientation.Portrait };
  }
 }
}
{% endhighlight %}

### ISupportOrientation in action
Now that our view is ready, let's return to __AppDelegate.cs__ and __GetSupportedInterfaceOrientations__. Depending on your application, the orientation rules might change, but in this example I defined __Portrait__ to be the default orientation.
{% highlight csharp %}
public override UIInterfaceOrientationMask GetSupportedInterfaceOrientations(UIApplication application, UIWindow forWindow)
{
 if (Xamarin.Forms.Application.Current == null || Xamarin.Forms.Application.Current.MainPage == null)
 {
  return UIInterfaceOrientationMask.Portrait;
 }

 var navigationPage = Xamarin.Forms.Application.Current.MainPage as NavigationPage;

 if(navigationPage != null)
 {
  var orientationPage = navigationPage.CurrentPage as ISupportOrientation;

  if (orientationPage != null)
  {
   UIInterfaceOrientationMask supportedMask = new UIInterfaceOrientationMask();

   foreach (var orientation in orientationPage.SupportedOrientation)
   {
    switch (orientation)
    {
     case DeviceOrientations.Portrait:
      supportedMask |= UIInterfaceOrientationMask.Portrait;
      break;
  
     case DeviceOrientations.Landscape:
      supportedMask |= UIInterfaceOrientationMask.Landscape;
	  break;

     default:
      break;
    }
   }
   return supportedMask;
  }
 }
 return UIInterfaceOrientationMask.Portrait;
}
{% endhighlight %}
As you can see this is a pretty simple implementation and not optimized in any special way. We check the current view for the __ISupportOrientation__ and if multiple orientations is supported they are combined in the __UIInterfaceOrientationMask__.

The implementation is available as a Gist [here](https://gist.github.com/rasmuschristensen/5cfee152366eb19adac599a606b857e6/)  












