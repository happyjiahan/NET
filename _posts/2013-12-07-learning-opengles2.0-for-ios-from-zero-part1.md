---
layout: 	post
title:		从零开始学习OpenGL ES 2.0 iOS开发技术 part 1
category:	ios开发
tages:		开发

---

最近在学习如何在iOS开发中使用OpenGL ES 2.0进行绘图操作，有一些心得和经验，与大家分享。首先说明一点，这篇教程的目标读者是已经对iOS的开发有一些基础，但是对OpenGL ES 2.0没有任何基础，又希望在iOS开发中使用OpenGL ES 2.0的开发者。

这是本系列教程的第一部分，本部分的任务是使用OpenGL把View的背景色填充为红色，达到如下图所示的效果。

<img src="../album/view_red.png" style="width: 200px;"/>

呃！是不是有点太简单了呢？直接设置`view.backgroundColor = [UIColor redColor];`不就搞定了吗？是很简单，现在让我们看看如何通过OpenGL ES 2.0来达到相同的效果吧！

OK，Let's coding! 

首先，在Xcode5下新建一个**iOS/Application/Empty Application**的工程，工程名为**HelloGLKit_1**。

<img src="../album/create_empty_project.png" style="width: 500px;" />

我们先运行下，看看到如下图所示的画面。它仅有一个背景色为白色的UIWindow。
<img src="../album/window_white.png" style="width: 200px; border: 3px #ff0000 dotted;"/>
代码如下

{% highlight objc linenos %}
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
    self.window = [[UIWindow alloc] initWithFrame:[[UIScreen mainScreen] bounds]];
    // Override point for customization after application launch.
    self.window.backgroundColor = [UIColor whiteColor];
    [self.window makeKeyAndVisible];
    return YES;
}
{% endhighlight %}


