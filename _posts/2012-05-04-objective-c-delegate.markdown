---
layout: post
title: objective-c delegate的用法
date: 2012-05-04 00:16
comments: true
categories: [objective-c]
---
首先声明一个protocol

SampleDelegate.h
{% highlight objective-c %}
@protocol SampleDelegate<NSObject>
  - (void)delegateFunc:(id)sender;
@end
{% endhighlight %}

ClassB
ClassB.h
{% highlight objective-c %}
@interface ClassB : NSObject {
    id<SampleDelegate> delegate;
}
@property (nonatomic,assign) id<SampleDelegate> delegate;
@end
{% endhighlight %}

ClassB.m
{% highlight objective-c %}
@implementation ClassB
@synthesize delegate;
//省略初始化等函数
- (void)test
{
    if ([self.delegate respondsToSelector:@selector(delegateFunc:)]) {
        [self.delegate delegateFunc:self];
    }
}

@end
{% endhighlight %}

ClassA.h
{% highlight objective-c %}
@interface ClassA : NSObject <SampleDelegate>

@end
{% endhighlight %}

ClassA.h
{% highlight objective-c %}
@implementation ClassA
- (void)setDelegate
{
    ClassB *b = [[ClassB alloc] init];
	b.delegate = self;
}

- (void)delegateFunc
{
    //do something
}
@end
{% endhighlight %}

### delegate method的命名规则
以小写的委托方object开头，比如application，window，view等等，省略NS等接头词，然后加上will，should，did，has等助动词

### 总结，疑问
一路下来感觉delegate就是在ClassA里生成ClassB的instance b，然后ClassA把自己的instance的指针传给b，在b里面就可以callback回ClassA里的instance method，本质上应该还是callback，不知道自己的理解是否正确

不知道除了这个还有什么用法
