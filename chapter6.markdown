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

    // the parent constructor
    function Parent(name) {
        this.name = name || 'Adam';
    }
    
    // adding functionality to the prototype
    Parent.prototype.say = function () {
        return this.name;
    };
    
    // empty child constructor
    function Child(name) {}
    
    // inheritance magic happens here
    inherit(Child, Parent);
    
上面的代码定义了两个构造函数Parent()和Child()，say()方法被添加到了Parent()构建函数的原型（prototype）中，inherit()函数完成了继承的工作。inherit()函数并不是原生提供的，需要自己实现。让我们来看一看比较大众的实现它的几种方法。