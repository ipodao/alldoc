> **Why use restify and not express?**  
Express目标是浏览器，支持模板和渲染，但Restify不支持。  
Restify exists to let you build "strict" API services that are maintanable and observable. Restify comes with automatic DTrace support for all your handlers, if you're running on a platform that supports DTrace.  

# 安装

	npm install restify

# 服务器端API

骨架：

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

Try hitting that with the following curl commands to get a feel for what restify is going to turn that into:

	curl -is http://localhost:8080/hello/mark -H 'accept: text/plain'
	HTTP/1.1 200 OK
	Content-Type: text/plain
	Content-Length: 10
	Date: Mon, 31 Dec 2012 01:32:44 GMT
	Connection: keep-alive
	
	hello mark
	
	
	$ curl -is http://localhost:8080/hello/mark
	HTTP/1.1 200 OK
	Content-Type: application/json
	Content-Length: 12
	Date: Mon, 31 Dec 2012 01:33:33 GMT
	Connection: keep-alive
	
	"hello mark"
	
	
	$ curl -is http://localhost:8080/hello/mark -X HEAD -H 'connection: close'
	HTTP/1.1 200 OK
	Content-Type: application/json
	Content-Length: 12
	Date: Mon, 31 Dec 2012 01:42:07 GMT
	Connection: close

curl默认使用`Connection: keep-alive`。In order to make the HEAD method return right away, you'll need to pass Connection: close.

Since curl is often used with REST APIs, restify provides a plugin to work around this idiosyncrasy in curl. The plugin checks whether the user agent is curl. If it is, it sets the Connection header to "close" and removes the "Content-Length" header.

	server.pre(restify.pre.userAgentConnection());

See the [pre](http://mcavage.me/node-restify/#pre) method for more information.

## 创建服务器

调用createServer API。（`listen()`的参数与[http.Server.listen](http://nodejs.org/docs/latest/api/http.html#http_server_listen_port_hostname_backlog_callback)相同）：

	var restify = require('restify');
	
	var server = restify.createServer({
	  certificate: ...,
	  key: ...,
	  name: 'MyApp',
	});
	
	server.listen(8080);

选项：

- `certificate`：字符串。If you want to create an **HTTPS** server, pass in the PEM-encoded certificate and key
- `key`：字符串。If you want to create an HTTPS server, pass in the PEM-encoded certificate and key
- `formatters`：对象。Custom response formatters for res.send()
- `log`：对象。You can optionally pass in a bunyan instance; not required
- `name`：字符串。By default, this will be set in the Server response header, 默认是restify
- `spdy`：对象。Any options accepted by node-spdy
- `version`：字符串。A default version to set for all routes
- `handleUpgrades`：布尔。Hook the upgrade event from the node HTTP server, pushing Connection: Upgrade requests through the regular request handling chain; defaults to false

## 通用处理器：`server.use()`

restify服务器有一个`use()`方法，可以接受方法`function (req, res, next)`。服务器按照处理器注册的顺序执行，so if you want some common handlers to run before any of your routes, issue calls to use() before defining routes.

调用`use()`或路由，可以传入单个函数（`function(res, res, next)`）或函数数组（`[function(req, res, next)]`）。

## 路由

restify routing, in 'basic' mode, is pretty much identical to express/sinatra, in that HTTP verbs are used with a parameterized resource to determine what chain of handlers to run. Values associated with named placeholders are available in `req.params`. Note that values will be URL-decoded before being passed to you.

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

You can also pass in a `RegExp` object and access the capture group with `req.params` (which will not be interpreted in any way):

	server.get(/^\/([a-zA-Z0-9_\.~-]+)\/(.*)/, function(req, res, next) {
		console.log(req.params[0]);
		console.log(req.params[1]);
		res.send(200);
		return next();
	});

Here any request like:

	curl localhost:8080/foo/my/cats/name/is/gandalf

Would result in `req.params[0]` being `foo` and `req.params[1]` being `my/cats/name/is/gandalf`. Basically, you can do whatever you want.

注意`next()`的使用。You are responsible for calling next() in order to run the next handler in the chain. 传入一个`Error`对象，让服务器自动返回一个响应。

You can pass in `false` to not error, but to stop the handler chain. This is useful if you had a `res.send` in an early filter, which is not an error, and you possibly have one later you want to short-circuit.

Lastly, you can pass in a string name to `next()`, and restify will lookup that route, and assuming it exists will run the chain from where you left off. So for example:

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

Note that `foo` only gets run once in that example. A few caveats:

- 如果指定的路由名不存在，restify响应500。
- 不要产生循环调用。restify won't check that.
- Lastly, you cannot "chain" next('route') calls; you can only delegate the routing chain once (this is a limitation of the way routes are stored internally, and may be revisited someday).

### 版本化的路由

Most REST APIs tend to need versioning, and restify ships with support for [semver](http://semver.org/) versioning in an Accept-Version header, the same way you specify NPM version dependencies:

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

Try hitting with:

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

In the first case, we didn't specify an `Accept-Version` header at all, so restify treats that like sending a *. Much as not sending an `Accept` header means the client gets the server's choice. Restify will choose the **first** matching route, in the order specified in the code. In the second case, we explicitly asked for for V1, which got us the same response, but then we asked for V2 and got back JSON. Finally, we asked for a version that doesn't exist and got an error (notably, we didn't send an `Accept` header, so we got a JSON response). Which segues us nicely into content negotiation.

You can default the versions on routes by passing in a version field at server creation time. Lastly, you can support multiple versions in the API by using an array:

	server.get({path: PATH, version: ['2.0.0', '2.1.0']}, sendV2);

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

You can handle errors in restify a few different ways. First, you can always just call `res.send(err)`. You can also shorthand this in a route by doing:

	server.get('/hello/:name', function(req, res, next) {
	  return database.get(req.params.name, function(err, user) {
	    if (err)
	      return next(err);
	
	    res.send(user);
	    return next();
	  });
	});

If you invoke` res.send()` with an error that has a `statusCode` attribute, that will be used, otherwise a default of 500 will be used (unless you're using `res.send(4xx, new Error('blah))`).

Alternatively, restify 2.1 supports a next.ifError API:

	server.get('/hello/:name', function(req, res, next) {
	  return database.get(req.params.name, function(err, user) {
	    next.ifError(err);
	    res.send(user);
	    next();
	  });
	});

### HttpError

Now the obvious question is what that exactly does (in either case). restify tries to be programmer-friendly with errors by exposing all HTTP status codes as a subclass of `HttpError`. So, for example, you can do this:

	server.get('/hello/:name', function(req, res, next) {
	  return next(new restify.ConflictError("I just don't like you"));
	});
	
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

The core thing to note about an `HttpError` is that it has a numeric code (statusCode) and a body. The statusCode will automatically set the HTTP response status code, and the body attribute by default will be the message.

All status codes between 400 and 5xx are automatically converted into an `HttpError` with the name being 'PascalCase' and spaces removed. For the complete list, take a look at the [node source](https://github.com/joyent/node/blob/v0.6/lib/http.js#L152-205).

From that code above` 418: I'm a teapot` would be `ImATeapotError`, as an example.

### RestError

Now, a common problem with REST APIs and HTTP is that they often end up needing to overload 400 and 409 to mean a bunch of different things. There's no real standard on what to do in these cases, but in general you want machines to be able to (safely) parse these things out, and so restify defines a convention of a `RestError`. A RestError is a subclass of one of the particular HttpError types, and additionally sets the body attribute to be a `JS object` with the attributes code and message. For example, here's a built-in `RestError`:

	var server = restify.createServer();
	server.get('/hello/:name', function(req, res, next) {
	  return next(new restify.InvalidArgumentError("I just don't like you"));
	});
	
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

The built-in restify errors are:

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

You can always add your own by subclassing `restify.RestError` like:

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

Basically, a `RestError` takes a statusCode, a restCode, a message, and a "constructorOpt" so that V8 correctly omits your code from the stack trace (you don't have to do that, but you probably want it). In the example above, we also set the name property so console.log(new MyError()); looks correct.

## （未）Socket.IO





