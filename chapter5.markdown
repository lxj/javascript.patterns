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

