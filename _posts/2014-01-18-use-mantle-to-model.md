---
layout: 	post
title:		使用Mantle处理Model层对象
category:	iOS
tages:		

---

我们都会在开发中遇到如何对Model层对象进行建模的问题，比如，将服务器请求下来的json转换为我们本地的Object。这部分，有许多令人讨厌的代码要写，比如类型的转换、json的解析等等，没有什么技术含量，但是又必须去写。

当我们习惯了这种方式后，我们往往就麻木了，认为这些东西是必须要写的，所以，虽然很痛苦很恶心，但是还是会硬着头皮去写，去写那些可恶的代码。

那么，真的没有更好的解决办法吗？

今天，我们就一起来学习一下[Mantle](https://github.com/MantleFramework/Mantle)，看看它是如何帮我们处理Model Object的。

我们要对每天的天气情况构建一个对象，数据从这个接口返回[openweathermap](http://api.openweathermap.org/data/2.5/weather?lat=39.915354&lon=116.578584&units=imperial)，包括经纬度、城市名、时间、温度、风向、气压等天气参数。

	@interface JHWeatherCondition : MTLModel <MTLJSONSerializing>
	
	@property (nonatomic, strong) NSDate *date;
	@property (nonatomic, copy) NSString *locationName;
	@property (nonatomic, assign) CGFloat humidity;
	@property (nonatomic, assign) CGFloat temperature;
	@property (nonatomic, assign) CGFloat temperatureMax;
	@property (nonatomic, assign) CGFloat temperatureMin;
	@property (nonatomic, strong) NSDate *sunriseTime;
	@property (nonatomic, strong) NSDate *sunsetTime;
	@property (nonatomic, copy) NSString *condition;
	@property (nonatomic, assign) CGFloat windSpeed;
	@property (nonatomic, assign) CGFloat windDegree;
	@property (nonatomic, copy) NSString *icon;
	
	@end

如上，我们首先定义所需要的属性，由于Mantle是基于KVO实现的，所以必须是属性才可以，普通的成员变量是无法使用Mantle的。

	@interface JHWeatherCondition : MTLModel <MTLJSONSerializing>

我们需要继承MTLModel，同时需要实现<MTLJSONSerializing>，告诉Mantle如何根据我们的规则把Json格式转换为Model Object。下面的方法就是<MTLJSONSerializing>必须要实现的一个方法，它指明了如何把json的keypath和Model Object的key对应起来。比如，self.date就对应于json中key为dt的部分。

	+ (NSDictionary *)JSONKeyPathsByPropertyKey
	{
	    return @{
	             @"date": @"dt",
	             @"locationName": @"name",
	             @"humidity": @"main.humidity",
	             @"temperature": @"main.temp",
	             @"temperatureMax": @"main.temp_max",
	             @"temperatureMin": @"main.temp_min",
	             @"sunriseTime": @"sys.sunrise",
	             @"sunsetTime": @"sys.sunset",
	             @"condition": @"weather.main",
	             @"windSpeed": @"wind.speed",
	             @"windDegree": @"wind.deg",
	             @"icon": @"weather.icon",
	             };
	}

上面的基本规则都很直白，一看就明白，无需多说。下面我们来讨论一些具体的类型转换相关的内容。

在json中，我们可以注意到`dt: 1390097793`，明显是一个double类型的Unix时间戳，但是我们在`JHWeatherCondition`中定义却是`@property (nonatomic, strong) NSDate *date;`，是一个`NSDate`的对象，那么我们该怎么去把这两个类型进行转换呢？还是Mantle本身能智能的帮我们转换？

答案是，Mantle没办法自动的帮我们做这种类型转换，但它提供了相关的Delegate，我们可以实现这些Delegate方法，这样，Mantle就能按照我们指定的方式进行转换了。

Mantle提供了两种方式。第一种是实现<MTLJSONSerializing>协议中的下面这个方法。注释写的很清楚，大家仔细看看。

	// Specifies how to convert a JSON value to the given property key. If
	// reversible, the transformer will also be used to convert the property value
	// back to JSON.
	//
	// If the receiver implements a `+<key>JSONTransformer` method, MTLJSONAdapter
	// will use the result of that method instead.
	//
	// Returns a value transformer, or nil if no transformation should be performed.
	+ (NSValueTransformer *)JSONTransformerForKey:(NSString *)key;

使用这种方式，大概的实现方法如下：

	+ (NSValueTransformer *)JSONTransformerForKey:(NSString *)key
	{
	    if ([key isEqualToString:@"date"]) {
	        return [MTLValueTransformer reversibleTransformerWithForwardBlock:
	                ^id(NSNumber *number)
	                {
	                    NSTimeInterval secs = [number doubleValue];
	                    return [NSDate dateWithTimeIntervalSince1970:secs];
	                } reverseBlock:^id(NSDate *d) {
	                    return @([d timeIntervalSince1970]);
	                }];
	    } else if ([key isEqualToString:@"condition"]) {
	        return [MTLValueTransformer reversibleTransformerWithForwardBlock:
	                ^id(NSArray *values) {
	                    return [values firstObject];
	                } reverseBlock:^id(NSString *str) {
	                    return @[str];
	                }];
	        
	    }
	    
	    return nil;
	}

这种方式有个坏处，代码会很容易膨胀，`if-else-if`会很多，比较难维护。


第二种方式的基本思路就是把第一种方式的`if-else-if`拆开，独立成一个个的小方法，便于维护。它的方法名必须遵循特定的规则，规则如下：

	SEL selector = MTLSelectorWithKeyPattern(key, "JSONTransformer");

代码很明白，下面的注释也很明白。

	// Specifies how to convert a JSON value to the given property key. If
	// reversible, the transformer will also be used to convert the property value
	// back to JSON.
	//
	// If the receiver implements a `+<key>JSONTransformer` method, MTLJSONAdapter
	// will use the result of that method instead.
	//
	// Returns a value transformer, or nil if no transformation should be performed.
	+ (NSValueTransformer *)JSONTransformerForKey:(NSString *)key;

使用第二种方式，大概的样子如下，是一个个独立的小函数，这样看起来就清晰多了，我个人推荐使用第二中方式。

	+ (NSValueTransformer *)dateJSONTransformer
	{
	    return [MTLValueTransformer reversibleTransformerWithForwardBlock:
	            ^id(NSNumber *number)
	            {
	                NSTimeInterval secs = [number doubleValue];
	                return [NSDate dateWithTimeIntervalSince1970:secs];
	            } reverseBlock:^id(NSDate *d) {
	                return @([d timeIntervalSince1970]);
	            }];
	}
	
	+ (NSValueTransformer *)sunriseTimeJSONTransformer
	{
	    return [self dateJSONTransformer];
	}
	
	+ (NSValueTransformer *)sunsetTimeJSONTransformer
	{
	    return [self dateJSONTransformer];
	}
	
	+ (NSValueTransformer *)conditionJSONTransformer
	{
	    return [MTLValueTransformer reversibleTransformerWithForwardBlock:
	            ^id(NSArray *values) {
	                return [values firstObject];
	            } reverseBlock:^id(NSString *str) {
	                return @[str];
	            }];
	}

OK，基本工作我们已经都完成了，下面我们需要使用下面的一句代码就可以得到我们的Model Object了。

	JHWeatherCondition *weatherCondition = 
		[MTLJSONAdapter modelOfClass:[JHWeatherCondition class] fromJSONDictionary:json error:nil];
		
对于由一个Model Object转换为json格式，需要调用:
	
	NSDictionary *jsonDictionary = [MTLJSONAdapter JSONDictionaryFromModel:weatherCondition];
	
wow! 貌似很简单的样子啊！

怎么样，上面的就是使用Mantle处理json和Model Object的基本步骤，和你之前的处理方式相比，哪种更好呢？

文章写完了，说点闲话吧！

习惯这个东西真的很可怕，渐渐的你就会麻木，认为那是理所当然的。我们一定要时时的对自己保持警醒，多思考，不要被习惯或者所谓的权威所束缚中。其实，更多情况下，束缚住我们自己的就是我们自己，其实就是自己懒，懒得去思考。一天到晚，貌似我们很忙，但是，真正用来思考的时间又会有多少呢？自己真的很忙吗？麻木的忙，忙的麻木。


