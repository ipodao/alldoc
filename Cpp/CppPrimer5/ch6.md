[toc]

## 6. 函数

### 6.1 函数基础

为了与C兼容，当函数形参列表为空时，可以用`void`：

```cpp
    void f1(){/* ... */ }  // implicit void parameter list
    void f2(void){ /* ... */ } // explicit void parameter list
```

返回类型可以是`void`。**返回类型不可以是数组类型（§ 3.5）或函数类型**。但可以返回到数组或函数的指针。

#### 6.1.1. Local Objects

C++中， **名字有作用域(§ 2.2.4)，对象有生命周期**。

- 名字的作用域指，在哪块程序中名字是可见的
- 对象的生命周期指运行中对象存在的时间

定义在函数体内的参数和变量称为局部（local）变量。局部变量隐藏外部作用域的同名变量。

##### Automatic Objects

与普通局部变量相对应的对象，are created when the function’s control path passes through the variable’s definition. They are destroyed when control passes through the end of the block in which the variable is defined. 仅在某个块执行中存在的对象称为自动对象。After execution exits a block, the values of the automatic objects created in that block are undefined.

参数是自动对象。Storage for the parameters is allocated when the function begins. 参数在函数体的作用域内定义。

对应局部变量的自动对象，如果定义中不含初始化器，则它们将被**默认初始化**(§ 2.2.1)。即未初始化的内建类型的局部变量的值是未定的。

##### Local static Objects

It can be useful to have a local variable whose lifetime continues across calls to the function. We obtain such objects by defining a local variable as static. Each local static object is initialized before the first time execution passes through the object’s definition. Local statics are not destroyed when a function ends; 它们在程序终止时被销毁。

例如，下面的函数能记录被调用的次数：

```cpp
    size_t count_calls()
    {
        static size_t ctr = 0;  // value will persist across calls
        return ++ctr;
    }
    int main()
    {
        for(size_t i = 0; i != 10; ++i)
            cout << count_calls() << endl;
        return 0;
    }
```

未不显式初始化局部静态变量，它们将被值初始化(§ 3.3.1)。即内建类型的局部静态变量会被初始化为0。

#### 6.1.2. 函数声明

与其他名字一样，函数名要先声明再使用。与变量名一样，函数名只能被定义一次，却能被声明多次。只要我们不使用，函数可以只声明不定义（一个例外见§ 15.3）。

函数声明类似于函数定义，只是没有Body。函数声明也称为函数原型。

函数声明的参数可以没有参数名。但列出参数名有助于帮助理解语义：

```cpp
    void print(vector<int>::const_iterator beg,
    	vector<int>::const_iterator end);
```

之前提出，变量应该在头中声明在源文件中定义。出于相同目的，函数应该在头中声明在源文件中定义。源文件应该包含函数声明的头，这样编译器能检查声明与定义是否一致。

#### 6.1.3. Separate Compilation

分离（Separate）编译允许我们将程序分成多个文件。每个文件可以被独立编译。例子，假设函数`fact`定义在文件`fact.cc`，它的声明在头文件`Chapter6.h`。`main`函数在第二个文件中`factMain.cc`。要产生可执行文件，需要所有文件：

```
$ CC factMain.cc fact.cc  # generates factMain.exe or a.out
$ CC factMain.cc fact.cc -o main # generates main or main.exe
```

其中CC是编译器。

改变某个文件后，期望只重新编译改变的文件。Most compilers provide a way to separately compile each file. 此过程常产生一个`.obj`(Windows)或`.o`(UNIX)文件。

The compiler lets us *link* object files together to form an executable. On the system we use, we would separately compile our program as follows:

```
$ CC -c factMain.cc # generates factMain.o
$ CC -c fact.cc  # generates fact.o
$ CC factMain.o fact.o  # generates factMain.exe or a.out
$ CC factMain.o fact.o -o main # generates main or main.exe
```

### 6.2 参数传递

> 参数初始化与变量初始化的方式相同。

如果形参是引用(§ 2.3.1)则形参绑定到实参（按引用传递）。否则实参拷贝到形参，此时形参与实参是独立的对象（按值传递）。

#### 6.2.1. 按值传递

> 熟悉C的程序员常通过指针访问函数外的对象，**但C++程序员一般使用引用**。

如果参数是指针，也是按值传递——指针值被拷贝。于是形参和实现指向同一个对象。

#### 6.2.2. 按引用传递

最佳实践：若函数内不会改变引用参数｛｛不会修改参数内容｝｝，引用应该被置为`const`（指向常量的引用）。

```cpp
	// compare the length of two strings
    bool isShorter(const string &s1, const string &s2)
    {
        return s1.size() < s2.size();
    }
```

#### 6.2.3. const形参与实参

回忆关于顶层常量的讨论（§ 2.4.3），顶级常量指对象本身是常量：

```cpp
    const int ci = 42; // 不能改变ci；常量是顶级的
    int i = ci; // 可以：拷贝ci时，顶级常量会被忽略
    int * const p = &i; // p是常量
    *p = 0; // 但p指向的不是常量，因此可以赋值
```

当我们拷贝一个实参去初始化一个形参时，顶层常量会被忽略。可以将一个常量的实参或非常量的实参传给一个常量的形参：

```cpp
	void fcn(const int i) { /* fcn不能修改i*/ }
```

可以向`fcn`传`const int`或普通`int`。

因为顶层常量可以被忽略，因此下面两个声明的差别不够大，因此不允许这种重载：

```cpp
	void fcn(const int i) { /* fcn can read but not write to i*/ }
	void fcn(int i) { /* . . .*/ } // error: redefines fcn(int)
```

##### 指针、引用与const

我们可以用一个非常量的对象初始化一个低级常量，但反之不行。

```cpp
    int i = 42;
    const int *cp = &i; // 可以：但cp不能修改i (§ 2.4.2)
    const int &r = i; // 可以：但r不能修改i (§ 2.4.1)
    const int &r2 = 42; // ok: (§ 2.4.1)
    int *p = cp; // 错误：cp指向常量，但p指向变量(§ 2.4.2)
    int &r3 = r; // 错误：r指向常量，但r3指向变量(§ 2.4.1)
    int &r4 = 42; // 错误：不能用字面量初始化一个非常量的引用(§ 2.3.1)
```

同样的规则适用于参数传递：

```cpp
	void reset(int *ip);
	void reset(int &i);
	string::size_type find_char(const string &s, char c,
    	string::size_type &occurs)
    int i= 0;
    const int ci = i;
    string::size_type ctr = 0;
    reset(&i); // calls the version of reset that has an int* parameter
    reset(&ci); // 错误，不能用指向常量int的指针初始化指向非常量int的指针
    reset(i); // 调用(int&)
    reset(ci); // 错误：不能把常量引用赋给非常量引用
    reset(42); // 错误：不能绑定普通引用到一个字面量
    reset(ctr); // error: types don't match; ctr has an unsigned type
    // ok: find_char's first parameter is a reference to const
    find_char("Hello World!", 'o', ctr);
```

引用版本的`reset`方法只能接收`int`类型的对象。不能传字面量，一个求值结果为`int`的表达式，需要转换的对象，或`const int`对象。Similarly, we may pass only an `int*` to the pointer version of reset(§ 6.2.1). 但我们可以将一个字面量传给`find_char`(§ 6.2.2)。因为形参是到常量的引用。

##### 尽量使用到常量的引用

一个常见的错误是，将函数定义成非常量引用，但其实函数不会修改此参数。而且限制了实参可以使用的类型。

例如如果把 `find_char`改成错误的定义：

```cpp
    // bad design: the first parameter should be a const string&
    string::size_type find_char(string &s, char c,
        string::size_type&occurs);
```

则`find_char("Hello World",'o', ctr);`将不再能通过编译。

更坏的是，我们无法在定义正确的函数内使用这个版本的`find_char`函数：

```cpp
    bool is_sentence(const string &s)
    {
        // if there's a single period at the end of s, then s is a sentence
        string::size_type ctr = 0;
        return find_char(s, '.', ctr) == s.size() - 1 && ctr == 1;
    }
```

#### 6.2.4. 数组参数

数组两个特性：不能拷贝数组(§ 3.5.1)，使用数组时，常被转换为指针(§ 3.5.3)。因为不能拷贝，因此不能按值传递。因为数组常被转换为指针，将数组传给函数时，实际传递的是数组第一个元素的指针。

```cpp
    void print(const int*);
    void print(const int[]); // shows the intent that the function takes an array
    void print(const int[10]); // dimension for documentation purposes (at best)
```

这三个声明是等价的。每个函数实际接受的参数都是`const int*`。因此实参只要是`const int*`：

```cpp
    int i= 0, j[2] = {0, 1};
    print(&i); // ok: &i is int*
    print(j);  // ok: j is converted to an int* that points to j[0]
```

> 当函数不需要修改数组元素时，数组参数应该是指向常量的指针(§ 2.4.2)。

如果实参是数组，实参会被自动转换为指向第一个元素的指针，数组的大小不被关心。由于函数不知道数组大小，因此一般需要附加参数。有三种策略：

##### 数组结尾放一个标记元素

例如，C风格的字符串，最后一个是null字符。

```cpp
    void print(const char *cp)
    {
        if(cp)  // 如果cp不是空指针
            while(*cp)  // 不是空字符
                cout << *cp++;
    }
```

##### 使用标准库的约定

传递数组首指针和数组后指针。

```cpp
    void print(const int *beg, const int *end)
    {
        // print every element starting at beg up to but not including end
        while(beg != end)
            cout<< *beg++ << endl; // print the current element
        // and advance the pointer
    }

    int j[2]= {0, 1};
	print(begin(j), end(j)); // begin and end functions, see § 3.5.3
```

##### 显式传递size参数

```cpp
    void print(const int ia[], size_t size)
    {
        for(size_t i = 0; i != size; ++i) {
            cout << ia[i] << endl;
        }
    }

	int j[] = { 0, 1 };  // int array of size 2
	print(j, end(j) - begin(j));
```

##### 数组引用参数

形参可以是到数组的引用。

```cpp
	// 注意the dimension is part of the type
    void print(int (&arr)[10])
    {
        for(auto elem : arr)
            cout<< elem << endl;
    }
```

注意括号。

因为数组的大小是其类型的一部分。it is safe to rely on the dimension in the body of the function. 但这样限制了函数的用途。实参数组只能是10个元素。We’ll see in § 16.1.1 how we might write this function in a way that would allow us to pass a reference parameter to an array of any size.

##### 多维数组做参数

与任何数组一样，多维数组传入的实际是指向第一个元素的指针(§ 3.6)。The size of the second (and any subsequent) dimension is part of the element type and must be specified:

```cpp
    // matrix points to the first element in an array whose elements are arrays of ten ints
    void print(int (*matrix)[10], int rowSize) { /* . . .*/ }
```

We can also define our function using array syntax. As usual, the compiler ignores the first dimension, so it is best not to include it:

```cpp
    // equivalent definition
    void print(int matrix[][10], int rowSize) { /* . . .*/ }
```

#### （未）6.2.5. main: Handling Command-Line Options

#### 6.2.6 变长参数

新标准提供了两种解决办法：如果参数类型相同，可以传入一个库类型`initializer_list`。如果参数类型不同，可以编写一种特殊的函数，称为variadic template, which we’ll cover in § 16.4。

C++还有一种特殊的参数类型**ellipsis**，可以用于传递数量可变的实参。但这种功能一般只需要用在需要与 C 函数接口的程序。

##### `initializer_list`

如果参数类型相同，可以用`initializer_list`传递数量不定的参数。`initializer_list`是库类型，表示一个数组。该类型定义在头文件`initializer_list`。

`initializer_list`支持的操作：

- `initializer_list<T> lst;`：默认初始化，列表为空
- `initializer_list<T> lst{a, b, c...};`：从初始值列表中拷贝。列表中的元素是常量。
- `lst2(lst)`，`lst2 = lst`：拷贝或赋值一个`initializer_list`不拷贝元素，两个列表共享相同的元素。
- `lst.size()`：列表中的元素数
- `lst.begin()`, `lst.end()`：

`initializer_list`是一个模板类型。

```cpp
	initializer_list<string> ls; // initializer_list of strings
	initializer_list<int> li; // initializer_list of ints
```

与`vector`不同的是，`initializer_list`中的值总是常量；没有办法改变`initializer_list`中的元素的值。We can write our function to produce error messages from a varying number of arguments as follows:

```cpp
    void error_msg(initializer_list<string> il)
    {
        for(auto beg = il.begin(); beg != il.end(); ++beg)
        	cout << *beg << " ";
        cout << endl;
    }
```

`initializer_list`的实参放在一个大括号中：

```cpp
    // expected, actual are strings
    if (expected != actual)
    	error_msg({"functionX", expected, actual});
    else
    	error_msg({"functionX", "okay"});
```

有`initializer_list`参数的函数可以同时有其他参数。

```cpp
    void error_msg(ErrCodee, initializer_list<string> il)
    {
        cout << e.msg() << ": ";
        for(const auto &elem : il)
        	cout << elem << " " ;
        cout << endl;
    }
```

Because `initializer_list` has `begin` and `end` members, we can use a range for(§ 5.4.3) to process the elements.

调用：

```cpp
    if (expected!= actual)
    	error_msg(ErrCode(42), {"functionX", expected, actual});
    else
    	error_msg(ErrCode(0), {"functionX", "okay"});
```

##### Ellipsis参数

Ellipsis parameters are in C++ to allow programs to interface to C code that uses a C library facility named `varargs`. In particular, objects of most class types are not copied properly when passed to an ellipsis parameter. 除此之外，ellipsis参数不应被用于其他目的。Your C compiler documentation will describe how to use varargs.

An ellipsis parameter may appear only as the **last** element in a parameter list and may take either of two forms:

```cpp
    void foo(parm_list,...);
    void foo(...);
```

The first form specifies the type(s) for some of foo’s parameters. Arguments that correspond to the specified parameters are type checked as usual. No type checking is done for the arguments that correspond to the ellipsis parameter. In this first form, the comma following the parameter declarations is optional.

### 6.3. 返回类型

#### 6.3.2. 有返回值的函数

##### 值是如何被返回的

返回值与初始化变量和参数的方式相同。返回值被用于初始化一个临时变量，这个临时变量作为函数调用的结果。

```cpp
    // return the plural version of word if ctr is greater than 1
    string make_plural(size_t ctr, const string &word,
        const string &ending)
    {
        return(ctr > 1) ? word + ending : word;
    }
```

返回的结果，或者是`word`的一个拷贝，或者是`word`和`ending`相加后的临时字符串。

如果函数返回引用，则此引用只是它引用的对象的另一个名字。

##### 永远不要返回到局部对象的引用或指针

返回返回后，其存储空间会被释放。此时，到局部对象的引用不再有效：

```cpp
    // disaster: this function returns a reference to a local object
    const string &manip()
    {
        string ret;
        // transform ret in some way
        if (!ret.empty())
        	return ret; // WRONG: returning a reference to a local object!
        else
        	return "Empty"; // WRONG: "Empty" is a local temporary string
    }
```

两个`return`返回的值都是未定义的。

同样的，返回到局部对象的指针也是错误的。

##### 返回引用返回的是左值

函数调用是左值还是右值取决于函数返回值类型。返回**引用**的函数调用是左值，否则是右值。如果引用不是常量，则可以给函数调用赋值：

```cpp
    char &get_val(string &str, string::size_type ix)
    {
    	return str[ix]; // get_val assumes the given index is valid
    }
    int main()
    {
        strings("a value");
        cout << s << endl;  // prints a value
        get_val(s, 0) = 'A'; // changes s[0] to A
        cout << s << endl;  // prints A value
        return 0;
    }
```

If the return type is a reference to const, then (as usual) we may not assign to the result of the call:

```cpp
	shorterString("hi", "bye") = "X"; // error: return value is const
```

##### 列表初始化返回值

新标准允许函数返回花括号包围的一组值。与其他返回值一样，列表用于初始化用作返回值的临时量。如果列表是空的，则临时量将被值初始化(§ 3.3.1)。其他情况下，返回的值取决于函数的返回值。

As an example, recall the `error_msg` function from § 6.2.6. That function took a varying number of string arguments and printed an error message composed from the given strings. Rather than calling `error_msg`, in this function we’ll return a vector that holds the error-message strings:

```cpp
    vector<string> process()
    {
        // . . .
        // expected and actual are strings
        if(expected.empty())
        	return {};  // return an empty vector
        else if (expected == actual)
        	return {"functionX", "okay"}; // return list-initialized vector
        else
            return {"functionX", expected, actual};
    }
```

如果函数返回一个内建类型，花括号中至多有一个值，and that value must not require a narrowing conversion (§ 2.2.1). If the function returns a class type, then the class itself defines how the intiailizers are used (§ 3.3.1).

##### Return from main

There is one exception to the rule that a function with a return type other than void
must return a value: The mainfunction is allowed to terminate without a return. If
control reaches the end of mainand there is no return, then the compiler implicitly
inserts a return of 0.
As we saw in § 1.1(p. 2), the value returned from mainis treated as a status
indicator. A zero return indicates success; most other values indicate failure. A nonzero
value has a machine-dependent meaning. To make return values machine
independent, the cstdlibheader defines two preprocessor variables (§ 2.3.2, p. 54)
that we can use to indicate success or failure:
Click hereto view code image
int main()
{
if(some_failure)
returnEXIT_FAILURE;  // defined in cstdlib
else
returnEXIT_SUCCESS;  // defined in cstdlib
}
Because these are preprocessor variables, we must not precede them with std::, nor
may we mention them in usingdeclarations.

#### 6.3.3. 返回到数组的指针

因为无法拷贝数组，因此函数无法返回数组。但可以返回到数组的指针或引用(§ 3.5.1)。返回数组指针或引用的函数的定义很难看。需要一些方式简化。最直接的方式是使用类别别名：

```cpp
    typedef int arrT[10]; // arrT是别名
    using arrtT = int[10]; // 等价定义arrT
    arrT* func(int i); // func returns a pointer to an array of five ints
```

##### 返回到数组的指针

不使用别名时，注意括号：

```cpp
    int arr[10];  // arr is an array of ten ints
    int *p1[10];  // p1是数组，10个指针
    int (*p2)[10] = &arr; // p2是指针，指向具有10个元素的数组
```

此时：

```cpp
	Type(*function(parameter_list))[dimension]
```
As a concrete example, the following declares `func` without using a type alias:

```cpp
	int (*func(int i))[10];
```

##### 返回类型放后

新标准允许返回类型放后面。任何函数都可以这样做。特别适于返回类型复杂的函数。参数列表后加`->`，接着是返回类型。同时原来返回类型的位置放`auto`：

```cpp
	// fcn takes an int argument and returns a pointer to an array of ten ints
	auto func(int i) -> int(*)[10];
```

##### 使用`decltype`

```cpp
    int odd[] = {1,3,5,7,9};
    int even[] = {0,2,4,6,8};
    // returns a pointer to an array of five int elements
    decltype(odd) *arrPtr(int i)
    {
    	return (i % 2) ? &odd : &even; // returns a pointer to the array
    }
```

`arrPtr`返回的指针指向具有5个元素的数组。The only tricky part is that we must remember that decltype does not automatically convert an array to its corresponding pointer type. The type returned by decltypeis an array type, to which we must add a `*` to indicate that `arrPtr` returns a pointer.

### 6.4. 重载函数

同一作用域、同名但参数列表不同的函数重载。例子：

```cpp
	void print(constchar *cp);
	void print(const int *beg, const int *end);
	void print(const int ia[], size_t size);
```

> The main function may not be overloaded.

##### 定义重载函数

We can call `lookup` passing a value of any of several types:

```cpp
Record lookup(const Account&);  // find by Account
Record lookup(const Phone&);  // find by Phone
Record lookup(const Name&);  // find by Name
Account acct;
Phone phone;
Record r1 = lookup(acct);  // call version that takes an Account
Record r2 = lookup(phone); // call version that takes a Phone
```

If the parameter lists of two functions match but the return types differ, then the second declaration is an error:

```cpp
    Record lookup(constAccount&);
    bool lookup(const Account&);  // 错误
```

##### 两个参数类型是否不同

看起来不同的两个参数列表其实是相同的：

```cpp
    // each pair declares the same function
    Record lookup(const Account &acct);
    Record lookup(const Account&); // parameter names are ignored
    typedef Phone Telno;
    Record lookup(const Phone&);
    Record lookup(const Telno&); // Telno and Phone are the same type
```

##### 重载与常量参数

As we saw in § 6.2.3, top-level const has no effect on the objects that can be passed to the function. 带顶级常量的参数与不带的参数等价：

```cpp
    Record lookup(Phone);
    Record lookup(const Phone); // redeclares Record lookup(Phone)
    Record lookup(Phone*);
    Record lookup(Phone* const); // redeclares Record lookup(Phone*)
```

第一个和第二个声明等价。

但使用低级常量的两个参数不等价：

```cpp
    // declarations for four independent, overloaded functions
    Record lookup(Account &); // 引用指向的是非常量
    Record lookup(const Account&); // 引用指向的是常量
    Record lookup(Account *); // new function, takes a pointer to Account
    Record lookup(const Account *); // new function, takes a pointer to const
```

Because there is no conversion (§ 4.11.2) from const, we can pass a const object (or a pointer to const) only to the version with a const parameter. Because there is a conversion to const, we can call either function on a nonconst object or a pointer to nonconst. However, as we’ll see in §6.6.1, the compiler will prefer the nonconst versions when we pass a nonconst object or pointer to nonconst.

##### `const_cast`与重载

In § 4.11.3 we noted that `const_cast`s are most useful in the context of overloaded functions. As one example, recall our `shorterString` function from §6.3.2:

```cpp
    // return a reference to the shorter of two strings
    const string &shorterString(const string &s1, const string &s2)
    {
    	return s1.size() <= s2.size() ? s1 : s2;
    }
```

函数返回到常量的引用。调用这个函数时，可以传入两个非常量的字符串，但返回的却是一个常量的字符串。但我们期望有一个`shorterString`版本，传入的字符串不是常量时，返回的也不是常量。可以通过`const_cast`实现：

```cpp
    string &shorterString(string &s1, string &s2)
    {
        auto &r = shorterString(const_cast<const string&>(s1),
        	const_cast<const string&>(s2));
        return const_cast<string&>(r);
    }
```

#### 6.4.1. 重载与作用域

> Ordinarily, it is a bad idea to declare a function locally. However, to explain how scope interacts with overloading, we will violate this practice and use local function declarations.

重载与作用域没有特殊关系：与平时一样，如果我们在内层作用域声明一个名字，它将隐藏外层作用域的名字。**名字不会跨作用域重载**：

```cpp
    string read();
    void print(const string &);
    void print(double);  // 重载
    void fooBar(int ival)
    {
        bool read = false; // 新作用域：隐藏外部的read
        strings = read(); // 错误：read不再是函数
        // 不好的做法：不要在内层作用域声明函数
        void print(int); // 将隐藏外面的 所有 重载版本的print
        print("Value:"); // 错误：隐藏了print(const string &)
        print(ival); // 正确：print(int)是可见的
        print(3.14); // 正确：调用的是print(int)；但print(double)被隐藏了
    }
```

调用`print`时，当编译器在内层作用域找到声明时，便会忽略外层的名字。

> In C++, name lookup happens before type checking.

第一次调用传入字符串字面量，但作用域中唯一的`print`声明只接收`int`参数。字符串无法被转换为`int`，因此出错。`print(const string&)`已被隐藏，不考虑。

### 6.5. 其他特性

本节讲默认实参，内联和`constexpr`函数。

#### 6.5.1. 默认实参

```cpp
    typedef string::size_type sz;
    string screen(sz ht = 24, sz wid = 80, char backgrnd = ' ');
```

这里，为每个形参都提供给了一个默认值。若一个参数带默认值，则它后面的参数都要带。


##### 调用带默认实参的函数

下面的调用都是游戏的：
```cpp
    string window;
    window = screen(); // equivalent to screen(24,80,' ')
    window = screen(66);// equivalent to screen(66,80,' ')
    window = screen(66, 256); // screen(66,256,' ')
    window = screen(66, 256, '#'); // screen(66,256,'#')
```

调用时，只能省略最右端的实参。

##### 默认实参声明

尽管最常见的做法是在头文件中只声明函数一次。但声明多次也是合法的。但在一个作用域中，参数只能被指定一次默认值。后续的声明，可以向之前没有指定过默认值的参数指定默认值。As usual, defaults can be specified only if all parameters to the right already have defaults. For example, given

```cpp
// no default for the height or width parameters
string screen(sz, sz, char = ' ');
```

不能改变之前声明的默认值：

```cpp
string screen(sz, sz, char = '*'); // error: redeclaration
```

但可以增加默认值：

```cpp
string screen(sz= 24, sz = 80, char);
```

> 最佳实践：默认实参一般应该在头文件中的函数声明中指定

##### 默认实参初始化

局部变量不能用于默认实参。除此之外，任何表达式都可以，只要类型能兼容：

```cpp
    // wd, def, ht必须位于函数之外（不是局部变量）
    sz wd = 80;
    char def = ' ';
    sz ht();
    string screen(sz = ht(), sz = wd, char = def);
    string window = screen(); // calls screen(ht(), 80, ' ')
```

Names used as default arguments are resolved in the scope of the function declaration. 这些名字表示的值在调用时求值：

```cpp
    void f2()
    {
    	def = '*'; // 改变默认实参的值
    	sz wd = 100; // 隐藏了外层的定义
    	window= screen(); // calls screen(ht(), 80, '*')
    }
```

调用`screen`时，`def`参数将使用更新后的值。但注意，重定义的内层的局部变量`wd`与默认实参无关，因此不影响调用。

#### 6.5.2. 内联与`constexpr`函数

##### inline函数避免函数调用的开销

定义内联函数，在函数返回类型前加`inline`关键字：

```cpp
    inline const string & shorterString(const string &s1, const string &s2)
    {
        return s1.size() <= s2.size() ? s1 : s2;
    }
```

> 内联只是给编译器的请求，编译器可以选择忽略

Many compilers will not inline a recursive function. A 75-line function will almost surely not be expanded inline.

##### `constexpr`函数

`constexpr`函数是可以被用于常量表达式(§ 2.4.4)的函数。`constexpr`的限制是：The return type and the type of each parameter in a must be a literal type (§ 2.4.4), and the function body must contain exactly one return
statement:

```cpp
	constexpr int new_sz() { return 42; }
    constexpr int foo = new_sz(); // ok: foo is a constant expression
```

Here we defined `new_sz` as a `constexpr` that takes no arguments. The compiler can verify—at compile time—that a call to new_szreturns a constant expression, so we can use `new_sz` to initialize our `constexpr` variable, foo.

When it can do so, the compiler will replace a call to a `constexpr` function with its resulting value. In order to be able to expand the function immediately, `constexpr` functions are implicitly inline.

A `constexpr` function body may contain other statements so long as those statements generate no actions at run time. For example, a `constexpr` function may contain null statements, type aliases (§ 2.5.1), and using declarations.

A `constexpr` functionis permitted to return a value that is not a constant:

```cpp
// scale(arg) isa constant expression if arg is a constant expression
constexpr size_t scale(size_t cnt) { return new_sz() * cnt; }
```

The scale function will return a constant expression if its argument is a constant expression but not otherwise:
Click hereto view code image
int arr[scale(2)];// ok: scale(2) is a constant expression
int i = 2;  // i is not a constant expression
int a2[scale(i)];  // error: scale(i) is not a constant expression

When we pass a constant expression—such as the literal 2—then the return is a constant expression. In this case, the compiler will replace the call to scale with the resulting value.

If we call scalewith an expression that is not a constant expression—such as on the intobject i—then the return is not a constant expression. If we use scalein a context that requires a constant expression, the compiler checks that the result is a constant expression. If it is not, the compiler will produce an error message.

> A constexpr functionis not required to return a constant expression.

##### 将内联与`constexpr`函数放入头文件

Unlike other functions, inline and `constexpr` functions may be defined multiple times in the program. After all, the compiler needs the definition, not just the declaration, in order to expand the code. However, all of the definitions of a given inline or constexprmust match exactly. As a result, inlineand constexpr functions normally are defined in headers.

#### （未）6.5.3. 帮助调试

### 6.6 函数匹配

若重载的函数参数数量相同，且其中的有些参数的类型可以转换，则匹配并不是那么容易。例如：

void f();
voidf(int);
void f(int, int);
void f(double, double = 3.14);
f(5.6); // calls void f(double, double)

##### 找出候选和可行（Viable）函数

第一步找出候选函数。候选函数要求名字匹配，且其声明在调用时可见。在上面的例子中，有四个候选函数`f`。

第二步，匹配参数。参数匹配的函数是可行（viable）函数。要可行，函数的参数数量首先要匹配。其次，实参形参类型能够转换。根据参数数量可以去掉两个候选函数。

在这个例子中，剩下的两个函数都是可行的：

- `f(int)`可行，因为可以将`double`转换为`int`
- `f(double, double)`可行，因为第二个参数有默认值，第一个参数精确匹配。

> If there are no viable functions, the compiler will complain that there is no matching function.

##### 寻找最佳匹配

第三步，从可行（viable）函数中找最佳匹配。This process looks at each argument in the call and selects the viable function (or functions) for which the corresponding parameter best matches the argument. We’ll explain the details of “best” in the next section, but the idea is that the closer the types of the argument and parameter are to each other, the better the match.

In our case, there is only one (explicit) argument in the call. That argument has type `double`. To call `f(int)`, the argument would have to be converted from double to int. The other viable function, `f(double, double)`, is an exact match for this argument. An exact match is better than a match that requires a conversion.

Therefore, the compiler will resolve the call `f(5.6)` as a call to the function that has two double parameters. The compiler will add the default argument for the second, missing argument.

##### 多个参数的函数匹配

若实参有多个，情况将更加复杂。例如：
```cpp
f(42, 2.56);
```

The set of viable functions is selected in the same way as when there is only one parameter. The compiler selects those functions that have the required number of parameters and for which the argument types match the parameter types. In this case, the viable functions are `f(int, int)` and `f(double, double)`. The compiler then determines, argument by argument, which function is (or functions are) the best match. There is an overall best match if there is one and only one function for which

- The match for each argument is no worse than the match required by any other viable function
- There is at least one argument for which the match is better than the match provided by any other viable function
- If after looking at each argument there is no single function that is preferable, then the call is in error. The compiler will complain that the call is ambiguous.

In this call, when we look only at the first argument, we find that the function `f(int, int)` is an exact match. To match the second function, the intargument 42 must be converted to double. A match through a built-in conversion is “less good” than one that is exact. Considering only the first argument, `f(int, int)` is a better match than `f(double, double)`.

When we look at the second argument, `f(double, double)` is an exact match to the argument 2.56. Calling `f(int, int)` would require that 2.56 be converted from doubleto int. When we consider only the second parameter, the function `f(double, double)` is a better match.

The compiler will reject this call because it is ambiguous: Each viable function is a better match than the other on one of the arguments to the call. It might be tempting to force a match by explicitly casting (§ 4.11.3) one of our arguments.

However, in well-designed systems, argument casts should not be necessary.

> **Best Practices** Casts should not be needed to call an overloaded function. The need for a cast suggests that the parameter sets are designed poorly.

#### （未）6.6.1. 实参类型转换

In order to determine the best match, the compiler ranks the conversions that could be used to convert each argument to the type of its corresponding parameter.

Conversions are ranked as follows:

1. An exact match. An exact match happens when:
	- The argument and parameter types are identical.
    - The argument is converted from an array or function type to the corresponding pointer type. (§ 6.7 covers function pointers.)
    - A top-level constis added to or discarded from the argument.
2. Match through a const conversion (§ 4.11.2).
3. Match through a promotion (§ 4.11.1).
4. Match through an arithmetic (§ 4.11.1) or pointer conversion (§ 4.11.2).
5. Match through a class-type conversion. (§ 14.9 covers these conversions.)

### 6.7. 指向函数的指针

与其他指针一样，函数指针指向一个特定的类型。函数的类型由返回值类型和参数类型决定。函数名不是函数类型的一部分。例如：

```cpp
    // compares lengths of two strings
    bool lengthCompare(const string &, const string &);
```

的类型是`bool(const string&, const string&)`。声明函数指针时，指针名放在原来函数名的地方：

```cpp
    // pf是指针
    bool (*pf)(const string &, const string &); // 为初始化指针
```

> 注意，`*pf`两端的括号是必要的。若省略，then we declare pfas a function that returns a pointer to bool

##### 使用函数指针

当将函数名用作值时，函数会被自动转换为指针。例如把`lengthCompare`赋给`pf`：

```cpp
    pf = lengthCompare; // pf现在指向一个名为lengthCompare的函数
    pf = &lengthCompare; // 等价；取值是可选的
```

可以使用函数指针调用函数。不需要解引用指针：

```cpp
    bool b1 = pf("hello", "goodbye");  // calls lengthCompare
    bool b2 = (*pf)("hello", "goodbye"); // equivalent call
    bool b3 = lengthCompare("hello", "goodbye"); // equivalent call
```

There is no conversion between pointers to one function type and pointers to another function type. However, as usual, we can assign `nullptr`(§ 2.3.2, p. 53) or a **zero**-valued integer constant expression to a function pointer to indicate that the pointer does not point to any function:

```cpp
    string::size_type sumLength(const string&, const string&);
    bool cstringCompare(const char*, const char*);
    pf = 0; // ok: pf points to no function
    pf = sumLength;  // error: return type differs
    pf = cstringCompare; // error: parameter types differ
    pf = lengthCompare;  // ok: function and pointer types match exactly
```

##### 指向重载函数

```cpp
    void ff(int*);
    void ff(unsigned int);
    void (*pf1)(unsigned int) = ff;  // pf1 points to ff(unsigned)
```

the compiler uses the type of the pointer to determine which overloaded function to use. The type of the pointer must match one of the overloaded functions exactly:

```cpp
    void (*pf2)(int)= ff; // error: no ff with a matching parameter list
    double (*pf3)(int*) = ff; // error: return type of ff and pf3 don't match
```

##### 函数指针参数

与数组一样(§ 6.2.4)，函数参数不能是一个函数，但可以是函数指针。函数参数看起来是一个函数类型，但实际会被当成指针：

```cpp
// 第三个参数是一个函数类型，会被当作函数指针
void useBigger(const string &s1, const string &s2,
	bool pf(const string &, const string &));
// 等价声明，显式指明这是个指针
void useBigger(const string &s1, const string &s2,
	bool(*pf)(const string &, const string &));
```

将函数做实参，会被自动转换为指针：
```cpp
useBigger(s1, s2, lengthCompare);
```

利用类型别名(§ 2.5.1)、decltype(§ 2.5.3)可以简化代码 ：
```cpp
// Func and Func2 have function type
typedef bool Func(const string&, const string&);
typedef decltype(lengthCompare) Func2; // equivalent type
// FuncP and FuncP2 have pointer to function type
typedef bool(*FuncP)(const string&, const string&);
typedef decltype(lengthCompare) *FuncP2;  // equivalent type
```

Here we’ve used `typedef` to define our types. Both Func and Func2 are function types, whereas FuncP and FuncP2 are pointer types. It is important to note that `decltype` returns the function type; the automatic conversion to pointer is not done. Because decltype returns a function type, if we want a pointer we must add the * ourselves. We can redeclare useBigger using any of these types:

```cpp
// equivalent declarations of useBigger using type aliases
void useBigger(const string&, const string&, Func);
void useBigger(const string&, const string&, FuncP2);
```

Both declarations declare the same function. In the first case, the compiler will automatically convert the function type represented by Functo a pointer.

##### 返回函数指针

与数组一样(§ 6.3.3)，我们不能返回一个函数类型但可以返回函数指针。且必须显式声明为指针类型。使用时，最好结合别名以简化书写：

```cpp
using F = int(int*, int); // F is a function type, not a pointer
using PF = int(*)(int*, int); // PF is a pointer type
```

The thing to keep in mind is that, unlike what happens to parameters that have function type, the return type is not automatically converted to a pointer type. We must explicitly specify that the return type is a pointer type:

```cpp
PF f1(int); // ok: PF是函数指针；f1返回这个指针
F f1(int); // 错误：F是函数类型；f1 can't return a function
F *f1(int); // ok: explicitly specify that the return type is a pointer to function
```

Of course, we can also declare `f1` directly, which we’d do as

```cpp
	int (*f1(int))(int*,int);
```

Reading this declaration from the inside out, we see that f1 has a parameter list, so f1 is a function. f1 is preceded by a * so f1 returns a pointer. The type of that pointer itself has a parameter list, so the pointer points to a function. That function returns an int.

For completeness, it’s worth noting that we can simplify declarations of functions that return pointers to function by using a trailing return (§ 6.3.3):

```cpp
auto f1(int)-> int (*)(int*, int);
```

##### Using auto or decltype for Function Pointer Types

If we know which function(s) we want to return, we can use `decltype` to simplify writing a function pointer return type. For example, assume we have two functions, both of which return a `string::size_type` and have two const string& parameters. We can write a third function that takes a string parameter and returns a pointer to one of these two functions as follows:

```cpp
string::size_type sumLength(conststring&, const string&);
string::size_type largerLength(const string&, const string&);
// depending on the value of its string parameter,
// getFcn returns a pointer to sumLength or to largerLength
decltype(sumLength) *getFcn(const string &);
```

The only tricky part in declaring getFcnis to remember that when we apply decltype to a function, it returns a function type, not a pointer to function type. We must add a * to indicate that we are returning a pointer, not a function.











	