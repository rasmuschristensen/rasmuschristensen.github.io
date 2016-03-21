---
layout: post
title:  "Image gallery with Xamarin Forms"
date:   2016-03-21
---

<p class="intro">
<span class="dropcap">O</span>ut of the box Xamarin Forms provides many standard cross platform controls. Recently I had the need to display images in an image gallery and among the many controls Xamarin Forms provides, they aren't exactly image galleries. So my solution was to build a cross platform image gallery. 
</p>


### The concept of an image gallery
We'll create a reuseable control, capable of displaying images from an observablecollection. The control will support horizontal scrolling, and when a new photo is captured with the camera, it's added to the gallery immediatly, using databinding. Further more we'll take advantage of tap gestures and display a big version of an image, when it's selected. 

The source code is available [here](https://github.com/rasmuschristensen/XamarinFormsImageGallery)
For a functional demo, watch the video at the bottom of this post or checkout the GitHub Repository.


<img src="{{ '/assets/img/imagegalleryLayout.png' | prepend: site.baseurl }}" alt="imageGalleryLayout">

### Building an image gallery
The core of the image gallery is a standard __StackLayout__ with images inside. The gallery itself will expose an __ItemsSource__ property where we can databind an observable collection. Each time a photo is captured, the gallery will append the new photo. If we tap an image, it will be displayed in the preview area. Further more when the gallery becomes wider than the screen, horizontal scroll will be added. Time to checkout the code.

{% highlight csharp%}
public class ImageGallery : ScrollView
{
    readonly StackLayout _imageStack;

    public ImageGallery ()
    {
        this.Orientation = ScrollOrientation.Horizontal;

        _imageStack = new StackLayout {
            Orientation = StackOrientation.Horizontal
        };

        this.Content = _imageStack;
    }

    public IList<View> Children {
        get {
            return _imageStack.Children;
        }
    }
{% endhighlight%}
We start by creating a class which inherits from __ScrollView__ and set its orientation to __Horizontal__. Inside the ScrollView we make a __StackLayout__ to contain all our __Images__, even though we define it as __View__ in the code above. More on this later when we get to the __ItemTemplate__ support.

### ItemsSource support
We want to ensure that out image gallery supports databinding with MVVM and hence we want to expose a bindable property where the consumer of the control can attach a data bound collection. We do this by adding a BindableProperty __ItemsSource__.

{% highlight csharp %}
public static readonly BindableProperty ItemsSourceProperty =
BindableProperty.Create<ImageGallery, IList> (
        view => view.ItemsSource,
        default(IList), 
        BindingMode.TwoWay,
        propertyChanging: (bindableObject, oldValue, newValue) => {
            ((ImageGallery)bindableObject).ItemsSourceChanging ();
        },
        propertyChanged: (bindableObject, oldValue, newValue) => {
            ((ImageGallery)bindableObject).ItemsSourceChanged (bindableObject, oldValue, newValue);
        }
);

public IList ItemsSource {
    get {
        return (IList)GetValue (ItemsSourceProperty);
    }
    set {

        SetValue (ItemsSourceProperty, value);
    }
}
{% endhighlight %}
With this property we get access to __ItemsSourceChanged__ and __ItemsSourceChanging__. The latter is not in scope here. __ItemsSourceChanged__ provides access to values currently in the source and also ___added___ or ___removed___ items. In this post we only focus on items being added to the source, but it's fairly easy to extend with remove. 
{% highlight csharp%}
var notifyCollection = newValue as INotifyCollectionChanged;
if (notifyCollection != null) {
    notifyCollection.CollectionChanged += (sender, args) => {
        if (args.NewItems != null) {
            foreach (var newItem in args.NewItems) {

                var view = (View)ItemTemplate.CreateContent ();
                var bindableObject = view as BindableObject;
                if (bindableObject != null)
                    bindableObject.BindingContext = newItem;
                _imageStack.Children.Add (view);
            }
        }
    };
}
{% endhighlight %}
We hook into  __CollectionChanged__ of our ItemsSource. For each new item added, we create an instance of the current __ItemTemplate__ defined in XAML. The __View__ we get from here, will be treated as a __BindableObject__ itself, and from this point of view ___"untyped"___.  This means we can add almost any custom object to our collection and make the current defined __ItemTemplate__ responsible for rendering.

As you can see from the source code, we can make usage of __SelectedIndex__ but I've decided to make use of __GestureRecognizers__ defined in the __ItemTemplate__ when selecting an image to preview.

### ItemTemplate for render and gesture support
The __ItemTemplate__ we define when using our image gallery control is ___just a standard___ defined item template as you would also create if you use ListView etc. 
{% highlight csharp %}
<custom:ImageGallery.ItemTemplate>
    <DataTemplate>
        <Image
            Source="{Binding Source}"
            Aspect="AspectFit">
            <Image.GestureRecognizers>
                <TapGestureRecognizer
                    Command="{Binding Path=BindingContext.PreviewImageCommand, Source={x:Reference ThePage}}"
                    CommandParameter="{Binding ImageId}" />
            </Image.GestureRecognizers>
        </Image>
    </DataTemplate>
</custom:ImageGallery.ItemTemplate>
{% endhighlight %}
First we add an Image and bind it to the __Source__ property of the items in the image gallery __ItemsSource__ property, here implemented as an ImageSource. Next we define a __TapGestureRecognizer__, this one is a bit more tricky. Because we only have a single view and a single viewmodel in this sample, we make a __Binding__ to a __Command__(___PreviewImageCommand___) on the MainViewModel, bound to the __MainView__ aka the current __BindingContext__. Along the command we've also defined a __CommandParameter__ which is a unique id of each image. This is used by the PreviewImageCommand, so select the image to preview.

With all this in place, all we need is a basic ViewModel with an ObservableCollection of images and a command to handle the preview gesture. You can find this in the [GitHub Repository](https://github.com/rasmuschristensen/XamarinFormsImageGallery).
 
The final app will work like the video below, for both IOS and Android. 
<img src="{{ '/assets/img/imagegallery.gif' | prepend: site.baseurl }}" alt="appvideo">&nbsp;
<img src="{{ '/assets/img/androidimagegallery.png' | prepend: site.baseurl }}" alt="appimage">






