---
layout: 	post
title:		iOS 类簇(Class Cluster)使用心得 
category:	iOS
tages:		

---

我们都知道在iOS中类簇的使用是非常普遍的，比如`NSNumber`、`NSString`、`NSArray`等等都是类簇。我们以`NSNumber`举例来说，对于int，bool, unsigned int 等等数据类型，我们如何把它们封装成类的形式呢？ 通常情况下我们可能会想到，对于每一种数据类型独立封装成一个类，比如对于int 类型我们可以做一个NSInt的类，以此类推。这样想是正确的，但是，我们再来想想这样会有什么问题呢？当需要支持的数据类型越来越多时，这些类也会相应的越来越多，那么对于开发者来说，我们就需要记住越来越多的类。那么还有没有好的方案呢？哈哈，这个时候类簇就闪亮登场了。

现在我们遇到的问题是，相似的类越来越多，对于开发者来说需要太多的记忆。显然，我们解决问题的方向肯定是朝着减少暴露给开发者的类的数量这个方向。让我们来回忆一下设计模式，抽象工场模式不就是干这个事情的嘛！

对于`NSNumber`，我们使用一个`NSNumber`来作为int，bool，long等原生数据类型的工厂，对外定义一堆的工厂方法，在`NSNumber`内部则转调给相应的像`NSInt`这样的内部类去实现具体的逻辑。这样，我们遇到的问题就得到了完美的解决。

下面我们来自己实现一个具体的类簇。场景是这样的，我现在有一个App，只支持英文版，现在我想让其也支持中文版和日文版等等。
我们先来看看模型类吧！

	@interface PowerUpDetail : WMModelObject

	@property (nonatomic, copy)     NSString* name;
	@property (nonatomic, copy)     NSString* desc;

	@end
name 和 desc 分别是英文版对应的字段，现在我想实现中文版，并为以后的其他语言版本做好扩展结构。

现在有几个方案：
	
	1.	继承PowerupDetail，实现子类PowerupDetailCN和PowerupDetailEN在所有调用PowerupDetail的地方，区分出到底应该调用PowerupDetailCN还是PowerupDetailEN。
	2.	把PowerupDetail做成类簇，对外暴露通用接口，对内转调给内部类来实现具体逻辑。

方案1是最容易想到的，但是它的确定是需要对已有的调用代码进行修改，而且如果将来随着越来越多的语言被支持，这些类将会非常多，在调用的地方需要一堆的if-else语句来区分，代码很丑，工作量很大。

方案2是比较好的一种方案，做到了对外暴露最小化，而所有调用的地方都可以不用修改，所有的修改可以只在PowerupDetail这个类中即可完成。

下面上代码~\(≧▽≦)/~啦啦啦

在PowerUpDetail.h 文件中：

	@interface PowerUpDetail : WMModelObject

	@property (nonatomic, copy)     NSString* name;
	@property (nonatomic, copy)     NSString* desc;

	- (NSString *)localizedName;
	- (NSString *)localizedDesc;

	@end

在PowerUpDetail.m 文件中：

	#pragma mark - For English Version
	@interface _PowerUpDetailEN : PowerUpDetail
	@end

	@implementation _PowerUpDetailEN
	@end

	#pragma mark - For Chinese Version
	@interface _PowerUpDetailCH : PowerUpDetail
	@property (nonatomic, copy) NSString *nameCH;
	@property (nonatomic, copy) NSString *descCH;
	@end

	@implementation _PowerUpDetailCH

	- (instancetype)initWithDictionary:(NSDictionary *)dic
	{
	    if (self = [super initWithDictionary:dic]) {
	        _nameCH = [[dic objectForKey:@"nameCH"] copy];
	        _descCH = [[dic objectForKey:@"descCH"] copy];
	    }
	    return self;
	}

	- (NSString *)localizedName
	{
	    return self.nameCH;
	}

	- (NSString *)localizedDesc
	{
	    return self.descCH;
	}

	@end


	#pragma mark - Class Cluster
	@implementation PowerUpDetail

	+ (instancetype)alloc
	{
	    if ([self class] == [PowerUpDetail class]) {
	        if (WMIsChinesePreference) {
	            return [_PowerUpDetailCH alloc];
	        } else {
	            return [_PowerUpDetailEN alloc];
	        }
	    } else {
	        return [super alloc];
	    }
	}

	- (instancetype)initWithDictionary:(NSDictionary *)dic
	{
	    if (self = [super initWithDictionary:dic]) {
	        _name       = [[dic objectForKey:@"name"] copy];
	        _desc       = [[dic objectForKey:@"desc"] copy];
	    }
	    return self;
	}

	- (NSString *)localizedName
	{
	    return self.name;
	}

	- (NSString *)localizedDesc
	{
	    return self.desc;
	}

	@end

这里，我实现了两个内部类`_PowerUpDetailCH`和`_PowerUpDetailEN`，他们都继承于`PowerUpDetail`，而`PowerUpDetail`暴露给外边两个方法：		
		- (NSString *)localizedName;
		- (NSString *)localizedDesc;

这些方法可以在子类中按需要进行复写。

需要特别注意下类簇`PowerUpDetail`的alloc方法，这里要注意 if ([self class] == [PowerUpDetail class]) 这个条件的使用，否则会出现死循环，原因你应该很容易看出来，我就不啰嗦了！

	+ (instancetype)alloc
	{
	    if ([self class] == [PowerUpDetail class]) {
	        if (WMIsChinesePreference) {
	            return [_PowerUpDetailCH alloc];
	        } else {
	            return [_PowerUpDetailEN alloc];
	        }
	    } else {
	        return [super alloc];
	    }
	}

OK，我讲完了，类簇其实很简单，就是一个抽象工厂模式的iOS版本的实现。

另外，推荐阅读一下这篇文章[类簇在iOS开发中的应用](http://limboy.me/ios/2014/01/04/class-cluster.html)，我也是从这篇文章中学到的知识，非常感谢李忠。




