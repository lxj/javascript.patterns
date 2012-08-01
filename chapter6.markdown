<a name="a1"></a>
# 代码复用模式

代码复用是一个既重要又有趣的话题，因为努力在自己或者别人写的代码上写尽量少且可以复用的代码是件很自然的事情，尤其当这些代码是经过测试的、可维护的、可扩展的、有文档的时候。

当我们说到代码复用的时候，想到的第一件事就是继承，本章会有很大篇幅讲述这个话题。你将看到好多种方法来实现“类式（classical）”和一些其它方式的继承。但是，最最重要的事情，是你需要记住终极目标——代码复用。继承是达到这个目标的一种方法，但是不是唯一的。在本章，你将看到怎样基于其它对象来构建新对象，怎样使用混元，以及怎样在不使用继承的情况下只复用你需要的功能。

在做代码复用的工作的时候，谨记Gang of Four 在书中给出的关于对象创建的建议：“优先使用对象创建而不是类继承”。（译注：《设计模式：可复用面向对象软件的基础》（Design Patterns: Elements of Reusable Object-Oriented Software）是一本设计模式的经典书籍，该书作者为Erich Gamma、Richard Helm、Ralph Johnson和John Vlissides，被称为“Gang of Four”，简称“GoF”。）


## 类式继承 vs 现代继承模式

在讨论JavaScript的继承这个话题的时候，经常会听到“类式继承”的概念，那我们先看一下什么是类式（classical）继承。classical一词并不是来自某些古老的、固定的或者是被广泛接受的解决方案，而仅仅是来自单词“class”。（译注：classical也有“经典”的意思。）

很多编程语言都有原生的类的概念，作为对象的蓝本。在这些语言中，每个对象都是一个指定类的实例（instance），并且（以Java为例）一个对象不能在不存在对应的类的情况下存在。在JavaScript中，因为没有类，所以类的实例的概念没什么意义。JavaScript的对象仅仅是简单的键值对，这些键值对都可以动态创建或者是改变。

但是JavaScript拥有构造函数（constructor functions），并且有语法和使用类非常相似的new运算符。

在Java中你可能会这样写：

	Person adam = new Person();

在JavaScript中你可以这样：

	var adam = new Person();

除了Java是强类型语言需要给adam添加类型Person外，其它的语法看起来是一样的。JavaScript的创建函数调用看起来感觉Person是一个类，但事实上，Person仅仅是一个函数。语法上的相似使得非常多的开发者陷入对JavaScript类的思考，并且给出了很多模拟类的继承方案。这样的实现方式，我们叫它“类式继承”。顺便也提一下，所谓“现代”继承模式是指那些不需要你去想类这个概念的模式。

当需要给项目选择一个继承模式时，有不少的备选方案。你应该尽量选择那些现代继承模式，除非团队已经觉得“无类不欢”。

本章先讨论类式继承，然后再关注现代继承模式。

## 类式继承的期望结果

实现类式继承的目标是基于构造函数Child()来创建一个对象，然后从另一个构造函数Parent()获得属性。

> 尽管我们是在讨论类式继承，但还是尽量避免使用“类”这个词。“构造函数”或者“constructor”虽然更长，但是更准确，不会让人迷惑。通常情况下，应该努力避免在跟团队沟通的时候使用“类”这个词，因为在JavaScript中，很可能每个人都会有不同的理解。

下面是定义两个构造函数Parent()和Child()的例子：

    //parent构造函数
    function Parent(name) {
        this.name = name || 'Adam';
    }
    
    //给原型增加方法
    Parent.prototype.say = function () {
        return this.name;
    };
    
    //空的child构造函数
    function Child(name) {}
    
    //继承
    inherit(Child, Parent);
    
上面的代码定义了两个构造函数Parent()和Child()，say()方法被添加到了Parent()构建函数的原型（prototype）中，inherit()函数完成了继承的工作。inherit()函数并不是原生提供的，需要自己实现。让我们来看一看比较大众的实现它的几种方法。

## 类式继承1——默认模式

最常用的一种模式是使用Parent()构造函数来创建一个对象，然后把这个对象设为Child()的原型。这是可复用的inherit()函数的第一种实现方法：

	function inherit(C, P) {
		C.prototype = new P();
	}

需要强调的是原型（prototype属性）应该指向一个对象，而不是函数，所以它需要指向由父构造函数创建的实例（对象），而不是构造函数自己。换句话说，请注意new运算符，有了它这种模式才可以正常工作。

之后在应用中使用new Child()创建对象的时候，它将通过原型拥有Parent()实例的功能，像下面的例子一样：

	var kid = new Child();
	kid.say(); // "Adam"

### 跟踪原型链

在这种模式中，子对象既继承了（父对象）“自己的属性”（添加给this的实例属性，比如name），也继承了原型中的属性和方法（比如say()）。

我们来看一下在这种继承模式中原型链是怎么工作的。为了讨论方便，我们假设对象是内在中的一块空间，它包含数据和指向其它空间的引用。当使用new Parent()创建一个对象时，这样的一块空间就被分配了（图6-1中的2号）。它保存着name属性的数据。如果你尝试访问say()方法（比如通过(new Parent).say()），2号空间中并没有这个方法。但是在通过隐藏的链接__proto__指向Parent()构建函数的原型prototype属性时，就可以访问到包含say()方法的1号空间（Parent.prototype）了。所有的这一块都是在幕后发生的，不需要任何额外的操作，但是知道它是怎样工作的以及你正在访问或者修正的数据在哪是很重要的。注意，__proto__在这里只是为了解释原型链，这个属性在语言本身中是不可用的，尽管有一些环境提供了（比如Firefox）。

![图6-1 Parent()构造函数的原型链](./Figure/chapter6/6-1.jpg)

图6-1 Parent()构造函数的原型链

现在我们来看一下在使用inherit()函数之后再使用var kid = new Child()创建一个新对象时会发生什么。见图6-2。

![图6-2 继承后的原型链](./Figure/chapter6/6-2.jpg)

图6-2 继承后的原型链

Child()构造函数是空的，也没有属性添加到Child.prototype上，这样，使用new Child()创建出来的对象都是空的，除了有隐藏的链接__proto__。在这个例子中，__proto__指向在inherit()函数中创建的new Parent()对象。

现在使用kid.say()时会发生什么？3号对象没有这个方法，所以通过原型链找到2号。2号对象也没有这个方法，所以也通过原型链找到1号，刚好有这个方法。接下来say()方法引用了this.name，这个变量也需要解析。于是沿原型链查找的过程又走了一遍。在这个例子中，this指向3号对象，它没有name属性。然后2号对象被访问，并且有name属性，值为“Adam”。

最后，我们多看一点东西，假如我们有如下的代码：

	var kid = new Child();
	kid.name = "Patrick";
	kid.say(); // "Patrick"

图6-3展现了这个例子的原型链：

![图6-3 继承并且给子对象添加属性后的原型链](./Figure/chapter6/6-3.jpg)

图6-3 继承并且给子对象添加属性后的原型链

设定kid.name并没有改变2号对象的name属性，但是它直接在3号对象上添加了自己的name属性。当kid.say()执行时，say方法在3号对象中找，然后是2号，最后到1号，像前面说的一样。但是这一次在找this.name（和kid.name一样）时很快，因为这个属性在3号对象中就被找到了。

如果通过delete kid.name的方式移除新添加的属性，那么2号对象的name属性将暴露出来并且在查找的时候被找到。

### 这种模式的缺点

这种模式的一个缺点是既继承了（父对象）“自己的属性”，也继承了原型中的属性。大部分情况下你可能并不需要“自己的属性”，因为它们更可能是为实例对象添加的，并不用于复用。

> 一个在构造函数上常用的规则是，用于复用的成员（译注：属性和方法）应该被添加到原型上。

在使用这个inherit()函数时另外一个不便是它不能够让你传参数给子构造函数，这些参数有可能是想再传给父构造函数的。考虑下面的例子：

	var s = new Child('Seth');
	s.say(); // "Adam"

这并不是我们期望的结果。事实上传递参数给父构造函数是可能的，但这样需要在每次需要一个子对象时再做一次继承，很不方便，因为需要不断地创建父对象。


## 类式继承2——借用构造函数

下面这种模式解决了从子对象传递参数到父对象的问题。它借用了父对象的构造函数，将子对象绑定到this，同时传入参数：

	function Child(a, c, b, d) {
		Parent.apply(this, arguments);
	}

使用这种模式时，只能继承在父对象的构造函数中添加到this的属性，不能继承原型上的成员。

使用借用构造函数的模式，子对象通过复制的方式继承父对象的成员，而不是像类式继承1中那样获得引用。下面的例子展示了这两者的不同：

	//父构造函数
	function Article() {
		this.tags = ['js', 'css'];
	}
	var article = new Article();

	//BlogPost通过类式继承1（默认模式）从article继承
	function BlogPost() {}
	BlogPost.prototype = article;
	var blog = new BlogPost();
	//注意你不需要使用`new Article()`，因为已经有一个实例了

	//StaticPage通过借用构造函数的方式从Article继承
	function StaticPage() {
		Article.call(this);
	}
	var page = new StaticPage();
	
	alert(article.hasOwnProperty('tags')); // true
	alert(blog.hasOwnProperty('tags')); // false
	alert(page.hasOwnProperty('tags')); // true

在上面的代码片段中，Article()被两种方式分别继承。默认模式使blog可以通过原型链访问到tags属性，所以它自己并没有tags属性，hasOwnProperty()返回false。page对象有自己的tags属性，因为它是使用借用构造函数的方式继承，复制（而不是引用）了tags属性。

注意在修改继承后的tags属性时的不同：

	blog.tags.push('html');
	page.tags.push('php');
	alert(article.tags.join(', ')); // "js, css, html"

在这个例子中，blog对象修改了tags属性，同时，它也修改了父对象，因为实际上blog.tags和article.tags是引向同一个数组。而对pages.tags的修改并不影响父对象article，因为pages.tags在继承的时候是一份独立的拷贝。


### 原型链

我们来看一下当我们使用熟悉的Parent()和Child()构造函数和这种继承模式时原型链是什么样的。为了使用这种继承模式，Child()有明显变化：

	//父构造函数
	function Parent(name) {
		this.name = name || 'Adam';
	}

	//在原型上添加方法
	Parent.prototype.say = function () {
		return this.name;
	};

	//子构造函数
	function Child(name) {
		Parent.apply(this, arguments);
	}

	var kid = new Child("Patrick");
		kid.name; // "Patrick"
	typeof kid.say; // "undefined"

如果看一下图6-4，就能发现new Child对象和Parent之间不再有链接。这是因为Child.prototype根本就没有被使用，它指向一个空对象。使用这种模式，kid拥有了自己的name属性，但是并没有继承say()方法，如果尝试调用它的话会出错。这种继承方式只是一种一次性地将父对象的属性复制为子对象的属性，并没有__proto__链接。

![图6-4 使用借用构造函数模式时没有被关联的原型链](./Figure/chapter6/6-4.jpg)

图6-4 使用借用构造函数模式时没有被关联的原型链


### 利用借用构造函数模式实现多继承

使用借用构造函数模式，可以通过借用多个构造函数的方式来实现多继承：

	function Cat() {
		this.legs = 4;
		this.say = function () {
			return "meaowww";
		}
	}

	function Bird() {
		this.wings = 2;
		this.fly = true;
	}

	function CatWings() {
		Cat.apply(this);
		Bird.apply(this);
	}

	var jane = new CatWings();
	console.dir(jane);

结果如图6-5，任何重复的属性都会以最后的一个值为准。

![图6-5 在Firebug中查看CatWings对象](./Figure/chapter6/6-5.jpg)

图6-5 在Firebug中查看CatWings对象


### 借用构造函数的利与弊

这种模式的一个明显的弊端就是无法继承原型。如前面所说，原型往往是添加可复用的方法和属性的地方，这样就不用在每个实例中再创建一遍。

这种模式的一个好处是获得了父对象自己成员的拷贝，不存在子对象意外改写父对象属性的风险。

那么，在上一个例子中，怎样使一个子对象也能够继承原型属性呢？怎样能使kid可以访问到say()方法呢？下一种继承模式解决了这个问题。


## 类式继承3——借用并设置原型

综合以上两种模式，首先借用父对象的构造函数，然后将子对象的原型设置为父对象的一个新实例：

	function Child(a, c, b, d) {
		Parent.apply(this, arguments);
	}
	Child.prototype = new Parent();

这样做的好处是子对象获得了父对象自己的成员，也获得了父对象中可复用的（在原型中实现的）方法。子对象也可以传递任何参数给父构造函数。这种行为可能是最接近Java的，子对象继承了父对象的所有东西，同时可以安全地修改自己的属性而不用担心修改到父对象。

一个弊端是父构造函数被调用了两次，所以不是很高效。最后，（父对象）自己的属性（比如这个例子中的name）也被继承了两次。

我们来看一下代码并做一些测试：

	//父构造函数
	function Parent(name) {
		this.name = name || 'Adam';
	}

	//在原型上添加方法
	Parent.prototype.say = function () {
		return this.name;
	};

	//子构造函数
	function Child(name) {
		Parent.apply(this, arguments);
	}
	Child.prototype = new Parent();

	var kid = new Child("Patrick");
	kid.name; // "Patrick"
	kid.say(); // "Patrick"
	delete kid.name;
	kid.say(); // "Adam"

跟前一种模式不一样，现在say()方法被正确地继承了。可以看到name也被继承了两次，在删除掉自己的拷贝后，在原型链上的另一个就被暴露出来了。

图6-6展示了这些对象之间的关系。这些关系有点像图6-3中展示的，但是获得这种关系的方法是不一样的。

![图6-6 除了继承“自己的属性”外，原型链也被保留了](./Figure/chapter6/6-6.jpg)

图6-6 除了继承“自己的属性”外，原型链也被保留了


## 类式继承4——共享原型

不像前一种类式继承模式需要调用两次父构造函数，下面这种模式根本不会涉及到调用父构造函数的问题。

一般的经验是将可复用的成员放入原型中而不是this。从继承的角度来看，则是任何应该被继承的成员都应该放入原型中。这样你只需要设定子对象的原型和父对象的原型一样即可：

	function inherit(C, P) {
		C.prototype = P.prototype;
	}

这种模式的原型链很短并且查找很快，因为所有的对象实际上共享着同一个原型。但是这样也有弊端，那就是如果子对象或者在继承关系中的某个地方的任何一个子对象修改这个原型，将影响所有的继承关系中的父对象。（译注：这里应该是指会影响到所有从这个原型中继承的对象。）

如图6-7，子对象和父对象共享同一个原型，都可以访问say()方法。但是，子对象不继承name属性。

![图6-7 （父子对象）共享原型时的关系](./Figure/chapter6/6-7.jpg)

图6-7 （父子对象）共享原型时的关系


## 类式继承5——临时构造函数

下一种模式通过打断父对象和子对象原型的直接链接解决了共享原型时的问题，同时还从原型链中获得其它的好处。

下面是这种模式的一种实现方式，F()函数是一个空函数，它充当了子对象和父对象的代理。F()的prototype属性指向父对象的原型。子对象的原型是一这个空函数的一个实例：

	function inherit(C, P) {
		var F = function () {};
		F.prototype = P.prototype;
		C.prototype = new F();
	}

这种模式有一种和默认模式（类式继承1）明显不一样的行为，因为在这里子对象只继承原型中的属性（图6-8）。

![图6-8 使用临时（代理）构造函数F()实现类式继承](./Figure/chapter6/6-8.jpg)

图6-8 使用临时（代理）构造函数F()实现类式继承

这种模式通常情况下都是一种很棒的选择，因为原型本来就是存放复用成员的地方。在这种模式中，父构造函数添加到this中的任何成员都不会被继承。

我们来创建一个子对象并且检查一下它的行为：

	var kid = new Child();

如果你访问kid.name将得到undefined。在这个例子中，name是父对象自己的属性，而在继承的过程中我们并没有调用new Parent()，所以这个属性并没有被创建。当访问kid.say()时，它在3号对象中不可用，所以在原型链中查找，4号对象也没有，但是1号对象有，它在内在中的位置会被所有从Parent()创建的构造函数和子对象所共享。


### 存储父类（Superclass）

在上一种模式的基础上，还可以添加一个指向原始父对象的引用。这很像其它语言中访问超类（superclass）的情况，有时候很方便。

我们将这个属性命名为“uber”，因为“super”是一个保留字，而“superclass”则可能误导别人认为JavaScript拥有类。下面是这种类式继承模式的一个改进版实现：

	function inherit(C, P) {
		var F = function () {};
		F.prototype = P.prototype;
		C.prototype = new F();
		C.uber = P.prototype;
	}


### 重置构造函数引用

这个近乎完美的模式上还需要做的最后一件事情就是重置构造函数（constructor）的指向，以便未来在某个时刻能被正确地使用。

如果不重置构造函数的指向，那所有的子对象都会认为Parent()是它们的构造函数，而这个结果完全没有用。使用前面的inherit()的实现，你可以观察到这种行为：

	// parent, child, inheritance
	function Parent() {}
	function Child() {}
	inherit(Child, Parent);

	// testing the waters
	var kid = new Child();
	kid.constructor.name; // "Parent"
	kid.constructor === Parent; // true

constructor属性很少用，但是在运行时检查对象很方便。你可以重新将它指向期望的构造函数而不影响功能，因为这个属性更多是“信息性”的。（译注：即它更多的时候是在提供信息而不是参与到函数功能中。）

最终，这种类式继承的Holy Grail版本看起来是这样的：

	function inherit(C, P) {
		var F = function () {};
		F.prototype = P.prototype;
		C.prototype = new F();
		C.uber = P.prototype;
		C.prototype.constructor = C;
	}

类似这样的函数也存在于YUI库（也许还有其它库）中，它将类式继承的方法带给了没有类的语言。如果你决定使用类式继承，那么这是最好的方法。

> “代理函数”或者“代理构造函数”也是指这种模式，因为临时构造函数是被用作获取父构造函数原型的代理。

一种常见的对Holy Grail模式的优化是避免每次需要继承的时候都创建一个临时（代理）构造函数。事实上创建一次就足够了，以后只需要修改它的原型即可。你可以用一个立即执行的函数来将代理函数存储到闭包中：

	var inherit = (function () {
		var F = function () {};
		return function (C, P) {
			F.prototype = P.prototype;
			C.prototype = new F();
			C.uber = P.prototype;
			C.prototype.constructor = C;
		}
	}());
	

## Klass

有很多JavaScript类库模拟了类，创造了新的语法糖。具体的实现方式可能会不一样，但是基本上都有一些共性，包括：

- 有一个约定好名字的方法，如initialize、_init或者其它相似的名字，会被自动调用，来充当类的构造函数。
- 类可以从其它类继承
- 在子类中可以访问到父类（superclass）

> 我们在这里做一下变化，在本章的这部分自由地使用“class”单词，因为主题就是模拟类。

为避免讨论太多细节，我们来看一下JavaScript中一种模拟类的实现。首先，这种解决方案从客户的角度来看将如何被使用？

	var Man = klass(null, {
		__construct: function (what) {
			console.log("Man's constructor");
			this.name = what;
		},
		getName: function () {
			return this.name;
		}
	});
	
这种语法糖的形式是一个名为klass()的函数。在一些实现方式中，它可能是Klass()构造函数或者是增强的Object.prototype，但是在这个例子中，我们让它只是一个简单的函数。

这个函数接受两个参数：一个被继承的类和通过对象字面量提供的新类的实现。受PHP的影响，我们约定类的构造函数必须是一个名为\_\_construct的方法。在前面的代码片段中，建立了一个名为Man的新类，并且它不继承任何类（意味着继承自Object）。Man类有一个在\_\_construct建立的自己的属性name和一个方法getName()。这个类是一个构造函数，所以下面的代码将正常工作（并且看起来像类实例化的过程）：

	var first = new Man('Adam'); // logs "Man's constructor"
	first.getName(); // "Adam"
	
现在我们来扩展这个类，创建一个SuperMan类：

	var SuperMan = klass(Man, {
		__construct: function (what) {
			console.log("SuperMan's constructor");
		},
		getName: function () {
			var name = SuperMan.uber.getName.call(this);
			return "I am " + name;
		}
	});
	
这里，klass()的第一个参数是将被继承的Man类。值得注意的是，在getName()中，父类的getName()方法首先通过SuperMan类的uber静态属性被调用。我们来测试一下：

	var clark = new SuperMan('Clark Kent');
	clark.getName(); // "I am Clark Kent"
	
第一行在console中记录了“Man's constructor”，然后是“Superman's constructor”。在一些语言中，父类的构造函数在子类构造函数被调用的时候会自动执行，这个特性也可以模拟。

用instanceof运算符测试返回希望的结果：

	clark instanceof Man; // true
	clark instanceof SuperMan; // true
	
最后，我们来看一下klass()函数是怎样实现的：

	var klass = function (Parent, props) {
	
		var Child, F, i;
		
		// 1.
		// new constructor
		Child = function () {
			if (Child.uber && Child.uber.hasOwnProperty("__construct")) {
				Child.uber.__construct.apply(this, arguments);
			}
			if (Child.prototype.hasOwnProperty("__construct")) {
				Child.prototype.__construct.apply(this, arguments);
			}
		};
		
		// 2.
		// inherit
		Parent = Parent || Object;
		F = function () {};
		F.prototype = Parent.prototype;
		Child.prototype = new F();
		Child.uber = Parent.prototype;
		Child.prototype.constructor = Child;
	
		// 3.
		// add implementation methods
		for (i in props) {
			if (props.hasOwnProperty(i)) {
				Child.prototype[i] = props[i];
			}
		}
		
		// return the "class"
		return Child;
	};
	
这个klass()实现有三个明显的部分：

1. 创建Child()构造函数，这也是最后返回的将被作为类使用的函数。在这个函数里面，如果__construct方法存在的话将被调用。同样是在父类的\_\_construct（如果存在）被调用前使用静态的uber属性。也可能存在uber没有定义的情况——比如从Object继承，因为它是在Man类中被定义的。
2. 第二部分主要完成继承。只是简单地使用前面章节讨论过的Holy Grail类式继承模式。只有一个东西是新的：如果Parent没有传值的话，设定Parent为Object。
3. 最后一部分是类真正定义的地方，循环需要实现的方法（如例子中的\_\_constructt和getName），并将它们添加到Child的原型中。

什么时候使用这种模式？其实，最好是能避免则避免，因为它带来了在这门语言中不存在的完整的类的概念，会让人疑惑。使用它需要学习新的语法和新的规则。也就是说，如果你或者你的团队对类感到习惯并且同时对原型感到不习惯，这种模式可能是一个可以探索的方向。这种模式允许你完全忘掉原型，好处就是你可以将语法变种得像其它你所喜欢的语言一样。
