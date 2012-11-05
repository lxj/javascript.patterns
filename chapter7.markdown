<a name="a1"></a>
# 设计模式

在GoF（Gang of Four）的书中提出的设计模式为面向对象的软件设计中遇到的一些普遍问题提供了解决方案。它们已经诞生很久了，而且被证实在很多情况下是很有效的。这正是你需要熟悉它的原因，也是我们要讨论它的原因。

尽管这些设计模式跟语言和具体的实现方式无关，但它们多年来被关注到的方面仍然主要是在强类型静态语言比如C++和Java中的应用。

JavaScript作为一种基于原型的弱类型动态语言，使得有些时候实现某些模式时相当简单，甚至不费吹灰之力。

让我们从第一个例子——单例模式——来看一下在JavaScript中和静态的基于类的语言有什么不同。

## 单例

单例模式的核心思想是让指定的类只存在唯一一个实例。这意味着当你第二次使用相同的类去创建对象的时候，你得到的应该和第一次创建的是同一个对象。

这如何应用到JavaScript中呢？在JavaScript中没有类，只有对象。当你创建一个对象时，事实上根本没有另一个对象和它一样，这个对象其实已经是一个单例。使用对象字面量创建一个简单的对象也是一种单例的例子：

	var obj = {
		myprop: 'my value'
	};
	
在JavaScript中，对象永远不会相等，除非它们是同一个对象，所以既然你创建一个看起来完全一样的对象，它也不会和前面的对象相等：

	var obj2 = {
		myprop: 'my value'
	};
	obj === obj2; // false
	obj == obj2; // false
	
所以你可以说当你每次使用对象字面量创建一个对象的时候就是在创建一个单例，并没有特别的语法迁涉进来。

> 需要注意的是，有的时候当人们在JavaScript中提出“单例”的时候，它们可能是在指第5章讨论过的“模块模式”。

### 使用new

JavaScript没有类，所以一字一句地说单例的定义并没有什么意义。但是JavaScript有使用new、通过构造函数来创建对象的语法，有时候你可能需要这种语法下的一个单例实现。这也就是说当你使用new、通过同一个构造函数来创建多个对象的时候，你应该只是得到同一个对象的不同引用。

> 温馨提示：从一个实用模式的角度来说，下面的讨论并不是那么有用，只是更多地在实践模拟一些语言中关于这个模式的一些问题的解决方案。这些语言主要是（静态强类型的）基于类的语言，在这些语言中，函数并不是“一等公民”。

下面的代码片段展示了期望的结果（假设你忽略了多元宇宙的设想，接受了只有一个宇宙的观点）：

	var uni = new Universe();
	var uni2 = new Universe();
	uni === uni2; // true
	
在这个例子中，uni只在构造函数第一次被调用时创建。第二次（以及后续更多次）调用时，同一个uni对象被返回。这就是为什么uni === uni2的原因——因为它们实际上是同一个对象的两个引用。那么怎么在JavaScript达到这个效果呢？

当对象实例this被创建时，你需要在Universe构造函数中缓存它，以便在第二次调用的时候返回。有几种选择可以达到这种效果：

- 你可以使用一个全局变量来存储实例。不推荐使用这种方法，因为通常我们认为使用全局变量是不好的。而且，任何人都可以改写全局变量的值，甚至可能是无意中改写。所以我们不再讨论这种方案。
- 你也可以将对象实例缓存在构造函数的属性中。在JavaScript中，函数也是对象，所以它们也可以有属性。你可以写一些类似Universe.instance的属性来缓存对象。这是一种漂亮干净的解决方案，不足之处是instance属性仍然是可以被公开访问的，别人写的代码可能修改它，这样就会失去这个实例。
- 你可以将实例包裹在闭包中。这可以保持实例是私有的，不会在构造函数之外被修改，代价是一个额外的闭包。

让我们来看一下第二种和第三种方案的实现示例。

### 将实例放到静态属性中

下面是一个将唯一的实例放入Universe构造函数的一个静态属性中的例子：

	function Universe() {
	
	// do we have an existing instance?
	if (typeof Universe.instance === "object") {
		return Universe.instance;
	}
	
	// proceed as normal
	this.start_time = 0;
	this.bang = "Big";
	
	// cache
	Universe.instance = this;
	
	// implicit return:
	// return this;
	}
	
	// testing
	var uni = new Universe();
	var uni2 = new Universe();
	uni === uni2; // true

如你所见，这是一种直接有效的解决方案，唯一的缺陷是instance是可被公开访问的。一般来说它被其它代码误删改的可能是很小的（起码比全局变量instance要小得多），但是仍然是有可能的。

### 将实例放入闭包中

另一种实现基于类的单例模式的方法是使用一个闭包来保护这个唯一的实例。你可以通过第5章讨论过的“私有静态成员模式”来实现。唯一的秘密就是重写构造函数：

	function Universe() {
	
		// the cached instance
		var instance = this;
		
		// proceed as normal
		this.start_time = 0;
		this.bang = "Big";
		
		// rewrite the constructor
		Universe = function () {
			return instance;
		};
	}
	
	// testing
	var uni = new Universe();
	var uni2 = new Universe();
	uni === uni2; // true
	
第一次调用时，原始的构造函数被调用并且正常返回this。在后续的调用中，被重写的构造函数被调用。被重写怕这个构造函数可以通过闭包访问私有的instance变量并且将它返回。

这个实现实际上也是第4章讨论的自定义函数的又一个例子。如我们讨论过的一样，这种模式的缺点是被重写的函数（在这个例子中就是构造函数Universe()）将丢失那些在初始定义和重新定义之间添加的属性。在这个例子中，任何添加到Universe()的原型上的属性将不会被链接到使用原来的实现创建的实例上。（注：这里的“原来的实现”是指实例是由未被重写的构造函数创建的，而Universe()则是被重写的构造函数。）

下面我们通过一些测试来展示这个问题：

	// adding to the prototype
	Universe.prototype.nothing = true;
	
	var uni = new Universe();
	
	// again adding to the prototype
	// after the initial object is created
	Universe.prototype.everything = true;
	
	var uni2 = new Universe();
	
	Testing:
	// only the original prototype was
	// linked to the objects
	uni.nothing; // true
	uni2.nothing; // true
	uni.everything; // undefined
	uni2.everything; // undefined
	
	// that sounds right:
	uni.constructor.name; // "Universe"
	
	// but that's odd:
	uni.constructor === Universe; // false
	
uni.constructor不再和Universe()相同的原因是uni.constructor仍然是指向原来的构造函数，而不是被重新定义的那个。

如果一定被要求让prototype和constructor的指向像我们期望的那样，可以通过一些调整来做到：

	function Universe() {
	
		// the cached instance
		var instance;
		
		// rewrite the constructor
		Universe = function Universe() {
			return instance;
		};
		
		// carry over the prototype properties
		Universe.prototype = this;
		
		// the instance
		instance = new Universe();
		
		// reset the constructor pointer
		instance.constructor = Universe;
		
		// all the functionality
		instance.start_time = 0;
		instance.bang = "Big";
		
		return instance;
	}
	
现在所有的测试结果都可以像我们期望的那样了：

	// update prototype and create instance
	Universe.prototype.nothing = true; // true
	var uni = new Universe();
	Universe.prototype.everything = true; // true
	var uni2 = new Universe();
	
	// it's the same single instance
	uni === uni2; // true
	
	// all prototype properties work
	// no matter when they were defined
	uni.nothing && uni.everything && uni2.nothing && uni2.everything; // true
	// the normal properties work
	uni.bang; // "Big"
	// the constructor points correctly
	uni.constructor === Universe; // true
	
另一种可选的解决方案是将构造函数和实例包在一个立即执行的函数中。当构造函数第一次被调用的时候，它返回一个对象并且将私有的instance指向它。在后续调用时，构造函数只是简单地返回这个私有变量。在这种新的实现下，前面所有的测试代码也会和期望的一样：

	var Universe;
	
	(function () {
	
		var instance;
		
		Universe = function Universe() {
		
		if (instance) {
			return instance;
		}
		
		instance = this;
		
		// all the functionality
		this.start_time = 0;
		this.bang = "Big";
		
		};
	
	}());