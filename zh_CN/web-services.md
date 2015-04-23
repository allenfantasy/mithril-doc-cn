## Web Services

Mithril provides a high-level utility for working with web services, which allows writing asynchronous code relatively procedurally.

It provides a number of useful features out of the box:

-	The ability to get an early reference to a container that will hold the asynchronous response
-	The ability to queue operations to be performed after the asynchronous request completes
-	The ability to "cast" the response to a class of your choice
-	The ability to unwrap data in a response that includes metadata properties

## 网络交互

Mithril提供一系列高抽象的工具同网络服务器交互，并且能以相对顺序的风格书写异步代码。

它提供了一些很有用的开箱即用的功能：

-	能够获得一个持有异步的响应的早期的引用(an early reference)
-	能够在异步请求完成后顺序执行操作队列
-	能够将响应转化成你选择的类
-	能够从包含元数据的响应中获取数据

---

### Basic usage

The basic usage pattern for `m.request` returns an [`m.prop`](mithril.prop.md) getter-setter, which is populated when the AJAX request completes.

The returned getter-setter can be thought of as a box: you can pass this reference around cheaply, and you can "unwrap" its value when needed.

### 基本用法

`m.request`的基本用法是返回一个[`m.prop`](mithril.prop.md)创建的getter-setter,它会在AJAX请求完成后被创建。

返回的getter-setter可以被看做一个盒子：你可以轻松地传递它，也可以在需要的时候打开它得到里面的值。

```javascript
var users = m.request({method: "GET", url: "/user"});

//assuming the response contains the following data: `[{name: "John"}, {name: "Mary"}]`
//then when resolved (e.g. in a view), the `users` getter-setter will contain a list of users
//假设响应包含的是这样的数据: `[{name: "John"}, {name: "Mary"}]`
//当完成的时候 (比如在一个视图里), `users` getter-setter会包含该user列表
//i.e. 即 users() //[{name: "John"}, {name: "Mary"}]
```

Note that this getter-setter holds an *undefined* value until the AJAX request completes. Attempting to unwrap its value early will likely result in errors.

The returned getter-setter also implements the [promise](mithril.deferred.md) interface (also known as a *thennable*): this is the mechanism you should always use to queue operations to be performed on the data from the web service.

The simplest use case of this feature is to implement functional value assignment via `m.prop` (i.e. the same thing as above). You can bind a pre-existing getter-setter by passing it in as a parameter to a `.then` method:

需要注意的是：该getter-setter在AJAX请求完成前持有的值为*undefined*。过早尝试使用它的值很可能会带来错误的结果。

该getter-setter还实现了[promise](mithril.deferred.md)接口(或者叫*thennable*): 当你需要顺序执行一串对来自服务器的数据进行的操作时，往往需要用到这个机制。

一个使用该功能的最简单的例子：使用`m.prop` (也就是上面提到的东西)实现函数式赋值。你可以绑定一个getter-setter，只需要将它作为`.then`方法的参数即可。

```javascript
var users = m.prop([]); //default value 默认值

m.request({method: "GET", url: "/user"}).then(users)
//assuming the response contains the following data: `[{name: "John"}, {name: "Mary"}]`
//then when resolved (e.g. in a view), the `users` getter-setter will contain a list of users
//假设响应包含的是这样的数据: `[{name: "John"}, {name: "Mary"}]`
//当完成的时候 (比如在一个视图里), `users` getter-setter会包含该user列表
//i.e. 即 users() //[{name: "John"}, {name: "Mary"}]
```

This syntax allows you to bind intermediate results before piping them down for further processing, for example:

该语法允许你在使用响应的数据做操作前绑定中间结果，如:

```javascript
var users = m.prop([]); //default value 默认值
var doSomething = function() { /*...*/ }

m.request({method: "GET", url: "/user"}).then(users).then(doSomething)
```

While both basic assignment syntax and thennable syntax can be used to the same effect, typically it's recommended that you use the assignment syntax in the first example whenever possible, as it's easier to read.

The thennable mechanism is intended to be used in three ways:

-	In the model layer: to process web service data in transformative ways (e.g. filtering a list based on a parameter that the web service doesn't support)
-	In the controller layer: to bind redirection code upon a condition
-	In the controller layer: to bind error messages

虽然基础的赋值语法和thennable语法都能得到同样的效果，但我们通常会建议你使用第一个示例中的赋值语法，如果可以的话，因为它更容易读。

thennable机制被设计用于下面三种途径：

-	在模型层中: 对来自服务器的数据做变形 (如：基于某个服务器不支持的参数对列表进行过滤)
-	在控制层中: 绑定基于某个条件做重定向的代码
-	在控制层中: 绑定错误信息

#### Processing web service data

This step is meant to be done in the model layer. Doing it in the controller level is also possible, but philosophically not recommended, because by tying logic to a controller, the code becomes harder to reuse due to unrelated controller dependencies.

In the example below, the `listEven` method returns a getter-setter that resolves to a list of users containing only users whose id is even.

#### 加工服务器数据

该步骤要在模型层完成。虽然在控制层也可以做这个，但是(计算机)哲学上我们不推荐这么做，因为一旦将逻辑绑在控制器上，代码会因为无关的控制器依赖而变得难以重用。

在下面的示例中，`listEven`方法返回一个解析后只包含id为偶数的user列表的getter-setter。

```javascript
//model
var User = {}

User.listEven = function() {
	return m.request({method: "GET", url: "/user"}).then(function(list) {
		return list.filter(function(user) {return user.id % 2 == 0});
	});
}

//controller
var controller = function() {
	this.users = User.listEven()
}
```

#### Bind redirection code

This step is meant to be done in the controller layer. Doing it in the model level is also possible, but philosophically not recommended, because by tying redirection to the model, the code becomes harder to reuse due to overly tight coupling.

In the example below, we use the previously defined `listEven` model method and queue a controller-level function that redirects to another page if the user list is empty.

#### 绑定重定向代码

该步骤要在控制层完成。在模型层虽然可行，但是(计算机)哲学上我们不推荐这么做，因为一旦将重定向逻辑绑在模型上，代码会因为过紧的耦合而变得难以重用。

在下面的示例中，我们使用之前定义的`listEven`模型方法并在后面列了一个控制层函数用于在user列表为空时重定向到其他页面。

```javascript
//controller
var controller = function() {
	this.users = User.listEven().then(function(users) {
		if (users.length == 0) m.route("/add");
	})
}
```

#### Binding errors

Mithril thennables take two functions as optional parameters: the first parameter is called if the web service request completes successfully. The second parameter is called if it completes with an error.

Error binding is meant to be done in the controller layer. Doing it in the model level is also possible, but generally leads to more code in order to connect all the dots.

In the example below, we bind an error getter-setter to our previous controller so that the `error` variable gets populated if the server throws an error.

#### 绑定错误信息

Mithril的thennable使用两个函数作为可选参数：第一个参数会在请求成功完成时被调用。第二个参数会在请求完成后并报错时被调用。

错误信息的绑定应该在控制层完成。在模型层虽然可行，但往往会因为需要联系全部的点(connect all the dots)而带来更多的代码。

在下面的示例中，我们给之前的控制器绑定了一个错误信息的getter-setter，这样，一旦服务器抛出错误信息，`error`就会被赋值(get populated)。

```javascript
//controller
var controller = function() {
	this.error = m.prop("")
	
	this.users = User.listEven().then(function(users) {
		if (users.length == 0) m.route("/add");
	}, this.error)
}
```

If the controller doesn't already have a success callback to run after a request resolves, you can still bind errors like this:

如果控制器并没有表示请求成功的回调，你也可以像这样绑定错误信息:

```javascript
//controller
var controller = function() {
	this.error = m.prop("")
	
	this.users = User.listEven().then(null, this.error)
}
```

---

### Queuing Operations

As you saw, you can chain operations that act on the response data. Typically this is required in three situations:

-	In model-level methods if client-side processing is needed to make the data useful for a controller or view
-	In the controller, to redirect after a model service resolves
-	In the controller, to bind error messages

In the example below, we take advantage of queuing to debug the AJAX response data prior to doing further processing on the user list:

### 操作队列

如你所见：你可以在响应的数据上做链式操作。通常在以下三种情形下我们会这么干：

-	需要在客户端的数据层方法中对数据进行加工才能使他们能够被控制层或者视图层使用。
-	模型层对数据处理后需要在控制器中进行重定向时
-	控制器中绑定错误信息

下面的示例中，我们使用队列在进一步对user列表操作之前调试AJAX响应的数据。

```javascript
var users = m.request({method: "GET", url: "/user"})
	.then(log)
	.then(function(users) {
		//add one more user to the response
		//向响应的数据加入更多user
		return users.concat({name: "Jane"})
	})
	
function log(value) {
    console.log(value)
    return value
}

//assuming the response contains the following data: `[{name: "John"}, {name: "Mary"}]`
//then when resolved (e.g. in a view), the `users` getter-setter will contain a list of users
//假设响应包含的是这样的数据: `[{name: "John"}, {name: "Mary"}]`
//当完成的时候 (比如在一个视图里), `users` getter-setter会包含该user列表
//i.e. 即 users() //[{name: "John"}, {name: "Mary"}, {name: "Jane"}]
```

---

### Casting the Response Data to a Class

It's possible to auto-cast a JSON response to a class. This is useful when we want to control access to certain properties in an object, as opposed to exposing all the fields in POJOs (plain old Javascript objects) for arbitrary processing.

In the example below, `User.list` returns a list of `User` instances.

### 将响应数据转化成类

我们可以将JSON格式的响应自动转化为类。这在我们想要控制某些属性的访问权限时很有帮助,我们反对暴露POJO(不含逻辑的javascript对象)中所有的域然后随心所欲地处理。

下面的示例中，`User.list`返回`User`实例的列表

```javascript
var User = function(data) {
	this.name = m.prop(data.name);
}

User.list = function() {
	return m.request({method: "GET", url: "/user", type: User});
}

var users = User.list();
//assuming the response contains the following data: `[{name: "John"}, {name: "Mary"}]`
//then when resolved (e.g. in a view), `users` will contain a list of User instances
//假设响应包含的是这样的数据: `[{name: "John"}, {name: "Mary"}]`
//当完成的时候 (比如在一个视图里), `users`会包含一个User实例列表
//i.e. 即 users()[0].name() == "John"
```

---

### Unwrapping Response Data

Often, web services return the relevant data wrapped in objects that contain metadata.

Mithril allows you to unwrap the relevant data, by providing two callback hooks: `unwrapSuccess` and `unwrapError`.

These hooks allow you to unwrap different parts of the response data depending on whether it succeed or failed.

### 解包响应的数据

通常，服务器会采用将数据打包成包含元数据的对象的方式来返回有关数据。

Mithril中，你只需要提供两个回调钩子：`unwrapSuccess`和`unwrapError`即可解包相关数据。

这两个钩子会根据你请求的成功与否来决定解出响应数据哪个部分的包。

```javascript
var users = m.request({
	method: "GET",
	url: "/user",
	unwrapSuccess: function(response) {
		return response.data;
	},
	unwrapError: function(response) {
		return response.error;
	}
});

//assuming the response is: `{data: [{name: "John"}, {name: "Mary"}], count: 2}`
//then when resolved (e.g. in a view), the `users` getter-setter will contain a list of users
//假设响应数据是这样: `{data: [{name: "John"}, {name: "Mary"}], count: 2}`
//当完成的时候 (比如在一个视图里),`users` getter-setter会包含一个user列表
//i.e. 即 users() //[{name: "John"}, {name: "Mary"}]
```

---

### Using Different Data Transfer Formats

By default, `m.request` uses JSON to send and receive data to web services. You can override this by providing `serialize` and `deserialize` options:

### 使用不同的数据格式

默认情况下，`m.request`使用JSON格式来与服务器收发数据。你可以通过设置`serialize`和`deserialize`选项来重写序列化及反序列化函数。

```javascript
var users = m.request({
	method: "GET",
	url: "/user",
	serialize: mySerializer,
	deserialize: myDeserializer
});
```

One typical way to override this is to receive as-is responses. The example below shows how to receive a plain string from a txt file.

一个较为常见的案例是通过重写来接收一个纯文本的响应。下面的示例展示了如何从txt文件中获取一个普通的字符串。

```javascript
var file = m.request({
	method: "GET",
	url: "myfile.txt",
	deserialize: function(value) {return value;}
});
```

