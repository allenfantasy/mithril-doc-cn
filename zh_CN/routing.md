## Routing
## 路由

Routing is a system that allows creating Single-Page-Applications (SPA), i.e. applications that can go from one page to another without causing a full browser refresh.

It enables seamless navigability while preserving the ability to bookmark each page individually, and the ability to navigate the application via the browser's history mechanism.

Mithril provides utilities to handle three different aspect of routing:

-	defining a list of routes
-	programmatically redirecting between routes
-	making links in templates routed transparently and unobtrusively

路由系统允许使用者构建单页应用，即可以在不完全刷新浏览器的情况下从一个页面到另一个页面的应用。

它可以无缝地在页面内导航，同时保留了对每个页面单独作为书签的能力，以及通过浏览器的历史机制在应用内导航的能力。（原文：It enables seamless navigability while preserving the ability to bookmark each page individually, and the ability to navigate the application via the browser's history mechanism）

Mithril 提供了处理三种路由情况的工具：

- 定义一组路由
- 通过代码在路由间进行跳转
- 让模板中的链接透明且不显眼地对应某个路由（原文：making links in templates routed transparently and unobstrusively）

---

### Defining routes
### 定义路由

To define a list of routes, you need to specify a host DOM element, a default route and a key-value map of possible routes and respective [modules](mithril.module.md) to be rendered.

The example below defines three routes, to be rendered in `<body>`. `home`, `login` and `dashboard` are modules. We'll see how to define a module in a bit.

定义一个路由列表，你需要指定一个主的DOM元素，一个默认的路由，和所有路由和对应呈现的[模块](mithril.module.md)的键值映射表。

下面的例子定义了三个路由，呈现的内容在`<body>`标签下，`home`，`login`和`dashboard`是模块。我们很快将看到如何定义一个模块。

```javascript
m.route(document.body, "/", {
	"/": home,
	"/login": login,
	"/dashboard": dashboard,
});
```

Routes can take arguments, by prefixing words with a colon `:`.

The example below shows a route that takes a `userID` parameter.

通过在单词前面添加一个冒号`:`，路由可以接受参数。

下面的例子展示了一个接受`userID`参数的路由。

```javascript
//a sample module
//一个模块例子
var dashboard = {
	controller: function() {
		this.id = m.route.param("userID");
	},
	view: function(controller) {
		return m("div", controller.id);
	}
}

//setup routes to start w/ the `#` symbol
//设置路由在开头带/不带 `#` 符号
m.route.mode = "hash";

//define a route
//定义一个路由
m.route(document.body, "/dashboard/johndoe", {
	"/dashboard/:userID": dashboard
});
```

This redirects to the URL `http://server/#/dashboard/johndoe` and yields:

这个路由会重定向到 URL `http://server/#/dashboard/johndoe`，结果为：

```markup
<body>johndoe</body>
```

Above, `dashboard` is a module. It contains `controller` and `view` properties. When the URL matches a route, the respective module's controller is instantiated and passed as a parameter to the view.

In this case, since there's only one route, the app redirects to the default route `"/dashboard/johndoe"` and, under the hood, it calls `m.module(document.body, dashboard)`.

The string `johndoe` is bound to the `:userID` parameter, which can be retrieved programmatically in the controller via `m.route.param("userID")`.

在以上例子中，`dashboard` 是一个模块。它包含 `controller` 和 `view` 属性。当 URL 符合一个路由（的规则）时，相应的模块的 controller 会被实例化，并作为一个参数传递给 view。

在这个例子中，由于只有一个路由，应用将重定向到默认的路由 `"/dashboard/johndoe"`，在内部它调用 `m.module(document.body, dashboard)`。

字符串 `johndoe` 被绑定到 `:userID` 参数上，它可以在 controller 中通过 `m.route.param("userID")` 的方式获得。【注：为了通顺，programmatically不译】

The `m.route.mode` property defines which URL portion is used to implement the routing mechanism. Its value can be set to either "search", "hash" or "pathname". The default value is "search".

-	`search` mode uses the querystring. This allows named anchors (i.e. `<a href="#top">Back to top</a>`, `<a name="top"></a>`) to work on the page, but routing changes causes page refreshes in IE8, due to its lack of support for `history.pushState`.

	Example URL: `http://server/?/path/to/page`

-	`hash` mode uses the hash. It's the only mode in which routing changes do not cause page refreshes in any browser. However, this mode does not support named anchors and browser history lists.

	Example URL: `http://server/#/path/to/page`

-	`pathname` mode allows routing URLs that contain no special characters, however this mode requires server-side setup in order to support bookmarking and page refreshes. It also causes page refreshes in IE8.
	
	Example URL: `http://server/path/to/page`

	The simplest server-side setup possible to support pathname mode is to serve the same content regardless of what URL is requested. In Apache, this URL rewriting can be achieved using [mod_rewrite](https://httpd.apache.org/docs/current/mod/mod_rewrite.html).

`m.route.mode` 属性定义了用 URL 的哪个部分来实现路由机制。它的值可以设为 "search", "hash" 或者是 "pathname"。默认的值是 "search"。

- `search` 模式用查询字符串。它允许带名字的锚节点（即 `<a href="#top">Back to top</a>`, `<a name="top"></a>`）在页面上生效。但在 IE 8 中路由的变化会导致页面的刷新，因为 IE 8 对 `history.pushState` 不支持。

    URL 例子：`http://server/?/path/to/page`

- `hash` 模式使用 hash（译者注：井号 #）。这是唯一的，在所有浏览器上都不会在路由改变时刷新页面的模式。但是，这种模式不支持带名字的锚节点，以及浏览器的历史列表。

    URL 例子：`http://server/#/path/to/page`

- `pathname` 模式允许对不带任何特殊字符的 URL 进行路由，但这个模式要求服务器端的设置，以支持书签和重新加载页面。在 IE 8 中这个模式也会导致页面刷新。

    URL 例子：`http://server/path/to/page`

    最简单的支持 pathname 模式的可行的服务器配置是（对所有的请求）返回相同的内容，不管请求的 URL 是什么。在 Apache 中，这样的 URL 重写可以用 [mod_rewrite](https://httpd.apache.org/docs/current/mod/mod_rewrite.html) 来实现。

---

### Redirecting
### 重定向

You can programmatically redirect to another page. Given the example in the "Defining Routes" section:

你可以通过代码重定向到另一个页面。我们来看下 "定义路由" 一节中的这个例子：

```javascript
m.route("/dashboard/marysue");
```

redirects to `http://server/#/dashboard/marysue`

这个例子将页面重定向到 `http://server/#/dashboard/marysue`

---

### Mode abstraction
### 模式分离

This method is meant to be used with a virtual element's `config` attribute. For example:

这个方法要和 virtual element 的 `config` 属性一起使用。例如：

```javascript
//Note that the '#' is not required in `href`, thanks to the `config` setting.
//注意到这里因为有 `config` 设置，在 `href` 中不需要加 '#'，
m("a[href='/dashboard/alicesmith']", {config: m.route});
```

This makes the href behave correctly regardless of which `m.route.mode` is selected. It's a good practice to always use the idiom above, instead of hardcoding `?` or `#` in the href attribute.

See [`m()`](mithril.md) for more information on virtual elements.

这样不管 `m.route.mode` 的值是什么， href 属性都会是正确的。好的实践是：总是使用以上的用法，而不是在 href 属性中硬编码 `?` 或者 `#`。

查看 [`m()`](mithril.md) 了解关于 virtual elements 的更多信息。