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
	
## 工厂模式

使用工厂模式的目的就是创建对象。它通常被在类或者类的静态方法中实现，目的是：

- 执行在建立相似的对象时进行的一些重复操作
- 让工厂的使用者在编译阶段创建对象时不必知道它的特定类型（类）

第二点在静态的基于类的语言中更重要，因为在（编译阶段）提前不知道类的情况下，创建类的实例是不普通的行为。但在JavaScript中，这部分的实现却是相当容易的事情。

使用工厂方法（或类）创建的对象被设计为从同一个父对象继承；它们是特定的实现一些特殊功能的子类。有些时候这个共同的父对象就是包含工厂方法的同一个类。

我们来看一个示例实现，我们有：

- 一个共同的父构造函数CarMaker。
- CarMaker的一个静态方法叫factory()，用来创建car对象。
- 特定的从CarMaker继承而来的构造函数CarMaker.Compact，CarMaker.SUV，CarMaker.Convertible。它们都被定义为父构造函数的静态属性以便保持全局空间干净，同时在需要的时候我们也知道在哪里找到它们。

我们来看一下已经完成的实现会怎么被使用：

	var corolla = CarMaker.factory('Compact');
	var solstice = CarMaker.factory('Convertible');
	var cherokee = CarMaker.factory('SUV');
	corolla.drive(); // "Vroom, I have 4 doors"
	solstice.drive(); // "Vroom, I have 2 doors"
	cherokee.drive(); // "Vroom, I have 17 doors"
	
这一段：

	var corolla = CarMaker.factory('Compact');
	
可能是工厂模式中最知名的。你有一个方法可以在运行时接受一个表示类型的字符串，然后它创建并返回了一个和请求的类型一样的对象。这里没有使用new的构造函数，也没有看到任何对象字面量，仅仅只有一个函数根据一个字符串指定的类型创建了对象。

这里是一个工厂模式的示例实现，它能让上面的代码片段工作：

	// parent constructor
	function CarMaker() {}
	
	// a method of the parent
	CarMaker.prototype.drive = function () {
		return "Vroom, I have " + this.doors + " doors";
	};
	
	// the static factory method
	CarMaker.factory = function (type) {
		var constr = type,
		newcar;
		
		// error if the constructor doesn't exist
		if (typeof CarMaker[constr] !== "function") {
			throw {
				name: "Error",
				message: constr + " doesn't exist"
			};
		}
		
		// at this point the constructor is known to exist
		// let's have it inherit the parent but only once
		if (typeof CarMaker[constr].prototype.drive !== "function") {
			CarMaker[constr].prototype = new CarMaker();
		}
		// create a new instance
		newcar = new CarMaker[constr]();
		// optionally call some methods and then return...
		return newcar;
	};
	
	// define specific car makers
	CarMaker.Compact = function () {
		this.doors = 4;
	};
	CarMaker.Convertible = function () {
		this.doors = 2;
	};
	CarMaker.SUV = function () {
		this.doors = 24;
	};
	
工厂模式的实现中没有什么是特别困难的。你需要做的仅仅是寻找请求类型的对象的构造函数。在这个例子中，使用了一个简单的名字转换以便映射对象类型和创建对象的构造函数。继承的部分只是一个公共的重复代码片段的示例，它可以被放到工厂方法中而不是被每个构造函数的类型重复。（译注：指通过原型继承的代码可以在factory方法以外执行，而不是放到factory中每调用一次都要执行一次。）

### 内置对象工厂

作为一个“野生的工厂”的例子，我们来看一下内置的全局构造函数Object()。它的行为很像工厂，因为它根据不同的输入创建不同的对象。如果传入一个数字，它会使用Number()构造函数创建一个对象。在传入字符串和布尔值的时候也会发生同样的事情。任何其它的值（包括空值）将会创建一个正常的对象。

下面是这种行为的例子和测试，注意Object调用时可以不用加new：

	var o = new Object(),
		n = new Object(1),
		s = Object('1'),
		b = Object(true);
		
	// test
	o.constructor === Object; // true
	n.constructor === Number; // true
	s.constructor === String; // true
	b.constructor === Boolean; // true

Object()也是一个工厂这一事实可能没有太多实际用处，仅仅是觉得值得作为一个例子提一下，告诉我们工厂模式是随处可见的。

## 迭代器

在迭代器模式中，你有一些含有有序聚合数据的对象。这些数据可能在内部用一种复杂的结构存储着，但是你希望提供一种简单的方法来访问这种结构中的每个元素。数据的使用者不需要知道你是怎样组织你的数据的，他们只需要操作一个个独立的元素。

在迭代器模式中，你的对象需要提供一个next()方法。按顺序调用next()方法必须返回序列中的下一个元素，但是“下一个”在你的特定的数据结构中指什么是由你自己来决定的。

假设你的对象叫agg，你可以通过简单地在循环中调用next()来访问每个数据元素，像这样：

	var element;
	while (element = agg.next()) {
		// do something with the element ...
		console.log(element);
	}
	
在迭代器模式中，聚合对象通常也会提供一个方便的方法hasNext()，这样对象的使用者就可以知道他们已经获取到你数据的最后一个元素。当使用另一种方法——hasNext()——来按顺序访问所有元素时，是像这样的：

	while (agg.hasNext()) {
		// do something with the next element...
		console.log(agg.next());
	}

## 装饰器

在装饰器模式中，一些额外的功能可以在运行时被动态地添加到一个对象中。在静态的基于类的语言中，处理这个问题可能是个挑战，但是在JavaScript中，对象本来就是可变的，所以给一个对象添加额外的功能本身并不是什么问题。

装饰器模式的一个很方便的特性是可以对我们需要的特性进行定制和配置。刚开始时，我们有一个拥有基本功能的对象，然后可以从可用的装饰器中去挑选一些需要用到的去增加这个对象，甚至如果顺序很重要的话，还可以指定增强的顺序。

### 用法

我们来看一下这个模式的示例用法。假设你正在做一个卖东西的web应用，每个新交易是一个新的sale对象。这个对象“知道”交易的价格并且可以通过调用sale.getPrice()方法返回。根据环境的不同，你可以开始用一些额外的功能来装饰这个对象。假设一个场景是这笔交易是发生在加拿大的一个省Québec，在这种情况下，购买者需要付联邦税和Québec省税。根据装饰器模式的用法，你需要指明使用联邦税装饰器和Québec省税装饰器来装饰这个对象。然后你还可以给这个对象装饰一些价格格式的功能。这个场景的使用方式可能是像这样：

	var sale = new Sale(100); // the price is 100 dollars
	sale = sale.decorate('fedtax'); // add federal tax
	sale = sale.decorate('quebec'); // add provincial tax
	sale = sale.decorate('money'); // format like money
	sale.getPrice(); // "$112.88"
	
在另一种场景下，购买者在一个不需要交省税的省，并且你想用加拿大元的格式来显示价格，你可以这样做：

	var sale = new Sale(100); // the price is 100 dollars
	sale = sale.decorate('fedtax'); // add federal tax
	sale = sale.decorate('cdn'); // format using CDN
	sale.getPrice(); // "CDN$ 105.00"
	
如你所见，这是一种在运行时很灵活的方法来添加功能和调整对象。我们来看一下如何来实现这种模式。

### 实现

一种实现装饰器模式的方法是让每个装饰器成为一个拥有应该被重写的方法的对象。每个装饰器实际上是继承自已经被前一个装饰器增强过的对象。装饰器的每个方法都会调用父对象（继承自的对象）的同名方法并取得值，然后做一些额外的处理。

最终的效果就是当你在第一个例子中调用sale.getPrice()时，实际上是在调用money装饰器的方法（图7-1）。但是因为每个装饰器会先调用父对象的方法，money的getPrice()先调用quebec的getPrice()，而它又会去调用fedtax的getPrice()方法，依次类推。这个链会一直走到原始的未经装饰的由Sale()构造函数实现的getPrice()。

![图7-1 装饰器模式的实现](./Figure/chapter7/7-1.jpg)
图7-1 装饰器模式的实现

这个实现以一个构造函数和一个原型方法开始：

	function Sale(price) {
		this.price = price || 100;
	}
	Sale.prototype.getPrice = function () {
		return this.price;
	};
	
装饰器对象将都被作为构造函数的属性实现：

	Sale.decorators = {};
	
我们来看一个装饰器的例子。这是一个对象，实现了一个自定义的getPrice()方法。注意这个方法首先从父对象的方法中取值然后修改这个值：

	Sale.decorators.fedtax = {
		getPrice: function () {
			var price = this.uber.getPrice();
			price += price * 5 / 100;
			return price;
		}
	};
	
使用类似的方法我们可以实现任意多个需要的其它装饰器。他们的实现方式像插件一样来扩展核心的Sale()的功能。他们甚至可以被放到额外的文件中，被第三方的开发者来开发和共享：

	Sale.decorators.quebec = {
		getPrice: function () {
			var price = this.uber.getPrice();
			price += price * 7.5 / 100;
			return price;
		}
	};
	
	Sale.decorators.money = {
		getPrice: function () {
			return "$" + this.uber.getPrice().toFixed(2);
		}
	};
	
	Sale.decorators.cdn = {
		getPrice: function () {
			return "CDN$ " + this.uber.getPrice().toFixed(2);
		}
	};
	
最后我们来看decorate()这个神奇的方法，它把所有上面说的片段都串起来了。记得它是这样被调用的：

	sale = sale.decorate('fedtax');
	
字符串'fedtax'对应在Sale.decorators.fedtax中实现的对象。被装饰过的最新的对象newobj将从现在有的对象（也就是this对象，它要么是原始的对象，要么是经过最后一个装饰器装饰过的对象）中继承。实现这一部分需要用到前面章节中提到的临时构造函数模式。我们也设置一个uber属性给newobj以便子对象可以访问到父对象。然后我们从装饰器中复制所有额外的属性到被装饰的对象newobj中。最后，在我们的例子中，newobj被返回并且成为被更新过的sale对象。

	Sale.prototype.decorate = function (decorator) {
		var F = function () {},
			overrides = this.constructor.decorators[decorator],
			i, newobj;
		F.prototype = this;
		newobj = new F();
		newobj.uber = F.prototype;
		for (i in overrides) {
			if (overrides.hasOwnProperty(i)) {
				newobj[i] = overrides[i];
			}
		}
		return newobj;
	};
	
### 使用列表实现

我们来看另一个明显不同的实现方法，受益于JavaScript的动态特性，它完全不需要使用继承。同时，我们也可以简单地将前一个方面的结果作为参数传给下一个方法，而不需要每一个方法都去调用前一个方法。

这样的实现方法还允许很容易地反装饰（undecorating）或者撤销一个装饰，这仅仅需要从一个装饰器列表中移除一个条目。

用法示例也会明显简单一些，因为我们不需要将decorate()的返回值赋值给对象。在这个实现中，decorate()不对对象做任何事情，它只是简单地将装饰器加入到一个列表中：

	var sale = new Sale(100); // the price is 100 dollars
	sale.decorate('fedtax'); // add federal tax
	sale.decorate('quebec'); // add provincial tax
	sale.decorate('money'); // format like money
	sale.getPrice(); // "$112.88"
	
Sale()构造函数现在有了一个作为自己属性的装饰器列表：

	function Sale(price) {
		this.price = (price > 0) || 100;
		this.decorators_list = [];
	}
	
可用的装饰器仍然被实现为Sale.decorators的属性。注意getPrice()方法现在更简单了，因为它们不需要调用父对象的getPrice()来获取结果，结果已经作为参数传递给它们了：

	Sale.decorators = {};
	
	Sale.decorators.fedtax = {
		getPrice: function (price) {
			return price + price * 5 / 100;
		}
	};
	
	Sale.decorators.quebec = {
		getPrice: function (price) {
			return price + price * 7.5 / 100;
		}
	};
	
	Sale.decorators.money = {
		getPrice: function (price) {
			return "$" + price.toFixed(2);
		}
	};
	
最有趣的部分发生在父对象的decorate()和getPrice()方法上。在前一种实现方式中，decorate()还是多少有些复杂，而getPrice()十分简单。在这种实现方式中事情反过来了：decorate()只需要往列表中添加条目而getPrice()做了所有的工作。这些工作包括遍历现在添加的装饰器的列表，然后调用它们的getPrice()方法，并将结果传递给前一个：

	Sale.prototype.decorate = function (decorator) {
		this.decorators_list.push(decorator);
	};
	
	Sale.prototype.getPrice = function () {
		var price = this.price,
			i,
			max = this.decorators_list.length,
			name;
		for (i = 0; i < max; i += 1) {
			name = this.decorators_list[i];
			price = Sale.decorators[name].getPrice(price);
		}
		return price;
	};
	
装饰器模式的第二种实现方式更简单一些，并且没有引入继承。装饰的方法也会简单。所有的工作都由“同意”被装饰的方法来做。在这个示例实现中，getPrice()是唯一被允许装饰的方法。如果你想有更多可以被装饰的方法，那遍历装饰器列表的工作就需要由每个方法重复去做。但是，这可以很容易地被抽象到一个辅助方法中，给它传一个方法然后使这个方法“可被装饰”。如果这样实现的话，decorators_list属性就应该是一个对象，它的属性名字是方法名，值是装饰器对象的数组。

## 策略模式

策略模式允许在运行的时候选择算法。你的代码的使用者可以在处理特定任务的时候根据即将要做的事情的上下文来从一些可用的算法中选择一个。

使用策略模式的一个例子是解决表单验证的问题。

你可以创建一个validator对象，有一个validate()方法。这个方法被调用时不用区分具体的表单类型，它总是会返回同样的结果——一个没有通过验证的列表和错误信息。但是根据具体的需要验证的表单和数据，你代码的使用者可以选择进行不同类别的检查。你的validator选择最佳的策略来处理这个任务，然后将具体的数据检查工作交给合适的算法去做。

