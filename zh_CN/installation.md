## 安装

Mithril 可以通过不同的渠道获得：

---

### 直接下载

你可以 [在这里下载最新版本的zip文件](http://lhorie.github.io/mithril/mithril.min.zip)。

历史版本的链接可以在 [change log](change-log.html) 里找到。

使用 Mithril 时，将其从 zip 文件中解压出来，并用一个 script 标签指向 `.js` 文件：

```markup
<script src="mithril.min.js"></script>
```

注意，如果要支持旧版本的 IE，你需要包含 [一些 shim](tools.md#internet-explorer-compatibility)。

---

### CDNs（内容分发网络）

你还可以在 [cdnjs](http://cdnjs.com/libraries/mithril/) 和 [jsDelivr](http://www.jsdelivr.com/#!mithril) 中找到 Mithril。

不同网站会使用框架的同一个版本，内容分发网络使得代码库可以在这些网站中被缓存起来，通过在离用户地理位置较近的服务器上提供代码文件的方式，减少（网站的）延迟。

#### cdnjs

```markup
<script src="//cdnjs.cloudflare.com/ajax/libs/mithril/$version/mithril.min.js"></script>
```

#### jsDelivr

```markup
<script src="//cdn.jsdelivr.net/mithril/$version/mithril.min.js"></script>
```

---

### NPM

NPM 是 [NodeJS](http://nodejs.org/) 默认使用的包管理器。如果你已经在使用 NodeJS 或者打算使用 [Grunt](http://gruntjs.com/) 来建立一个构建系统，你可以使用 NPM 以便于保持 Mithril 版本的更新。

假设你已经装好了 NodeJS，你可以输入这行代码来下载 Mithril：

```
npm install mithril
```

然后，用一个 script 标签指向下载的文件，来使用 Mithril：

```markup
<script src="/node_modules/mithril/mithril.min.js"></script>
```

---

### Bower

[Bower](http://bower.io) 也是一个 [NodeJS](http://nodejs.org/) 的包管理器。如果你已经在使用 NodeJS 或者打算使用 [Grunt](http://gruntjs.com/) 来建立一个构建系统，你可以用 Bower 以便于保持 Mithril 版本的更新。

假设你已经装好了 NodeJS，你可以用以下指令安装 Bower：

```
npm install -g bower
```

然后你可以用以下指令安装 Mithril：

```
bower install mithril
```

然后，用一个 script 标签指向下载的文件，来使用 Mithril：

```markup
<script src="/bower_components/mithril/mithril.min.js"></script>
```

---

### Component

[Component](http://component.io) 又是另一个 [NodeJS](http://nodejs.org/) 的包管理器。如果你已经在使用 NodeJS 或者打算使用 [Grunt](http://gruntjs.com/) 来建立一个构建系统，你可以用 Component 以便于保持 Mithril 版本的更新。

假设你已经装好了 NodeJS，你可以用以下指令安装 Component：

```
npm install -g component
```

然后你可以用以下指令安装 Mithril：

```
component install lhorie/mithril
```

然后，用一个 script 标签指向下载的文件，来使用 Mithril：

```markup
<script src="/components/lhorie/mithril/master/mithril.js"></script>
```

---

### Rails

Jordan Humphreys 写了一个 gem 来将 Mithril 整合到 Rails 中：

[Mithril-Rails](https://github.com/mrsweaters/mithril-rails)

这个 gem 包括对 [MSX](https://github.com/insin/msx) 的支持，MSX 是一种 HTML 模板语法，由 Jonathan Buchanan 创建。

---

### Github

你也可以 [直接从 Github](https://github.com/lhorie/mithril) fork 最新的稳定版项目。

如果你希望使用最新的（开发）版本，你可以 [fork 开发项目](https://github.com/lhorie/mithril.js)。

需要注意，尽管 Mithril 有集成环境下的测试，最新的（开发）版本可能偶尔会崩溃。如果你对帮忙改进 Mithril 感兴趣，欢迎你使用最新的（开发）版本，并反映你找到的 bug。

[按照这个网页上的指示](https://help.github.com/articles/syncing-a-fork)，你可以对 fork 后的 Mithril 版本进行更新。

#### 在 NPM 上使用最新的开发版本

在 NPM 中，用以下指令来使用最新的开发版本：

```
npm install git://github.com/lhorie/mithril.js/tarball/next --save
```

