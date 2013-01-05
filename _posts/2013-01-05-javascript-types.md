---
layout: post
title: "JavaScript Types"
description: ""
category : jslang
tags : [JavaScript， types， jslang， dev]
---
{% include JB/setup %}

## Agenda

* 常用类型
* 常用类型之间的隐式和显示转换
* 类型转换的性能开销测试

## Quiz

        null >= 0;
        null == 0;
        1 + {};
        {} + 1;
        var o = {
            valueOf: function () {
                return 1;
            }
        };
        ++o;
        var o = {
            toString: function () {
                return 1;
            }
        };
        ++o;
        var o = {
            valueOf: function () {
                return 'hello';
            }，
            toString: function () {
                return 1;
            }
        };
        ++o;


## JavaScript Types

### Primitive Type:

* Null: null
* Undefined: undefined
* Number: 3.1415926， 1e+6， NaN， Infinity， etc.
* Boolean: true and false
* String: \'hello\'， \'\'， \' \'， etc.

### Everything else is Object:

* Built-in Object: Number， Boolean， String， Array， Function， Object， Error， Global(parseInt， NaN， undefined， etc.)， Math
* Host Object: BOM， DOM


## 类型判断

* typeof Operator
* instanceof Operator
* constructor Property

### typeof Operator

* 尽管`typeof(null) === "object"`，但是它仍然是一个primitive value，但是ECMAScript.next中有可能[`typeof(null) === "null"`](http://wiki.ecmascript.org/doku.php?id=harmony:typeof_null)
* [ECMA-262 11.4.3 typeof](http://bclary.com/2004/11/07/#a-11.4.3)


## instanceof Operator

    x instanceof Constructor

如果Constructor.prototype在x原型(__proto__)链上，那么就返回true

    function Foo(){}
    var f1 = new Foo();
    f1 instanceof Foo; //true
    Foo instanceof Object; //true
    Object instanceof Function;//true
    Function instanceof Object; //true
    Function instanceof Function;//true

* instanceof运算符与typeof运算符相似，用于识别正在处理的对象的类型111
* 与typeof方法不同的是，instanceof要求开发者明确地确认对象为某特定类型
* [Cross-Frame Issue](http://jsfiddle.net/starandtina/VQ3NC/)