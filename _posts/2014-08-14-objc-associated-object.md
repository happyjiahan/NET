---
layout: 	post
title:		深入学习Objective-C（二）理解 objc 关联对象 (Associated Objects)
category:	iOS
tages:		

---

今天看了下 objc 2.0 引入的强大特性：关联对象，下面把我的理解与大家分享一下。

我们都知道，我们在普通的 objc 类中，一般我们都会把成员变量声明在@interface中，如果你想把成员变量暴露在头文件中，你可以把它声明在实现文件中，甚至你也可以放在类扩展的区域中，但是，你却不能在普通的类目中声明成员变量。因为普通的类目只是用来扩展方法的，不能用来扩展成员变量。

有些时候，我们在设计代码时，会把代码分离成很多个类目，这样方便代码的管理。如下所示：

![](../album/associate-objects/1.png)

对每种类型的 API 单独放在对应的类目中，在代码组织上就非常清晰。但是，现在有一个这样的问题，有时候，我们需要在类目中保存一些状态，可以使用成员变量的形式来实现，但是，这些状态又是相对独立的，只在这个类目中会被用到，其他的类目不会使用它。所以，我们想把这个成员变量隐藏在这个类目中，对于其他的类目，这个成员变量都不可见。
	
如果类目允许扩展成员变量的话，这个问题就很好解决，直接在类目的实现文件里声明一个成员变量即可。这样既能对外部隐藏这个成员变量，又能在这个类目中使用它。很完美的解决方案，但是不幸的是，objc 不允许在类目中扩展成员变量。所以你不得不在类的声明中声明需要的成员变量，而且还需要把它暴露出来，以使你的类目能够使用它。这样，随着类目越来越多，你不得不在类中声明越来越多原本需要对外隐藏的成员变量。

我们上面遇到的问题，在一定程度上可以使用关联对象来解决，稍后我们会看一个具体的例子。现在，我们先来看看关联对象的基本使用方法。

我们在使用关联对象时，主要会用到下面两个方法：

	void objc_setAssociatedObject(id object, void *key, id value, objc_AssociationPolicy policy)
	
	id objc_getAssociatedObject(id object, void *key)

我们通过代码来看看如何使用它：

    NSURL *u1 = [NSURL URLWithString:@"blog.codingcoder.com"];
    objc_setAssociatedObject(u1, @"u1key", @"s1assciated", OBJC_ASSOCIATION_COPY);
    
    id get1 = objc_getAssociatedObject(u1, @"u1key");
    
    
    NSURL *u2 = [NSURL URLWithString:@"u2"];
    id get2 = objc_getAssociatedObject(u2, @"u1key");

objc_setAssociatedObject 就是把一个对象关联到另外一个对象上去，比如下面的代码就是把一个字符串`@"XXXX"`关联到对象`u1`上面。这样就类似于对象`u1`对象上多了一个成员变量。

	objc_setAssociatedObject(u1, @"u1key", @"XXXX", OBJC_ASSOCIATION_COPY);
	
特别注意一点，这里的关联对象，是关联到某一个具体的对象的，并不是关联到类的。

    NSURL *u2 = [NSURL URLWithString:@"u2"];
    id get2 = objc_getAssociatedObject(u2, @"u1key");
    
上面的代码中，`u2`变量并没有关联一个对象，所以，你不能通过`objc_getAssociatedObject`获取一个值。

另外需要注意一点，对象和被关联对象的生命周期是相互独立的。

> 根据WWDC 2011, Session 322 (第36分钟左右)发布的内存销毁时间表，被关联的对象在生命周期内要比对象本身释放的晚很多。它们会在被 NSObject -dealloc 调用的 object_dispose() 方法中释放。

下面我们来看一个例子，相信 AFNetworking 这个类库大家应该都很熟悉。在这个类库中，有一个`UIImageView+AFNetworking`的类目。

![](../album/associate-objects/2.png)

我们看一下源码：

![](../album/associate-objects/3.png)

我们可以看到，它生命了一个私有属性`af_imageRequestOperation`，而它通过自定义 getter /  setter 方法，使用`objc_getAssociatedObject` 和 `objc_setAssociatedObject`方法来达到类似与添加成员变量的目的，而这个变量对外部是完全隐藏的。所以，这也就解决了我们最开始描述的那个类目无法扩展成员变量的问题。

最后，需要注意的一点是，这种objc_setAssociatedObject的方式是运用了 objc runtime 的特性，它确实很强大，但是，相对应的，不正确的使用或者滥用 runtime 会带来比较麻烦的 bug。所以，只有在你确定必须要使用 runtime 特性时，而且你完全明白你这么做所带来的影响时，才使用这种特性。一定不要为了使用这种技术而使用它。


参考文章：
	
+	[http://nshipster.cn/associated-objects/](http://nshipster.cn/associated-objects/)
+	[理解 Objective-C Runtime](http://www.justinyan.me/post/1624)





