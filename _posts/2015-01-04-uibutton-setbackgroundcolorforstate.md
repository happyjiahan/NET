---
layout: 	post
title:		UIButton setBackgroundColor:ForState）
category:	ios开发
tages:		ios开发

---

# UIButton setBackgroundColor:ForState



在使用`UIButton`时，很多时候我们需要一个类似于`- (void)setBackgroundColor:(UIColor *)color forState:(UIControlState)state`这样的方法，来实现在不同的状态下使用不同的`backgroundColor`。遗憾的是，iOS默认并没有实现这个方法，那我们就自己来实现它。

让我们先来看看对于设置`BackgroundImage`，`UIButton`提供了如下方法：
```
- (void)setBackgroundImage:(UIImage *)image forState:(UIControlState)state UI_APPEARANCE_SELECTOR; // default is nil

- (UIImage *)backgroundImageForState:(UIControlState)state;

```

类似的，我们的函数实现声明如下：

```
- (void)setBackgroundColor:(UIColor *)color forState:(UIControlState)state;

- (UIColor *)backgroundColorForState:(UIControlState)state;

```

