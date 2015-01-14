# UIButton setBackgroundColor:ForState

在使用`UIButton`时，很多时候我们需要一个类似于`- (void)setBackgroundColor:(UIColor *)color forState:(UIControlState)state`这样的方法，来实现在不同的状态下使用不同的`backgroundColor`。遗憾的是，iOS默认并没有实现类似的方法，iOS默认实现的方法有如下几个：
- (void)setBackgroundImage:(UIImage *)image forState:(UIControlState)state UI_APPEARANCE_SELECTOR; // default is nil

- (UIImage *)backgroundImageForState:(UIControlState)state;
