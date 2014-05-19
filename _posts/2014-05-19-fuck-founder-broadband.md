---
layout: 		post
title:		方正宽带给我的网站挂广告，Fuck U !
category:	Fuck
tages:		

---

昨天发现我的个人网站右下角怎么突然出现了广告，当时就很纳闷，我明明没有加广告啊。但是刷新后又不见了，所以没有在意。今天早上，打开网站，fuck, 又出现广告了，奶奶的，这次我一定要查出来是谁给我挂的广告。

直接看源码，发现下面的广告代码。

	<script src="http://59.108.34.106/static/FloatingContent/rllqVohJtpBdTtvDvsHu1w/floating-frame.js" type="text/javascript"></script> 

让我们来看看这个IP地址是谁的吧，通过IP地址查询网站结果如下
	
	查询的 IP：59.108.34.106 来自：北京市 方正宽带

	GeoIP: Beijing, China

	China Telecom Beijing
	
我艹，竟然是方正宽带，我们小区的宽带就是方正宽带的。

现在元凶找到了，真他妈恶心，堂堂一个运营商偷偷摸摸的干这种下流的事情，以后再不用方正宽带了。

我的解决办法就是先用本地hosts的方式屏蔽了这个IP地址，但是这种方式并不能根本解决问题。大家如果有好的方案可以和我说下，先谢过了啊。

Fuck U，方正宽带！！！！！！