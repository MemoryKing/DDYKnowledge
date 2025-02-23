# Swift8-琐碎知识点

* 三方库一处引入，处处使用
	
	```
	// 可以建立一个文件放进去，如 Const.swift
	@_exported import IQKeyboardManagerSwift
	@_exported import Alamofire
	@_exported import Moya
	@_exported import Kingfisher
	@_exported import SnapKit
	@_exported import SwiftyJSON
	@_exported import ObjectMapper
	```
	
* 几种定时器

1. Timer   

	```
	// 创建方式1 自动加入runloop
	timer = Timer.scheduledTimer(timeInterval: 1, target: self, selector: #selector(timerRun), userInfo: nil, repeats: true)
	
	// 创建方式2 手动加入runloop 
	timer = Timer.init(timeInterval: 1, target: self, selector: #selector(timerRun), userInfo: nil, repeats: true)
 	RunLoop.current.add(timer!, forMode: .default)
 	
 	// 创建方式3 自动加入runloop iOS10后block回调
 	timer = Timer.scheduledTimer(withTimeInterval: 1, repeats: true, block: { (timer: Timer) in
  		weakSelf?.timerRun()
	})
	
 	// 创建方式4 手动加入runloop iOS10后block回调
 	timer = Timer.init(timeInterval: 1, repeats: true, block: { (timer: Timer) in
  		weakSelf?.timerRun()
	})
	RunLoop.current.add(timer!, forMode: .default)
	// 还有其他API，暂时用不到
	```
	
	方式1和方式2带来一个严重的问题--循环引用导致内存泄漏[参考苹果文档](https://developer.apple.com/documentation/foundation/timer)    
	由于内部获取传入的target对象的指针并强引用该对象，再加上runloop对timer的强引用，必然导致循环引用。    
	即使声明局部变量(runloop--target--timer三者关系)，即使再加上用weak形式(weak对象和原对象地址相同)，也改变不了循环引用问题的存在
	方式3和方式4由于采用了block回调的方式，变相的将target编程timer自己，从而阻断了闭环，解决了循环引用问题，但只适用于iOS10及以后系统
	要想兼容性的(主要iOS8-iOS9)解决就要另选方案  
	
	解决方案1
	
	写Timer的扩展，实现类似iOS10以后的block回调形式(内部判断，版本满足iOS10则直接调用系统block形式，不满足则自己转换成block形式)
	
	```
	extension Timer {
	
		public static func ddyScheduledTimer(withTimeInterval interval: TimeInterval, repeats: Bool, block: @escaping (Timer) -> Void) -> Timer {
			if #available(iOS 10, *) {
				return scheduledTimer(withTimeInterval: interval, repeats: repeats, block: { block($0) })
			} else {
            	return scheduledTimer(timeInterval: interval, target: self, selector: #selector(ddyTimerInvoke(_:)), userInfo: block, repeats: repeats)
        	}
    	}

		public class func ddyInit(timeInterval interval: TimeInterval, repeats: Bool, block: @escaping (Timer) -> Void)  -> Timer {
        	if #available(iOS 10, *) {
            	return self.init(timeInterval: interval, repeats: repeats, block: { block($0) })
        	} else {
            	return self.init(timeInterval: interval, target: self, selector: #selector(ddyTimerInvoke(_:)), userInfo: block, repeats: repeats)
        	}
    	}

    	@objc static func ddyTimerInvoke(_ timer: Timer) {
        	if let block = timer.userInfo as? (Timer) -> Void {
            	block(timer)
        	}
   		}
	}
	```
	
	解决方案2
	
	
	
	
	
	
 
2. dispatchSourceTimer  

	```
	
	```

3. CADisplayLink  

	```
	
	```
	
* 结构体(struct) 和 类(class) 的区别

	swift中     

	- class是引用类型，struct是值类型。    
		值类型在传递和赋值时将进行复制，而引用类型则只会使用引用对象的一个"指向"。所以他们两者之间的区别就是两个类型的区别。
	- class 可以继承，这样子类可以使用父类的特性和方法
	- class 可以被多次引用
	- struct 更轻量级，适用于复制操作，相比于一个class的实例被多次引用更加安全。
无须担心内存memory leak或者多线程冲突问题


	C语言中    
	
	- struct只是作为一种复杂数据类型定义，不能用于面向对象编程。
	
	C++中
	
	- 对于成员访问权限以及继承方式，class默认private的，而struct是public的。
	- class还可以用于表示模板类型，struct则不行

* 2B程序员 普通程序员 大神程序员

	交换数组中索引下标m和n处两个数的位置(m,n均不会越界)
	
	```
	// 2B程序员
	func swap1(_ numberArray: inout Array<Int>, _ m: Int, _ n: Int) {
		let temp = numberArray[m]
		numberArray[m] = numberArray[n]
		numberArray[n] = temp
	}
	// 普通程序员
	func swap2<T>(_ array: inout [T], _ m: Int, _ n: Int) {
		let temp = array[m]
		array[m] = array[n]
		array[n] = temp
	}
	// 大神程序员
	func swap3<T>(_ nums: inout [T], _ m: Int, _ n: Int) {
		(nums[m], nums[n]) = (nums[n], nums[m])
	}	
	
	// 调用
	var numberArray = [1, 2, 3, 4, 5]
   	swap1(&numberArray, 1, 2)
	// swap2(&numberArray, 1, 2)
	// swap3(&numberArray, 1, 2)
	```
	
	输入一个整数，输出其加10后的数字
	
	```
	// 2B程序员
	func add1(_ number: Int) -> Int {
		return number + 10
	}
	// 普通程序员
	func add2(_ number: Int, add number2: Int) -> Int {
		return number + number2
	}
	// 大神程序员
	func add3(_ number: Int) ->(Int) -> Int {
		return { number2 in
			return number + number2
		}
	}
	
	// 调用
	let num1 = add1(2)
	let num2 = add2(2, add: 10)
	let num3 = add3(2)(10)
	print("\(num1) \(num2) \(num3)")
	```

	敲黑板，划重点
	
	1. inout关键字
		swift有两种参数传递方式    
		值传递: 传递的参数是副本，调用参数过程不影响原始数据     
		指针传递: 把参数内存地址传递过去，调用过程会影响原始数据     
		
		class默认指针传递，Int,Float,Bool,Character,Array,Set,Dictionary,enum,struct默认值传递;       
		想将值传递参数变成引用方式指针传递,用inout关键字实现。
		
		注    
		函数声明中参数用inout修饰以达到函数内部改变外部传入的参数，在调用时变量前加 & 符号;          
		参数被inout修饰，就不用var和let修饰了。
	2. 范型       
		当函数只是传入参数类型不同，其他都相同，此时不用具体类型，而用范型    
	3. 元组   
		元组能简洁的交换两个变量值   
	4. Currying(柯里化)    
		通过局部套用(部分求值)实现多参变单参的函数式思想     
		其特点为:只用表达式(单纯运算过程，总是有返回值)，不用语句(执行某种操作，没有返回值)，不修改值，只返回新值。   
		  
		优点：    
		- 代码简洁    
		- 提高代码复用性    
		- 代码管理方便，相互之间不依赖，每个函数都 是一个独立的模块，很容易进行单元测试。   
		- 易于“并发编程”,因为不修改变量的值，都是返回新值。   
		- 最大的好处就是能把函数当参数用!!  
	[参考](https://www.jianshu.com/p/fc8c13ce7157) 
		