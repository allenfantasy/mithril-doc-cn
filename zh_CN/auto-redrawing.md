## The Auto-Redrawing System
## 自动重绘系统（The Auto-Redrawing System）

Mithril is designed around the principle that data always flows from the model to the view. This makes it easy to reason about the state of the UI and to test it. In order to implement this principle, the rendering engine must run a redraw algorithm globally to ensure no parts of the UI are out of sync with the data. While at first glance, it may seem expensive to run a global redraw every time data changes, Mithril makes it possible to do this efficiently thanks to its fast diffing algorithm, which only updates the DOM where it needs to be updated. Because the DOM is by far the largest bottleneck in rendering engines, Mithril's approach of running a diff against a virtual representation of the DOM and only batching changes to the real DOM as needed is surprisingly performant.

In addition, Mithril attempts to intelligently redraw only when it is appropriate in an application lifecycle. Most frameworks redraw aggressively and err on the side of redrawing too many times because, as it turns out, determining the best time to do a redraw is quite complicated if we want to be as efficient as possible.

Mithril 基于以下原则来设计：数据总是从 model 向 view 流动。这样一来，理解和测试 UI 状态就变得简单了。为了实行这个原则，渲染引擎（rendering engine）必须在全局执行一个重绘算法，来保证所有的 UI 和数据保持同步。乍一看，在每次数据变化时执行全局的重绘显得很昂贵，但 Mithril 通过它的快速的 diff 算法让高效执行这个步骤成为可能。Mithril 的 diff 算法只在需要更新 DOM 时才对其进行更新。因为 DOM 是目前为止渲染引擎最大的瓶颈，Mithril 的方法令人惊讶的高效：通过对 DOM 的一个虚拟表示进行 diff 操作，并对 DOM 的（写）操作进行批量处理。

此外，Mithril 尝试在应用的生命周期中智能地只在合适的时间进行重绘。大多数的框架都积极地重绘，并因过多的重绘而犯错，因为事实证明，判断重绘的最好时间是非常复杂的，如果我们希望尽可能地高效的话。

Mithril employs a variety of mechanisms to decide the best time and the best strategy to redraw. By default, Mithril is configured to auto-redraw from scratch after module controllers are initialized, and it is configured to diff after event handlers are triggered. In addition, it's possible for non-Mithril asynchronous callbacks to trigger auto-redrawing by calling `m.startComputation` and `m.endComputation` in appropriate places (see below). Any code that is between a `m.startComputation` and its respective `m.endComputation` call is said to live in the *context* of its respective pair of function calls.

It's possible to defer a redraw by calling `m.request` or by manually nesting [`m.startComputation` and `m.endComputation`](mithril.computation.md) contexts. The way the redrawing engine defers redrawing is by keeping an internal counter that is incremented by `m.startComputation` and decremented by `m.endComputation`. Once that counter reaches zero, Mithril redraws. By strategically placing calls to this pair of functions, it is possible to stack asynchronous data services in any number of ways within a context without the need to pass state variables around the entire application. The end result is that you can call `m.request` and other integrated data services seamlessly, and Mithril will wait for all of the asynchronous operations to complete before attempting to redraw.

Mithril 使用各种各样的机制来决定重绘的最佳时间和策略。默认地，Mithril 的配置是在模块的 controller 实例化后从头进行自动重绘，在事件函数被触发后执行 diff 操作。此外，可以通过在合适的地方调用 `m.startComputation` 和 `m.endComputation` ，在非 Mithril 的异步回调函数中触发 Mithril 的自动重绘（见下文）。任何在 `m.startComputation` 和其对应的 `m.endComputation` 之间的代码被称为：生存在其相对应的一对函数调用的 *上下文* 中。

延迟一个重绘是可能的：调用 `m.request` 或者手动地嵌套 [`m.startComputation` and `m.endComputation`](mithril.computation.md) 的上下文。渲染引擎延迟重绘的方法是保留一个内部的计数器，在 `m.startComputation` 时增加，在 `m.endComputation` 时减少。当计数器的值为 0 时，Mithril 执行重绘。通过有策略地放置这些函数对（译者注：指 m.startComputation 和 m.endComputation）的调用位置，可以在一个上下文中以任意形式和数量堆叠异步的数据服务，而不需要在应用中传递状态变量。最终结果是，你可以无缝的调用 `m.request` 或者其他整合的数据服务，Mithril 会在尝试重绘前，等待所有的异步操作完成。

In addition to being aware of data availability when deciding to redraw, Mithril is also aware of browser availability: if several redraws are triggered in a short amount of time, Mithril batches them so that at most only one redraw happens within a single animation frame (around 16ms). Since computer screens are not able to display changes faster than a frame, this optimization saves CPU cycles and helps UIs stay responsive even in the face of spammy data changes.

Mithril also provides several hooks to control its redrawing behavior with a deep level of granularity: [`m.startComputation` and `m.endComputation`](mithril.computation.md) create redrawable contexts. [`m.redraw`](mithril.redraw.md) forces a redraw to happen in the next available frame (or optionally, it can redraw immediately for synchronous processing). [`m.redraw.strategy`](mithril.redraw.md#strategy) can change the way Mithril runs the next scheduled redraw. Finally, the low-level [`m.render`](mithril.render.md) can also be used if a developer chooses to opt out of rest of the framework altogether.

除了在决定重绘时考虑到数据的可达性之外，Mithril 还考虑到了浏览器：如果在短时间内出现了多次重绘，Mithril 将对它们集中起来处理，这样在一次 animation frame 中（约 16 毫秒），只有一次重绘。由于电脑屏幕的显示速率不能比一帧（的速率）快，这样的优化节约了 CPU 的周期，帮助 UI 在大量的数据变化时保持响应度。

Mithril 同时提供若干个 hook 在更深层的一个粒度来控制其重绘行为：[`m.startComputation` and `m.endComputation`](mithril.computation.md) 创建可重绘的上下文。[`m.redraw`](mithril.redraw.md) 强制在下一帧（或者立即进行重绘，这是可选的）执行一个重绘。 [`m.redraw.strategy`](mithril.redraw.md#strategy) 可以改变 Mithril 执行下一次计划中的重绘的方式。最后，底层的 [`m.render`](mithril.render.md) 也可以用，如果开发者选择将框架的所有其他部分排除在外的话.（原文：Finally, the low-level m.render can also be used if a developer chooses to opt out of rest of the framework altogether）

---

### Integrating with The Auto-Redrawing System
### 和自动重绘系统整合

If you need to do custom asynchronous calls without using Mithril's API, and find that your views are not redrawing, or that you're being forced to call [`m.redraw`](mithril.redraw.md) manually, you should consider using `m.startComputation` / `m.endComputation` so that Mithril can intelligently auto-redraw once your custom code finishes running.

In order to integrate asynchronous code to Mithril's autoredrawing system, you should call `m.startComputation` BEFORE making an asynchronous call, and `m.endComputation` at the end of the asynchronous callback.

如果你在不使用 Mithril 的 API 的情况下做特定的异步调用时发现你的 view 没有重绘，或者发现你被迫要手动调用 [`m.redraw`](mithril.redraw.md)，你就要考虑在一个异步调用 **之前** 使用 `m.startComputation`，在异步回调函数的最后调用 `m.endComputation`。

```javascript
//this service waits 1 second, logs "hello" and then notifies the view that
//it may start redrawing (if no other asynchronous operations are pending)
//这个服务等待 1 秒钟，打一个 "hello" 的 log，然后通知 view:
//它可以开始重绘了（如果没有其他的异步操作在等待执行的话）
var doStuff = function() {
	m.startComputation(); //call `startComputation` before the asynchronous `setTimeout`
	//在异步的 `setTimetout` 前调用 `startComputation`

	setTimeout(function() {
		console.log("hello");

		m.endComputation(); //call `endComputation` at the end of the callback
	}, 1000);
	// 在回调的最后调用 `endComputation`
};
```

To integrate synchronous code, call `m.startComputation` at the beginning of the method, and `m.endComputation` at the end.

和同步的代码进行整合时，在调用方法之前，调用 `m.startComputation`，在方法结束之后调用 `m.endComputation`。

```javascript
window.onfocus = function() {
	m.startComputation(); //call before everything else in the event handler
	// 在事件函数中的任何事情发生之前调用
	doStuff();

	m.endComputation(); //call after everything else in the event handler
	// 在事件函数中的任何事情发生之后调用
}
```

For each `m.startComputation` call a library makes, it MUST also make one and ONLY one corresponding `m.endComputation` call.

You should not use these methods if your code is intended to run repeatedly (e.g. by using `setInterval`). If you want to repeatedly redraw the view without necessarily waiting for user input, you should manually call [`m.redraw`](mithril.redraw.md) within the repeatable context.

对于每个 `m.startComputation` 的调用，都要有一个对应的 `m.endComputation` 的调用。

如果你的代码是要重复执行的（例如使用 `setInterval`），那你不应该用这些方法。如果你想重复地重绘 view 同时不等待用户的输入，你要手动地在每个重复的上下文中调用 [`m.redraw`](mithril.redraw.md)。

---

### Integrating multiple execution threads

### 和多个执行线程整合

When [integrating with third party libraries](integration.md), you might find that you need to call asynchronous methods from outside of Mithril's API.

In order to integrate non-trivial asynchronous code with Mithril's auto-redrawing system, you need to ensure all execution threads call `m.startComputation` / `m.endComputation`.

An execution thread is basically any amount of code that runs before other asynchronous threads start to run.

Integrating multiple execution threads can be done in two different ways: in a layered fashion or in comprehensive fashion.

当 [和第三方库整合](integration.md) 时，你会发现你需要调用 Mithril API 以外的异步方法。

为了将这些重要的异步代码和 Mithril 的自动重绘系统整合起来，你需要保证所有的执行线程都调用了 `m.startComputation` / `m.endComputation`。

一个执行线程从根本上说是任何在其他异步线程开始执行之前执行的代码。

和多个执行线程整合可以通过两种不同的方式完成：分层的（layered）或者全面的（comprehensive）。

#### Layered integration
#### 分层的整合

Layered integration is recommended for modular code where many different APIs may be put together at the application level.

Below is an example where various methods implemented with a third party library can be integrated in layered fashion: any of the methods can be used in isolation or in combination.

Notice how `doBoth` repeatedly calls `m.startComputation` since that method calls both `doSomething` and `doAnother`. This is perfectly valid: there are three asynchronous computations pending after the `jQuery.when` method is called, and therefore, three pairs of `m.startComputation` / `m.endComputation` in play.

在许多不同的 API 放在应用层时，推荐分层整合来实现代码的模块化。

下面是一个例子。在例子中，用第三方库实现的不同方法可以用分层的方式来整合：任何的方法都可以单独使用或者组合使用。

注意到 `doBoth` 方法是如何重复地调用 `m.startComputation` 的，因为它调用了 `doSomething` 和 `doAnother`。这是完全合法的：总共有 3 个异步的计算等待着在 `jQuery.when` 方法执行（后执行），也就是，3 对 `m.startComputation` / `m.endComputation` 在起作用。

```javascript
var doSomething = function(callback) {
	m.startComputation(); //call `startComputation` before the asynchronous AJAX request
	// 在 AJAX 请求前调用 `startComputation`

	return jQuery.ajax("/something").done(function() {
		if (callback) callback();

		m.endComputation(); //call `endComputation` at the end of the callback
		// 在回调的最后调用 `endComputation`
	});
};
var doAnother = function(callback) {
	m.startComputation(); //call `startComputation` before the asynchronous AJAX request
	// 在 AJAX 请求前调用 `startComputation`

	return jQuery.ajax("/another").done(function() {
		if (callback) callback();
		m.endComputation(); //call `endComputation` at the end of the callback
		// 在回调的最后调用 `endComputation`
	});
};
var doBoth = function(callback) {
	m.startComputation(); //call `startComputation` before the asynchronous synchronization method
	// 在异步的同步方法前调用 `startComputation`

	jQuery.when(doSomething(), doAnother()).then(function() {
		if (callback) callback();

		m.endComputation(); //call `endComputation` at the end of the callback
		// 在回调的最后调用 `endComputation`
	})
};
```

#### Comprehensive integration
#### 全面的整合

Comprehensive integration is recommended if integrating a monolithic series of asynchronous operations. In contrast to layered integration, it minimizes the number of `m.startComputation` / `m.endComputation` calls to avoid clutter.

The example below shows a convoluted series of AJAX requests implemented with a third party library.

当和成为整体的一系列异步操作进行整合时，推荐使用全面的方式进行整合。和分层整合不同，它将 `m.startComputation` / `m.endComputation` 调用减到最小，以避免混乱。

以下的例子展示了一系列用第三方库实现的复杂的 AJAX 请求。

```javascript
var doSomething = function(callback) {
	m.startComputation(); //call `startComputation` before everything else
  //在其他代码开始前调用 `startComputation`

	jQuery.ajax("/something").done(function() {
		doStuff();
		jQuery.ajax("/another").done(function() {
			doMoreStuff();
			jQuery.ajax("/more").done(function() {
				if (callback) callback();

				m.endComputation(); //call `endComputation` at the end of everything
				// 在所有代码的最后调用 `endComputation`
			});
		});
	});
};
```
