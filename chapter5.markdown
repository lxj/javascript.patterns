# 对象创建模式

在JavaScript中创建对象很容易——可以通过使用对象直接量或者构造函数。本章将在此基础上介绍一些常用的对象创建模式。

JavaScript语言本身简单、直观，通常也没有其他语言那样的语法特性：命名空间、模块、包、私有属性以及静态成员。本章将介绍一些常用的模式，以此实现这些语法特性。

我们将对命名空间、依赖声明、模块模式以及沙箱模式进行初探——它们帮助更好地组织应用程序的代码，有效地减轻全局污染的问题。除此之外，还会对包括：私有和特权成员、静态和私有静态成员、对象常量、链以及类式函数定义方式在内的话题进行讨论。  

## 命名空间模式（Namespace Pattern）

命名空间可以帮助减少全局变量的数量，与此同时，还能有效地避免命名冲突、名称前缀的滥用。

JavaScript默认语法并不支持命名空间，但很容易可以实现此特性。为了避免产生全局污染，你可以为应用或者类库创建一个（通常就一个）全局对象，然后将所有的功能都添加到这个对象上，而不是到处申明大量的全局函数、全局对象以及其他全局变量。

看如下例子：

	// BEFORE: 5 globals
	// Warning: antipattern
	// constructors 
	function Parent() {} 
	function Child() {}
	// a variable
	var some_var = 1;

	// some objects
	var module1 = {}; 
	module1.data = {a: 1, b: 2}; 
	var module2 = {};

可以通过创建一个全局对象（通常代表应用名）来重构上述这类代码，比方说， MYAPP，然后将上述例子中的函数和变量都变为该全局对象的属性：

	// AFTER: 1 global
	// global object 
	var MYAPP = {};

	// constructors
	MYAPP.Parent = function () {}; 
	MYAPP.Child = function () {};

	// a variable 
	MYAPP.some_var = 1;

	// an object container 
	MYAPP.modules = {};

	// nested objects
	MYAPP.modules.module1 = {}; 
	MYAPP.modules.module1.data = {a: 1, b: 2}; 
	MYAPP.modules.module2 = {};

这里的MYAPP就是命名空间对象，对象名可以随便取，可以是应用名、类库名、域名或者是公司名都可以。开发者经常约定全局变量都采用大写（所有字母都大写），这样可以显得比较突出（不过，要记住，一般大写的变量都用于表示常量）。

这种模式是一种很好的提供命名空间的方式，避免了自身代码的命名冲突，同时还避免了同一个页面上自身代码和第三方代码（比如：JavaScript类库或者小部件）的冲突。这种模式在大多数情况下非常适用，但也有它的缺点：

* 代码量稍有增加；在每个函数和变量前加上这个命名空间对象的前缀，会增加代码量，增大文件大小
* 该全局实例可以被随时修改
* 命名的深度嵌套会减慢属性值的查询

本章后续要介绍的沙箱模式则可以避免这些缺点。


###通用命名空间函数

随着程序复杂度的提高，代码会分置在不同的文件中以特定顺序来加载，这样一来，就不能保证你的代码一定是第一个申明命名空间或者改变量下的属性的。甚至还会发生属性覆盖的问题。所以，在创建命名空间或者添加属性的时候，最好先检查下是否存在，如下所示：

	// unsafe
	var MYAPP = {};
	// better
	if (typeof MYAPP === "undefined") {
		var MYAPP = {}; 
	}
	// or shorter
	var MYAPP = MYAPP || {};

如上所示，不难看出，如果每次做类似操作都要这样检查一下就会有很多重复性的代码。比方说，要申明**MYAPP.modules.module2**，就要重复三次这样的检查。所以，我们需要一个重用的**namespace()**函数来专门处理这些检查工作，然后用它来创建命名空间，如下所示：

	// using a namespace function
	MYAPP.namespace('MYAPP.modules.module2');

	// equivalent to:
	// var MYAPP = {
	//	modules: {
	//		module2: {}
	//	}
	// };

下面是上述namespace函数的实现案例。这种实现是无损的，意味着如果要创建的命名空间已经存在，则不会再重复创建：

	var MYAPP = MYAPP || {};
	MYAPP.namespace = function (ns_string) { 
		var parts = ns_string.split('.'),
			parent = MYAPP, 
			i;

		// strip redundant leading global 
		if (parts[0] === "MYAPP") {
			parts = parts.slice(1); 
		}

		for (i = 0; i < parts.length; i += 1) {
			// create a property if it doesn't exist
			if (typeof parent[parts[i]] === "undefined") {
				parent[parts[i]] = {}; 
			}
			parent = parent[parts[i]];
		}
		return parent;
	};

上述实现支持如下使用：

	// assign returned value to a local var
	var module2 = MYAPP.namespace('MYAPP.modules.module2'); 
	module2 === MYAPP.modules.module2; // true

	// skip initial `MYAPP` 
	MYAPP.namespace('modules.module51');

	// long namespace 
	MYAPP.namespace('once.upon.a.time.there.was.this.long.nested.property');

图5-1 展示了上述代码创建的命名空间对象在Firebug下的可视结果

![MYAPP命名空间在Firebug下的可视结果](http://img04.taobaocdn.com/tps/i4/T1_8m_Xd8iXXXXXXXX-434-216.png)

图5-1 MYAPP命名空间在Firebug下的可视结果

## 声明依赖

JavaScript库往往是模块化而且有用到命名空间的，这使用你可以只使用你需要的模块。比如在YUI2中，全局变量YAHOO就是一个命名空间，各个模块作为全局变量的属性，比如YAHOO.util.Dom（DOM模块）、YAHOO.util.Event（事件模块）。

将你的代码依赖在函数或者模块的顶部进行声明是一个好主意。声明就是创建一个本地变量，指向你需要用到的模块：

	var myFunction = function () {
		// dependencies
		var event = YAHOO.util.Event,
			dom = YAHOO.util.Dom;

		// use event and dom variables
		// for the rest of the function...
	};

这是一个相当简单的模式，但是有很多的好处：

- 明确的声明依赖是告知你代码的用户，需要保证指定的脚本文件被包含在页面中。
- 将声明放在函数顶部使得依赖很容易被查找和解析。
- 本地变量（如dom）永远会比全局变量（如YAHOO）要快，甚至比全局变量的属性（如YAHOO.util.Dom）还要快，这样会有更好的性能。使用了依赖声明模式之后，全局变量的解析在函数中只会进行一次，在此之后将会使用更快的本地变量。
- 一些高级的代码压缩工具比如YUI Compressor和Google Closure compiler会重命名本地变量（比如event可能会被压缩成一个字母，如A），这会使代码更精简，但这个操作不会对全局变量进行，因为这样做不安全。

下面的代码片段是关于是否使用依赖声明模式对压缩影响的展示。尽管使用了依赖声明模式的test2()看起来复杂，因为需要更多的代码行数和一个额外的变量，但在压缩后它的代码量却会更小，意味着用户只需要下载更少的代码：

	function test1() {
		alert(MYAPP.modules.m1);
		alert(MYAPP.modules.m2);
		alert(MYAPP.modules.m51);
	}

	/*
	minified test1 body:
	alert(MYAPP.modules.m1);alert(MYAPP.modules.m2);alert(MYAPP.modules.m51)
	*/

	function test2() {
		var modules = MYAPP.modules;
		alert(modules.m1);
		alert(modules.m2);
		alert(modules.m51);
	}

	/*
	minified test2 body:
	var a=MYAPP.modules;alert(a.m1);alert(a.m2);alert(a.m51)
	*/


## 私有属性和方法

JavaScript不像Java或者其它语言，它没有专门的提供私有、保护、公有属性和方法的语法。所有的对象成员都是公有的：

	var myobj = {
		myprop: 1,
		getProp: function () {
			return this.myprop;
		}
	};
	console.log(myobj.myprop); // `myprop` is publicly accessible console.log(myobj.getProp()); // getProp() is public too

当你使用构造函数创建对象的时候也是一样的，所有的成员都是公有的：

	function Gadget() {
		this.name = 'iPod';
		this.stretch = function () {
			return 'iPad';
		};
	}
	var toy = new Gadget();
	console.log(toy.name); // `name` is public console.log(toy.stretch()); // stretch() is public

### 私有成员

尽管语言并没有用于私有成员的专门语法，但你可以通过闭包来实现。在构造函数中创建一个闭包，任何在这个闭包中的部分都不会暴露到构造函数之外。但是，这些私有变量却可以被公有方法访问，也就是在构造函数中定义的并且作为返回对象一部分的那些方法。我们来看一个例子，name是一个私有成员，在构造函数之外不能被访问：

	function Gadget() {
		// private member
		var name = 'iPod';
		// public function
		this.getName = function () {
			return name;
		};
	}
	var toy = new Gadget();

	// `name` is undefined, it's private
	console.log(toy.name); // undefined
	// public method has access to `name`
	console.log(toy.getName()); // "iPod"

如你所见，在JavaScript创建私有成员很容易。你需要做的只是将私有成员放在一个函数中，保证它是函数的本地变量，也就是说让它在函数之外不可以被访问。

