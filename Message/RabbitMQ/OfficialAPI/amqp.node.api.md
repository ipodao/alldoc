# amqplib

http://www.squaremobius.net/amqp.node/doc/channel_api.html

[toc]

```js
var amqp = require('amqplib');
```

客户端API贴近协议模型。主要做法是创建信道（channels）发送命令。Most errors in AMQP invalidate just the channel which had problems, so this ends up being a fairly natural way to use AMQP. The downside is that it doesn't give any guidance on useful ways to use AMQP; that is, it does little beyond giving access to the various AMQP methods.

AMQP中多数操作都是 RPCs，在协议的信道层是同步的，但从库的角度看是异步的。于是，多数方法返回一个promise，在未来提供服务器的响应（常常包含有用的信息，如产生的标识符）。RPCs are queued by the channel if it is already waiting for a reply -- synchronising on RPCs in this way is implicitly required by the protocol specification.

失败的操作将：

- 拒绝当前的RPC，如果有
- 令channel对象失效，进一步操作会抛出异常
- 拒绝等待发送的所有RPC
- channel对象发出'error'事件
- channel对象发出'close'事件

Since the RPCs are effectively synchronised, any such channel error is very likely to have been caused by the outstanding RPC. However, it's often sufficient to fire off a number of RPCs and check only the returned promise for the last, since it'll be rejected if it or any of its predecessors fail.

The exception thrown on operations subsequent to a failure or closure also contains the stack at the point that the channel was closed, in the field `stackAtStateChange`. This may be useful to determine what has caused an unexpected closure.

```js
connection.createChannel().then(function(ch) {
  ch.close();
  try {
    ch.close();
  } catch (alreadyClosed) {
    console.log(alreadyClosed.stackAtStateChange);
  }
});
```

Promises returned from methods are amenable to composition using, for example, when.js's functions:
```js
amqp.connect().then(function(conn) {
  var ok = conn.createChannel();
  ok = ok.then(function(ch) {
    return when.all([
      ch.assertQueue('foo'),
      ch.assertExchange('bar'),
      ch.bindQueue('foo', 'bar', 'baz'),
      ch.consume('foo', handleMessage)
    ]);
  });
  return ok;
}).then(null, console.warn);
```

Often, AMQP commands have an `arguments` table that can contain arbitrary values, usually used by implementation-specific extensions like [RabbitMQ's consumer priorities](http://www.rabbitmq.com/consumer-priority.html). This is accessible as the option `arguments`, an object: if an API method does not account for an extension in its `options`, you can fall back to using the `arguments` object, though bear in mind that the field name will usually be 'x-something', while the options are just 'something'. Values passed in `options`, if understood by the API, will override those given in `arguments`.

```js
var common_options = {durable: true, noAck: true};
ch.assertQueue('foo', common_options);
// Only 'durable' counts for queues

var bar_opts = Object.create(common_options);
bar_opts.autoDelete = true;
// "Subclass" our options
ch.assertQueue('bar', bar_opts);

var foo_consume_opts = Object.create(common_options);
foo_consume_opts.arguments = {'x-priority': 10};
ch.consume('foo', console.log, foo_consume_opts);
// Use the arguments table to give a priority, even though it's
// available as an option

var bar_consume_opts = Object.create(foo_consume_opts);
bar_consume_opts.priority = 5;
ch.consume('bar', console.log, bar_consume_opts);
// The 'priority' option will override that given in the arguments
// table
```

## connect([url], [socketOptions])

连接到一个AMQP 0-9-1服务器，optionally given an AMQP URL (see [AMQP URI syntax](http://www.rabbitmq.com/uri-spec.html)) and socket options. 协议部分（`amqp:` 或`amqps:`）是必需的；省略的部分默认是'`amqp://guest:guest@localhost:5672`'。如果`url`全部省略，默认是'`amqp://localhost`'。

为了方便，省略路径部分，会被解析成虚拟机`/`。根据URI规范，结尾只有一个斜杠'`amqp://localhost/`'会被理解为名字为空的虚拟机。虚拟机名字必须被转义，例如，虚拟机`/foo`需要转义成'`%2Ffoo`'，如'`amqp://localhost/%2Ffoo`'。

AMQP调优参数可以通过URI的查询部分指定，如'`amqp://localhost?frameMax=0x1000`'。

- `frameMax`, the size in bytes of the maximum frame allowed over the connection. 0 means no limit (but since frames have a size field which is an unsigned 32 bit integer, it's perforce 2^32 - 1); I default it to 0x1000, i.e. 4kb, which is the allowed minimum, will fit many purposes, and not chug through Node.JS's buffer pooling.
- `channelMax`, the maximum number of channels allowed. Default is 0, meaning `2^16 - 1`.
- `heartbeat`: 心跳周期，单位秒。默认0，没有心跳。OMG，没有心跳！
- `locale`: the desired locale for error messages, I suppose. RabbitMQ only ever uses en_US; which, happily, is the default.

The socket options will be passed to the socket library (`net` or `tls`). They must be fields set on the object supplied; i.e., not on a prototype. This is useful for supplying certificates and so on for an SSL connection; see the [SSL guide](http://squaremo.github.com/amqp.node/doc/ssl.html).

The socket options may also include the key `noDelay`, with a boolean value. If the value is true, this sets `TCP_NODELAY` on the underlying socket.

### 返回值及异常

返回的promise，若解决，提供一个打开的`ChannelModel`；若被拒，提供一个sympathetically-worded error (in en_US)。

URI格式错误将导致connect()抛出异常；其他文件，如被拒绝或TCP连接中断，将导致promise被拒。

RabbitMQ since version 3.2.0 will send a frame to notify the client of authentication failures, which results in a rejected promise; RabbitMQ before version 3.2.0, per the AMQP specification, will close the socket in the case of an authentication failure, making a dropped connection ambiguous (it will also wait a few seconds before doing so).

### 心跳

If you supply a non-zero period in seconds as the heartbeat parameter, the connection will be monitored for liveness. If the client fails to read data from the connection for two successive intervals, the connection will emit an error and close. It will also send heartbeats to the server (in the absence of other data).

## new ChannelModel(connection)

此构造器表示一个a connection in the channel API. It takes as an argument a `connection.Connection`; though it is better to use `connect()`（上一节）, which will open the connection for you. It is exported as a potential extension point.

### ChannelModel#close

干净的关闭连接。Will immediately invalidate any unresolved operations, so it's best to make sure you've done everything you need to before calling this. 返回一个promise，当连接和底层的Socket关闭后解决。并且此时，ChannelModel会送出'close'事件。

尽管不是绝对必须，但在退出 前关闭连接，能够避免服务器日志中出现一些警告：
```js
var open = amqp.connect();
open.then(function(conn) {
  var ok = doStuffWithConnection(conn);
  return ok.then(conn.close.bind(conn));
}).then(null, console.warn);
```

Note that I'm synchronising on the return value of `doStuffWithConnection()`, presumably a promise, so that I can be sure I'm all done.

可以在程序被中断时关闭连接：

```js
var open = amqp.connect();
open.then(function(conn) {
  process.once('SIGINT', conn.close.bind(conn));
  return doStuffWithConnection(conn);
}).then(null, console.warn);
```

NB it's no good using `process.on('exit', ...)`, since close() needs to do I/O.

### ChannelModel#on('close', function() {...})

Emitted once the closing handshake initiated by `#close()` has completed; or, if server closed the connection, once the client has sent the closing handshake; or, if the underlying stream (e.g., socket) has closed.







