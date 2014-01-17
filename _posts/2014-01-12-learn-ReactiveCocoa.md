---
layout: 	post
title:		学习ReactiveCocoa
category:	iOS
tages:		

---

很早就从唐巧等技术大牛的博客那里听说过ReactiveCocoa了，但是一直没有好好学习一下，主要原因就是自己懒，今天刚好在学习[iOS 7 Best Practices; A Weather App Case Study: Part 2/2](http://www.raywenderlich.com/55386/ios-7-best-practices-part-2)时，遇到了RAC, 其中的很多地方自己搞不明白，于是就找了些资料，给自己好好补补课。

先把我找到的几篇博客贴上，大家可以看看，都是好文啊，顺便赞一下各位技术大牛，写的博客真的很棒！

+	[github上的官方文档](https://github.com/ReactiveCocoa/ReactiveCocoa)
+	[nshipster上Mattt Thompson大神的文章](http://nshipster.com/reactivecocoa/)
+	[ReactiveCocoa for a better world](https://github.com/blog/1107-reactivecocoa-for-a-better-world)
+	[Functional Reactive Programming on iOS with ReactiveCocoa](http://www.teehanlax.com/blog/reactivecocoa/)
+	[Getting Started with ReactiveCocoa](http://www.teehanlax.com/blog/getting-started-with-reactivecocoa/)
+	[ReactiveCocoa与Functional Reactive Programming](http://blog.leezhong.com/ios/2013/06/19/frp-reactivecocoa.html)
+	[说说ReactiveCocoa 2](http://blog.leezhong.com/ios/2013/12/27/reactivecocoa-2.html)
+	[Basic MVVM with ReactiveCocoa](http://cocoasamurai.blogspot.com/2013/03/basic-mvvm-with-reactivecocoa.html)
+	[http://twocentstudios.com/blog/2013/04/03/the-making-of-vinylogue/](http://twocentstudios.com/blog/2013/04/03/the-making-of-vinylogue/)

下面，我边写代码，边说说自己对RAC的理解吧。

假设我们需要监听一个TextField的text，当其text变化时，我们就打印出一条Log信息。这是一个很简单的需求，你可能会如下实现，先监听UITextFieldTextDidChangeNotification通知，再打印出textField.text。

{% highlight objc linenos %}
[[NSNotificationCenter defaultCenter] addObserver:self
                     selector:@selector(textDidChanged:)
                         name:UITextFieldTextDidChangeNotification
                       object:nil];
{% endhighlight %}

{% highlight objc linenos %}
- (void)textDidChanged:(NSNotification *)ntf
{
    UITextField *textField = [ntf object];
    NSLog(@"%@", textField.text);
}
{% endhighlight %}

上面的实现很简单，貌似没有什么不妥的，可是，请注意，我们把一个简单的需求的实现放在了两个地方，一个地方去监听通知，一个地方打印Log。当代码很多时，这种一个需求，需要多个地方配合实现的代码会很难读懂。同理，对于objc中非常普遍的Delegate, Notification, KVO都存在同样的问题，所以Block出现了。Block可以使我们在一个地方完成我们想要的事情，不需要在多个地方跳转着去看代码。

那么，既然我们在学习RAC，让我们看看上述的需求，用RAC的方式可以怎么来实现吧！

{% highlight objc linenos %}
[self.textField.rac_textSignal subscribeNext:^(NSString *newText) {
   NSLog(@"text = %@", newText);
}];
{% endhighlight %}

就是这么简单，rac_textSignal是RAC扩展的分类，使我们可以方便的使用系统自带的各种控件。相关的内容可以看看[说说ReactiveCocoa 2](http://blog.leezhong.com/ios/2013/12/27/reactivecocoa-2.html), 讲的很好。

可能有的初学RAC的朋友会如下实现，

{% highlight objc linenos %}
[RACObserve(self.textField, text) subscribeNext:^(NSString *newText) {
    NSLog(@"text = %@", newText);
}];
{% endhighlight %}

但是，好像我们用键盘输入文字时，并没有打出相关的Log啊？这是为什么呢？因为，这样实现只能监听

    self.textField.text = @"xxx";
    self.textField.text = @"aaaa";
    
这样显示对TextField.text进行设值的操作，不能监听通过键盘输入赋值。

大家都知道，UITextField有一个delegate是UITextFieldDelegate，它有如下的代理方法：

	@protocol UITextFieldDelegate <NSObject>

	@optional
	
	- (BOOL)textFieldShouldBeginEditing:(UITextField *)textField;        // return NO to disallow editing.
	- (void)textFieldDidBeginEditing:(UITextField *)textField;           // became first responder
	- (BOOL)textFieldShouldEndEditing:(UITextField *)textField;          // return YES to allow editing to stop and to resign first responder status. NO to disallow the editing session to end
	- (void)textFieldDidEndEditing:(UITextField *)textField;             // may be called if forced even if shouldEndEditing returns NO (e.g. view removed from window) or endEditing:YES called
	
	- (BOOL)textField:(UITextField *)textField shouldChangeCharactersInRange:(NSRange)range replacementString:(NSString *)string;   // return NO to not change text
	
	- (BOOL)textFieldShouldClear:(UITextField *)textField;               // called when clear button pressed. return NO to ignore (no notifications)
	- (BOOL)textFieldShouldReturn:(UITextField *)textField;              // called when 'return' key pressed. return NO to ignore.
	
	@end
	
但是，同时又给出了如下的3个通知：

	UIKIT_EXTERN NSString *const UITextFieldTextDidBeginEditingNotification;
	UIKIT_EXTERN NSString *const UITextFieldTextDidEndEditingNotification;
	UIKIT_EXTERN NSString *const UITextFieldTextDidChangeNotification;

特别是UITextFieldTextDidChangeNotification，没有相关的回调方法可以实现相同的功能，大家有没有思考过为什么呢？这里的两种方式是否也决定了我们上面讨论的RAC的两个监听TextField.text的方式呢？到目前为止我也不确定，等抽空仔细查查文档，再来把这里补上吧。

如果这时，可恶的产品经理又提出了一个新需求，我们只要打印以字母‘j’开头的，其他的不打印。用RAC的过滤Singal就相当简单了，代码如下，不解释：

	[[RACObserve(self.textField, text) filter:^BOOL(NSString *newText) {
	    return [newText hasPrefix:@"j"];
	}] subscribeNext:^(NSString *newText) {
	    NSLog(@"text = %@", newText);
	}];
	
	
我们再来考虑一个需求，就是通常的注册流程中，只有当第一次输入的密码和第二次确认的密码一致时，才会让你点击注册按钮，否则，注册按钮都是不可点的。使用RAC，我们怎么实现这个需求呢？请看代码。

    RAC(self.regButton, enabled) =
    [RACSignal combineLatest:@[self.pwdTextField.rac_textSignal,
                               self.confirmPwdTextField.rac_textSignal]
                      reduce:^id(NSString *pwd, NSString *confirmPwd){
                          return @([pwd isEqualToString:confirmPwd]);
                      }];                      
非常简洁吧！reduce的作用是根据接收到的值，再返回一个新的值，这里是@(YES)和@(NO)，必须是object。

有了RAC，我们对于像UIAlertView这样的Delegate形式的控件，可以使用Block了，我认为这真是太方便了！翠花，给爷上代码！:]

    UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"确认要删除该话题吗？"
                                                    message:nil
                                                   delegate:nil
                                          cancelButtonTitle:@"不删除"
                                          otherButtonTitles:@"删除", nil];
    @weakify(alert);
    [alert.rac_buttonClickedSignal subscribeNext:^(NSNumber *buttonIndex) {
        @strongify(alert);
        NSString *buttonTitle = [alert buttonTitleAtIndex:[buttonIndex integerValue]];
        if ([buttonTitle isEqualToString:@"删除"]) {
            NSLog(@"delete, %@", buttonTitle);
        } else {
            NSLog(@"not delete, %@", buttonTitle);
        }
    }];
    [alert show];
这种实现方式，需要特别注意使用ARC时，对传入Block中的变量的内存管理。
    

或者，也可以这样实现：

    UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"确认要删除该话题吗？"
                                                    message:nil
                                                   delegate:nil
                                          cancelButtonTitle:@"不删除"
                                          otherButtonTitles:@"删除", nil];
    [alert.rac_buttonClickedSignal subscribeNext:^(NSNumber *buttonIndex) {
        switch ([buttonIndex integerValue]) {
            case 0:
            {
                // do something
                break;
            }
                
            case 1:
            {
                // do something
                break;
            }
                
            default:
                break;
        }
    }];
    [alert show];
    

	
未完待续....




