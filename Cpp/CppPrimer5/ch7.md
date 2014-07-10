[toc]

## 7 类

### 7.1. 定义抽象数据类型

#### 7.1.1. 定义`Sales_data`类

`Sales_data`相关接口包含以下操作：

- An isb nmember function to return the object’s ISBN
- A combine member function to add one Sales_data object into another
- A function named add to add two `Sales_data` objects
- A read function to read data from an istream into a `Sales_data` object
- A print function to print the value of a `Sales_data` object on an ostream

##### 使用`Sales_data`类

在学习定义之前，先看看如何使用：

```cpp
	Sales_data total;
	if (read(cin, total)) { // read the first transaction
		Sales_data trans;
		while(read(cin, trans)) {
			if(total.isbn() == trans.isbn())
				total.combine(trans); // update the running total
			else {
                print(cout, total) << endl;
                total = trans; // process the next book
            }
        }
        print(cout, total) << endl; // print the last transaction
    } else {  // there was no input
        cerr << "No data?!" << endl;  // notify the user
    }
```

#### 7.1.2. 定义`Sales_data`

成员函数可以定义在类内，或类外。非成员函数但属于接口的，（如`add`, `read`）定义在类外。

```cpp
    struct Sales_data{
        // new members: operations on Sales_data objects
        std::string isbn() const { return bookNo; }
        Sales_data& combine(const Sales_data&);
        double avg_price() const;
        // data members
        std::string bookNo;
        unsigned units_sold = 0;
        double revenue = 0.0;
    };

    // 非成员但属于接口
    Sales_data add(const Sales_data&, const Sales_data&);
    std::ostream &print(std::ostream&, const Sales_data&);
    std::istream &read(std::istream&, Sales_data&);
```

> 定义在类内的函数是隐式内联的(§ 6.5.2)。

##### `this`

`isbn`使用`this`的等价实现：

```cpp
	std::string isbn() const { return this->bookNo; }
```

##### const成员函数

注意到`isbn`函数后面的`const`关键字。`const`用于限定隐式的`this`指针。

默认，一个指向类的、非const版本的指针的类型是常量指针（指针本身是常量，不是指针指向的是常量，因为不能改变this的绑定）。例如`Sales_data`的成员函数内的`this`的类型是`Sales_data * const`。**这样的指针不能绑定到常量对象**。于是，我们不能在一个常量对象上调用一个普通的成员函数。｛｛例如，假如有`const Sales_data data`，这里data是一个常量对象。｝｝

因为`isbn`其实不会修改对象，我们期望函数内的`this`是个`const Sales_data * const`，即，指向常量的指针。实现方法是，在函数参数列表后添加`const`关键字。这种函数称为常量成员函数。

> 常量对象，或指向常量对象的指针、引用，只能调用常量成员函数。

##### 类作用域与成员函数

类自己是一个作用域。成员函数的作用域嵌套在类的作用域内。因此在`isbn`中使用`bookNo`，会被解析成`Sales_data`中的数据成员。

注意到`isbn`可以使用`bookNo`，虽然它定义在`isbn`之后。§ 7.4.1会谈到，编译器会分两步处理类。成员的声明先被编译，然后是成员函数的函数体。因此在成员函数内部，可以使用其他成员，不论定义的先后顺序。

##### 在类外 *定义* 一个成员函数

如果成员被声明为常量成员函数，则定义也必须带`const`关键字。同时前面必须加类名限定：

```cpp
    double Sales_data::avg_price() const {
        if(units_sold)
        	return revenue/units_sold;
        else
         return 0;
    }
```

当编译器遇见函数名时，剩下的代码就相当于位于类的作用域内。因此`revenue`和`units_sold`都会被解析为`Sales_data`的数据成员。

##### 函数返回对象本身

`combine`需要返回对象本身，且以左值的形式。于是返回类型必须是引用类型。通过`*this`获得当前对象。

```cpp
    Sales_data& Sales_data::combine(const Sales_data &rhs)
    {
        units_sold += rhs.units_sold; // add the members of rhs into
        revenue += rhs.revenue;
        return *this; // 返回对象自身
    }
```

#### 7.1.3. 定义与类相关的非成员函数

类作者常要定义一些辅助函数，如`add`、`read`、`print`函数。尽快这些函数概念上是接口的一部分，但它们却不属于类。与普通函数类似，它们的声明和定义一般是分开的。这些函数的声明一般与类声明放在同一个头文件。

#### 7.1.4. 构造器

构造器是个很复杂的主题，后续还会有多个章节涉及。

构造器没有返回值。类可以有多个构造器。

构造器不能声明为常量函数(§7.1.2)。对象在构造完后才能是常量的。

##### 合成的默认构造器

之前的`Sales_data`类没有定义任何构造器。却可以被正常使用。

```cpp
	Sales_data total;
	Sales_data trans;
```

上面两个变量，将被默认初始化。类通过默认一个特殊的构造器控制默认初始化：默认构造器。默认构造器是无参构造器。

若类不定义默认构造器，编译器将自动产生一个，称为合成（synthesized）默认构造器。对于多数类，该默认构造器初始化成员的方式是：

- 如果有**类内初始化**(§ 2.6.1)，用它先初始化成员
- 否则，默认初始化(§ 2.2.1)成员。

##### 一些类不能依赖合成的默认构造器

只有相对简单的类可以依赖合成的默认构造器。最常见的原因是，仅当我们不定义其他构造器时，编译器才会产生默认构造器。即，如果开发人员需要控制类的初始化，则需要全面控制。

第二个原因是，合成的构造器有时会做错事。例如，内建或复合类型（例如数组和指针）的对象在一个块内定义时，若被默认初始化，则其值是未定的(§ 2.2.1)。**类的成员也适用此规则**。于是这类成员应在类中被初始化，或为其定义构造器。否则类的实例中此成员的值将是未定义的｛｛未定义不是零或null。未定义比零或null可怕。可能是非零值。这是根本无法判断这个值是否有意义｝｝。

第三个原因是，编译器有时无法合成。例如，一个成员是**类**类型，且那个类没有默认构造器，此时编译器将无法初始化该成员。此时，需要我们自己定义默认构造器。否则类将没有可用的默认构造器。We’ll see in § 13.1.6 additional circumstances that prevent the compiler from generating an appropriate default constructor.

##### 定义`Sales_data`的构造器

为`Sales_data`定义四个构造器，分别接收以下参数：

- `istream&` from which to read a transaction.
- A `const string&` representing an ISBN, an `unsigned` representing the count of how many books were sold, and a `double` representing the price at which the books sold.
- A `const string&` representing an ISBN. This constructor will use default values for the other members.
- An empty parameter list (i.e., the default constructor) which as we’ve just seen we must define because we have defined other constructors.

```cpp
    struct Sales_data{
        // constructors added
        Sales_data() = default;
        Sales_data(const std::string &s): bookNo(s) { }
        Sales_data(const std::string &s, unsigned n, double p):
        	bookNo(s), units_sold(n), revenue(p*n) { }
        Sales_data(std::istream&);

        // other members as before
        std::string isbn() const { return bookNo; }
        Sales_data& combine(const Sales_data&);
        double avg_price() const;
        std::string bookNo;
        unsigned units_sold = 0;
        double revenue = 0.0;
    };
```

##### `= default`

先解释默认构造器：

```cpp
	Sales_data() = default;
```

这里要定义默认构造器的唯一原因是我们需要定义其他构造器。我们想这个构造器与合成的相同。在新标准下，可以用`= default`告诉编译器为我们产生默认构造器。`= default`可以放在类内声明处**或**在类外定义处。与其他函数一样，如果`= default`出现在类内，则默认构造器将被内联；如果在类外定义处，成员默认将不会内联。

##### 构造器初始化列表

```cpp
	Sales_data(const std::string&s): bookNo(s) { }
	Sales_data(const std::string &s, unsigned n, double p):
		bookNo(s), units_sold(n), revenue(p*n) { }
```

冒号之后的是构造器的**初始化列表**：定义一个或多个数据成员的初始值。列表中多个成员逗号分隔，成员的初始值在括号（或花括号）内。未在初始化类别中出现的成员的初始化与合成构造器的初始化方式一致。

It is usually best for a constructor to use an in-class initializer｛｛为什么是构造器使用类内初始化，这里的意思是利用？——如果有有类内初始化，构造器应该依赖那个初始化，不要自己再初始化｝｝ if one exists and gives the member the correct value. On the other hand, **if your compiler does not yet support in-class initializers**, then every constructor should explicitly initialize every member of **built-in** type.｛｛反而是内建类型需要被初始化！｝｝

> 最佳实践：构建器不应覆盖类内初始化，除非想用另一个值。If you can’t use in-class initializers, each constructor should explicitly initialize every member of built-in type.

##### 在类外定义构造器

```cpp
    Sales_data::Sales_data(std::istream &is)
    {
    	read(is, *this);
    }
```

注意，即使上面的构造器的初始化列表为空，**成员仍会先被初始化，再执行构造器内的代码**。未出现在初始化列表中的成员被**类内初始化**或**默认初始化**。对于`Sales_data`意味着，当构造器执行时，`bookNo`将是空字符串，`units_sold`和`revenue`是0。

#### 7.1.5. 拷贝、赋值和销毁

对象在多种情况下可能被拷贝：如初始化一个变量时，向函数传值参时，或函数返回对象时(§ 6.2.1, § 6.3.2)。使用赋值运算符时对象被赋值(§ 4.4)。如果我们不定义这些函数，编译器将为我们合成。

例子，当执行`total = trans;`，似乎是指向：
```cpp
    // default assignment for Sales_data is equivalent to:
    total.bookNo = trans.bookNo;
    total.units_sold = trans.units_sold;
    total.revenue = trans.revenue;
```

We’ll show how we can define our own versions of these operations in Chapter 13.

##### 一些类不能依赖合成的版本

尽管编译器会合成拷贝、赋值、析构操作，但有时默认的版本的行为并不正确。In particular, the synthesized versions are unlikely to work correctly for classes that allocate resources that reside outside the class objects themselves. As one example, in Chapter 12 we’ll see how C++ programs allocate and manage dynamic memory. § 13.1.4会讲到，涉及管理动态内存的类，一般不能依赖合成的默认版本。

但值得注意的是，多数需要动态内存的类，可以（并且应该）使用`vector`或`string`来关系所需内存。Classes that use vectors and strings avoid the complexities involved in allocating and deallocating memory.

合成拷贝、赋值、析构操作，能够正确处理含有`vector`或`string`成员的类。当我们拷贝或赋值的一个`vector`成员，`vector`类会负责正确的拷贝或赋值。当对象被销毁时，会销毁`vector`成员，销毁`vector`内的所有成员。

### 7.2. 访问控制与封装

```cpp
    class Sales_data{
    public: // 访问控制
        Sales_data() = default;
        Sales_data(conststd::string &s, unsigned n, double p):
        	bookNo(s),units_sold(n), revenue(p*n) { }
        Sales_data(conststd::string &s): bookNo(s) { }
        Sales_data(std::istream&);
        std::string isbn() const { return bookNo; }
        Sales_data& combine(const Sales_data&);
    private: // 访问控制
        double avg_price() const
        	{return units_sold ? revenue/units_sold : 0; }
        std::string ookNo;
        unsigned units_sold = 0;
        double revenue = 0.0;
    };
```

类可以包含零个或多个访问说明符。同一个说明符可以出现多次。说明符的影响直到下一个说明符出现为止。

##### `class`和`struct`关键字

`class`和`struct`都能定义类。二者唯一的区别是默认的访问级别不同。用`struct`则默认是public，用`class`默认是private。

As a matter of programming style, when we define a class intending for all of its members to be public, we use struct. If we intend to have private members, then we use class.

#### 7.2.1. 友元

现在`Sales_data`的数据成员是私有的，于是`read`, `print`将不再能通过编译。

类可以令其他类或函数称为类的友元，获得对非公有成员的访问。要声明一个函数是友元，在类中列出函数的声明，并添加`friend`关键字：

```cpp
    class Sales_data {
    // 友元声明
    friend Sales_data add(const Sales_data&, const Sales_data&);
    friend std::istream &read(std::istream&, Sales_data&);
    friend std::ostream &print(std::ostream&, const Sales_data&);
    // other members and access specifiers as before
    public:
        Sales_data() = default;
        Sales_data(conststd::string &s, unsigned n, double p):
        	bookNo(s),units_sold(n), revenue(p*n) { }
        Sales_data(const std::string &s): bookNo(s) { }
        Sales_data(std::istream&);
        std::string isbn() const { return bookNo; }
        Sales_data& combine(const Sales_data&);
    private:
        std::string bookNo;
        unsigned units_sold = 0;
        double revenue = 0.0;
    };

    // 函数声明
    Sales_data add(const Sales_data&, const Sales_data&);
    std::istream &read(std::istream&, Sales_data&);
    std::ostream &print(std::ostream&, const Sales_data&);
```

友元声明只能放在类定义内。可以出现在类的任意位置。友元不是类的成员，因此不受访问控制节的影响。

##### 友元的声明

友元的声明只声明访问性。它不是在声明函数。因此，函数仍需要在某处声明。为了方便类使用者使用友元，一般将友元声明与类定义放在一起。

Some compilers allow calls to a friend function when there is no ordinary declaration for that function. Even if your compiler allows such calls, it is a good idea to provide separate declarations for friends.

### 7.3. 类的其他特性

These features include type members, in-class initializers for members of class type, mutable data members, inline member functions, returning `*this` from a member function, more about how we define and use class types, and class friendship.

#### 7.3.1. 类成员再探

##### 定义一个类型成员

除了数据成员和函数成员，类的成员还有类型成员。即定义类型的别名。且此名字的可访问性遵从一般成员：公有或私有。

```cpp
    class Screen {
    public:
    	typedef std::string::size_type pos;
    private:
        pos cursor = 0;
        pos height = 0, width = 0;
        std::string contents;
    };
```

除了使用`typedef`还可以使用`using`：

```cpp
    class Screen{
    public:
    	using pos = std::string::size_type;
    	// other members as before
    };
```

与其他成不同，数据成员要在使用前被定义。因此，数据成员一般在类的最前面。原因在7.4.1解释。

##### 令成员内联

定义在类内的成员函数自动是内联的。

可以在类内显式声明函数是内联的。还可以在类外函数定义处指定函数是内联的：

```cpp
    inline Screen &Screen::move(pos r, pos c)
    {
        pos row = r * width;
        cursor= row + c ;
        return *this;
    }
```

可以在声明和定义处都指定函数是内联的。但根本不需要这么做。

> 内联成员函数与内联函数一样，应该定义在头文件，与类定义在一起。

##### 重载成员函数

成员函数一样可以被重载。The same function-matching (§ 6.4) process is used for calls to member functions as for nonmember functions.

##### `mutable`数据成员

极端情况下，我们想修改一个数据成员，即使在一个常量成员函数中。这种数据成员需要在声明时加`mutable`关键字。`mutable`数据成员不能是`const`，即使它是一个常量对象的成员。

例子，给`Screen`一个mutable成员`access_ctr`，用于追逐所有成员函数被调用次数：

```cpp
    class Screen{
    public:
    	void some_member() const;
    private:
    	mutable size_t access_ctr;
    // other members as before
    };
    void Screen::some_member() const
    {
    	++access_ctr;
    	// ...
    }
```

尽管`some_member`是一个常量成员函数，它可以改变`access_ctr`的值。

##### 类类型的成员的初始化

新标准中，提供默认值**最好**的方式是使用**类内初始化**(§ 2.6.1)：

```cpp
    class Window_mgr {
        private:
	        std::vector<Screen> screens{ Screen(24, 80, ' ') };
    };
```

类内初始化必须使用`=`形式的初始化，或**直接初始化**（使用花括号）。

#### 7.3.2. 返回`*this`的函数

在常量方法内返回`*this`，返回类型应是`const Sales_data &`。

##### 根据`const`重载

我们可以基于成员函数是否常量成员函数重载函数，其原因，与可以根据指针形参指向的是否是常量重载函数的原因(§ 6.4)相同。非常量的版本不会成为常量对象的可行（viable）函数。对常量对象，只能调用常量成员函数。对于非常量对象，可以调用任意版本，但非常量版本将更好的匹配。

下面将定义一个私有成员函数`do_display`，`display`函数将调用该函数，最后返回调用对象本身：

```cpp
    class Screen {
    public:
    	// 根据对象是否是常量重载
    	Screen& display(std::ostream &os)
    		{do_display(os); return *this; }
    	const Screen &display(std::ostream &os) const
    		{do_display(os); return *this; }
    private:
        // function to do the work of displaying a Screen
        void do_display(std::ostream &os) const {os << contents;}
        // other members as before
    };
```

与其他上下文一样，当成员函数调用另一个成员函数时，`this`指针会被隐式的传递。因此当`display`调用`do_display`，它自己的`this`指针会被隐式传递给`do_display`。当非常量版本的`display`调用`do_display`，它的`this`指针会被隐式的从指向非常量的指针转换为指向常量的指针(§ 4.11.2)。

最后`display`会返回对象自身。对于非常量函数，`this`指向非常量对象，因此函数返回的引用指向非常量。常量成员函数返回的引用指向常量。

当我们调用`display`时，调用哪个函数取决于对象是否是常量：

```cpp
    Screen myScreen(5,3);
    const Screen blank(5, 3);
    myScreen.set('#').display(cout); // calls non const version
    blank.display(cout); // calls const version
```

#### 7.3.3. Class Types

We can refer to a class type directly, by using the class name as a type name. Alternatively, we can use the class name following the keyword `class` or `struct`:

```cpp
    Sales_data item1; // default-initialized object of type Sales_data
	class Sales_data item1; // equivalent declaration
```

Both methods of referring to a class type are equivalent. The second method is inherited from C and is also valid in C++.

##### 类声明

就像可以分离函数的声明与定义一样，可以声明一个类，而不是定义它：

```cpp
	class Screen; // declaration of the Screen class
```

这种声明，有时被称为前向声明，introduces the name Screen into the program and indicates that Screen refers to a class type.

After a declaration and before a definition is seen, the type Screen is an incomplete type—it’s known that Screen is a class type but not known what members that type contains.

不完整类型的使用是受限的：可以定义此类型的指针或引用类型，可以声明（不是定义）使用此类型做参数或返回值的函数。

A class must be defined—not just declared—before we can write code that creates objects of that type. Otherwise, the compiler does not know how much storage such objects need. Similarly, the class must be defined before a reference or pointer is used to access a member of the type. After all, if the class has not been defined, the compiler can’t know what members the class has.

With one exception that we’ll describe in § 7.6(p. 300), data members can be specified to be of a class type only if the class has been defined. The type must be complete because the compiler needs to know how much storage the data member requires. 因为类在类Body结束后才算定义完成，因此类不能定义自己类类型的成员。但可以定义指向自己类类型的成员的指针或引用：

```cpp
	class Link_screen{
		Screen window;
		Link_screen *next;
		Link_screen *prev;
	};
```

#### 7.3.4. 再探友元

类的友元可以是另一个类。In addition, a friend function can be defined inside the class body. Such functions are implicitly inline.

##### 类之间的友元

让`Window_mgr`称为`Screen`的友元类：

```cpp
    class Screen {
        // Window_mgr的成员可以访问Screen的私有成员
        friend class Window_mgr;
        // ... rest of the Screen class
    };
````

注意友元不是传递的。例如，`Window_mgr`的友元对`Screen`没有特殊的访问权限。

##### Making A Member Function a Friend

如果不想让整个`Window_mgr`类做友元，只想其`clear`函数做友元。可以

```cpp
    class Screen{
        // Window_mgr::clear must have been declared before class Screen
        friend void Window_mgr::clear(ScreenIndex);
        // ... rest of the Screen class
    };
```

Making a member function a friend requires careful structuring of our programs to accommodate interdependencies among the declarations and definitions. In this example, we must order our program as follows:

- 先定义`Window_mgr`，声明、但不能定义clear。`Screen`必须在`clear`使用其成员前声明。
- 然后，定义`Screen`，包含对友元函数`clear`的声明。
- 最后，定义`clear`。

##### 重载函数与友元

一次只能令一个版本成为友元。

```cpp
    // overloaded storeOn functions
    extern std::ostream& storeOn(std::ostream &, Screen &);
    extern BitMap& storeOn(BitMap &, Screen &);
    class Screen {
        //ostream version ofstoreOn may access theprivate parts of
        Screen objects
        friend std::ostream& storeOn(std::ostream &, Screen &);
        // . . .
    };
```

##### 友元声明与作用域

Classes and nonmember functions need not have been declared before they are used in a friend declaration. When a name first appears in a friend declaration, that name is implicitly assumed to be part of the surrounding scope. However, the friend itself is not actually declared in that scope (§ 7.2.1). Even if we define the function inside the class, we must still provide a declaration outside of the class itself to make that function visible. A declaration must exist even if we only call the friend from members of the friendship granting class:

```cpp
    struct X{
        friend void f() { /* friend function can be defined in the class body  */}
        X(){ f(); } // error: no declaration for f
        void g();
        void h();
    };
    void X::g() { return f(); } // error: f hasn't been declared
    void f(); // declares the function defined inside X
    void X::h() { return f(); } // ok: declaration for f is now in scope
```

It is important to understand that a friend declaration affects access but is not a declaration in an ordinary sense.

> Remember, somecompilers do not enforce the lookup rules for friends (§7.2.1).

### 7.4. Class Scope

每个类定义了自己的新的作用域。在类作用域之外，普通数据成员和函数成员需要通过成员访问运算符(§ 4.6)才能访问。在类外通过作用域运算符访问类型成员。

```cpp
	Screen::pos ht= 24, wd = 80; // use the pos type defined by Screen
	Screen scr(ht, wd, ' ');
    Screen *p = &scr;
    char c = scr.get(); // fetches the get member from the object scr
    c = p->get();  // fetchesthe get member from the object to which p points
```

##### Scope and embers Defined outside the Class

类是一个作用域解释了，为什么在类外定义成员函数时需要提供类名(§ 7.1.2)。从类名出现处往后，包括参数列表和函数体，都在类的作用域内。于是可以不加限定符访问其他成员。

例如
```cpp
    void Window_mgr::clear(ScreenIndex i)
    {
        Screen &s = screens[i];
        s.contents= string(s.height * s.width, ' ');
    }
```

其中`ScreenIndex`是定义在类`WindowMgr`内的一个类型。

但函数返回类型一般在函数名前。于是此类型在类作用域之外。于是返回类型必须指定所属类。

```cpp
    class Window_mgr {
    public:
    	// add a Screen to the window and returns its index
    	ScreenIndex addScreen(const Screen&);
    	// other members as before
    };
    // return type is seen before we're in the scope of Window_mgr
    Window_mgr::ScreenIndex  Window_mgr::addScreen(const Screen &s)
    {
    	screens.push_back(s);
    	returnscreens.size() - 1;
    }
```

#### （未）7.4.1. Name Lookup and Class Scope

In the programs we’ve written so far, **name lookup**(the process of finding which declarations match the use of a name) has been relatively straightforward:


- First, look for a declaration of the name in the block in which the name was used. Only names declared before the use are considered.
- If the name isn’t found, look in the enclosing scope(s).
- If no declaration is found, then the program is in error.

The way names are resolved inside member functions defined inside the class may seem to behave differently than these lookup rules. However, in this case,
appearances are deceiving. Class definitions are processed in two phases:

- 首先编译成员声明
- Function bodies are compiled only after the entire class has been seen.

> Member function definitions are processed afterthe compiler processes all of the declarations in the class.

Classes are processed in this two-phase way to make it easier to organize class code. Because member function bodies are not processed until the entire class is seen, they can use any name defined inside the class. If function definitions were processed at the same time as the member declarations, then we would have to order the member functions so that they referred only to names already seen.

##### Name Lookup for Class Member Declarations

This two-step process applies only to names used in the body of a member function. Names used in declarations, including names used for the return type and types in the parameter list, must be seen before they are used. If a member declaration uses a name that has not yet been seen inside the class, the compiler will look for that name
in the scope(s) in which the class is defined. For example:

```cpp
    typedef double Money;
    string bal;
    class Account {
    public:
        Money balance() { return bal; }
    private:
        Moneybal;
        //...
    };
```

When the compiler sees the declaration of the balance function, it will look for a declaration of Money in the Account class. The compiler considers only declarations inside Account that appear before the use of Money. Because no matching member is found, the compiler then looks for a declaration in the enclosing scope(s). In this example, the compiler will find the `typedef` of `Money`. That type will be used for the return type of the function balanceand as the type for the data member bal. On the other hand, the function body of balance is processed only after the entire class is seen. Thus, the return inside that function returns the member named bal, not the string from the outer scope.

##### Type Names Are Special

Ordinarily, an inner scope can redefine a name from an outer scope even if that nam has already been used in the inner scope. However, in a class, if a member uses a　name from an outer scope and that name is a type, then the class may not　subsequently redefine that name:

```cpp
    typedef double Money;
    class Account {
    public:
 		Money balance() { return bal; }  // uses Money fromthe outer scope
    private:
    	typedef double Money; // error: cannot redefine Money
    	Money bal;
    //...
    };
```

It is worth noting that even though the definition of Money inside Account uses the same type as the definition in the outer scope, this code is still in error. Although it is an error to redefine a type name, compilers are not required to diagnose this error. Some compilers will quietly accept such code, even though the program is in error.

> 类型名定义最好在类开始。这样使用这个类型的任何成员将能看到这个类型名。

##### Normal Block-Scope Name Lookup inside Member Definitions

A name used in the body of a member function is resolved as follows:

- First, look for a declaration of the name inside the member function. As usual, only declarations in the function body that precede the use of the name are considered.
- If the declaration is not found inside the member function, look for a declaration inside the class. All the members of the class are considered.
- If a declaration for the name is not found in the class, look for a declaration that is in scope before the member function definition.

Ordinarily, it is a bad idea to use the name of another member as the name for a parameter in a member function. However, in order to show how names are resolved, we’ll violate that normal practice in our `dummy_fcn` function:

```cpp
    // note:this code is for illustration purposes only and reflects bad practice
    // it is generally a bad idea to use the same name for a parameter and a member
    int height; // defines a name subsequently used inside Screen
    class Screen {
    public:
        typedef std::string::size_type pos;
        void dummy_fcn(pos height) {
            cursor = width * height; // which height? the parameter
        }
    private:
        pos cursor = 0;
        pos height = 0, width = 0;
    };
```

When the compiler processes the multiplication expression inside dummy_fcn, it first looks for the names used in that expression in the scope of that function. A function’s parameters are in the function’s scope. Thus, the name height, used in the body of
`dummy_fcn`, refers to this parameter declaration.

In this case, the heightparameter hides the member named height. If we
wanted to override the normal lookup rules, we can do so:
Click hereto view code image
// badpractice: names local to member functions shouldn't hide member names
void Screen::dummy_fcn(pos height) {
cursor= width * this->height;  // member height
// alternative way to indicate the member
cursor= width * Screen::height; // member height
}



























