# DOM和浏览器中的模式

在本书的前面几章中，我们主要关注了JavaScript核心（ECMAScript），并没有涉及太多关于在浏览器中使用JavaScript的内容。在本章，我们将探索一些在浏览器环境中的模式，因为这是最常见的JavaScript程序环境。浏览器脚本编程也是大部分不喜欢JavaScript的人对这门语言的认知。这当然是可以理解，因为在浏览器中有非常多不一致的宿主对象和DOM实现。很明显，任何能够减轻客户端脚本编程的痛楚的最佳初中都是大有益处的。

在本章中，你会看到一些零散的模式，包括DOM编程、事件处理、远程脚本、页面脚本的加载策略以及将JavaScript部署到生产环境的步骤。

但首先，让我们来简要讨论一下如何做客户端脚本编程。

## 分离

在web应用开发中主要关注的有三种东西：

- 内容

	即HTML文档
- 表现

	指定文档样式的CSS
- 行为

	JavaScript，用来处理用户交互和页面的动态变化

尽可能地将这三者分离可以加强应用在各种用户代理（译注：user agent，即为用户读取页面并呈现的软件，一般指浏览器）的可到达性（译注：delivery，指可被用户代理接受并理解的程度），比如图形浏览器、纯文本浏览器、用于残障人士的辅助技术、移动设备等等。分离常常是和渐进增强的思想一起实现的，我们从一个给最简单的用户代理的最基础的体验（纯HTML）开始，当用户代理的兼容性提升时再添加更多的可以为体验加分的东西。如果浏览器支持CSS，那么用户会看到文档更好的呈现。如果浏览器支持JavaScript，那文档会更像一个应用，提供更多的特性来增强用户体验。

在实践中，分离意味者：

- 在关掉CSS的情况下测试页面，看页面是否仍然可用，内容是否可以呈现和阅读
- 在关掉JavaScript的情况下测试页面，确保页面仍然可以完成它的主要功能，所有的链接都可以正常工作（没有href="#"的链接），表单仍然可以正常填写和提交
- 不要使用内联的事件处理（如onclick）或者是内联的style属性，因为它们不属于内容层
- 使用语义化的HTML元素，比如头部和列表等

JavaScript（行为）层的地位不应该很显赫，也就是说它不应该成为页面正常工作必须的东西，不应该使得用户在使用不支持的浏览器操作时存在障碍。它只应该被用来增强页面。

通常比较优雅的用来处理浏览器差异的方法是特性检测。它的思想是你不应该使用浏览器类型检测来决定代码的逻辑，而是应该检测在当前环境中你需要使用的某个方法或者是属性是否存在。浏览器检测一般认为是一种“反模式”（译注：anitpattern，指不好的模式）。虽然有的情况下不可避免要使用，但它应该是最后考虑的选择，并且应该只在特性检测没有办法给出明确答案（或者造成明显性能问题）的时候使用：

	// antipattern
	if (navigator.userAgent.indexOf('MSIE') !== −1) {
		document.attachEvent('onclick', console.log);
	}

	// better
	if (document.attachEvent) {
		document.attachEvent('onclick', console.log);
	}

	// or even more specific
	if (typeof document.attachEvent !== "undefined") {
		document.attachEvent('onclick', console.log);
	}

分离也有助于开发、维护，减少升级一个现有应用的难度，因为当出现问题的时候，你知道去看哪一块。当出现一个JavaScript错误的时候，你不需要去看HTML或者是CSS就能修复它。

## DOM编程

操作页面的DOM树是在客户端JavaScript编程中最普遍的动作。这也是导致开发者头疼的最主要原因（这也导致了JavaScript名声不好），因为DOM方法在不同的浏览器中实现得有很多差异。这也是为什么使用一个抽象了浏览器差异的JavaScript库能显著提高开发速度的原因。

我们来看一些在访问和修改DOM树时推荐的模式，主要考虑点是性能方面。

### DOM访问

DOM操作性能不好，这是影响JavaScript性能的最主要原因。性能不好是因为浏览器的DOM实现通常是和JavaScript引擎分离的。从浏览器的角度来讲，这样做是很有意义的，因为有可能一个JavaScript应用根本不需要DOM，而除了JavaScript之外的其它语言（如IE的VBScript）也可以用来操作页面中的DOM。

一个原则就是DOM访问的次数应该被减少到最低，这意味者：

- 避免在环境中访问DOM
- 将DOM引用赋给本地变量，然后操作本地变量
- 当可能的时候使用selectors API
- 遍历HTML collections时缓存length（见第2章）

看下面例子中的第二个（better）循环，尽管它看起来更长一些，但却要快上几十上百倍（取决于具体浏览器）：

	// antipattern
	for (var i = 0; i < 100; i += 1) {
		document.getElementById("result").innerHTML += i + ", ";
	}

	// better - update a local variable var i, content = "";
	for (i = 0; i < 100; i += 1) {
		content += i + ",";
	}
	document.getElementById("result").innerHTML += content;

在下一个代码片段中，第二个例子（使用了本地变量style）更好，尽管它需要多写一行代码，还需要多定义一个变量：

	// antipattern
	var padding = document.getElementById("result").style.padding,
		margin = document.getElementById("result").style.margin;
	
	// better
	var style = document.getElementById("result").style,
		padding = style.padding,
		margin = style.margin;

使用selectors API是指使用这个方法：

	document.querySelector("ul .selected");
	document.querySelectorAll("#widget .class");

这两个方法接受一个CSS选择器字符串，返回匹配这个选择器的DOM列表（译注：querySelector只返回第一个匹配的DOM）。selectors API在现代浏览器（以及IE8+）可用，它总是会比你使用其它DOM方法来做同样的选择要快。主流的JavaScript库的最近版本都已经使用了这个API，所以你有理由去检查你的项目，确保使用的是最新版本。

给你经常访问的元素加上一个id属性也是有好处的，因为document.getElementById(myid)是找到一个DOM元素最容易也是最快的方法。

### DOM操作

除了访问DOM元素之外，你可能经常需要改变它们、删除其中的一些或者是添加新的元素。更新DOM会导致浏览器重绘（repaint）屏幕，也经常导致重排(reflow)（重新计算元素的位置），这些操作代价是很高的。

再说一次，通用的原则仍然是尽量少地更新DOM，这意味着我们可以将变化集中到一起，然后在“活动的”（live）文档树之外去执行这些变化。

当你需要添加一棵相对较大的子树的时候，你应该在完成这棵树的构建之后再放到文档树中。为了达到这个目的，你可以使用文档碎片（document fragment）来包含你的节点。

不要这样添加节点：

	// antipattern
	// appending nodes as they are created

	var p, t;

	p = document.createElement('p');
	t = document.createTextNode('first paragraph');
	p.appendChild(t);
	document.body.appendChild(p);

	p = document.createElement('p');
	t = document.createTextNode('second paragraph');
	p.appendChild(t);
	document.body.appendChild(p);

一个更好的版本是创建一个文档碎片，然后“离线地”（译注：即不在文档树中）更新它，当它准备好之后再将它加入文档树中。当你将文档碎片添加到DOM树中时，碎片的内容将会被添加进去，而不是碎片本身。这个特性非常好用。所以当有好几个没有被包裹在同一个父元素的节点时，文档碎片是一个很好的包裹方式。

下面是使用文档碎片的例子：

	var p, t, frag;

	frag = document.createDocumentFragment();

	p = document.createElement('p');
	t = document.createTextNode('first paragraph');
	p.appendChild(t);
	frag.appendChild(p);

	p = document.createElement('p');
	t = document.createTextNode('second paragraph');
	p.appendChild(t);
	frag.appendChild(p);

	document.body.appendChild(frag);

这个例子和前面例子中每段更新一次相比，文档树只被更新了一下，只导致一次重排/重绘。

当你添加新的节点到文档中时，文档碎片很有用。当你需要更新已有的节点时，你也可以将这些变化集中。你可以将你要修改的子树的父节点克隆一份，然后对克隆的这份做修改，完成之后再去替换原来的元素。

	var oldnode = document.getElementById('result'),
		clone = oldnode.cloneNode(true);

	// work with the clone...

	// when you're done:
	oldnode.parentNode.replaceChild(clone, oldnode);

	