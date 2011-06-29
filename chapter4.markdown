# 函数

熟练运用函数是JavaScript程序员的必备技能，因为在JavaScript中函数实在是太常用了。它能够完成的任务种类非常之多，而在其他语言中则需要很多特殊的语法支持才能达到这种能力。

在本章将会介绍在JavaScript中定义函数的多种方式，包括函数表达式和函数声明、以及局部作用域和变量声明提前的工作原理。然后会介绍一些有用的模式，帮助你设计API（为你的函数提供更好的接口）、搭建代码架构（使用尽可能少的全局对象）、并优化性能（避免不必要的操作）。

现在让我们来一起揭秘JavaScript函数，我们首先从一些背景知识开始说起。

## 背景知识

JavaScript的函数具有两个主要特性，正是这两个特性让它们与众不同。第一个特性是，函数是一等对象（first-class object），第二个是函数提供作用域支持。

函数是对象，那么：

- 可以在程序执行时动态创建函数
- 可以将函数赋值给变量，可以将函数的引用拷贝至另一个变量，可以扩充函数，除了某些特殊场景外均可被删除。
- 可以将函数作为参数传入另一个函数，也可以被当作返回值返回。
- 函数可以包含自己的属性和方法

对于一个函数A来说，首先它是对象，拥有属性和方法，其中某个属性碰巧是另一个函数B，B可以接受函数作为参数，假设这个函数参数为C，当执行B的时候，返回另一个函数D。乍一看这里有一大堆相互关联的函数。当你开始习惯函数的许多用法时，你会惊叹原来函数是如此强大、灵活并富有表现力。通常说来，一说到JavaScript的函数，我们首先认为它是对象，它具有一个可以“执行”的特性，也就是说我们可以“调用”这个函数。

我们通过new Function()构造器来生成一个函数，这时可以明显看出函数是对象：

	// antipattern
	// for demo purposes only
	var add = new Function('a, b', 'return a + b');
	add(1, 2); // returns 3

在这段代码中，毫无疑问add()是一个对象，毕竟它是由构造函数创建的。这里并不推荐使用Function()构造器创建函数（和eval()一样糟糕），因为程序逻辑代码是以字符串的形式传入构造器的。这样的代码可读性差，写起来也很费劲，你不得不对逻辑代码中的引号做转义处理，并需要特别关注为了让代码保持一定的可读性而保留的空格和缩进。

函数的第二个重要特性是它能提供作用域支持。在JavaScript中没有块级作用域（译注：在JavaScript1.7中提供了块级作用域部分特性的支持，可以通过let来声明块级作用域内的“局部变量”），也就是说不能通过花括号来创建作用域，JavaScript中只有函数作用域（译注：这里作者的表述只针对函数而言，此外JavaScript还有全局作用域）。在函数内所有通过var声明的变量都是局部变量，在函数外部是不可见的。刚才所指花括号无法提供作用域支持的意思是说，如果在if条件句内、或在for或while循环体内用var定义了变量，这个变量并不是属于if语句或for（while）循环的局部变量，而是属于它所在的函数。如果不在任何函数内部，它会成为全局变量。在第二章里提到我们要减少对全局命名空间的污染，那么使用函数则是控制变量的作用域的不二之选。

### 术语释义

首先我们先简单讨论下创建函数相关的术语，因为精确无歧义的术语约定和我们所讨论的各种模式一样重要。

看下这个代码片段：

	// named function expression
	var add = function add(a, b) {
		return a + b;
	};

这段代码描述了一个函数，这种描述称为“带有命名的函数表达式”。

如果函数表达式将名字省略掉（比如下面的示例代码），这时它是“无名字的函数表达式”，通常我们称之为“匿名函数”，比如：

	// function expression, a.k.a. anonymous function
	var add = function (a, b) {
		return a + b;
	};

因此“函数表达式”是一个更广义的概念，“带有命名的函数表达式”是函数表达式的一种特殊形式，仅仅当需要给函数定义一个可选的名字时使用。

当省略第二个add，它就成了无名字的函数表达式，这不会对函数定义和调用语法造成任何影响。带名字和不带名字唯一的区别是函数对象的name属性是否是一个空字符串。name属性属于语言的扩展（未在ECMA标准中定义），但很多环境都实现了。如果不省略第二个add，那么属性add.name则是"add"，name属性在用Firebug的调试过程中非常有用，还能让函数递归调用自身，其他情况可以省略它。

最后来看一下“函数声明”，函数声明的语法和其他语言中的语法非常类似：

	function foo() {
		// function body goes here
	}

从语法角度讲，带有命名的函数表达式和函数声明非常像，特别是当不需要将函数表达式赋值给一个变量的时候（在本章后面所讲到的回调模式中有类似的例子）。多数情况下，函数声明和带命名的函数表达式在外观上没有多少不同，只是它们在函数执行时对上下文的影响有所区别，下一小节会讲到。

两种语法的一个区别是末尾的分号。函数声明末尾不需要分号，而函数表达式末尾是需要分号的。推荐你始终不要丢掉函数表达式末尾的分号，即便JavaScript可以进行分号补全，也不要冒险这样做。

>另外我们经常看到“函数直接量”。它用来表示函数表达式或带命名的函数表达式。由于这个术语是有歧义的，所以最好不要用它。


### 声明 vs 表达式：命名与提前

那么，到底应该用哪个呢？函数声明还是函数表达式？在不能使用函数声明语法的场景下，只能使用函数表达式了。下面这个例子中，我们给函数传入了另一个函数对象作为参数，以及给对象定义方法:

	// this is a function expression,
	// pased as an argument to the function `callMe`
	callMe(function () {
		// I am an unnamed function expression
		// also known as an anonymous function
	});

	// this is a named function expression
	callMe(function me() {
		// I am a named function expression
		// and my name is "me"
	});

	// another function expression
	var myobject = {
		say: function () {
			// I am a function expression
		}
	};

函数声明只能出现在“程序代码”中，也就是说在别的函数体内或在全局。这个定义不能赋值给变量或属性，同样不能作为函数调用的参数。下面这个例子是函数声明的合法用法，这里所有的函数foo()，bar()和local()都使用函数声明来定义：

	// global scope
	function foo() {}

	function local() {
		// local scope
		function bar() {}
		return bar;
	}

### 函数的name属性

选择函数定义模式的另一个考虑是只读属性name的可用性。尽管标准规范中并未规定，但很多运行环境都实现了name属性，在函数声明和带有名字的函数表达式中是有name的属性定义的。在匿名函数表达式中，则不一定有定义，这个是和实现相关的，在IE中是无定义的，在Firefox和Safari中是有定义的，但是值为空字符串。

	function foo() {} // declaration
	var bar = function () {}; // expression
	var baz = function baz() {}; // named expression

	foo.name; // "foo"
	bar.name; // ""
	baz.name; // "baz"

在Firebug或其他工具中调试程序时name属性非常有用，它可以用来显示当前正在执行的函数。同样可以通过name属性来递归的调用函数自身。如果你对这些场景不感兴趣，那么请尽可能的使用匿名函数表达式，这样会更简单、且冗余代码更少。

和函数声明相比而言，函数表达式的语法更能说明函数是一种对象，而不是某种特别的语言写法。

>我们可以将一个带名字的函数表达式赋值给变量，变量名和函数名不同，这在技术上是可行的。比如：`var foo = function bar(){};`。然而，这种用法的行为在浏览器中的兼容性不佳（特别是IE中），因此并不推荐大家使用这种模式。

### 函数提前

通过前面的讲解，你可能以为函数声明和带名字的函数表达式是完全等价的。事实上不是这样，主要区别在于“声明提前”的行为。

>术语“提前”并未在ECMAScript中定义，但是并没有其他更好的方法来描述这种行为了。

我们知道，不管在函数内何处声明变量，变量都会自动提前至函数体的顶部。对于函数来说亦是如此，因为他们也是一种对象，赋值给了变量。需要注意的是，函数声明定义的函数不仅能让声明提前，还能让定义提前，看一下这段示例代码：

	// antipattern
	// for illustration only

	// global functions
	function foo() {
		alert('global foo');
	}
	function bar() {
		alert('global bar');
	}

	function hoistMe() {

		console.log(typeof foo); // "function"
		console.log(typeof bar); // "undefined"

		foo(); // "local foo"
		bar(); // TypeError: bar is not a function

		// function declaration:
		// variable 'foo' and its implementation both get hoisted

		function foo() {
			alert('local foo');
		}

		// function expression:
		// only variable 'bar' gets hoisted
		// not the implementation
		var bar = function () {
			alert('local bar');
		};
	}
	hoistMe();

在这段代码中，和普通的变量一样，hoistMe()函数中的foo和bar被“搬运”到了顶部，覆盖了全局的foo和bar。不同之处在于，局部的foo()定义提前至顶部并能正常工作，尽管定义它的位置并不靠前。bar()的定义并未提前，只是声明提前了。因此当程序执行到bar()定义的位置之前，它的值都是undefined，并不是函数（防止当前上下文查找到作用域链上的全局的bar()，也就“覆盖”了全局的bar()）。

到目前为止我们介绍了必要的背景知识和函数定义相关的术语，下面开始介绍一些JavaScript所提供的函数相关的好的模式，我们从回调模式开始。同样，再次强调JavaScript函数的两个特殊特性，掌握这两点至关重要：

- 函数是对象
- 函数提供局部变量作用域

## 回调模式

函数是对象，也就意味着函数可以当作参数传入另外一个函数中。当你给函数writeCode()传入一个函数参数introduceBugs()，在某个时刻writeCode()执行了（或调用了）introduceBugs()。在这种情况下，我们说introduceBugs()是一个“回调函数”，简称“回调”：

	function writeCode(callback) {
		// do something...
		callback();
		// ...
	}

	function introduceBugs() {
		// ... make bugs
	}

	writeCode(introduceBugs);

注意introduceBugs()是如何作为参数传入writeCode()的，当作参数的函数不带括号。括号的意思是执行函数，而这里我们希望传入一个引用，让writeCode()在合适的时机执行它（调用它）。

### 一个回调的例子

我们从一个例子开始，首先介绍无回调的情况，然后在作修改。假设你有一个通用的函数，用来完成某种复杂的逻辑并返回一大段数据。假设我们用findNodes()来命名这个通用函数，这个函数用来对DOM树进行遍历，并返回我所感兴趣的页面节点：

	var findNodes = function () {
		var i = 100000, // big, heavy loop
			nodes = [], // stores the result
			found; // the next node found
		while (i) {
			i -= 1;
			// complex logic here...
			nodes.push(found);
		}
		return nodes;
	};

保持这个函数的功能的通用性并一贯返回DOM节点组成的数组，并不会发生对节点的实际操作，这是一个不错的注意。可以将操作节点的逻辑放入另外一个函数中，比如放入一个hide()函数中，这个函数用来隐藏页面中的节点元素：

	var hide = function (nodes) {
		var i = 0, max = nodes.length;
		for (; i < max; i += 1) {
			nodes[i].style.display = "none";
		}
	};

	// executing the functions
	hide(findNodes());

这个实现的效率并不高，因为它将findNodes()所返回的节点数组重新遍历了一遍。最好在findNodes()中选择元素的时候就直接应用hide()操作，这样就能避免第二次的遍历，从而提高效率。但如果将hide()的逻辑写死在findNodes()的函数体内，findNodes()就变得不再通用了（译注：如果我将hide()的逻辑替换成其他逻辑怎么办呢？），因为修改逻辑和遍历逻辑耦合在一起了。如果使用回调模式，则可以将隐藏节点的逻辑写入回调函数，将其传入findNodes()中适时执行：

	// refactored findNodes() to accept a callback
	var findNodes = function (callback) {
		var i = 100000,
			nodes = [],
			found;
		
		// check if callback is callable
		if (typeof callback !== "function") {
			callback = false;
		}
		while (i) {
			i -= 1;

			// complex logic here...

			// now callback:
			if (callback) {
				callback(found);
			}

			nodes.push(found);
		}
		return nodes;
	};

这里的实现比较直接，findNodes()多作了一个额外工作，就是检查回调函数是否存在，如果存在的话就执行它。回调函数是可选的，因此修改后的findNodes()也是和之前一样使用，是可以兼容旧代码和旧API的。

这时hide()的实现就非常简单了，因为它不用对元素列表做任何遍历了：

	// a callback function
	var hide = function (node) {
		node.style.display = "none";
	};

	// find the nodes and hide them as you go
	findNodes(hide);

正如代码中所示，回调函数可以是事先定义好的，也可以是一个匿名函数，你也可以将其称作main函数，比如这段代码，我们利用同样的通用函数findNodes()来完成显示元素的操作：

	// passing an anonymous callback
	findNodes(function (node) {
		node.style.display = "block";
	});

### 回调和作用域

在上一个例子中，执行回调函数的写法是：

	callback(parameters);

尽管这种写法可以适用大多数的情况，而且足够简单，但还有一些场景，回调函数不是匿名函数或者全局函数，而是对象的方法。如果回调函数中使用this指向它所属的对象，则回调逻辑往往并不像我们希望的那样执行。

假设回调函数是paint()，它是myapp的一个方法：

	var myapp = {};
	myapp.color = "green";
	myapp.paint = function (node) {
		node.style.color = this.color;
	};

函数findNodes()大致如下：

	var findNodes = function (callback) {
		// ...
		if (typeof callback === "function") {
			callback(found);
		}
		// ...
	};

当你调用findNodes(myapp.paint)，运行结果和我们期望的不一致，因为this.color未定义。因为findNodes()是全局函数，this指向的是全局对象。如果findNodes()是dom对象的方法（类似dom.findNodes()），那么回调函数内的this则指向dom，而不是myapp。

解决办法是，除了传入回调函数，还需将回调函数所属的对象当作参数传进去：

	findNodes(myapp.paint, myapp);

同样需要修改findNodes()的逻辑，增加对传入的对象的绑定：

	var findNodes = function (callback, callback_obj) {
		//...
		if (typeof callback === "function") {
			callback.call(callback_obj, found);
		}
		// ...
	};

在后续的章节会对call()和apply()有更详细的讲述。

其实还有一种替代写法，就是将函数当作字符串传入findNodes()，这样就不必再写一次对象了，换句话说：

	findNodes(myapp.paint, myapp);

可以写成：
	
	findNodes("paint", myapp);

在findNodes()中的逻辑则需要修改为：

	var findNodes = function (callback, callback_obj) {

		if (typeof callback === "string") {
			callback = callback_obj[callback];
		}

		//...
		if (typeof callback === "function") {
			callback.call(callback_obj, found);
		}
		// ...
	};

### 异步事件监听

JavaScript中的回调模式已经是我们的家常便饭了，比如，如果你给网页中的元素绑定事件，则需要提供回调函数的引用，以便事件发生时能调用到它。这里有一个简单的例子，我们将console.log()作为回调函数绑定了document的点击事件：

	document.addEventListener("click", console.log, false);

客户端浏览器中的大多数编程都是事件驱动的，当网页下载完成，则触发load事件，当用户和页面产生交互时也会触发多种事件，比如click、keypress、mouseover、mousemove等等。正是由于回调模式的灵活性，JavaScript天生适于事件驱动编程。回调模式能够让程序“异步”执行，换句话说，就是让程序不按顺序执行。

“不要打电话给我，我会打给你”，这是好莱坞很有名的一句话，很多电影都有这句台词。电影中的主角不可能同时应答很多个电话呼叫。在JavaScript的异步事件模型中也是同样的道理。电影中是留下电话号码，JavaScript中是提供一个回调函数，当时机成熟时就触发回调。有时甚至提供了很多回调，有些回调压根是没用的，但由于这个事件可能永远不会发生，因此这些回调的逻辑也不会执行。比如，假设你从此不再用“鼠标点击”，那么你之前绑定的鼠标点击的回调函数则永远也不会执行。

### 超时

另外一个最常用的回调模式是在调用超时函数时，超时函数是浏览器window对象的方法，共有两个：setTimeout()和setInterval()。这两个方法的参数都是回调函数。

	var thePlotThickens = function () {
		console.log('500ms later...');
	};
	setTimeout(thePlotThickens, 500);

再次需要注意，函数thePlotThickens是作为变量传入setTimeout的，它不带括号，如果带括号的话则立即执行了，这里只是用到这个函数的引用，以便在setTimeout的逻辑中调用到它。也可以传入字符串“thePlotThickens()”，但这是一种反模式，和eval()一样不推荐使用。

### 库中的回调

回调模式非常简单，但又很强大。可以随手拈来灵活运用，因此这种模式在库的设计中也非常得宠。库的代码要尽可能的保持通用和重用，而回调模式则可帮助库的作者完成这个目标。你不必预料和实现你所想到的所有情形，因为这会让库变的膨胀而臃肿，而且大多数用户并不需要这些多余的特性支持。相反，你将精力放在核心功能的实现上，提供回调的入口作为“钩子”，可以让库的方法变得可扩展、可定制。

## 返回函数

函数是对象，因此当然可以作为返回值。也就是说，函数不一定非要返回一坨数据，函数可以返回另外一个定制好的函数，或者可以根据输入的不同按需创造另外一个函数。

这里有一个简单的例子：一个函数完成了某种功能，可能是一次性初始化，然后都基于这个返回值进行操作，这个返回值恰巧是另一个函数：

	var setup = function () {
		alert(1);
		return function () {
			alert(2);
		};
	};

	// using the setup function
	var my = setup(); // alerts 1
	my(); // alerts 2

因为setup()把返回的函数作了包装，它创建了一个闭包，我们可以用这个闭包来存储一些私有数据，这些私有数据可以通过返回的函数进行操作，但在函数外部不能直接读取到这些私有数据。比如这个例子中提供了一个计数器，每次调用这个函数计数器都会加一：

	var setup = function () {
		var count = 0;
		return function () {
			return (count += 1);
		};
	};

	// usage
	var next = setup();
	next(); // returns 1
	next(); // 2
	next(); // 3

## 自定义函数

我们动态定义函数，并将函数赋值给变量。如果将你定义的函数赋值给已经存在的函数变量的话，则新函数会覆盖旧函数。这样做的结果是，旧函数的引用就丢弃掉了，变量中所存储的引用值替换成了新的。这样看起来这个变量指代的函数逻辑就发生了变化，或者说函数进行了“重新定义”或“重写”。说起来有些拗口，实际上并不复杂，来看一个例子：

	var scareMe = function () {
		alert("Boo!");
		scareMe = function () {
			alert("Double boo!");
		};
	};
	// using the self-defining function
	scareMe(); // Boo!
	scareMe(); // Double boo!

当函数中包含一些初始化操作，并希望这些初始化只执行一次，那么这种模式是非常适合这个场景的。因为能避免的重复执行则尽量避免，函数的一部分可能再也不会执行到。在这个场景中，函数执行一次后就被重写为另外一个函数了。

使用这种模式可以帮助提高应用的执行效率，因为重新定义的函数执行的更少。

>这种模式的另外一个名字是“函数的懒惰定义”，因为直到函数执行一次后才重新定义，可以说它是“某个时间点之后才存在”，简称“懒惰定义”。

这种模式有一种明显的缺陷，就是之前给原函数添加的功能在重定义之后都丢失了。如果将这个函数定义为不同的名字，函数赋值给了很多不同的变量，或作为对象的方法使用，那么新定义的函数有可能不会执行，原始的函数会照旧执行（译注：由于函数的赋值是引用的赋值，函数赋值给多个变量只是将引用赋值给了多个变量，当某一个变量定义了新的函数，也只是变量的引用值发生变化，原函数本身依旧存在，当程序中存在某个变量的引用还是旧函数的话，旧函数还是会依旧执行）。

让我们来看一个例子，scareMe()函数在这里作为一等对象来使用：

1. 给他增加了一个属性
2. 函数对象赋值给一个新变量
3. 函数依旧可以作为方法来调用

看一下这段代码：

	// 1. adding a new property
	scareMe.property = "properly";

	// 2. assigning to a different name
	var prank = scareMe;

	// 3. using as a method
	var spooky = {
		boo: scareMe
	};

	// calling with a new name
	prank(); // "Boo!"
	prank(); // "Boo!"
	console.log(prank.property); // "properly"

	// calling as a method
	spooky.boo(); // "Boo!"
	spooky.boo(); // "Boo!"
	console.log(spooky.boo.property);

	// "properly"
	// using the self-defined function
	scareMe(); // Double boo!
	scareMe(); // Double boo!
	console.log(scareMe.property); // undefined



