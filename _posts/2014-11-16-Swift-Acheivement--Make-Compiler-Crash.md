---
layout: post
title: "☑️ [Swift 成就] 写出让编译器崩溃的代码"
---

我只是想试着写一个用来偷懒的扩展。

```swift
import UIKit

extension UIViewController {

  class func instanceWithStoryboard() -> Self {
    let mirr = reflect(self)        // 这一行就让编译器Crash了
    println("\(mirr.valueType)")
    return self.init()
  }

}
```

![image](/assets/images/swift-crash.png)
