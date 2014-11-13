---
layout: post
title: 简单易用的ACKeyboardObserver
---

这是一个非常小的类，主要目的是把键盘相关的6个通知，转换为Delegate方式。
同时自带一个函数，用于快速实现键盘相关动画。

## 安装

使用 `CocoaPods`:

```ruby
pod 'ACKeyboardObserver', '~> 0.1'
```

手工:

[https://github.com/yanke-guo/ACKeyboardObserver](https://github.com/yanke-guo/ACKeyboardObserver)

## 食用方法

- 在 ViewController 中实现 `ACKeyboardObserverDelegate`

```objective-c
- (void)keyboardWillEmitEvent:(ACKeyboardEvent)event withChange:(ACKeyboardChange)change;
- (void)keyboardDidEmitEvent:(ACKeyboardEvent)event withChange:(ACKeyboardChange)change;
```

- 初始化一个对象

```objective-c
self.keyboardObserver = [ACKeyboardObserver observerWithDelegate:self];
[self.keyboardObserver start];
```

- 使用 `ACKeyboardFastAnimate` 函数来快速实现键盘相关动画