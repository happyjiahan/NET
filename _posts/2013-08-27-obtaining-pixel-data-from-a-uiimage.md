---
layout: 	post
title:		如何从UIImage中获取像素信息
category:	ios开发
tages:		开发

---


翻译自[Obtaining pixel data from a UIImage](http://b2cloud.com.au/tutorial/obtaining-pixel-data-from-a-uiimage)


UIImage是我们在ios中非常熟悉的数据结构了，用它来存储图片非常方便。在OpenCV类库中，使用是另外的数据结构来存储图片，这时，我们就面临一个问题：如何才能把UIImage装换为其他类库也能识别的数据结构呢？

尽管不同的图片处理类库有不同的数据结构来处理图片，但是有一种结构是被所有的图片处理类库所识别的，那就是**raw**格式，也称之为**原生格式**。在raw格式中，每个像素通过一个**unsigned byte**的数组来表示，这个数组中存储了这个像素的RGBA值或者灰度值。如果你处理的是视频格式，你可能需要使用[**YUV**](http://en.wikipedia.org/wiki/YUV)，否则，你需要使用的就是RGBA值或者灰度值，这些信息会被存储在一个数组中。你可能会想，我如何才能从UIImage中方便的获取相应的像素信息呢？啊哈，你有福了。我实现了一个**Category**来解决这个问题，如下:

*	[UIImage+Pixels.h](http://b2cloud.com.au/files/UIImage+Pixels.h)

*	[UIImage+Pixels.m](http://b2cloud.com.au/files/UIImage+Pixels.m)

你可以使用如下代码来获取每个像素的灰度值：
	
	for(int i=0;i<self.size.height;i++)
	{
		for(int y=0;y<self.size.width;y++)
		{
			NSLog(@"0x%X",pixelData[(i*((int)self.size.width))+y]);
		}
	}
由于*0x00*代表没有亮度，而*0xFF*代表最大的亮度，所以，对于黑色的图片，返回的是一系列的*0x00*，对于白色的照片，返回的是一系列的*0xFF*。

你可以使用如下代码获取每个像素的RGBA值：
	
	for(int i=0;i<self.size.height;i++)
	{
		for(int y=0;y<self.size.width;y++)
		{
			unsigned char r = pixelData[(i*((int)self.size.width)*4)+(y*4)];
			unsigned char g = pixelData[(i*((int)self.size.width)*4)+(y*4)+1];
			unsigned char b = pixelData[(i*((int)self.size.width)*4)+(y*4)+2];
			unsigned char a = pixelData[(i*((int)self.size.width)*4)+(y*4)+3];
			NSLog(@"r = 0x%X g = 0x%X b = 0x%X a = 0x%X",r,g,b,a);
		}
	}
	
对于红色的图片，你会得到如下结果：*r = 0xFF g = 0×0 b = 0×0 a = 0xFF*

对于蓝色的图片，你会得到如下结果：*r = 0×0 g = 0×0 b = 0xFF a = 0xFF*

对于每个像素，需要通过**red, green, blue, alpha**四个值来代表，即**RGBA值**。对于**alpha**值，你可能会感到疑惑，这个值是标志图片的不透明度的，*0x00*代表100%透明，*0xFF*代表100%不透明。我发现，在大多数图片处理中，你最好忽略**alpha**这个值。


下面我们来讨论下如何从UIImage中取出某个特定坐标的像素？

对于灰度值，由于每个像素只由一个值代表，故我们可以把一张图片的像素布局看成一个表格，每个像素有自己的一个索引，对于第一个像素它的索引是*（0, 0）*，第二个像素索引是*（1, 0）*。用如下代码即可取到对应坐标的像素的灰度值：
	
	int x = 10;
	int y = 2;
	unsigned char pixel = pixels[(y*((int)whiteImage.size.width))+x];
	
对于RGBA值，基本原理和上面的灰度值一样，但是需要注意的一点是，RGBA值需要四个字节来去表示一个像素。
{% highlight cpp %}
	
	int x = 10;
	int y = 2;
	unsigned char pixel = pixels[(y*((int)whiteImage.size.width)*4)+(x*4)];
	
{% endhighlight %}
	
如果你看到了这里，那就说明你已经能够从UIImage中获取像素信息了，Congratulations!
但是，最后有一点非常重要的事情你需要注意。你使用完这些方法给你的数据之后，你需要手动的去`free`这些数据，否则就会出现内存泄露。可能有些人会疑惑我为什么把返回值设定为`(unsigned char*)`而不是一个`NSArray`，答案很简单，直接操作`(unsigned char*)`要比操作`NSArray`快很多，而我的方法默认是为那些对操作速度要求很高的同学提供的。
