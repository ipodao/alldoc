[toc]

## 9 顺序容器

> 本章建立在第三章（§ 3.2, § 3.3, § 3.4）的基础之上。

### 9.1 顺序容器概述

顺序容器（见下表）都提供到元素的快速的顺序访问。它们的性能在以下方面有差异：1）向元素添加删除元素的开销；2）非顺序访问容器内元素的开销。

- `vector`：大小灵活的数组。支持快速随机访问。在最后之外的位置上插入删除可能会慢。
- `deque`：双端队列。支持快速随机访问。在最前和最后的快速插入删除。
- `list`：双向链表。只支持双向顺序访问。在任何地方都可以快速插入删除。
- `forward_list`：单向链表。只支持一个方向上的顺序访问。在任何地方都可以快速插入删除。
- `array`：固定大小的数组。支持快速随机访问。**不能添加和删除元素。**
- `string`：一个特殊容器，类似于`vector`，容纳字符。快速随机访问。在最后快速插入删除。

除`array`是固定容器外，其他容器提供高效灵活的内存管理：可以添加移除元素，改变容器大小。容器存储元素的策略对这些操作的性能有显著的影响。有时这些策略甚至决定某个容器是否提供特定操作。

例如，`string`和`vector`在连续的内存中存储元素。因为元素连续，因此从索引计算元素地址是快速的。但在容器中部添加删除元素要花时间：目标元素之后的元素要被移动来维持连续性。有时，添加元素需要分配额外空间，此时所有元素要被移动到新空间。

`list`和`forward_list`支持在容器任意位置快速添加删除元素。但不支持随机访问元素。访问元素只能通过迭代。而且这些容器与`vector`, `deque`, `array`相比，内存开销较大（substantial）。

`deque`支持随机访问。在`deque`中部添加删一般是昂贵的。但在两端添加删除元素都是快速的。

新标准引入了`forward_list`和`array`。`array`比内建数组更安全、更易使用。但它仍是固定大小的。因此不支持调整容器大小。

`forward_list`目标是媲美最好的、手写的、单向列表。因此`forward_list`没有`size`方法，因为相对于手写的链表，存储和计算大小会带来额外开销。但其他容器`size`是一个快速的常量时间的操作。

> 新的库容器比之前的显著的快（原因见§ 13.6）。The library containers almost certainly perform as well as (and usually better than) even the most carefully crafted alternatives. **现代C++程序应该使用库容器**，而不是基本结构，如数组。

#### 应该使用哪个顺序容器？

> 多数情况下应该使用`vector`，除非有却有原因使用其他容器。

选择容器的经验法则：

- 多数情况下应该使用`vector`，除非有却有原因使用其他容器。
- 如果程序有大量小的元素且空间开销很重要，不要使用`list`或`forward_list`。
- 如果需要随机访问，选择`vector`或`deque`。
- 如果需要在中部增删元素，选择`list`或`forward_list`
- 如果仅是在前后增删，不需要在中部，选择`deque`
- 如果仅在读入时会在中间插入，后续却需要随机访问。则输入阶段使用`list`，输入完成后将`list`拷贝到`vector`。

如果既需要随机访问又需要在中间插入删除，则决定取决于访问`list`或`forward_list`中的元素，或在`vector`或`deque`中增删的相对开销。一般需要实测。

> 最佳实践：如果不确定使用哪个容器。则尽量选`vector`和`list`都有的操作：用迭代，尽量避免随机访问。

### 9.2. 库容器概述

有一些操作是所有容器共有的：

类型别名：

- `iterator`：容器的迭代器的类型
- `const_iterator`：只读迭代器类型，不能修改元素
- `size_type`：无符号整数，确保能表示容器最大大小
- `difference_type`：有符号整数，确保能容纳两个迭代器的差
- `value_type`：元素类型
- `reference`：元素的左值类型；等价于`value_type&`
- `const_reference`：元素的常量左值类型（`const value_type&`）

构造器：

- `C c`：默认构造，空容器
- `C c1(c2)`：拷贝c2到c1
- `C c(b, e)`：范围拷贝，范围由`b`和`e`表述（不适用与`array`）
- `C c{a,b,c...}`：列表初始化

赋值与swap：

- `c1 = c2`：将c1中的元素替换成c2
- `c1 = {a,b,c...}`：替换c1。不适用于`array`
- `a.swap(b)`：交互a和b中的元素
- `swap(a, b)`：与`a.swap(b)`等价

大小：

- `c.size()`：元素个数（不能用于`forward_list`）
- `c.max_size()`：容器能容纳的最大元素数
- `c.empty()`：容器是否为空

添加删除元素（不能用于`array`）：

- `c.insert(args)`
- `c.emplace(inits)`：使用 inits 构造一个元素
- `c.erase(args)`：
- `c.clear()`：

迭代器：

- `c.begin()` `c.end()`
- `c.cbegin()` `c.cend()`：返回`const_iterator`

可反向的容器（不适用于`forward_list`）：

- `reverse_iterator`：反向迭代器
- `const_reverse_iterator`
- `c.rbegin()` `c.rend()`
- `c.crbegin()` `c.crend()`：返回`const_reverse_iterator`

每个容器定义在自己的头文件中。头文件名一般与类型名相同。如`deque`在头文件`deque`中。容器是类模板。

```cpp
	list<Sales_data>
	deque<double>
```

#### 容器能容纳的类型的限制

顺序容器基本能容纳所有类型。甚至元素本身可以又是容器。

```cpp
	vector<vector<string>> lines;  // vector of vectors
```

> 注意：之前的编译器可能要求尖括号之间有空格：`vector<vector<string> >`。

有时容器内元素需要满足一些要求。例如有的顺序容器的构造器取一个大小做参数，此时要用到元素的默认构造器。但有些类没有默认构造器。因此不能用此种构造器构造容器：

```cpp
    // assume noDefault is a type without a default constructor
    vector<noDefault> v1(10, init); // 可以：提供了初始化元素
    vector<noDefault> v2(10); // 不可以！必须提供一个初始化元素
```

#### 9.2.1. 迭代器

迭代器也提供公共接口。例如所有库容器的迭代器都实现了increment operator，移到下一个元素。

##### 迭代器范围

迭代器范围由两个迭代器划定。一个指向第一个元素，一个指向最后一个元素后面。库使用闭开范围`[begin, end)`。`end`可以等于`begin`但不能再超前了。编译器不做此类检查，需要我们自己保证。

```cpp
	while (begin != end) {
		*begin = val;
		++begin;
	}
```

#### 9.2.2. 容器的类型成员

容器定义了一些类型，如`size_type`、`iterator`和`const_iterator`。

一些容器提供反向迭代器，`++`反向移向一个元素。

使用这些类型时，前面要加类限定：

```cpp
	list<string>::iterator iter;
	vector<int>::difference_type count;
```

#### 9.2.3. `begin`和`end`成员函数

有几个版本。带`r`的返回反向迭代器。带`c`的返回常量版本：

```cpp
	list<string> a= {"Milton", "Shakespeare", "Austen"};
	auto it1 = a.begin(); // list<string>::iterator
	auto it2 = a.rbegin(); // list<string>::reverse_iterator
	auto it3 = a.cbegin(); // list<string>::const_iterator
	auto it4 = a.crbegin(); // list<string>::const_reverse_iterator
```

不带`c`的函数有重载版本。例如有两个版本的`begin`。一个是常量成员函数，返回容器的`const_iterator`。另一个是非常量成员函数，返回`iterator`。`rbegin`, `end`, `rend`三个函数也有两个版本。对非常量对象调用这些方法，返回`iterator`。对常量对象调用这些方法返回常量版本的迭代器。与指向常量的指针和引用一样，可以将普通`iterator`转换为响应的`const_iterator`，但反之不然。

新标准引入带`c`版本的目的是，让`auto`可用。

```cpp
    // 显式指定类型
    list<string>::iterator it5 = a.begin();
    list<string>::const_iterator it6 = a.begin();
    // iterator or const_iterator depending on a's type of a
    auto it7 = a.begin(); // const_iterator only if a is const
    auto it8 = a.cbegin(); // it8 is const_iterator
```

`auto`和`begin` `end`连用，迭代器的类型取决于容器的类型。与我们打算如何使用迭代器无关。The `c` versions let us get a `const_iterator` regardless of the type of the container.

> 最佳实践：当不需要写访问时，使用`cbegin`和`cend`。

#### 9.2.4. 定义和初始化一个容器

每个容器类型都定义了一个默认构造器。默认构造器创建一个空容器（`array`除外）。还有一个构造器指定容器大小和初始值（`array`除外）。

##### 通过拷贝另一个容器初始化

可以拷贝整个容器，或一个范围（`array`除外）。

通过拷贝另一个容器创建新容器，容器和元素类型需要一致。但若传入迭代器，则容器类型可以不相同。新旧容器的元素类型也可以不同，只要能转化。

```cpp
	list<string> authors = {"Milton", "Shakespeare", "Austen"};
    vector<const char*> articles = {"a", "an", "the"};
    list<string> list2(authors); // ok: types match
    deque<string> authList(authors); // 错误：容器类型不匹配
    vector<string> words(articles); // 错误：元素类型不匹配
    // 可以：const char*可以转换为string
    forward_list<string> words(articles.begin(), articles.end());
```

##### 列表初始化

新标准允许用列表初始化容器

```cpp
	list<string> authors = {"Milton", "Shakespeare", "Austen"};
	vector<const char*> articles = {"a", "an", "the"};
```

##### 容器容量相关的构造器（顺序容器）

顺序容器（`array`）除外，可以通过一个大小和一个可选的初始化元素初始化。若不提供元素初始化器，库使用值初始化（§ 3.3.1）：

```cpp
    vector<int> ivec(10, -1); // 10个元素，每个初始化为-1
    list<string> svec(10, "hi!");
    forward_list<int> ivec(10); // 10个元素，初始化为0
    deque<string> svec(10); // 10个元素，初始化为空串
```

如果元素没有默认构造器，则只能指定初始化元素。

> The constructors that take a size are valid only for sequential containers; they are not supported for the **associative containers**.

##### 库`array`的大小固定

正如内建的数组的大小是其类型的一部分，库`array`的大小也是其类型的一部分。定义`array`时除了要指定元素类型，也要指定容器大小：

```cpp
	array<int, 42>  // type is: array that holds 42 ints
	array<string, 10> // type is: array that holds 10 strings
```

因为要指定`array`大小，`array`不支持常见容器的构造器。

与其他容器不同的是，默认构造器构造的`array`不是空的：`array`定义了多大就有多少个呀un苏。这些元素是默认初始化的（§2.2.1）（与内建数组的情况一样，§ 3.5.1）。若想列表初始化数组，初始列表数量必须小于等于`array`的大小。当初始化器少于`array`大小，后面缺少的元素被值初始化（§ 3.3.1）。若元素类型是类类型，则类必须有默认构造器才能被值初始化：

```cpp
    array<int, 10> ia1; // 10个默认初始化的整数
    array<int, 10> ia2 = {0,1,2,3,4,5,6,7,8,9}; // 列表初始化
    array<int, 10> ia3 = {42}; // ia3[0] is 42, 剩下的是0
```

不能拷贝和赋值内建数组。但`array`没有此限制：

```cpp
	int digs[10] = {0,1,2,3,4,5,6,7,8,9};
	int cpy[10] = digs; // 错误：内建数组不支持拷贝或赋值
	array<int, 10> digits = {0,1,2,3,4,5,6,7,8,9};
	array<int, 10> copy = digits; // ok: so long as array types match
```

As with any container, the initializer must have the same type as the container we are creating. For arrays, the element type and the size must be the same, because the size of an array is part of its type.

#### 9.2.5. 赋值和swap

赋值操作将左面的容器整个替换成右边的：

```cpp
	c1 = c2;
    c1 = {a, b, c}; // after the assignment c1 has size 3
```

`swap(a, b)`或`a.swap(b)`。交换往往比拷贝快得多。

`seq.assign(b, e)`：替换`seq`中的元素，b和e是迭代器。迭代器不能引用`seq`中的元素。
`seq.assign(il)`：用列表初始化seq。
`seq.assign(n, t)`：n个t元素。

`assign`不能用于`array`或associative容器。

如果左右容器原来的大小不同，**赋值后左容器大小等于右容器**。

与内建数组不同，库`array`支持赋值。运算符左右两侧必须是相同类型：

```cpp
	array<int, 10> a1 = {0,1,2,3,4,5,6,7,8,9};
	array<int, 10> a2 = {0}; // elements all have value 0
	a1 = a2; // replaces elements in a1
	a2 = {0}; // error: cannot assign to an array from a braced list
```

**因为右边操作数的大小可能与左边不同**，`array`不支持`assign`，也不允许用列表赋值。

#### 使用`assign`（只有顺序容器能用）

赋值运算符要求左右两个操作数类型相同。它将右侧操作数的所有元素拷贝到左侧。顺序容器（除了`array`）还有一个成员函数`assign`，允许不同但兼容的类型的赋值，or assign from a subsequence of a container。The `assign` operation replaces all the elements in the left-hand container with (copies of) the elements specified by its arguments. 例如，可以利用`assign`，把`vector<const char*>`中的一块范围赋给`string`：

```cpp
    list<string> names;
    vector<const char*> oldstyle;
    names = oldstyle; // 错误：容器类型不匹配
    // ok: can convert from  const char* to string
    names.assign(oldstyle.cbegin(), oldstyle.cend());
```

> 警告：Because the existing elements are replaced, the iterators passed to `assign` must not refer to the container on which `assign` is called.

第二个版本的`assign`，指定重复数量的元素：

```cpp
    // equivalent to slist1.clear();
    // followed by slist1.insert(slist1.begin(), 10, "Hiya!");
    list<string> slist1(1); // one element, which is the empty string
    slist1.assign(10, "Hiya!"); // ten elements; each one is Hiya  !
```


#### 使用 `swap`

`swap`交换两个类型相同的容器的内容。

```cpp
    vector<string> svec1(10); // vector with ten elements
    vector<string> svec2(24); // vector with 24 elements
    swap(svec1, svec2);
```

除`array`外，交换操作保证是快速的——元素本身不会被交换；交换的只是内部数据结构。除`array`外，`swap`不会拷贝、删除元素，保证在常量时间完成。

元素不会被移动意味着，（除`string`外），迭代器、指向容器的引用和指针不再有效。它们仍旧指向`swap`之前的元素。但`swap`后，这些元素到了另一个容器内。For example, had `iter` denoted the `string` at position `svec1[3]` before the swap, it will denote the element at position `svec2[3]` after the swap. Differently from the containers, a call to `swap` on a `string` may invalidate iterators, references and pointers.

与其他容器不同，`swap`两个`array`会交换元素。因此交换`array`的时间正比于元素数目。`swap`后，指针、引用、迭代器仍旧指向`swap`之前它们指向的元素。Of course, the value of that element has been swapped with the corresponding element in the other array.

新标准有两个交互方法，分别是成员方法和非成员方法。之前只有成员版本。The nonmember  swap is of most importance in generic programs. 就习惯来数哦，最好使用非成员版本的`swap`。

#### 9.2.6. 容器大小操作

容器提供三个大小相关操作。`size`返回容器内元素数量；`empty`当容器为空是返回true；`max_size` returns a number that is greater than or equal to the number of elements a container of that type can contain。

`forward_list`只提供`max_size`和`empty`，不提供`size`。

#### 9.2.7. 关系运算符

所有容器都支持`==`和`!=`；所有容器（除了unordered associative containers）支持关系运算符（`>`, `>=`, `<`, `<=`）。两个运算数必须是相同容器且元素类型相同。即`vector<int>`不能与`vector<double>`比较。

Comparing two containers performs a pairwise comparison of the elements:

- If both containers are the same size and all the elements are equal, then the two containers are equal; otherwise, they are unequal.
- If the containers have different sizes but every element of the smaller one is equal to the corresponding element of the larger one, then the smaller one is less than the other.
- If neither container is an initial subsequence of the other, then the comparison depends on comparing the first unequal elements.

The following examples illustrate how these operators work:

```cpp
    vector<int> v1= { 1, 3, 5, 7, 9, 12 };
    vector<int> v2 = { 1, 3, 9 };
    vector<int> v3 = { 1, 3, 5, 7 };
    vector<int> v4 = { 1, 3, 5, 7, 9, 12 };
    v1 < v2 // true; v1 and v2 differ at element [2]: v1[2] is less than v2[2]
    v1 < v3 // false; all elements are equal, but v3 has fewer of them;
    v1 == v4 // true; each element is equal and v1 and v4 have the same size()
    v1 == v2 // false; v2 has fewer elements than v1
```

##### 关系运算符使用元素的关系运算符

> 要使用关系运算符比较容器，元素类型必须定义有比较运算符

For example, the Sales_data type that we defined in Chapter 7 does not define either the `==` or the `<` operation. Therefore, we cannot compare two containers that hold `Sales_data` elements:

```cpp
	vector<Sales_data> storeA,storeB;
	if (storeA < storeB) // error: Sales_data has no less-than operator
```

### 9.3. 顺序容器操作

顺序容器与关联（associative）容器的区别在于如何组织元素。此区别应元素的存储、访问、添加、删除。前面讲了所有容器共有的操作，接下来是顺序容器特有的操作。

#### 9.3.1. 向顺序容器添加元素

Excepting `array`, all of the library containers provide flexible memory management. 可以在运行时动态增删元素，改变容器大小。

下面的操作不适用与`array`；`forward_list`有特殊版本的`insert`和`emplace`；`push_back`和`emplace_back`对`forward_list`无效。`push_front`和`emplace_front`对`vector`和`string`无效：

- `c.push_back(t)`，`c.emplace_back(args)`：用t创建一个元素，或通过args构造一个元素，插入c的尾部。返回void。
- `c.push_front(t)`，`c.emplace_front(args)`：用t创建一个元素，或通过args构造一个元素，插入c的头部。返回void。
- `c.insert(p, t)`，`e.emplace(p, args)`：用t创建一个元素，或通过args构造一个元素；在迭代器p之前插入，返回指向新元素的迭代器。
- `c.insert(p, n, t)`：在迭代器p之前插入元素t，插入n次。返回指向插入的第一个的迭代器。如果n为零，返回p。
- `c.insert(p, b, e)`：在迭代器p之前插入一个范围。b和e不能指向c中的元素。返回指向插入的第一个的迭代器。如果范围为空，返回p。
- `c.insert(p, il)`：在迭代器p之前插入。il是大括号包围的一组元素。返回指向插入的第一个的迭代器。如果范围为空，返回p。

> 警告：向`vector`, `string`, `deque`插入元素可能使已存在的迭代器、指向容器的引用和指针失效。

##### 使用`push_back`

除`array`和`forward_list`外，每个容器，包括`string`都支持`push_back`。

```cpp
    // read from standard input, putting each word onto the end of container
    string word;
    while (cin >> word)
    container.push_back(word); // container可以是vector
```

新插入的元素是`word`的一个**拷贝**！！
Because string is just a container of characters, we can use push_backto add characters to the end of the string:

```cpp
    void pluralize(size_tcnt, string &word)
    {
    	if(cnt > 1)
    	word.push_back('s'); // same as word += 's'
    }
```

> 核心概念：容器元素是被拷贝的！！
> 使用一个对象初始化一个容器，或将对象插入容器，放入容器的是对象值的拷贝，不是对象自身。就像向非引用形参传递参数一样，容器内的元素与初始对象没有关系。

##### 使用`push_front`

`list`, `forward_list`, `deque`支持`push_front`。该操作在容器开头插入元素：

```cpp
	list<int> ilist;
	// add elements to the start of ilist
	for (size_t ix = 0; ix != 4; ++ix)
		ilist.push_front(ix);
```

Note that `deque`, which like `vector` offers fast random access to its elements, provides the `push_front` member even though `vector` does not. A `deque` guarantees constant-time insert and delete of elements at the beginning and end of the container. As with vector, inserting elements other than at the front or back of a dequeis a potentially expensive operation.

##### 在容器特定位置插入元素

`insert`允许我们在容器的任一点插入零到多个元素。支持`insert`的有`vector`, `deque`, `list`, `string`。`forward_list` provides specialized versions of these members that we’ll cover in §9.3.4.

Each of the `insert` functions takes an iterator as its first argument. 元素在迭代器指向的元素之前插入新元素。For example, this statement

```cpp
	slist.insert(iter, "Hello!"); // insert "Hello!" just before iter
```

对于不支持`push_front`的容器，可以利用`insert`在头部插入新元素。

```cpp
    vector<string> svec;
    list<string> slist;
    // equivalent to calling slist.push_front("Hello!");
    slist.insert(slist.begin(), "Hello!");
    // no push_front on vector but we can insert before begin()
    // warning: inserting anywhere but at the end of a vector might be slow
    svec.insert(svec.begin(), "Hello!");
```

##### 插入一块元素

The version that takes an element count and a value adds the specified number of identical elements before the given position:

```cpp
	svec.insert(svec.end(), 10, "Anna");
```

This code inserts ten elements at the end of svecand initializes each of those elements to the string "Anna".

The versions of insert that take a pair of iterators or an initializer list insert the elements from the given range before the given position:

```cpp
	vector<string> v = {"quasi", "simba", "frollo", "scar"};
	// insert the last two elements of v at the beginning of slist
	slist.insert(slist.begin(), v.end() - 2, v.end());
	slist.insert(slist.end(), {"these", "words", "will",
    	"go","at", "the", "end"});
	// run-time error: iterators denoting the range to copy from
	// must not refer to the same container as the one we are changing
	slist.insert(slist.begin(), slist.begin(), slist.end());
```

When we pass a pair of iterators, those iterators may not refer to the same container as the one to which we are adding elements. Under the new standard, the versions of insert that take a count or a range return an iterator to the first element that was inserted. (In prior versions of the library, these operations returned void.) If the range is empty, no elements are inserted, and the operation returns its first parameter.

##### 使用`insert`的返回值

We can use the value returned by insert to repeatedly insert elements at a specified position in the container:

```cpp
    list<string> 1st;
    auto iter = 1st.begin();
    while (cin >> word)
    	iter = 1st.insert(iter, word); // 始终指向当前第一个元素，于是相当于push_front
```

##### Emplace操作

新标准引入了三个新成员函数——`emplace_front`, `emplace`, `emplace_back`——它们构造而不是拷贝元素。

When we call a push or insert member, we pass objects of the element type and those objects are copied into the container. 但调用`emplace`函数时，传入的是构造元素的构造器的实参。For example, assuming c holds `Sales_data`(§ 7.1.4) elements:

```cpp
    // 利用Sales_data的三参数构造器
    c.emplace_back("978-0590353403", 25, 15.99);
    // 错误：there is no version of push_back that takes three arguments
    c.push_back("978-0590353403", 25, 15.99);
    // ok: we create a temporary Sales_data object to pass to push_back
    c.push_back(Sales_data("978-0590353403", 25, 15.99));
```

The arguments to an emplace function vary depending on the element type. The arguments must match a constructor for the element type:

```cpp
    // iter refers to an element in c, which holds Sales_data elements
    c.emplace_back(); // 用Sales_data的默认构造器
    c.emplace(iter, "999-999999999"); // uses Sales_data(string)
    // uses the Sales_data constructor that takes an ISBN, a count, and a price
    c.emplace_front("978-0590353403", 25, 15.99);
```

#### 9.3.2. 访问元素

下面列出了访问顺序容器中元素的方法。

**The access operations are undefined if the container has no elements.**

at和下标运算符只对`string` `vector` `deque` `array`有效。`back`对`forward_list`无效。

- `c.back()`：返回到c中最后一个元素的引用。如果c为空，未定义。
- `c.front()`：返回到c中第一个元素的引用。如果c为空，未定义。
- `c[n]`：n是无符号整数。返回到元素的引用。如果`n >= c.size()`，未定义。
- `c.at(n)`：返回下标指定的元素的引用。入股下标越界，抛出`out_of_range`异常。

在空容器上调用`front`或`back`，与下标越界一样都是严重错误。

所有的顺序容器，包括`array`都有`front`成员。除`forward_list`之外的容器有`back`成员。

```cpp
    // check that there are elements before dereferencing an iterator or calling front or back
    if (!c.empty()) {
    	// val和val2是第一个元素的拷贝
        auto val = *c.begin(), val2 = c.front();
        // val3和val4是最后一个元素的拷贝
        auto last = c.end();
        auto val3 = *(--last); // can't decrement forward_list iterators
        auto val4 = c.back(); // not supported by forward_list
    }
```

##### 访问成员返回的是引用

访问容器内元素的成员函数返回的是引用。如果容器是常量对象，则返回的是到常量的引用。

```
    if (!c.empty()){
    	c.front() = 42; // assigns 42 to the first element in c
    	auto &v = c.back(); // get a reference to the last element
    	v = 1024; // changes the element in c
	    auto v2 = c.back(); // v2不是引用，是c.back()的一个拷贝
	    v2 = 0; // no change to the element in c
    }
```

与其他情况一样，若使用`auto`存储这些方法的返回值时，若期望得到引用，记得加`&`。

##### 下标与安全随机访问

提供快速随机访问的容器（`string`, `vector`, `deque`, `array`）也提供下标运算符。返回的是引用。由程序自己检查下标越界，下标运算符不会检查。

下标越界是验证的程序错误，但编译器无法检测出。

`at`成员函数与下标类似，但如果越界，将抛出`out_of_range`异常(§ 5.6)：

```cpp
    vector<string> svec; // empty vector
    cout << svec[0]; // run-time error: there are no elements in svec!
    cout << svec.at(0); // throws an out_of_range exception
```

#### 9.3.3. 移除元素

下面的运算会改变容器大小，因此`array`不支持。`forward_list`有一个特殊版本的erase。`pop_back`对`forward_list`无效；`pop_front`对`vector`和`string`无效。

- `c.pop_back()`：移除c中的最后一个元素。如果c为空未定义。返回void。
- `c.pop_front()`：移除c中的第一个元素。如果c为空未定义。返回void。
- `c.erase(p)`：移除迭代器p指定的元素。返回其后元素的迭代器。如果p位于最后一个元素后，结果未定义。
- `c.erase(b, e)`：移除范围。返回被移除的最后一个元素之后元素的迭代器。
- `c.clear()`：移除c的所有元素。返回void。

在开头结尾之外的位置移除元素，使得`deque`的迭代器、引用、指针失效。对于`vector`和`string`，移除点之后的迭代器、引用、指针失效。

> 移除元素的成员函数不会检查参数。需要在移除前确保它们存在。

##### `pop_front`和`pop_back`

`pop_front`对`vector`和`string`无效。`pop_back`对`forward_list`无效。

不能在空容器上删除元素。

##### 在任意位置移除

例子：移除奇数。

```cpp
	list<int> lst = {0,1,2,3,4,5,6,7,8,9};
    auto it = lst.begin();
    while (it != lst.end())
    	if(*it % 2) // if the element is odd
    		it = lst.erase(it); // erase this element
    	else
    		++it;
```

##### 移除多个元素

```cpp
    // delete the range of elements between two iterators
    // returns an iterator to the element just after the last removed element
    elem1 = slist.erase(elem1, elem2); // after the call elem1 == elem2
```

情况操作等价方式：

```cpp
    slist.clear(); // delete all the elements within the container
    slist.erase(slist.begin(), slist.end()); // equivalent
```

#### 9.3.4. （未）`forward_list`的特殊操作

#### 9.3.5. （未）调整容器大小
#### 9.3.6. 操纵容器可能导致迭代器失效

添加删除容器元素可能导致到容器元素的指针、引用、迭代器失效。无效的指针、引用、迭代器不再指向容器内的元素。使用无效的指针、引用、迭代器是严重的编程错误，与使用未经初始化的指针一样。

在向容器添加元素后：

- 对于vector和string，或引起容器重新分配，指针、引用、迭代器失效。若不会导致重新分配，则indirect references to elements before the insertion remain valid; those to elements after the insertion are invalid.
- Iterators, pointers, and references to a deque are invalid if we add elements anywhere but at the front or back. If we add at the front or back, iterators are invalidated, but references and pointers to existing elements are not.
- Iterators, pointers, and references (including the off-the-end and the before-the-beginning iterators) to a list or forward_list remain valid,

It should not be surprising that when we remove elements from a container, iterators, pointers, and references to the removed elements are invalidated. After all, those elements have been destroyed. After we remove an element,
• All other iterators, references, or pointers (including the off-the-end and the
before-the-beginning iterators) to a listor forward_listremain valid.
• All other iterators, references, or pointers to a dequeare invalidated if the
removed elements are anywhere but the front or back. If we remove elements at
the back of the deque, the off-the-end iterator is invalidated but other iterators,
references, and pointers are unaffected; they are also unaffected if we remove
from the front.
• All other iterators, references, or pointers to a vectoror stringremain valid
for elements before the removal point. Note: The off-the-end iterator is always
invalidated when we remove elements.



