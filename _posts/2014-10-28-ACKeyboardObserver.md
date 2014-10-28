---
layout: post
title: Introduction to ACKeyboardObserver
categories: ios
---

A small class which turns 6 keyboard notifications into a delegate with two methods, extracting values from `NSNotification`,  and providing an easy-to-use function for making a keyboard related animation.

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

### ACKeyboardObserver.h

```objective-c
//
//  ACKeyboardObserver.h
//
//  Created by YANKE Guo on 12/12/13.
//

#import <Foundation/Foundation.h>
#import <UIKit/UIKit.h>

#pragma mark - Basic Struct

typedef struct {
  CGRect                  beginRect;
  CGRect                  endRect;
  NSTimeInterval          duration;
  UIViewAnimationCurve    animationCurve;
} ACKeyboardChange;

typedef NS_ENUM(NSInteger, ACKeyboardEvent) {
  ACKeyboardEventShow,
  ACKeyboardEventHide,
  ACKeyboardEventChangeFrame
};

#pragma mark - Fast keyboard animation

typedef void (^ACKeyboardAnimationBlock)(CGRect beginRect,CGRect endRect);

//  convert UIViewAnimationCurve to UIViewAnimationOptions
UIViewAnimationOptions IGAnimationOptionsWithCurve(UIViewAnimationCurve curve);

//  convient method to make a animation attached with a ACKeyboardChange
void ACKeyboardFastAnimate(ACKeyboardChange change,ACKeyboardAnimationBlock block);

#pragma mark - Delegate Protocol

@class ACKeyboardObserver;

@protocol ACKeyboardObserverDelegate <NSObject>

@optional

- (void)keyboardWillEmitEvent:(ACKeyboardEvent)event withChange:(ACKeyboardChange)change;
- (void)keyboardDidEmitEvent:(ACKeyboardEvent)event withChange:(ACKeyboardChange)change;

@end

@interface ACKeyboardObserver : NSObject

@property (nonatomic,assign,readonly) id<ACKeyboardObserverDelegate> delegate;
@property (nonatomic,assign,readonly,getter = isObserving) BOOL observing;

+ (instancetype)observerWithDelegate:(id<ACKeyboardObserverDelegate>)delegate;

- (id)initWithDelegate:(id<ACKeyboardObserverDelegate>)delegate;

- (void)start;

- (void)stop;

@end
```

### ACKeyboardObserver.m

```objective-c
//
//  ACKeyboardObserver.m
//
//  Created by YANKE Guo on 12/12/13.
//

#import "ACKeyboardObserver.h"

#pragma mark - Fast keyboard animation

void ACKeyboardFastAnimate(ACKeyboardChange change,ACKeyboardAnimationBlock block)
{
  [UIView animateWithDuration:change.duration
                        delay:0
                      options:IGAnimationOptionsWithCurve(change.animationCurve)
                   animations:^{
                     block(change.beginRect,change.endRect);
                   } completion:nil];
}

UIViewAnimationOptions IGAnimationOptionsWithCurve(UIViewAnimationCurve curve)
{
  switch (curve) {
    case UIViewAnimationCurveEaseInOut:
      return UIViewAnimationOptionCurveEaseInOut;
    case UIViewAnimationCurveEaseIn:
      return UIViewAnimationOptionCurveEaseIn;
    case UIViewAnimationCurveEaseOut:
      return UIViewAnimationOptionCurveEaseOut;
    case UIViewAnimationCurveLinear:
    default:
      return UIViewAnimationOptionCurveLinear;
  }
}

ACKeyboardChange ACKeyboardChangeFromDictionary(NSDictionary * dict)
{
  return (ACKeyboardChange) {
    .beginRect = [[dict valueForKey:UIKeyboardFrameBeginUserInfoKey] CGRectValue],
    .endRect = [[dict valueForKey:UIKeyboardFrameEndUserInfoKey] CGRectValue],
    .duration = [[dict valueForKey:UIKeyboardAnimationDurationUserInfoKey] doubleValue],
    .animationCurve = (UIViewAnimationCurve)[[dict valueForKey:UIKeyboardAnimationCurveUserInfoKey] integerValue]
  };
}

#pragma mark - Class

@implementation ACKeyboardObserver

- (id)initWithDelegate:(id<ACKeyboardObserverDelegate>)delegate
{
  if (self = [super init]) {
    _delegate = delegate;
  }
  return self;
}

+ (instancetype)observerWithDelegate:(id<ACKeyboardObserverDelegate>)delegate
{
  return [[ACKeyboardObserver alloc] initWithDelegate:delegate];
}

- (void)dealloc
{
  [[NSNotificationCenter defaultCenter] removeObserver:self];
}

- (void)start
{
  _observing = YES;
  [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(_keyboardWillDismiss:) name:UIKeyboardWillHideNotification object:nil];
  [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(_keyboardWillChangeFrame:) name:UIKeyboardWillChangeFrameNotification object:nil];
  [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(_keyboardWillShow:) name:UIKeyboardWillShowNotification object:nil];
  [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(_keyboardDidChangeFrame:) name:UIKeyboardDidChangeFrameNotification object:nil];
  [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(_keyboardDidShow:) name:UIKeyboardDidShowNotification object:nil];
  [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(_keyboardDidDismiss:) name:UIKeyboardDidHideNotification object:nil];
}

- (void)_keyboardWillDismiss:(NSNotification*)notification
{
  [self _emitWillEvent:ACKeyboardEventHide withChange:ACKeyboardChangeFromDictionary(notification.userInfo)];
}

- (void)_keyboardWillShow:(NSNotification*)notification
{
  [self _emitWillEvent:ACKeyboardEventShow withChange:ACKeyboardChangeFromDictionary(notification.userInfo)];
}

- (void)_keyboardWillChangeFrame:(NSNotification*)notification
{
  [self _emitWillEvent:ACKeyboardEventChangeFrame withChange:ACKeyboardChangeFromDictionary(notification.userInfo)];
}

- (void)_keyboardDidDismiss:(NSNotification*)notification
{
  [self _emitDidEvent:ACKeyboardEventHide withChange:ACKeyboardChangeFromDictionary(notification.userInfo)];
}

- (void)_keyboardDidShow:(NSNotification*)notification
{
  [self _emitDidEvent:ACKeyboardEventShow withChange:ACKeyboardChangeFromDictionary(notification.userInfo)];
}

- (void)_keyboardDidChangeFrame:(NSNotification*)notification
{
  [self _emitDidEvent:ACKeyboardEventChangeFrame withChange:ACKeyboardChangeFromDictionary(notification.userInfo)];
}

- (void)_emitWillEvent:(ACKeyboardEvent)event withChange:(ACKeyboardChange)change
{
  if (_delegate && [_delegate respondsToSelector:@selector(keyboardWillEmitEvent:withChange:)]) {
    [_delegate keyboardWillEmitEvent:event withChange:change];
  }
}

- (void)_emitDidEvent:(ACKeyboardEvent)event withChange:(ACKeyboardChange)change
{
  if (_delegate && [_delegate respondsToSelector:@selector(keyboardDidEmitEvent:withChange:)]) {
    [_delegate keyboardDidEmitEvent:event withChange:change];
  }
}

- (void)stop
{
  [[NSNotificationCenter defaultCenter] removeObserver:self];
  _observing = NO;
}

@end
```
