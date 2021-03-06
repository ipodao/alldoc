[toc]

## 8. IO 库

C++语言本身不直接处理IO。IO由标准库处理。支持文件、控制台、内存、字符串I/O。

The IO library defines operations to read and write values of the built-in types. In addition, classes, such as `string`, typically define similar IO operations to work on objects of their class type as well.

本章介绍IO库的基础。Later chapters will cover additional capabilities: Chapter 14 will look at how we can write our own input and output operators, and Chapter 17 will cover how to control formatting and how to perform random access on files.

Our programs have already used many IO library facilities. Indeed, we introduced most of these facilities in § 1.2:

- `istream`输入流
- `ostream`输出流
- `cin`，一个`istream`对象，读取标准输入
- `cout`，一个`ostream`对象，写到标准输出
- `cerr`，一个`ostream`对象，写到标准错误
- `>>`运算符，从`istream`对象读取
- `<<`运算符，写到`ostream`流对象
- `getline`函数(§ 3.2.2)，从`istream`读取一行数据到`string`

### 8.1. IO 类

之前介绍的IO类型和对象操纵`char`数据。但应用还可以读写需要宽字符支持的语言。

下表列出各种IO类型。这些类型定义在三个头中：`iostream`定义了基本的类型。`fstream`
定义了读写文件的类型。`sstream`定义了读写内存中的`string`的类型。

- 头`iostream`：`istream` `wistream`读流。`ostream` `wostream`写流。`iostream` `wiostream`读写流。
- 头`fstream`：`ifstream` `wifstream`读文件。`ofstream` `wofstream`写文件。`fstream` `wfstream`读写文件。
- 头`sstream`：`istringstream` `wistringstream`读`string`。`ostringstream` `wostringstream`写`string`。`stringstream` `wstringstream`读写`string`。

为支持宽字符，库定义了一些操纵`wchar_t`的类型和对象。例如`wcin`, `wcout`, `wcerr`。

##### IO类型之间的关系

概念上，设备或字符宽度的差异不影响使用IO操作的方式。例如，不管目标是控制台、磁盘或字符串，不管内容应被放入`char`还是`wchar_t`，都可以使用`>>`读取数据。

库利用**继承**屏蔽上述差异。类`ifstream`和`istringstream`继承自`istream`。例如，我们可以像使用`istream`一样使用`ifstream`或`istringstream`。可以像使用`cin`一样使用这些类型的对象。例如`getline`、`>>`对`ifstream`或`istringstream`都有效。 类似的，`ofstream`和`ostringstream`继承自`ostream`。Therefore, we can use objects of these types in the same ways that we have used `cout`.

> 下面介绍的内容对各种流都有效：普通流、文件流、字符串流；字符和宽字符版本。

#### 8.1.1. 不能拷贝或赋值

不能拷贝或赋值IO类型。于是IO类型也不能做形参或返回值。但可以传递或返回引用。读写IO对象改变其状态，因此引用不能是常量的。

```cpp
    ofstream out1, out2;
    out1 = out2; // 错误，不能赋值！
    ofstream print(ofstream); // error: can't initialize the ofstream parameter
    out2 = print(out2); // 错误：不能拷贝
```

#### 8.1.2. 条件状态

IO总会发生错误。有些是可恢复的，有些不能。IO类定义了函数和标志，用于访问和操作流的状态。

- `strm::iostate`：strm是任意流对象。`iostate`是整数，类型取决于及其。表示流的条件状态。它的值包括：
	- `strm::badbit`：表示流被损坏（corrupted）。
	- `strm::failbit`：IO操作失败。
	- `strm::eofbit`：流遇到文件末尾。
	- `strm::goodbit`：流没有遇到错误。这个值保证为0。
- `s.eof()`：如果eofbit置位返回true。
- `s.fail()`：如果failbit或badbit置位返回true。
- `s.bad()`：如果badbit置位返回true。
- `s.good()`：流正常返回true
- `s.clear()`：清除所有状态，将流置为有效状态。
- `s.clear(flags)`：清除某个标志。`flags`的类型是`strm::iostate`。返回void。
- `s.setstate(flags)`：添加某个标志。`flags`的类型是`strm::iostate`。返回void。
- `s.rdstate()`：返回当前的标志，返回某个`strm::iostate`值。

判断流对象的状态的最简单的方法是将对象作为条件：

```cpp
	while (cin>> word)
		// ok: read operation successful . . .
```

##### 查询流的状态

The IO classes define four `constexpr` values (§ 2.4.4) of type `iostate` that represent particular bit patterns. These values are used to indicate particular kinds of IO conditions. They can be used with the bitwise operators (§ 4.8) to test or set multiple flags in one operation.

`badbit`表示系统级别的错误，不可恢复。`failbit`遇到可恢复错误时置位。如期望读入整数时读到了字符。遇到文件末尾会同时设置`eofbit`和`failbit`。`goodbit`，保证为0，表示流正常。若`badbit`, `failbit`, `eofbit`置位，则测试流对象返回false。

此外库还定义了一组函数，对应这些状态。

##### 管理条件状态

To turn off a single condition, we use the `rdstate` member and the bitwise operators to produce the desired new state. For example, the following turns off `failbit` and `badbit` but leaves `eofbit` untouched:

```cpp
    // turns off failbit and badbit but all other bits unchanged
    cin.clear(cin.rdstate() & ~cin.failbit & ~cin.badbit);
```

#### 8.1.3. 管理输出缓存

每个输出流管理一个缓冲，which it uses to hold the data that the program reads and writes. For example, when the following code is executed

```cpp
	os <<"please enter a value: ";
```

操作系统可能先将字符串放入缓存然后再打印。

触发缓冲刷出（写到实际设备或文件）的条件有：

- The program completes normally. All output buffers are flushed as part of the return from `main`.
- 缓冲满了，需要先刷出才能继续写入。
- We can flush the buffer explicitly using a manipulator such as `endl`(§ 1.2).
- We can use the `unitbuf` manipulator to set the stream’s internal state to empty the buffer after each output operation. By default, `unitbuf` is set for `cerr`, so that writes to `cerr` are flushed immediately.
- An output stream might be tied to another stream. In this case, the buffer of the tied stream is flushed whenever the tied stream is read or written. By default, `cin` and `cerr` are both tied to cout. 因此，读`cin`或写`cerr`会刷出cout的缓冲。

> 注意：程序崩溃时缓存不会被刷出
When you debug a program that has crashed, it is essential to make sure that any output you thinkshould have been written was actually flushed. Countless hours of programmer time have been wasted tracking through code that appeared not to have executed when in fact the buffer had not been flushed and the output was pending when the program crashed.

##### Flushing the Output Buffer

Our programs have already used the `endl` manipulator, which ends the current line and flushes the buffer. There are two other similar manipulators: `flush` and `ends`. `flush` flushes the stream but adds no characters to the output; `ends` inserts a `null` character into the buffer and then flushes it:

```cpp
	cout <<"hi!" << endl; // writes hi and a newline, then flushes the buffer
	cout << "hi!" << flush; // writes hi, then flushes the buffer; adds no data
```

##### The `unitbuf` Manipulator

If we want to flush after every output, we can use the `unitbuf` manipulator. This manipulator tells the stream to do a flush after every subsequent write. The `nounitbuf` manipulator restores the stream to use normal, **system-managed** buffer flushing:

```cpp
	cout << unitbuf; // all writes will be flushed immediately
	// any output is flushed immediately, no buffering
	cout << nounitbuf; // returns to normal buffering
```

##### Tying Input and Output Streams Together

When an input stream is tied to an output stream, any attempt to read the input stream will first flush the buffer associated with the output stream. The library ties `cout` to `cin`, so the statement

```cpp
	cin >> ival;
```

causes the buffer associated with `cout` to be flushed.

> Interactive systems usually should tie their input stream to their output stream. Doing so means that all output, which might include prompts to the user, will be written before attempting to read the input.

有两个版本的`tie`：One version takes no argument and returns a pointer to the output stream, if any, to which this object is currently tied. The function returns the null pointer if the stream is not tied.

The second version of `tie` takes a pointer to an `ostream` and ties itself to that `ostream`. That is, `x.tie(&o)` ties the stream `x` to the output stream `o`.

We can tie either an `istream` or an `ostream` object to another `ostream`:

```cpp
    cin.tie(&cout);  // illustration only: the library ties cin and cout for us
    // old_tie points to the stream (if any) currently tied to cin
    ostream *old_tie = cin.tie(nullptr); // cin is no longer tied
    // ties cin and cerr; not a good idea because cin should be tied to cout
    cin.tie(&cerr);  // reading cin flushes cerr, not cout
    cin.tie(old_tie); // reestablish normal tie between cin and cout
```
To tie a given stream to a new output stream, we pass `tie` a pointer to the new stream. To untie the stream completely, we pass a null pointer. Each stream can be tied to at most one stream at a time. However, multiple streams can tie themselves to the same `ostream`.

### 8.2. 文件输入输出

头`fstream`：`ifstream` `wifstream`读文件。`ofstream` `wofstream`写文件。`fstream` `wfstream`读写文件。

In § 17.5.3 we’ll describe how to use the same file for both input and output.

这三个类型的使用方法与`cin`和`cout`相同。例如，都支持`<<`、`>>`和`getline`(§ 3.2.2)。

这三个类型还定义了只有文件类型有的操作：

- `fstream fstrm`：创建一个流，尚未绑定到文件。`fstream`是三个文件流类型中的一个。
- `fstream fstrm(s)`：创建一个流，打开一个名为`s`的文件。`s`的类型可以是`string`或C风格字符串。此构造器是explicit(7.5.4)。默认的文件`mode`取决于`fstream`的类型。
- `fstream fstrm(s, mode)`：显式指定打开的`mode`。
- `fstrm.open(s)`、`fstrm.open(s, mode)`：打开文件绑定到流。
- `fstrm.close()`：关闭流绑定的文件。
- `fstrm.is_open()`：与流关联的文件是否已打开且未关闭。

#### 8.2.1 使用文件流对象

新标准允许文件名是库`string`或C风格的字符数组。

##### Using an `fstream` in Place of an `iostream&`

This fact means that functions that are written to take a reference (or pointer) to one of the `iostream` types can be called on behalf of the corresponding `fstream`(or `sstream`) type. 即如果一个函数取形参`ostream&`，则调用时实参可以是`ofstream`。

For example, we can use the `read` and `print` functions from § 7.1.3 to read from and write to named files. In this example, we’ll assume that the names of the input and output files are passed as arguments to main(§ 6.2.5):

```cpp
    ifstream input(argv[1]); // open the file of sales transactions
    ofstream output(argv[2]); // open the output file
    Sales_data total; // variable to hold the running sum
    if (read(input, total)) { // read the first transaction
    	Sales_datatrans; // variable to hold data for the next transaction
    	while(read(input,trans)) { // read the remaining transactions
    		if(total.isbn() == trans.isbn()) // check isbns
    			total.combine(trans); // update the running total
    		else {
    			print(output,total) << endl; // printthe results
    			total= trans; // process the next book
    		}
    	}
    	print(output,total) << endl; // print the last transaction
    } else // there was no input
    	cerr << "No data?!" << endl;
```

Aside from using named files, this code is nearly identical to the version of the addition program on page 255. The important part is the calls to `read` and to `print`.

We can pass our `fstream` objects to these functions even though the parameters to those functions are defined as `istream&` and `ostream&`, respectively.

##### open和close

若创建流时不绑定文件，可以后续通过`open`绑定：

```cpp
	ifstream in(ifile); // 构建流并打开文件
	ofstream out; // 构建了流，但尚未与文件关联
	out.open(ifile + ".copy"); // open the specified file
```

若调用`open`失败，`failbit`会被置位。最好在调用`open`检查是否成功：

```cpp
    if (out) // check that the open succeeded
    // the open succeeded, so we can use the file
```

若流已经与打开的文件关联，再用此流打开文件，会导致`failbit`置位。此时此流已不可再被使用。要将流与另外的文件关联。必须先调用`close`。关闭后可与新文件关联：

```cpp
    in.close(); // close the file
    in.open(ifile + "2"); // open another file
```

如果`open`成功，流状态被设为`good()`返回true。

##### 自动构建和析构

```cpp
// foreach file passed to the program
for (auto p = argv + 1; p != argv + argc; ++p) {
	ifstream input(*p); // create input and open the file
	if(input) { // if the file is ok, ''process'' this file
		process(input);
	} else
		cerr << "couldn't open: " + string(*p);
	} // input goes out of scope and is destroyed on each iteration
```

每次循环构建一个新的`ifstream`对象（`input`）。每次循环结束后，`input`被摧毁。当一个`fstream`对象离开作用域，它绑定的文件被自动关闭。

> `fstream`对象被销毁时，会自动调用`close`。

#### 8.2.2. 文件模式

Each stream has an associated file mode that represents how the file may be used. 下面列出了文件模式：

- `in`：打开，用于输入
- `out`：打开，用于输出
- `app`：seek to the end before every write
- `ate`：seek to the end immediately after the open
- `trunc`：truncate the file
- `binary`：以二进制模式做IO

能指定的模式有以下的限制：

- `out`只能用于`ofstream`或`fstream`。
- `in`只能用于`ifstream`或`fstream`。
- `trunc` may be set only when `out` is also specified.
- `app`：只要不指定`trunc`就可以使用此模式。若指定了`app`，文件总是以输出模式打开，即使未显式指定`out`。
- 默认，以`out`模式打开的文件会被清空。若不想，需要指定`app`模式。or we must also specify `in`, in which case the file is open for both input and output (§17.5.3 will cover using the same file for input and output).
- The `ate` and `binary` modes may be specified on any file stream object type and in combination with any other file modes.

Each file stream type defines a default file mode that is used whenever we do not otherwise specify a mode. Files associated with an `ifstream` are opened in `in` mode; files associated with an `ofstream` are opened in `out` mode; and files associated with an `fstream` are opened with both `in` and `out` modes.

##### Opening a File in `out` Mode Discards Existing Data

默认，打开`ofstream`，文件的内容会被清调。若不想清掉，指定`app`：

```cpp
    // file1 is truncated in each of these cases
    ofstream out("file1"); // out and trunc are implicit
    ofstream out2("file1", ofstream::out); // trunc is implicit
    ofstream out3("file1", ofstream::out | ofstream::trunc);
    // to preserve the file's contents, we must explicitly specify app mode
    ofstream app("file2", ofstream::app); // out is implicit
    ofstream app2("file2", ofstream::out | ofstream::app);
```

##### 每次调用`open`时改变模式

The file mode of a given stream may change each time a file is opened.

```cpp
    ofstream out;  // no file mode is set
    out.open("scratchpad"); // mode implicitly out and trunc
    out.close();  // close out so we can use it for a different file
    out.open("precious", ofstream::app);  // mode is out and app
    out.close();
```

> Any time `open` is called, the file mode is set, either explicitly or implicitly. Whenever a mode is not specified, the default value is used.

### 8.3. `string`流

头`sstream`：`istringstream` `wistringstream`读`string`。`ostringstream` `wostringstream`写`string`。`stringstream` `wstringstream`读写`string`。

下面是字符串流特有的操作：

- `sstream strm`：`strm`未绑定。`sstream`是上述类型中任一个。
- `sstream strm(s)`：`strm`持有`string s`的拷贝。此构建器是`explicit`。
- `strm.str()`：返回`strm`持有的`string`的拷贝。
- `strm.str(s)`：拷贝`string s`到`strm`。返回void。

#### 8.3.1. 使用`istringstream`

An `istringstream` is often used when we have some work to do on an entire line, and other work to do with individual words within a line.
As one example, assume we have a file that lists people and their associated phone numbers. Some people have only one number, but others have several, and so on. Our input file might look like the following:

```cpp
morgan 2015552368 8625550123
drew 9735550130
lee6095550132 2015550175 8005550000
```

Each record in this file starts with a name, which is followed by one or more phone numbers. We’ll start by defining a simple class to represent our input data:

```cpp
// membersare public by default; see § 7.2 (p. 268)
struct PersonInfo {
    string name;
    vector<string> phones;
};
```

Objects of type `PersonInfo` will have one member that represents the person’s name and a `vector` holding a varying number of associated phone numbers.

Our program will read the data file and build up a `vector` of `PersonInfo`.

```cpp
    string line, word; // will hold a line and word from input, respectively
    vector<PersonInfo> people;
    // read the input a line at a time until cin hits end-of-file (or another error)
    while (getline(cin, line)) {
        PersonInfo info;
        istringstream record(line);
        record >> info.name; // read the name
        while(record >> word) // read the phone numbers
        	info.phones.push_back(word); // and store them
        people.push_back(info);// append this record to people
    }
```

#### 8.3.2. 使用`ostringstream`

An `ostringstream` is useful when we need to build up our output a little at a time but do not want to print the output until later. For example, we might want to validate and reformat the phone numbers we read in the previous example. If all the numbers are valid, we want to print a new file containing the reformatted numbers. If a person has any invalid numbers, we won’t put them in the new file. Instead, we’ll write an error message containing the person’s name and a list of their invalid numbers.

Because we don’t want to include any data for a person with an invalid number, we can’t produce the output until we’ve seen and validated all their numbers. We can, however, “write” the output to an in-memory `ostringstream`:

```cpp
for (const auto &entry : people) { // for each entry in people
    ostringstream formatted, badNums; // objects created on each loop
    for(const auto &nums : entry.phones) { // for each number
    	if(!valid(nums)) {
    		badNums << " " << nums;  // string in badNums
    	} else
    		// ''writes'' to formatted's string
    		formatted << " " << format(nums);
    }
    if(badNums.str().empty())  // there were no bad numbers
    	os << entry.name << " "  // print the name
    	<< formatted.str() << endl; // and reformatted numbers
    else // otherwise, print the name and bad numbers
    	cerr << "input error: " << entry.name
    	<< " invalid number(s) " << badNums.str() <<
        endl;
}
```

