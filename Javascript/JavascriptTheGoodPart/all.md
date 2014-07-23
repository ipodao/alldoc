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

## 3 对象

Javascript的简单类型包括数字、字符串、布尔值（true和false）、`null`值和`undefined`值。其他所有值都是对象。

数字、字符串和布尔值“貌似”是对象，因为它们拥有方法，但它们是不可变的。Javascript中的对象是可变的键值集合。在Javascript中，数组是对象、函数是对象、正则表达式是对象。

对象是属性的容器，其中每个属性都有名字和值。属性的名字可以是包含空字符串在内的任意字符串。属性值可以是除`undefined`值之外的任何值。

Javascript中的对象没有类。

JavaScript includes a prototype linkage feature that allows one object to inherit the properties of another. 如果使用正确，它可以减少初始化时间和内存消耗。

### 3.1 对象字面量

对象字面量是一对花括号内的零个或多个“名值对”。

```js
var empty_object = {};

var stooge = {
    "first-name": "Jerome",
    "last-name": "Howard"
};
```

在对象字面量中，若属性名是一个合法的Javascript标识符且不是保留字，可以不使用引号括住属性名 。例如，`"first–name"`的引号是必需的，`first_name`可以不用引号。

对象可以嵌套：

```js
var flight = {
    airline: "Oceanic",
    number: 815,
    departure: {
        IATA: "SYD",
        time: "2004-09-22 14:55",
        city: "Sydney"
    },
    arrival: {
        IATA: "LAX",
        time: "2004-09-23 10:42",
        city: "Los Angeles"
    }
};
```

### 3.2 检索

利用`.`或中括号。如果字符串是一个合法的Javascript标识符且非保留字，可以使用`.`否则应使用中括号。

```js
stooge["first-name"] // "Joe"
flight.departure.IATA // "SYD"
```

如果检索的属性不存在，返回`undefined`。注意不是`null`。

```js
stooge["middle-name"]    // undefined
flight.status            // undefined
stooge["FIRST-NAME"]     // undefined
```

`||`可用于帮助填充值：

```js
var middle = stooge["middle-name"] || "(none)";
var status = flight.status || "unknown";
```

尝试从`undefined`取值将抛出TypeError异常。可以使用`&&`预作判断。

```js
flight.equipment                              // undefined
flight.equipment.model                        // throw "TypeError"
flight.equipment && flight.equipment.model    // undefined
```

### 3.3 更新

直接赋值。

```js
stooge['first-name'] = 'Jerome';
```

如果对象之前没有某个属性值，赋值将添加该属性。

### 3.4 引用

对象通过引用来传递。它们永远不会被拷贝。

### 3.5 原型

每个对象都链接（linked）到一个原型对象，并且它可以从中继承属性。所有通过对象**字面量**创建的对象都链接到`Object.prototype`——JavaScript标准中的对象。

创建对象时可以选择某个对象做它的原型。Javascript本身提供的机制杂乱又复杂，可以被显著简化。我们将给Object对象增加一个`beget`方法。使用该对象可以将一个对象作为原型创建新对象。

```js
if (typeof Object.beget !== 'function') {
     Object.beget = function (o) {
         var F = function () {};
         F.prototype = o;
         return new F();
     };
}
var another_stooge = Object.beget(stooge);
```

原型链接不影响更新。当改变对象时，原型对象不会受影响。

```js
another_stooge['first-name'] = 'Harry';
another_stooge['middle-name'] = 'Moses';
another_stooge.nickname = 'Moe';
```

试验：

```js
var stooge={}
stooge['first-name'] = 'A';
var another_stooge = Object.beget(stooge);
another_stooge['first-name'] = 'B';

another_stooge['first-name']; //’B’
stooge['first-name']; //’A’
```

原型链接只对检索值有效：尝试获得某个对象的属性值，若该对象本身没有该属性，从它的原型中找属性，如果没有再从原型的原型中找。如果`Obejct.prototype`也没找到，返回`undefined`。该过程称为委托（delegation）。

原型关系是一种动态的关系。如果我们添加一个新的属性到原型中。该属性会立即对所有基于该原型创建的对象可见。

```js
stooge.profession = 'actor';
another_stooge.profession    // 'actor'
```

### 3.6 反射

检查对象有什么属性，可以通过检索该属性并验证取得的值。

`typeof`操作符可以用于确定属性的类型：

```js
typeof flight.number     // 'number'
typeof flight.status      // 'string'
typeof flight.arrival     // 'object'
typeof flight.manifest   // 'undefined'
```

注意原型链中的有些属性也会产生一个值，但它们是函数（函数产生值）：

````js
typeof flight.toString    // 'function'
typeof flight.constructor // 'function'
```

有些值是函数（产生的），可以通过`typeof`去除。

`hasOwnProperty()`可以确认对象自身（注意Own这个词）是否有某个属性（返回true）。`hasOwnProperty()`不检查原型链。

```js
flight.hasOwnProperty('number')       // true
flight.hasOwnProperty('constructor')    // false
```

注意如果flight对象定义有方法fun1()，则flight.hasOwnProperty(‘func1’)返回true！

### 3.7 枚举

**for in** 语句用来遍历一个对象的所有属性名。该枚举过程会列出所有属性——包括**函数**和原型中的属性。可以使用`hasOwnProperty()`方法过滤掉原型中的属性，在此基础上使用`typeof`排除函数：

```js
var name;
for (name in another_stooge) {
    if (typeof another_stooge[name] !== 'function') {
        document.writeln(name + ': ' + another_stooge[name]);
    }
}
```

> 属性名的出现顺序是不确定的。


### 3.8 删除

`delete`运算符用于删除对象的属性。它不会触及原型链中的任何对象。

删除属性可能让原型链中的属性浮现出来。

```js
another_stooge.nickname    // 'Moe'
// Remove nickname from another_stooge, revealing the nickname of the prototype.
delete another_stooge.nickname;
another_stooge.nickname    // 'Curly'
```

### 3.9 减少全局变量污染

最小化全局变量的方法是在程序中只创建一个全局变量，让此变量成为应用的容器：

```js
var MYAPP = {};

MYAPP.stooge = {
    "first-name": "Joe",
    "last-name": "Howard"
};

MYAPP.flight = {
    airline: "Oceanic",
    number: 815,
    departure: {
        IATA: "SYD",
        time: "2004-09-22 14:55",
        city: "Sydney"
    },
    arrival: {
        IATA: "LAX",
        time: "2004-09-23 10:42",
        city: "Los Angeles"
    }
};
```

闭包也可以有效的减少全局污染，见下章。

## 4 函数

### 4.1 函数对象

对象字面量产生的对象连接到`Object.prototype`。而函数对象连接到`Function.prototype`（该原型对象本身连接到`Object.prototype`）。每个函数在创建时附有两个附加的隐藏属性：函数的上下文和实现函数行为的代码（the function's context and the code that implements the function's behavior）。

译注：Javascript创建一个函数对象时，会给该对象设置一个“调用”属性。当Javascript调用一个函数时，可以理解为调用此函数的“调用”属性。

Every function object is also created with a `prototype` property. Its value is an object with a constructor property whose value is the function（就是这个函数本身）. This is distinct from the hidden link to `Function.prototype`. 这个令人费解的构造过程将在下一节展示。

因为函数是对象，**函数可以存放在变量中，作为参数传递**。函数可以返回函数。因为函数也是对象，因此**函数可以拥有方法**。

### 4.2 函数字面量

通过函数字面量创建函数：

```js
var add = function (a, b) {
    return a + b;
};
```

函数字面量包含四个部分。

- 第一部分是保留字function。
- 第二部分是函数名，可以被省略。如果没有名字，函数是匿名函数，如上面的例子。
- 第三部分是参数列表。
- 第四部分是包含为大括号内的语句。

函数定义可以嵌套。内部函数可以访问到外围函数的**参数和变量**。通过函数字面量创建的函数对象包含到外部上下文的连接，这被称为 **闭包** 。这是Javascript强大表现力的根基。
