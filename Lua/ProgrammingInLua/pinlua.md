[toc]

## Preface

Lua被设计为与**C/C++**等其他语言集成。Lua不打算做其他语言更擅长做的事，如底层操作。

Lua是胶水语言。但不仅是胶水语言。

Lua非常可扩展。可以通过Lua或C扩展。设置Lua的基本功能都是由外部库实现的。Lua与**C/C++**集成非常容易。有人把Lua看做DSL。

Lua是跨平台的。包括Windows、IOS、Android、树莓派、Arduino等。所有平台的源代码几乎是一样的。Lua不使用条件编译适配各个平台。Lua坚持使用标准ANSI (ISO) C。

本书针对Lua 5.2。

## I 语言

## 1. 入门

第一个例子：

```lua
	print("Hello World")
```

If you save the above program in a file hello.lua, the following command should run it:
```lua
	% lua hello.lua
```

费波那契数列：

```lua
    -- defines a factorial function
    function fact (n)
    	if n == 0 then
    		return 1
    	else
    		return n * fact(n-1)
    	end
    end
    print("enter a number:")
    a = io.read("*n") -- reads a number
    print(fact(a))
```

### 1.1 Chunk

Each piece of code that Lua executes, such as a file or a single line in interactive mode, is called a chunk. A chunk is simply a sequence of commands (or statements).

Lua needs no separator between consecutive statements, 但如果愿意可以使用分号。我个人习惯，只有一行有多个语句时采用分号。换行在Lua语法中没有特殊含义。例如下面四个写法是等价的：

```lua
    a = 1
    b = a*2

    a = 1;
    b = a*2;

    a = 1; b = a*2

    a = 1 b = a*2 -- ugly, but valid
```

Because Lua is used also as a data-description language, chunks with several megabytes are not uncommon. The Lua interpreter has no problems at all with large chunks.


退出交互式编译器：`os.exit()`。

在交互式编译器中执行文件：`dofile`。如：

```lua
    function norm (x, y)
        return (x^2 + y^2)^0.5
    end
    function twice (x)
        return 2*x
    end
```

```
    > dofile("lib1.lua") -- load your library
    > n = norm(3.4, 1.0)
    > print(twice(n)) --> 7.0880180586677
```

### 1.2 词法约定

标识符可以含有字母、数字和下环线。不能以数字开头。

```
    i j i10 _ij aSomewhatLongName _INPUT
```

避免使用下划线开头跟大写字母（如`_VERSION`）。它们保留做特殊用途。Usually, I reserve the identifier `_` (a single underscore) for dummy variables.

In older versions of Lua, the concept of what is a letter depended on the locale. However, such letters make your program unsuitable to run in systems that do not support that locale. Lua 5.2只接受大小写26个字母，不能使用其他字符。

下面是保留字

    and break do else elseif
    end false goto for function
    if in local nil not
    or repeat return then true
    until while

Lua是大小写敏感的。

单行注释以`--`开头。块注释（多行）以`--[[`开头`]]`结尾。A common trick to comment out a piece of code is to enclose
the code between --[[ and --]], like here:

### 1.3 全局变量

全局变量可以不用声明，直接使用：

```lua
    print(b) --> nil
    b = 10
    print(b) --> 10
```

If you assign `nil` to a global variable, Lua behaves as if the variable had never been used:

```lua
    b = nil
    print(b) --> nil
```

### （未）1.4 The Stand-Alone Interpreter

## 2. 类型与值

Lua是动态类型语言。值自己携带类型。

有8个基本类型：nil, boolean, number, string, userdata, function, thread, and table。`type`函数可以给出值的类型名（字符串）：

```lua
    print(type("Hello world")) --> string
    print(type(10.4*3)) --> number
    print(type(print)) --> function
    print(type(type)) --> function
    print(type(true)) --> boolean
    print(type(nil)) --> nil
    print(type(type(X))) --> string
```

变量可以持有任意类型，可以改换类型。

```lua
    print(type(a)) --> nil ('a' is not initialized)
    a = 10
    print(type(a)) --> number
    a = "a string!!"
    print(type(a)) --> string
    a = print -- yes, this is valid!
    a(type(a)) --> function
```

Lua中函数是一等公民。

### 2.1 Nil

Nil is a type with a single value, `nil`, whose main property is to be different from any other value. Lua uses `nil` as a kind of non-value, to represent the absence of a useful value. As we have seen, a global variable has a nil value by default, before its first assignment, and you can assign nil to a global variable to **delete** it.

### 2.2 布尔

布尔类型有两个值：false和true。However, booleans do not hold a monopoly of condition values: 在Lua中所有值都都表示一个条件。布尔测试时，布尔false和nil被当作假其他都是真。特别的，数字零和空字符串偶是真。


### 2.3 数字

数字类型表示浮点数，双精度。Lua没有整数。Some people fear that even a simple increment or comparison can go weird with floating-point numbers. Reality, however, is not like that. 现在几乎所有平台表示浮点数都遵循IEEE 754标准。Following this standard, the only possible source of errors is a representation error, which happens when a number cannot be exactly represented. An operation rounds its result only if that result has no exact representation. Any operation with a result that has an exact representation must give that exact result.

事实是，小于2^53（接近10^16）的整数都能被双精度浮点数精确表示。When you use a double to represent an integer, there is no rounding error at all, 除非整数超过2^53。特别的，Lua数字类型可以表示任何32位整数，不会有rounding问题。

Of course, fractional numbers can have representation errors. The situation here is not different from what happens with pen and paper. If we want to write `1/7` in decimal, we will have to stop somewhere. If we use ten digits to represent a number, `1/7` becomes rounded to 0:142857142. If we compute `1/7*7` using ten digits, the result will be `0:999999994`, which is different from 1.

Moreover, numbers that have a finite representation in decimal can have an infinite representation in binary. For instance, `12.7-20+7.3` is not exactly zero when computed with doubles, because both `12.7` and `7.3` do not have an exact finite representation in binary (see Exercise 2.3).

Before we go on, remember: integers do have exact representations and therefore do not have rounding errors.

Most modern CPUs do floating-point arithmetic as fast as (or even faster than) integer arithmetic. Nevertheless, it is easy to compile Lua so that it uses another type for numbers, such as longs or single-precision floats. This is particularly useful for platforms without hardware support for floating point, such as embedded systems. See file `luaconf.h` in the distribution for details.

可以写十六进制常量，前缀`0x`。Since Lua 5.2, hexadecimal constants can also have a fractional part and a binary exponent (prefixed by ‘p’ or ‘P’), as in the following examples:

    0xff (255) 0x1A3 (419) 0x0.2 (0.125) 0x1p-1 (0.5)
    0xa.bp2 (42.75)

### 2.4 字符串

Lua is eight-bit clean and its strings can contain characters with any numeric code, including embedded zeros. This means that you can store any binary data into a string. You can also store Unicode strings in any representation (UTF-8, UTF-16, etc.). Lua的标准字符串库不提供对这些表示的显式支持。Nevertheless, we can handle UTF-8 strings quite reasonably, as we will discuss in Section 21.7.

Lua的字符串是不可变的。

```lua
    a = "one string"
    b = string.gsub(a, "one", "another") -- change string parts
    print(a) --> one string
    print(b) --> another string
```

Lua的字符串遵从自动内存管理，like all other Lua objects (tables, functions, etc.). This means that you do not have to worry about allocation and deallocation of strings; Lua handles this for you. Programs that manipulate strings with 100K or 1M characters are not unusual in Lua.

可以通过前缀运算符‘#’（长度运算符）获得字符串长度：

```lua
    a = "hello"
    print(#a) --> 5
    print(#"good\0bye") --> 8
```

#### 字面量

字面量被单引号或双引号包围：

```lua
    a = "a line"
    b = 'another line'
```

We can specify a character in a string also by its numeric value through the escape sequences \ddd and \x\hh, where ddd is a sequence of up to three decimal digits and hh is a sequence of exactly two hexadecimal digits. As a somewhat complex example, the two literals "alo\n123\"" and '\97lo\10\04923"' have the same value, in a system using ASCII: 97 is the ASCII code for ‘a’, 10 is the code for newline, and 49 is the code for the digit ‘1’. (In this example we must write 49 with three digits, as \049, because it is followed by another digit; otherwise Lua would read the number as 492.) We can also write that same string as '\x61\x6c\x6f\x0a\x31\x32\x33\x22', representing each character by its hexadecimal code.

#### 长字符串

以两个中括号包围的字符串。字符串可以跨多行。不会解析转义字符。如果第一个字符是换行，会被忽略。

```lua
    page = [[
    <html>
    <head>
    <title>An HTML Page</title>
    </head>
    <body>
    <a href="http://www.lua.org">Lua</a>
    </body>
    </html>
    ]]
    write(page)
```

Sometimes, you may want to enclose a piece of code containing something like `a=b[c[i]]` (notice the `]]` in this code), or you may need to enclose some code that already has some code commented out. To handle such cases, you can add any number of equal signs between the two open brackets, as in `[===[`. After this change, the literal string ends only at the next closing brackets with the same number of equal signs in between (`]===]`, in our example). The scanner ignores pairs of brackets with a different number of equal signs. By choosing an appropriate number of signs, you can enclose any literal string without having to add escapes into it.

This same facility is valid for comments, too. For instance, if you start a long comment with `--[=[`, it extends until the next `]=]`. This facility allows you easily to comment out a piece of code that contains parts already commented out.

为防止行过长，允许安全的换行，Lua 5.2 offers the escape sequence \z: it skips all subsequent characters in the string until the first non-space character. The next example illustrates its use:

```lua
    data = "\x00\x01\x02\x03\x04\x05\x06\x07\z
    \x08\x09\x0A\x0B\x0C\x0D\x0E\x0F"
```

The \z at the end of the first line skips the following end-of-line and the indentation of the next line, so that the byte \x07 is directly followed by \x08 in the resulting string.

#### 类型转换

Lua提供运行时的数字和字符串的自动转换。算术运算中的字符串会被转换为数字：

```lua
    print("10" + 1) --> 11
    print("10 + 1") --> 10 + 1
    print("-5.3e-10"*"2") --> -1.06e-09
    print("hello" + 1) -- ERROR (cannot convert "hello")
```

这种转换也发生在其他表达式，需要数字的时候，如调用`math.sin`。

与此相反，当Lua期望一个字符串时却遇到数字时，会将数字转换为字符串：

```lua
	print(10 .. 20) --> 1020
```

`..`是Lua的字符串连接运算符。这个运算符放在数字后，必须加空格，否则Lua会把第一个点当作小数点。

现在我们不在觉得这些自动转换是一种好的设计。最好不要依赖它们。They are handy in a few places, but add complexity both to the language and to programs that use them. After all, strings and numbers are different things, despite these conversions. 形如`10=="10"`的比较是假值。

若需显式将字符串转换为数字，可以使用函数`tonumber`。如果不能转换返回`nil`。

```lua
    line = io.read() -- read a line
    n = tonumber(line) -- try to convert it to a number
    if n == nil then
    	error(line .. " is not a valid number")
    else
    	print(n*2)
    end
```

将数字转换为字符串可以使用函数`tostring`，或将数字连接到一个空字符串：

```lua
    print(tostring(10) == "10") --> true
    print(10 .. "" == "10") --> true
```

### 2.5 Tables

类型table是关联数组（associative arrays）。关联数组的索引不仅可以是数字，也可以是字符串，或语言中的其他值。`nil`除外。

Tables是Lua中唯一的数据结构。我们用Table一种结构表示了普通数组、集合、记录或其他数据结构。Lua还要tables表示包（packages）和对象。如`io.read`，实际上，`io`是table，`read`是table的索引。

Lua的table不是值也不是变量；它们是对象。If you are familiar with arrays in Java or Scheme, then you have a fair idea of what I mean. You may think of a table as a dynamically allocated object; your program manipulates only references (or pointers) to them. Lua never does hidden copies or creation of new tables behind the scenes. Moreover, you do not have to declare a table in Lua; in fact, there is no way to declare one. 通过构造器表达式（constructor expression）创建一个table。最简单的形式是`{}`：

```lua
    a = {} -- create a table and store its reference in 'a'
    k = "x"
    a[k] = 10 -- new entry, with key="x" and value=10
    a[20] = "great" -- new entry, with key=20 and value="great"
    print(a["x"]) --> 10
    k = 20
    print(a[k]) --> "great"
    a["x"] = a["x"] + 1 -- increments entry "x"
    print(a["x"]) --> 11
```

A table is always anonymous. There is no fixed relationship between a variable that holds a table and the table itself:

```lua
    a = {}
    a["x"] = 10
    b = a -- 'b' refers to the same table as 'a'
    print(b["x"]) --> 10
    b["x"] = 20
    print(a["x"]) --> 20
    a = nil -- only 'b' still refers to the table
    b = nil -- no references left to the table
```

若不再有引用指向table，Lua的垃圾收集会最终删除Table，释放内存。

一个Table的索引可以是多种类型的。

```lua
    a = {} -- empty table
    -- create 1000 new entries
    for i = 1, 1000 do a[i] = i*2 end
    	print(a[9]) --> 18
    a["x"] = 10
    print(a["x"]) --> 10
    print(a["y"]) --> nil
```

注意最后一行：与全局变量一样，Table的字段（field）若未被初始化，求值为`nil`。与全局变量一样，给一个字段赋`nil`将删除该字段。This is not a coincidence: Lua stores global variables in ordinary tables. We will discuss this subject further in Chapter 14.

获取字段值时，`a.name`等价于`a["name"]`。

```lua
    a.x = 10 -- same as a["x"] = 10
    print(a.x) -- same as print(a["x"])
    print(a.y) -- same as print(a["y"])
```

要表示一个传统数组或列表，只要让整数做索引。不需要也不能声明大小，只需按需使用：

```lua
    -- read 10 lines, storing them in a table
    a = {}
    for i = 1, 10 do
        a[i] = io.read()
    end
```

索引可以从任何整数开始。但Lua习惯从1开始！Lua中一些工具遵从这个约定。

Usually, when you manipulate a list you must know its length. It can be a constant or it can be stored somewhere. Often we store the length of a list in a non-numeric field of the table; for historical reasons, several programs use the field “n” for this purpose.

Often, however, the length is implicit. Remember that any non-initialized index results in `nil`; 可以利用这个值标记边界。但有时列表内部也有nil。我们称这种列表为序列（sequence）。对于序列，Lua提供长度运算符‘#’。For instance, you could print the lines read in the last example with the following code:

```lua
-- print the lines
for i = 1, #a do
	print(a[i])
end
```

### 2.6 函数

Lua中函数是一等值：可以将函数存储到一个变量中，将函数作为参数传递，将函数作为函数返回值。Moreover, Lua offers good support for functional programming, including nested functions with proper lexical scoping; just wait until Chapter 6. Finally, first-class functions play a key role in Lua’s object-oriented facilities, as we will see in Chapter 16.

Lua可以调用C写的函数。Lua中标准库都是C编写的。We will discuss Lua functions in Chapter 5 and C functions in Chapter 27.

### 2.7 Userdata 和 Threads

Userdata类型使得任意C类型可以被存储到Lua变量。It has no predefined operations in Lua, except assignment and equality test. Userdata are used to represent new types created by an application program or a library written in C; 例如，标准I/O库用它们表示打开的文件。We will discuss more about userdata later, when we get to the C API.

We will explain the thread type in Chapter 9, where we discuss coroutines.

## 4. 表达式

Expressions denote values. Expressions in Lua include the numeric constants and string literals, variables, unary and binary operations, and function calls. Expressions include also the unconventional function definitions and table constructors.

### 3.1 算术运算符

加减乘除乘方取模：‘+’、‘-’、‘*’、‘/’、‘^’、‘%’。All of them operate on real numbers. For instance, `x^0.5` computes the square root of x, while `x^(-1/3)` computes the inverse of its cubic root.

取模运算符的规则：

```lua
	a % b == a - math.floor(a/b)*b
```

For integer operands, it has the usual meaning, with the result always having the same sign as the second argument. For real operands, it has some extra uses. 例如，`x%1`是`x`的小数部分，`x-x%1`是整数部分。`x-x%0.01`是x精确到两位小数：

```lua
    x = math.pi
    print(x - x%0.01) --> 3.14
```

As another example of the use of the modulo operator, suppose you want to check whether a vehicle turning a given angle will start to backtrack. If the angle is given in degrees, you can use the following formula:

```lua
	local tolerance = 10
    function isturnback (angle)
    	angle = angle % 360
    	return (math.abs(angle - 180) < tolerance)
    end
```

This definition works even for negative angles:

```lua
	print(isturnback(-180)) --> true
```

If we want to work with radians instead of degrees, we simply change the constants in our function:

```lua
    local tolerance = 0.17
    function isturnback (angle)
    	angle = angle % (2*math.pi)
    	return (math.abs(angle - math.pi) < tolerance)
    end
```

The operation `angle%(2*math.pi)` is all we need to normalize any angle to a value in the interval `[0; 2Pi)`.

### 3.2 关系运算符

Lua provides the following relational operators:

```lua
	< > <= >= == ~=
```

All these operators always produce a boolean value.

`nil`只与自己相等。

table和userdata比较的是引用。

```lua
    a = {}; a.x = 1; a.y = 0
    b = {}; b.x = 1; b.y = 0
    c = a
```

you have a==c but a~=b.

顺序比较运算符只能用于数字或字符串。其他类型只能比较等或不等。Lua compares strings in alphabetical order, which follows the locale set for Lua. For instance, with a Portuguese Latin-1 locale, we have `"acai"<"açaí"<"acorde"`.

混合比较字符串和数字将报错。

### 3.3 逻辑运算符

逻辑运算符包括`and`, `or`, `not`。Like control structures, all logical operators consider both the boolean false and nil as false, and anything else as true.

如果第一个运算符为真，`and`返回第一个参数，否则返回第二个。如果第一个参数为假，`or`返回第一个参数，否则返回第二个：

```lua
    print(4 and 5) --> 5
    print(nil and 13) --> nil
    print(false and 13) --> false
    print(4 or 5) --> 4
    print(false or 5) --> 5
```

`and`和`or`都使用短路。

一个常见的有用的写法是`x = x or v`。等价于：

```lua
	if not x then x = v end
```

另一种写法是`a and b or c`。等价于C的`a ? b : c`。例如：

```lua
	max = (x > y) and x or y
```

`not`运算符总是返回布尔值：

```lua
    print(not nil) --> true
    print(not false) --> true
    print(not 0) --> false
    print(not not 1) --> true
    print(not not nil) --> false
```

### 3.4 连接

字符串连接运算符`..`。如果操作数是数字，Lua将其转换为字符串。

```lua
    print("Hello " .. "World") --> Hello World
    print(0 .. 1) --> 01
    print(000 .. 01) --> 01
```

### 3.5 长度运算符

长度运算符可以用于字符串或table。On strings, it gives the number of bytes in the string. On tables, it gives the length of the sequence represented by the table.

The length operator provides several common Lua idioms for manipulating sequences:

```lua
    print(a[#a]) -- prints the last value of sequence 'a'
    a[#a] = nil -- removes this last value
    a[#a + 1] = v -- appends 'v' to the end of the list
```
As we saw in the last chapter, the length operator is **unpredictable** for lists with holes (nils). It only works for sequences, which we defined as lists without holes. More precisely, a sequence is a table where the numeric keys comprise a set 1 ... n for some n. (Remember that any key with value nil is actually not in the table.) In particular, a table with no numeric keys is a sequence with length zero.

Over the years, there have been many proposals to extend the meaning of the length operator to lists with holes, but this extension is easier said than done. The problem is that, because a list is actually a table, the concept of “length” is somewhat fuzzy. For instance, consider the list resulting from the following code:

```lua
    a = {}
    a[1] = 1
    a[2] = nil -- does nothing, as a[2] is already nil
    a[3] = 1
    a[4] = 1
```

It is easy to say that the length of this list is four, and that is has a hole at index 2. However, what can we say about the next similar example?

```lua
    a = {}
    a[1] = 1
    a[10000] = 1
```

Should we consider a as a list with 10000 elements, where 9998 of them are nil? Now, the program does this:

```lua
	a[10000] = nil
```

What is the list length now? Should it be 9999, because the program deleted the last element? Or maybe still 10000, as the program only changed the last element to nil? Or should the length collapse to 1?

Another common proposal is to make the `#` operator return the total number of elements in the table. This semantics is clear and well defined, but not useful at all. Consider all previous examples and think how useful would be such operator for real algorithms over lists or arrays. Yet more troubling are nils at the end of the list. What should be the length of the following list?

```lua
	a = {10, 20, 30, nil, nil}
```

记住，在Lua中，一个值为nil的字段跟没有这个字段是没有区别的。Therefore, the previous table is equal to {10, 20, 30}; its length is 3, not 5.

You may consider that a nil at the end of a list is a very special case. However, many lists are built by adding elements one by one. Any list with holes that was built that way must have had nils at its end along the way. Most lists we use in our programs are sequences (e.g., a file line cannot be nil) and, therefore, most of the time the use of the length operator is safe. If you really need to handle lists with holes, 你需要显式的把长度存到某个地方。

### 3.6 优先级

Operator precedence in Lua follows the table below, from the higher to the lower priority:

    ^
    not # - (unary)
    * / %
    + -
    ..
    < > <= >= ~= ==
    and
    or

All binary operators are left associative, except for ‘^’ (exponentiation) and ‘..’ (concatenation), which are right associative.

### 3.7 Table构造

Constructors are expressions that create and initialize tables.

最简单的构造器是空构造器，`{}`，创建一个空表；Constructors also initialize lists. For instance, the statement

```lua
	days = {"Sunday", "Monday", "Tuesday", "Wednesday",
		"Thursday", "Friday", "Saturday"}
```

will initialize `days[1]` with the string “Sunday” (第一个元素的索引是1不是0！), days[2] with “Monday”, and so on:

```lua
	print(days[4]) --> Wednesday
```

Lua also offers a special syntax to initialize a table record-like, as in the next example:

```lua
	a = {x=10, y=20}
```

This previous line is equivalent to these commands:

```lua
	a = {}; a.x=10; a.y=20
```

The original expression, however, is faster, because Lua creates the table already with the right size.

We can mix record-style and list-style initializations in the same constructor:

```lua
    polyline = {color="blue",
        thickness=2,
        npoints=4,
        {x=0, y=0}, -- polyline[1]
        {x=-10, y=0}, -- polyline[2]
        {x=-10, y=1}, -- polyline[3]
        {x=0, y=1} -- polyline[4]
    }
```

Those two constructor forms have their limitations. For instance, you cannot initialize fields with negative indices, nor with string indices that are not proper identifiers. For such needs, there is another, more general, format. In this format, we explicitly write the index to be initialized as an expression, between square brackets:

```lua
	opnames = {["+"] = "add", ["-"] = "sub",
		["*"] = "mul", ["/"] = "div"}
		i = 20; s = "-"
		a = {[i+0] = s, [i+1] = s..s, [i+2] = s..s..s}
		print(opnames[s]) --> sub
		print(a[22]) --> ---
```

This syntax is more cumbersome, but more flexible too: both the list-style and the record-style forms are special cases of this more general syntax. The constructor `{x=0 ,y=0}` is equivalent to `{["x"]=0,["y"]=0}`, and the constructor `{"r","g","b"}` is equivalent to `{[1]="r",[2]="g",[3]="b"}`.

You can always put a comma after the last entry. These trailing commas are optional, but are always valid:

```lua
	a = {[1]="red", [2]="green", [3]="blue",}
```

构造器中，逗号可以任意的被替换为分号：

```lua
	{x=10, y=45; "one", "two", "three"}
```

## 4. 语句

### 4.1 赋值

```lua
	a = "hello" .. "world"
	t.n = t.n + 1
```

Lua允许多赋值，即把一组值赋给一组变量。列表用逗号分隔。

```lua
	a, b = 10, 2*x
```

Lua先求所有值，再执行赋值。因此多赋值可以用于交换值：

```lua
	x, y = y, x -- swap 'x' for 'y'
	a[i], a[j] = a[j], a[i] -- swap 'a[i]' for 'a[j]'
```

如果值比变量数量少，不够的变量被赋予nil。如果值比变量多，多个值将被丢弃：

```lua
    a, b, c = 0, 1
    print(a, b, c) --> 0 1 nil
    a, b = a+1, b+1, b+2 -- value of b+2 is ignored
    print(a, b) --> 1 2
    a, b, c = 0
    print(a, b, c) --> 0 nil nil
```

多赋值的意义在于可以返回多个值，见5.1节。

### 4.2 局部变量与块

除了全局变量，Lua还支持局部变量。We create local variables with the local statement:

```lua
	j = 10 -- global variable
	local i = 1 -- local variable
```

Unlike global variables, local variables have their scope limited to the block where they are declared. A block is the body of a control structure, the body of a function, or a chunk (the file or string where the variable is declared):

```lua
    x = 10
    local i = 1 -- local to the chunk
    while i <= x do
    	local x = i*2 -- local to the while body
    	print(x) --> 2, 4, 6, 8, ...
    	i = i + 1
    end
    if i > 20 then
    	local x -- local to the "then" body
    	x = 20
    	print(x + 2) -- (would print 22 if test succeeded)
    else
    	print(x) --> 10 (the global one)
    end
    print(x) --> 10 (the global one)
```

Beware that this example will not work as expected if you enter it in interactive mode. In interactive mode, each line is a chunk by itself (unless it is not a complete command). As soon as you enter the second line of the example (local i=1), Lua runs it and starts a new chunk in the next line. By then, the local declaration is already out of scope. To solve this problem, we can delimit the whole block explicitly, bracketing it with the keywords `do–end`. Once you enter the `do`, the command completes only at the corresponding `end`, so Lua does not execute each line by itself.

These do blocks are useful also when you need finer control over the scope of some local variables:

```lua
    do
    	local a2 = 2*a
    	local d = (b^2 - 4*a*c)^(1/2)
    	x1 = (-b + d)/a2
    	x2 = (-b - d)/a2
    end -- scope of 'a2' and 'd' ends here
    print(x1, x2)
```

It is good programming style to use local variables whenever possible. Local variables help you avoid cluttering the global environment with unnecessary names. 而且访问局部变量与全局变量快。Finally, a local variable vanishes as soon as its scope ends, allowing the garbage collector to release its value.

Lua handles local-variable declarations as statements. As such, you can write local declarations anywhere you can write a statement. The scope of the declared variables begins after the declaration and goes until the end of the block. Each declaration can include an initial assignment, which works the same way as a conventional assignment. 如果声明后面没有显式的赋值，变量将被赋予`nil`：

```lua
    local a, b = 1, 10
    if a < b then
    	print(a) --> 1
    	local a -- '= nil' is implicit
    	print(a) --> nil
    end -- ends the block started at 'then'
    print(a, b) --> 1 10
```

```lua
	local foo = foo
```

上面的代码创建局部变量`foo`，并用全局变量`foo`初始化。

### 4.3 控制结构

所有的控制结构都有显式的结尾：`end`终止if, for and while。`until`终止repeat。

记得，Lua中，0和空串在条件中是true。

#### if then else

```lua
	if a < 0 then a = 0 end
	if a < b then return a else return b end
	if line > MAXLINES then
		showpage()
		line = 0
	end

    if op == "+" then
    	r = a + b
    elseif op == "-" then
    	r = a - b
    elseif op == "*" then
    	r = a*b
    elseif op == "/" then
    	r = a/b
    else
    	error("invalid operation")
    end
```

#### while

```lua
	local i = 1
	while a[i] do
		print(a[i])
		i = i + 1
	end

#### repeat

```lua
    -- print the first non-empty input line
    repeat
    	line = io.read()
    until line ~= ""
    print(line)
```

Unlike in most other languages, in Lua the scope of a local variable declared inside the loop includes the condition:

```lua
	local sqr = x/2
	repeat
		sqr = (sqr + x/sqr)/2
		local error = math.abs(sqr^2 - x)
    until error < x/10000 -- local 'error' still visible here
```

#### 数字for

for语句有两个变体：数字for和通用for。数字for的语法如下：

```lua
	for var = exp1, exp2, exp3 do
		<something>
	end
```

值从exp1开始，到exp2，步长是exp3。若exp3省略，则步长为1。

```lua
	for i = 1, f(x) do print(i) end
	for i = 10, 1, -1 do print(i) end
```

若循环没有上界，可以使用常量`math.huge`：

```lua
	for i = 1, math.huge do
		if (0.3*i^3 - 20*i^2 - 500 >= 0) then
			print(i)
			break
		end
	end
```

三个表达式在循环开始前会被求值一次。控制变量是局部变量，作用域在for循环内。循环结束后不存在！

若在循环结束后，想知道控制变量的值，需要一个额外的变量，适时将控制变量赋给它：

```lua
    -- find a value in a list
    local found = nil
    for i = 1, #a do
        if a[i] < 0 then
            found = i -- save value of 'i'
            break
        end
    end
    print(found)
```

不要改变控制变量的值。如果需要退出循环，应使用`break`。

#### 通用for

通用for遍历迭代函数返回的值：

```lua
	-- print all values of table 't'
	for k, v in pairs(t) do print(k, v) end
```

`pairs`是一个遍历table的迭代器。

标准库提供了几个迭代器，which allow us to iterate over the lines of a file (`io.lines`), the pairs of a table (`pairs`), the entries of a sequence (`ipairs`), the words of a string (`string.gmatch`), and so on.

可以编写自己的迭代器。见Chapter 7。

通用for的控制变量也是局部变量。不能改变它们的值。

构建反向表。如

```lua
	days = {"Sunday", "Monday", "Tuesday", "Wednesday",
		"Thursday", "Friday", "Saturday"}

的反向表是：

```lua
	revDays = {["Sunday"] = 1, ["Monday"] = 2,
        ["Tuesday"] = 3, ["Wednesday"] = 4,
        ["Thursday"] = 5, ["Friday"] = 6,
        ["Saturday"] = 7}
```

```lua
	revDays = {}
	for k, v in pairs(days) do
		revDays[v] = k
	end
```

#### 4.4 break, return, and goto

处于语法原因，return语句只能是块的最后一条语句｛｛否则，后续语句没有可能执行｝｝。

A goto statement jumps the execution of a program to a corresponding
label. There has been a long going debate about goto, with some people arguing even today that they are harmful to programming and should be banned from programming languages. Nonetheless, several current languages offer goto, with good reason. They are a powerful mechanism and, when used with care, can only improve the quality of your code.

In Lua, the syntax for a goto statement is quite conventional: it is the reserved word goto followed by the label name, which can be any valid identifier. The syntax for a label is a little more convoluted: it has two colons followed by the label name followed by more two colons, like in `::name::`. This convolution is intentional, to make programmers think twice before using a goto.

Lua poses some restrictions to where you can jump with a goto. First, labels follow the usual visibility rules, so you cannot jump into a block (because a label inside a block is not visible outside it). Second, you cannot jump out of a function. (Note that the first rule already excludes the possibility of jumping into a function.) Third, you cannot jump into the scope of a local variable.

A typical and well-behaved use of a goto is to simulate some construction that you learned from another language but that is absent from Lua, such as continue, multi-level break, multi-level continue, redo, local error handling, etc.

A `continue` statement is simply a goto to a label at the end of a loop block; a redo statement jumps to the beginning of the block:

```lua
    while some_condition do
        ::redo::
        if some_other_condition then goto continue
        else if yet_another_condition then goto redo
        end
        <some code>
        ::continue::
    end
```

A useful detail in the specification of Lua is that the scope of a local variable ends on the last non-void statement of the block where the variable is defined; labels are considered void statements. To see the usefulness of this detail, consider the next fragment:

```lua
    while some_condition do
    	if some_other_condition then goto continue end
        local var = something
    	<some code>
    	::continue::
    end
```

You may think that this goto jumps into the scope of variable var. However, the continue label appears after the last non-void statement of the block, and therefore it is not inside the scope of var. The goto is also useful to write state machines. As an example, Listing 4.1 shows a program that checks whether its input has an even number of zeros. There are better ways to write this specific program, but this technique is useful if you want to translate a finite automata into Lua code automatically (think about dynamic code generation).

As another example, let us consider a simple maze game. The maze has several rooms, each with up to four doors: north, south, east, and west. At each step, the user enters a movement direction. If there is a door in this direction, the user goes to the corresponding room; otherwise, the program prints a warning. The goal is to go from an initial room to a final room. This game is a typical state machine, where the current room is the state. We can implement this maze with one block for each room, using a goto to move from one room to another. Listing 4.2 shows how we could write a small maze with four rooms. For this simple game, you may find that a data-driven program, where you describe the rooms and movements with tables, is a better design. However, if the game has several special situations in each room, then this state-machine design is quite appropriate.

Listing 4.1. An example of a state machine with goto:

```lua
::s1:: do
local c = io.read(1)
if c == '0' then goto s2
elseif c == nil then print'ok'; return
else goto s1
end
end
::s2:: do
local c = io.read(1)
if c == '0' then goto s1
elseif c == nil then print'not ok'; return
else goto s2
end
end
goto s1
```

Listing 4.2. A maze game:

```lua
goto room1 -- initial room
::room1:: do
local move = io.read()
if move == "south" then goto room3
elseif move == "east" then goto room2
else
print("invalid move")
goto room1 -- stay in the same room
end
end
::room2:: do
local move = io.read()
if move == "south" then goto room4
elseif move == "west" then goto room1
else
print("invalid move")
goto room2
end
end
::room3:: do
local move = io.read()
if move == "north" then goto room1
elseif move == "east" then goto room4
else
print("invalid move")
goto room3
end
end
::room4:: do
print("Congratulations, you won!")
end
```

Listing 4.3. A strange (and invalid) use of a goto:

```lua
function getlabel ()
return function () goto L1 end
::L1::
return 0
end
function f (n)
if n == 0 then return getlabel()
else
local res = f(n - 1)
print(n)
return res
end
end
x = f(10)
x()
```

## 5. 函数

如果函数只有一个参数。且实参是字符串字面量或Table构造器，则括号是可选的：

```lua
	print "Hello World" <--> print("Hello World")
	dofile 'a.lua' <--> dofile ('a.lua')
	print [[a multi-line <--> print([[a multi-line
    message]] message]])
	f{x=10, y=20} <--> f({x=10, y=20})
	type{} <--> type({})
```

Lua also offers a special syntax for object-oriented calls, the colon operator. An expression like `o:foo(x)` is just another way to write `o.foo(o,x)`, that is, to call `o.foo` adding `o` as a first extra argument. In Chapter 16, we will discuss such calls (and object-oriented programming) in more detail.

Lua程序可以使用C编写的函数（或宿主程序使用的其他语言）。例如，标准库中的函数都是C写的。调用函数时，C编写的函数与Lua编写的函数没有区别。

函数定义：

```lua
    -- add the elements of sequence 'a'
    function add (a)
    	local sum = 0
    	for i = 1, #a do
    		sum = sum + a[i]
    	end
    	return sum
    end
```

实参和形参数量可以不同。规则与多赋值一样。

```lua
    function f (a, b) print(a, b) end

	f(3) --> 3 nil
    f(3, 4) --> 3 4
    f(3, 4, 5) --> 3 4 (5 is discarded)
```

### 5.1 多个结果

Lua函数可以返回多个结果。

```lua
	s, e = string.find("hello Lua users", "Lua")
	print(s, e) --> 7 9
```

（注意字符串的第一个字符的下标是1。）

定义返回多值的函数：

```lua
    function maximum (a)
        local mi = 1 -- index of the maximum value
        local m = a[mi] -- maximum value
        for i = 1, #a do
        	if a[i] > m then
                mi = i; m = a[i]
            end
        end
        return m, mi
    end
    print(maximum({8, 10, 23, 12, 5})) --> 23 3
```

```lua
    function foo0 () end -- returns no results
    function foo1 () return "a" end -- returns 1 result
    function foo2 () return "a", "b" end -- returns 2 results
```

返回多值与多赋值：

```lua
x,y,z = 10, foo2() -- x=10, y="a", z="b"
```

值列表中，函数调用后面的值无效：

```lua
x, y = foo2(), 20 -- x="a", y=20
x, y = foo0(), 20, 30 -- x = nil, y=20, 30 is discarded
```

When a function call is the last (or the only) argument to another call, all results from the first call go as arguments. We have seen examples of this construction already, with print. Because the print function can receive a variable number of arguments, the statement `print(g())` prints all results returned by g.

```lua
print(foo0()) -->
print(foo1()) --> a
print(foo2()) --> a b
print(foo2(), 1) --> a 1
print(foo2() .. "x") --> ax (see next)
```

When the call to `foo2` appears inside an expression, Lua adjusts the number of results to one; so, in the last line, the concatenation uses only the “a”.

If we write `f(g())` and f has a fixed number of arguments, Lua adjusts the number of results of g to the number of parameters of f, as we saw previously.

A constructor also collects all results from a call, without any adjustments:

```lua
    t = {foo0()} -- t = {} (an empty table)
    t = {foo1()} -- t = {"a"}
    t = {foo2()} -- t = {"a", "b"}
```

As always, this behavior happens only when the call is the last expression in the list; calls in any other position produce exactly one result:

```lua
	t = {foo0(), foo2(), 4} -- t[1] = nil, t[2] = "a", t[3] = 4
```

Finally, a statement like `return f()` returns all values returned by f:

```lua
    function foo (i)
        if i == 0 then return foo0()
        elseif i == 1 then return foo1()
        elseif i == 2 then return foo2()
        end
    end
    print(foo(1)) --> a
    print(foo(2)) --> a b
    print(foo(0)) -- (no results)
    print(foo(3)) -- (no results)
```

You can force a call to return exactly one result by enclosing it in an extra pair of parentheses:

```lua
    print((foo0())) --> nil
    print((foo1())) --> a
    print((foo2())) --> a
```

Beware that a return statement does not need parentheses around the returned value; any pair of parentheses placed there counts as an extra pair. Therefore, a statement like `return(f(x))` always returns one single value, no matter how many values f returns. Sometimes this is what you want, sometimes not.

A special function with multiple returns is `table.unpack`. It receives an array and returns as results all elements from the array, starting from index 1:

```lua
	print(table.unpack{10,20,30}) --> 10 20 30
	a,b = table.unpack{10,20,30} -- a=10, b=20, 30 is discarded
```

An important use for `unpack` is in a generic call mechanism. A generic call mechanism allows you to call any function, with any arguments, dynamically. In ANSI C, for instance, there is no way to code a generic call. You can declare a function that receives a variable number of arguments (with `stdarg.h`) and you can call a variable function, using pointers to functions. However, you cannot call a function with a variable number of arguments: each call you write in C has a fixed number of arguments, and each argument has a fixed type. In Lua, if you want to call a variable function `f` with variable arguments in an array a, you simply write this:

```lua
	f(table.unpack(a))
```

The call to unpack returns all values in `a`, which become the arguments to f. For instance, consider the following call:

```lua
	print(string.find("hello", "ll"))
```

You can dynamically build an equivalent call with the following code:

```lua
	f = string.find
	a = {"hello", "ll"}
	print(f(table.unpack(a)))
```

`unpack`利用长度运算符确定元素数量，so it works only on proper sequences. 若需要，可以显式限定范围：

```lua
	print(table.unpack({"Sun", "Mon", "Tue", "Wed"}, 2, 3))
	--> Mon Tue
```

Although the predefined `unpack` function is written in C, we could write it also in Lua, using recursion:

```lua
    function unpack (t, i, n)
    	i = i or 1
    	n = n or #t
    	if i <= n then
    		return t[i], unpack(t, i + 1, n)
    	end
    end
```

The first time we call it, with a single argument, i gets 1 and n gets the length of the sequence. Then the function returns t[1] followed by all results from `unpack(t,2,n)`, which in turn returns t[2] followed by all results from `unpack(t,3,n)`, and so on, stopping after n elements.

### 5.2 Variadic Functions

A function in Lua can be variadic, that is, it can receive a variable number of arguments. Although print is defined in C, we can define variadic functions in Lua, too.

As a simple example, the following function returns the summation of all its arguments:

```lua
    function add (...)
        local s = 0
        for i, v in ipairs{...} do
        	s = s + v
        end
        return s
    end
    print(add(3, 4, 10, 25, 12)) --> 54
```

The three dots (`...`) in the parameter list indicate that the function is variadic. When we call this function, Lua collects all its arguments internally; we call these collected arguments the extra arguments of the function. A function can access its extra arguments using again the three dots, now as an expression. In our example, the expression `{...}` results in an array with all collected arguments. The function then traverses the array to add its elements.

We call the expression `...` a vararg expression. It behaves like a multiple return function, returning all extra arguments of the current function. For instance, the command `print(...)` prints all extra arguments of the function.

Likewise, the next command creates two local variables with the values of the first two optional arguments (or nil if there are no such arguments):

```lua
	local a, b = ...
```

Actually, we can emulate the usual parameter-passing mechanism of Lua translating function `foo (a, b, c)` to 

```lua
	function foo (...)
		local a, b, c = ...
```

A function like the next one simply returns all arguments in its call:

```lua
	function id (...) return ... end
```

It is a multi-value identity function. The next function behaves exactly like another function foo, except that before the call it prints a message with its arguments:

```lua
	function foo1 (...)
		print("calling foo:", ...)
		return foo(...)
	end
```

这个特性可以用于包装函数。

Let us see another useful example. Lua provides separate functions for formatting text (`string.format`) and for writing text (`io.write`). It is straightforward to combine both functions into a single variadic function:

```lua
    function fwrite (fmt, ...)
        return io.write(string.format(fmt, ...))
    end
```

Variadic functions can have any number of fixed parameters before the variadic part. Lua assigns the first arguments to these parameters; the rest (if any) goes as extra arguments.

Below we show some examples of calls and the corresponding parameter
values:

```lua
    fwrite() -- fmt = nil, no extra arguments
    fwrite("a") -- fmt = "a", no extras
    fwrite("%d%d", 4, 5) -- fmt = "%d%d", extras = 4 and 5
```

(Note that the call fwrite() raises an error, because string.format needs a string as its first argument.)

To iterate over its extra arguments, a function can use the expression `{...}` to collect them all in a table, as we did in our definition of add.

In the rare occasions when the extra arguments can be valid nils, however, the table created with {...} may not be a proper sequence. For instance, there is no way to detect in such a table whether there were trailing nils in the original arguments. For these occasions, Lua offers the `table.pack` function.（Lua 5.2加入） This function receives any number of arguments and returns a new table with all its arguments, just like `{...}`; but this table has also an extra field “n”, with the total number of arguments. The following function uses `table.pack` to test whether none of its arguments is `nil`:

```lua
    function nonils (...)
        local arg = table.pack(...)
        for i = 1, arg.n do
        	if arg[i] == nil then return false end
        end
        return true
    end
    print(nonils(2,3,nil)) --> false
    print(nonils(2,3)) --> true
    print(nonils()) --> true
    print(nonils(nil)) --> false
```

Remember, however, that `{...}` is cleaner and faster than `table.pack(...)` when the extra arguments cannot be `nil`.

### 5.3 具名实参

Lua中参数传递是基于位置的。Sometimes, however, it is useful to specify the arguments by name. To illustrate this point, let us consider the function `os.rename`, which renames a file. Quite often, we forget which name comes first, the new or the old; therefore, we may want to redefine this function to receive two named arguments:

```lua
	-- 无效代码
	rename(old="temp.lua", new="temp1.lua")
```

Lua has no direct support for this syntax, but we can have the same final effect, with a small syntax change. 方法是将所有的实参打包进一个table，函数只取一个参数，即这个Table。 The special syntax that Lua provides for function calls, with just one table constructor as argument, helps the trick:

```lua
	rename{old="temp.lua", new="temp1.lua"}
```

`rename`的定义：

```lua
	function rename (arg)
		return os.rename(arg.old, arg.new)
	end
```

## 6. 函数深入

















