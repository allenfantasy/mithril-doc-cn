## 优化性能

在某些少有的情况如页面因为其自身原因变得极其复杂的时候，有一些方法可以提高 Mithril 的性能。

首先也是最重要的，你应该先认真考虑性能优化是不是真的是你的最后一招。从本性上说（by nature），对性能进行优化会接受一些激进的假设，可能会在边界情况下崩溃，同时会使得代码变得难以理解。依据经验，在存在其他解决（问题）的方法的时候，你 *千万不要* 对性能进行优化。

例如，如果你在构建一个有几千行的表格时，发现模板（的速度）很慢，你首先应当考虑减少表格显示的条目，因为从用户体验的角度，没有人真的会去看几千行记录。通过给表格进行分页，提供搜索和过滤功能，可以提高用户的体验，同时解决在重绘或者页面初始化时性能的问题。

---

## 编译模板

你也可以有选择地对使用了 `m()` 方法的模板进行预编译：利用 [Sweet.js](https://github.com/mozilla/sweet.js) 运行 [`template-compiler.sjs`](tools/template-compiler.sjs) 的宏。在使用 Mithril 时，这个步骤不是必须的，但它是一种可以从应用中“挤出”多一点点性能的简单方法，同时不需要修改代码。

编译一个模板会将模板内的嵌套函数调用转换为一个原始的虚拟 DOM 树（这仅仅是一个原生的 Javascript 对象的集合，它们会通过 [`m.render`](mithril.render.md) 来渲染。这意味着编译后的模板不需要解析 `m("div#foo")` 中的字符串，也不会引发函数调用的花销。

值得一提的是，Mithril 在其他地方有内建的机制来处理真正的瓶颈，如浏览器重绘管理和 DOM 的更新。这个可选的编译工具仅仅是锦上添花，来加速 Javascript 模板的运行速度（其实在没有编译时已经很快了 - 详情请看 [首页的性能章节](http://lhorie.github.io/mithril/index.html#performance)）。

宏处理普通的 Mithril 模板，如下面这个例子：

```javascript
var view = function() {
	return m("a", {href: "http://google.com"}, "test");
}
```

它对 `m()` 方法的调用进行预处理，用以下输出代替之：

```javascript
var view = function() {
	return {tag: "a", attrs: {href: "http://google.com"}, children: "test"};
}
```

需要注意的是，编译后的模板应该是通过一个自动构建的流程生成，模板也不应该被手动修改。

---

### 安装 NodeJS 和 SweetJS 来进行一次性地编译

SweetJS 需要一个 [NodeJS](http://nodejs.org) 环境。可以访问其官网，使用官网提供的安装程序来安装 NodeJS。

NodeJS 提供了一个命令行的包管理工具来安装 SweetJS。在终端中，输入：

```
npm install -g sweet.js
```

要编译一个文件，可以输入：

```
sjs -r -m /template-compiler.sjs -o <output-filename>.js <input-filename>.js
```

---

### Automating Compilation

### 将编译工作自动化

如果你希望将编译工作自动化，你可以使用 [GruntJS](http://gruntjs.com)，一个任务自动化工具。如果你对 GruntJS 不大熟悉，你可以在它们的网站上找到一个教程。

假设你已经安装了 NodeJS，执行以下命令来安装 GruntJS：

```
npm install -g grunt-cli
```

当安装完成后，在你的项目根目录中新建两个文件：`package.json` 和 `Gruntfile.js`

`package.json`

```javascript
{
	"name": "project-name-goes-here",
	"version": "0.0.0", // 必须遵循这个格式
	"devDependencies": {
		"grunt-sweet.js": "*"
	}
}
```

`Gruntfile.js`

```javascript
module.exports = function(grunt) {
	grunt.initConfig({
		sweetjs: {
			modules: ["template-compiler.sjs"],
			compile: {expand: true, cwd: ".", src: "**/*.js", dest: "destination-folder-goes-here/"}
		}
	});

	grunt.loadNpmTasks('grunt-sweet.js');

	grunt.registerTask('default', ['sweetjs']);
}
```

要确保将 `project-name-goes-here` 和 `destination-folder-goes-here` 这两个占位符用实际的项目名称和项目目录路径来代替。

在你的项目的根目录下执行以下命令来自动执行任务：

```
grunt
```

更多关于 grunt-sweet.js 的任务和其选项 [可以在这里查阅](https://github.com/natefaubion/grunt-sweet.js)
