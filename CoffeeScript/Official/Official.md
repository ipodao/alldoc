[toc]

## 安装

The CoffeeScript compiler is itself [written in CoffeeScript](http://coffeescript.org/documentation/docs/grammar.html), using the [Jison parser generator](http://jison.org/). The command-line version of coffee is available as a Node.js utility. The core compiler however, does not depend on Node, and can be run in any JavaScript environment, or in the browser (see "Try CoffeeScript", above).

安装：
```shell
sudo npm install -g coffee-script
```

Or, if you want to install to `/usr/local`, and don't want to use npm to manage it, open the `coffee-script` directory and run:
```shell
sudo bin/cake install
```

## 使用

`coffee` 命令可以用于执行脚本，编译`.coffee`为`.js`，或打开 REPL。`coffee`命令的选项：
- `-c`, `--compile`：将一个`.coffee`文件编译为同名的`.js`文件。
- `-m`, `--map`：Generate source maps alongside the compiled JavaScript files. Adds `sourceMappingURL` directives to the JavaScript as well.
- `-i`, `--interactive`：Launch an interactive CoffeeScript session to try short snippets. Identical to calling coffee with no arguments.
- `-o`, `--output [DIR]`：Write out all compiled JavaScript files into the specified directory. Use in conjunction with `--compile` or `--watch`.
- `-j`, `--join [FILE]`：Before compiling, concatenate all scripts together in the order they were passed, and write them into the specified file. Useful for building large projects.
- `-w`, `--watch`：Watch files for changes, rerunning the specified command when any file is updated.
- `-p`, `--print`：Instead of writing out the JavaScript as a file, print it directly to **stdout**.
- `-s`, `--stdio`：Pipe in CoffeeScript to **STDIN** and get back JavaScript over **STDOUT**. Good for use with processes written in other languages. An example:
```cat src/cake.coffee | coffee -sc```
- `-l`, `--literate`: Parses the code as Literate CoffeeScript. You only need to specify this when passing in code directly over stdio, or using some sort of extension-less file name.
- `-e`, `--eval`: Compile and print a little snippet of CoffeeScript directly from the command line. For example:
```coffee -e "console.log num for num in [10..1]"```
- `-b`, `--bare`: Compile the JavaScript without the top-level function safety wrapper.
- `-t`, `--tokens`: Instead of parsing the CoffeeScript, just lex it, and print out the token stream: `[IDENTIFIER square] [ASSIGN =] [PARAM_START (]` ...
- `-n`, `--nodes`： Instead of compiling the CoffeeScript, just lex and parse it, and print out the parse tree:
```
	Expressions
	  Assign
		Value "square"
		Code "x"
		  Op *
			Value "x"
			Value "x"
```
- `--nodejs`: The node executable has some useful options you can set, such as `--debug`, `--debug-brk`, `--max-stack-size`, and `--expose-gc`. Use this flag to forward options directly to Node.js. To pass multiple flags, use `--nodejs` multiple times.

例子：
- 将`src`目录中的`.coffee`编译到平行的目录`lib`：
```shell
coffee --compile --output lib/ src/
```
- 监视，在文件改变后重新编译：
```shell
coffee --watch --compile experimental.coffee
```
- 将多个文件合并成一个脚本：
```shell
coffee --join project.js --compile src/*.coffee
```
- Print out the compiled JS from a one-liner:
```shell
coffee -bpe "alert i for i in [0..10]"
```
- 监控并编译整个工程：
```shell
coffee -o lib/ -cw src/
```
- Start the CoffeeScript REPL (`Ctrl-D` to exit, `Ctrl-V` for multi-line):
```shell
coffee
```

## Literate CoffeeScript

Besides being used as an ordinary programming language, CoffeeScript may also be written in "literate" mode. If you name your file with a `.litcoffee` extension, you can write it as a Markdown document — a document that also happens to be executable CoffeeScript code. The compiler will treat any indented blocks (Markdown's way of indicating source code) as code, and ignore the rest as comments.

Just for kicks, a little bit of the compiler is currently implemented in this fashion: See it [as a document](https://gist.github.com/jashkenas/3fc3c1a8b1009c00d9df), [raw](https://raw.github.com/jashkenas/coffee-script/master/src/scope.litcoffee), and [properly highlighted in a text editor](http://cl.ly/LxEu).

I'm fairly excited about this direction for the language, and am looking forward to writing (and more importantly, reading) more programs in this style. More information about Literate CoffeeScript, including an [example program](https://github.com/jashkenas/journo), are [available in this blog post](http://ashkenas.com/literate-coffeescript).

## 语言参考

CoffeeScript 使用空白符区分代码块。不需要使用分号分隔表达式。行尾就是分隔。（分号用于分隔一行上的多个表达式）。使用缩进（而不是大括号）围绕 function、if、switch 和 try/catch 的代码块。

调用函数传参时不需要使用扩汗。The implicit call wraps forward to the end of the line or block expression.
`console.log sys.inspect object` → `console.log(sys.inspect(object));`

## Functions

函数定义包括：参数列表（括号内，可选）、箭头、函数体。空函数: `->`

```javascript
square = (x) -> x * x
cube   = (x) -> square(x) * x
```
```javascript
var cube, square;

square = function(x) {
  return x * x;
};

cube = function(x) {
  return square(x) * x;
};
```

函数参数可以有默认值，which will be used if the incoming argument is missing (`null` or `undefined`).
```javascript
fill = (container, liquid = "coffee") ->
  "Filling the #{container} with #{liquid}..."
```
```javascript
var fill;

fill = function(container, liquid) {
  if (liquid == null) {
    liquid = "coffee";
  }
  return "Filling the " + container + " with " + liquid + "...";
};
```

## 对象与数组

如果一个属性/元素一行，逗号是可选的。对象可以利用缩进创建（而不是括号），类似于YAML。
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
```
```javascript
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
```
```javascript
$('.account').attr({
  "class": 'active'
});

log(object["class"]);
```

## 词法作用域（Lexical Scoping）与变量安全

The CoffeeScript compiler takes care to make sure that all of your variables are properly declared within lexical scope — 不需要使用 `var` 关键字。
```javascript
outer = 1
changeNumbers = ->
  inner = -1
  outer = 10
inner = changeNumbers()
```
```javascript
var changeNumbers, inner, outer;

outer = 1;

changeNumbers = function() {
  var inner;
  inner = -1;
  return outer = 10;
};

inner = changeNumbers();
```

注意到，所有的变量声明都被提到最近作用域（变量首次出现的）的顶端。`outer`没有在函数中被重新声明，because it's already in scope。

This behavior is effectively identical to Ruby's scope for local variables. 因为无法直接使用`var`，因此无法故意掩盖外部变量，you may only refer to it. So be careful that you're not reusing the name of an external variable accidentally, if you're writing a deeply nested function.

CoffeeScript的输出会被包裹进一个匿名函数：`(function(){ ... })()`。包裹与自动产生`var`关键字，避免了不小心污染全局命名空间的情况。

If you'd like to create top-level variables for other scripts to use, attach them as properties on `window`, or on the `exports` object in CommonJS. The existential operator (covered below), gives you a reliable way to figure out where to add them; if you're targeting both CommonJS and the browser: `exports ? this`

## If, Else, Unless 与条件赋值

书写 if/else 时不必使用括号的大括号。多行条件用拖进分隔。还有一种后缀形式，`if`或`unless`放后面。

CoffeeScript会尽量将`if`便意味Javascript的三元运算，否则使用闭包包裹（closure wrapping otherwise）。CoffeeScript 中没有显式的三元运算符，你只需要使用单行`if`。

```javascript
mood = greatlyImproved if singing

if happy and knowsIt
  clapsHands()
  chaChaCha()
else
  showIt()

date = if friday then sue else jill
```
```javascript
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

## Splats...

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
```
```javascript
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

## 循环与Comprehensions

Most of the loops you'll write in CoffeeScript will be comprehensions over arrays, objects, and ranges. Comprehensions replace (and compile into) `for` loops, with optional guard clauses and the value of the current array index. 与循环不同的是，数组 comprehensions 是表达式，可以被返回和赋值。
```javascript
# Eat lunch.
eat food for food in ['toast', 'cheese', 'wine']

# Fine five course dining.
courses = ['greens', 'caviar', 'truffles', 'roast', 'cake']
menu i + 1, dish for dish, i in courses

# Health conscious meal.
foods = ['broccoli', 'spinach', 'chocolate']
eat food for food in foods when food isnt 'chocolate'
```
```javascript
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

Comprehensions should be able to handle most places where you otherwise would use a loop, each/forEach, map, or select/filter, for example:
`shortNames = (name for name in list when name.length < 5)`

If you know the start and end of your loop, or would like to step through in fixed-size increments, you can use a range to specify the start and end of your comprehension.

```javascript
countdown = (num for num in [10..1])
```
```javascript
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

To step through a *range comprehension* in fixed-size chunks, use by, for example:
```javascript
evens = (x for x in [0..10] by 2)
```

Comprehensions 还可以用来遍历键值对：

```javascript
yearsOld = max: 10, ida: 9, tim: 11

ages = for child, age of yearsOld
  "#{child} is #{age}"
```
```javascript
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

If you would like to iterate over just the keys that are defined on the object itself, by adding a `hasOwnProperty` check to avoid properties that may be inherited from the prototype, use
`for own key, value of object`.

CoffeeScript 提供的唯一一种底层循环是 while 循环。在CoffeeScript中，while 循环可以用作表达式，返回一个数组。

```javascript
# Econ 101
if this.studyingEconomics
  buy()  while supply > demand
  sell() until supply > demand

# Nursery Rhyme
num = 6
lyrics = while num -= 1
  "#{num} little monkeys, jumping on the bed.
    One fell out and bumped his head."
```
```javascript
var lyrics, num;

if (this.studyingEconomics) {
  while (supply > demand) {
    buy();
  }
  while (!(supply > demand)) {
    sell();
  }
}

num = 6;

lyrics = (function() {
  var _results;
  _results = [];
  while (num -= 1) {
    _results.push("" + num + " little monkeys, jumping on the bed. One fell out and bumped his head.");
  }
  return _results;
})();
```

For readability, the `until` keyword is equivalent to `while not`, and the `loop` keyword is equivalent to `while true`.

When using a JavaScript loop to generate functions, it's common to insert a closure wrapper in order to ensure that loop variables are closed over, and all the generated functions don't just share the final values. CoffeeScript provides the `do` keyword, which immediately invokes a passed function, forwarding any arguments.


```javascript
for filename in list
  do (filename) ->
    fs.readFile filename, (err, contents) ->
      compile filename, contents.toString()
```
```javascript
var filename, _fn, _i, _len;

_fn = function(filename) {
  return fs.readFile(filename, function(err, contents) {
    return compile(filename, contents.toString());
  });
};
for (_i = 0, _len = list.length; _i < _len; _i++) {
  filename = list[_i];
  _fn(filename);
}
```

## Array Slicing and Splicing with Ranges

Ranges 还可以用于抽出数组中一个片段。两点`(3..6)`，范围为(3, 4, 5, 6)；三点`(3...6)`，范围为`(3, 4, 5)`。省略第一个参数，默认取0。省略最后一个参数，默认取数组长度。
```javascript
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9]
start   = numbers[0..2]
middle  = numbers[3...-2]
end     = numbers[-2..]
copy    = numbers[..]
```
```javascript
var copy, end, middle, numbers, start;

numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9];
start = numbers.slice(0, 3);
middle = numbers.slice(3, -2);
end = numbers.slice(-2);
copy = numbers.slice(0);
```

相同的语法还可以被用于赋值，替换数组中一段值：
```javascript
numbers = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
numbers[3..6] = [-3, -4, -5, -6]
```
```javascript
var numbers, _ref;

numbers = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9];
[].splice.apply(numbers, [3, 4].concat(_ref = [-3, -4, -5, -6])), _ref;
```

Note that JavaScript strings are immutable, and can't be spliced.

## 一切都是表达式（至少，尽可能是）

若函数没i有`return`语句，将返回最后一个值。CoffeeScript compiler 尝试确保所有语句都可以用作表达式。Watch how the return gets pushed down into each possible branch of execution in the function below.

```javascript
grade = (student) ->
  if student.excellentWork
    "A+"
  else if student.okayStuff
    if student.triedHard then "B" else "B-"
  else
    "C"

eldest = if 24 > 21 then "Liz" else "Ike"
```
```javascript
var eldest, grade;

grade = function(student) {
  if (student.excellentWork) {
    return "A+";
  } else if (student.okayStuff) {
    if (student.triedHard) {
      return "B";
    } else {
      return "B-";
    }
  } else {
    return "C";
  }
};

eldest = 24 > 21 ? "Liz" : "Ike";
```

Even though functions will always return their final value, it's both possible and **encouraged** to return early from a function body writing out the explicit `return` (return value), when you know that you're done.

Because variable declarations occur at the top of scope, assignment can be used within expressions, even for variables that haven't been seen before:
```javascript
six = (one = 1) + (two = 2) + (three = 3)
```
```javascript
var one, six, three, two;
six = (one = 1) + (two = 2) + (three = 3);
```

Things that would otherwise be statements in JavaScript, when used as part of an expression in CoffeeScript, are converted into expressions by wrapping them in a closure. This lets you do useful things, like assign the result of a comprehension to a variable:
```javascript
# The first ten global properties.
globals = (name for name of window)[0...10]
```
```javascript
var globals, name;

globals = ((function() {
  var _results;
  _results = [];
  for (name in window) {
    _results.push(name);
  }
  return _results;
})()).slice(0, 10);
```

As well as silly things, like passing a try/catch statement directly into a function call:
```javascript
alert(
  try
    nonexistent / undefined
  catch error
    "And the error is ... #{error}"
)

```
```javascript
var error;

alert((function() {
  try {
    return nonexistent / void 0;
  } catch (_error) {
    error = _error;
    return "And the error is ... " + error;
  }
})());
```

There are a handful of statements in JavaScript that can't be meaningfully converted into expressions, namely `break`, `continue`, and `return`. If you make use of them within a block of code, CoffeeScript won't try to perform the conversion.

## 运算符与别名

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

### 存在运算符

Javascript中没有判断变量存在的运算符。`if (variable) ...`在变量是0、空串、false的情况下也不执行。CoffeeScript 的存在运算符 `?` 仅当变量不是null或undefined时返回true。

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

The accessor variant of the existential operator `?.` can be used to soak up null references in a chain of properties. Use it instead of the dot accessor `.` in cases where the base value may be null or undefined. If all of the properties exist then you'll get the expected result, if the chain is broken, `undefined` is returned instead of the TypeError that would be raised otherwise.

```javascript
zip = lottery.drawWinner?().address?.zipcode
```
```javascript
var zip, _ref;

zip = typeof lottery.drawWinner === "function" ? (_ref = lottery.drawWinner().address) != null ? _ref.zipcode : void 0 : void 0;
```

Soaking up nulls is similar to Ruby's andand gem, and to the safe navigation operator in Groovy.

## Classes, Inheritance, and Super

JavaScript's prototypal inheritance has always been a bit of a brain-bender, with a whole family tree of libraries that provide a cleaner syntax for classical inheritance on top of JavaScript's prototypes: Base2, Prototype.js, JS.Class, etc. The libraries provide syntactic sugar, but the built-in inheritance would be completely usable if it weren't for a couple of small exceptions: it's awkward to call super (the prototype object's implementation of the current function), and it's awkward to correctly set the prototype chain.

Instead of repetitively attaching functions to a prototype, CoffeeScript provides a basic `class` structure that allows you to name your class, set the superclass, assign prototypal properties, and define the constructor, in a single assignable expression.

Constructor functions are named, to better support helpful stack traces. In the first class in the example below, `this.constructor.name` is "Animal".
```javascript
class Animal
  constructor: (@name) ->

  move: (meters) ->
    alert @name + " moved #{meters}m."

class Snake extends Animal
  move: ->
    alert "Slithering..."
    super 5

class Horse extends Animal
  move: ->
    alert "Galloping..."
    super 45

sam = new Snake "Sammy the Python"
tom = new Horse "Tommy the Palomino"

sam.move()
tom.move()
```
```javascript
var Animal, Horse, Snake, sam, tom,
  __hasProp = {}.hasOwnProperty,
  __extends = function(child, parent) { for (var key in parent) { if (__hasProp.call(parent, key)) child[key] = parent[key]; } function ctor() { this.constructor = child; } ctor.prototype = parent.prototype; child.prototype = new ctor(); child.__super__ = parent.prototype; return child; };

Animal = (function() {
  function Animal(name) {
    this.name = name;
  }

  Animal.prototype.move = function(meters) {
    return alert(this.name + (" moved " + meters + "m."));
  };

  return Animal;

})();

Snake = (function(_super) {
  __extends(Snake, _super);

  function Snake() {
    return Snake.__super__.constructor.apply(this, arguments);
  }

  Snake.prototype.move = function() {
    alert("Slithering...");
    return Snake.__super__.move.call(this, 5);
  };

  return Snake;

})(Animal);

Horse = (function(_super) {
  __extends(Horse, _super);

  function Horse() {
    return Horse.__super__.constructor.apply(this, arguments);
  }

  Horse.prototype.move = function() {
    alert("Galloping...");
    return Horse.__super__.move.call(this, 45);
  };

  return Horse;

})(Animal);

sam = new Snake("Sammy the Python");

tom = new Horse("Tommy the Palomino");

sam.move();

tom.move();
```

If structuring your prototypes classically isn't your cup of tea, CoffeeScript provides a couple of lower-level conveniences. The `extends` operator helps with proper prototype setup, and can be used to create an inheritance chain between any pair of constructor functions; `::` gives you quick access to an object's prototype; and `super()` is converted into a call against the immediate ancestor's method of the same name.

```javascript
String::dasherize = ->
  this.replace /_/g, "-"
```
```javascript
String.prototype.dasherize = function() {
  return this.replace(/_/g, "-");
};
```

Finally, class definitions are blocks of executable code, which make for interesting metaprogramming possibilities. Because in the context of a class definition, `this` is the class object itself (the constructor function), you can assign **static** properties by using `@property: value`, and call functions defined in parent classes: `@attr 'title', type: 'text'`


## Destructuring Assignment

To make extracting values from complex arrays and objects more convenient, CoffeeScript implements ECMAScript Harmony's proposed [destructuring assignment](http://wiki.ecmascript.org/doku.php?id=harmony:destructuring) syntax. When you assign an array or object literal to a value, CoffeeScript breaks up and matches both sides against each other, assigning the values on the right to the variables on the left. In the simplest case, it can be used for parallel assignment:

```javascript
theBait   = 1000
theSwitch = 0

[theBait, theSwitch] = [theSwitch, theBait]
```
```javascript
var theBait, theSwitch, _ref;

theBait = 1000;
theSwitch = 0;

_ref = [theSwitch, theBait], theBait = _ref[0], theSwitch = _ref[1];
```

But it's also helpful for dealing with functions that return multiple values.

```javascript
weatherReport = (location) ->
  # Make an Ajax request to fetch the weather...
  [location, 72, "Mostly Sunny"]

[city, temp, forecast] = weatherReport "Berkeley, CA"
```
```javascript
var city, forecast, temp, weatherReport, _ref;

weatherReport = function(location) {
  return [location, 72, "Mostly Sunny"];
};

_ref = weatherReport("Berkeley, CA"), city = _ref[0], temp = _ref[1], forecast = _ref[2];
```

Destructuring assignment can be used with any depth of array and object nesting, to help pull out deeply nested properties.

```javascript
futurists =
  sculptor: "Umberto Boccioni"
  painter:  "Vladimir Burliuk"
  poet:
    name:   "F.T. Marinetti"
    address: [
      "Via Roma 42R"
      "Bellagio, Italy 22021"
    ]

{poet: {name, address: [street, city]}} = futurists
```
```javascript
var city, futurists, name, street, _ref, _ref1;

futurists = {
  sculptor: "Umberto Boccioni",
  painter: "Vladimir Burliuk",
  poet: {
    name: "F.T. Marinetti",
    address: ["Via Roma 42R", "Bellagio, Italy 22021"]
  }
};

_ref = futurists.poet, name = _ref.name, (_ref1 = _ref.address, street = _ref1[0], city = _ref1[1]);
```

Destructuring assignment can even be combined with splats.

```javascript
	tag = "<impossible>"

	[open, contents..., close] = tag.split("")
```
```javascript
	var close, contents, open, tag, _i, _ref,
	  __slice = [].slice;

	tag = "<impossible>";

	_ref = tag.split(""), open = _ref[0], contents = 3 <= _ref.length ? __slice.call(_ref, 1, _i = _ref.length - 1) : (_i = 1, []), close = _ref[_i++];
```

Expansion can be used to retrieve elements from the end of an array without having to assign the rest of its values. It works in function parameter lists as well.

```javascript
text = "Every literary critic believes he will
        outwit history and have the last word"

[first, ..., last] = text.split " "
```
```javascript
var first, last, text, _ref;

text = "Every literary critic believes he will outwit history and have the last word";

_ref = text.split(" "), first = _ref[0], last = _ref[_ref.length - 1];
```

Destructuring assignment is also useful when combined with class constructors to assign properties to your instance from an options object passed to the constructor.

```javascript
class Person
  constructor: (options) ->
    {@name, @age, @height} = options

tim = new Person age: 4
```
```javascript
var Person, tim;

Person = (function() {
  function Person(options) {
    this.name = options.name, this.age = options.age, this.height = options.height;
  }

  return Person;

})();

tim = new Person({
  age: 4
});
```

## 函数绑定

In JavaScript, the `this` keyword is dynamically scoped to mean the object that the current function is attached to. If you pass a function as a callback or attach it to a different object, the original value of this will be lost. If you're not familiar with this behavior, this [Digital Web article](http://www.digital-web.com/articles/scope_in_javascript/) gives a good overview of the quirks.

The fat arrow `=>` can be used to both define a function, and to bind it to the current value of `this`, right on the spot. This is helpful when using callback-based libraries like Prototype or jQuery, for creating iterator functions to pass to `each`, or event-handler functions to use with `bind`. Functions created with the fat arrow are able to access properties of the `this` where they're defined.

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

## 嵌入Javascript

Hopefully, you'll never need to use it, but if you ever need to intersperse snippets of JavaScript within your CoffeeScript, you can use backticks to pass it straight through.

```javascript
hi = `function() {
  return [document.title, "Hello JavaScript"].join(": ");
}`
```
```javascript
var hi;

hi = function() {
  return [document.title, "Hello JavaScript"].join(": ");
};
```

## Switch/When/Else

Switch statements in JavaScript are a bit awkward. You need to remember to break at the end of every case statement to avoid accidentally falling through to the default case. CoffeeScript prevents accidental fall-through, and can convert the `switch` into a returnable, assignable expression. The format is: `switch` condition, `when` clauses, `else` the default case.

As in Ruby, switch statements in CoffeeScript can take multiple values for each when clause. If any of the values match, the clause runs.

```javascript
switch day
  when "Mon" then go work
  when "Tue" then go relax
  when "Thu" then go iceFishing
  when "Fri", "Sat"
    if day is bingoDay
      go bingo
      go dancing
  when "Sun" then go church
  else go work
```
```javascript
switch (day) {
  case "Mon":
    go(work);
    break;
  case "Tue":
    go(relax);
    break;
  case "Thu":
    go(iceFishing);
    break;
  case "Fri":
  case "Sat":
    if (day === bingoDay) {
      go(bingo);
      go(dancing);
    }
    break;
  case "Sun":
    go(church);
    break;
  default:
    go(work);
}
```

Switch statements can also be used without a control expression, turning them in to a cleaner alternative to if/else chains.

```javascript
score = 76
grade = switch
  when score < 60 then 'F'
  when score < 70 then 'D'
  when score < 80 then 'C'
  when score < 90 then 'B'
  else 'A'
# grade == 'C'
```
```javascript
var grade, score;

score = 76;

grade = (function() {
  switch (false) {
    case !(score < 60):
      return 'F';
    case !(score < 70):
      return 'D';
    case !(score < 80):
      return 'C';
    case !(score < 90):
      return 'B';
    default:
      return 'A';
  }
})();
```

### Try/Catch/Finally

Try/Catch/Finally


---

```javascript
```
```javascript
```