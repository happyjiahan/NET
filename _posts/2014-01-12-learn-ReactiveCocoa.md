---
layout: 	post
title:		学习ReactiveCocoa
category:	iOS
tages:		

---

很早就从唐巧等技术大牛的博客那里听说过ReactiveCocoa了，但是一直没有好好学习一下，主要原因就是自己懒，今天刚好在学习[iOS 7 Best Practices; A Weather App Case Study: Part 2/2](http://www.raywenderlich.com/55386/ios-7-best-practices-part-2)时，遇到了RAC, 其中的很多地方自己搞不明白，于是就找了些资料，给自己好好补补课。

先把我找到的几篇博客贴上，大家可以看看，都是好文啊，顺便赞一下各位技术大牛，写的博客真的很棒！

+	[github上的官方文档](https://github.com/ReactiveCocoa/ReactiveCocoa)
+	[nshipster上Mattt Thompson大神的文章](http://nshipster.com/reactivecocoa/)
+	[ReactiveCocoa for a better world](https://github.com/blog/1107-reactivecocoa-for-a-better-world)
+	[Functional Reactive Programming on iOS with ReactiveCocoa](http://www.teehanlax.com/blog/reactivecocoa/)
+	[Getting Started with ReactiveCocoa](http://www.teehanlax.com/blog/getting-started-with-reactivecocoa/)
+	[ReactiveCocoa与Functional Reactive Programming](http://blog.leezhong.com/ios/2013/06/19/frp-reactivecocoa.html)
+	[说说ReactiveCocoa 2](http://blog.leezhong.com/ios/2013/12/27/reactivecocoa-2.html)
+	[Basic MVVM with ReactiveCocoa](http://cocoasamurai.blogspot.com/2013/03/basic-mvvm-with-reactivecocoa.html)


下面，我边写代码，边说说自己对RAC的理解吧。

假设我们需要监听一个TextField的text，当其text变化时，我们就打印出一条Log信息。这是一个很简单的需求，你可能会如下实现，先监听UITextFieldTextDidChangeNotification通知，再打印出textField.text。

{% highlight objc linenos %}
[[NSNotificationCenter defaultCenter] addObserver:self
                     selector:@selector(textDidChanged:)
                         name:UITextFieldTextDidChangeNotification
                       object:nil];
{% endhighlight %}

{% highlight objc linenos %}
- (void)textDidChanged:(NSNotification *)ntf
{
    UITextField *textField = [ntf object];
    NSLog(@"%@", textField.text);
}
{% endhighlight %}

上面的实现很简单，貌似没有什么不妥的，可是，请注意，我们把一个简单的需求的实现放在了两个地方，一个地方去监听通知，一个地方打印Log。当代码很多时，这种一个需求，需要多个地方配合实现的代码会很难读懂。同理，对于objc中非常普遍的Delegate, Notification, KVO都存在同样的问题，所以Block的出现了。Block可以使我们再一个地方完成我们想要的事情，不需要在多个地方跳转着去看代码。

那么，既然我们在学习RAC，让我们看看上述的需求，用RAC的方式可以怎么来实现吧！
{% highlight objc linenos %}
[self.textField.rac_textSignal subscribeNext:^(NSString *newText) {
   NSLog(@"text = %@", newText);
}];
{% endhighlight %}



