---
layout: post
title: Introduction to ACKeyboardObserver
---

A small class which turns 6 keyboard notifications into a delegate with two methods, extracting values from `NSNotification`,  and providing an easy-to-use function for making a keyboard related animation.

## Installation

Via `CocoaPods`:

```ruby
pod 'ACKeyboardObserver', '~> 0.1'
```

Manually:

[https://github.com/yanke-guo/ACKeyboardObserver](https://github.com/yanke-guo/ACKeyboardObserver)

## Usage

- Implement `ACKeyboardObserverDelegate` in view controller.

```objective-c
- (void)keyboardWillEmitEvent:(ACKeyboardEvent)event withChange:(ACKeyboardChange)change;
- (void)keyboardDidEmitEvent:(ACKeyboardEvent)event withChange:(ACKeyboardChange)change;
```

- Initialize a instance.

```objective-c
self.keyboardObserver = [ACKeyboardObserver observerWithDelegate:self];
[self.keyboardObserver start];
```

- Use `ACKeyboardFastAnimate` to create a keyboard-related animation in delegate methods.

## Files

- [ACKeyboardObserver.h](/files/ACKeyboardObserver/ACKeyboardObserver.h)

- [ACKeyboardObserver.m](/files/ACKeyboardObserver/ACKeyboardObserver.m)
