[toc]

## 事件

所有与事件相关的API：http://api.jquery.com/category/events/

### 绑定事件处理器

#### .bind()

http://api.jquery.com/bind/

向元素添加事件处理器。返回`jQuery`。签名：

```
    .bind( eventType [, eventData ], handler )
    .bind( eventType [, eventData ] [, preventBubble ] )
    .bind( events )
```

参数：

- `eventType`。类型：字符串。一个或多个DOM事件类型，如"click"；或自定义事件名。
- `eventData`。类型：对象。传给事件处理器的数据。
- `handler`。类型：函数`( Event eventObject )`。回调函数，处理器。
- `preventBubble`。类型：布尔。Setting the third argument to false will attach a function that prevents the default action from occurring and stops the event from bubbling. 默认为true。

从jQuery 1.7开始，推荐用`.on()`向文档元素添加事件处理器。对于之前的版本，`.bind()`用于直接向元素绑定事件处理器。调用`.bind()`时，元素必须存在。

`eventType`可以是任何字符串。自定义事件名可以通过Javascript手工触发，如`.trigger()`或`.triggerHandler()`。

如果`eventType`字符串包含点，则事件属于某个命名空间。例如，`.bind( "click.name", handler )`，`click`是事件类型，`name`是命名空间。Namespacing allows us to unbind or trigger some events of a type without affecting others. 参见对`.unbind()`的讨论。

如果绑定多个处理器，则它们都将执行。执行顺序与绑定顺序相同。在所有事件处理器执行后，事件继续沿标准事件传播路径传播。

A basic usage of .bind() is:

```js
    $( "#foo" ).bind( "click", function() {
      alert( "User clicked on 'foo.'" );
    });
```

##### 多个事件

一次可以绑定多个事件类型。空格分隔：

```js
    $( "#foo" ).bind( "mouseenter mouseleave", function() {
      $( this ).toggleClass( "entered" );
    });
```

As of jQuery 1.4 we can bind multiple event handlers simultaneously by passing an object of event type/handler pairs:

```js
    $( "#foo" ).bind({
      click: function() {
        // Do something on click
      },
      mouseenter: function() {
        // Do something on mouseenter
      }
    });
```

##### 事件处理器

处理器方法中，this指向处理器绑定的元素。

```js
    $( "#foo" ).bind( "click", function() {
      alert( $( this ).text() );
    });
```

As of jQuery 1.4.2 duplicate event handlers can be bound to an element instead of being discarded. This is useful when the event data feature is being used, or when other unique data resides in a closure around the event handler function.

从jQuery 1.4.3开始，事件处理器的位置上可以传false。This will bind an event handler equivalent to: `function(){ return false; }`. This function can be removed at a later time by calling: `.unbind( eventName, false )`.

##### 事件对象

事件处理器的第一个参数是事件对象。

在事件处理器中返回false等价于在事件对象上调用`.preventDefault()`和`.stopPropagation()`。

```js
    $( document ).ready(function() {
      $( "#foo" ).bind( "click", function( event ) {
        alert( "The mouse cursor is at (" +
          event.pageX + ", " + event.pageY +
          ")" );
      });
    });
```

##### 事件数据

`eventData`一般不会用到。它可以解决闭包的一些问题。如：

```js
    var message = "Spoon!";
    $( "#foo" ).bind( "click", function() {
      alert( message );
    });
    message = "Not in the face!";
    $( "#bar" ).bind( "click", function() {
      alert( message );
    });
```

两个回调执行时，显示的消息都将是`Not in the face!`。解决办法是使用`eventData`：

```js
    var message = "Spoon!";
    $( "#foo" ).bind( "click", {
      msg: message
    }, function( event ) {
      alert( event.data.msg );
    });
    message = "Not in the face!";
    $( "#bar" ).bind( "click", {
      msg: message
    }, function( event ) {
      alert( event.data.msg );
    });
```

> See the `.trigger()` method reference for a way to pass data to a handler at the time the event happens rather than when the handler is bound.

As of jQuery 1.4 we can no longer attach data (and thus, events) to object, embed, or applet elements because critical errors occur when attaching data to Java applets.

### .delegate()

http://api.jquery.com/delegate/

Attach a handler to one or more events for all elements that match the selector, now or in the future, based on a specific set of root elements. 返回jQuery。
