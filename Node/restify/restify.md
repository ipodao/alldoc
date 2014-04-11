> **为什么使用restify而不是express?**  
Express目标是浏览器，支持模板和渲染，但Restify不支持。
Restify exists to let you build "strict" API services that are maintanable and observable. Restify comes with automatic DTrace support for all your handlers, if you're running on a platform that supports DTrace.

# 安装

```shell
npm install restify
```

> ME  在Windows下安装会报错，应该是依赖dtrace是一个*C++*插件，需要*C++*编译环境。还好这个依赖是可选的。报错仍可以完成安装！

# 服务器端API

骨架：
```javascript
var restify = require('restify');

function respond(req, res, next) {
    res.send('hello ' + req.params.name);
    next();
}

var server = restify.createServer();
server.get('/hello/:name', respond);
server.head('/hello/:name', respond);

server.listen(8080, function() {
	console.log('%s listening at %s', server.name, server.url);
});
```

## 创建服务器

调用`createServer`。（`listen()`的参数与[http.Server.listen](http://nodejs.org/docs/latest/api/http.html#http_server_listen_port_hostname_backlog_callback)相同）：
```javascript
var restify = require('restify');

var server = restify.createServer({
  certificate: ...,
  key: ...,
  name: 'MyApp',
});

server.listen(8080);
```

选项：
- `certificate`：字符串。If you want to create an **HTTPS** server, pass in the PEM-encoded certificate and key
- `key`：字符串。If you want to create an HTTPS server, pass in the PEM-encoded certificate and key
- `formatters`：对象。为`res.send()`指定一个自定义的响应格式化器
- `log`：对象。You can optionally pass in a bunyan instance; not required
- `name`：字符串。将设置到响应头`Server`。默认是restify。
- `spdy`：对象。Any options accepted by node-spdy
- `version`：字符串。所有路由的默认版本。
- `handleUpgrades`：布尔。Hook the upgrade event from the node HTTP server, pushing Connection: Upgrade requests through the regular request handling chain; defaults to false

## 通用处理器：`server.use()`

restify服务器有一个`use()`方法，可以接受处理器方法`function (req, res, next)`。服务器按照处理器注册的顺序执行，因此若想在所有路由前执行通用处理器，在定义路由前调用`use()`。

调用`use()`或路由，可以传入单个函数（`function(res, res, next)`）或函数数组（`[function(req, res, next)]`）。

## 路由

restify的路由，在'basic'模式下，与express/sinatra的非常像。in that HTTP verbs are used with a parameterized resource to determine what chain of handlers to run.利用`req.params`可以读取到命名的占位符，这些值已经被URL解码。
```javascript
function send(req, res, next) {
    res.send('hello ' + req.params.name);
    return next();
}

server.post('/hello', function create(req, res, next) {
    res.send(201, Math.random().toString(36).substr(3, 8));
    return next();
});
server.put('/hello', send);

server.get('/hello/:name', send);

server.head('/hello/:name', send);

server.del('hello/:name', function rm(req, res, next) {
    res.send(204);
    return next();
});
```
You can also pass in a `RegExp` object and access the capture group with `req.params` (which will not be interpreted in any way):
```javascript
server.get(/^\/([a-zA-Z0-9_\.~-]+)\/(.*)/, function(req, res, next) {
    console.log(req.params[0]);
    console.log(req.params[1]);
    res.send(200);
    return next();
});
```
Here any request like:
```shell
curl localhost:8080/foo/my/cats/name/is/gandalf
```

Would result in `req.params[0]` being `foo` and `req.params[1]` being `my/cats/name/is/gandalf`. Basically, you can do whatever you want.

记得调用`next()`，才能让链中的下一个处理器执行。若传入一个`Error`对象，服务器自动返回一个响应。也可以不传`Error`，传`false`，停止处理器链。This is useful if you had a `res.send` in an early filter, which is not an error, and you possibly have one later you want to short-circuit.

可以向`next()`传入一个名字（字符串），restify将据此找路由，and assuming it exists will run the chain from where you left off. 例如：
```javascript
var count = 0;

server.use(function foo(req, res, next) {
    count++;
    next();
});

server.get('/foo/:id', function (req, res, next) {
   next('foo2');
});

server.get({
    name: 'foo2',
    path: '/foo/:id'
}, function (req, res, next) {
   assert.equal(count, 1);
   res.send(200);
   next();
});
```

Note that `foo` only gets run once in that example. A few caveats:
- 如果指定的路由名不存在，restify响应500。
- 不要产生循环调用。restify不会检查 won't check that.
- Lastly, you cannot "chain" next('route') calls; you can only delegate the routing chain once (this is a limitation of the way routes are stored internally, and may be revisited someday).

### 版本化的路由

多数 REST APIs 需要版本化，restify 支持 [semver](http://semver.org/) 版本化，利用`Accept-Version`头，格式与 NPM 依赖版本相同。
```javascript
var restify = require('restify');

var server = restify.createServer();

function sendV1(req, res, next) {
    res.send('hello: ' + req.params.name);
    return next();
}

function sendV2(req, res, next) {
    res.send({hello: req.params.name});
    return next();
}

var PATH = '/hello/:name';
server.get({path: PATH, version: '1.1.3'}, sendV1);
server.get({path: PATH, version: '2.0.0'}, sendV2);

server.listen(8080);
```

Try hitting with:
```shell
curl -s localhost:8080/hello/mark
"hello: mark"
$ curl -s -H 'accept-version: ~1' localhost:8080/hello/mark
"hello: mark"
$ curl -s -H 'accept-version: ~2' localhost:8080/hello/mark
{"hello":"mark"}
$ curl -s -H 'accept-version: ~3' localhost:8080/hello/mark | json
{
  "code": "InvalidVersion",
  "message": "GET /hello/mark supports versions: 1.1.3, 2.0.0"
}
```

第一个例子，根本没指定`Accept-Version`头，so restify treats that like sending a `*`。Restify将选择第一个匹配的路由，按照在代码中定义的顺序。最后，我们尝试访问不存在的版本，得到一个错误。

在创建服务器时，可以指定路由的默认版本。

路由可以支持多个版本：
```javascript
server.get({path: PATH, version: ['2.0.0', '2.1.0']}, sendV2);
```

## Upgrade Requests

Incoming HTTP requests that contain a `Connection: Upgrade` header are treated somewhat differently by the node HTTP server. If you want restify to push Upgrade requests through the regular routing chain, you need to enable `handleUpgrades` when creating the server.

To determine if a request is eligible for Upgrade, check for the existence of` res.claimUpgrade()`. This method will return an object with two properties: the `socket` of the underlying connection, and the first received data `Buffer` as `head` (may be zero-length).

Once `res.claimUpgrade()` is called, res itself is marked *unusable* for further HTTP responses; any later attempt to send() or end(), etc, will throw an Error. Likewise if res has already been used to send at least part of a response to the client, `res.claimUpgrade()` will throw an Error. Upgrades and regular HTTP Response behaviour are mutually exclusive on any particular connection.

Using the Upgrade mechanism, you can use a library like [watershed](https://github.com/jclulow/node-watershed) to negotiate WebSockets connections. For example: 

	var ws = new Watershed(); 
	server.get('/websocket/attach', function upgradeRoute(req, res, next) { 
		if (!res.claimUpgrade) { 
			next(new Error('Connection Must Upgrade For WebSockets')); 
			return; 
		}

## Content Negotiation

If you're using` res.send()` restify will automatically select the content-type to respond with, by finding the first registered formatter defined. Note in the examples above we've not defined any formatters, so we've been leveraging the fact that restify ships with `application/json`,  `text/plain` and` application/octet-stream` formatters. You can add additional formatters to restify by passing in a hash of content-type -> parser at server creation time:

	var server = restify.createServer({
	  formatters: {
	    'application/foo': function formatFoo(req, res, body) {
	      if (body instanceof Error)
	        return body.stack;
	
	      if (Buffer.isBuffer(body))
	        return body.toString('base64');
	
	      return util.inspect(body);
	    }
	  }
	});

You can do whatever you want, but you probably want to check the type of body to figure out what type it is, notably for Error/Buffer/everything else. You can always add more formatters later by just setting the formatter on server.formatters, but it's probably sane to just do it at construct time. Also, note that if a content-type can't be negotiated, the default is `application/octet-stream`. Of course, you can always explicitly set the content-type:

	res.setHeader('content-type', 'application/foo');
	res.send({hello: 'world'});

Note that there are typically at least three content-types supported by restify: json, text and binary. When you override or append to this, the "priority" might change; to ensure that the priority is set to what you want, you should set a q-value on your formatter definitions, which will ensure sorting happens the way you want:

	restify.createServer({
	  formatters: {
	    'application/foo; q=0.9': function formatFoo(req, res, body) {
	      if (body instanceof Error)
	        return body.stack;
	
	      if (Buffer.isBuffer(body))
	        return body.toString('base64');
	
	      return util.inspect(body);
	    }
	  }
	});

Lastly, you don't have to use any of this magic, as a restify response object has all the "raw" methods of a node [ServerResponse](http://nodejs.org/docs/latest/api/http.html#http.ServerResponse) on it as well.

	var body = 'hello world';
	res.writeHead(200, {
	  'Content-Length': Buffer.byteLength(body),
	  'Content-Type': 'text/plain'
	});
	res.write(body);
	res.end();

## 错误处理

错误处理有几种方式。第一种是调用`res.send(err)`。在路由中，可以将这种方式简化为：
```javascript
server.get('/hello/:name', function(req, res, next) {
  return database.get(req.params.name, function(err, user) {
    if (err)
      return next(err);

    res.send(user);
    return next();
  });
});
```

如果调用`res.send()`，传入错误，且设置了`statusCode`属性，将使用该状态码，否则使用500。还可以使用`res.send(4xx, new Error('blah))`。

或者，restify 2.1 支持 `next.ifError` API：
```javascript
server.get('/hello/:name', function(req, res, next) {
    return database.get(req.params.name, function(err, user) {
    	next.ifError(err);
    	res.send(user);
    	next();
    });
});
```
### HttpError

Now the obvious question is what that exactly does (in either case). restify tries to be programmer-friendly with errors by exposing all HTTP status codes as a subclass of `HttpError`. So, for example, you can do this:
```javascript
server.get('/hello/:name', function(req, res, next) {
	return next(new restify.ConflictError("I just don't like you"));
});
```

```shell
$ curl -is -H 'accept: text/*' localhost:8080/hello/mark
HTTP/1.1 409 Conflict
Content-Type: text/plain
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET
Access-Control-Allow-Headers: Accept, Accept-Version, Content-Length, Content-MD5, Content-Type, Date, Api-Version
Access-Control-Expose-Headers: Api-Version, Request-Id, Response-Time
Connection: close
Content-Length: 21
Content-MD5: up6uNh2ejV/C6JUbLlvsiw==
Date: Tue, 03 Jan 2012 00:24:48 GMT
Server: restify
Request-Id: 1685313e-e801-4d90-9537-7ca20a27acfc
Response-Time: 1

I just don't like you
```

The core thing to note about an `HttpError` is that it has a numeric code (statusCode) and a body. The statusCode will automatically set the HTTP response status code, and the body attribute by default will be the message.

All status codes between 400 and 5xx are automatically converted into an `HttpError` with the name being 'PascalCase' and spaces removed. For the complete list, take a look at the [node source](https://github.com/joyent/node/blob/v0.6/lib/http.js#L152-205).

From that code above `418: I'm a teapot` would be `ImATeapotError`, as an example.

### RestError

Now, a common problem with REST APIs and HTTP is that they often end up needing to overload 400 and 409 to mean a bunch of different things. There's no real standard on what to do in these cases, but in general you want machines to be able to (safely) parse these things out, and so restify defines a convention of a `RestError`. A `RestError` is a subclass of one of the particular `HttpError` types, and additionally sets the body attribute to be a `JS object` with the attributes code and message. For example, here's a built-in `RestError`:
```javascript
var server = restify.createServer();
server.get('/hello/:name', function(req, res, next) {
  return next(new restify.InvalidArgumentError("I just don't like you"));
});
```

```shell
$ curl -is localhost:8080/hello/mark | json
HTTP/1.1 409 Conflict
Content-Type: application/json
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET
Access-Control-Allow-Headers: Accept, Accept-Version, Content-Length, Content-MD5, Content-Type, Date, Api-Version
Access-Control-Expose-Headers: Api-Version, Request-Id, Response-Time
Connection: close
Content-Length: 60
Content-MD5: MpEcO5EQFUZ2MNeUB2VaZg==
Date: Tue, 03 Jan 2012 00:50:21 GMT
Server: restify
Request-Id: bda456dd-2fe4-478d-809c-7d159d58d579
Response-Time: 3

{
  "code": "InvalidArgument",
  "message": "I just don't like you"
}
```

内建的错误：
- RestError
- BadDigestError
- BadMethodError
- InternalError
- InvalidArgumentError
- InvalidContentError
- InvalidCredentialsError
- InvalidHeaderError
- InvalidVersionError
- MissingParameterError
- NotAuthorizedError
- RequestExpiredError
- RequestThrottledError
- ResourceNotFoundError
- WrongAcceptError

可以创建`restify.RestError`的子类：
```javascript
var restify = require('restify');
var util = require('util');

function MyError(message) {
    restify.RestError.call(this, {
        restCode: 'MyError',
        statusCode: 418,
        message: message,
        constructorOpt: MyError
    });
    this.name = 'MyError';
};
util.inherits(MyError, restify.RestError);
```

Basically, a `RestError` takes a statusCode, a restCode, a message, and a "constructorOpt" so that V8 correctly omits your code from the stack trace (you don't have to do that, but you probably want it). In the example above, we also set the name property so `console.log(new MyError())`; looks correct.

## （未）Socket.IO

## Server API

### 事件

Restify会发送`http.Server`的所有事件，此外，还有：

#### Event: 'NotFound'

`function (request, response, cb) {}`

请求URL不存在。restify会先检查此事件的监听器，如果没有，使用默认的 404 处理器。如果你监听该事件，应该由你响应客户端。

#### Event: 'MethodNotAllowed'

`function (request, response, cb) {}`

没有路由处理该 HTTP 动词。若没有该事件的监听器，默认使用 405 处理器。如果你监听该事件，应该由你响应客户端。

#### Event: 'VersionNotAllowed'

`function (request, response, cb) {}`

When a client request is sent for a route that exists, but does not match the version(s) on those routes, restify will emit this event. 若没有该事件的监听器，默认使用 400 处理器。如果你监听该事件，应该由你响应客户端。

#### Event: UnsupportedMediaType'

`function (request, response, cb) {}`

When a client request is sent for a route that exist, but has a content-type mismatch, restify will emit this event. 若没有该事件的监听器，默认使用 415 处理器。 如果你监听该事件，应该由你响应客户端。

#### Event: 'after'

`function (request, response, route, error) {}`

Emitted after a route has finished all the handlers you registered. 可以在这里记录审计日志。`route`参数是刚运行的`Route`对象。注意当使用默认的 404/405/BadVersion 处理器时，该事件也会被触发，但`route`为null。如果你注册了自己的监听器，该事件不会触发，除非你调用了`cb`参数。

#### Event: 'uncaughtException'

`function (request, response, route, error) {}`

Emitted when some handler throws an uncaughtException somewhere in the chain. 默认行为只是调用`res.send(error)`，让restify内建处理转换，but you can override to whatever you want here.

### 属性

A restify server has the following properties on it:
- `name`：字符串，服务器名
- `version`：字符串，路由默认版本
- `log`：对象，bunyan实例
- `acceptable`：Array(String)，服务器可以响应的*content-types*列表
- `url`：字符串，Once listen() is called, this will be filled in with where the server is running

### 方法

#### `address()`

包装Node的`address()`。

#### `listen(port, [host], [callback])` 或 `listen(path, [callback])`

包装Node的`listen()`。

#### `close()`

包装Node的`close()`。

#### `pre()`

Allows you to add in handlers that run before routing occurs. This gives you a hook to change request headers and the like if you need to. Note that `req.params` will be undefined, as that's filled in after routing.

```javascript
server.pre(function(req, res, next) {
    req.headers.accept = 'application/json';  // screw you client!
    return next();
});
```

#### `use()`

Allows you to add in handlers that run no matter what the route.

## 自带的插件

restify自带的插件：
- Accept header parsing
- Authorization header parsing
- CORS handling plugin
- Date header parsing
- JSONP support
- Gzip Response
- Query string parsing
- Body parsing (JSON/URL-encoded/multipart form)
- Static file serving
- Throttling
- Conditional request handling
- Audit logger

例子：

```javascript
var server = restify.createServer();
server.use(restify.acceptParser(server.acceptable));
server.use(restify.authorizationParser());
server.use(restify.dateParser());
server.use(restify.queryParser());
server.use(restify.jsonp());
server.use(restify.gzipResponse());
server.use(restify.bodyParser());
server.use(restify.throttle({
    burst: 100,
    rate: 50,
    ip: true,
    overrides: {
        '192.168.1.1': {
        	rate: 0,        // unlimited
        	burst: 0
        }
    }
}));
server.use(restify.conditionalRequest());
```

### Accept Parser

解析 `Accept` 头，and ensures that the server can respond to what the client asked for. You almost always want to just pass in `server.acceptable` here, as that's an array of content types the server knows how to respond to (with the formatters you've registered). If the request is for a non-handled type, this plugin will return an error of 406.
```javascript
server.use(restify.acceptParser(server.acceptable));
```

### Authorization Parser
```javascript
server.use(restify.authorizationParser());
```

尽力解析 `Authorization` 头。目前只支持 HTTP Basic Auth 和 [HTTP Signature](https://github.com/joyent/node-http-signature)。When this is used, `req.authorization` will be set to something like:
```javascript
{
  scheme: <Basic|Signature|...>,
  credentials: <Undecoded value of header>,
  basic: {
    username: $user
    password: $password
  }
}
```

`req.username` will also be set, and defaults to 'anonymous'. If the scheme is unrecognized, the only thing avaiable in `req.authorization` will be scheme and credentials - it will be up to you to parse out the rest.

### CORS

```javascript
server.use(restify.CORS());
```

Supports tacking [CORS](http://www.w3.org/TR/cors) headers into actual requests (as defined by the spec). Note that preflight requests are automatically handled by the router, and you can override the default behavior on a per-URL basis with server.opts(:url, ...). To fully specify this plugin, a sample invocation is:
```javascript
server.use(restify.CORS({
    origins: ['foo.com', 'bar.com'],   // defaults to ['*']
    credentials: true                  // defaults to false
    headers: ['x-foo']                 // sets expose-headers
}));
```

### Date Parser

```javascript
server.use(restify.dateParser());
```

Parses out the HTTP Date header (if present) and checks for clock skew (default allowed clock skew is 300s, like Kerberos). You can pass in a number, which is interpreted in seconds, to allow for clock skew.
```javascript
// Allows clock skew of 1m
server.use(restify.dateParser(60));
```

### QueryParser

```javascript
server.use(restify.queryParser());
```

Parses the HTTP query string (i.e., `/foo?id=bar&name=mark`). If you use this, the parsed content will always be available in `req.query`, additionally params are merged into `req.params`. You can disable by passing in `mapParams: false` in the options object:
```javascript
server.use(restify.queryParser({ mapParams: false }));
```

### JSONP

Supports checking the query string for callback or jsonp and ensuring that the content-type is appropriately set if JSONP params are in place. There is also a default application/javascript formatter to handle this.

You should set the queryParser plugin to run before this, but if you don't this plugin will still parse the query string properly.

### BodyParser

Blocks your chain on reading and parsing the HTTP request body. Switches on `Content-Type` and does the appropriate logic. 目前支持`application/json`, `application/x-www-form-urlencoded` 和 `multipart/form-data`。
```javascript
server.use(restify.bodyParser({
    maxBodySize: 0,
    mapParams: true,
    mapFiles: false,
    overrideParams: false,
    multipartHandler: function(part) {
        part.on('data', function(data) {
          /* do something with the multipart data */
        });
    },
    multipartFileHandler: function(part) {
        part.on('data', function(data) {
          /* do something with the multipart file data */
        });
    },
    keepExtensions: false,
    uploadDir: os.tmpdir()
 }));
```

选项：
- `maxBodySize`。The maximum size in bytes allowed in the HTTP body. Useful for limiting clients from hogging server memory.
- `mapParams`。if `req.params` should be filled with parsed parameters from HTTP body.
- `mapFiles`。if `req.params` should be filled with the contents of files sent through a multipart request. [formidable](https://github.com/felixge/node-formidable) is used internally for parsing, and a file is denoted as a multipart part with the filename option set in its `Content-Disposition`. This will only be performed if `mapParams` is true.
- `overrideParams`。if an entry in `req.params` should be overwritten by the value in the body if the names are the same. For instance, if you have the route `/:someval`, and someone posts an *x-www-form-urlencoded* *Content-Type* with the body `someval=happy` to `/sad`, the value will be happy if `overrideParams` is true, sad otherwise.
- `multipartHandler`。a callback to handle any multipart part which is not a file. If this is omitted, the default handler is invoked which may or may not map the parts into `req.params`, depending on the mapParams-option.
- `multipartFileHandler`。a callback to handle any multipart file. It will be a file if the part have a `Content-Disposition` with the filename parameter set. This typically happens when a browser sends a from and there is a parameter similar to `<input type="file" />`. If this is not provided, the default behaviour is to map the contents into `req.params`.
- `keepExtensions`。if you want the uploaded files to include the extensions of the original files (multipart uploads only). Does nothing if `multipartFileHandler` is defined.
- `uploadDir` - Where uploaded files are intermediately stored during transfer before the contents is mapped into `req.params`. Does nothing if `multipartFileHandler` is defined.

### RequestLogger

Sets up a child *bunyan* logger with the current request id filled in, along with any other parameters you define.
```javascript
server.use(restify.requestLogger({
    properties: {
        foo: 'bar'
    },
    serializers: {...}
}));
```

You can pass in no options to this, in which case only the request id will be appended, and no serializers appended (this is also the most performant); the logger created at server creation time will be used as the parent logger.

### GzipResponse
```javascript
server.use(restify.gzipResponse());
```
If the client sends an `accept-encoding: gzip` header (or one with an appropriate q-val), then the server will automatically gzip all response data. Note that only gzip is supported, as this is most widely supported by clients in the wild. This plugin will overwrite some of the internal streams, so any calls to `res.send`, `res.write`, etc., will be compressed. A side effect is that the `content-length` header cannot be known, and so `transfer-encoding: chunked` will always be set when this is in effect. This plugin has no impact if the client does not send `accept-encoding: gzip`.

### Serve Static

The serveStatic module is different than most of the other plugins, in that it is expected that you are going to map it to a route, as below:

server.get(/\/docs\/current\/?.*/, restify.serveStatic({
  directory: './documentation/v1',
  default: 'index.html'
}));
The plugin will enforce that all files under directory are served. The  directory served is relative to the process working directory. You can also provide a default parameter such as index.html for any directory that lacks a direct file match. You can specify additional restrictions by passing in a match parameter, which is just a RegExp to check against the requested file name. Lastly, you can pass in a maxAge numeric, which will set the Cache-Control header. Default is 3600 (1 hour).

### Throttle

restify ships with a fairly comprehensive implementation of [Token bucket](http://en.wikipedia.org/wiki/Token_bucket), with the ability to throttle on IP (or `x-forwarded-for`) and username (from `req.username`). You define "global" request rate and burst rate, and you can define overrides for specific keys. Note that you can always place this on per-URL routes to enable different request rates to different resources (if for example, one route, like `/my/slow/database` is much easier to overwhlem than `/my/fast/memcache`).

```javascript
server.use(restify.throttle({
    burst: 100,
    rate: 50,
    ip: true,
    overrides: {
        '192.168.1.1': {
            rate: 0,        // unlimited
            burst: 0
        }
    }
}));
```

If a client has consumed all of their available rate/burst, an HTTP response code of *429 Too Many Requests* is returned.

选项：
- rate	Number	Steady state number of requests/second to allow
- burst	Number	If available, the amount of requests to burst to
- ip	Boolean	Do throttling on a /32 (source IP)
- xff	Boolean	Do throttling on a /32 (X-Forwarded-For)
- username	Boolean	Do throttling on req.username
- overrides	Object	Per "key" overrides
- tokensTable	Object	Storage engine; must support put/get
- maxKeys	Number	If using the built-in storage table, the maximum distinct throttling keys to allow at a time

Note that `ip`, `xff` and `username` are XOR'd.

#### Using an external storage mechanism for key/bucket mappings.

By default, the restify throttling plugin uses an in-memory LRU to store mappings between throttling keys (i.e., IP address) to the actual bucket that key is consuming. If this suits you, you can tune the maximum number of keys to store in memory with options.maxKeys; the default is 10000.

In some circumstances, you want to offload this into a shared system, such as Redis, if you have a fleet of API servers and you're not getting steady and/or uniform request distribution. To enable this, you can pass in options.tokensTable, which is simply any Object that supports put and get with a String key, and an Object value.


