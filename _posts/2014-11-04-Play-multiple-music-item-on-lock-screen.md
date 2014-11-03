---
layout: post
title: Play multiple music items in background
---

iOS has a strict background mechanism.

If you want to play multiple music item in background, i.e. with lock screen on.

It is suggested to use a `AVQueuePlayer` instead of `AVAudioPlayer`

Apple says, once player stoped playing, even with background mode set to `audio`, the app suspend immediatelly.

That makes no chance to start a new `AVAudioPlayer` after previous `AVAudioPlayer` finished.

With `AVQueuePlayer`, until all items are played, `AVQueuePlayer` declares itself as done.

Also, it's important to set `NSFileProtectionKey` to `NSFileProtectionNone` for files gonna queued.

Especially, `NSFileProtectionKey`of files in `Documents/Inbox`, i.e. files opened from other apps, cannot be modified.

Consider moving them out of that folder. (NOT TESTED)
