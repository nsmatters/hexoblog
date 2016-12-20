---
title: 运行时——NSInvocation与消息机制
date: 2016-12-19 15:28:19
tags: iOS
---

以前在学习iOS开发时，接触到`NSInvocation`这个类，觉得这个类很强大，但是复杂的设置过程也很让人迷惑（当时不怎么理解）。而且在平时的开发中基本上没有用到过，从此消失在视野里了。在回顾runtime的知识时，发现`NSInvocation`和消息机制有些相像。所以再次认识一下`NSInvocation`。

# NSInvocation

## 用法
``` Objective-c
NSMethodSignature *methodSignature = [self methodSignatureForSelector:@selector(doWork:)];
NSInvocation *invocation = [NSInvocation invocationWithMethodSignature:methodSignature];
invocation.selector = @selector(doWork:);
NSInteger value = 2;
[invocation setArgument:&value atIndex:2];
[invocation invokeWithTarget:self];
```
需要注意的是，已经生成的`NSInvocation`对象的`methodSignature`不能更改，并且创建的时候使用`invocationWithMethodSignature`类方法创建。不能通过`alloc`和`init`的方式创建。生成的对象并不会retain设置的参数，需要手动通过`argumentsRetained`方法retain。

## 使用场景
1. 将`NSInvocation`对象保存起来在将来使用。
用《Head First设计模式》中的命令模式做个例子，实现一个只有开和关按钮的遥控器。
``` Objective-C
//灯
@interface Light : NSObject
- (void)open;
- (void)close;
@end

@interface Light (Command)
- (NSInvocation *)onCommand;
- (NSInvocation *)offCommand;
@end

@implementation Light
- (void)open
{
    NSLog(@"Light opened");
}
- (void)close
{
    NSLog(@"Light closed");
}
@end

@implementation Light (Command)
- (NSInvocation *)onCommand
{
    NSMethodSignature *methodSignature = [self methodSignatureForSelector:@selector(open)];
    NSInvocation *invocation = [NSInvocation invocationWithMethodSignature:methodSignature];
    invocation.target = self;
    invocation.selector = @selector(open);
    [invocation retainArguments];
    return invocation;
}
- (NSInvocation *)offCommand
{
    NSMethodSignature *methodSignature = [self methodSignatureForSelector:@selector(close)];
    NSInvocation *invocation = [NSInvocation invocationWithMethodSignature:methodSignature];
    invocation.target = self;
    invocation.selector = @selector(close);
    [invocation retainArguments];
    return invocation;
}
@end

//遥控器
@interface RemoteControl : NSObject
@property (nonatomic, strong) NSInvocation *onSlot;
@property (nonatomic, strong) NSInvocation *offSlot;
- (void)onButtonWasPressed;
- (void)offButtonWasPressed;
@end
@implementation RemoteControl

- (void)onButtonWasPressed {
    [self.onSlot invoke];
}
- (void)offButtonWasPressed
{
    [self.offSlot invoke];
}
@end

//测试
Light *light = [Light new];
RemoteControl *remoteControl = [RemoteControl new];
remoteControl.onSlot = light.onCommand;
remoteControl.offSlot = light.offCommand;
[remoteControl onButtonWasPressed];
```

2. 和Timer一起使用
``` Objective-C
[NSTimer scheduledTimerWithTimeInterval:1 invocation:invocation repeats:NO];
```
感觉这种场景很少用到。需要注意的是，timer会让invocation去retain它的参数。


# 方法
看到上边的例子提到了`NSMethodSignature`和经常使用的`@selector`函数。所以接下来看看它们表示的内容和作用是什么。

## NSMethodSignature
方法签名的初始化使用的是一个返回值和参数经过编码的数组，编码使用`@encode()`编译指令。`-(void)doWork:(NSInteger)value`的方法签名输出如下：
```
    number of arguments = 3
    frame size = 224
    is special struct return? NO
    return value: -------- -------- -------- --------
        type encoding (v) 'v'
        flags {}
        modifiers {}
        frame {offset = 0, offset adjust = 0, size = 0, size adjust = 0}
        memory {offset = 0, size = 0}
    argument 0: -------- -------- -------- --------
        type encoding (@) '@'
        flags {isObject}
        modifiers {}
        frame {offset = 0, offset adjust = 0, size = 8, size adjust = 0}
        memory {offset = 0, size = 8}
    argument 1: -------- -------- -------- --------
        type encoding (:) ':'
        flags {}
        modifiers {}
        frame {offset = 8, offset adjust = 0, size = 8, size adjust = 0}
        memory {offset = 0, size = 8}
    argument 2: -------- -------- -------- --------
        type encoding (q) 'q'
        flags {isSigned}
        modifiers {}
        frame {offset = 16, offset adjust = 0, size = 8, size adjust = 0}
        memory {offset = 0, size = 8}
```
可以看到返回值的编码是`v`，参数有三个，分别是`self`、`_cmd`、`value`。`self`的编码是`@`，`_cmd`的编码是`:`，`NSInteger`类型的value编码是`q`。

关于类型编码可以看[Type Encodings](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtTypeEncodings.html)

## SEL
通过`@selector()`编译指令可以得到一个类型为`SEL`的变量，

## IMP

## Method


# 消息机制

## objc_msgSend



# Swizzle


