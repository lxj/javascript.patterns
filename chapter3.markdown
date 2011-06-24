# 第三章 直接量和构造函数

JavaScript中的直接量模式更加简洁、富有表现力，且在定义对象时不容易出错。本章将对直接量展开讨论，包括对象、数组和正则表达式直接量，以及为什么要使用等价的内置构造器函数来创建它们，比如Object()和Array()等。本章同样会介绍JSON格式，JSON是使用数组和对象直接量的形式定义的一种数据转换格式。本章还会讨论自定义构造函数，包括如何强制使用new以确保构造函数的正确执行。

本章还会补充讲述一些基础知识，比如内置包装对象Number()、String()和Boolean()，以及如何将它们和原始值（数字、字符串和布尔值）比较。最后，快速介绍一下Error()构造函数的用法。

## 对象直接量

我们可以将JavaScript中的对象简单的理解为名值对组成的散列表（hash table），在其他编程语言中被称作“关联数组”。其中的值可以是原始值也可以是对象，不管是什么类型，它们都是“属性”（properties），属性值同样可以是函数，这时属性就被称为“方法”（methods）。

JavaScript中自定义的对象（用户定义的本地对象）任何时候都是可变的。内置本地对象的属性也是可变的。你可以先创建一个空对象，然后在需要时给它添加功能。“对象直接量写法（object literal notation）”是按需创建对象的一种理想方式。

看一下这个例子：

	// start with an empty object
	var dog = {};

	// add one property
	dog.name = "Benji";

	// now add a method
	dog.getName = function () {
		return dog.name;
	};

在这个例子中，我们首先定义了一个空对象，然后添加了一个属性和一个方法，在程序的生命周期内的任何时刻都可以：

1.更改属性和方法的值，比如：

	dog.getName = function () {
		// redefine the method to return
		// a hardcoded value
		return "Fido";
	};

2.完全删除属性/方法

	delete dog.name;

3.添加更多的属性和方法

	dog.say = function () {
		return "Woof!";
	};
	dog.fleas = true;

其实不必每次开始都创建空对象，对象直接量模式可以直接在创建对象时添加功能，就像下面这个例子所展示的：

	var dog = {
		name: "Benji",
		getName: function () {
			return this.name;
		}
	};

> 在本书中多次提到“空对象”（“blank object”和“empty object”）。这只是某种简称，要知道JavaScript中根本不存在真正的空对象，理解这一点至关重要。即使最简单的`{}`对象包含从Object.prototype继承来的属性和方法。我们提到的“空（empty）对象”只是说这个对象没有自己的属性，不考虑它是否有继承来的属性。

### 对象直接量语法

如果你从来没有接触过对象直接量写法，第一次碰到可能会感觉怪怪的。但越到后来你就越喜欢它。本质上讲，对象直接量语法包括：

- 将对象主体包含在一对花括号内（`{` and `}`）。
- 对象内的属性或方法之间使用逗号分隔。最后一个名值对后也可以有逗号，但在IE下会报错，所以尽量不要在最后一个属性或方法后加逗号。
- 属性名和值之间使用冒号分隔
- 如果将对象赋值给一个变量，不要忘了在右括号`}`之后补上分号

### 通过构造函数创建对象

JavaScript中没有类的概念，这给JavaScript带来了极大的灵活性，因为你不必提前知晓关于对象的任何信息，也不需要类的“蓝图”。但JavaScript同样具有构造函数，它的语法和Java或其他语言中基于类的对象创建非常类似。

你可以使用自定义的构造函数来创建实例对象，也可以使用内置构造函数来创建，比如Object()、Date()、String()等等。

下面这个例子展示了用两种等价的方法分别创建两个独立的实例对象：

	// one way -- using a literal
	var car = {goes: "far"};

	// another way -- using a built-in constructor
	// warning: this is an antipattern
	var car = new Object();
	car.goes = "far";

从这个例子中可以看到，直接量写法的一个明显优势是，它的代码更少。“创建对象的最佳模式是使用直接量”还有一个原因，它可以强调对象就是一个简单的可变的散列表，而不必一定派生自某个类。

另外一个使用直接量而不是Object构造函数创建实例对象的原因是，对象直接量不需要“作用域解析”（scope resolution）。因为新创建的实例有可能包含了一个本地的构造函数，当你调用Object()的时候，解析器需要顺着作用域链从当前作用域开始查找，直到找到全局Object构造函数为止。

### 获得对象的构造器

创建实例对象时能用对象直接量就不要使用new Object()构造函数，但有时你希望能继承别人写的代码，这时就需要了解构造函数的一个“特性”（也是不使用它的另一个原因），就是Object()构造函数可以接收参数，通过参数的设置可以把实例对象的创建委托给另一个内置构造函数，并返回另外一个实例对象，而这往往不是你所希望的。

下面的示例代码中展示了给new Object()传入不同的参数：数字、字符串和布尔值，最终得到的对象都是由不同的构造函数生成的：

	// Warning: antipatterns ahead

	// an empty object
	var o = new Object();
	console.log(o.constructor === Object); // true

	// a number object
	var o = new Object(1);
	console.log(o.constructor === Number); // true
	console.log(o.toFixed(2)); // "1.00"

	// a string object
	var o = new Object("I am a string");
	console.log(o.constructor === String); // true
	// normal objects don't have a substring()
	// method but string objects do
	console.log(typeof o.substring); // "function"

	// a boolean object
	var o = new Object(true);
	console.log(o.constructor === Boolean); // true

Object()构造函数的这种特性会导致一些意想不到的结果，特别是当参数不确定的时候。最后再次提醒不要使用new Object()，尽可能的使用对象直接量来创建实例对象。

## 自定义构造函数

除了对象直接量和内置构造函数之外，你也可以通过自定义的构造函数来创建实例对象，正如下面的代码所示：

	var adam = new Person("Adam");
	adam.say(); // "I am Adam"

这里用了“类”Person创建了实例，这种写法看起来很像Java中的实例创建。两者的语法的确非常接近，但实际上JavaScript中没有类的概念，Person是一个函数。

Person构造函数是如何定义的呢？看下面的代码：

	var Person = function (name) {
		this.name = name;
		this.say = function () {
			return "I am " + this.name;
		};
	};

当你通过关键字new来调用这个构造函数时，函数体内将发生这些事情：

- 创建一个空对象，将它的引用赋给this，继承函数的原型。
- 通过this将属性和方法添加至这个对象
- 最后返回this指向的新对象（如果没有手动返回其他的对象）

用代码表示这个过程如下：

	var Person = function (name) {
		// create a new object
		// using the object literal
		// var this = {};

		// add properties and methods
		this.name = name;
		this.say = function () {
			return "I am " + this.name;
		};

		//return this;
	};

正如这段代码所示，say()方法添加至this中，结果是，不论何时调用new Person()，在内存中都会创建一个新函数（译注：所有Person的实例对象中的方法都是独占一块内存的）。显然这是效率很低的，因为所有实例的say()方法是一模一样的，因此没有必要“拷贝”多份。最好的办法是将方法添加至Person的原型中。

	Person.prototype.say = function () {
		return "I am " + this.name;
	};

我们将会在下一章里详细讨论原型和继承。现在只要记住将需要重用的成员和方法放在原型里即可。

关于构造函数的内部工作机制也会在后续章节中有更细致的讨论。这里我们只做概要的介绍。刚才提到，构造函数执行的时候，首先创建一个新对象，并将它的引用赋给this：

	// var this = {};

事实并不完全是这样，因为“空”对象并不是真的空，这个对象继承了Person的原型，看起来更像：

	// var this = Object.create(Person.prototype);

在后续章节会进一步讨论Object.create()。

### 构造函数的返回值

用new调用的构造函数总是会返回一个对象，默认返回this所指向的对象。如果构造函数内没有给this赋任何属性，则返回一个“空”对象（除了继承构造函数的原型之外，没有“自己的”属性）。

尽管我们不会在构造函数内写return语句，也会隐式返回this。但我们是可以返回任意指定的对象的，在下面的例子中就返回了新创建的that对象。

	var Objectmaker = function () {

		// this `name` property will be ignored
		// because the constructor
		// decides to return another object instead
		this.name = "This is it";

		// creating and returning a new object
		var that = {};
		that.name = "And that's that";
		return that;
	};

	// test
	var o = new Objectmaker();
	console.log(o.name); // "And that's that"

我们看到，构造函数中其实是可以返回任意对象的，只要你返回的东西是对象即可。如果返回值不是对象（字符串、数字或布尔值），程序不会报错，但这个返回值被忽略，最终还是返回this所指的对象。

## 强制使用new的模式

我们知道，构造函数和普通的函数无异，只是通过new调用而已。那么如果调用构造函数时忘记new会发生什么呢？漏掉new不会产生语法错误也不会有运行时错误，但可能会造成逻辑错误，导致执行结果不符合预期。这是因为如果不写new的话，函数内的this会指向全局对象（在浏览器端this指向window）。

当构造函数内包含this.member之类的代码，并直接调用这个函数（省略new），实际会创建一个全局对象的属性member，可以通过window.member或member访问到它。这必然不是我们想要的结果，因为我们要努力确保全局命名空间的整洁干净。

	// constructor
	function Waffle() {
		this.tastes = "yummy";
	}

	// a new object
	var good_morning = new Waffle();
	console.log(typeof good_morning); // "object"
	console.log(good_morning.tastes); // "yummy"

	// antipattern:
	// forgotten `new`
	var good_morning = Waffle();
	console.log(typeof good_morning); // "undefined"
	console.log(window.tastes); // "yummy"

ECMAScript5中修正了这种非正常的行为逻辑。在严格模式中，this是不能指向全局对象的。如果在不支持ES5的JavaScript环境中，仍然后很多方法可以确保构造函数的行为即便在省略new调用时也不会出问题。

### 命名约定

最简单的选择是使用命名约定，前面的章节已经提到，构造函数名首字母大写（MyConstructor），普通函数和方法名首字母小写（myFunction）。

### 使用that

遵守命名约定的确能帮上一些忙，但约定毕竟不是强制，不能完全避免出错。这里给出了一种模式可以确保构造函数一定会按照构造函数的方式执行。不要将所有成员挂在this上，将它们挂在that上，并返回that。

	function Waffle() {
	var that = {};
		that.tastes = "yummy";
		return that;
	}

如果要创建简单的实例对象，甚至不需要定义一个局部变量that，可以直接返回一个对象直接量，就像这样：

	function Waffle() {
		return {
			tastes: "yummy"
		};
	}

不管用什么方式调用它（使用new或直接调用），它同都会返回一个实例对象：

	var first = new Waffle(),
		second = Waffle();
	console.log(first.tastes); // "yummy"
	console.log(second.tastes); // "yummy"

这种模式的问题是丢失了原型，因此在Waffle()的原型上的成员不会继承到这些实例对象中。

> 需要注意的是，这里用的that只是一种命名约定，that不是语言的保留字，可以将它替换为任何你喜欢的名字，比如self或me。

### 调用自身的构造函数

为了解决上述模式的问题，能够让实例对象继承原型属性，我们使用下面的方法。在构造函数中首先检查this是否是构造函数的实例，如果不是，再通过new调用构造函数，并将new的结果返回：

	function Waffle() {

		if (!(this instanceof Waffle)) {
			return new Waffle();
		}
		this.tastes = "yummy";

	}
	Waffle.prototype.wantAnother = true;

	// testing invocations
	var first = new Waffle(),
		second = Waffle();

	console.log(first.tastes); // "yummy"
	console.log(second.tastes); // "yummy"

	console.log(first.wantAnother); // true
	console.log(second.wantAnother); // true

另一种检查实例的通用方法是使用arguments.callee，而不是直接将构造函数名写死在代码中：

	if (!(this instanceof arguments.callee)) {
		return new arguments.callee();
	}

这里需要说明的是，在任何函数内部都会自行创建一个arguments对象，它包含函数调用时传入的参数。同时arguments包含一个callee属性，指向它所在的正在被调用的函数。需要注意，ES5严格模式中是禁止使用arguments.callee的，因此最好对它的使用加以限制，并删除任何你能在代码中找到的实例（译注：这里作者的表述很委婉，其实作者更倾向于全面禁止使用arguments.callee）。

## 数组直接量

和其他的大多数一样，JavaScript中的数组也是对象。可以通过内置构造函数Array()来创建数组，类似对象直接量，数组也可以通过直接量形式创建。而且更推荐使用直接量创建数组。

这里的实例代码给出了创建两个具有相同元素的数组的两种方法，使用Array()和使用直接量模式：

	// array of three elements
	// warning: antipattern
	var a = new Array("itsy", "bitsy", "spider");

	// the exact same array
	var a = ["itsy", "bitsy", "spider"];

	console.log(typeof a); // "object", because arrays are objects
	console.log(a.constructor === Array); // true

### 数组直接量语法