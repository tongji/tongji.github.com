---
layout: post
title: "JavaScript Types"
description: "JavaScript Types"
category : jslang
tags : [JavaScript， types， jslang， dev]
---
{% include JB/setup %}

## Agenda

1. 常用类型；  
2. 常用类型之间的隐式和显示转换；  
3. 类型转换的性能开销测试

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

### Primitive Types

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

<table class="neat">
    <thead>
        <tr><th class='title'>Type</th><th class='title'>Result</th></tr>
    </thead>
    <tbody>
        <tr><td>Undefined</td><td>'undefined'</td></tr>
        <tr><td>Null</td><td class="red">'object'</td></tr>
        <tr><td>Boolean</td><td>'boolean'</td></tr>
        <tr><td>Number</td><td>'number'</td></tr>
        <tr><td>String</td><td>'string'</td></tr>
        <tr><td>Object</td><td>'object'</td></tr>
        <tr><td>Array</td><td class="red">'object'</td></tr>
        <tr><td>Date</td><td>'object'</td></tr>
        <tr><td>RegExp</td><td>'object'</td></tr>
        <tr><td>Function</td><td>'function'</td></tr>
    </tbody>
</table>

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


## constructor property

    function Foo(){}
    var f1 = new Foo();
    Foo.prototype.constructor === Foo; //true
    f1.constructor === Foo; //true

* ![js-object-layout](/img/js-object-layout.jpg)
* [ECMA-262 13.2 Creating Function Objects](http://bclary.com/2004/11/07/#a-13.2)


## valueOf && toString

* 所有的对象都会从Object的原型（Object.prototype）上继承valueOf和toString；
* Object.prototype.valueOf()方法只是简单的返回对象本身；
* Object.prototype.toString()方法默认会返回'[object' + [[Class]] + ']'；
* 但是，我们经常需要overriden它们默认的行为
* [valueOf && toString sample](http://jsfiddle.net/starandtina/3VPxq/)
* [15.2.4.4 Object.prototype.valueOf()](http://bclary.com/2004/11/07/#a-15.2.4.4)
* [ECMA-262 15.2.4.2 Object.prototype.toString](http://bclary.com/2004/11/07/#a-15.2.4.2)
* [ECMA-262 15.3.4 Properties of the Function Prototype Object](http://bclary.com/2004/11/07/#a-15.3.4)
* [ECMA-262 15.4.4 Properties of the Array Prototype Object](http://bclary.com/2004/11/07/#a-15.4.4)


## Conversion Between Types

* 类型转换是在runtime时做的；
* 隐式类型转换常常发生在操作符中（包括算术操作符、关系操作符等等），一般会调用valueOf或toString；
* Number、Boolean、String、Object以函数方式调用时，会进行显示类型转换
    * ToPrimitive: 用于将对象类型转换为原生类型
    * ToNumber: 将输入转换为Number类型；
    * ToBoolean: 将输入转换为Boolean类型；
    * ToString: 将输入转换为String类型；
    * ToObject: 将原生类型转换为对象类型


## Convert to Primitive

ToPrimitive内部方法主要用于将对象类型转换为原生类型.

    function ToPrimitive(x， hint) {
      if (!IS_SPEC_OBJECT(x)) return x；
      if (hint == NO_HINT) hint = (IS_DATE(x)) ? STRING_HINT : NUMBER_HINT；
      return (hint == NUMBER_HINT) ? %DefaultNumber(x) : %DefaultString(x)；
    }

* 如果x不是Object，直接返回它；
* hint参数的默认值是NUMBER_HINT，但是如果x是Date，那么hint为STRING_HINT；
* 依据hint参数，调用内部[[DefaultValue]]方法，从而获取x的primitive value


## Convert to Primitive

### [[[DefaultValue]](hint)](http://bclary.com/2004/11/07/#a-8.6.2.6)

* 每个对象都有一个内部方法[[DefaultValue]]；
* 如果hint为NUMBER_HINT

        1. 如果存在valueOf方法且可调用，那么立即执行它，否则跳到3；
        2. 如果Result(1)返回是primitive value，则返回该值；
        3. 如果存在toString方法且可调用，那么立即执行它，否则跳到5；
        4. 如果Result(3)返回的是primitive value，则返回该值；
        5. 抛出TyperError

* 如果hint为STRING_HINT，过程与上述相反；
* [ToPrimitive sample](http://jsfiddle.net/starandtina/hanKB/)
* [ECMA-262 9.1 ToPrimitive](http://bclary.com/2004/11/07/#a-9.1)


## Convert to Number

    function ToNumber(x) {
      if (IS_NUMBER(x)) return x；
      if (IS_STRING(x)) {return %_HasCachedArrayIndex(x) ? %_GetCachedArrayIndex(x) : %StringToNumber(x)；}
      if (IS_BOOLEAN(x)) return x ? 1 : 0；
      if (IS_UNDEFINED(x)) return $NaN；
      return (IS_NULL(x)) ? 0 : ToNumber(%DefaultNumber(x))；
    }

* 如果参数x是Number，返回x；
* 如果参数x是String，那么尽可能转化为Number，否则为NaN（但是空字符串会转换为0）；
* 如果参数x是Boolean，那么true->1，false->0；
* 如果参数X是Undefined，那么返回NaN； 
* 如果参数x是Null，那么返回0；
* 如果参数x是Object，则会调用ToNumber(ToPrimitive(x， NUMBER_HINT))


## Convert to Number 

    var numberValue = stringValue * 0；
    var numberValue = stringValue * 1；
    var numberValue = stringValue / 1；

    var numberValue = +stringValue；
    var numberValue = Number(x)；

* [ToNumber sample](http://jsfiddle.net/starandtina/X4dWv/)
* [jsperf Compare](http://jsperf.com/js-convert-to-number/2)
* [ECMA-262 9.2 ToNumber](http://bclary.com/2004/11/07/#a-9.3)


## Convert to Boolean

    function ToBoolean(x) {
      if (IS_BOOLEAN(x)) return x；
      if (IS_STRING(x)) return x.length != 0；
      if (x == null) return false；
      if (IS_NUMBER(x)) return !((x == 0) || NUMBER_IS_NAN(x))；
      return true；
    }

* 如果参数x是Boolean，返回x；
* 如果参数x是String，那么空字符串会转换为false(但是" "并不会转换为false)；
* 如果参数x是Undefined或Null，那么返回false； 
* 如果参数x是Number，那么0和NaN返回false，其它返回true；
* 如果参数x是Object，那么返回ture


## Convert to Boolean

    var boolValue = !!x；
    var boolValue = Boolean(x)；

* [ToBoolean sample](http://jsfiddle.net/starandtina/uW8Ln/)
* [ECMA-262 9.2 ToBoolean](http://bclary.com/2004/11/07/#a-9.2)
* [jsperf Compare](http://jsperf.com/js-convert-to-boolean)


## Convert to String

    function ToString(x) {
      if (IS_STRING(x)) return x；
      if (IS_NUMBER(x)) return %_NumberToString(x)；
      if (IS_BOOLEAN(x)) return x ? 'true' : 'false'；
      if (IS_UNDEFINED(x)) return 'undefined'；
      return (IS_NULL(x)) ? 'null' : %ToString(%DefaultString(x))；
    }

* 如果参数x是String，返回x；
* 如果参数x是Number，NaN –>'NaN'， Infinity ->'Infinity'；
* 如果参数x是Boolean，true->'true'，false->'false'； 
* 如果参数x是Undefined，返回'undefined'；
* 如果参数x是Null，那么返回'null'；
* 如果参数x是Object，则会调用ToString(ToPrimitive(x， STRING_HINT))


## Convert to String

    var stringValue = value + ''；
    var stringValue = String(value)；

* [ToString sample](http://jsfiddle.net/starandtina/g3tUX/)
* [ECMA-262 9.2 ToString](http://bclary.com/2004/11/07/#a-9.8)
* [jsperf Compare](http://jsperf.com/js-convert-to-string/2)


## Convert to Object

与ToPrimive相对的就是ToObject，它主要用于将原生类型转换为对象类型。

    function ToObject(x) {
      if (IS_STRING(x)) return new $String(x)；
      if (IS_NUMBER(x)) return new $Number(x)；
      if (IS_BOOLEAN(x)) return new $Boolean(x)；
      if (IS_NULL_OR_UNDEFINED(x) && !IS_UNDETECTABLE(x)) {
        throw %MakeTypeError('null_to_object'， [])；
      }
      return x；
    }

* ToObject相当于一个工厂函数；
* 如果x是Null或Undefined，则会抛出TypeError异常；
* 如果x是String、Number或Boolean，则会调用相应类型的构造函数；


## Convert to Object

当在primitive type上调用方法时，js会自动将它转换为与之对应的Object
    
    var obj = Object(value);
    var obj = new Object(value);

* [ToObject sample](http://jsfiddle.net/starandtina/DxUst/)
* [ECMA-262 9.9 ToObject](http://bclary.com/2004/11/07/#a-9.9)
* [ECMA-262 15.2 Object Objects](http://bclary.com/2004/11/07/#a-15.2)


## Implicit Type Conversion

    var obj = {
        toString:function(){
            return 'Holmes';
        }，
        valueOf:function(){
            return 1;
        }
    };
     
    alert([obj， 'FE'].join(' '));//Holmes FE
    alert(obj + 'Holmes'); //1Holmes


## Side Effect Of Implicit Type Conversion

    var obj = {
        toString:function(){
            return 1;
        }，
        valueOf:function(){
            return 'Holmes';
        }
    };
     
    alert([obj， 'FE'].join(' '));//1 FE
    alert(obj + 'Holmes'); //Holmes Holmes

* js并不保证toString方法一定会返回string，而valueOf方法也可以随意返回相应类型的primitive value；
* 尽量使用String(obj)，而不是obj.toString()，因为它可以尽量保证返回的肯定是string(ToPrimitive时，也有可能抛出TypeError)


## Conclusions

* 尽量使用直接进行类型转换，不要依赖于js内部的类型转换；
* 实现自己的valueOf和toString方法；


## Reference

* [V8 runtime.js](http://code.google.com/p/v8/source/browse/trunk/src/runtime.js)
* [Javascript Type-Conversion](http://jibbering.com/faq/notes/type-conversion/)
* [Object-to-Primitive Conversions in JavaScript](http://www.adequatelygood.com/2010/3/Object-to-Primitive-Conversions-in-JavaScript)
* [ECMA-262-3 in detail. Chapter 7.2. OOP: ECMAScript implementation](http://dmitrysoshnikov.com/ecmascript/chapter-7-2-oop-ecmascript-implementation/#built-in-native-and-host-objects)