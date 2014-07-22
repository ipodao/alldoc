
### Javascript精粹

第一章可以先不求全部理解。

第二章：

遍历数组/对象更好的方法是使用Underscore这样的库。

注意哪些值是假值。

注意null和undefined不同。

```js
var a; // undefined;
if(a) ... // 不执行
a = null; // null;
if(a) ... // 不执行
```

`if(a)`可以兼顾检查a为null和undefined的情况，因为二者都是假值。

若向判断字符串a是否为null或undefined或空串，一条语句`if(a)`也可以都

当a为null或undefined时，调用a的成员都会报异常。一般写成`a && a.mem`的形式。

合理使用&&和||简化代码书写：

&&可以防止空指针。如`var a = obj && obj.msg`：如果Obj为null，则a为null，否则a为obj.msg。

||可以提供默认值。如`var a = v || 1`。如果v是null/undefined。则a为1。

合起来：`var a = obj && obj.value || 1`。

等价代码：
```js
var a;
if(obj) a = obj.value;
if(! obj.value) a = 1;
```

