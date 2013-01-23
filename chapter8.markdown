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

## 事件

在浏览器脚本编程中，另一块充满兼容性问题并且带来很多不愉快的区域就是浏览器事件，比如click，mouseover等等。同样的，一个JavaScript库可以解决支持IE（9以下）和W3C标准实现的双倍工作量。

我们来看一下一些主要的点，因为你在做一些简单的页面或者快速开发的时候可能不会使用已有的库，当然，也有可能你正在写你自己的库。

### 事件处理

麻烦是从给元素绑定事件开始的。假设你有一个按钮，点击它的时候增加计数器的值。你可以添加一个内联的onclick属性，这在所有的浏览器中都能正常工作，但是会违反分离和渐进增强的思想。所以你应该尽力在JavaScript中来做绑定，而不是在标签中。

假设你有下面的标签：

	<button id="clickme">Click me: 0</button>

你可以将一个函数赋给节点的onclick属性，但你只能这样做一次：

	// suboptimal solution
	var b = document.getElementById('clickme'),
		count = 0;
	b.onclick = function () {
		count += 1;
		b.innerHTML = "Click me: " + count;
	};

如果你希望在按钮点击的时候执行好几个函数，那么在维持松耦合的情况下就不能用这种方法来做绑定。从技术上讲，你可以检测onclick是否已经包含一个函数，如果已经包含，就将它加到你自己的函数中，然后替换onclick的值为你的新函数。但是一个更干净的解决方案是使用addEventListener()方法。这个方法在IE8及以下版本中不存在，在这些浏览器需要使用attachEvent()。

当我们回头看条件初始化模式（第4章）时，会发现一个示例实现是一个很好的解决跨浏览器事件监听的套件。现在我们不讨论细节，只看一下如何给我们的按钮绑定事件：

	var b = document.getElementById('clickme');
	if (document.addEventListener) { // W3C
		b.addEventListener('click', myHandler, false);
	} else if (document.attachEvent) { // IE
		b.attachEvent('onclick', myHandler);
	} else { // last resort
		b.onclick = myHandler;
	}

现在当按钮被点击时，myHandler会被执行。让我们来让这个函数实现增加按钮文字“Click me: 0”中的数字的功能。为了更有趣一点，我们假设有好几个按钮，一个myHandler()函数来处理所有的按钮点击。如果我们可以从每次点击的事件对象中获取节点和节点对应的计数器值，那为每个按钮保持一个引用和计数器就显得不高效了。

我们先看一下解决方案，稍后再来做些评论：

	function myHandler(e) {

		var src, parts;

		// get event and source element 
		e = e || window.event;
		src = e.target || e.srcElement;

		// actual work: update label
		parts = src.innerHTML.split(": ");
		parts[1] = parseInt(parts[1], 10) + 1;
		src.innerHTML = parts[0] + ": " + parts[1];

		// no bubble
		if (typeof e.stopPropagation === "function") {
		e.stopPropagation();
		}
		if (typeof e.cancelBubble !== "undefined") {
		e.cancelBubble = true;
		}

		// prevent default action
		if (typeof e.preventDefault === "function") {
		e.preventDefault();
		}
		if (typeof e.returnValue !== "undefined") {
		e.returnValue = false;
		}

	}

一个在线的例子可以在<http://jspatterns.com/book/8/click.html>找到。

在这个事件处理函数中，有四个部分：

- 首先，我们需要访问事件对象，它包含事件的一些信息以及触发这个事件的页面元素。事件对象会被传到事件处理回调函数中，但是使用onclick属性时需要使用全局属性window.event来获取。
- 第二部分是真正用于更新文字的部分
- 接下来是阻止事件冒泡。在这个例子中它不是必须的，但通常情况下，如果你不阻止的话，事件会一直冒泡到文档根元素甚至window对象。同样的，我们也需要用两种方法来阻止冒泡：W3C标准方式（stopPropagation()）和IE的方式（使用cancelBubble）
- 最后，如果需要的话，阻止默认行为。有一些事件（点击链接、提交表单）有默认的行为，但你可以使用preventDefault()（IE是通过设置returnValue的值为false的方式）来阻止这些默认行为。

如你所见，这里涉及到了很多重复性的工作，所以使用第7章讨论过的外观模式创建自己的事件处理套件是很有意义的。

### 事件委托

事件委托是通过事件冒泡来实现的，它可以减少分散到各个节点上的事件处理函数的数量。如果有10个按钮在一个div元素中，你可以给div绑定一个事件处理函数，而不是给每个按钮都绑定一个。

我们来的睦一个实例，三个按钮放在一个div元素中（图8-1）。你可以在<http://jspatterns.com/book/8/click-delegate.html>看到这个事件委托的实例。

![图8-1 事件委托示例：三个在点击时增加计数器值的按钮](./figure/chapter8/8-1.jpg)

图8-1 事件委托示例：三个在点击时增加计数器值的按钮

结构是这样的：

	<div id="click-wrap">
		<button>Click me: 0</button>
		<button>Click me too: 0</button>
		<button>Click me three: 0</button>
	</div>

你可以给包裹按钮的div绑定一个事件处理函数，而不是给每个按钮绑定一个。然后你可以使用和前面的示例中一样的myHandler()函数，但需要修改一个小地方：你需要将你不感兴趣的点击排除掉。在这个例子中，你只关注按钮上的点击，而在同一个div中产生的其它的点击应该被忽略掉。

myHandler()的改变就是检查事件来源的nodeName是不是“button”:

	// ...
	// get event and source element
	e = e || window.event;
	src = e.target || e.srcElement;

	if (src.nodeName.toLowerCase() !== "button") {
		return;
	}
	// ...

事件委托的坏处是筛选容器中感兴趣的事件使得代码看起来更多了，但好处是性能的提升和更干净的代码，这个好处明显大于坏处，因此这是一种强烈推荐的模式。

主流的JavaScript库通过提供方便的API的方式使得使用事件委托变得很容易。比如YUI3中有Y.delegate()方法，它允许你指定一个用来匹配包裹容器的CSS选择器和一个用于匹配你感兴趣的节点的CSS选择器。这很方便，因为如果事件发生在你不关心的元素上时，你的事件处理回调函数不会被调用。在这种情况下，绑定一个事件处理函数很简单：

	Y.delegate('click', myHandler, "#click-wrap", "button");

感谢YUI抽象了浏览器的差异，已经处理好了事件的来源，使得回调函数更简单了：

	function myHandler(e) {

		var src = e.currentTarget,
			parts;

		parts = src.get('innerHTML').split(": ");
		parts[1] = parseInt(parts[1], 10) + 1;
		src.set('innerHTML', parts[0] + ": " + parts[1]);

		e.halt();
	}

你可以在<http://jspatterns.com/book/8/click-y-delegate.html>看到实例。

## 长时间运行的脚本

你可能注意到过，有时候浏览器会提示脚本运行时间过长，询问用户是否要停止执行。这种情况你当然不希望发生在自己的应用中，不管它有多复杂。

同时，如果脚本运行时间太长的话，浏览器的UI将变得没有响应，用户不能点击任何东西。这是一种很差的用户体验，应该尽量避免。

在JavaScript中没有线程，但你可以在浏览器中使用setTimeout来模拟，或者在现代浏览器中使用web workers。

### setTimeout()

它的思想是将一大堆工作分解成为一小段一小段，然后每隔1毫秒运行一段。使用1毫秒的延迟会导致整个任务完成得更慢，但是用户界面会保持可响应状态，用户会觉得浏览器没有失控，觉得更舒服。

> 1毫秒（甚至0毫秒）的延迟执行命令在实际运行的时候会延迟更多，这取决于浏览器和操作系统。设定0毫秒的延迟并不意味着马上执行，而是指“尽快执行”。比如，在IE中，最短的延迟是15毫秒。

### Web Workers

现代浏览器为长时间运行的脚本提供了另一种解决方案：web workers。web workers在浏览器内部提供了后台线程支持，你可以将计算量很大的部分放到一个单独的文件中，比如my_web_worker.js，然后从主程序（页面）中这样调用它：

	var ww = new Worker('my_web_worker.js');
	ww.onmessage = function (event) {
		document.body.innerHTML +=
			"<p>message from the background thread: " + event.data + "</p>";
	};

下面展示了一个做1亿次简单的数学运算的web worker：

	var end = 1e8, tmp = 1;

	postMessage('hello there');

	while (end) {
		end -= 1;
		tmp += end;
		if (end === 5e7) { // 5e7 is the half of 1e8
			postMessage('halfway there, `tmp` is now ' + tmp);
		}
	}

	postMessage('all done');

web worker使用postMessage()来和调用它的程序通讯，调用者通过onmessage事件来接受更新。onmessage事件处理函数接受一个事件对象作为参数，这个对象含有一个由web worker传过来data属性。类似的，调用者（在这个例子中）也可以使用ww.postMessage()来给web worker传递数据，web worker可以通过一个onmessage事件处理函数来接受这些数据。

上面的例子会在浏览器中打印出：

	message from the background thread: hello there
	message from the background thread: halfway there, `tmp` is now 3749999975000001 message from the background thread: all done

## 远程脚本编程

现代web应用经常会使用远程脚本编程和服务器通讯，而不刷新当前页面。这使得web应用更灵活，更像桌面程序。我们来看一下几种用JavaScript和服务器通讯的方法。

### XMLHttpRequest

现在，XMLHttpRequest是一个特别的对象（构造函数），绝大多数浏览器都可以用，它使得我们可以从JavaScript来发送HTTP请求。发送一个请求有以下三步：

1. 初始化一个XMLHttpRequest对象（简称XHR）
2. 提供一个回调函数，供请求对象状态改变时调用
3. 发送请求

第一步很简单：

	var xhr = new XMLHttpRequest();

但是在IE7之前的版本中，XHR的功能是使用ActiveX对象实现的，所以需要做一下兼容处理。

第二步是给readystatechange事件提供一个回调函数：

	xhr.onreadystatechange = handleResponse;

最后一步是使用open()和send()两个方法触发请求。open()方法用于初始化HTTP请求的方法（如GET，POST）和URL。send()方法用于传递POST的数据，如果是GET方法，则是一个空字符串。open()方法的最后一个参数用于指定这个请求是不是异步的。异步是指浏览器在等待响应的时候不会阻塞，这明显是更好的用户体验，因此除非必须要同步，否则异步参数应该使用true：

	xhr.open("GET", "page.html", true);
	xhr.send();

下面是一个完整的示例，它获取新页面的内容，然后将当前页面的内容替换掉（可以在<http://jspatterns.com/ book/8/xhr.html>看到示例）：

	var i, xhr, activeXids = [
		'MSXML2.XMLHTTP.3.0',
		'MSXML2.XMLHTTP',
		'Microsoft.XMLHTTP'
	];

	if (typeof XMLHttpRequest === "function") { // native XHR
		xhr = new XMLHttpRequest();
	} else { // IE before 7
		for (i = 0; i < activeXids.length; i += 1) {
			try {
				xhr = new ActiveXObject(activeXids[i]);
				break;
			} catch (e) {}
		}
	}

	xhr.onreadystatechange = function () {
		if (xhr.readyState !== 4) {
			return false;
		}
		if (xhr.status !== 200) {
			alert("Error, status code: " + xhr.status);
			return false;
		}
		document.body.innerHTML += "<pre>" + xhr.responseText + "<\/pre>"; };

	xhr.open("GET", "page.html", true);
	xhr.send("");

代码中的一些说明：

- 因为IE6及以下版本中，创建XHR对象有一点复杂，所以我们通过一个数组列出ActiveX的名字，然后遍历这个数组，使用try-catch块来尝试创建对象。
- 回调函数会检查xhr对象的readyState属性。这个属性有0到4一共5个值，4代表“complete”（完成）。如果状态还没有完成，我们就继续等待下一次readystatechange事件。
- 回调函数也会检查xhr对象的status属性。这个属性和HTTP状态码对应，比如200（OK）或者是404（Not found）。我们只对状态码200感兴趣，而将其它所有的都报为错误（为了简化示例，否则需要检查其它不代表出错的状态码）。
- 上面的代码会在每次创建XHR对象时检查一遍支持情况。你可以使用前面提到过的模式（如条件初始化）来重写上面的代码，使得只需要做一次检查。

### JSONP

JSONP（JSON with padding）是另一种发起远程请求的方式。与XHR不同，它不受浏览器同源策略的限制，所以考虑到加载第三方站点的安全影响的问题，使用它时应该很谨慎。

一个XHR请求的返回可以是任何类型的文档：

- XML文档（过去很常用）
- HTML片段（很常用）
- JSON数据（轻量、方便）
- 简单的文本文件及其它

使用JSONP的话，数据经常是被包裹在一个函数中的JSON，函数名称在请求的时候提供。

JSONP的请求URL通常是像这样：

	http://example.org/getdata.php?callback=myHandler

getdata.php可以是任何类型的页面或者脚本。callback参数指定用来处理响应的JavaScript函数。

这个URL会被放到一个动态生成的\<script\>元素中，像这样：

	var script = document.createElement("script");
	script.src = url;
	document.body.appendChild(script);

服务器返回一些作为参数传递给回调函数的JSON数据。最终的结果实际上是页面中多了一个新的脚本，这个脚本的内容就是一个函数调用，如：

	myHandler({"hello": "world"});

（译注：原文这里说得不是太明白。JSONP的返回内容如上面的代码片段，它的工作原理是在页面中动态插入一个脚本，这个脚本的内容是函数调用+JSON数据，其中要调用的函数是在页面中已经定义好的，数据以参数的形式存在。一般情况下数据由服务端动态生成，而函数由页面生成，为了使返回的脚本能调用到正确的函数，在请求的时候一般会带上callback参数以便后台动态返回处理函数的名字。）

#### JSONP示例：井字棋

我们来看一个使用JSONP的井字棋游戏示例，玩家就是客户端（浏览器）和服务器。它们两者都会产生1到9之间的随机数，我们使用JSONP去取服务器产生的数字（图8-2）。

你可以在<http://jspatterns.com/book/8/ttt.html>玩这个游戏。

![图8-2 使用JSONP的井字棋游戏](./figure/chapter8/8-2.jpg)

图8-2 使用JSONP的井字棋游戏

界面上有两个按钮：一个用于开始新游戏，一个用于取服务器下的棋（客户端下的棋会在一定数量的延时之后自动进行）：

	<button id="new">New game</button>
	<button id="server">Server play</button>

界面上包含9个单元格，每个都有对应的id，比如：

	<td id="cell-1">&nbsp;</td>
	<td id="cell-2">&nbsp;</td>
	<td id="cell-3">&nbsp;</td>
	...

整个游戏是在一个全局对象ttt中实现：

	var ttt = { 
		// cells played so far
		played: [],

		// shorthand 
		get: function (id) {
			return document.getElementById(id);
		},

		// handle clicks
		setup: function () {
			this.get('new').onclick = this.newGame;
			this.get('server').onclick = this.remoteRequest;
		},

		// clean the board
		newGame: function () {
			var tds = document.getElementsByTagName("td"),
				max = tds.length, 
				i;
			for (i = 0; i < max; i += 1) {
				tds[i].innerHTML = "&nbsp;";
			}
			ttt.played = [];
		},

		// make a request
		remoteRequest: function () {
			var script = document.createElement("script");
			script.src = "server.php?callback=ttt.serverPlay&played=" + ttt.played.join(',');
			document.body.appendChild(script);
		},

		// callback, server's turn to play
		serverPlay: function (data) {
			if (data.error) {
				alert(data.error);
				return;
			}

			data = parseInt(data, 10);
			this.played.push(data);

			this.get('cell-' + data).innerHTML = '<span class="server">X<\/span>';

			setTimeout(function () {
				ttt.clientPlay();
			}, 300); // as if thinking hard
		},

		// client's turn to play
		clientPlay: function () {
			var data = 5;

			if (this.played.length === 9) {
				alert("Game over");
				return;
			}

			// keep coming up with random numbers 1-9 
			// until one not taken cell is found 
			while (this.get('cell-' + data).innerHTML !== "&nbsp;") {
				data = Math.ceil(Math.random() * 9);
			}
			this.get('cell-' + data).innerHTML = 'O';
			this.played.push(data);
		} 
	};

ttt对象维护着一个已经填过的单元格的列表ttt.played，并且将它发送给服务器，这样服务器就可以返回一个没有玩过的数字。如果有错误发生，服务器会像这样响应：

	ttt.serverPlay({"error": "Error description here"});

如你所见，JSONP中的回调函数必须是公开的并且全局可访问的函数，它并不一定要是全局函数，也可以是一个全局对象的方法。如果没有错误发生，服务器将会返回一个函数调用，像这样：

	ttt.serverPlay(3);

这里的3是指3号单元格是服务器要下棋的位置。在这种情况下，数据非常简单，甚至都不需要使用JSON格式，只需要一个简单的值就可以了。

### 框架（frame）和图片信标(image beacon)

另外一种做远程脚本编程的方式是使用框架。你可以使用JavaScript来创建框架并改变它的src URL。新的URL可以包含数据和函数调用来更新调用者，也就是框架之外的父页面。

远程脚本编程中最最简单的情况是你只需要传递一点数据给服务器，而并不需要服务器的响应内容。在这种情况下，你可以创建一个新的图片，然后将它的src指向服务器的脚本：

	new Image().src = "http://example.org/some/page.php";

这种模式叫作图片信标，当你想发送一些数据给服务器记录时很有用，比如做访问统计。因为信标的响应对你来说完全是没有用的，所以通常的做法（不推荐）是让服务器返回一个1x1的GIF图片。更好的做法是让服务器返回一个“204 No Content”HTTP响应。这意味着返回给客户端的响应只有响应头（header）而没有响应体（body）。



