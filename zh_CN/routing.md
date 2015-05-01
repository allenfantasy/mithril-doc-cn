## 路由

路由系统允许使用者构建单页应用，即可以在不完全刷新浏览器的情况下从一个页面到另一个页面的应用。

它可以无缝地在页面内导航，同时保留了对每个页面单独作为书签的能力，以及通过浏览器的历史机制在应用内导航的能力。（原文：It enables seamless navigability while preserving the ability to bookmark each page individually, and the ability to navigate the application via the browser's history mechanism）

Mithril 提供了处理三种路由情况的工具：

- 定义一组路由
- 通过代码在路由间进行跳转
- 让模板中的链接透明且不显眼地对应某个路由（原文：making links in templates routed transparently and unobstrusively）

---

### 定义路由

定义一个路由列表，你需要指定一个主的DOM元素，一个默认的路由，和所有路由和对应呈现的[模块](mithril.module.md)的键值映射表。

下面的例子定义了三个路由，呈现的内容在`<body>`标签下，`home`，`login`和`dashboard`是模块。我们很快将看到如何定义一个模块。

```javascript
m.route(document.body, "/", {
	"/": home,
	"/login": login,
	"/dashboard": dashboard,
});
```

通过在单词前面添加一个冒号`:`，路由可以接受参数。

下面的例子展示了一个接受`userID`参数的路由。

```javascript
//一个模块例子
var dashboard = {
	controller: function() {
		this.id = m.route.param("userID");
	},
	view: function(controller) {
		return m("div", controller.id);
	}
}

//设置路由在开头带/不带 `#` 符号
m.route.mode = "hash";

//定义一个路由
m.route(document.body, "/dashboard/johndoe", {
	"/dashboard/:userID": dashboard
});
```

这个路由会重定向到 URL `http://server/#/dashboard/johndoe`，结果为：

```markup
<body>johndoe</body>
```

在以上例子中，`dashboard` 是一个模块。它包含 `controller` 和 `view` 属性。当 URL 符合一个路由（的规则）时，相应的模块的 controller 会被实例化，并作为一个参数传递给 view。

在这个例子中，由于只有一个路由，应用将重定向到默认的路由 `"/dashboard/johndoe"`，在内部它调用 `m.module(document.body, dashboard)`。

字符串 `johndoe` 被绑定到 `:userID` 参数上，它可以在 controller 中通过 `m.route.param("userID")` 的方式获得。

`m.route.mode` 属性定义了用 URL 的哪个部分来实现路由机制。它的值可以设为 "search", "hash" 或者是 "pathname"。默认的值是 "search"。

- `search` 模式用查询字符串。它允许带名字的锚节点（即 `<a href="#top">Back to top</a>`, `<a name="top"></a>`）在页面上生效。但在 IE 8 中路由的变化会导致页面的刷新，因为 IE 8 对 `history.pushState` 不支持。

    URL 例子：`http://server/?/path/to/page`

- `hash` 模式使用 hash（译者注：井号 #）。这是唯一的，在所有浏览器上都不会在路由改变时刷新页面的模式。但是，这种模式不支持带名字的锚节点，以及浏览器的历史列表。

    URL 例子：`http://server/#/path/to/page`

- `pathname` 模式允许对不带任何特殊字符的 URL 进行路由，但这个模式要求服务器端的设置，以支持书签和重新加载页面。在 IE 8 中这个模式也会导致页面刷新。

    URL 例子：`http://server/path/to/page`

    最简单的支持 pathname 模式的可行的服务器配置是（对所有的请求）返回相同的内容，不管请求的 URL 是什么。在 Apache 中，这样的 URL 重写可以用 [mod_rewrite](https://httpd.apache.org/docs/current/mod/mod_rewrite.html) 来实现。

---

### 重定向

你可以通过代码重定向到另一个页面。我们来看下 "定义路由" 一节中的这个例子：

```javascript
m.route("/dashboard/marysue");
```

这个例子将页面重定向到 `http://server/#/dashboard/marysue`

---

### 模式分离

这个方法要和 virtual element 的 `config` 属性一起使用。例如：

```javascript
//注意到这里因为有 `config` 设置，在 `href` 中不需要加 '#'，
m("a[href='/dashboard/alicesmith']", {config: m.route});
```

这样不管 `m.route.mode` 的值是什么， href 属性都会是正确的。好的实践是：总是使用以上的用法，而不是在 href 属性中硬编码 `?` 或者 `#`。

查看 [`m()`](mithril.md) 了解关于 virtual elements 的更多信息。