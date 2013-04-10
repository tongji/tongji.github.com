---
layout: post
title: "JavaScript 闭包"
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

当创建一个执行上下文时，会按顺序进行如下的操作：

1. 首先，在函数的上下文中，会创建一个`激活对象`。激活对象是一种与规范相关的内部实现。你可以把它当做一个对象，是因为它也拥有可访问的命名属性，但是它与普通的对象不同的地方在于，它并没有prototype，并且你不能通过JavaScript代码直接访问；

2. 接下来的一步是创建`arguments`对象。`arguments`是一种类数组的整数数值索引对象，它的值与实参一一对应。`arguments`也有拥有[`length`和`callee`属性](http://bclary.com/2004/11/07/#a-10.1.8)。初始化`激活对象`时会创建一个`arguments`属性，其值为该'arguments'对象；

3. 接下来，会创建函数执行上下文的作用域链，该作用域链上包括一系列的对象。每一个函数对象都有一个内部的`[[scope]]`属性，函数执行上下文的作用域链在函数调用时创建出来，它包括了`激活对象`和该`[[scope]]'属性上的对象，并且`激活对象`是被添加在链的最前端的；

4. 之后就是[变量初始化](http://bclary.com/2004/11/07/#a-10.1.3)的过程。要注意的是，这个阶段是通用的行为，与上下文类型无关（不管是全局上下文还是函数上下文都是一致的）。在执行代码之前，进入函数执行上下文，`激活对象`会充当变量对象，同时被一些属性填充：


        1. 函数的形参：变量对象的一个属性，其属性名就是形参的名字，其值就是实参的值；对于没有传递的参数，其值为undefined；
        2. 函数声明：变量对象的一个属性，其属性名和值都是函数对象创建出来的；如果变量对象已经包含了相同名字的属性，则替换它的值；
        3. 变量声明：变量对象的一个属性，其属性名即为变量名，其值为undefined;如果变量名和已经声明的函数名或者函数的参数名相同，则不会影响已经存在的属性。

`变量对象`上的那些与本地声明变量有关的属性，都只是初始化时候的`undefined`。直到代码执行阶段，它们才会真正的初始化。

5. 最后是确定`this`值。一般是由函数的caller提供`this`，如果由call所提供的`this`不是对象（`null`也不是对象），那么`this`则指向全局对象。

全局的执行上下文有一点不同的是它并没有`arguments`对象，所以它并不需要`激活对象`来引用它们。全局的执行上下文也是需要一个作用域链并且它的作用域链只包含一个对象，即全局对象。全局的执行上下文也会经历变量初始化阶段，它的内部函数定义是最本顶级的函数声明，由这些构成了JavaScript代码。在变量初始化阶段，全局对象会被当做`变量对象`，这也就是为什么全局的函数声明会被当做全局对象的属性的原因，全局变量声明也类似。

全局执行上下文同时也使用了一个指向全局对象的引用来表示`this`。

### 作用域链 和 [[scope]]

作用域链与内部函数息息相关，它其实就是所有内部上下文的变量对象列表。函数调用执行上下文的作用域链是在函数调用的时候创建出来的，它包含了活跃对象和该函数的内部`[[scope]]`属性。

在ECMAScript中，函数也是对象，它们可以在变量初始化时通过函数声明创建出来，或者是通过执行函数表达式或Function constructor创建。、
使用Function constructor创建的函数对象始终会有一个只包含全局对象的`[[scope]]`属性。
使用函数声明或函数表达式创建的函数对象，它的`[[scope]]`属性为它所在的执行上下文的作用域链。
以最简单的全局函数声明为例：

    function exampleFunction(formalParameter){
        ...   // function body code
    }


该函数对象是在全局执行上下文的变量初始化阶段创建的，而全局执行上下文的作用域链只包含全局对象。因此创建该函数对象时，它的`[[scope]]`属性只包含它所在的执行上下文中的全局对象。

A similar scope chain is assigned when a function expression is executed in the global context:-
在全局上下文中执行函数表达式，也是类似的：

    var exampleFuncRef = function(){
        ...   // function body code
    }

唯一不同的地方在于，该函数表达式的属性是在变量初始化阶段创建，但是它的该函数对象并没有真创建，直到函数执行阶段。

在函数上下文中所创建的内部函数声明和函数表达式对象会拥有更复杂的作用域链。看看下面的代码，它定义了一个拥有内部函数的函数，之后立即执行该函数：
    function exampleOuterFunction(formalParameter){
        function exampleInnerFuncitonDec(){
            ... // inner function body
        }
        ...  // the rest of the outer function body.
    }

    exampleOuterFunction( 5 );

外部函数声明所创建的函数对象是在全局上下文中创建的，所以它的`[[scope]]`属性只包含全局对象一个。

当执行该外部函数时，与内部函数相关的一个新的函数执行上下文会被创建，并伴随着一个`激活对象`。新的执行上下文的作用域链包含了该`激活对象`和外部函数的`[[scope]]`属性所指向的作用域链（只包含一个全局对象）。
在内部函数的变量初始化阶段，会创建与该内部函数声明相关的函数对象，并且该函数对象的`[[scope]]`属性会包含创建该内部函数的执行上下文的作用域链。

到目前为止，所有的这一切都是由代码的执行和结构自动控制的。上层执行上下文的作用域链定义了函数对象的`[[scope]]`属性，而`[[scope]]`属性定义了执行上下文的作用域链(与相应的激活对象一起)。同时ECMAScript也提供了`with`和`catch`去修改作用域链。

[`with`语句](http://bclary.com/2004/11/07/#a-9.9)会将它的参数当做一个表达式执行，如果该表达式可以转化为对象，则将它添加到当前执行上下文的作用域链的最前面，之后`with`会执行它所包含的语句（通过是一个block语句），结束执行后，会从作用域链中移除该表达式对象，恢复作用域链至之前的样子。

A function declaration could not be affected by a with statement as they result in the creation of function objects during variable instantiation, but a function expression can be evaluated inside a with statement:-
`with`中的函数声明是不会被`with`所影响的，因为它们是在变量初始化阶段就创建了函数对象，但是函数表达式则会受到影响，因为函数表达示的执行是在`with`中完成的。如下例：

    /* create a global variable - y - that refers to an object:- */
    var y = {x:5}; // object literal with an - x - property
    function exampleFuncWith(){
        var z;
        /* Add the object referred to by the global variable - y - to the
           front of he scope chain:-
        */
        with(y){
            /* evaluate a function expression to create a function object
               and assign a reference to that function object to the local
               variable - z - :-
            */
            z = function(){
                ... // inner function expression body;
            }
        }
        ... 
    }

    /* execute the - exampleFuncWith - function:- */
    exampleFuncWith();

当`exampleFuncWith`执行时，当前执行上下文的作用域链包含它的激活对象和全局对象。当执行内部的函数表达式`z`时，`with`的执行会将全局的`y`对象加入至作用域链的最前端。同时，由`z`所创建的函数表达式对象的`[[scope]]`属性则包含了创建它的执行上下文的作用域链，该作用域链包含了对象`[y, exampleFuncWith的激活对象, 全局对象]`。

当与`with`语句相关的block执行完毕时，作用域链会恢复至调用`with`前状态（移除`y`），但是此时`z`的作用域链中仍然包含'y'，并且始终在最前面。

### 标识符解析

Identifiers are resolved against the scope chain. ECMA 262 categorises this as a keyword rather than an identifier, which is not unreasonable as it is always resolved dependent on the this value in the execution context in which it is used, without reference to the scope chain.

Identifier resolution starts with the first object in the scope chain. It is checked to see if it has a property with a name that corresponds with the identifier. Because the scope chain is a chain of objects this checking encompasses the prototype chain of that object (if it has one). If no corresponding value can be found on the first object in the scope chain the search progresses to the next object. And so on until one of the objects in the chain (or one of its prototypes) has a property with a name that corresponds with the identifier or the scope chain is exhausted.

The operation on the identifier happens in the same way as the use of property accessors on objects described above. The object identified in the scope chain as having the corresponding property takes the place of the object in the property accessor and the identifier acts as a property name for that object. The global object is always at the end of the scope chain.

As execution contexts associated with function calls will have the Activation/Variable object at the front of the chain, identifiers used in function bodies are effectively first checked to see whether they correspond with formal parameters, inner function declaration names or local variables. Those would be resolved as named properties of the Activation/Variable object.

## 闭包

## 闭包能做什么？

## 闭包的副作用

## Internet Explorer的内存泄漏问题


## 参考

1. [What is the Execution Context & Stack in JavaScript?](http://davidshariff.com/blog/what-is-the-execution-context-in-javascript/)
2. [Identifier Resolution and Closures in the JavaScript Scope Chain](http://davidshariff.com/blog/javascript-scope-chain-and-closures/) 
3. [ECMA-262-3 in detail. Chapter 1. Execution Contexts](http://dmitrysoshnikov.com/ecmascript/chapter-1-execution-contexts/)


[ECMAScript规范]: http://bclary.com/2004/11/07/