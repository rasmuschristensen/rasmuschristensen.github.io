---
layout: post
title:  "Infinite ListView scrolling with behaviors"
date:   2016-03-04
---

<p class="intro">
<span class="dropcap">S</span>crolling a ListView with behaviors makes your code clean and reuseable 
</p>

By default Xamarin Forms support scrolling and making infinite scrolling by using events. If you like me are developing with the MVVM mindset, 
events can become a bit messy when everything else is handled by comamnds. Let's take advantage of __Behaviors__ and see how we can turn events into a command driven code style.


### The beer order app
We'll create an app to list beer orders. All orders will have a created date, and this is the key to our listing. We'll make the app display
the orders in decending order by the created date with the newest on top. As a best practics data should always be loaded in batches to reduce newwork traffic and load. The app will load 20 items per page, meaning when we scroll to the bottom, we will load the next 20 orders by the created date. 
The app will load data from an inapp store, but can easily be replaced with your own external storage.

The source code is available [here](https://github.com/rasmuschristensen/SimpleListViewInfiniteScrolling)

The final app will work like the video below, in a mix of refresh and scrolling. The page title indicates the number of items currently loaded. 
<img src="{{ '/assets/img/infinitebrew.gif' | prepend: site.baseurl }}" alt="appvideo">


### From event to behavior
The ListView exposes the event __ItemAppearing__ each time an item in the ListView becomes visible. What we need to do is check whether the item is the last visible item in the ListView. In our app we have no unique identifier to page from, instead we want to page by the date of the orders. 

First we create a class __ListViewPagningBehavior__ which inherits the generic Behavior with the type of our ListView as the type, __Behavior<ListView>__. We will override the methods __OnAttachedTo__ and __OnDetachingFrom__ as the entry point to subscribe and unsubscribe the events we'll listen to. 

{% highlight csharp %}
protected override void OnAttachedTo (ListView bindable)
{			
    base.OnAttachedTo (bindable);

    AssociatedObject = bindable;
    bindable.BindingContextChanged += OnBindingContextChanged;
    bindable.ItemAppearing += OnItemAppearing;
}

protected override void OnDetachingFrom (ListView bindable)
{
    base.OnDetachingFrom (bindable);

    bindable.BindingContextChanged -= OnBindingContextChanged;
    bindable.ItemAppearing -= OnItemAppearing;
    AssociatedObject = null;
}
{% endhighlight %}

Besides the __OnItemAppearing__ event, we also listen to the change of binding context.

### Time to do paging
Each time a new item appears in the ListView, the following method will be executed.
{% highlight csharp %}
private void OnItemAppearing (object sender, ItemVisibilityEventArgs e)
{		
    // sanity checks hidden - see the github repository

    object parameter = Converter.Convert (e, typeof(object), null, null);
    if (Command.CanExecute (parameter)) {
        Command.Execute (parameter);
    }       
}
{% endhighlight%}

Note the __Command__ introduced here. It's defined and exposed by our behavior. This way when we can use the behavior from a XAML view and bind to the __ICommand__, we want to handle the specific action. The comamnd is exposed from the behavior this way.
{% highlight csharp %}
public static readonly BindableProperty CommandProperty = BindableProperty.Create ("Command", typeof(ICommand), typeof(ListViewPagningBehavior), null);

public ICommand Command {
    get { return (ICommand)GetValue (CommandProperty); }
    set { SetValue (CommandProperty, value); }
}
{% endhighlight %} 

### Type conversion
As the command is type specific and our behavior is not, we need to add a __value converter__ to handle the conversion. It's exposed from the behavior the same way as the command.
{% highlight csharp %}
public static readonly BindableProperty InputConverterProperty = BindableProperty.Create ("Converter", typeof(IValueConverter), typeof(ListViewPagningBehavior), null);
{% endhighlight %} 

The converter itself is pretty straight forward and for this purpose we don't need to implement __ConvertBack__.
{% highlight csharp %}
public class ItemVisibilityEventArgstemConverter : IValueConverter
{
    public object Convert (object value, Type targetType, object parameter, CultureInfo culture)
    {
        var eventArgs = value as ItemVisibilityEventArgs;
        return eventArgs.Item;
    }
}
{% endhighlight %}

### XAML glue
With the behavior in place, it's time to wire it all up in XAML. 
I've defined some local namespaces to seperate the components, and added the converter as a StaticResource.

<img src="{{ '/assets/img/xamlbehavior.png' | prepend: site.baseurl }}" alt="xamlbehavior">

The source code is available [here](https://github.com/rasmuschristensen/SimpleListViewInfiniteScrolling)

<img src="{{ '/assets/img/infinitebrew.gif' | prepend: site.baseurl }}" alt="appvideo">