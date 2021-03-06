## 组件

### 部件化

组件是 Mithril [层级 MVC](http://en.wikipedia.org/wiki/Hierarchical_model%E2%80%93view%E2%80%93controller) 的机制。

在 Mithril 中，[模块](mithril.module.md) 就是组件。在组件之间不需要交叉通信的场景下（比如，一个 dashboard 中充满着各种互不相关的部件），通常使用体积大的 controller 会比较方便（即：controller 来持有状态和方法）

以下是一个这样的组件的层级结构例子：

```javascript
//"根"模块
var dashboard = {};

dashboard.controller = function() {
	this.userProfile = new userProfile.controller();
	this.projectList = new projectList.controller();
}

dashboard.view = function(ctrl) {
	return [
		userProfile.view(ctrl.userProfile)
		projectList.view(ctrl.projectList)
	]
}
```

在上面的代码片段中，有三个模块：`dashboard`, `userProfile` 和 `projectList`。每一个子组件都可以作为一个单独的页面被渲染，但我们在这里看到它们是如何被组成成一个更大的页面的。

重要的一点是，如果你有体积大的 controller，你永远不要在一个 view 里面对一个 controller 的类进行实例化（或者调用带有这些操作的函数）。作为事件触发的结果（译者注：controller 被实例化后触发事件）， View 会重新被渲染，这样会重写子组件 controller 的状态。

---

### Divide and conquer
### 分而治之

另一个人们需要组件的原因是有些页面自身是庞大且复杂的，需要被分成小的部分，以便保持代码的可维护性。

在这种情况下，组件之间通常需要频繁的通信，且经常是以意外的方式进行。出于互联性的需要，使用体积大的 controller 的模式并不适合。

管理这种类型的组件的最好方法是将 controller 中的代码移到 view-model 中。

一个 view-model 可以被看做是一种特殊类型的 model 实体。你也许熟悉作为 ORM 类，对应数据表的 model 实体的概念，但实际上，model 层是一个抽象的区域，你应该将所有和数据以及数据相关的业务逻辑都放在这个区域中。

根据定义，view-model 是持有关于应用状态的实体。例如，哪个标签页是打开的，哪些过滤条件正在应用到表格中，一个可重置的输入框的当前值，等等。这样的数据一般不会对应 ORM schema，因为它是和 UI 相关的，不是规范化的数据。

将一个大的 controller 重构成 view-model 给予了数据更好的可达性，同时提供了一个可扩展的结构，以管理非 ORM 状态，以及限定它们的作用域。

下面的例子说明了我们可以从一个基于大 controller 的代码迁移到小的 controller。假设我们有一个展示用户列表的模块：

```javascript
var userList = {}

userList.controller = function() {
	this.users = m.request({method: "GET", url: "/users"})
}

userList.view = function(ctrl) {
	return ctrl.users().map(function(user) {
		return user.name
	})
}
```

你可以看到，controller 持有用户列表的状态。这里的问题是，当在一个大的组件层级中，任何的 view 都很难找到这个特定的 controller 实例，因此很难获得其数据以及调用其中的方法。

```javascript
var userList = {}

userList.controller = function() {
	userList.vm.init()
}

userList.vm = {}
userList.vm.init = function() {
	this.users = m.request({method: "GET", url: "/users"})
}

userList.view = function() {
	return userList.vm.users().map(function(user) {
		return user.name
	})
}
```

这个模式使我们可以在 view model 从应用代码的任何位置得到 userList 模块的数据，前提是 view model 已经被实例化。注意到重构大的 controller 是简单的：我们将 controller 函数的函数体移到了 view model 的 `init` 函数中，以及在 view 中修改对于 controller 的引用。

接下来可以推广这个模式，建立 view-model 依赖树：

```javascript
userList.controller = function() {
    //这里我们指定，这个组件需要一个 `search` 的 view-model
	userList.vm.init()
	search.vm.init()
}
```

这样，我们能够保证所需的 view-model 的所有的数据和方法在这个组件的任何地方都是可获得的，尽管组件有多个子 view 和 view-model。

你也许已经注意到，我们只是简单的将一个组件分成了更小的部分，而没有为每个部分提供 controller。你要知道这些部分不是 Mithril 的模块（因为它们没有 `view` 和 `controller` 函数），所以它们不是组件。

检查是否有 controller 作为一个功能单位，可以给你一个简单的方法来判断你的分段代码仅仅是一个更大的组件的一部分，还是它确实是一个模块，其本身就是一个可重用的组件。

如果我们决定一个功能单元确实是一个可重用的组件，我们只需要给它添加一个 controller，这样它就遵循了模块的接口。

```javascript
//假设我们在 `search` 中已经有一个 view 了，为 `search` 添加一个 controller 使我们可以像使用一个独立的组件一样使用 `search`
search.controller = function() {
	search.vm.init()
}

userList.controller = function() {
	userList.vm.init()
	
    //controller 封装了其负责的作用域
	new search.controller()
}
```

强烈推荐你使用小 controller 加 view-model 的这种模式。将逻辑从大 controller 移到 model 层会使你的代码结构更加接近原本的 MVC 模式（在 MVC 模式中，controller 仅用于告诉 view 在给定上下文中可能执行哪些动作），从长远看，还可以有效的减少模块间交叉通信的复杂度。

#### 作用域到命名空间

Sometimes you might find that organizing code into various namespaces results in repetitive declarations of the namespace.

有时候你会发现，将代码组织在不同的命名空间下会导致重复声明命名空间。

```javascript
//重复地声明命名空间
myApp.users.index.controller = function() {/*...*/}

myApp.users.index.vm = {/*...*/}

myApp.users.index.view = function() {
	return myApp.users.index.vm.something
}
```

然而（在 Mithril 中）并没有规定你应该如何组织你的代码，而命名空间通常可以用简单的 Javascript 技巧来完成，你可以用简单的 Javascript 模式来为一个长的命名空间取别名，减少打字的次数：

```javascript
!function() {
	var module = myApp.users.index = {}

	module.vm = {/*...*/}

	module.controller = function() {/*...*/}

	module.view = function() {
		var vm = module.vm
		
		return vm.something
	}
}()
```

---

### 使用函数库

应用程序通常需要可重用的 UI 控件（不是由 HTML 提供的）。让我们来逐步了解如何来实现这样一个 UI 控件。在这里例子中，我们会创建一个非常简单的 autocompleter 控件。

在一开始我们可以把控件构造成单件模块，和我们在之前章节中做的组件一样。以下是实现代码：

```javascript
var autocompleter = {}
autocompleter.vm = {
	term: m.prop(""),
	filter: function(item) {
		return autocompleter.vm.term() && item.name.toLowerCase().indexOf(autocompleter.vm.term().toLowerCase()) > -1
	}
}
autocompleter.view = function(ctrl) {
	var vm = autocompleter.vm
	return [
		m("div", [
			m("input", {oninput: m.withAttr("value", vm.term), value: vm.term})
		]),
		ctrl.data().filter(vm.filter).map(function(item) {
			return m("div", {onclick: ctrl.binds.bind(this, item)}, item.name);
		})
	];
}
```

在我们之前的例子中，我们将逻辑和 UI 状态放置在 view model 实体中，在我们的 view 中使用。`<input>` 通过一个绑定来更新 `term` 的值，`filter` 函数用于裁切显示出来的数据条目，根据条目是否和用户输入的内容相关。

按现在这样，这个组件的问题在于这个模块和它的 view-model 是单件，所以组件在一个页面中只能被使用一次。幸好这很容易解决：我们可以把整个过程放置在一个工厂函数中。

```javascript
var autocompleter = function() {
	var autocompleter = {}
	autocompleter.vm = {
		term: m.prop(""),
		search: function(value) {
			autocompleter.vm.term(value.toLowerCase())
		},
		filter: function(item) {
			return autocompleter.vm.term() && item.name.toLowerCase().indexOf(autocompleter.vm.term()) > -1
		}
	}
	autocompleter.view = function(ctrl) {
		return [
			m("div", [
				m("input", {oninput: m.withAttr("value", autocompleter.vm.search)})
			]),
			ctrl.data().filter(autocompleter.vm.filter).map(function(item) {
				return m("div", {onclick: ctrl.binds.bind(this, item)}, item.name);
			})
		];
	}
	return autocompleter
}
```

你可以看到，代码和之前的一样，除了原来的代码被包裹在了一个函数中，函数将模块作为返回值。这使我们可以轻松的创建 autocompleter 控件。

```javascript
//这是一个使用 autocompleter 的例子
var dashboard = {}
dashboard.controller = function() {
	dashboard.vm.init()
}
dashboard.vm = {}
dashboard.vm.init = function() {
	this.users = m.prop([{id: 1, name: "John"}, {id: 2, name: "Bob"}, {id: 2, name: "Mary"}]);
	this.selectedUser = m.prop()
	this.userAC = new autocompleter()
	
	this.projects = m.prop([{id: 1, name: "John's project"}, {id: 2, name: "Bob's project"}, {id: 2, name: "Mary's project"}]);
	this.selectedProject = m.prop()
	this.projectAC = new autocompleter()
};

dashboard.view = function() {
	var vm = dashboard.vm
	return m("div", [
		vm.userAC.view({data: vm.users, binds: vm.selectedUser}),
		vm.projectAC.view({data: vm.projects, binds: vm.selectedProject}),
	]);
};

//初始化
m.module(document.body, dashboard);
```

在上述的使用例子中，我们创建了一个最顶层的 `dashboard` 模块，还对两个 `autocompleter` 模块进行了初始化，连同一些填入 autocompleter 的数据，以及绑定数据的 getter-setter。

