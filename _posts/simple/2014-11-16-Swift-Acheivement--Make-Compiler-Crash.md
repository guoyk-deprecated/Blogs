---
layout: post
title: "[Swift] Achievement: Crash the XCode with a Sigle Line of Code"
---

我只是想试着写一个用来偷懒的扩展。

I just want to write a samll extension.

```swift
import UIKit

extension UIViewController {

  class func instanceWithStoryboard() -> Self {
    let mirr = reflect(self)        // This line crash the whole Xcode
    println("\(mirr.valueType)")
    return self.init()
  }

}
```

![image](/assets/images/swift-crash.png)
