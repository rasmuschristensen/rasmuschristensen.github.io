---
layout: post
title:  "Code Templates In Xamarin Studio"
description:  "Code Templates In Xamarin Studio"
date:   2016-04-25
---

<p class="intro">
<span class="dropcap">W</span>riting the same code structure over and over seems a bit non productive. Luckily there is a simple way in Xamarin Studio to remove this repetative work and speed up things, __Code Templates__  
</p>

### Basic Code Templates
Out of the box Xamarin Studio offers some default Code Templates. If you type __prop__ followed by tapping __TAB__ twice in the editor, you get a default structure of a property. This will bring the template a live and you can the enter the __type__ press __TAB__ and the the name of the property. All very simple and useful.

<img src="{{ '/assets/img/codetemplate.gif' | prepend: site.baseurl }}" alt="appvideo">

### Writing your own custom Code Templates
You can use the built in  __Code Templates__, but often you use a third party library with a specific code style or you have some kind of code convention in your team. This is where you quite easy can create your own code templates.

One of the plugins I use very often is [MVVMHelpers](https://github.com/jamesmontemagno/mvvm-helpers)
The syntax for setting a property with this library is as follow where the property is implemented using a backing field.
{% highlight csharp %}
string _name 
public string Name 
{
    get{ return _name;}
    set{ SetProperty(ref _name, value);}
}
{% endhighlight %}
As this is not the default syntax for the default __prop__ template (part of Xamarin Studio by default) , we need to implement a new one.
Start by opening __Preferences -> Text Editor -> Code Templates__. Here you'll see a list of all the existing templates. Click __Add__ to create a new template.
//IMAGE
Enter a __name__, __description__ and a __group__.

Now when it comes to the template we need to follow a specific syntax, as we want the consumer of the template to be able to specify the type and name of our property. As you can see I enter __%type%__ for the type ___%name%__ for the backing field, the _ is just part of my coding style. Finally __%NAME%__ as the name of the property
{% highlight csharp %}
$type$ _$name$;
public $type$ $NAME$ {
	get 
	{
		return _$name$;
	}
	set 
	{
		 SetProperty(ref _$name$, value);
	}
}
{% endhighlight %}

As you can see this can speed things up a bit and save you from writing boiler plate code over and over. As an enhancement you can also make use of the __%end%__ keyword, to decide where the cursor should be positioned after the template is complete. An idea for another code tempate might be unit testing with the __Arrange__ __Act__ __Assert__ pattern __AAA__. 

### How could Code Templates be improved
One thing that would be really great would be the ability to __share__ code templates among your team as you can with Visual Studio and Resharper, or just to have a Code Template sharing repository to consume. 









