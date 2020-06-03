# 简答题

**引用计数的工作原理与优缺点**
	
	原理：为对象设置一个计数器，每引用一次释放一次引用计数器会主动对引用次数进行更改，当计数器为0时即可立即释放
	优点：（1）当发现垃圾时即时回收（2）最大限度的减少程序暂停
	缺点：（1）无法回收循环引用的对象（2）因为引用计数器要时刻监控计数器，时间开销比较大


**标记整理的工作流程**

	（1）递归查找所有可达对象并进行标记，不可达对象不标记
	（2）移动所有可达对象，且按照内存地址次序依次排列，然后将末端内存地址以后的内存全部回收完成释放内存空间
	 优点：相比引用计数，可以解决循环引用对象的回收问题
	 缺点：不是即时回收，回收时程序会暂停执行
 
**描述V8引擎中新生代存储区垃圾回收的机制**

	新生代中用 Scavenge 算法来处理。所谓 Scavenge 算法，是把新生代空间对半划分为两个区域，一半是对象区域，一半是空闲区域，
	新生区要划分为对象区域和空闲区域，新加入的对象都会存放到对象区域，当对象区域快被写满时，就需要执行一次垃圾清理操作。
	在垃圾回收过程中，首先要对对象区域中的垃圾做标记；标记完成之后，就进入垃圾清理阶段，副垃圾回收器会把这些存活的对象复制到空闲区域中，
	同时它还会把这些对象有序地排列起来，所以这个复制过程，也就相对于完成了内存清理操作，复制后空闲区域就没有内存碎片了。
	完成复制后，对象区域与空闲区域进行角色翻转，也就是原来的对象区域变成空闲区域，原来的空闲区域变成了对象区域。
	这样就完成了垃圾对象的回收操作，同时这种角色翻转的操作还能让新生代中的这两块区域无限重复使用下去。
	由于新生代中采用的 Scavenge 算法，所以每次执行清理操作时，都需要将存活的对象从对象区域复制到空闲区域。
	但复制操作需要时间成本，如果新生区空间设置的太大了，那么每次清理的时间就会过久，所以为了执行效率，一般新生区的空间会被设置的比较小。
	也正是因为新生区的空间不大，所以很容易被存活的对象装满整个区域。为了解决这个问题，JavaScript 引擎采用了对象晋升策略，
	也就是经过两次垃圾回收依然还存活的对象，会被移动到老生区中。

**描述增量标记算法在何时使用及工作原理**

		为了解决标记清除的长停顿问题,把堆栈分为多个域，每次仅从一个域收集垃圾，从而造成较小的应用程序中断
		可以让应用程序线程和垃圾回收线程交替使用,即一次收集一部分垃圾,接着切到实际应用程序,直到垃圾收集完成为止,使用的还是复制算法和标记清除算法的结合

# 代码题1
###练习1
// 使用函数组合fp.flowRight()重新实现下面这个函数

	let isLastInStock = function(cars) {
		let last_car = fp.last(cars)
		return fp.prop('in_stock',last_car)
	}
修改后

	// 先获取最后一条数据，然后获取此数据的in_stock属性
	let isLastInStockB = fp.flowRight(fp.prop('in_stock'),fp.last)
	let res = isLastInStockB(cars)
	console.log(res) //false

###练习2
// 使用fp.flowRight()、fp.prop()和fp.first()获取第一个car的name

	let isLastInStockC = fp.flowRight(fp.prop('name'),fp.first)
	let res = isLastInStockC(cars)
	console.log(res) //Ferrari FF

###练习3
// 使用帮助函数_average重构averageDollarValue,使用函数组合的方式实现

	let _average = function(xs) {
		return fp.reduce(fp.add,0,xs)/xs.length
	}
	let averageDollarValue = function(cars){
		let dollar_values = fp.map(function(car){
			return car.dollar_value
		},cars)
		return _average(dollar_values)
	}
	
修改后：
 
	let xx = fp.flowRight(_average,fp.map(i=>i.dollar_value))
	console.log(xx(cars)) //790700
	

###练习4
// 使用flowRight写一个sanitizeNames()函数，返回一个下划线连接的小写字符串，把数组中的name转换为这种形式：例如：sanitizeNames(["Hello World])=>["hello_world]

	使用函数方式
	function sanitizeNames(str){
		return fp.flowRight(_underscore, fp.toLower, fp.first)(str)
	}
	使用匿名函数方式
	let sanitizeNames = fp.flowRight(_underscore, fp.toLower,log, fp.first)
	console.log(sanitizeNames(['Hello World'])) //hello_world

# 代码题2
###练习1
// 使用fp.add(x,y)和fp.map(f,x)创建一个能让functor里的值增加的函数ex1

	let ex1 = maybe.map(i=>{
		let x = 0
		fp.map((value) => {
			x = fp.add(x,value)
		},i)
		return x

	})
	console.log(ex1) //MayBe { _value: 12 }

###练习2
// 实现一个函数ex2,能够使用fp.first获取列表的第一个元素

	let xs = Container.of(['do','ray','me','fa','so','la','ti','do'])
	console.log(xs)
	let ex2 =xs.map(i=>{
		return fp.first(i)
	})
	console.log(ex2) // Container { _value: 'do' }

###练习3
// 练习3 实现一个函数ex3，使用safeProp和fp.first找到user的名字的首字母

	let safeProp = fp.curry(function(x,o){
		return MayBe.of(o[x])
	})
	let user = {
		id: 2,
		name: 'Albert'
	}

	let cenVar = safeProp('name', user)
	let ex3 = cenVar.map(i=>{
		return fp.first(i)
	})
	console.log(ex3,'ex3') //MayBe { _value: 'A' }

###练习4
// 使用Maybe重写ex4，不要有if语句

	let ex4 = function(n) {
		if (n) {
			return parseInt(n)
		}
	}

	let ex5 = MayBe.of(4.5)
	.map(i=>{
		return parseInt(i) 
	})
	console.log(ex5) //MayBe { _value: 4 }

	let ex6 = MayBe.of(null)
	.map(i=>{
		return parseInt(i) 
	})
	console.log(ex6) //MayBe { _value: null }

