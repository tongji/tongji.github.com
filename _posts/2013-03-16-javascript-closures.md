---
layout: post
title: "JavaScript closures"
description: "JavaScript 闭包"
category: 
tags: ["JavaScript", "闭包", "closure", "js-lang", "dev"]
---
{% include JB/setup %}

## 介绍

闭包是js中最强大的特性之一，但是如果没有理解就随意使用，是不可能很好地使用它们的。然而，创建闭包却相当容易，
甚至是在不知不觉中，但是伴随而来的是潜在的危害，特别是在一些相很常见的web浏览器环境中。
为了在使用闭包时扬长避短，就必须先搞懂它的工作原理。但是，要理解闭包，首先得搞懂作用域链是如何工作的，
以及对象的属性名称是如何解析的。

官方解释：

> **闭包**：闭包是指一个拥有自由变量（它"关闭"了该表达式）和绑定了这些变量的环境的表示式（通常是一个函数）

说得通俗点就是：闭包是指一个函数对象在建立时，绑定了当时作用域下有效的自由变量。所以支持闭包的语言，
必须支持First-Class function，创建函数对象并不就等于创建了闭包，创建函数对象并且同时绑定了自由变量，
那么该函数对象才能称之为闭包。

> **自由变量**：自由变量是相对于函数而言的，即不是区域变量也不是参数变量，它的作用范围基本是在函数定义范围内的，
所以它是有界变量。

对于闭包最简单的解释是ECMAScript中支持`内部函数`，即函数定义或函数表达式是可以被定义在其它`外部函数`中。并且该内部函数可以自由访问所有的本地变量、参数和那些在外部函数中的声明的内部函数。

当其中一个内部函数可以在它被嵌套的那个外部函数之外被访问时，一个闭包就会被创建。所以该内部函数有可能在外部函数返回后被执行。
在这个时候，它仍然可以自由访问本地变量、参数和那些在外部函数中的声明的内部函数。即使外部函数已经返回了，但是这些本地变量、参数或是函数声音仍然保持不变，并且依然可以与该内部函数相互作用。 

然而，想要正确地理解闭包就需要明白它背后的工作机制以及相当多的技术细节。部分ECMA-262的内部细节在接下来的章节里面会详细阐述，绝大部分是不能被轻易忽略的。个别对对象属性名称解析已经很熟悉的人可以跳过该节，但是只有已经很熟悉闭包的人才能跳过下面的章节，并且他们可以停止阅读以继续探索它们。

## 对象属性名称的解析

ECMAScript定义了两类对象，**本地对象**和**宿主对象**，同时本地对象还有一个子类**内置对象**([ECMA 262 3rd Ed Section 4.3](http://bclary.com/2004/11/07/#a-4.3))。本地对象是独立于宿主环境的ECMAScript实现提供的对象。所有的非本地对象都是宿主对象，即宿主对象是由ECMAScript实现的宿主环境提供的对象（包括所有的BOM、DOM），如document对象、DOM节点等等。

本地对象形式很松散，很适当充当命名属性的容器。对象上的已定义的属性都拥有它的名字和值，属性的名字可以是包括空字符串在内的任意字符串，属性值可以是另外一个对象（函数同时也是对象）或是`String`、`Number`、`Boolean`、`Null`或`Undefined`等的原生类型值。在这里，如果属性值为`undefined`似乎感觉很奇怪，但是确实有可能将`undefined`赋予某个对象的属性，但是这样做并不意味着删除了该属性，该属性仍然在对象上，只是它的值为`undefined`而已。

为了尽可能多地去了解内部的细节，下面是关于如何读取或者设置对象属性值的一个简单例子。

### 属性赋值

你可以随意的创建对象的属性，又或是给已存在的属性赋值，如下：
    
    var objectRef = new Object(); //创建一个对象

我们可以创建为该对象创建名为`testNumber`的属性：

    objectRef.testNumber = 5;
    objectRef["testNumber"] = 5;

在创建`testNumber`属性前，该对象是没有`testNumber`属性，但是当给该对象赋值时，`testNmber`属性即被创建。
之后对该属性的赋值并不会创建一个新的属性，只是重新为`testNumber`赋值而已；

    objectRef.testNumber = 8;
    objectRef["testNumber"] = 8;

JavaScript中的对象的prototype本身也是对象（之后会有详细的解释），即是说该prototype也是可以有自己的属性的。但是并不能通过给对象赋值从而去设置该prototype的属性的。如果值被赋予给对象中没有相应名字的属性，那么该属性会被创建并且同时获取该值。如果它已经拥有了该属性，那么它的值会被重置。

### 属性值获取

从对象中获取属性值，同时也会涉及到该对象的prototype。如果某个对象拥有该属性，那么该属性的值被会返回：

    objectRef.testNumber = 8;
    var val = objectRef.testNumber;
   
但是所有的对象几乎都有prototype，prototype本身也是对象，所以它们也有可能有prototype，而该prototype又可能有自己的prototype...从而形成了一条链，我们称之为原型链。当原型链中的某个对象的prototype为`null`时，该原型链就结束了。Object的构造器的默认原型是为null的，所以：

    var objectRef = new Object(); 

会创建一个对象，该对象的原型是`Object.prototype`，而`Object.prototype`的prototype为null。所以`objectRef`的原型链只包含一个对象：Object.prototpye。但是：

    function MyObject1(formalParameter){
        this.testNumber = formalParameter;
    }

    function MyObject2(formalParameter){
        this.testString = formalParameter;
    }

    /*
     * 该操作默认替换了所有MyObject2的实例的默认原型
     */
    var myObject1 = new MyObject1( 8 );
    MyObject2.prototype = myObject1;
    var objectRef = new MyObject2( "String_Value" );

我们创建了`MyObject2`的实例并将它赋值给了`objectRef`，此时`objectRef`拥有了自己的原型链。

原型链中的第一个对象即是赋值给`MyObject2`构造器的原型属性的`MyObject1`的实例`myObject1`。`myObject1`的实例也有自己的原型，即是默认的`MyObject1.prototype`，而`MyObject1.prototype`的原型则是`Object.prototype`。但是`Object.prototype`并没有自己的原型，所以原型链到此结束。

当试图去读取`objectRef`对象上的某个属性时，整个原型链都会参与到属性读取的过程中来，简单来说即是：

    var val = objectRef.testString; 

`objectRef`对象有自己的属性`testString`，它的值为`String_Value`；

    var val = objectRef.testNumber;

`objectRef`对象上并没有`testNumber`属性，但是`val`却可以获取到值为8而不是`undefined`，那是因为如果不能在该对象上找到相应属性的话，那么散解析器会沿着原型链查找该属性，直至找到`myObject1`。`MyObject1`和`MyObject2`上都没有定义`toString`方法，但是如果你读取`toString`的值时：

    var val = objectRef.toString;

变量`val`会被赋予一个函数，该函数是`Object.prototype`上的`toString`属性。这也是通过原型链找到的。

最后，如果试图去读取一个并不存在的属性时：

    var val = objectRef.unfoundProperty;

它会返回`undefined`，因为无法从对象本身或者原型链中获取`unfoundProperty`属性，最后到达`Object.prototype`的prototype上，它是null，所以返回`undefinde`。

获取对象的属性时，会首先返回第一个找到的值，或许在该对象上，又或许是在原型链上。如果没有相应属性存在时，那么给对象的属性赋值时会在对象上创建一个属性，之后如果需要获取该属性的值，那么会直接从该对象上获取，而不需要去遍历原型链。

假设我们执行`objectRef.testNumber = 3`，那么就会在`objectRef`上创建`testNumber`属性，并且以后对于`objectRef.testNumber`属性值的获取都是从`objectRef`上获取，而不会牵涉到它的原型链，同时原型链上`myObject1`的`testNumber`值为仍然为8。也就是说，获取`objectRef`以其原型链上都存在的属性时，原型链上的会被隐藏。

## 标识符解析、执行上下文和作用域链

### 执行上下文

An execution context is an abstract concept used by the ECMSScript specification (ECMA 262 3rd edition) to define the behaviour required of ECMAScript implementations. The specification does not say anything about how execution contexts should be implemented but execution contexts have associated attributes that refer to specification defined structures so they might be conceived (and even implemented) as objects with properties, though not public properties.

执行上下文是一个抽象的概念，是[ECMAScript规范][ECMAScript规范]用于定义ECMAScript实现所需要的行为。ECMAScript规范并没有从技术实现的角度定义执行上下文应该如何实现以及它的具体结构和类型，这是实现规范的ECMAScript引擎所要考虑的问题。但是涉及到与规范的结构和类型相关的执行上下文的相应属性，ECMAScript引擎可以构想为对象属性（甚至是实现），但不一定是公共属性。


所有的JavaScript代码都是在一个执行上下文中执行的。全局代码（在程序级别上执行的，如外部JS文件或者内联的JS代码）是在全局执行上下文中执行的。而函数的每次调用都与一个与之关联的执行上下文。使用`eval`函数执行的代码也有自己不同的执行上下文。执行上下文的详细信息可以参考[Entering An Execution Context](http://bclary.com/2004/11/07/#a-10.2)

当一个JavaScript函数被调用时，它就进入了一个执行上下文，如果另外一个函数被调用（或是同样的函数被递归调用），一个新的执行上下文就会被创建，同时在该函数的调用过程中，会始终在该执行上下文中。当函数调用返回时，就会返回至原先的执行上下文中。因此，JavaScript的执行会形成一个执行上下文栈。

When an execution context is created a number of things happen in a defined order. First, in the execution context of a function, an "Activation" object is created. The activation object is another specification mechanism. It can be considered as an object because it ends up having accessible named properties, but it is not a normal object as it has no prototype (at least not a defined prototype) and it cannot be directly referenced by javascript code.
当创建一个执行上下文时，会进行如下的操作：

1. 首先，在函数的上下文中，会创建一个`激活对象`。激活对象

The next step in the creation of the execution context for a function call is the creation of an arguments object, which is an array-like object with integer indexed members corresponding with the arguments passed to the function call, in order. It also has length and callee properties (which are not relevant to this discussion, see the spec for details). A property of the Activation object is created with the name "arguments" and a reference to the arguments object is assigned to that property.

Next the execution context is assigned a scope. A scope consists of a list (or chain) of objects. Each function object has an internal [[scope]] property (which we will go into more detail about shortly) that also consists of a list (or chain) of objects. The scope that is assigned to the execution context of a function call consists of the list referred to by the [[scope]] property of the corresponding function object with the Activation object added at the front of the chain (or the top of the list).

Then the process of "variable instantiation" takes place using an object that ECMA 262 refers to as the "Variable" object. However, the Activation object is used as the Variable object (note this, it is important: they are the same object). Named properties of the Variable object are created for each of the function's formal parameters, and if arguments to the function call correspond with those parameters the values of those arguments are assigned to the properties (otherwise the assigned value is undefined). Inner function definitions are used to create function objects which are assigned to properties of the Variable object with names that correspond to the function name used in the function declaration. The last stage of variable instantiation is to create named properties of the Variable object that correspond with all the local variables declared within the function.

The properties created on the Variable object that correspond with declared local variables are initially assigned undefined values during variable instantiation, the actual initialisation of local variables does not happen until the evaluation of the corresponding assignment expressions during the execution of the function body code.

It is the fact that the Activation object, with its arguments property, and the Variable object, with named properties corresponding with function local variables, are the same object, that allows the identifier arguments to be treated as if it was a function local variable.

Finally a value is assigned for use with the this keyword. If the value assigned refers to an object then property accessors prefixed with the this keyword reference properties of that object. If the value assigned (internally) is null then the this keyword will refer to the global object.

The global execution context gets some slightly different handling as it does not have arguments so it does not need a defined Activation object to refer to them. The global execution context does need a scope and its scope chain consists of exactly one object, the global object. The global execution context does go through variable instantiation, its inner functions are the normal top level function declarations that make up the bulk of javascript code. The global object is used as the Variable object, which is why globally declared functions become properties of the global object. As do globally declared variables.

The global execution context also uses a reference to the global object for the this object.

## 闭包

## 闭包能做什么？

## 闭包的副作用

## Internet Explorer的内存泄漏问题


## 参考

1. [What is the Execution Context & Stack in JavaScript?](http://davidshariff.com/blog/what-is-the-execution-context-in-javascript/)
2. [Identifier Resolution and Closures in the JavaScript Scope Chain](http://davidshariff.com/blog/javascript-scope-chain-and-closures/) 
3. [ECMA-262-3 in detail. Chapter 1. Execution Contexts](http://dmitrysoshnikov.com/ecmascript/chapter-1-execution-contexts/)


[ECMAScript规范]: http://bclary.com/2004/11/07/