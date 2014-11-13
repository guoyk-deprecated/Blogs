---
layout: post
title: 记一次 iOS Debug 三重坑
---

最近，在Debug的时候，掉进了一个三重坑，许久才爬上来。

问题表象很令人纠结，直接 `⌘ + R` 本地运行的App没有问题，但是一旦 `Archive` ，`AdHoc` 分发了之后，别人装上了一启动就闪退。

刚开始我以为是代码签名/配给文件的原因，后来发现远没有这么简单。

### 本地运行 `Release` 版本

修改 `Schema` ，让 `⌘ + R`能够启动 `Release` 配置下编译的App。

![ScreenShot1](/assets/images/screenshot-1.jpg)

可以通过 `⌘ + <` 快捷键打开这个对话框，或者从菜单中点选 `Product` -> `Schema` -> `Edit Schema...`。

另外一种方式是跑 `Profile`，默认也是 `Release` 配置。

然后就不负众望地崩溃了，感觉不像是代码签名的问题。并且 XCode 给我报了启动超时的错误，让我有种不祥的预感（卧槽是不是我用Swift写了太多静态变量)。

### Time Profile

`Instruments` 绝对是神器，相比之下安卓的调试工具简直是战五渣。

`⌘ + I`后，我选择了 `Time Profiler` 这个模板，这样我可以看到启动的时候，哪个方法消耗的时间最久。

结果显示 `NewRelic` `NewRelic.startWithApplicationToken("XXXX")` 这个方法，吃了四分之一的启动时间。导致 `AppDelegate` 在方法 `application(_:didFinishLaunchingWithOptions:)` 中卡了太久，没有返回 `true`。

然后iOS仁慈地把App杀了。

妥妥地给这个调用括上 `dispatch_async`，哪凉快哪呆着去。

启动崩溃的问题解决了，但是有两个 view controller 还是处于一点就崩溃的状态。

### 崩溃日志

崩溃日志可以在`Devices (⌘ + Shift + 2)`窗口找到，选择一个设备，点击`Crash Logs`，等一下就能看到设备上的崩溃日志。

Xcode 会自动地符号化所有内存地址，这样看日志就不会像看天书一样对着一堆内存偏移发蒙了。所有的地址都会标注上方法名，哪个文件的哪一行。

藉此，我定位了一行出现问题的代码。

```swift
gDAO.fetchQuestionsForGroupID(self.group!.id as Int, loadMore: loadMore)
```

`self.group` 是一个 `Objective-C` 写的类（为了使用JSONModel我也是蛮拼的），`self.group.id` 是一个 `NSNumber`，理论上不会为`nil`。

看样子 `NSNumber` 用 `as`转换成 `Int` (Swift) 在 `Release` 配置下会崩溃，但是 `Debug` 配置不会，估计是开启优化参数之后，行为多少有些不同。

改成下面这样就好了:

```swift
gDAO.fetchQuestionsForGroupID(self.group!.id.integerValue, loadMore: loadMore)
```

### In-house 分发

我常年使用某网站提供的免费企业证书签发服务，然后这次被坑了。用他们的企业证书签发的应用启动就崩溃，但是如果是 AdHoc 的话，就木有问题。

### 结论

`Debug` 没问题 `Release` 就崩溃到不行，乍听上去这问题太难解决了，实则不然。

即便是 `Release` 状态下的 App，苹果依旧提供了很多可以调试的方法，并且 `Debug` 和 `Release` 之间的区别都是可以追踪的，无非是代码签名，编译器优化选项等等，一个一个排除，最终能够解决。

> The victory shall be mine.
