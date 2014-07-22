[toc]

## 1. 精华

### 1.1 为什么要使用JavaScript

浏览器的API和文档对象模型（DOM）相当糟糕，导致Javascript遭到不公平指责。在任何语言中处理DOM都是一件痛苦的事情，它的规范制定的拙劣且实现不一致。本书很少涉及DOM，我认为写一本关于DOM的精华的书是不可能完成的任务。

### 1.2 分析JavaScript

Javascript中好的想法包括函数、弱类型、动态对象、对象字面量表示法。

JavaScript's functions are first class objects with (mostly) lexical scoping.
译注3：Javascript中的函数根据词法来划分作用域，而不是动态地划分作用域。参见《Javascript权威指南》第5版，8.8.1，词法作用域。

Javascript是第一个主流的lambda语言。

Javascript的原型继承（prototypal inheritance）是一项有争议的特性。Javascript的对象系统没有类的概念，对象直接从其他对象继承属性。

Javascript有一项特性是非常糟糕的：JavaScript depends on global variables for linkage. All of the top-level variables of all compilation units are tossed together in a common namespace called the global object. This is a bad thing because global variables are evil, and in JavaScript they are fundamental. Fortunately, as we will see, JavaScript also gives us the tools to mitigate this problem.

### 1.3 一个简单的试验场

本书自始至终将使用下面的method方法定义新方法：

```js
Function.prototype.method = function (name, func) {
    this.prototype[name] = func;
    return this;
};
```

第4章将详细解释。

## 2 语法

### 2.1 空白

Javascript提供两种注释：`/* */`和`//`。

### 2.2 标识符

保留字：
```
abstract
boolean break byte
case catch char class const continue
debugger default delete do double
else enum export extends
false final finally float for function
goto
if implements import in instanceof int interface
long
native new null
package private protected public
return
short static super switch synchronized
this throw throws transient true try typeof
var volatile void
while with
```

### 2.3 数字

Javascript只有一种数字类型。内部表示为64位的浮点数。没有单独的整数，因此1和1.0是相同值。

> Javascript没有整数类型

指数形式，如`1e2`，值是100。

值`NaN`是一个数值。它表示一个不能产生正常结果的运算结果。`NaN`不等于任何值，包括它自己。可以使用函数`isNaN(number)`检测`NaN`。

> 判断一个数字`number`是不是`NaN`，须用`isNaN(number)`。不能用`number === NaN`

值`Infinity`表示所有大于`1.79769313486231570e+308`的值。

数字拥有方法（参见第8章）。Javascript有一个对象Math，包含一套用于数字的方法。例如`Math.floor(number)`方法将一个数字转换成一个整数。

### 2.4 字符串

字符串字面量可以被包围在单引号或双引号中。\是转义字符。

由于创建Javascript的时候Unicode是一个16位的字符集，所有Javascript中所有字符都是16位的。

Javascript没有字符类型（至哟字符串）。

字符串有一个`length`属性。如`”seven”.length`。

字符串是不可变的。

可以使用`+`连接字符串。下面的表达式值为true。
```js
'c' + 'a' + 't' === 'cat'
```

字符串的方法见第8章。
```js
'cat'.toUpperCase() === 'CAT'
```

### 2.5 语句

定义变量：

```js
var a;
var a, b;
var a = 1, b;
```

在web浏览器中，每个`<script>`标签都是一个编译单元，被编译并立即执行。由于缺少连接器，Javascript把它们一起抛入一个公共的全局名字空间中。附录A有更多关于全局变量的内容。

switch, while, for, do语句可以带一个前置标签，供break语句使用。

控制语句：if, switch, while, for, do, break, return, throw。

下列值被当作false：

- false
- null
- undefined
- 空字符串
- 数字0
- 数字NaN

其他值都被当作true，包括true、字符串”false”、空数组`[]`，空对象`{}`，以及所有对象。

switch语句：表达式可以产生数字或字符串。
如果没有找到匹配执行default。
一个case子句可以包含一个或多个case表达式。case表达式不一定是常量。为防止继续执行下一个case，case语句之后应跟随一个强制跳转语句，如break。

for循环有两个形式。一种是

```js
for(initializtion;condition;increment){}
```

另一种是for-in。枚举对象的所有属性名（或键名）。

通常需要检查`object.hasOwnProperty(varibale)`来确定这个属性名就是该对象的成员，还是从其原型链中得到的。

```js
for (myvar in obj) {
    if (obj.hasownProperty(myvar)) {
        ...
    }
}
```

try–catch

throw语句中的表达式通常为一个对象字面量，它包含一个name属性和message属性。异常捕获后可以获取这些属性。

```js
try {
	throw {name: "xx", message: "xxx"};
} catch(e) {
	console.log(e.name);
}
```

return语句若不指定返回值，返回`undefined`。


### 2.6 表达式

> 不要死记运算符优先级。所有有歧义的地方直接加括号！

`typeof`运算符的值有'number', 'string', 'boolean', 'undefined', 'function', 'object'。第6章和附录A有更多关于`typeof`的内容。

> 数组或null的`typeof`结果是’object’！

`/`可能产生非整数结果（即使两个运算符都是整数）。

如果第一个运算数的值为假，那么运算符`&&`的结果为第一个运算符的值，否则取第二个运算符的值。

### 2.7 字面量

字面量包括数字字面量、字符串字面量、对象字面量、数组字面量、函数字面量、正则表达式字面量。

对象字面量，如：`{a: 1, b: true}`。数组字面量，如：`[1, 2, 'a', 4]`。

正则表达式字面量被两个斜杠包围。如：`/(.*)[a-z]/`。

