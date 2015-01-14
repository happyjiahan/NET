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



