---
layout: 	post
title:		自己动手实现一个像Clear似的手势驱动的Todo List应用（1/3）
category:	ios开发
tages:		ios开发

---

**翻译自[How to Make a Gesture-Driven To-Do List App Like Clear: Part 1/3](http://www.raywenderlich.com/21842/how-to-make-a-gesture-driven-to-do-list-app-part-13)
感谢原作者的分享**

这篇文章是教程组成员[Colin Eberhardt](http://www.raywenderlich.com/about#colineberhardt)编写的，他是[ShinobiControls](http://www.shinobicontrols.com/)的CTO,你可以在这里下载他公司的app[ShinobiPlay](http://click.linksynergy.com/fs-bin/stat?id=9QfxPcziZp0&offerid=146261&type=3&subid=0&tmpid=1826&RD_PARM1=https%253A%252F%252Fitunes.apple.com%252Fus%252Fapp%252Fshinobiplay%252Fid545634307%253Fmt%253D8%2526uo%253D4%2526partnerId%253D30),这是作者的[Google+](https://plus.google.com/u/0/104181672098184856535?rel=author) 和 [Twitter](https://twitter.com/ColinEberhardt)，你可以通过它们和作者交流。

本系列教程一共有3篇，我讲教会你自己动手做一个简单地Todo List的App, 这个App就像Clear应用那样，你找不到一个按钮。普通的界面元素是不是让你很心烦，应为几乎所有的应用上都有它们的身影。YES，今天我们就一起学习开发一个手势驱动的的Todo List，是不是很有趣？在我们的应用中，我们将通过一些列的手势和用户产生交互，我们会使用swipe、pull-to-add、pinch等手势，而不再使用通常的Button、Switch等普通界面元素。

本系列教程是为中高级的开发者准备的，你将会使用**gradient layers**, **animations**等技术，并且会使用自定义的**Table View**。这些都要求你具备基本的IOS开发知识，如果你是一个新手，我建议你先看看[这些教程](http://www.raywenderlich.com/tutorials)。

如果你想要更好地学习如何使用手势，本教程就在合适不过了，是不是有点等不及了，少年！哈啊，Come on, baby!
