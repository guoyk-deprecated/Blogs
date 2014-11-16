---
layout: post
title: "[转载] 使用 CADisplayLink 对齐自定义drawRect"
---

摘录的一片文章，讲述如何使用 `CADisplayLink` 对齐自定义drawRect以优化性能。

> Orignal Title:  The Holy Grail of iOS Animation Intervals?

> Orignal Author: BEN BOJKO

> Orignal Url:   [http://www.bigspaceship.com/ios-animation-intervals/](http://www.bigspaceship.com/ios-animation-intervals/)

> 'me' in content below means original author.

> Please contact [abuse@yanke.io](mailto:abuse@yanke.io) if you have any complaint.

Over the past few weeks I’ve had the chance to take my first baby steps in iOS and Objective-C land. 

Sticking to the premise of the journey being the reward, my goal has been to learn anything that would break the system. 

Thankfully it’s amazingly fun and simple to create application UI and business logic in iOS. 

Smooth and beautiful transitions are what we’ve all come to expect from anything that graces the glossy touch screens of our magical pocket devices. 

Their integration goes deep into the innards of iOS and they are a breeze to set up. 

But what happens when you want to stray away from the beautiful default? I set out to investigate how I could achieve my own custom interval based animations and here are the results.

When I started programming, I found a lot of joy in experimenting with animations that react to input in real time. 

This would typically result in particle simulations or music visualizations. 

While tweens like CSS transitions interpolate a value from A to B over a fixed set of time, these animations would typically update their values on a frame-by-frame basis. 

Think of a particle moving with a speed of 100 pixels per second in any direction based on forces around it instead of it moving from left to right in two seconds. 

In Flash one could typically resort to the good old onEnterFrame event and in JavaScript requestAnimationFrame is becoming increasingly widely adapted.

While Apple has made tweens in iOS shamefully easy with the Core Animation framework, looking for the requestAnimationFrame of Objective-C became a surprisingly difficult scavenger hunt through the bowels of the Apple documentation. 

What follows is a recap of my quest to find the holy grail of iOS animation intervals, and ultimately, what I found to be the best solution for myself as an iOS greenhorn.

### TIMER BASED INTERVALS

The first approach I tested was using an NSTimer. 

This Objective-C equivalent of JavaScript’s setInterval let’s you send a message to a target in a constant interval. 

In theory, sending a redraw message to a view object in an interval of 1/60s should result in a smooth animation at 60 frames per second.

```objective-c
- (id) init {
	// ...
	NSTimer *timer = [NSTimer timerWithTimeInterval:1.0/60.0 target:self selector:@selector(update:) userInfo:nil repeats:YES];
	[[NSRunLoop mainRunLoop] addTimer:timer forMode:NSDefaultRunLoopMode];
	// ...
}

- (void) update {
	double speed = 10.0;	// px per frame
	[myView moveWithSpeed: speed];
}
```

Once implemented, though, even the simplest of animations suffered from frequent stuttering. 

A common problem that can cause stuttering in animations is that the render time for each frame can vary slightly. 

Even though the test animation was very basic I tried compensating for this effect and changed from a frame-based animation to a time-based animation by incorporating the actual render time of each frame.

```objective-c
- (void) update {
	double currentTime = CACurrentMediaTime();
	double renderTime = currentTime - frameTimestamp;
	frameTimestamp = currentTime;

	double speed = 10.0 * renderTime * 60.0;	// px per 1/60 of a second (≈ 1 frame)
	[myView moveWithSpeed: speed];
}
```

Sidenote: `CACurrentMediaTime()` seems to be the most efficient and precise way to get relative timestamps. 

Beware of using NSDate since it syncs remotely, which can cause occasional delays.

Unfortunately the animation still stuttered away like an old tractor plowing through a barren potato field during dry season. 

Thinking this might have something to do specifically with how NSTimer is implemented, I went out to look for alternatives. 

A widely spread option seemed to be `performSelector:withObject:afterDelay:` instead of having a NSTimer instance handle the interval. 

Unfortunately the result was equally unsatisfying: The animation was simply not smooth and individual steps didn’t seem to occur on regular intervals. 

I even went to the extent of playing around with separating drawing into a different thread, but nothing seemed to help.

### TAPPING INTO SCREEN REFRESHES

Taking a closer look at the actual frame intervals, the render times didn’t seem to fluctuate much and stayed within ±10% of the target. 

Since this was a decent result and the animation speed itself already compensated for any fluctuation I only had one suspicion left: the update interval might not be syncing up with iOS redrawing the screen. 

This could result in an update occasionally being called twice, but only being drawn once or vice versa.

There had to be a missing link that allows Core Animation transitions to run as smooth as they do, so I gave the official documentation another shot.

Outside of the typical introductory articles, Apple also provides framework overviews. Since Core Animation is part of the Quartz Core Framework I simply clicked through every class, regardless of its name and then I struck gold.

As expected, the missing link was to sync up the update interval with actual screen refreshes. 

Thankfully the Quartz Core framework has a very easy to use wrapper called `CADisplayLink`. 

Even though the name doesn’t directly imply it, this class is specifically made for animations where “your data is extremely likely to change in every frame” (OpenGL ES Programming Guide, Drawing to a Frame Buffer Object). It attempts to send a message to a target every time something is redrawn to the screen, so it was exactly what I needed. All I needed to do was replace the timer code with `CADisplayLink`:

```objective-c
- (id) init {
	// ...
	CADisplayLink *displayLink = [CADisplayLink displayLinkWithTarget:self selector:@selector(update:)];
	[displayLink addToRunLoop:[NSRunLoop mainRunLoop] forMode:NSDefaultRunLoopMode];
	// ...
}

- (void) update {
	double currentTime = [displayLink timestamp];
	double renderTime = currentTime - frameTimestamp;
	frameTimestamp = currentTime;

	double speed = 10.0 * renderTime * 60.0;
	[myView moveWithSpeed: speed];
}
```

As you can see we can also use CADisplayLink’s timestamp property instead of CACurrentMediaTime() to get a relative timestamp on each frame. 

The result was an incredibly smooth animation that could keep up with native Core Animation transitions. 

My old tractor just turned into a hovercraft.

### Conclusion

The lesson that I personally learned from this is that with the abundance of tutorials, articles, books and opinions out there it’s still worth checking the raw documentation every once in awhile. 

You might just find a gem that you weren’t aware of before.

Did I find the Holy Grail, though? Probably not. 

I’m only fresh to the game and my Objective-C legs are still shakey, so I would love to hear how you would approach this quest. 

If there are any implications I have overlooked or if you have a different preferred solution, please share your opinions in the comments below.

### 结论 (Conclusion in Chinese)

在需要实现时间相关动画的时候，合理使用 `CADisplayLink` 来取代 `NSTimer`，或者 `performSelector:withObject:afterDelay:`，可以使得动画与系统的界面刷新对齐，达到更好的性能。

同时可以使用 `CADisplayLink` 的 `-timestamp` 方法高性能地获取时间戳，可以进一步优化性能。
