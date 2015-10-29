---
layout: post
title: "第52条：别忘了NSTimer会保留其目标对象 (Swift)"
date: 2015-10-29 19:46:10 +0800
comments: true
categories: iOS
---
在《Effective Objective-C: 编写高质量iOS与OS X代码的52个有效方法》一书中，第52条指出了NSTimer因为会保留其目标对象从而引入引用环的事实，并且给出了Objective—C版的解决方案来避免引用环。本文在这篇文章基础上给出了swift的实现方式，并且设计了一个新类简化了NSTimer的使用方式。
<!-- more -->
#### NSTimer问题简述
我们在使用NSTimer做计时器的时候通常的使用方式如下

	var timer: NSTimer
	func startTimer() {
		timer = NSTimer.scheduledTimerWithTimeInterval(5.0,
								 target: self, 
								 selector: "doSomething",
								 userInfo: nil, 
								 repeats: true)
	}
	
	func doSomething() {
		// do something
	}
	
	func stopTimer() {
		timer.invalidate()
		timer = nil
	}
	
这段代码的问题就在于timer通常是用实例变量存放的，因此实例保留了timer；但同时scheduledTimerWithTimeInterval函数中指明了target为self，因此timer也保留了实例，导致引用环的产生。`这时，实例必须在某个时间调用stopTimer()来停掉Timer，否则引用环将会导致内存泄露`。注意，即使在deinit中释放掉timer也是无济于事的。在引用环的被打破之前，deinit函数将不会被调用。