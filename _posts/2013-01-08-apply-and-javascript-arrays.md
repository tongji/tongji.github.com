---
layout: post
title: "Apply 和 JavaScript 数组"
description: ""
category : jslang
tags : [JavaScript, apply, array, jslang, dev]
author :
  name : zhangxin
---
{% include JB/setup %}

前端开发工程师应该已经都很熟悉[apply](http://ecma262-5.com/ELS5_HTML.htm#Section_15.3.4.3)的这种使用方法，但是当`apply`遇到数组时，还会有其它一些你可能不知道的使用方法。

## apply 方法

apply是一个定义在Function.prototype上的方法，所以所有的function都可以使用它。
它接受第一个参数作为调用上下文中this的值，第二个参数接受数组类型（或者是类数组的对象，比如arguments）。

使用方法：

    func.apply(thisValue, [arg1, arg2, ...])

上面的这种调用方式等价于：
    
    thisValue.tmp = func
    thisValue.tmp(arg1, arg2, ...)
    delete thisValue.tmp

tmp是一个临时属性，如果不考虑thisValue, 等价于：

    func(arg1, arg2, ...)

从上面可以看出，apply让我们可以以数组类型做为函数调用的参数。下面介绍一下apply与数组的三种tricks。

## 1. 让某些函数接受数组类型参数

在JavaScript中，没有一个函数可以获取一个数值数组中最大的值，但是Math.max接受0个或任意多个number值，然后返回参数中最大的值，但是它并不以数组类型的值作为参数。但是，我们可以通过`apply`来达到我们的目的：

    Math.max.apply(null, [100, 10, 55]) // 100

等价于：

    Math.max(100, 10, 55) // 100

由于Math.max是静态的函数，它并不关心this值，所以我们传入null也没有关系。

## 2. 消除稀疏数组

### 稀疏数组(sparse array)

在JavaScript上，数组其实也是一种从number到value的map，只不过它的key是number而已。并且数组中的某些项有可能为空（产生hole）或者
某个数组元素的值为`undefined`，但是在某些数组的遍历方法中（如forEach, map, etc.），前者会被忽略，而后者不会：


    > ["a", , "b"].forEach(function (x) { console.log(x) })
    a
    b
    
    > ["a", undefined, "b"].forEach(function (x) { console.log(x) })
    a
    undefined
    b

可以使用`in`操作符检查数组中是否存在hole：

    > 1 in ["a", ,"b"]
    false
    > 1 in ["a", undefined, "b"]
    true

但是如果读取hole的值，与`undefined`元素的结果一样，它的值是undefined。

    > ["a", ,"b"][1]
    undefined
    > ["a", undefined, "b"][1]
    undefined

### 消除holes

我们可以利用`apply`和[`Array`](http://ecma262-5.com/ELS5_HTML.htm#Section_15.4.1)将这些holes转化为值为`undefined`的数组元素：

    > Array.apply(null, ['a', , 'b'])
    ['a', undefined, 'b']

`apply`并不会忽略holes，相反，它会读取它的值(`undefined`)，然后将它当作参数传递给Array。并且，你也可以自定义该function以替代Array。

    > function returnArgs() { return [].slice.call(arguments) }
    > returnArgs.apply(null, ["a",,"b"])
    [ 'a', undefined, 'b' ]

但是，这里有一个例外。如果`Array`接收的是一个number类型的参数，那么就会创建一个长度为该数值的空数组:

    > Array.apply(null, [ 3 ])
    [ , ,  ]

因此，如果你需要这个功能，最好自己去实现，由于hole被数组的遍历方法忽略，所以我们可以这样做：

    function fillHoles(arr) {
        var result = [];

        for(var i=0; i < arr.length; i++) {
            result[i] = arr[i];
        }
        return result;
    }

    > fillHoles(["a",,"b"])
    [ 'a', undefined, 'b' ]


[underscore][underscore]的[compact](http://underscorejs.org/#compact)方法会移除所有假值（包括 false, null, 0, "", undefined and NaN）:

    > _.compact(["a",,"b"])
    [ 'a', 'b' ]
    > _.compact(["a", undefined, "b"])
    [ 'a', 'b' ]
    > _.compact(["a", false, "b"])
    [ 'a', 'b' ]

## 3. flatten arrays

`apply`还可以帮你把元素嵌套的二维数组变为一维数组，我们可以利用`apply`提供的unwrap功能来让`concat`帮助我们实现：

    > Array.prototype.concat.apply([], [['a'], ['b']])
    ['a', 'b']

`concat`也支持非数组元素：

    > Array.prototype.concat.apply([], [["a"], "b"])
    ['a', 'b']

`apply`的`this`值必须是[]，因为`concat`是数组的实例方法，并不单单是一个函数。再看下面的例子：

    > Array.prototype.concat.apply([], [[["a"]], ["b"]])
    [ [ 'a' ], 'b' ]

它只能完成一个层级的flatten，并不支持任意深度的嵌套数组。

但是[underscore][underscore]的中的[flatten](http://underscorejs.org/#flatten)方法支持任意深度的嵌套数组:
    
    > _.flatten([[["a"]], ["b"]])
    [ 'a', 'b' ]

## 参考链接

* [Apply and arrays: three tricks](http://www.2ality.com/2012/07/apply-tricks.html)
* [underscore](http://underscorejs.org/)


[underscore]: http://underscorejs.org/ 'underscore'