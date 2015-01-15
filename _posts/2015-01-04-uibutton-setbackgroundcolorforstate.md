---
layout: 	post
title:		 UIButton setBackgroundColor:ForState
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

具体实现如下所示，代码很简单，不再赘述。

```

@interface WMButton : UIButton
@property (nonatomic, copy) NSString *name;

- (void)setBackgroundColor:(UIColor *)backgroundColor forState:(UIControlState)state;

- (UIColor *)backgroundColorForState:(UIControlState)state;

@end

```

```

@implementation WMButton
{
    NSMutableDictionary *_colors;
}

- (instancetype)initWithFrame:(CGRect)frame
{
    if (self = [super initWithFrame:frame]) {
        _colors = [NSMutableDictionary dictionary];
    }
    return self;
}

- (void)setBackgroundColor:(UIColor *)backgroundColor forState:(UIControlState)state
{
    // If it is normal then set the standard background here
    if(state == UIControlStateNormal)
    {
        [super setBackgroundColor:backgroundColor];
    }
    
    // Store the background colour for that state
    [_colors setValue:backgroundColor forKey:[self keyForState:state]];
}

- (UIColor *)backgroundColorForState:(UIControlState)state
{
    return [_colors valueForKey:[self keyForState:state]];
}


- (void)setHighlighted:(BOOL)highlighted
{
    // Do original Highlight
    [super setHighlighted:highlighted];
    
    // Highlight with new colour OR replace with orignial
    NSString *highlightedKey = [self keyForState:UIControlStateHighlighted];
    UIColor *highlightedColor = [_colors valueForKey:highlightedKey];
    if (highlighted && highlightedColor) {
        [super setBackgroundColor:highlightedColor];
    } else {
        // 由于系统在调用setSelected后，会再触发一次setHighlighted，故做如下处理，否则，背景色会被最后一次的覆盖掉。
        if ([self isSelected]) {
            NSString *selectedKey = [self keyForState:UIControlStateSelected];
            UIColor *selectedColor = [_colors valueForKey:selectedKey];
            [super setBackgroundColor:selectedColor];
        } else {
            NSString *normalKey = [self keyForState:UIControlStateNormal];
            [super setBackgroundColor:[_colors valueForKey:normalKey]];
        }
    }
}

- (void)setSelected:(BOOL)selected
{
    // Do original Selected
    [super setSelected:selected];
    
    // Select with new colour OR replace with orignial
    NSString *selectedKey = [self keyForState:UIControlStateSelected];
    UIColor *selectedColor = [_colors valueForKey:selectedKey];
    if (selected && selectedColor) {
        [super setBackgroundColor:selectedColor];
    } else {
        NSString *normalKey = [self keyForState:UIControlStateNormal];
        [super setBackgroundColor:[_colors valueForKey:normalKey]];
    }
}

- (NSString *)keyForState:(UIControlState)state
{
    return [NSString stringWithFormat:@"state_%d", state];
}

@end

```


使用时，如下调用即可：

```
[button setBackgroundColor:[UIColor clearColor] forState:UIControlStateNormal];
[button setBackgroundColor:[UIColor clearColor] forState:UIControlStateHighlighted];
[button setBackgroundColor:HotPinkColor forState:UIControlStateSelected];
        
```


需要注意的一点是，有些人可能会这样使用

```
[button setBackgroundColor:HotPinkColor forState:UIControlStateHighlighted | UIControlStateSelected];

```

这种用法是不被支持的，虽然可以实现，但是，其实对于iOS默认提供方法`- (void)setBackgroundImage:(UIImage *)image forState:(UIControlState)state UI_APPEARANCE_SELECTOR; // default is nil`这种位于的用法也是不被完全支持的，大家可以试试看。


下面我们再来详细讨论下`UIButton`切换state的顺序问题。

当`UIButton`被按下时，会启动一个计时器，每隔一段时间，都会去检测按钮是否还处在被按下的状态。如果系统检测到它还处于被按下的状态，则就会切换到`UIControlStateHighlighted`，否则，恢复到`UIControlStateNormal`。当你长按后，在当前按钮的区域抬起手时，会切换到`UIControlStateSelected`，但是，需要注意的是，这次切换不仅会触发`setSelected:`被调用，也会触发`setHighlighted:`的一次调用。


```
2015-01-08 19:43:58.782  V |: highlighted = 1 @ WMButton(#38).-[WMButton setHighlighted:] 

2015-01-08 19:44:00.227  V |: highlighted = 1 @ WMButton(#38).-[WMButton setHighlighted:] 

2015-01-08 19:44:00.277  V |: highlighted = 1 @ WMButton(#38).-[WMButton setHighlighted:] 

2015-01-08 19:44:00.294  V |: highlighted = 1 @ WMButton(#38).-[WMButton setHighlighted:] 

2015-01-08 19:44:00.327  V |: highlighted = 1 @ WMButton(#38).-[WMButton setHighlighted:] 

2015-01-08 19:44:00.344  V |: highlighted = 1 @ WMButton(#38).-[WMButton setHighlighted:] 

2015-01-08 19:44:00.377  V |: highlighted = 1 @ WMButton(#38).-[WMButton setHighlighted:] 

2015-01-08 19:44:00.494  V |: highlighted = 1 @ WMButton(#38).-[WMButton setHighlighted:] 

2015-01-08 19:44:00.528  V |: highlighted = 1 @ WMButton(#38).-[WMButton setHighlighted:] 

2015-01-08 19:44:00.577  V |: highlighted = 1 @ WMButton(#38).-[WMButton setHighlighted:] 

2015-01-08 19:44:00.660  V |: highlighted = 1 @ WMButton(#38).-[WMButton setHighlighted:] 

2015-01-08 19:44:00.777  V |: highlighted = 1 @ WMButton(#38).-[WMButton setHighlighted:] 

2015-01-08 19:44:00.894  V |: highlighted = 1 @ WMButton(#38).-[WMButton setHighlighted:] 

2015-01-08 19:44:00.927  V |: highlighted = 1 @ WMButton(#38).-[WMButton setHighlighted:] 

2015-01-08 19:44:00.944  V |: highlighted = 1 @ WMButton(#38).-[WMButton setHighlighted:] 

2015-01-08 19:44:01.127  V |: highlighted = 1 @ WMButton(#38).-[WMButton setHighlighted:] 

2015-01-08 19:44:01.661  V |: highlighted = 1 @ WMButton(#38).-[WMButton setHighlighted:] 

2015-01-08 19:44:01.696  V |: highlighted = 1 @ WMButton(#38).-[WMButton setHighlighted:] 

2015-01-08 19:44:01.762  V |: highlighted = 1 @ WMButton(#38).-[WMButton setHighlighted:] 

2015-01-08 19:44:01.795  V |: highlighted = 1 @ WMButton(#38).-[WMButton setHighlighted:] 

2015-01-08 19:44:01.862  V |: highlighted = 1 @ WMButton(#38).-[WMButton setHighlighted:] 

2015-01-08 19:44:01.895  V |: highlighted = 1 @ WMButton(#38).-[WMButton setHighlighted:] 

2015-01-08 19:44:01.930  V |: highlighted = 1 @ WMButton(#38).-[WMButton setHighlighted:] 

2015-01-08 19:44:03.961  V |: highlighted = 1 @ WMButton(#38).-[WMButton setHighlighted:] 

2015-01-08 19:44:05.162  V |: highlighted = 1 @ WMButton(#38).-[WMButton setHighlighted:] 

2015-01-08 19:44:05.328  V |: highlighted = 1 @ WMButton(#38).-[WMButton setHighlighted:] 

2015-01-08 19:44:05.346  V |: selected = 1 @ WMButton(#56).-[WMButton setSelected:] 

2015-01-08 19:44:05.347  V |: highlighted = 0 @ WMButton(#38).-[WMButton setHighlighted:]
```

