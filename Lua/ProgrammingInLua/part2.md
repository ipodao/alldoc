[toc]

## Part II Tables and Objects

## 11. 数据结构

本章的内容是，如何用table实现各种常见的数据结构，如数组、记录、列表、队列、集合等。More to the point, Lua tables implement all these structures efficiently.

For instance, we seldom write a search in Lua, because tables offer direct access to any type.

### 11.1 数组

在Lua中实现数组，只要用整数作为Table的索引。数组没有固定大小，按需增长。

```lua
    a = {} -- new array
    for i = 1, 1000 do
    	a[i] = 0
    end
```

数组的下标可以从任何值开始，不仅限于0或1：

```lua
    -- creates an array with indices from -5 to 5
    a = {}
    for i = -5, 5 do
    	a[i] = 0
    end
```

但在Lua中，习惯以1开始。Lua库依赖此约定。长度运算符也是。If your arrays do not start with 1, you will not be able to use these facilities.

利用构造器可以创建并同时初始化数组：

```lua
	squares = {1, 4, 9, 16, 25, 36, 49, 64, 81}
```

### 11.2 矩阵和多维数组

使用数组的数组：即Table的元素是另一个Table。例如：

```lua
    mt = {} -- create the matrix
    for i = 1, N do
        mt[i] = {} -- create a new row
        for j = 1, M do
        	mt[i][j] = 0
        end
    end
```

对于稀疏数组（内部有`nil`值），不能用长度运算符。不过，这没什么，因为从头到尾遍历稀疏数组是无效的。应该使用`pairs`直接遍历有值的元素。

```lua
    function mult (a, rowindex, k)
        local row = a[rowindex]
        for i, v in pairs(row) do
        	row[i] = v * k
        end
    end
```

注意Table没有本质顺序（intrinsic order），因此`pairs`不保证按顺序。若要顺序，可能需要链表等机制。

### 11.3 链表

Because tables are dynamic entities, it is easy to implement linked lists in Lua. Each node is represented by a table and links are simply table fields that contain references to other tables. For instance, let us implement a basic list, where each node has two fields, `next` and `value`. A simple variable is the list root:

```lua
	list = nil
```

To insert an element at the beginning of the list, with a value v, we do:

```lua
	list = {next = list, value = v}
```
To traverse the list, we write:

```lua
    local l = list
    while l do
    	<visit l.value>
    	l = l.next
    end
```

Other kinds of lists, such as double-linked lists or circular lists, are also implemented easily. However, you seldom need those structures in Lua, because usually there is a simpler way to represent your data without using linked lists. For instance, we can represent a stack with an (unbounded) array.

### 11.4 队列和双向队列

A simple way to implement queues in Lua is with functions `insert` and `remove` from the table library. These functions insert and remove elements in any position of an array, moving other elements to accommodate the operation.

However, these moves can be **expensive** for large structures. A more efficient implementation uses two indices, one for the first element and another for the last:

```lua
    function ListNew ()
    	return {first = 0, last = -1}
    end
```

To avoid polluting the global space, we will define all list operations inside a table, properly called `List` (that is, we will create a module). Therefore, we rewrite our last example like this:

```lua
    List = {}
    function List.new ()
    	return {first = 0, last = -1}
    end
```

Now, we can insert or remove an element at both ends in constant time:

```lua
    function List.pushfirst (list, value)
        local first = list.first - 1
        list.first = first
        list[first] = value
    end
    function List.pushlast (list, value)
        local last = list.last + 1
        list.last = last
        list[last] = value
    end
    function List.popfirst (list)
        local first = list.first
        if first > list.last then error("list is empty") end
        local value = list[first]
        list[first] = nil -- to allow garbage collection
        list.first = first + 1
        return value
    end
    function List.poplast (list)
        local last = list.last
        if list.first > last then error("list is empty") end
        local value = list[last]
        list[last] = nil -- to allow garbage collection
        list.last = last - 1
        return value
    end
```

If you use this structure in a strict queue discipline, calling only pushlast and popfirst, both first and last will increase continually. However, because we represent arrays in Lua with tables, you can index them either from 1 to 20 or from 16777216 to 16777236. Because Lua uses double precision to represent numbers, your program can run for two hundred years, doing one million insertions per second, before it has problems with overflows.

### 11.5 Sets 和 Bags

Suppose you want to list all identifiers used in a program source; somehow you need to filter the reserved words out of your listing. Some C programmers could be tempted to represent the set of reserved words as an array of strings, and then to search this array to know whether a given word is in the set. To speed up the search, they could even use a binary tree to represent the set.

In Lua, an efficient and simple way to represent such sets is to put the set elements as indices in a table. Then, instead of searching the table for a given element, you just index the table and test whether the result is `nil` or not. In our example, we could write the next code:

```lua
    reserved = {
    	["while"] = true, ["end"] = true,
    	["function"] = true, ["local"] = true,
    }
    for w in allwords() do
    	if not reserved[w] then
    		<do something with ’w’> -- 'w' is not a reserved word
    	end
    end
```

(Because these words are reserved in Lua, we cannot use them as identifiers; for instance, we cannot write `while=true`. Instead, we use the `["while"]=true` notation.)

You can have a clearer initialization using an auxiliary function to build the set:

```lua
    function Set (list)
        local set = {}
        for _, l in ipairs(list) do set[l] = true end
        	return set
    end
    reserved = Set{"while", "end", "function", "local", }
```

Bags, also called multisets, differ from regular sets in that each element can appear multiple times. An easy representation for bags in Lua is similar to the previous representation for sets, but with a counter associated to each key. To insert an element we increment its counter:

```lua
    function insert (bag, element)
    	bag[element] = (bag[element] or 0) + 1
    end
```

To remove an element we decrement its counter:

```lua
    function remove (bag, element)
    	local count = bag[element]
    	bag[element] = (count and count > 1) and count - 1 or nil
    end
```

We only keep the counter if it already exists and it is still greater than zero.

### （未）11.6 String Buffers

### （未）11.7 Graphs

## （未）12. 数据文件与持久化

In this chapter, we will see how we can use Lua to eliminate all code for reading data from our programs, simply by writing the data in an appropriate format.

## 13. Metatables 和 Metamethods

Lua中每个值能执行什么操作是确定的。例如，两个数可以相加，两个表却不能相加。除非使用metatables，

Metatables allow us to change the behavior of a value when confronted with an undefined operation. 例如，使用metatables，我们可以定义如何让两个表相加：当Lua尝试两个表相加时，它会检查是否有一个表有metatable，且这个原表是否有`__add`字段。如果有，则字段的值，被称为metamethod，是一个函数，用于计算和。

Lua中每个值都可以关联元表。Tables and userdata have individual metatables; values of other types share one single metatable for all values of that type. 新创建的Table没有元表：

```lua
    t = {}
    print(getmetatable(t)) --> nil
```

可以通过`setmetatable`设置或改变任何表的元表：

```lua
    t1 = {}
    setmetatable(t, t1)
    print(getmetatable(t) == t1) --> true
```

通过Lua语言只能操纵表的元表；操纵其他类型值的元表只能通过C代码。(The main reason for this restriction is to curb excessive use of type-wide metatables. Experience with older versions of Lua has shown that those global settings frequently lead to non-reusable code.) As we will see later, in Chapter 21, the string library sets a metatable for strings. 其他类型默认没有元表：

```lua
    print(getmetatable("hi")) --> table: 0x80772e0
    print(getmetatable("xuxu")) --> table: 0x80772e0
    print(getmetatable(10)) --> nil
    print(getmetatable(print)) --> nil
```

Any table can be the metatable of any value; a group of related tables can share a common metatable, which describes their common behavior; a table can be its own metatable, so that it describes its own individual behavior. Any configuration is valid.

### 13.1 算术元方法

本节引入一个i额就爱你的的例子解释如何使用元表。Suppose we are using tables to represent sets, with functions to compute set union, intersection, and the like, as shown in Listing 13.1. To keep our namespace clean, we store these functions inside a table called `Set`.

Now, we want to use the addition operator (`+`) to compute the union of two sets. 为此所有表示集合的Table将共享一个元表。This metatable will define how they react to the addition operator. Our first step is to create a regular table, which we will use as the metatable for sets:

```lua
	local mt = {} -- metatable for sets
```

The next step is to modify the `Set.new` function, which creates sets. The new version has only one extra line, which sets `mt` as the metatable for the tables that it creates:

```lua
    function Set.new (l) -- 2nd version
        local set = {}
        setmetatable(set, mt)
        for _, v in ipairs(l) do set[v] = true end
        return set
    end
```

After that, every set we create with `Set.new` will have that same table as its metatable:

```lua
    s1 = Set.new{10, 20, 30, 50}
    s2 = Set.new{30, 1}
    print(getmetatable(s1)) --> table: 00672B60
    print(getmetatable(s2)) --> table: 00672B60
```

Finally, we add to the metatable the metamethod, a field `__add` that describes how to perform the addition:

```lua
	mt.__add = Set.union
```

Listing 13.1. A simple implementation for sets:
```lua
    Set = {}
    -- create a new set with the values of a given list
    function Set.new (l)
        local set = {}
        for _, v in ipairs(l) do set[v] = true end
        return set
    end
    function Set.union (a, b)
        local res = Set.new{}
        for k in pairs(a) do res[k] = true end
        for k in pairs(b) do res[k] = true end
        return res
    end
    function Set.intersection (a, b)
        local res = Set.new{}
        for k in pairs(a) do
        	res[k] = b[k]
        end
        return res
    end
    -- presents a set as a string
    function Set.tostring (set)
        local l = {} -- list to put all elements from the set
        for e in pairs(set) do
        	l[#l + 1] = e
        end
        return "{" .. table.concat(l, ", ") .. "}"
    end
    -- print a set
    function Set.print (s)
    	print(Set.tostring(s))
    end
```

After that, whenever Lua tries to add two sets it will call the `Set.union` function, with the two operands as arguments.

With the metamethod in place, we can use the addition operator to do set unions:

```lua
    s3 = s1 + s2
    Set.print(s3) --> {1, 10, 20, 30, 50}
```

Similarly, we may set the multiplication operator to perform set intersection:

```lua
    mt.__mul = Set.intersection
    Set.print((s1 + s2)*s1) --> {10, 20, 30, 50}
```

For each arithmetic operator there is a corresponding field name in a metatable. Besides `__add` and `__mul`, there are `__sub` (for subtraction), `__div` (for division), `__unm` (for negation), `__mod` (for modulo), and `__pow` (for exponentiation). We may define also the field `__concat`, to describe a behavior for the concatenation operator.

When we add two sets, there is no question about what metatable to use. However, we may write an expression that mixes two values with different metatables, for instance like this:

```lua
	s = Set.new{1,2,3}
	s = s + 8
```

When looking for a metamethod, Lua does the following steps: if the first value has a metatable with an `__add` field, Lua uses this field as the metamethod, independently of the second value; otherwise, if the second value has a metatable with an `__add` field, Lua uses this field as the metamethod; otherwise, Lua raises an error.

Lua does not care about these mixed types, but our implementation does. If we run the `s=s+8` example, the error we get will be inside `Set.union`: `bad argument #1 to 'pairs' (table expected, got number)`

If we want more lucid error messages, we must check the type of the operands explicitly before attempting to perform the operation:

```lua
    function Set.union (a, b)
    	if getmetatable(a) ~= mt or getmetatable(b) ~= mt then
    		error("attempt to 'add' a set with a non-set value", 2)
        end
    <as before>
```

Remember that the second argument to error (2, in this example) directs the error message to where the operation was called.

### 13.2 关系元方法

Metatables also allow us to give meaning to the relational operators, through the metamethods `__eq` (equal to), `__lt` (less than), and `__le` (less than or equal to). There are no separate metamethods for the other three relational operators:

Lua translates `a~=b` to `not(a==b)`, `a>b` to `b<a`, and `a>=b` to `b<=a`. Until version 4.0, Lua translated all order operators to a single one, by translating `a<=b` to `not(b<a)`. However, this translation is incorrect when we have a partial order, that is, when not all elements in our type are properly ordered. For instance, floating-point numbers are not totally ordered in most machines, because of the value Not a Number (**NaN**). According to the IEEE 754 standard, NaN represents undefined values, such as the result of `0/0`. The standard specifies that any comparison that involves NaN should result in false.

This means that `NaN<=x` is always false, but `x<NaN` is also false. It also implies that the translation from `a<=b` to `not(b<a)` is not valid in this case. In our example with sets, we have a similar problem. An obvious (and useful) meaning for `<=` in sets is set containment: `a<=b` means that a is a subset of b. With this meaning, again it is possible that both `a<=b` and `b<a` are false; therefore, we need separate implementations for `__le` (less or equal) and `__lt` (less than):

```lua
    mt.__le = function (a, b) -- set containment
        for k in pairs(a) do
        	if not b[k] then return false end
        end
        return true
    end
    mt.__lt = function (a, b)
        return a <= b and not (b <= a)
    end
```

Finally, we can define set equality through set containment:

```lua
    mt.__eq = function (a, b)
    	return a <= b and b <= a
    end
```

After these definitions, we are ready to compare sets:

```lua
    s1 = Set.new{2, 4}
    s2 = Set.new{4, 10, 2}
    print(s1 <= s2) --> true
    print(s1 < s2) --> true
    print(s1 >= s1) --> true
    print(s1 > s1) --> false
    print(s1 == s2 * s1) --> true
```

For types that have a complete order, we do not need to define a `__leq` metamethod. In its absence, Lua will use the `__lt` entry. The equality comparison has some restrictions. If two objects have different basic types or different metamethods, the equality operation results in false, without even calling any metamethod. So, a set will always be different from a number, no matter what its metamethod says.

### 13.3 库定义的元方法

So far, all the metamethods we have seen are for the Lua core. It is the virtual machine that detects that the values involved in an operation have metatables with metamethods for that particular operation. However, because metatables are regular tables, anyone can use them. So, it is a common practice for libraries to define their own fields in metatables.

Function `tostring` provides a typical example. As we saw earlier, `tostring` represents tables in a rather simple format:

```lua
	print({}) --> table: 0x8062ac0
```

Function print always calls `tostring` to format its output. However, when formatting any value, tostring first checks whether the value has a `__tostring` metamethod. In this case, tostring calls the metamethod to do its job, passing the object as an argument. Whatever this metamethod returns is the result of tostring.

In our example with sets, we have already defined a function to present a set as a string. So, we need only to set the `__tostring` field in the metatable:

```lua
	mt.__tostring = Set.tostring
```

After that, whenever we call print with a set as its argument, print calls `tostring` that calls `Set.tostring`:

```lua
	s1 = Set.new{10, 4, 5}
	print(s1) --> {4, 5, 10}
```

Functions `setmetatable` and `getmetatable` also use a metafield, in this case to protect metatables. Suppose you want to protect your sets, so that users can neither see nor change their metatables. If you set a `__metatable` field in the metatable, `getmetatable` will return the value of this field, whereas `setmetatable` will raise an error:

```lua
    mt.__metatable = "not your business"
    s1 = Set.new{}
    print(getmetatable(s1)) --> not your business
    setmetatable(s1, {})
    stdin:1: cannot change protected metatable
```

In Lua 5.2, `pairs` and `ipairs` also got metatables, so that a table can modify the way it is traversed (and non-table objects can be traversed).

### （未）13.4 Table-Access Metamethods

## 14. （未）环境

Lua将所有的全局变量放在一个表中，称为全局环境。（更准确的锁，Lua将环境变量放在多个环境中，但我们先暂时简化成一个）。我们可以像操纵其他表一样操纵这个表。Lua将环境自己存放在全局变量`_G`中。于是`_G._G`等价于`_G`。

下面的代码邪乎所有全局变量的名字：

```lua
	for n in pairs(_G) do print(n) end
```

### 14.1 Global Variables with Dynamic Names

没什么。

### 14.2 全局变量声明

Lua中的全局变量不需要声明。于是用户有拼错的风险且很难觉察。为防止这种问题，有多种方法。我们可以使用metatable改变访问全局变量时的行为。

第一步是侦测到全局变量并不存在：

```lua
    setmetatable(_G, {
    	__newindex = function (_, n)
    		error("attempt to write to undeclared variable " .. n, 2)
        end,
    	__index = function (_, n)
    		error("attempt to read undeclared variable " .. n, 2)
    	end,
    })
```

此后，尝试访问一个不存在的全局变量将报错：

```lua
    > print(a)
    stdin:1: attempt to read undeclared variable a
```

但如何设置新的变量？一个办法是使用`rawset`，绕过元方法：

```lua
    function declare (name, initval)
    	rawset(_G, name, initval or false)
    end
```

`or`是为了全局总是获取到一个非`nil`的值。

更简单的方法是限制，给新全局变量赋值不能发生在函数内部，allowing free assignments in the outer level of a chunk。要检查赋值发生在哪，必须使用debug库。The call `debug.getinfo(2, "S")` returns a table whose field what tells whether the function that called the metamethod is a main chunk, a regular Lua function, or a C function. (We will see `debug.getinfo` in more detail in Chapter 24.) Using this function, we can rewrite the `__newindex` metamethod like this:

```lua
    __newindex = function (t, n, v)
        local w = debug.getinfo(2, "S").what
        if w ~= "main" and w ~= "C" then
        	error("attempt to write to undeclared variable " .. n, 2)
        end
        rawset(t, n, v)
    end
```

This new version also accepts assignments from C code, as this kind of code usually knows what it is doing.

To test whether a variable exists, we cannot simply compare it to nil because, if it is nil, the access will throw an error. Instead, we use rawget, which avoids the metamethod:

```lua
    if rawget(_G, var) == nil then
        -- 'var' is undeclared
        ...
    ends
```

As it is, our scheme does not allow global variables with nil values, as they would be automatically considered undeclared. But it is not difficult to correct this problem. All we need is an auxiliary table that keeps the names of declared variables. Whenever a metamethod is called, it checks in this table whether the variable is undeclared or not. The code can be like in Listing 14.1. Now even an assignment like x=nil is enough to declare a global variable.

The overhead for both solutions is negligible. With the first solution, the metamethods are never called during normal operation. In the second, they can be called, but only when the program accesses a variable holding a nil. The Lua distribution comes with a module `strict.lua` that implements a global-variable check that uses essentially the code we just reviewed. It is a good habit to use it when developing Lua code.

### 14.3 非全局环境

One of the problems with the environment is that it is global. Any modification you do on it affects all parts of your program. For instance, when you install a metatable to control global access, your whole program must follow the guidelines.

If you want to use a library that uses global variables without declaring them, you are in bad luck. In Lua, global variables do not need to be truly global. We can even say that Lua does not have global variables. That may sound strange at first, as we have been using global variables all along this text. Clearly, Lua goes to great lengths to give the programmer an illusion of global variables. Let us see how Lua builds this illusion.(Note that this mechanism was one of the parts of Lua that changed most from version 5.1 to 5.2. The following discussion refers to Lua 5.2 and very little of it applies to previous versions.)

Listing 14.1. Checking global-variable declaration:

```lua
    local declaredNames = {}
    setmetatable(_G, {
        __newindex = function (t, n, v)
            if not declaredNames[n] then
                local w = debug.getinfo(2, "S").what
                if w ~= "main" and w ~= "C" then
                    error("attempt to write to undeclared variable "..n, 2)
                end
                declaredNames[n] = true
            end
            rawset(t, n, v) -- do the actual set
        end,
    	__index = function (_, n)
        	if not declaredNames[n] then
        		error("attempt to read undeclared variable "..n, 2)
        	else
        		return nil
        	end
        end,
    })
```

Let us start with the concept of free names. A free name is a name that is
not bound to an explicit declaration, that is, it does not occur inside the scope of
a local variable (or a for variable or a parameter) with that name. For instance,
both var1 and var2 are free names in the following chunk:
	var1 = var2 + 3
Unlike what we said earlier, a free name does not refer to a global variable (at
least not in a direct way). Instead, the Lua compiler translates any free name
var to _ENV.var. So, the previous chunk is equivalent to this one:
	_ENV.var1 = _ENV.var2 + 3
But what is this new _ENV variable? It cannot be a global variable; otherwise,
we would be back to the original problem. Again, the compiler does the trick.
I already mentioned that Lua treats any chunk as an anonymous function.
Actually, Lua compiles our original chunk as the following code:
    local _ENV = <some value>
    return function (...)
    _ENV.var1 = _ENV.var2 + 3
    end
That is, Lua compiles any chunk in the presence of a predefined upvalue called
_ENV.
Usually, when we load a chunk, the load function initializes this predefined
upvalue with the global environment. So, our original chunk becomes equivalent
to this one:
    local _ENV = <the global environment>
    return function (...)
    _ENV.var1 = _ENV.var2 + 3
    end
The result of all these arrangements is that the var1 field of the global environment
gets the value of the var2 field plus 3.
At first sight, this may seem a rather convoluted way to manipulate global
variables. I will not argue that it is the simplest way, but it offers a flexibility
that is difficult to achieve with a simpler implementation.
Before we go on, let us summarize the handling of global variables in Lua 5.2:
 Lua compiles any chunk in the scope of an upvalue called _ENV.
 The compiler translates any free name var to _ENV.var.
 The load (or loadfile) function initializes the first upvalue of a chunk with
the global environment.
After all, it is not that complicated.
Some people get confused because they try to infer extra magic from these
rules. There is no extra magic. In particular, the first two rules are done entirely
by the compiler. Except for being predefined by the compiler, _ENV is a plain
regular variable. Outside compilation, the name _ENV has no special meaning at
all to Lua.3 Similarly, the translation from var to _ENV.var is a plain syntactic
translation, with no hidden meanings. In particular, after the translation, _ENV
will refer to whatever _ENV variable is visible at that point in the code, following
the standard visibility rules.

## 15. 模块与包

从5.1开始，Lua为模块和包定义了一组策略（包是模块的集合）。这些策略并未要求语言增加新的特性；使用的都是已有的特性：tables, functions, metatables, and environments。

从用户的视觉看，模块是通过`require`加载的代码（Lua代码或C代码），它创建并返回一个表。模块导出的所有内容，包括函数和常量，都定义在表中。这个表作为一个命名空间。

所有的标准库都是模块。例如，可以像这样使用数学库：

```lua
    local m = require "math"
    print(m.sin(3.14))
```

However, the stand-alone interpreter preloads all standard libraries with code equivalent to this:

```lua
    math = require "math"
    string = require "string"
    ...
```

模块是表，于是具有表的所有功能。例如，用户调用模块中的函数，有多种方法：

```lua
    local mod = require "mod"
    mod.foo()
```

模块可以用任意的本地名：

```lua
    local m = require "mod"
    m.foo()
```

In any way, remember that the module itself is loaded only once; it is up to the module to handle conflicting initializations.

### 15.1 require函数

The require function tries to keep to a minimum its assumptions about what a module is. For require, a module is just any code that defines some values (such as functions or tables containing functions). Typically, that code returns a table comprising the module functions. However, because this action is done by the module code, not by require, some modules may choose to return other values or even to have side effects.

要加载模块，质押调用require"modname"。第一步是检查表`package.loaded`，看模块是否已加载。如果已加载，直接返回。

If the module is not loaded yet, require searches for a Lua file with the module name. If it finds a Lua file, it loads it with `loadfile`. The result of that is a function that we call a loader. (The loader is a function that, when called, loads the module.)

If require cannot find a Lua file with the module name, it searches for a C library with the module name. If it finds a C library, it loads it with `package.loadlib` (which we discussed in Section 8.3), looking for a function called `luaopen_modname`. The loader in this case is the result of `loadlib`, that is, the function `luaopen_modname` represented as a Lua function.

No matter whether the module was found in a Lua file or a C library, `require` now has a loader for it. To finally load the module, require calls the loader with two arguments: the module name and the name of the file where it got the loader. (Most modules just ignore these arguments.) If the loader returns any value, require returns this value and stores it in the `package.loaded` table to return the same value in future calls for this same module. If the loader returns no value, require behaves as if the module returned true. Without this correction, a subsequent call to require would run the module again.

To force require into loading the same module twice, we simply erase the library entry from package.loaded: `package.loaded.<modname> = nil`. The next time the module is required, require will do all its work again.

#### 重命名模块

Usually, we use modules with their original names, but sometimes we must rename a module to avoid name clashes. A typical situation is when we need to load different versions of the same module, for instance for testing. Lua modules do not have their names fixed internally, so usually it is enough to rename the `.lua` file. However, we cannot edit a binary library to correct the name of its `luaopen_*` function. To allow for such renamings, require uses a small trick: if the module name contains a hyphen, require strips from the name its prefix up to the hyphen when creating the `luaopen_*` function name. For instance, if a module is named `a-b`, require expects its open function to be named `luaopen_b`, instead of `luaopen_a-b` (which would not be a valid C name anyway). So, if we need to use two modules named mod, we can rename one of them to `v1-mod`, for instance. When we call `m1=require"v1-mod"`, require will find the renamed file `v1-mod` and, inside this file, the function with the original name `luaopen_mod`.

#### 路径搜索

When searching for a Lua file, require uses a path that is a little different from typical paths. The typical path is a list of directories wherein to search for a given file. However, ANSI C (the abstract platform where Lua runs) does not have the concept of directories. Therefore, the path used by require is a list of templates, each of them specifying an alternative way to transform a module name (the argument to require) into a file name. More specifically, each template in the path is a file name containing optional question marks. For each template, require replaces the module name for each ‘?’ and checks whether there is a file with the resulting name; if not, it goes to the next template. The templates in a path are separated by semicolons. For instance, if the path is

	?;?.lua;c:\windows\?;/usr/local/lua/?/?.lua

then the call `require"sql"` will try to open the following Lua files:

    sql sql.lua
    c:\windows\sql
    /usr/local/lua/sql/sql.lua

The require function assumes only the semicolon (as the component separator) and the question mark; everything else, including directory separators and file extensions, is defined by the path itself.

The path that require uses to search for Lua files is always the current value of variable `package.path`. When Lua starts, it initializes this variable with the value of the environment variable `LUA_PATH_5_2`. If this environment variable is not defined, Lua tries the environment variable `LUA_PATH`. If both are not defined, Lua uses a compiled-defined default path.(In Lua 5.2, the command-line option -E prevents the use of those environment variables and forces the default.) When using the value of an environment variable, Lua substitutes the default path for any substring “;;”.

For instance, if you set `LUA_PATH_5_2` to “`mydir/?.lua;;`”, the final path will be the template “`mydir/?.lua`” followed by the default path.

The path used to search for a C library works exactly in the same way, but its value comes from variable `package.cpath` (instead of `package.path`). Similarly, this variable gets its initial value from the environment variables `LUA_CPATH_5_2` or `LUA_CPATH`. A typical value for this path in UNIX is like this:

	./?.so;/usr/local/lib/lua/5.2/?.so

Note that the path defines the file extension. The previous example uses `.so` for all templates; in Windows, a typical path would be more like this one:

	.\?.dll;C:\Program Files\Lua502\dll\?.dll

Function `package.searchpath` encodes all those rules for searching libraries. It receives a module name and a path, and looks for a file following the rules described here. It returns either the name of the first file that exists or nil plus an error message describing all files it unsuccessfully tried to open, as in the next example:

    > path = ".\\?.dll;C:\\Program Files\\Lua502\\dll\\?.dll"
    > print(package.searchpath("X", path))
    nil
    no file '.\X.dll'
    no file 'C:\Program Files\Lua502\dll\X.dll'

#### Searchers

In reality, require is a little more complex than we have described. The search for a Lua file and the search for a C library are just two instances of a more general concept of searchers. A searcher is simply a function that receives the module name and returns either a loader for that module or nil if it cannot find one.

The array package.searchers lists the searchers that require uses. When looking for a module, require calls each searcher in the list passing the module name, until one of them finds a loader for the module. If the list ends without a positive response, require raises an error.

The use of a list to drive the search for a module allows great flexibility to require. For instance, if you want to store modules compressed in zip files, you only need to provide a proper searcher function for that and add it to the list. However, more often than not, programs do not change the default contents of package.searchers. In this default configuration, the searcher for Lua files and the searcher for C libraries that we described earlier are respectively the second and the third elements in the list. Before them, there is the preload searcher.

The preload searcher allows the definition of an arbitrary function to load a module. It uses a table, called package.preload, to map module names to loader functions. When searching for a module name, this searcher simply looks for the given name in the table. If it finds a function there, it returns this function as the module loader. Otherwise, it returns nil. This searcher provides a generic method to handle some non-conventional situations. For instance, a C library statically linked to Lua can register its luaopen_ function into the preload table,
so that it will be called only when (and if) the user requires that module. In this way, the program does not waste time opening the module if it is not used.

The default content of package.searchers includes a fourth function that is relevant only for submodules. We will discuss it at Section 15.4.







