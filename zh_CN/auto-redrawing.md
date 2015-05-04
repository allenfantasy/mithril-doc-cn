## 自动重绘系统（The Auto-Redrawing System）

Mithril 基于以下原则来设计：数据总是从 model 向 view 流动。这样一来，理解和测试 UI 状态就变得简单了。为了实行这个原则，渲染引擎（rendering engine）必须在全局执行一个重绘算法，来保证所有的 UI 和数据保持同步。乍一看，在每次数据变化时执行全局的重绘显得很昂贵，但 Mithril 通过它的快速的 diff 算法让高效执行这个步骤成为可能。Mithril 的 diff 算法只在需要更新 DOM 时才对其进行更新。因为 DOM 是目前为止渲染引擎最大的瓶颈，Mithril 的方法令人惊讶的高效：通过对 DOM 的一个虚拟表示进行 diff 操作，并对 DOM 的（写）操作进行批量处理。

此外，Mithril 尝试在应用的生命周期中智能地只在合适的时间进行重绘。大多数的框架都积极地重绘，并因过多的重绘而犯错，因为事实证明，判断重绘的最好时间是非常复杂的，如果我们希望尽可能地高效的话。

Mithril 使用各种各样的机制来决定重绘的最佳时间和策略。默认地，Mithril 的配置是在模块的 controller 实例化后从头进行自动重绘，在事件函数被触发后执行 diff 操作。此外，可以通过在合适的地方调用 `m.startComputation` 和 `m.endComputation` ，在非 Mithril 的异步回调函数中触发 Mithril 的自动重绘（见下文）。任何在 `m.startComputation` 和其对应的 `m.endComputation` 之间的代码被称为：生存在其相对应的一对函数调用的 *上下文* 中。

延迟一个重绘是可能的：调用 `m.request` 或者手动地嵌套 [`m.startComputation` and `m.endComputation`](mithril.computation.md) 的上下文。渲染引擎延迟重绘的方法是保留一个内部的计数器，在 `m.startComputation` 时增加，在 `m.endComputation` 时减少。当计数器的值为 0 时，Mithril 执行重绘。通过有策略地放置这些函数对（译者注：指 m.startComputation 和 m.endComputation）的调用位置，可以在一个上下文中以任意形式和数量堆叠异步的数据服务，而不需要在应用中传递状态变量。最终结果是，你可以无缝的调用 `m.request` 或者其他整合的数据服务，Mithril 会在尝试重绘前，等待所有的异步操作完成。

除了在决定重绘时考虑到数据的可达性之外，Mithril 还考虑到了浏览器：如果在短时间内出现了多次重绘，Mithril 将对它们集中起来处理，这样在一次 animation frame 中（约 16 毫秒），只有一次重绘。由于电脑屏幕的显示速率不能比一帧（的速率）快，这样的优化节约了 CPU 的周期，帮助 UI 在大量的数据变化时保持响应度。

Mithril 同时提供若干个 hook 在更深层的一个粒度来控制其重绘行为：[`m.startComputation` and `m.endComputation`](mithril.computation.md) 创建可重绘的上下文。[`m.redraw`](mithril.redraw.md) 强制在下一帧（或者立即进行重绘，这是可选的）执行一个重绘。 [`m.redraw.strategy`](mithril.redraw.md#strategy) 可以改变 Mithril 执行下一次计划中的重绘的方式。最后，底层的 [`m.render`](mithril.render.md) 也可以用，如果开发者选择将框架的所有其他部分排除在外的话.（原文：Finally, the low-level m.render can also be used if a developer chooses to opt out of rest of the framework altogether）

---

### 和自动重绘系统整合

如果你在不使用 Mithril 的 API 的情况下做特定的异步调用时发现你的 view 没有重绘，或者发现你被迫要手动调用 [`m.redraw`](mithril.redraw.md)，你就要考虑使用 `m.startComputation` / `m.endComputation` 这样 Mithril 可以机智地在你的自定义代码完成执行后自动的重绘。

为了将异步代码和 Mithril 的自动重绘系统整合，你需要在一个异步调用 **之前** 使用 `m.startComputation`，在异步回调函数的 **最后** 调用 `m.endComputation`。

```javascript
//这个服务等待 1 秒钟，打一个 "hello" 的 log，然后通知 view:
//它可以开始重绘了（如果没有其他的异步操作在等待执行的话）
var doStuff = function() {
	m.startComputation(); // 在异步的 `setTimetout` 前调用 `startComputation`

	setTimeout(function() {
		console.log("hello");

		m.endComputation(); // 在回调的最后调用 `endComputation`
	}, 1000);
};
```

和同步的代码进行整合时，在调用方法之前，调用 `m.startComputation`，在方法结束之后调用 `m.endComputation`。

```javascript
window.onfocus = function() {
	m.startComputation(); // 在事件函数中的任何事情发生之前调用
	doStuff();

	m.endComputation(); // 在事件函数中的任何事情发生之后调用
}
```

对于每个 `m.startComputation` 的调用，都要有一个对应的 `m.endComputation` 的调用。

如果你的代码是要重复执行的（例如使用 `setInterval`），那你不应该用这些方法。如果你想重复地重绘 view 同时不等待用户的输入，你要手动地在每个重复的上下文中调用 [`m.redraw`](mithril.redraw.md)。

---

### 和多个执行线程整合

当 [和第三方库整合](integration.md) 时，你会发现你需要调用 Mithril API 以外的异步方法。

为了将这些重要的异步代码和 Mithril 的自动重绘系统整合起来，你需要保证所有的执行线程都调用了 `m.startComputation` / `m.endComputation`。

一个执行线程从根本上说是任何在其他异步线程开始执行之前执行的代码。

和多个执行线程整合可以通过两种不同的方式完成：分层的（layered）或者全面的（comprehensive）。

#### 分层的整合

在许多不同的 API 放在应用层时，推荐分层整合来实现代码的模块化。

下面是一个例子。在例子中，用第三方库实现的不同方法可以用分层的方式来整合：任何的方法都可以单独使用或者组合使用。

注意到 `doBoth` 方法是如何重复地调用 `m.startComputation` 的，因为它调用了 `doSomething` 和 `doAnother`。这是完全合法的：总共有 3 个异步的计算等待着在 `jQuery.when` 方法执行（后执行），也就是，3 对 `m.startComputation` / `m.endComputation` 在起作用。

```javascript
var doSomething = function(callback) {
	m.startComputation(); // 在 AJAX 请求前调用 `startComputation`

	return jQuery.ajax("/something").done(function() {
		if (callback) callback();

		m.endComputation(); // 在回调的最后调用 `endComputation`
	});
};
var doAnother = function(callback) {
	m.startComputation(); // 在 AJAX 请求前调用 `startComputation`

	return jQuery.ajax("/another").done(function() {
		if (callback) callback();
		m.endComputation(); // 在回调的最后调用 `endComputation`
	});
};
var doBoth = function(callback) {
	m.startComputation(); // 在异步的同步方法前调用 `startComputation`

	jQuery.when(doSomething(), doAnother()).then(function() {
		if (callback) callback();

		m.endComputation(); // 在回调的最后调用 `endComputation`
	})
};
```

#### 全面的整合

当和成为整体的一系列异步操作进行整合时，推荐使用全面的方式进行整合。和分层整合不同，它将 `m.startComputation` / `m.endComputation` 调用减到最小，以避免混乱。

以下的例子展示了一系列用第三方库实现的复杂的 AJAX 请求。

```javascript
var doSomething = function(callback) {
	m.startComputation();  //在其他代码开始前调用 `startComputation`

	jQuery.ajax("/something").done(function() {
		doStuff();
		jQuery.ajax("/another").done(function() {
			doMoreStuff();
			jQuery.ajax("/more").done(function() {
				if (callback) callback();

				m.endComputation(); // 在所有代码的最后调用 `endComputation`
			});
		});
	});
};
```
