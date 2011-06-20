# 第二章 准备知识

本章将对一些实质内容展开讨论，这些内容包括最佳实践、模式和编写高质量JavaScript代码的习惯，比如避免全局变量、使用单var声明、循环中的length预缓存、遵守编码约定等等。本章还包括一些非必要的编程习惯，但更多的关注点将放在总体的代码创建过程上，包括撰写API文档、组织相互评审以及使用JSLint。这些习惯和最佳实践可以帮助你写出更好的、更易读的和可维护的代码，当几个月后或数年后再重读你的代码时，你就会深有体会了。

## 编写可维护的代码

修复软件bug成本很高，而且随着时间的推移，它们造成的损失也越来越大，特别是在已经打包发布了的软件发现了bug的时候。当然最好是发现bug立刻解决掉，但前提是你对你的代码依然很熟悉，否则当你转身投入到另外一个项目的开发中后，根本不记得当初代码的模样了。过了一段时间后你再去阅读当初的代码你需要：

- 时间来重新学习并理解问题
- 时间去理解问题相关的代码

对大型项目或者公司来说还有一个不得不考虑的问题，就是解决这个bug的人和制造这个bug的人往往不是同一个人。因此减少理解代码所需的时间成本就显得非常重要，不管是隔了很长时间重读自己的代码还是阅读团队内其他人的代码。这对于公司的利益底线和工程师的幸福指数同样重要，因为每个人都宁愿去开发新的项目而不愿花很多时间和精力去维护旧代码。

另外一个软件开发中的普遍现象是，在读代码上花的时间要远远超过写代码的时间。常常当你专注于某个问题的时候，你会坐下来用一下午的时间产出大量的代码。当时的场景下代码是可以正常运行的，但当应用趋于成熟，会有很多因素促使你重读代码、改进代码或对代码做微调。比如：

- 发现了bug
- 需要给应用添加新需求
- 需要将应用迁移到新的平台中运行（比如当市场中出现了新的浏览器时）
- 代码重构
- 由于架构更改或者更换另一种语言导致代码重写

这些不确定因素带来的后果是，少数人花几小时写的代码需要很多人花几个星期去阅读它。因此，创建可维护的代码对于一个成功的应用来说至关重要。

可维护的代码意味着代码是：

- 可读的
- 一致的
- 可预测的
- 看起来像是同一个人写的
- 有文档的

本章接下来的部分会对这几点深入讲解。

## 减少全局对象

JavaScript 使用函数来管理作用域，在一个函数内定义的变量称作“局部变量”，局部变量在函数外部是不可见的。另一方面，“全局变量”是不在任何函数体内部声明的变量，或者是直接使用而未明的变量。

每一个JavaScript运行环境都有一个“全局对象”，不在任何函数体内使用this就可以获得对这个全局对象的引用。你所创建的每一个全局变量都是这个全局对象的属性。为了方便起见，浏览器都会额外提供一个全局对象的属性window，（常常）用以指向全局对象本身。下面的示例代码中展示了如何在浏览器中创建或访问全局变量：

	myglobal = "hello"; // antipattern
	console.log(myglobal); // "hello"
	console.log(window.myglobal); // "hello"
	console.log(window["myglobal"]); // "hello"
	console.log(this.myglobal); // "hello"

### 全局对象带来的困扰

全局变量的问题是，它们在JavaScript代码执行期间或者整个web页面中始终是可见的。它们存在于同一个命名空间中，因此命名冲突的情况时有发生，毕竟在应用程序的不同模块中，经常会出于某种目的定义相同的全局变量。

同样，常常网页中所嵌入的代码并不是这个网页的开发者所写，比如：

- 网页中使用了第三方的JavaScript库
- 网页中使用了广告代码
- 网页中使用了用以分析流量和点击率的第三方统计代码
- 网页中使用了很多组件，挂件和按钮等等

假设某一段第三方提供的脚本定义了一个全局变量result。随后你在自己写的某个函数中也定义了一个全局变量result。这时，第二个变量就会覆盖第一个，这时就会导致第三方脚本停止工作。

因此，为了让你的脚本和这个页面中的其他脚本和谐相处，要尽可能少的使用全局变量，这一点非常重要。本书随后的章节中会讲到一些减少全局变量的技巧和策略，比如使用命名空间或者立即执行的匿名函数等，但减少全局变量最有效的方法是坚持使用var来声明变量。

由于JavaScript的特点，我们经常有意无意的创建全局变量，毕竟在JavaScript中创建全局变量实在太简单了。首先，你可以不声明而直接使用变量，再者，JavaScirpt中具有“隐式全局对象”的概念，也就是说任何不通过var声明（译注：在JavaScript1.7及以后的版本中，可以通过let来声明块级作用域的变量）的变量都会成为全局对象的一个属性（可以把它们当作全局变量）。看一下下面这段代码：

	function sum(x, y) {
		// antipattern: implied global
		result = x + y;
		return result;
	}

这段代码中，我们直接使用了result而没有事先声明它。这段代码是能够正常工作的，但在调用这个方法之后，会产生一个全局变量result，这会带来其他问题。

解决办法是，总是使用var来声明变量，下面代码就是改进了的sum()函数：

	function sum(x, y) {
		var result = x + y;
		return result;
	}

这里我们要注意一种反模式，就是在var声明中通过链式赋值的方法创建全局变量。在下面这个代码片段中，a是局部变量，但b是全局变量，而作者的意图显然不是如此：

	// antipattern, do not use
	function foo() {
		var a = b = 0;
		// ...
	}

为什么会这样？因为这里的计算顺序是从右至左的。首先计算表达式b=0，这里的b是未声明的，这个表达式的值是0，然后通过var创建了局部变量a，并赋值为0。换言之，可以等价的将代码写成这样：

	var a = (b = 0);

如果变量b已经被声明，这种链式赋值的写法是ok的，不会意外的创建全局变量，比如：

	function foo() {
		var a, b;
		// ...
		a = b = 0; // both local
	}

> 避免使用全局变量的另一个原因是出于可移植性考虑的，如果你希望将你的代码运行于不同的平台环境（宿主），使用全局变量则非常危险。很有可能你无意间创建的某个全局变量在当前的平台环境中是不存在的，你认为可以安全的使用，而在其他的环境中却是存在的。

### 忘记var时的副作用

隐式的全局变量和显式定义的全局变量之间有着细微的差别，差别在于通过delete来删除它们的时候表现不一致。

- 通过var创建的全局变量（在任何函数体之外创建的变量）不能被删除。
- 没有用var创建的隐式全局变量（不考虑函数内的情况）可以被删除。

也就是说，隐式全局变量并不算是真正的变量，但他们是全局对象的属性成员。属性是可以通过delete运算符删除的，而变量不可以被删除：

	// define three globals
	var global_var = 1;
	global_novar = 2; // antipattern
	(function () {
		global_fromfunc = 3; // antipattern
	}());

	// attempt to delete
	delete global_var; // false
	delete global_novar; // true
	delete global_fromfunc; // true

	// test the deletion
	typeof global_var; // "number"
	typeof global_novar; // "undefined"
	typeof global_fromfunc; // "undefined"

在ES5严格模式中，给未声明的变量赋值会报错（比如这段代码中提到的两个反模式）。

### 访问全局对象

在浏览器中，我们可以随时随地通过window属性来访问全局对象（除非你定义了一个名叫window的局部变量）。但换一个运行环境这个方便的window可能就换成了别的名字（甚至根本就被禁止访问全局对象了）。如果不想通过这种写死window的方式来得到全局变量，有一个办法，你可以在任意层次嵌套的函数作用域内执行：

	var global = (function () {
		return this;
	}());

这种方式总是可以得到全局对象，因为在被当作函数执行的函数体内（而不是被当作构造函数执行的函数体内），this总是指向全局对象。但这种情况在ECMAScript5的严格模式中行不通，因此在严格模式中你不得不寻求其他的替代方案。比如，如果你在开发一个库，你会将你的代码包装在一个立即执行的匿名函数中（在第四章会讲到），然后从全局作用域中给这个匿名函数传入一个指向this的参数。

### 单 var 模式

在函数的顶部使用一个单独的var语句是非常推荐的一种模式，它有如下一些好处：

- 在同一个位置可以查找到函数所需的所有变量
- 避免当在变量声明之前使用这个变量时产生的逻辑错误（参照下一小节“声明提前：分散的 var 带来的问题”）
- 提醒你不要忘记声明变量，顺便减少潜在的全局变量
- 代码量更少（输入更少且更易做代码优化）

单var模式看起来像这样：

	function func() {
		var a = 1,
			b = 2,
			sum = a + b,
			myobject = {},
			i,
			j;
		// function body...
	}

你可以使用一个var语句来声明多个变量，变量之间用逗号分隔。也可以在这个语句中加入变量的初始化，这是一个非常好的实践。这种方式可以避免逻辑错误（所有未初始化的变量都被声明了，且值为undefined）并增加了代码的可读性。过段时间后再看这段代码，你会体会到声明不同类型变量的惯用名称，比如，你一眼就可看出某个变量是对象还是整数。

你可以在声明变量时多做一些额外的工作，比如在这个例子中就写了sum=a+b这种代码。另一个例子就是当代码中用到对DOM元素时，你可以把对DOM的引用赋值给一些变量，这一步就可以放在一个单独的声明语句中，比如下面这段代码：

	function updateElement() {
		var el = document.getElementById("result"),
			style = el.style;
		// do something with el and style...
	}

### 声明提前：分散的 var 带来的问题 

JavaScript 中是允许在函数的任意地方写任意多个var语句的，其实相当于在函数体顶部声明变量，这种现象被称为“变量提前”，当你在声明之前使用这个变量时，可能会造成逻辑错误。对于JavaScript来说，一旦在某个作用域（同一个函数内）里声明了一个变量，这个变量在整个作用域内都是存在的，包括在var声明语句之前。看一下这个例子：

	// antipattern
	myname = "global"; // global variable
	function func() {
		alert(myname); // "undefined"
		var myname = "local";
		alert(myname); // "local"
	}
	func();

这个例子中，你可能期望第一个alert()弹出“global”，第二个alert()弹出“local”。这种结果看起来是合乎常理的，因为在第一个alert执行时，myname还没有声明，这时就应该“寻找”全局变量中的myname。但实际情况并不是这样，第一个alert弹出“undefined”，因为myname已经在函数内有声明了（尽管声明语句在后面）。所有的变量声明都提前到了函数的顶部。因此，为了避免类似带有“歧义”的程序逻辑，最好在使用之前一起声明它们。

上一个代码片段等价于下面这个代码片段：

	myname = "global"; // global variable
	function func() {
		var myname; // same as -> var myname = undefined;
		alert(myname); // "undefined"
		myname = "local";
		alert(myname); // "local"
	}
	func();

>这里有必要对“变量提前”作进一步补充，实际上从JavaScript引擎的工作机制上看，这个过程稍微有点复杂。代码处理经过了两个阶段，第一阶段是创建变量、函数和参数，这一步是预编译的过程，它会扫描整段代码的上下文。第二阶段是代码的运行，这一阶段将创建函数表达式和一些非法的标识符（未声明的变量）。从实用性角度来讲，我们更愿意将这两个阶段归成一个概念“变量提前”，尽管这个概念并没有在ECMAScript标准中定义，但我们常常用它来解释预编译的行为过程。

## for 循环

在for循环中，可以对数组或类似数组的对象（比如arguments和HTMLCollection对象）作遍历，最普通的for循环模式形如：

	// sub-optimal loop
	for (var i = 0; i < myarray.length; i++) {
		// do something with myarray[i]
	}

这种模式的问题是，每次遍历都会访问数组的length属性。这降低了代码运行效率，特别是当myarray并不是一个数组而是一个HTMLCollection对象的时候。

HTMLCollection是由DOM方法返回的对象，比如：

- document.getElementsByName()
- document.getElementsByClassName()
- document.getElementsByTagName()

还有很多其他的HTMLCollection，这些对象是在DOM标准之前就已经在用了，这些HTMLCollection主要包括：

**document.images**

页面中所有的IMG元素

**document.links**

页面中所有的A元素

**document.forms**

页面中所有的表单

**document.forms[0].elements**

页面中第一个表单的所有字段

这些对象的问题在于，它们均是指向文档（HTML页面）中的活动对象。也就是说每次通过它们访问集合的length时，总是会去查询DOM，而DOM操作则是很耗资源的。

更好的办法是为for循环缓存住要遍历的数组的长度，比如下面这段代码：

	for (var i = 0, max = myarray.length; i < max; i++) {
		// do something with myarray[i]
	}

通过这种方法只需要访问DOM节点一次以获得length，在整个循环过程中就都可以使用它。

不管在什么浏览器中，在遍历HTMLCollection时缓存length都可以让程序执行的更快，可以提速两倍（Safari3）到一百九十倍（IE7）不等。更多细节可以参照Nicholas Zakas的《高性能JavaScript》，这本书也是由O'Reilly出版。

需要注意的是，当你在循环过程中需要修改这个元素集合（比如增加DOM元素）时，你更希望更新length而不是更新常量。

遵照单var模式，你可以将var提到循环的外部，比如：

	function looper() {
		var i = 0,
			max,
			myarray = [];
		// ...
		for (i = 0, max = myarray.length; i < max; i++) {
			// do something with myarray[i]
		}
	}

这种模式带来的好处就是提高了代码的一致性，因为你越来越依赖这种单var模式。缺点就是在重构代码的时候不能直接复制粘贴一个循环体，比如，你正在将某个循环从一个函数拷贝至另外一个函数中，必须确保i和max也拷贝至新函数里，并且需要从旧函数中将这些没用的变量删除掉。

最后一个需要对循环做出调整的地方是将i++替换成为下面两者之一：

	i = i + 1
	i += 1

JSLint提示你这样做，是因为++和--实际上降低了代码的可读性，如果你觉得无所谓，可以将JSLint的plusplus选项设为false（默认为true），本书所介绍的最后一个模式用到了: i += 1。

关于这种for模式还有两种变化的形式，做了少量改进，原因有二：

- 减少一个变量（没有max）
- 减量循环至0，这种方式速度更快，因为和零比较要比和非零数字或数组长度比较要高效的多

第一种变化形式是：

	var i, myarray = [];
	for (i = myarray.length; i--;) {
		// do something with myarray[i]
	}

第二种变化形式用到了while循环：

	var myarray = [],
		i = myarray.length;
	while (i--) {
		// do something with myarray[i]
	}

这些小改进只体现在性能上，此外，JSLint不推荐使用i--。

## for-in 循环

for-in 循环用于对非数组对象作遍历。通过for-in进行循环也被称作“枚举”。

从技术角度讲，for-in循环同样可以用于数组（JavaScript中数组即是对象），但不推荐这样做。当使用自定义函数扩充了数组对象时，这时更容易产生逻辑错误。另外，for-in循环中属性的遍历顺序是不固定的，所以最好数组使用普通的for循环，对象使用for-in循环。

可以使用对象的hasOwnProperty()方法将从原型链中继承来的属性过滤掉，这一点非常重要。看一下这段代码：

	// the object
	var man = {
		hands: 2,
		legs: 2,
		heads: 1
	};
	// somewhere else in the code
	// a method was added to all objects
	if (typeof Object.prototype.clone === "undefined") {
		Object.prototype.clone = function () {};
	}

在这段例子中，我们定义了一个名叫man的对象直接量。在代码中的某个地方（可以是man定义之前也可以是之后），给Object的原型中增加了一个方法clone()。原型链是实时的，这意味着所有的对象都可以访问到这个新方法。要想在枚举man的时候避免枚举出clone()方法，则需要调用hasOwnProperty()来对原型属性进行过滤。如果不做过滤，clone()也会被遍历到，而这不是我们所希望的：

	// 1.
	// for-in loop
	for (var i in man) {
		if (man.hasOwnProperty(i)) { // filter
			console.log(i, ":", man[i]);
		}
	}
	/*
	result in the console
	hands : 2
	legs : 2
	heads : 1
	*/

	// 2.
	// antipattern:
	// for-in loop without checking hasOwnProperty()
	for (var i in man) {
		console.log(i, ":", man[i]);
	}
	/*
	result in the console
	hands : 2
	legs : 2
	heads : 1
	clone: function()
	*/

另外一种的写法是通过Object.prototype直接调用hasOwnProperty()方法，像这样：

	for (var i in man) {
		if (Object.prototype.hasOwnProperty.call(man, i)) { // filter
			console.log(i, ":", man[i]);
		}
	}

这种做法的好处是，当man对象中重新定义了hasOwnProperty方法时，可以避免调用时的命名冲突（译注：明确指定调用的是Object.prototype上的方法而不是实例对象中的方法），这种做法同样可以避免冗长的属性查找过程（译注：这种查找过程多是在原型链上进行查找），一直查找到Object中的方法，你可以定义一个变量来“缓存”住它（译注：这里所指的是缓存住Object.prototype.hasOwnProperty）：

	var i,
		hasOwn = Object.prototype.hasOwnProperty;
	for (i in man) {
		if (hasOwn.call(man, i)) { // filter
			console.log(i, ":", man[i]);
		}
	}

>严格说来，省略hasOwnProperty()并不是一个错误。根据具体的任务以及你对代码的自信程度，你可以省略掉它以提高一些程序执行效率。但当你对当前要遍历的对象不确定的时候，添加hasOwnProperty()则更加保险些。

这里提到一种格式上的变化写法（这种写法无法通过JSLint检查），这种写法在for循环所在的行加入了if判断条件，他的好处是能让循环语句读起来更完整和通顺（“如果元素包含属性X，则拿X做点什么”）：

	// Warning: doesn't pass JSLint
	var i,
		hasOwn = Object.prototype.hasOwnProperty;
	for (i in man) if (hasOwn.call(man, i)) { // filter
		console.log(i, ":", man[i]);
	}

## （不）扩充内置原型

我们可以扩充构造函数的prototype属性，这是一种非常强大的特性，用来为构造函数增加功能，但有时这个功能强大到超过我们的掌控。

给内置构造函数比如Object()、Array()、和Function()扩充原型看起来非常诱人，但这种做法严重降低了代码的可维护性，因为它让你的代码变得难以预测。对于那些基于你的代码做开发的开发者来说，他们更希望使用原生的JavaScript方法来保持工作的连续性，而不是使用你所添加的方法（译注：因为原生的方法更可靠，而你写的方法可能会有bug）。

另外，如果将属性添加至原型中，很可能导致在那些不使用hasOwnProperty()做检测的循环中将原型上的属性遍历出来，这会造成混乱。

因此，不扩充内置对象的原型是最好的，你也可以自己定义一个规则，仅当下列条件满足时做例外考虑：




