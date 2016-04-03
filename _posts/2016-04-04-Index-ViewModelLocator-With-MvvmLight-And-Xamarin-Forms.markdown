---
layout: post
title:  "Indexed ViewModelLocator With MvvmLight And Xamarin Forms"
date:   2016-04-04
---

<p class="intro">
<span class="dropcap">W</span>orking with MVVMLight toolkit and Xamarin Forms removes a lot of boiler plate coding. When it comes to the basic of resolving viewmodels this clean code gets a bit blurry, so let's see how we can improve this.  
</p>

### The default Viewmodel resolving
Building a solution with MVVMLight, the most common solution to resolve viewmodels is a mix of SimpleIOC and a ViewModelLocator. The ViewModelLocator provides basic binding for each ViewModel it exposes and Views consume the ViewModels by databinding.
{% highlight xml %}
<ContentPage
	xmlns="http://xamarin.com/schemas/2014/forms"
	xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
	x:Class="BasicMvvmLight.Views.CarListView"
	BindingContext="{Binding CarVM, Source={StaticResource ViewModelLocator}}">
	<ContentPage.Content>
		<Label
			Text="{Binding Brand}">
		</Label>
	</ContentPage.Content>
</ContentPage>
{% endhighlight %}

{% highlight csharp %}
public class ViewModelLocator
{
    public ViewModelLocator ()
    {
        ServiceLocator.SetLocatorProvider (() => SimpleIoc.Default);

        SimpleIoc.Default.Register<CarViewModel> ();		
    }
    
    [System.Diagnostics.CodeAnalysis.SuppressMessage ("Microsoft.Performance",
        "CA1822:MarkMembersAsStatic",
        Justification = "This non-static member is needed for data binding purposes.")]
    public CarViewModel CarVM {
        get { return ServiceLocator.Current.GetInstance<CarViewModel> (); }
}
{% endhighlight %}

The code above ensures that the BindingContext in the __view__ is set by the ___CarVM___ binding where CarVM is the name of the binding in the ViewModelLocator providing the CarViewModel. As the solution grows, the __ViewModelLocator__ becomes a class with a lot of basic getter's just to resolve ViewModels, some pretty trivial code to write an maintain. 

### Indexed databinding
If we start by taking a look at the XAML code in the view, we currently have the __Binding CarVM__ as a ___hard___ databinding to a specific property on the __ViewModelLocator__. Luckily we have the option to use an indexed binding, where we can specify a simple name as a string, we want to bind to and not a specific property. This means we can take advantage of a single index on the viewmodel to resolve viewmodels and less code to maintain! The flow is illustrated below.

<img src="{{ '/assets/img/viewmodellocatorflow.png' | prepend: site.baseurl }}" alt="viewmodellocatorflow">

### Refactor View
The first thing we need to do is change the binding in the view to make use of the __indexer__. As you can see I've also changed the name of the index to __CarViewModel__. this is because we want to use the binding index as type lookup in the ViewModelLocator later meaning the name should match a corresponding type.
{% highlight xml %}
<ContentPage
	xmlns="http://xamarin.com/schemas/2014/forms"
	xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
	x:Class="BasicMvvmLight.Views.CarListView"
	BindingContext="{Binding [CarViewModel], Source={StaticResource ViewModelLocator}}">
	<ContentPage.Content>
		<Label
			Text="{Binding Brand}">
		</Label>
	</ContentPage.Content>
</ContentPage>
{% endhighlight %} 
  
### Refactor ViewModelLocator 
The refactor of our ViewModelLocator introduces some new concepts. As specified by the view, we should provide an __indexer__. The indexer is added by letting the ViewModelLocator inherit from __DynamicObject__. This makes __TryGetMember__ available, which the binding in the view will target. Combined with this, we add an indexer which use the final piece of the puzzle, the __ViewModelResolver__. 

{% highlight csharp %}
public class ViewModelLocator : DynamicObject
{
    static ViewModelResolver _resolver;

    public ViewModelLocator ()
    {
        ServiceLocator.SetLocatorProvider (() => SimpleIoc.Default);

        SimpleIoc.Default.Register<CarViewModel> ();
    }

    public static ViewModelResolver Resolver {
        get {
            if (_resolver == null) {
                _resolver = new ViewModelResolver ();
            }
            return _resolver;
        }
    }

    public object this [string viewModelName] {
        get {               
            return Resolver.Resolve (viewModelName);
        }
    }

    public override bool TryGetMember (GetMemberBinder binder, out object result)
    {
        result = this [binder.Name];
        return true;
    }
}
{% endhighlight %} 

### Introducing ViewModelResolver
The responsibility of the __ViewModelResolver__ is to resolve an instance of a ViewModel, from a name specifying its type. One thing to keep in mind, the __ViewModelResolver__ is located inside a __PCL__. This is necessary to know when we resolve the type from the assembly.
We find all the types defined and make a match on the provided name. When located, the instance is resolved and all dependencies it might might require (constructor based) will also be resolved, when using the DI Container.
{% highlight csharp %}
public class ViewModelResolver
{		
    public ViewModelResolver ()
    {}       

    public object Resolve (string viewModelName)
    {
        var vmtype = this.GetType ()
            .GetTypeInfo ()
            .Assembly
            .DefinedTypes
            .FirstOrDefault (t => t.Name.Equals (viewModelName))
            .AsType ();

        return ServiceLocator.Current.GetInstance (vmtype);
    }
}
{% endhighlight %}


With the ViewModelResolver in place we can now run the project and if we extend the app with more view and viewmodels, we just need to ensure everything is registered in the DI container. If another DI container like Autofac is used instead, it can simply be injected into the ViewModelResolver.

The complete source code is available here [GitHub Repository](https://github.com/rasmuschristensen/XamarinFormsImageGallery).






