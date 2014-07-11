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

#### 7.3.3. 类类型

当类型是类时，可以直接使用类名。或者，可以在类名前加`class`或`struct`：

```cpp
    Sales_data item1;
	class Sales_data item1; // 等价
```

第二种形式是从C继承来的。

##### 类声明

就像可以分离函数的声明与定义，可以声明一个类，而不是定义它：

```cpp
	class Screen; // 声明Screen类
```

这种声明，有时被称为前向声明。

声明之后，定义之前，类`Screen`是不完全的——其内部成员是未知的。不完整类型的使用是受限的：可以定义此类型的指针或引用类型，可以声明（不是定义）使用此类型做参数或返回值的函数。

创建类的对象前，类必须被定义。否则编译器不知道对象占用多少空间。Similarly, the class must be defined before a reference or pointer is used to access a member of the type.

若数据成员的类型是类A，则需要类A需要是以定义的（一个例外，见§ 7.6）。The type must be complete because the compiler needs to know how much storage the data member requires. 因为类在类Body结束后才算定义完成，因此类不能定义自己类类型的成员。但可以定义指向自己类类型的成员的指针或引用：

```cpp
	class Link_screen {
		Screen window;
		Link_screen *next;
		Link_screen *prev;
	};
```

#### 7.3.4. 再探友元

类的友元可以是另一个类。友元函数可以定义在类Body之内，这样的函数是隐式内联的。

##### 类之间的友元

让`Window_mgr`成为`Screen`的友元类：

```cpp
    class Screen {
        // Window_mgr的成员可以访问Screen的私有成员
        friend class Window_mgr;
        // ... rest of the Screen class
    };
````

注意友元不是传递的。例如，`Window_mgr`的友元对`Screen`没有特殊的访问权限。

##### 让成员函数做友元

如果不想让整个`Window_mgr`类做友元，只想其`clear`函数做友元。可以

```cpp
    class Screen {
        // Window_mgr::clear必须在类Screen之前声明
        friend void Window_mgr::clear(ScreenIndex);
        // ... rest of the Screen class
    };
```

让成员函数做友元，需要自习安排程序结构，安排声明和定义的顺序。这里顺序必须是：

- 先定义`Window_mgr`，声明、但不能定义`clear`。`Screen`必须在`clear`使用其成员前声明。
- 然后，定义`Screen`，包含对友元函数`clear`的声明。
- 最后，定义`clear`。

##### 重载函数与友元

一次只能令一个版本成为友元。

```cpp
    extern std::ostream& storeOn(std::ostream &, Screen &);
    extern BitMap& storeOn(BitMap &, Screen &);
    class Screen {
        friend std::ostream& storeOn(std::ostream &, Screen &);
        // . . .
    };
```

##### 友元声明与作用域

Classes and nonmember functions need not have been declared before they are used in a friend declaration. When a name first appears in a friend declaration, that name is implicitly assumed to be part of the surrounding scope. 但友元本身并不在这个作用域中(§ 7.2.1)。即使函数定义在类内部，也必须在类外提供一个函数声明。即使这个友元函数只会被类的成员函数调用，也必须有此声明：

```cpp
    struct X{
        friend void f() { /* 友元函数定义在类内部 */}
        X(){ f(); } // 错误：f未声明
        void g();
        void h();
    };
    void X::g() { return f(); } // 错误：f未被声明
    void f(); // 声明在这
    void X::h() { return f(); } // 现在可以了！
```

It is important to understand that a friend declaration affects access but is not a declaration in an ordinary sense.

> Remember, some compilers do not enforce the lookup rules for friends (§7.2.1).

### 7.4. 类作用域

类定义了自己的新的作用域。在类作用域之外，普通数据成员和函数成员需要通过成员访问运算符(§ 4.6)才能访问。在类外通过作用域运算符访问类型成员。

```cpp
	Screen::pos ht = 24, wd = 80; // pos是Screen中定义的类型
	Screen scr(ht, wd, ' ');
    Screen *p = &scr;
    char c = scr.get();
    c = p->get();
```

##### 作用域与在类外定义的成员

类是一个作用域解释了为什么在类外定义成员函数时需要提供类名(§ 7.1.2)。从类名出现处往后，包括参数列表和函数体，都在类的作用域内。于是可以不加限定符访问其他成员。

例如
```cpp
    void Window_mgr::clear(ScreenIndex i)
    {
        Screen &s = screens[i];
        s.contents = string(s.height * s.width, ' ');
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

#### 7.4.1. 名字查找与类作用域

目前所写的代码，名字查找（搜索哪个声明匹配对此名字的使用）都比较直接：

- First, look for a declaration of the name in the block in which the name was used. Only names declared before the use are considered.
- 如果名字没有找到，搜索嵌套作用域
- 如果没有声明，程序报错

The way names are resolved inside member functions defined inside the class may seem to behave differently than these lookup rules. However, in this case, appearances are deceiving. Class definitions are processed in two phases:

- 首先编译成员声明
- Function bodies are compiled only after the entire class has been seen.

> 成员函数定义的处理，发生在编译器处理了类中所有的声明之后。

Classes are processed in this two-phase way to make it easier to organize class code. Because member function bodies are not processed until the entire class is seen, 可以使用定义在类中的任意名字。If function definitions were processed at the same time as the member declarations, then we would have to order the member functions so that they referred only to names already seen.

##### Name Lookup for Class Member Declarations

This two-step process applies only to names used in the body of a member function. 声明中使用的名字，包括用作返回类型的名字，参数列表中的名字，必须在使用前可见。If a member declaration uses a name that has not yet been seen inside the class, the compiler will look for that name in the scope(s) in which the class is defined. For example:

```cpp
    typedef double Money;
    string bal;
    class Account {
    public:
        Money balance() { return bal; }
    private:
        Money bal;
        //...
    };
```

当编译器遇到`balance`函数声明时，它将在`Account`中寻找`Money`的声明。但编译器只会考虑`Account`中出现在使用`Money`之前的声明。因为找不到，于是寻找外层作用域。与此不同的是，`balance`的函数体在整个类可见后才会被处理。此时虽然`bal`声明在后面，但仍是可见的。

##### 类型名有特殊性

一般来说，内层作用域可以重新定义外层作用域的名字，即使在内存作用域内，该名字已被用过。但在类中，如果成员使用了外层作用域中的一个名字，且这个名字是一个类型名，则类不能在后面重新定义这个名字：

```cpp
    typedef double Money;
    class Account {
    public:
 		Money balance() { return bal; } // 使用外层作用域的Money
    private:
    	typedef double Money; // 错误：不能重新定义Money
    	Money bal;
    //...
    };
```

Although it is an error to redefine a type name, compilers are not required to diagnose this error. Some compilers will quietly accept such code, even though the program is in error.

> 类型名定义最好在类开始。这样使用这个类型的任何成员将能看到这个类型名。

##### Normal Block-Scope Name Lookup inside Member Definitions

在成员函数体内使用的名字的解析方式如下：

- 首先在成员函数内寻找名字的声明。与一般情况一样，只有在使用前声明的名字可用。
- 如果未找到，则在类内找声明。类的所有成员都会被考虑。
- If a declaration for the name is not found in the class, look for a declaration that is in scope before the member function definition.

一般情况下，不要使用成员的名字做成员函数参数的名字。However, in order to show how names are resolved, we’ll violate that normal practice in our `dummy_fcn` function:

```cpp
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

`dummy_fcn`内的`height`是参数列表中的`height`。若想访问数据成员`height`，可以：

```cpp
    void Screen::dummy_fcn(pos height) {
    	cursor= width * this->height; // member height
    	// 等价方式
	    cursor= width * Screen::height; // member height
    }
```

##### After Class Scope, Look in the Surrounding Scope

如果在函数内和类内都找不到名字，则在外层作用域中寻找。

若想使用外部作用域的名字，可以显式使用作用域运算符：

```cpp
	void Screen::dummy_fcn(pos height) {
		cursor = width * ::height; // which height? the global one
	}
```

##### Names Are Resolved Where They Appear within a File

When a member is defined outside its class, the third step of name lookup includes names declared in the scope of the member definition as well as those that appear in the scope of the class definition. For example:

```cpp
    int height; // defines a name subsequently used inside Screen
    class Screen {
    public:
        typedef std::string::size_type pos;
        void setHeight(pos);
        pos height = 0; // hides the declaration of height in the outer scope
    };
    Screen::pos verify(Screen::pos);
    void Screen::setHeight(pos var) {
        // var: refers to the parameter
        // height: refers to the class member
        // verify: refers to the global function
        height = verify(var);
    }
```

Notice that the declaration of the global function verify is not visible before the definition of the class Screen. However, the third step of name lookup includes the scope in which the member definition appears. In this example, the declaration for verify appears before setHeightis defined and may, therefore, be used.

### 7.5. 构造器再探

### 7.5.1. 构造器的初始化列表

定义变量时，我们一般会直接初始化它们；而不是先定义再初始化：

初始化跟赋值的区别，在对象数据成员上也一样。如果我们不在初始化列表中显式初始化成员，则在构造器体执行前，数据成员会被默认初始化。例如：

```cpp
    // 不好的写法
    Sales_data::Sales_data(const string &s, unsignedcnt, double price)
    {
        bookNo = s;
        units_sold = cnt;
        revenue = cnt * price;
    }
```

这里，是通过赋值给成员值，而不是初始化。

##### 构造器初始化{{列表}}有时是需要的

常常，但不总是可以忽略初始化成员和赋值的区别。常量成员和引用成员必须被初始化。类类型的成员，且类没有默认构造器的，也必须被初始化。例如：

```cpp
    class ConstRef{
    public:
    	ConstRef(int ii);
    private:
    	int i;
    	const int ci;
    	int& ri;
    };
```

`ci`和`ri`必须被初始化。As a result, omitting a constructor initializer for these members is an error:

```cpp
    // 错误！
    ConstRef::ConstRef(int ii)
    {  // 下面是赋值，不是初始化！
        i= ii; // ok
        ci= ii; // 错误：不能向常量赋值
        ri = i; // 错误：ri还未曾被初始化
    }
```

当构造器体开始执行后，初始化已经完成。初始化常量和引用唯一可能的地方是构造器初始化器。正确的写法是：

```cpp
	// ok: explicitly initialize reference and const members
	ConstRef::ConstRef(int ii): i(ii), ci(ii), ri(i) {  }
```

除了对错问题，还有效率问题。赋值多了一步。

##### 成员初始化顺序

每个成员只能在构造器初始化器中出现一次。

初始化列表只是给定初始化值，但未规定初始化执行顺序。

成员初始化的顺序由它们在类内定义的顺序决定。初始化列表顺序无关紧要。

少数情况下，初始化顺序是重要，例如：

```cpp
    class X{
        int i;
        int j;
    public:
        // undefined: i is initialized before j
        X(int val): j(val), i(j) { }
    };
```

In this case, the constructor initializer makes it appearas if j is initialized with val and then j is used to initialize i. However, i is initialized first. The effect of this initializer is to initialize i with the undefined value of j!

Some compilers are kind enough to generate a warning if the data members are listed in the constructor initializer in a different order from the order in which the members are declared.

> 最佳实践：构造器初始化列表的顺序最好与成员定义的顺序一致。最好不要用一个成员初始化另一个。

##### 默认实参与构造器

`Sales_data`默认的构造器，与只取一个`string`参数的构造器可以合并成一个——利用默认实参：

```cpp
    class Sales_data {
    public:
    Sales_data(std::strings = ""): bookNo(s) { }
    // remaining constructors unchanged
    Sales_data(std::strings, unsigned cnt, double rev):
    	bookNo(s),units_sold(cnt), revenue(rev*cnt) { }
    Sales_data(std::istream&is) { read(is, *this); }
    // remaining members as before
    };
```

这样的类仍算有默认构造器。

> A constructor that supplies default arguments for all its parameters also defines the default constructor.

#### 7.5.2. 代理构造器

新标准扩展了构造器初始化，允许使用代理构造器（delegating constructors）。代理构造器利用同一个类内的其他构造器完成初始化。

与其他构造器一样，代理构造器有一个成员初始化列表和一个函数体。它的成员初始化列表只有一项：类名，接着是括号内的一组参数。实参列表必须匹配类内的另一个构造器。

As an example, we’ll rewrite the `Sales_data` class to use delegating constructors as follows:

```cpp
    class Sales_data{
    public:
        // 非代理构造器
        Sales_data(std::strings, unsigned cnt, double price):
        	bookNo(s), units_sold(cnt), revenue(cnt * price) {
        }
        // 下面都是代理构造器
        Sales_data(): Sales_data("", 0, 0) {}
        Sales_data(std::string s): Sales_data(s, 0,0) {}
        Sales_data(std::istream& is): Sales_data()
            {read(is, *this); }
        // other members as before
    };
```

取`istream&`的构造器代理到默认构造器，默认构造器再代理到三个参数的构造器。Once those constructors complete their work, the body of the `istream&` constructor is run.

若构造器A代理到构造器B，则A的函数体要执行完，才能轮到B的函数体执行。

#### 7.5.3. 默认构造器的角色

当对象需要被默认初始化或值初始化时，默认构造器会被自动使用。默认初始化发生在：

- 在块级作用域内定义非静态变量或数组，但没有初始化器
- 数据成员是类类型，且使用合成的默认构造器
- 数据成员是类类型，但未在构造器初始化列表中显式初始化

值初始化发生在：

- 数组初始化时，我们提供的初始化器数量少于数组大小(§ 3.5.1)
- 定义局部静态对象，但未提供初始化器(§ 6.1.1)
- 当我们显式请求值初始化时，即使用`T()`的形式。这里`T`是一个类型（`vector`的构造器需要一个参数，指定`vector`大小）。

What may be less obvious is the impact on classes that have data members that do not have a default constructor:

```cpp
    class NoDefault{
    public:
    	NoDefault(const std::string&);
    	// additional members follow, but no other constructors
    };
    struct A {
    	NoDefault my_mem;
    };
    A a; // 错误：无法合成A的默认构造器
    struct B {
    	B(){} // 错误：未定义b_member的初始化
    	NoDefault b_member;
    };
```

##### Using the Default Constructor
The following declaration of objcompiles without complaint. However, when we try
to use obj
Click hereto view code image
Sales_data obj();  // ok: but defines a function, not an object
if (obj.isbn() == Primer_5th_ed.isbn())  // error: obj is a function
the compiler complains that we cannot apply member access notation to a function.
The problem is that, although we intended to declare a default-initialized object, obj
actually declares a function taking no parameters and returning an object of type Sales_data.
The correct way to define an object that uses the default constructor for initialization
is to leave off the trailing, empty parentheses:
Click hereto view code image
// ok: obj isa default-initialized object
Sales_data obj;

新手常犯的错误，想要使用默认构造器，却写成了
```cpp
	Sales_data obj(); // oops! declares a function, not an object
```
正确写法：
```cpp
	Sales_data obj2; // ok: obj2 is an object, not a function
```

#### 7.5.4. Implicit Class-Type Conversions

§ 4.11讲过，语言定义了几种内建类型的自动转换。类也可以定义隐式转换。每个可以被一个实参调用的构造器定义了一个从参数类型到该类的隐式转换。这类构造器有时被称为转换构造器。We’ll see in § 14.9 how to define conversions froma class type to another type.

`Sales_data`有两个这样的构造器。分别取string和istream参数。即，在期待`Sales_data`的地方，我们可以提供一个string或istream：

```cpp
	string null_book = "9-999-99999-9";
	// constructs a temporary Sales_data object
	item.combine(null_book);
```

`combine`方法本来接收`Sales_data`参数。编译器自动从string创建一个`Sales_data`。Because combine’s parameter is a reference to const, we can pass a
temporary to that parameter.

##### Only One Class-Type Conversion Is Allowed

In § 4.11.2 we noted that the compiler will automatically apply only one class-type conversion. 例如下面代码错误的原因是需要两次转换：


```cpp
    // (1) convert "9-999-99999-9" to string
    // (2) convert that (temporary) string to Sales_data
    item.combine("9-999-99999-9");
```

我们可以显式的将字符字面量转换为`string`或`Sales_data`：

```cpp
    item.combine(string("9-999-99999-9"));
    item.combine(Sales_data("9-999-99999-9"));
```

##### Class-Type Conversions Are Not Always Useful

Whether the conversion of a string to Sales_data is desired depends on how we think our users will use the conversion. In this case, it might be okay. The string in `null_book` probably represents a nonexistent ISBN.

More problematic is the conversion from istream to `Sales_data`:

```cpp
    // uses the istream constructor to build an object to pass to combine
    item.combine(cin);
```

This code implicitly converts cin to `Sales_data`. This conversion executes the `Sales_data` constructor that takes an istream. That constructor creates a (temporary) `Sales_data` object by reading the standard input. That object is then passed to combine.

This `Sales_data` object is a temporary (§ 2.4.1). We have no access to it once `combine` finishes. Effectively, we have constructed an object that is discarded after we add its value into `item`.

##### Suppressing Implicit Conversions Defined by Constructors

We can prevent the use of a constructor in a context that requires an implicit conversion by declaring the constructor as `explicit`:

```cpp
    class Sales_data{
    public:
    	Sales_data() = default;
    	Sales_data(const std::string &s, unsigned n, double p):
    		bookNo(s), units_sold(n), revenue(p*n) { }
    	explicit Sales_data(const std::string &s): bookNo(s) { }
    	explicit Sales_data(std::istream&);
    	// remaining members as before
    };
```

Now, neither constructor can be used to implicitly create a Sales_dataobject. Neither of our previous uses will compile:

```cpp
	item.combine(null_book); // error: string constructor is explicit
	item.combine(cin); // error: istream constructor is explicit
```

The `explicit` keyword is meaningful only on constructors that can be called with a single argument. Constructors that require more arguments are not used to perform an implicit conversion, so there is no need to designate such constructors as explicit. `explicit`关键字只出现在类中的构造器声明处。不能在类外的定义处：

```cpp
    // error: explicit allowed only on a constructor declaration
    explicit Sales_data::Sales_data(istream& is)
    {
    	read(is,*this);
    }
```

##### explicit Constructors Can Be Used Only for Direct Initialization

One context in which implicit conversions happen is when we use the **copy** form of initialization (with an `=` ) (§ 3.2.1). We cannot use an explicit constructor with this form of initialization; we must use direct initialization:

```cpp
    Sales_data item1(null_book); // ok: direct initialization
    // error: cannot use the copy form of initialization with an explicit constructor
    Sales_data item2 = null_book;
```

##### Explicitly Using Constructors for Conversions

Although the compiler will not use an explicit constructor for an implicit conversion, we can use such constructors explicitly to force a conversion:

```cpp
	// ok: the argument is an explicitly constructed Sales_data object
	item.combine(Sales_data(null_book));
	// ok: static_cast can use an explicit constructor
	item.combine(static_cast<Sales_data>(cin));
````

In the second call, we use a `static_cast`(§ 4.11.3) to perform an explicit, rather than an implicit, conversion.

##### Library Classes with explicit Constructors

Some of the library classes that we’ve used have single-parameter constructors:

- The string constructor that takes a single parameter of type `const char*` (§ 3.2.1) is **not** explicit.
- The `vector` constructor that takes a size (§ 3.3.1) is explicit.

#### 7.5.5. Aggregate Classes

An aggregate class gives users direct access to its members and has special initialization syntax. A class is an aggregate if

- 所有的数据成员是共有的
- 不定义任何构造器
- 无类内初始化(§ 2.6.1)
- 没有基类或虚函数

For example, the following class is an aggregate:
```cpp
    struct Data{
        int ival;
        string s;
    };
```

We can initialize the data members of an aggregate class by providing a braced list of member initializers:

```cpp
    // val1.ival= 0; val1.s = string("Anna")
    Data val1 = { 0, "Anna" };
```

The initializers must appear in declaration order of the data members.

As with initialization of array elements (§ 3.5.1), if the list of initializers has fewer elements than the class has members, the trailing members are value initialized (§ 3.5.1). The list of initializers must not contain more elements than the class has members.

It is worth noting that there are three significant drawbacks to explicitly initializing the members of an object of class type:

- It requires that all the data members of the class be public.
- It puts the burden on the user of the class (rather than on the class author) to correctly initialize every member of every object. Such initialization is tedious and error-prone because it is easy to forget an initializer or to supply an inappropriate initializer.
- If a member is added or removed, all initializations have to be updated.

#### 7.5.6. Literal Classes

In § 6.5.2 we noted that the parameters and return type of a `constexpr` function must be literal types. In addition to the arithmetic types, references, and pointers, certain classes are also literal types. Unlike other classes, classes that are literal types may have function members that are `constexpr`. Such members must meet all the requirements of a constexprfunction. These member functions are implicitly const(§ 7.1.2).

An aggregate class (§ 7.5.5) whose data members are all of literal type is a literal class. A nonaggregate class, that meets the following restrictions, is also a literal class:

- The data members all must have literal type.
- The class must have at least one constexpr constructor.
- If a data member has an in-class initializer, the initializer for a member of builtin type must be a constant expression (§ 2.4.4), or if the member has class type, the initializer must use the member’s own constexpr constructor.
- The class must use default definition for its destructor, which is the member that destroys objects of the class type (§ 7.1.5).

##### `constexpr` Constructors

Although constructors can’t be const(§ 7.1.4, p. 262), constructors in a literal class can be constexpr(§ 6.5.2, p. 239) functions. Indeed, a literal class must provide at least one constexpr constructor.

A constexpr constructor can be declared as = default(§ 7.1.4) (or as a deleted function, which we cover in § 13.1.6). Otherwise, a constexpr constructor must meet the requirements of a constructor—meaning it can have no return statement—and of a constexpr function—meaning the only executable statement it can have is a returnstatement (§ 6.5.2). As a result, the body of a constexpr constructor is typically empty. We define a constexpr constructor by preceding its declaration with the keyword constexpr:

class Debug{
public:
constexprDebug(bool b = true): hw(b), io(b), other(b) {
}
constexprDebug(bool h, bool i, bool o):
hw(h),io(i), other(o) {
}
constexprbool any() { return hw || io || other; }
voidset_io(bool b) { io = b; }
voidset_hw(bool b) { hw = b; }
voidset_other(bool b) { hw = b; }
private:
boolhw;  // hardware errors other than IO errors
boolio;  // IO errors
boolother; // other errors
};
A constexprconstructormust initialize every data member. The initializers must
either use a constexprconstructor or be a constant expression.
A constexprconstructoris used to generate objects that are constexprand for
parameters or return types in constexprfunctions:
Click hereto view code image
constexpr Debugio_sub(false, true, false);  // debugging IO
if (io_sub.any())  // equivalent to if(true)
cerr<< "print appropriate error messages" << endl;
constexpr Debug prod(false); // no debugging during production
if (prod.any())  // equivalent to if(false)
cerr<< "print an error message" << endl;

### 7.6. 静态类成员

##### 声明静态成员

向声明添加`static`关键字，则成员附属于类。静态成员也可以公有、私有。静态数据成员可以是常量、引用、数组、类类型等。

```cpp
    class Account{
    public:
    	void calculate() { amount += amount * interestRate; }
    	static double rate() { return interestRate; }
    	static void rate(double);
    private:
        std::string owner;
        double amount;
        static double interestRate;
        static double initRate();
    };
```

静态成员函数不与任何对象绑定，没有`this`指针。因此静态成员函数不能不能声明为`const`。不能在其中使用`this`，包括隐式的是使用——访问非静态成员。

##### 使用类静态成员

可以通过作用域运算符访问静态成员：

```cpp
    double r;
    r = Account::rate(); // access a static member using the scope operator
```

Even though static members are not part of the objects of its class, we can use an object, reference, or pointer of the class type to access a static member:

```cpp
    Account ac1;
    Account*ac2 = &ac1;
    // equivalent ways to call the static member rate function
    r = ac1.rate();  // through an Account object or reference
    r = ac2->rate();  // through a pointer to an Account object
```

Member functions can use static members directly, without the scope operator:

```cpp
    class Account{
    public:
    	void calculate() { amount += amount * interestRate; }
    private:
    	static double interestRate;
    	// remaining members as before
    };
```

##### 定义静态成员

与其他成员函数一样，静态成员函数的定义可以在类内或类外。在类外定义时不要重复`static`关键字。`static`只出现在声明处：

```cpp
    void Account::rate(double newRate)
    {
    	interestRate = newRate;
    }
```

静态成员不在构造器中初始化。也不在类内初始化。定义和初始化要放在类外。与其他对象一样，静态数据成员只能被定义一次。

与全局对象一样，静态数据成员定义在所有函数之外。因此一旦定义，它们存在直到程序结束。

```cpp
// define and initialize a static class member
double Account::interestRate = initRate();
```

Once the class name is seen, the remainder of the definition is in the scope of the class. As a result, we can use `initRate` without qualification as the initializer for rate.

> The best way to ensure that the object is defined exactly once is to put the definition of static data members in the same file that contains the definitions of the class non-inline member functions.

##### In-Class Initialization of static Data Members

Ordinarily, class static members may not be initialized in the class body. However, we can provide in-class initializers for static members that have const integral type and must do so for static members that are `constexpr`s of literal type (§7.5.6). The initializers must be constant expressions. Such members are themselves constant expressions; they can be used where a constant expression is required. For example, we can use an initialized staticdata member to specify the dimension of an array member:

```cpp
    class Account{
    public:
    	static double rate() { return interestRate; }
    	static void rate(double);
    private:
    	static constexpr int period = 30;// period is a constant expression
    double daily_tbl[period];
    };
````

If the member is used only in contexts where the compiler can substitute the member’s value, then an initialized constor constexpr staticneed not be separately defined. However, if we use the member in a context in which the value cannot be substituted, then there must be a definition for that member.

For example, if the only use we make of periodis to define the dimension of `daily_tbl`, there is no need to define period outside of Account. However, if we omit the definition, it is possible that even seemingly trivial changes to the program might cause the program to fail to compile because of the missing definition. For example, if we pass `Account::period` to a function that takes a const int&, then periodmust be defined.

If an initializer is provided inside the class, the member’s definition must not specify an initial value:
Click hereto view code image
// definitionof a static member with no initializer
constexpr int Account::period; // initializer provided in the class
definition

> 最佳实践：Even if a `const static` data member is initialized in the class body, that member ordinarily should be defined outside the class definition.

##### static Members Can Be Used in Ways Ordinary Members Can’t

As we’ve seen, static members exist independently of any other object. As a result, they can be used in ways that would be illegal for nonstatic data members. As one example, a static data member can have incomplete type (§ 7.3.3). In particular, a static data member can have the same type as the class type of which it is a member. A non static data member is restricted to being declared as a pointer or a reference to an object of its class:

```cpp
    class Bar{
    public:
    //...
    private:
        static Bar mem1; // ok: static member can have incomplete type
        Bar* mem2;  // ok: pointer member can have incomplete type
        Bar mem3;  // error: data members must have complete type
    };
```

Another difference between static and ordinary members is that we can use a static member as a default argument (§ 6.5.1):

```cpp
    class Screen{
    public:
        // bkground refers to the static member
        // declared later in the class definition
        Screen&clear(char = bkground);
    private:
    	static const char bkground;
    };
```

A non static data member may not be used as a default argument because its value is part of the object of which it is a member. Using a non static data member as a default argument provides no object from which to obtain the member’s value and so is an error.















