## 和其他库整合

可以通过 [虚拟元素的 `config` 属性](mithril.md#accessing-the-real-dom) 和第三方的库或者纯 javascript 代码进行整合。

推荐的做法是将整合的代码封装在一个组件或者一个 helper 函数中。

下面的例子展示了一个和 [select2 库](http://ivaynberg.github.io/select2) 整合的组件。

```javascript
//Select2 组件 （假设 jQuery 和 Select2 都包含在页面内）

/** @namespace */
var select2 = {};

/**
select2 配置的工厂函数。在这个文档内的参数指的是 `ctrl` 这个形参的属性
@param {Object} data - 用来 populate <option> 列表的数据
@param {Number} value - 我们在 `data` 中选择的条目的 id
@param {function(Object id)} onchange - 当选择（输入）变化时调用的事件函数。
	`id` 和 `value` 是一样的。
*/
select2.config = function(ctrl) {
	return function(element, isInitialized) {
		var el = $(element);

		if (!isInitialized) {
			//对 select2 进行初始化（如果它还没有被初始化）
			el.select2()
				//在 view 变化时，这个事件函数更新 controller
				.on("change", function(e) {
					//和自动重绘系统整合...
					m.startComputation();

					//...这样 Mithril 会在调用 controller 的回调函数后自动重绘
					if (typeof ctrl.onchange == "function") {
						ctrl.onchange(el.select2("val"));
					}

					m.endComputation();
					//整合结束
				});
		}

		//用最新的 controller 的值更新 view
		el.select2("val", ctrl.value);
	}
}

//这个 view 实现了 select2 的 `<select>` 渐进的增量模式
select2.view = function(ctrl) {
	return m("select", {config: select2.config(ctrl)}, [
		ctrl.data.map(function(item) {
			return m("option", {value: item.id}, item.name)
		})
	]);
};

//定义组件结束


//使用
var dashboard = {};

dashboard.controller = function() {
	//要展示的用户列表
	this.data = [{id: 1, name: "John"}, {id: 2, name: "Mary"}, {id: 3, name: "Jane"}];

	//选择 Mary
	this.currentUser = this.data[1];

	this.changeUser = function(id) {
		console.log(id)
	};
}

dashboard.view = function(ctrl) {
	return m("div", [
		m("label", "User:"),
		select2.view({data: ctrl.data, value: ctrl.currentUser.id, onchange: ctrl.changeUser})
	]);
}

m.module(document.body, dashboard);
```

`select2.config` 是一个工厂函数，基于一个给定的 controller 创建一个 `config` 函数。我们在 `select2.view` 函数外声明这个工厂函数，以避免使模板变得混乱。

用我们的工厂函数创建的 `config` 函数只在还没有执行初始化时才执行初始化的代码。这个 `if` 语句很重要，因为这个函数可能会因为 Mithril 的自动重绘系统而被多次调用，而我们并不想重新对 select2 在每次重绘时都进行初始化。

初始化的代码定义了一个 `change` 事件函数。因为这个函数不是用 Mithril 的模板引擎创建的（即：我们不在一个虚拟元素里定义属性），我们必须在自动重绘系统中手动的整合它。

这可以通过简单的在开头调用 `m.startComputation`，在函数的最后调用 `m.endComputation` 来完成。你必须为每次异步的执行线程添加一对这样的函数调用，除非这个线程已经整合过了。

例如，如果你要用 `m.request` 来调用一个 web 服务，你不需要添加额外的 `m.startComputation` / `m.endComputation` 调用（尽管，在事件函数中你还是需要第一对函数调用）。

另一方面，如果你要用 jQuery 来调用一个 web 服务，那你就要负责在调用 jQuery 的 ajax 前，添加一个对 `m.startComputation` 的调用，以及在（ajax 的）完成后的回调函数的最后添加一个对 `m.endComputation` 的调用，以及在 `change` 事件函数中添加一对调用。参考 [`auto-redrawing`](auto-redrawing.md) 指南中给出的例子。

一个关于 `config` 方法的重要注意事项是：你要避免在 `config` 函数的执行线程中，调用 `m.redraw`，`m.startComputation` 和 `m.endComputation`（执行线程基本上是：任何在其他异步线程开始执行之前执行的代码）。

尽管从技术上 Mithril 支持这样的用法，但依靠多次的重绘传递会降低性能，且使得你掉入无限循环的代码执行中，这是非常难 debug 的。

例子中的 `dashboard` 模块展示了一个开发者如何使用 select2 组件。

你总是应该对整合（第三方库）的组件加文档注释，这样其他（开发者）可以知道什么属性参数可以在初始化组件时使用。

---

## 和旧代码整合

如果你需要在一个页面中的多个地方添加分散的组件，你可以简单的对每个组件进行初始化，像你写一个正常的 Mithril 应用一样（即：使用 `m.render`，`m.module` 和 `m.route`）。

只有一个地方要注意：当对多个"岛"进行初始化时，它们的初始化调用是互相不知道对方的存在的，所以可能会造成过于频繁的重绘。为了对渲染进行优化，你应该在每个执行线程中的第一个组件初始化前，添加一个 `m.startComputation` 调用，以及在最后一个组件初始化后，添加一个 `m.endComputation` 调用。如下面的例子：

```javascript
m.startComputation()

m.module(document.getElementById("widget1-container"), widget1)

m.module(document.getElementById("widget2-container"), widget1)

m.endComputation()
```
