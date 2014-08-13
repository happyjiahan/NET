---
layout: 	post
title:		如何实现一个View Controller Containers Part 2
category:	iOS
tages:		

---


本文翻译自[View Controller Containers: Part II](http://stablekernel.com/blog/view-controller-containers-part-ii/)，感谢原作者的精彩文章。

请确认你已经看过了这个系列的[第一篇文章](http://stablekernel.com/blog/view-controller-containers-part-i/)。

在本篇文章中，我们将要实现一个全新的 view controller container，STKMenuViewController。回到我们上一篇介绍的container 的 ‘三巨头’ 的概念，让我们来看看我们这个 container 的对应关系：

	1.	STKMenuViewController的组织和遍历将和 UITabBarController 相同：有一个 view controllers 的列表，用户可以在任何时候从中选择。
	2.	STKMenuViewController的布局允许每一个 child view controller 的视图都是全屏展示的。刚开始时，在 view controllers 之间切换时并没有动画，但是我们会在后面的部分去增加动画相关的功能。
	3.	STKMenuViewController的 built-in interface 是一个全屏对的遮罩，当用户三指点击屏幕时，遮罩出现。
	
最后，当它的 built-in interface 可见时，STKMenuViewController看起来是这个样子的：

![](../album/view-controller-containers-part-ii/1.png)

在屏幕上三指点击将会使这个环形菜单视图展现出来。每一个代表child view controller 的图标将会在这个环形上展现出来。点击图标，将会展现出对应的 view controller。点击其他地方将会隐掉这个菜单。

当实现一个 view controller container 时，最好是从屏幕上的展示元素开始。让我们来制定以下我们将要完成的目标：

	1.	可以创建一个STKMenuViewController的实例，并且把它设置为 window 的 root view controller.
	2.	能够提供给STKMenuViewController一个 view controllers 的列表，使其可以在这些 view controllers 之间切换。
	3.	能够展示这个列表中first view controller 的视图。

通过实现这些目标，我们将要涵盖“三巨头”概念的前两个的主要方面：组织/遍历 和 布局。但是我们不会详细的介绍每一行代码，你可以从 github 上得到源码。我们将要重点关注关键的部分已经隐藏在他们后面的概念。

###站在巨人的肩膀上

由于要实现的 container view controller 和 tab bar controller 的组织一样，在布局上也很相似，所以，让我们先来看看 tab bar controller 的视图层次：

![](../album/view-controller-containers-part-ii/2.png)

没错，这是一个比较大的视图结构。tab bar controller 的 view 是一个 UILayoutContainerView 的实例。这个view 的子 view 是 tab bar 自己和一个 UITransitionView。这个 transition view 的子 view 是一个包含了selected view controller 的 view 的 UIViewControllerWrapperView。

这意味着，一个 tab bar controller 在新视图之间切换时遵从以下步骤：
	
	1.	新选中的 view controller 的 view 最为一个 subview 被包装到一个新的 UIViewControllerWrapperView 中，并且设置一些约束，这样这个新的 view 就能够适应它的 wrapper 的大小了。
	2.	这个 wrapper view 被加到 UITransitionView 上作为一个 subview，同时也需要设置一些约束以使其能够紧密贴合。
	3.	老的view 的 wrapper 被从 UITransitionView 中移除，这样也移除了老的 view。
	
这个视图结构的价值在于它很清楚的分离了每一个view角色，允许根据 tab bar controller 的基本行为进行划分。这就是 Programming 101 ：干净的角色分离通过减少在行为变化时需要改动的代码来最大程度的减少故障。而这种故障，其中一个变化产生另一个问题，是一个非常糟糕的类故障。

UILayoutContainerView 建立 tab bar 和 transition view 之间的相对布局只有一次。这也意味着当屏幕旋转时，这些规则的规则的建立也只有一次。通过把布局逻辑移到这个单独的 layer 中，不管我们将要切换到那个 view，tab bar controller 不需要从新建立约束条件，或者在布局它的界面时做一些额外的措施。

因此，UITransitionView 的position和 size是由它持有的关于 UILayoutContainerView 和 UITabBar的constraints决定的。这使得在视图之间的过渡遵从简单的步骤： 只需要把新的 view 和它的 wrapper 添加到这个 transition view 上，并且使其适配合适即可。
我们要是想改变 tab bar 的 size 或者 position，我们不需要改动视图切换相关的代码。

UIViewControllerWrapperView 的角色有一点不那么显而易见。为了理解它，我们需要首先理解一个 view controller 的最基本的规则就是它从不修改另外一个 view controller 的视图结构。这包括修改视图结构自身（添加或者移除视图）和改变 view 的各种属性（比如透明度和 frame）。

这条规则的原因很简单：弄糟另一个 view controller 的视图结构是不安全的。有三种可能的情况会发生。考虑一个例子，一个 view controller 试图去修改另外一个 view controller 的 button：

	[[someOtherViewController loginButton] setFrame:someFrame];
	
First possible type of failure, escalation level: ‘Aw, shucks.’ 这个`someOtherViewController`使用了一种机制来保证这个`loginButton`保持在原来的位置上，因此使这个代码没有效果。

Second possible type of failure, escalation level: ‘Oh, $#!@.’  这个`someOtherViewController`使用这个`loginButton`的位置作为其他元素的布局参考点，因此，整个界面都乱掉了或者一些约束不能被满足了，导致程序崩溃。

Third possible type of failure, escalation level: ‘You’re fired.’ 