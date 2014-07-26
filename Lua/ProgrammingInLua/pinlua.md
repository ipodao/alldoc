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








