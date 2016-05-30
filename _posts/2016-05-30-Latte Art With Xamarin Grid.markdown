---
layout: post
title:  "Latte Art With Xamarin Grid"
date:   2016-05-30
---

<p class="intro">
<span class="dropcap">A</span> very common task when working with UI is to stack elements upon each other in a z-order. Often a bit complicated task, using either a Relative- or AbsoluteLayout in Xamarin Forms. Why not use a Grid for simple tasks, like making latte art? 
</p>

### Simple stacking can be made complex
A while back I had to stack some UI elements in a project. Making a relative simple overlay, one image on top of another. One of the resources on the topic I found was 
(Xamarin.Forms in anger, Mall)[https://www.syntaxismyui.com/xamarin-forms-in-anger-mall-dash/] by Adam J Wolf. A great example of using the __RelativeLayout__. Also a bit complex just to stack single elements. ___Please note that if you need more control of the layout you should follow the great example by Adam___ 

### We got a simpler solution
While __Absolute-__ and __-RelativeLayout__ works great, especially if you have a complex overlay they are the ones to use. In my case I just needed a single element overlay. As we already use Grid as a layout in many views, we know that elements are added to rows and columns by specifying an index like Grid.Row="0" etc. Now what happens if we add multiple elements to the same index?
__The will be stacked!!__, first element at the bottom, the next one above and so on. Let's try, by simple adding a grid to a page and add 2 images to the same row.

{% highlight html %}
<ContentPage
BackgroundColor="Black">
<Grid>
	<Grid.RowDefinitions>
		<RowDefinition
			Height="*">
		</RowDefinition>
	</Grid.RowDefinitions>
	<Image
		Grid.Row="0"
		Source="coffee.png">
	</Image>
	<Image
		Grid.Row="0"
		Margin="0,-40,0,0"
		Source="foam.png">
	</Image>
</Grid>
</ContentPage>
{% endhighlight %}

The result speaks for itself!
__Your coffee is servered__
<img src="{{ '/assets/img/gridstack/coffeestack.png' | prepend: site.baseurl }}"   alt="coffee">
And why not add an __ActivityIndicator__ while brewing, it's just another element to stack.
<img src="{{ '/assets/img/gridstack/coffeebrew.png' | prepend: site.baseurl }}"  alt="coffeebrew">











