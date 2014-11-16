---
layout: post
title: "[杂记] XCode 6.1 崩溃"
---

今天出现了无解的一幕，有一个 `Swift` 文件，只要一编辑它，XCode 的 SourceKit 就必崩无误。然后代码高亮和提醒就完全嗝屁了。

经过测试，不是`XVim`的原因，删除`Derived Data/`也只能治标不治本。

感觉 `Yosemite`， `iOS 8.X`， `XCode 6.x` 一个个都是坑货。
