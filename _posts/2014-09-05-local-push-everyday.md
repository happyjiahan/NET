---
layout: 	post
title:		iOS Local Push Everyday
category:	iOS
tages:		

---

我想在每天晚上的8点钟，发一条 Local Push 通知用户今天的运动记录，该怎么实现呢？
我们知道，UILocalNotification 的 `fireDate` 是一个具体的时间，但是我们现在需要的不是说要在2014年9月5号的20点发一条 Local Push，而是在每天的20点都发一条 Local Push，这个时间就不好给一个具体的时间了。那么我们怎么办呢？

其实很简单，我们需要计算出下一个20点所对应的具体的时间，比如我先计算出当天的20点所对应的 NSDate *thisDate，如果thisDate已经过去了，那么我们就需要计算下一天的20点所对应的 nextDate，只需要在 thisDate 基础上再加上24个小时就是 nextDate。好了，看代码吧，知道明白思路了就很简单了。

	- (void)scheduleLocalNotificationAtHour:(NSInteger)hour minute:(NSInteger)minute
	{
	    NSCalendar *calendar = [NSCalendar autoupdatingCurrentCalendar];
	    NSDateComponents *dateComps = [calendar components:NSYearCalendarUnit|NSMonthCalendarUnit|NSDayCalendarUnit fromDate:[NSDate date]];
	    [dateComps setHour:hour];
	    [dateComps setMinute:minute];
	    NSDate *nextDate = [calendar dateFromComponents:dateComps];
	    if ([nextDate timeIntervalSinceNow] < 0) {
	        nextDate = [nextDate dateByAddingTimeInterval:24 * 60 * 60];
	    }
	    
	    UILocalNotification *localNotif = [[UILocalNotification alloc] init];
	    if (localNotif) {
	        localNotif.fireDate = nextDate;
	        localNotif.timeZone = [NSTimeZone defaultTimeZone];
	        
	        localNotif.alertBody = NSLocalizedString(@"alert body: You cat need a hug.", nil);
	                               
	        localNotif.alertAction = NSLocalizedString(@"alert action: View Details", nil);
	        
	        localNotif.soundName = UILocalNotificationDefaultSoundName;
	        localNotif.applicationIconBadgeNumber = 1;
	        
	        localNotif.userInfo = @{@"goto" : @"userinfo"};
	        
	        [[UIApplication sharedApplication] scheduleLocalNotification:localNotif];
	    }
	}


另外，推荐阅读一下这篇文章[ios时间那点事--NSCalendar NSDateComponents](http://my.oschina.net/yongbin45/blog/156181)




