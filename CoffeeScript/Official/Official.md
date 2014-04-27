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

## xxx 语言参考

## xxx Functions

## xxx 对象与数组

## xxx 词法作用域（Lexical Scoping）与变量安全

## xxx If, Else, Unless 与条件赋值

## xxx Splats...

## 循环与Comprehensions

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

## xxx 运算符与别名

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

## xxx 函数绑定

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

Try/catch statements are just about the same as JavaScript (although they work as expressions).

```javascript
try
  allHellBreaksLoose()
  catsAndDogsLivingTogether()
catch error
  print error
finally
  cleanUp()
```
```javascript
var error;

try {
  allHellBreaksLoose();
  catsAndDogsLivingTogether();
} catch (_error) {
  error = _error;
  print(error);
} finally {
  cleanUp();
}
```

```

## 字符串插值、块字符串、块注释

Ruby-style string interpolation is included in CoffeeScript. Double-quoted strings allow for interpolated values, using `#{ ... }`, and single-quoted strings are literal.

```javascript
author = "Wittgenstein"
quote  = "A picture is a fact. -- #{ author }"
sentence = "#{ 22 / 7 } is a decent approximation of π"
```
```javascript
var author, quote, sentence;
author = "Wittgenstein";
quote = "A picture is a fact. -- " + author;
sentence = "" + (22 / 7) + " is a decent approximation of π";
```

Multiline strings are allowed in CoffeeScript. Lines are joined by a single space unless they end with a backslash. Indentation is ignored.

```javascript
mobyDick = "Call me Ishmael. Some years ago --
  never mind how long precisely -- having little
  or no money in my purse, and nothing particular
  to interest me on shore, I thought I would sail
  about a little and see the watery part of the
  world..."
```
```javascript
var mobyDick;
mobyDick = "Call me Ishmael. Some years ago -- never mind how long precisely -- having little or no money in my purse, and nothing particular to interest me on shore, I thought I would sail about a little and see the watery part of the world...";
```
Block strings can be used to hold formatted or indentation-sensitive text (or, if you just don't feel like escaping quotes and apostrophes). The indentation level that begins the block is maintained throughout, so you can keep it all aligned with the body of your code.

```javascript
html = """
       <strong>
         cup of coffeescript
       </strong>
       """
```
```javascript
var html;
html = "<strong>\n  cup of coffeescript\n</strong>";
```

Double-quoted block strings, like other double-quoted strings, allow interpolation.

Sometimes you'd like to pass a block comment through to the generated JavaScript. For example, when you need to embed a licensing header at the top of a file. Block comments, which mirror the syntax for block strings, are preserved in the generated code.

```javascript
###
SkinnyMochaHalfCaffScript Compiler v1.0
Released under the MIT License
###
```
```javascript
/*
SkinnyMochaHalfCaffScript Compiler v1.0
Released under the MIT License
 */
```

## 正则表达式块

Similar to block strings and comments, CoffeeScript supports block regexes — extended regular expressions that ignore internal whitespace and can contain comments and interpolation. Modeled after Perl's `/x` modifier, CoffeeScript's block regexes are delimited by `///` and go a long way towards making complex regular expressions readable. To quote from the CoffeeScript source:

```javascript
OPERATOR = /// ^ (
  ?: [-=]>             # function
   | [-+*/%<>&|^!?=]=  # compound assign / compare
   | >>>=?             # zero-fill right shift
   | ([-+:])\1         # doubles
   | ([&|<>])\2=?      # logic / shift
   | \?\.              # soak access
   | \.{2,3}           # range or splat
) ///
```
```javascript
var OPERATOR;
OPERATOR = /^(?:[-=]>|[-+*\/%<>&|^!?=]=|>>>=?|([-+:])\1|([&|<>])\2=?|\?\.|\.{2,3})/;
```

---

```javascript
```
```javascript
```