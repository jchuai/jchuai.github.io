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
#### 打破引用环解决方案
书中给出了打破引用环的解决方案：扩展NSTimer类型，传递‘块’类型，从而打破对target的引用。Objective-C代码如下(书中代码)

	#import <Foundation/Foundation.h>
	
	@interface NSTimer (EOCBlocksSupport)
	
	+ (NSTimer*)eoc_scheduledTimerWithTimeInterval:
						 	(NSTimeInterval) interval
								block:(void(^)())block
								repeats:(BOOL)repeats;
	@end
	
	@implementation NSTimer (EOCBlocksSupport)
	
	+ (NSTimer*)eoc_scheduledTimerWithTimeInterval:
						 	(NSTimeInterval) interval
								block:(void(^)())block
								repeats:(BOOL)repeats
	{
		return [self scheduledTimerWithTimeInterval: interval
											target:self
					      selector:@selector(eoc_blockInvoke:)
					      userInfo:[block copy]
					      repeats:repeats];
	}
	
	+ (void)eoc_blockInvoke:(NSTimer*)timer {
		void (^block)() = timer.userInfo;
		if (block) {
			block();
		}
	}
	
可以看出，NSTimer的引用对象变成了self；对于实例对象，只是传递了block的copy，因此可以打断整个引用环。
但在用swift实现上述方案时有个问题:无法直接传递closue。从上面OC的代码可以看出，block（closue）是直接赋值给userInfo，来实现传递的。在Swift中，这个函数的userInfo类型为AnyObject？类型，但是Swift中的closue不属于AnyObject类型.

	let closure: () -> Void = { print("Hello1") } // ()->()
	closure is AnyObject //false

因此不可以直接将closure赋值给userInfo属性。我们需要定义一个类，来保存closure。在这里采用Generic泛类方式来实现。代码如下

	class Closure<T> {
    	var value: T
    	init(value: T) {
        	self.value = value
    	}
	}
	
此时可以通过定义Closure类实例，将closure传递给userInfo对象。全部代码如下：

	extension NSTimer {
    	class func eoc_scheduledTimerWithTimeInterval(ti: NSTimeInterval, 
    											closure: () -> Void, 
    											repeats: Bool) -> NSTimer {
        	let userInfo = Closure(value: closure)
        	return self.scheduledTimerWithTimeInterval(ti, 
        									target: self, 
        									selector: "excuteClosure:", 
        									userInfo: userInfo, 
        									repeats: repeats)
    	}
    
    	class func excuteClosure(timer: NSTimer) {
        	if let userInfo = timer.userInfo as? Closure<()->Void> {
            	let closure = userInfo.value
            	closure()
        	}
    	}
	}

	class Closure<T> {
    	var value: T
    	init(value: T) {
        	self.value = value
    	}
	}
	
#### 进一步优化
上面我们已经完成了对于NSTimer类型的封装，因为打破了引用环，在使用的时候，只需要在实例的deinit函数中调用timer.invalide()函数即可，不必再担心由于疏忽忘记调用invalide函数而使得timer一直存在。如果进一步的优化，那么调用invalide()函数这一步也可以省略掉，那就是定义个TimerManager类，在这个类中实现对timer的启动和停止。

	class TimerManager: NSObject {
    	private var _timer: NSTimer?
    
    	init(timeInterval: NSTimeInterval, closure: ()->Void, repeats: Bool) {
    	
        	_timer = NSTimer.eoc_scheduledTimerWithTimeInterval(timeInterval,
        										closure: closure, 
        										repeats: repeats)
    	}
    
    	class func eoc_scheduledTimerWithTimeInterval(timeInterval: NSTimeInterval, 
    											closure: ()->Void, 
    											repeats: Bool) -> TimerManager{
    											
        	return TimerManager(timeInterval: timeInterval, closure: closure, repeats: repeats)
    	}
    
    	deinit{
        	_timer?.invalidate()
        	_timer = nil
    	}
	}

在使用过程中，利用TimerManager代替NSTimer使用，在实例被释放的时候，TimerManager的deinit函数便会被触发，因此不需要在实例中来停止timer，一切交由TimerManger来处理。使用demo如下：

	class Demo: NSObject {
    	var timerManager: TimerManager?
    	func startTimer() {
        	timerManager = TimerManager.eoc_scheduledTimerWithTimeInterval(5.0, 
        								closure: doSomething, 
        								repeats: true)
    	}
    	func doSomething() {
        	// do something
    	}
    
    	func stopTimer() {
        	// 由于TimerManager会来管理Timer，因此不再需要手动关闭NSTimer
        	// timer.invalidate()
        	// timer = nil
    	}
	}
		    