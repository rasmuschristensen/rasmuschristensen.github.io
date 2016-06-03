---
layout: post
title:  "Black and white images with Xamarin"
description:  "How to make Black and white images with Xamarin on IOS"
date:   2016-05-26
---

<p class="intro">
<span class="dropcap">W</span>hen you take a picture with the phones camera or choose one from the image gallery, how do you then make it black and white?  
</p>

### Pure Xamarin
This post is a general purpose of how to turn a picture from colored to black and white. It's made with pure Xamarin, and if you want to use it in a Xamarin Forms project, just wrap is as a platform dependency, using [DependencyService](https://developer.xamarin.com/guides/xamarin-forms/dependency-service/introduction/). This implementation will focus on the IOS version.

### Let's code

The code we'll use is this one. Notice I wrapped it in a method with a parameter ot type __byte[]__ which is the image to convert, and a return value of type __byte[]_ the black and white edition of the image.

{% highlight csharp %}
public byte[] MakeGrayScale(byte[] imageAsBytes)
{
	UIImage image = ImageFromByteArray(imageAsBytes);
	var height = image.Size.Height;
	var width = image.Size.Width;
	var imageRect = new CGRect(0, 0, width, height);

	using (CGBitmapContext context = new CGBitmapContext(IntPtr.Zero,
											(int)width, (int)height, 8,
											0, CGColorSpace.CreateDeviceGray(),
											CGImageAlphaInfo.None))
	{
		context.DrawImage(imageRect, image.CGImage);

		UIKit.UIImage convertedImage = UIKit.UIImage.FromImage(context.ToImage(),
														 0, image.Orientation);

		return convertedImage.AsJPEG().ToArray();
	}
}
{% endhighlight %}

The concept is to make a new canvas of the same dimension as the original image, the __context__. When we define the context, we can also define a __ColorSpace__. Here we can choose __CreateDeviceGray__ to make it black and white and we also remoce the Alpha.
With the context and the __rectangle__ in the same dimension as the original image, we can draw the _new_ image on the __canvas__.
Finally we need to create a new __UIImage__ from the context and the orientation of the original image. We now have a copy of the original image, with a different __ColorSpace__.

The last step in the code snippet converts the image to JPEG and ensures we return it as __byte[]__
 
You can find the samle code as a gist [here](https://gist.github.com/rasmuschristensen/db61ca59c06fabb9201f60c7413646a7)










