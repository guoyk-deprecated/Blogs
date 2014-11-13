---
layout: post
title: iOS 后台播放音频
---
在开发同人音声播放器 `SheepPlayer` 中，遇到了一些 iOS 应用后台的一些坑。

iOS 的后台策略众所周知的非常严格，如果想要在后台播放多个独立的音频项目，比如说锁屏状态下，建议使用`AVQueuePlayer`，而不是`AVAudioPlayer`。

苹果说，要有光（泥奏凯）。苹果说，即便是开启了 `Background Mode` 的 `audio` 选项，一旦音频播放器停止播放，应用就会立即暂停运行。

如果是 `AVAudioPlayer` 的话，一旦播放完成，应用就立即被暂停，没有再次启动一个 `AVAudioPlayer` 播放下一个音频的机会。

但是，如果使用 `AVQueuePlayer` ，直到所有项目播放完成，应用才会被休眠。

> PS：

> 如果要在锁屏状态下播放音乐，被播放的文件必须设置 `NSFileProtectionKey` 为 `NSFileProtectionNone`。否则会被iOS的文件加密机制阻拦，出现找不到文件错误。

> PPS:

> `Documents/Inbox` 文件夹内的文件（就是从其他应用打开的文件），只能读取或者删除，是没法修改的，包括修改文件属性。一般情况下，大家都会把文件拷出来，再做处理。
