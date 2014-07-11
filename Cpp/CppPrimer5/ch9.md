[toc]

## 9 顺序容器

> 本章建立在第三章（§ 3.2, § 3.3, § 3.4）的基础之上。

### 9.1 顺序容器概述

顺序容器（见下表）都提供到元素的快速的顺序访问。它们的性能相对于以下方面有差异：1）向元素添加删除元素的开销；2）对容器内元素采取非顺序访问的开销。

- `vector`：大小灵活的数组。支持快速随机访问。在最后之外的位置上插入删除可能会慢。
- `deque`：双端队列。支持快速随机访问。在最前和最后的快速插入删除。
- `list`：双向链表。只支持双向顺序访问。在任何地方都可以快速插入删除。
- `forward_list`：单向链表。只支持一个方向上的顺序访问。在任何地方都可以快速插入删除。
- `array`：固定大小的数组。支持快速随机访问。**不能添加和删除元素。**
- `string`：一个特殊容器，类似于`vector`，容纳字符。快速随机访问。在最后快速插入删除。

除`array`是固定容器外，其他容器提供高效灵活的额内存管理。可以添加移除元素，改变容器大小。容器选择的存储元素的策略对这些操作的性能可能有显著的影响。有时这些策略甚至决定某个容器是否提供特定操作。

例如，`string`和`vector`在连续的内存中存储元素。因为元素连续，因此从索引计算元素地址是快速的。但在容器中部添加删除元素要花时间：目标元素之后的元素要被移动来维持连续性。有时，添加元素需要分配额外空间，此时所有元素要被移动到新空间。

`list`和`forward_list`支持在容器任意位置快速添加删除元素。但不支持随机访问元素。访问元素只能通过迭代。而且这些容器与`vector`, `deque`, `array`相比，内容从开销较大（substantial）。

`deque`支持随机访问。在`deque`中部添加删一般是昂贵的。但在两端添加删除元素都是快速的。

新标准引入了`forward_list`和`array`。`array`相对于内建数组更安全、更易使用。但它仍是固定大小的。因此不支持调整容器大小。

`forward_list`目标是媲美最好的、手写的、单向列表。因此`forward_list`没有`size`方法，因为相对于手写的链表，存储和计算大小有额外开销。但其他容器保证`size`是一个快速的常量时间的操作。

> For reasons we’ll explain in § 13.6(p. 531), the new library containers are dramatically faster than in previous releases. The library containers almost certainly perform as well as (and usually better than) even the most carefully crafted alternatives. 现代C++程序应该使用库容器，而不是基本结构，如数组。

#### 应该使用哪个顺序容器？

> 多数情况下应该使用`vector`，除非有却有原因使用其他容器。

选择容器的经验法则：

- 多数情况下应该使用`vector`，除非有却有原因使用其他容器。
- 如果程序有大量小的元素且空间开销很重要，不要使用`list`或`forward_list`。
- 如果需要随机访问，选择`vector`或`deque`。
- 如果需要在中部增删元素，选择`list`或`forward_list`
- 如果仅是在前后增删，不需要在中部，选择`deque`
- 如果仅在读入时会在中间插入，后续却需要随机访问。则输入阶段使用`list`，输入完成后将`list`拷贝到`vector`。

如果既需要随机访问又需要在中间插入删除，则决定取决于访问`list`或`forward_list`中元素，或在`vector`或`deque`中增删的相对开销。一般需要实测。

> 最佳实践：如果不确定使用哪个容器。则尽量选`vector`和`list`都有的操作：用迭代，尽量避免随机访问。

### 9.2. 容器库概述

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

- `c1 = c2`:将c1中的元素替换成c2
- `c1 = {a,b,c...}`：替换c1。不适用于`array`
- `a.swap(b)`：交互a和b中的元素
- `swap(a, b)`：与`a.swap(b)`等价

大小：

- `c.size()`：元素个数（不能用于`forward_list`）
- `c.max_size()`：容器能容纳的最大元素数
- `c.empty()`：容器是否为空

添加删除元素（不能用于`array`）：

- `c.insert(args)`
- `c.emplace(inits)`：使用ints构造一个元素
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
	list<Sales_data>  // list that holds Sales_data objects
	deque<double>  // deque that holds doubles
```

#### 容器能容纳的类型的限制

顺序容器基本能容纳所有类型。甚至元素本身可以又是容器。

```cpp
	vector<vector<string>> lines;  // vector of vectors
```

> 注意：之前的编译器可能要求尖括号之间有空格：`vector<vector<string> >`。

有时元素需要满足一些要求。例如有的顺序容器的构造器取一个大小，此时要用到元素的默认构造器。而有些类没有默认构造器。We can define a container that holds objects of such types, but we cannot construct such containers using only an element count:

```cpp
    // assume noDefault is a type without a default constructor
    vector<noDefault> v1(10, init); // 可以：提供了初始化元素
    vector<noDefault> v2(10); // 不可以！必须提供一个初始化元素
```

#### 9.2.1. 迭代器

迭代器也提供公共接口。例如所有库容器的迭代器都实现了increment operator，移到下一个元素。

##### Iterator Ranges

迭代器范围由两个迭代器划定。一个指向第一个元素，一个指向最后一个元素后面。库使用的闭开范围`[begin, end)`。`end`可以等于`begin`但不能再超前了。编译器不做此类检查，需要我们自己保证。

```cpp
	while (begin !=end) {
		*begin = val;
		++begin;
	}
```

#### 9.2.2. 容器的类型成员

容器定义了一些类型，如`size_type`、`iterator`和`const_iterator`。

一些容器提供反向迭代器，`++`移向前一个元素。

The remaining type aliases let us use the type of the elements stored in a container without knowing what that type is. If we need the element type, we refer to the container’s value_type. If we need a reference to that type, we use referenceor const_reference. These element-related type aliases are most useful in generic programs, which we’ll cover in Chapter 16.

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

不带`c`的函数实际是重载的。例如有两个版本的`begin`。一个是常量成员，返回容器的`const_iterator`。另一个是非常量成员，返回`iterator`。`rbegin`, `end`, `rend`三个函数也有两个版本。When we call one of these members on a nonconstobject, we get the version that returns iterator. We get a const version of the iterators only when we call these functions on a const object. As with pointers and references to const, we can convert a plain
`iterator` to the corresponding `const_iterator`, but not vice versa.

The `c` versions were introduced by the new standard to support using auto with begin and end functions (§ 2.5.2, p. 68). In the past, we had no choice but to say which type of iterator we want:

```cpp
    // type is explicitly specified
    list<string>::iterator it5 = a.begin();
    list<string>::const_iterator it6 = a.begin();
    // iterator or const_iterator depending on a's type of a
    auto it7 = a.begin();  // const_iterator only if a is const
    auto it8 = a.cbegin(); // it8 is const_iterator
```

When we use auto with begin or end, the iterator type we get depends on the container type. How we intend to use the iterator is irrelevant. The cversions let us get a `const_iterator` regardless of the type of the container.

> 最佳实践：当不需要写访问时，使用`cbegin`和`cend`。

#### 9.2.4. 定义和初始化一个容器

每个容器类型都定义了一个默认构造器。默认构造器创建一个空容器（`array`除外）。还有一个构造器指定容器大小和初始值（`array`除外）。

##### 通过拷贝另一个容器初始化

可以拷贝整个容器，或一个范围（`array`除外）。

To create a container as a copy of another container, the container and element types must match. When we pass iterators, there is no requirement that the container types be identical. 新旧容器的元素类型可以不同，只要能转化。

```cpp
    // each container has three elements, initialized from the given initializers
    list<string> authors = {"Milton", "Shakespeare", "Austen"};
    vector<const char*> articles = {"a", "an", "the"};
    list<string> list2(authors);  // ok: types match
    deque<string> authList(authors); // 错误：容器类型不匹配
    vector<string> words(articles);  // 错误：元素类型不匹配
    // ok: converts const char* elements to string
    forward_list<string> words(articles.begin(), articles.end());
```

> When we initialize a container as a copy of another container, the container type and element type of both containers must be identical.

##### 列表初始化

新保准允许用列表初始化容器

```cpp
	list<string> authors = {"Milton", "Shakespeare", "Austen"};
	vector<const char*> articles = {"a", "an", "the"};
```

##### 容器容量相关的构造器（顺序容器）

顺序容器（`array`）除外，可以通过一个大小和一个可选的初始化元素初始化。If we do not supply an element initializer, the library creates a value-initialized one for us § 3.3.1:

```cpp
    vector<int> ivec(10, -1); // 10个元素，每个初始化为-1
    list<string> svec(10, "hi!");
    forward_list<int>ivec(10); // ten elements, each initialized to 0
    deque<string> svec(10); // ten elements, each an empty string
```

如果元素没有默认构造器，则只能指定初始化元素。

> The constructors that take a size are valid onlyfor sequential containers; they are not supported for the associative containers.

##### 库array的大小固定

正如内建的数组的大小是其类型的一部分，库`array`的大小也是其类型的一份不。定义`array`时除了要指定元素类型，也要指定容器大小：

```cpp
	array<int, 42>  // type is: array that holds 42 ints
	array<string, 10> // type is: array that holds 10 strings
```

Because the size is part of the array’s type, arraydoes not support the normal container constructors. Those constructors, implicitly or explicitly, determine the size of the container. It would be redundant (at best) and error-prone to allow users to pass a size argument to an `array` constructor.

The fixed-size nature of arrays also affects the behavior of the constructors that array does define. 与其他容器不同的是，默认构造器构造的`array`不是空的：It has as many elements as its size. These elements are **default** initialized (§2.2.1, p. 43) just as are elements in a built-in array (§ 3.5.1, p. 114). 若想列表初始化数组，初始列表数量必须小于等于`array`的大小。If there are fewer initializers than the size of the array, the
initializers are used for the first elements and any remaining elements are **value** initialized (§ 3.3.1, p. 98). In both cases, if the element type is a class type, the class must have a default constructor in order to permit value initialization:

```cpp
    array<int, 10> ia1;  // 10个默认初始化的整数
    array<int, 10> ia2 = {0,1,2,3,4,5,6,7,8,9}; // 列表初始化
    array<int, 10> ia3 = {42}; // ia3[0] is 42, 剩下的是0
```

不能拷贝和赋值内建数组。但`array`没有此限制：

```cpp
	int digs[10] = {0,1,2,3,4,5,6,7,8,9};
	int cpy[10] = digs; // error: no copy or assignment for built-in arrays
	array<int, 10> digits = {0,1,2,3,4,5,6,7,8,9};
	array<int, 10> copy = digits; // ok: so long as array types match
```

As with any container, the initializer must have the same type as the container we are creating. For arrays, the element type and the size must be the same, because the size of an array is part of its type.

#### 9.2.5. 赋值和swap

赋值操作将左面的容器整个替换成右边的：

```cpp
	c1 = c2;
    c1 = {a,b,c}; // after the assignment c1 has size 3
```


`swap(a, b)`或`a.swap(b)`。交换往往比拷贝快得多。

`seq.assign(b, e)`：替换`seq`中的元素，b和e是迭代器。迭代器不能引用`seq`中的元素。
`seq.assign(il)`：用列表初始化seq。
`seq.assign(n, t)`：n个t元素。

assign不能用于`array`或associative容器。

如果左右容器原来的大小不同，赋值后左容器大小等于右容器。

与内建数组不同，库`array`支持赋值。运算符左右两侧必须是相同类型：

```cpp
	array<int, 10> a1 = {0,1,2,3,4,5,6,7,8,9};
	array<int, 10> a2 = {0}; // elements all have value 0
	a1 = a2; // replaces elements in a1
	a2 = {0}; // error: cannot assign to an array from a braced list
```

因为右边操作数的大小可能与左边不同，`array`不支持`assign`，也不允许用列表赋值。

#### 使用`assign`（只有顺序容器能用）

赋值运算符要求左右两个操作数类型相同。It copies all the elements from the right-hand operand into the left-hand operand. The sequential containers (except `array`) also define a member named `assign` that lets us assign from a different but compatible type, or assign from a subsequence of a container. The `assign` operation replaces all the elements in the left-hand container with (copies of) the elements specified by its arguments. For example, we can use assign to assign a range of `char*` values from a `vector` into a list of string:

```cpp
    list<string> names;
    vector<const char*> oldstyle;
    names = oldstyle; // 错误：容器类型不匹配
    // ok: can convert from  const char* to string
    names.assign(oldstyle.cbegin(), oldstyle.cend());
```

> 警告：Because the existing elements are replaced, the iterators passed to  `assign` must not refer to the container on which `assign` is called.

第二个版本的`assign`，重复指定数量的元素：

```cpp
    // equivalent to slist1.clear();
    // followed by slist1.insert(slist1.begin(), 10, "Hiya!");
    list<string> slist1(1); // one element, which is the empty string
    slist1.assign(10, "Hiya!"); // ten elements; each one is Hiya  !
```


#### 使用`swap`

The `swap` operation exchanges the contents of two containers of the same type. After the call to  swap, the elements in the two containers are interchanged:

```cpp
    vector<string> svec1(10); // vector with ten elements
    vector<string> svec2(24); // vector with 24 elements
    swap(svec1, svec2);
```

With the exception of `array`s, swapping two containers is guaranteed to be fast—the elements themselves are not swapped; 交换的只是内部数据结构。Excepting `array`, `swap`不会拷贝、删除元素，保证在常量时间完成。

The fact that elements are not moved means that, with the exception of `string`, iterators, references, and pointers into the containers are not invalidated. They refer to the same elements as they did before the swap. However, after the swap, those elements are in a different container. For example, had `iter` denoted the `string` at position `svec1[3]` before the swap, it will denote the element at position `svec2[3]` after the swap. Differently from the containers, a call to `swap` on a `string` may invalidate iterators, references and pointers.

Unlike how `swap` behaves for the other containers, swapping two arrays does exchange the elements. As a result, swapping two `array`s requires time proportional to the number of elements in the array.

After the swap, pointers, references, and iterators remain bound to the same element they denoted before the swap. Of course, the value of that element has been
swapped with the corresponding element in the other array.
 
In the new library, the containers offer both a member and nonmember version of
swap. Earlier versions of the library defined only the member version of swap. The
nonmember  swap is of most importance in generic programs. As a matter of habit, it
is best to use the nonmember version of  swap.










