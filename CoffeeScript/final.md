
[toc]

## 源

* 官方文档
* CoffeeScript: Accelerated JavaScript Development
	已整理：1、2


## 语言基础

CoffeeScript 使用空白符区分代码块。不需要使用分号分隔表达式。行尾就是分隔。（分号用于分隔一行上的多个表达式）。

使用缩进（而不是大括号）围绕 function、if、switch 和 try/catch 的代码块。

调用函数传参时不需要使用括号。The implicit call wraps forward to the end of the line or block expression.
`console.log sys.inspect object` → `console.log(sys.inspect(object));`

### 声明变量

The CoffeeScript compiler takes care to make sure that all of your variables are properly declared within lexical scope — 不要使用 `var` 关键字。

CoffeeScript的输出会被包裹进一个匿名函数：`(function(){ ... })()`。包裹与自动产生`var`关键字，避免了不小心污染全局命名空间的情况。

```javascript
outer = 1
changeNumbers = ->
  inner = -1
  outer = 10
inner = changeNumbers()

// ----->

var changeNumbers, inner, outer;

outer = 1;

changeNumbers = function() {
  var inner;
  inner = -1;
  return outer = 10;
};

inner = changeNumbers();
```

注意到，所有的变量声明都被提到最近作用域（变量首次出现的）的顶端。`outer`没有在函数中被重新声明，因为它已在作用域中了。

无法直接使用`var`，就无法掩盖外部变量。

If you'd like to create top-level variables for other scripts to use, attach them as properties on `window`, or on the `exports` object in CommonJS.

### 对象、数组字面量

定义对象、数组字面量时，若一行只写一项，则可以省略逗号。对象嵌套可以利用缩进创建（而不是括号），类似于YAML。
```javascript
song = ["do", "re", "mi", "fa", "so"]

singers = {Jagger: "Rock", Elvis: "Roll"}

bitlist = [
  1, 0, 1
  0, 0, 1
  1, 1, 0
]

kids =
  brother:
    name: "Max"
    age:  11
  sister:
    name: "Ida"
    age:  9

// ----->

var bitlist, kids, singers, song;

song = ["do", "re", "mi", "fa", "so"];

singers = {
  Jagger: "Rock",
  Elvis: "Roll"
};

bitlist = [1, 0, 1, 0, 0, 1, 1, 1, 0];

kids = {
  brother: {
    name: "Max",
    age: 11
  },
  sister: {
    name: "Ida",
    age: 9
  }
};
```

JavaScript中，如果保留字（如`class`）做属性名，需加引号。CoffeeScript可以帮你加引号。
```javascript
$('.account').attr class: 'active'

log object.class

// ----->

$('.account').attr({
  "class": 'active'
});

log(object["class"]);
```

### 运算符与别名

CoffeeScript 将 `==` 编译成 `===`，将 `!=` 编译成 `!==`。将 `is` 编译成 `===`，将 `isnt` 编译成`!==`。

`not` 是 `!` 的别名。

`and` 编译成 `&&`， `or` 编译成 `||`。

Instead of a newline or semicolon, `then` can be used to separate conditions from expressions, in while, if/else, and switch/when statements.

As in YAML, `on` and `yes` are the same as boolean `true`, while `off` and `no` are boolean `false`.

`unless` can be used as the inverse of `if`.

As a shortcut for `this.property`, you can use `@property`.

用 `in` 测试数组元素的存在性，用 `of` 测试Javascript对象键的存在性。

`**` 用于乘方（`Math.pow(a, b)`），`//` 用于整除（`Math.floor(a / b)`），and `%%` provides true mathematical modulo（`(a % b + b) % b`）.

```javascript
launch() if ignition is on

volume = 10 if band isnt SpinalTap

letTheWildRumpusBegin() unless answer is no

if car.speed < limit then accelerate()

winner = yes if pick in [47, 92, 13]

print inspect "My name is #{@name}"
```
```javascript
var volume, winner;

if (ignition === true) {
  launch();
}

if (band !== SpinalTap) {
  volume = 10;
}

if (answer !== false) {
  letTheWildRumpusBegin();
}

if (car.speed < limit) {
  accelerate();
}

if (pick === 47 || pick === 92 || pick === 13) {
  winner = true;
}

print(inspect("My name is " + this.name));
```

注意，`+`用于连接字符串时需要在两端留空格。下面的写法会报错：
```javascript
color = 'red'
x = color +5
```
因为`color +5`会被编译为`color(+5)`。因为前缀`+`是字符串转换为数字的捷径。


#### 链式比较

CoffeeScript borrows chained comparisons from Python — making it easy to test if a value falls within a certain range.

```javascript
cholesterol = 127
healthy = 200 > cholesterol > 60
```
```javascript
var cholesterol, healthy;
cholesterol = 127;
healthy = (200 > cholesterol && cholesterol > 60);
```

#### 存在运算符

Javascript中没有判断变量存在的运算符。`if (variable) ...`在变量是0、空串、false的情况下也不执行。CoffeeScript 的存在运算符 `?` 仅当变量不是`null`或`undefined`时返回true。

还可用于更安全的条件赋值｛｛仅当为null时赋值｝｝。当处理数字和字符串时，比 `||=` 更安全。
```javascript
solipsism = true if mind? and not world?

speed = 0
speed ?= 15

footprints = yeti ? "bear"
```
```javascript
var footprints, solipsism, speed;

if ((typeof mind !== "undefined" && mind !== null) && (typeof world === "undefined" || world === null)) {
  solipsism = true;
}

speed = 0;

if (speed == null) { //｛｛undefined呢｝｝
  speed = 15;
}

footprints = typeof yeti !== "undefined" && yeti !== null ? yeti : "bear";
```

#### 安全属性访问

`?.`代替`.`，可以用于消除属性链中的空指针。如果属性链中有null或undefined，将返回`undefined`。

```javascript
zip = lottery.drawWinner?().address?.zipcode
```
```javascript
var zip, _ref;

zip = typeof lottery.drawWinner === "function" ? (_ref = lottery.drawWinner().address) != null ? _ref.zipcode : void 0 : void 0;
```

Soaking up nulls is similar to Ruby's andand gem, and to the safe navigation operator in Groovy.

## 语句与表达式

### If, Else, Unless 与条件赋值

书写 if/else 时不必使用括号的大括号。多行条件用缩进分隔。
还有一种后缀形式：`if`或`unless`放后面。

CoffeeScript会尽量将`if`编译成Javascript的三元运算，否则使用闭包包裹（closure wrapping otherwise）。CoffeeScript中没有单独的三元运算符，你只需要使用单行`if`。

```javascript
mood = greatlyImproved if singing

if happy and knowsIt
  clapsHands()
  chaChaCha()
else
  showIt()

date = if friday then sue else jill

// ----->

var date, mood;

if (singing) {
  mood = greatlyImproved;
}

if (happy && knowsIt) {
  clapsHands();
  chaChaCha();
} else {
  showIt();
}

date = friday ? sue : jill;
```

### 后缀表达式

一个使用后缀表达式改写的例子。原来的代码：
```javascript
odd = (num)->
	if typeof num is 'number'
		if num is Math.round num
			if num >0
				num % 2 is 1
			else
			throw "#{num}is not positive"
		else
			throw"#{num}is not an integer"
	else
		throw"#{num}is not a number"
```

改写后：
```javascript
odd = (num)->
	unless typeof num is 'number'
		throw "#{num} is not a number"
	unless num is Math.round num
		throw "#{num} is not an integer"
	unless num >0
		throw "#{num} is not positive"
	num % 2 is 1
```


### 循环与Comprehensions

Comprehensions replace (and compile into) `for` loops, with optional guard clauses and the value of the current array index. 与循环不同的是，数组 comprehensions 是表达式，可以被返回和赋值。

```javascript
shortNames = (name for name in list when name.length < 5)

evens = (x for x in [0..10] by 2)
```


```javascript
# Eat lunch.
eat food for food in ['toast', 'cheese', 'wine']

# Fine five course dining.
courses = ['greens', 'caviar', 'truffles', 'roast', 'cake']
menu i + 1, dish for dish, i in courses

# Health conscious meal.
foods = ['broccoli', 'spinach', 'chocolate']
eat food for food in foods when food isnt 'chocolate'

// ----->

var courses, dish, food, foods, i, _i, _j, _k, _len, _len1, _len2, _ref;

_ref = ['toast', 'cheese', 'wine'];
for (_i = 0, _len = _ref.length; _i < _len; _i++) {
  food = _ref[_i];
  eat(food);
}

courses = ['greens', 'caviar', 'truffles', 'roast', 'cake'];

for (i = _j = 0, _len1 = courses.length; _j < _len1; i = ++_j) {
  dish = courses[i];
  menu(i + 1, dish);
}

foods = ['broccoli', 'spinach', 'chocolate'];

for (_k = 0, _len2 = foods.length; _k < _len2; _k++) {
  food = foods[_k];
  if (food !== 'chocolate') {
    eat(food);
  }
}
```

```javascript
countdown = (num for num in [10..1])

// ----->

var countdown, num;

countdown = (function() {
  var _i, _results;
  _results = [];
  for (num = _i = 10; _i >= 1; num = --_i) {
    _results.push(num);
  }
  return _results;
})();
```

有时仅想要副作用，不想要 comprehension 的结果。此时，可以在函数默认添加一个有意义的返回值，如`true`或`null`，避免不经意返回结果。

Comprehensions 还可以用来遍历键值对：

```javascript
yearsOld = max: 10, ida: 9, tim: 11

ages = for child, age of yearsOld
  "#{child} is #{age}"

// ----->

var age, ages, child, yearsOld;

yearsOld = {
  max: 10,
  ida: 9,
  tim: 11
};

ages = (function() {
  var _results;
  _results = [];
  for (child in yearsOld) {
    age = yearsOld[child];
    _results.push("" + child + " is " + age);
  }
  return _results;
})();
```

## 函数

函数定义包括：参数列表（括号内，可选）、箭头、函数体。空函数: `->`

```javascript
square = (x) -> x * x
cube   = (x) -> square(x) * x

// ----->

var cube, square;

square = function(x) {
  return x * x;
};

cube = function(x) {
  return square(x) * x;
};
```

最简单的函数：`-> 'hello'。

利用`do`直接运行函数：
```javascript
console.log do -> 'hello'
```
其实就相当于：
```javascript
console.log (-> 'hello')()
```

函数隐式返回最后一个表达式的值。但可以显式使用return。

> 两种函数声明
 在Javascript中有两种函数声明方式。一种是：
 ```javascript
 var cube1 = function(x) {...}
 ```
 另一种是：
 ```javascript
 function cube2 = function(x) {...}
 ```
 两种方式的最大区别是，如果在cube1定以前使用它会报错。但在cube2作用域范围内可以在定义前调用它。
 CoffeeScript只会生成cube1这样的风格（由于IE的问题）。因此在使用前一定要先定义。

隐式括号的问题。函数调用时省略括号。知道表达式结尾，隐式括号才会闭合。如，一个错误的写法：
```javascript
console.log(Math.round 3.1, Math.round 5.2)
```
会被解释为：
```javascript
console.log(Math.round(3.1, Math.round(5.2)))
```

为避免混乱，只在最外层函数调用时省略括号：
```javascript
console.log Math.round(3.1), Math.round(5.2)# 3,5
```

### 函数作用域

考虑下面两段代码的区别：
```javascript
age = 99
fun1 = -> age = 0
fun1()
console.log age
```

```javascript
fun1 = -> age = 0
age = 99
fun1()
console.log age
```

第一个输出0，但第二个输出99。通过`coffee -p`看产生的js发现完全不同。第二种写法，在函数内，`age`被重新定义成局部变量。
```javascript
(function() {
  var age, fun1;
  age = 99;
  fun1 = function() {
    return age = 0;
  };
  fun1();
  console.log("age is " + age);
}).call(this);
```
```javascript
(function() {
  var age, fun1;
  fun1 = function() {
    var age;
    return age = 0;
  };
  age = 99;
  fun1();
  console.log("age is " + age);
}).call(this);
```

应该尽力避免同名覆盖。


### 函数绑定

JavaScript中`this`的指向是动态的。If you're not familiar with this behavior, this [Digital Web article](http://www.digital-web.com/articles/scope_in_javascript/) gives a good overview of the quirks.

利用`=>`定义函数，会绑定`this`的当前值。

```javascript
Account = (customer, cart) ->
  @customer = customer
  @cart = cart

  $('.shopping_cart').bind 'click', (event) =>
    @customer.purchase @cart
```
```javascript
var Account;

Account = function(customer, cart) {
  this.customer = customer;
  this.cart = cart;
  return $('.shopping_cart').bind('click', (function(_this) {
    return function(event) {
      return _this.customer.purchase(_this.cart);
    };
  })(this));
};
```

If we had used `->` in the callback above, `@customer` would have referred to the undefined "customer" property of the DOM element, and trying to call `purchase()` on it would have raised an exception.

When used in a class definition, methods declared with the fat arrow will be automatically bound to each instance of the class when the instance is constructed.

### 属性参数（`@arg`）

下面两个写法是等价的：
```javascript
setName = (name)->@name = name
setName = (@name)-># no code required!
```

### 默认参数

函数参数可以有默认值，若传入`null`或`undefined`，使用默认值。
```javascript
fill = (container, liquid = "coffee") ->
  "Filling the #{container} with #{liquid}..."

// ----->

var fill;

fill = function(container, liquid) {
  if (liquid == null) {
    liquid = "coffee";
  }
  return "Filling the " + container + " with " + liquid + "...";
};
```

Coffee在幕后使用存在运算符，于是，传入`null`或`undefined`等同于省略参数。

### Splats...

JavaScript中通过`arguments`对象实现可变数量参数。CoffeeScript provides splats `...`, both for function definition as well as invocation, making variable numbers of arguments a little bit more palatable.

```javascript
gold = silver = rest = "unknown"

awardMedals = (first, second, others...) ->
  gold   = first
  silver = second
  rest   = others

contenders = [
  "Michael Phelps"
  "Liu Xiang"
  "Yao Ming"
  "Allyson Felix"
  "Shawn Johnson"
  "Roman Sebrle"
  "Guo Jingjing"
  "Tyson Gay"
  "Asafa Powell"
  "Usain Bolt"
]

awardMedals contenders...

alert "Gold: " + gold
alert "Silver: " + silver
alert "The Field: " + rest

// --->

var awardMedals, contenders, gold, rest, silver,
  __slice = [].slice;

gold = silver = rest = "unknown";

awardMedals = function() {
  var first, others, second;
  first = arguments[0], second = arguments[1], others = 3 <= arguments.length ? __slice.call(arguments, 2) : [];
  gold = first;
  silver = second;
  return rest = others;
};

contenders = ["Michael Phelps", "Liu Xiang", "Yao Ming", "Allyson Felix", "Shawn Johnson", "Roman Sebrle", "Guo Jingjing", "Tyson Gay", "Asafa Powell", "Usain Bolt"];

awardMedals.apply(null, contenders);

alert("Gold: " + gold);

alert("Silver: " + silver);

alert("The Field: " + rest);
```

Splats不一定非要呆在参数列表的最后：
```javascript
sandwich = (beginning, middle..., end)->
```

非Splats参数具有更高优先权。因此，如果实参只有2个，则`beginning`和`end`获得。

一个函数只有一个Splats参数才有意义。

Splat还可以用于分割数组：
```shell
coffee> birds = ['duck', 'duck', 'duck', 'duck', 'goose!']
coffee> [ducks..., goose] = birds
coffee> ducks
```

在函数调用（不是声明！）中，Splats可以将一个数组**展开**为一个参数列表：
```javascript
coffee> console.log 1, 2, 3, 4
1 2 3 4
coffee> arr = [1, 2, 3]
coffee> console.log arr, 4
[ 1, 2, 3] 4
coffee> console.log arr..., 4
1 2 3 4
```




