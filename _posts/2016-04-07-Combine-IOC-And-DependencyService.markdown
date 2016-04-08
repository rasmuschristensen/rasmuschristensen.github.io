---
layout: post
title:  "Howto Combine IOC And Xamarin DependencyService"
date:   2016-04-07
---

<p class="intro">
<span class="dropcap">I</span>n Xamarin Forms we can use the built-in DependencyService, to use native features. In the terms of dependency injection this approach does work, but it's more of a service locator. If we at the same time use an IOC container like Autofac or just SimpleIoC, this DependencService becomes a bit messy as it by default breaks the constructor injection. Well luckily we can get the best of both by combining the two.  
</p>

### Xamarin Forms DependencyService
In a basic Xamarin Forms application we can use the [DependencyService](https://developer.xamarin.com/guides/xamarin-forms/dependency-service/) to resolve platform specific/native dependencies if we use a PCL approach. Let's start by looking how this is done, by resolving a service to resize images implemented in iOS and Android.
{% highlight csharp %}
/*** PCL project ***/
reasonly _imageResizer;

public class ImageGalleryViewModel()
{
    public ImageGalleryViewModel()
    {
        _imageResizer = DependencyService.Get<IImageResizer> ()
    }
}
{% endhighlight %}
{% highlight csharp %}
/*** iOS project ***/
[assembly: Xamarin.Forms.Dependency (typeof(ImageResizer))]
namespace MyApp.iOS.Services
{
	public class ImageResizer : IImageResizer
	{
        public byte[] Resize (byte[] imageData, float width, float height)
		{
            //....impl omitted
        }
    }
}
{% endhighlight %}
{% highlight csharp %}
/*** Android project ***/
[assembly: Xamarin.Forms.Dependency (typeof(ImageResizer))]
namespace MyApp.Droid.Services
{
	public class ImageResizer : IImageResizer
	{
        public byte[] Resize (byte[] imageData, float width, float height)
		{
            //....impl omitted
        }
    }
}
{% endhighlight %}
The important step to note here, is how we get the dependency into the ViewModel. In many MVVM implementations a very common approach is contructor injection. Tthis makes it easy to inject other implementations during testing and most of all it's very clear to the consumer of the ViewModel, if it requires any dependencies. The use of the DependencyService in the code above, makes it more of a blackbox.

### Open the box and inject...
As Xamarin Forms DependencyService ___is a great service___, a more common usage in MVVM implementations, is to use an IoC Container like [Autofac](http://autofac.org/) or SimpleIOC that's baked into [MVVMLight](http://www.mvvmlight.net/).
These containes makes it possible to do constructor injection, by register all needed dependencies upfront. As we have a ViewModel, we can assume it's used together with a view as a BindingContext, so the first registration is to register the ViewModel in the container itself. To see how all this can be autowired, read my previous [post].(http://rasmustc.com/blog/Index-ViewModelLocator-With-MvvmLight-And-Xamarin-Forms/)

First we register the ViewModel as it's a dependency to a view.
{% highlight csharp %}
public class ViewModelLocator : DynamicObject
{    
    public ViewModelLocator ()
    {
        ServiceLocator.SetLocatorProvider (() => SimpleIoc.Default); 
        SimpleIoc.Default.Register<ImageGalleryViewModel> ();
    }
}
{% endhighlight %}
Next we change the signature of the __ImageGalleryViewModel__ to require __IImageResizer__ as a dependency.
{% highlight csharp %}
public ImageGalleryViewModel (IImageResizer imageResizer)
{% endhighlight %} 

As the IoC container is responsible of resolving dependencies, we need to ensure our viewmodel can be resolved with the new dependency from the container. Well it's really simple.
{% highlight csharp %}
public class ViewModelLocator : DynamicObject
{    
    public ViewModelLocator ()
    {
        ServiceLocator.SetLocatorProvider (() => SimpleIoc.Default);
        var di = SimpleIoc.Default;
        di.Register<IImageResizer> (() => DependencyService.Get<IImageResizer> ()); 
        di.Register<ImageGalleryViewModel> ();
    }
}
{% endhighlight %}
All we need to do is move the __DependencyService__ into the container registration. As the code runs on a specific platform, iOS or Android, our ViewModel is resolved. The specific platform implementation of __IImageResizer__ will then be resolved as the dependency required by our ViewModel, and now we have a more clean implementation.








