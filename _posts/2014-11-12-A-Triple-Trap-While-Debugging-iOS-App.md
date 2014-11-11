---
layout: post
title: A Triple Trap While Debugging iOS App
---

Recently, I got a triple trap while debugging a iOS App.

The problem is quite wired, the release version (installed by AdHoc remotely) of my app crash once opened. But debug version works fine.

At first, I thought it was a code-signing/provision problem, but it was not.

### Launch the release version locally

I switch configuration from `debug` to `release` in product schema, thus I can **launch the release version**.

![ScreenShot1](/assets/images/screenshot-1.jpg)

This dialog can be opened by pressing `⌘ + <`, or selecting from `Product` -> `Schema` -> `Edit Schema...`.

The alternative way is to `Profile` the product with `release` configuration by default.

Then it crashed, so it was not code-signing/provision problem. 

The binary generated with `release` configuration somehow has bugs that crash.

I'v also noticed Xcode complains several times that app launch time exceeded.

### Profile

`Instruments` is definitely the most handy mobile debug tool. It makes Android debug process like playing house.

I choose the `Time Profiler` template, by which I can see what takes the most of time while app is running.

The report indicated that the initializing of NewRelic `NewRelic.startWithApplicationToken("XXXX")` consumes a quarter of the launch time.

This took the `AppDelegate` a long time before return `true` in method `application(_:didFinishLaunchingWithOptions:)`, so iOS kill the app.

This issue is solved by wrapping NewRelic initializing code with `dispatch_async`, no more kill at launch.

But there still exist two view controllers which crash every time on loading.

### Crash Logs

Crash logs on device can be collected from `Devices` window. All logs will be symbolicated (convert hex memory address to method names and source line numbers) automatically.

After checking the log, I've successfully located one line of `Swift` code ( same code in two view controlelrs ) :

```swift
gDAO.fetchQuestionsForGroupID(self.group!.id as Int, loadMore: loadMore)
```

`self.group` is a instance of `Objective-C` class, `self.group.id` is a `NSNumber` which will never be nil.

It seems the conversion from `NSNumber` to `Int` crashes when swift code is compiled with `release` configuration.

Problem solved by changing that line to :

```swift
gDAO.fetchQuestionsForGroupID(self.group!.id.integerValue, loadMore: loadMore)
```

### In-house Distribution

I've been using a free iOS in-house distribution service for team testing for a long time, but this time, Apps signed by their enterprise certificates failed to start. Only Apps distributed in AdHoc can be opened by our testing team.

### Conclusion

At first glance, the problem I've experienced seems to be very knotty, it was caused by three independent reasons.

But the `release` version of App remains highly trackable and testable, and all the differences between `debug` and `release` are locatable (code-signing, compiler optimization, etc).

> The victory shall be mine.
