## Installation
## 安装

Mithril is available from a variety of sources:
Mithril 可以通过不同的渠道获得：

---

### Direct download
### 直接下载

You can [download a zip of the latest version version here](http://lhorie.github.io/mithril/mithril.min.zip).

Links to older versions can be found in the [change log](change-log.html).

In order to use Mithril, extract it from the zip file and point a script tag to the `.js` file:

你可以 [在这里下载最新版本的zip文件](http://lhorie.github.io/mithril/mithril.min.zip)。

历史版本的链接可以在 [change log](change-log.html) 里找到。

使用 Mithril 时，将其从 zip 文件中解压出来，并用一个 script 标签指向 `.js` 文件：

```markup
<script src="mithril.min.js"></script>
```

Note that in order to support older versions of IE, you need to include [some shims](tools.md#internet-explorer-compatibility).

注意，如果要支持旧版本的 IE，你需要包含 [一些 shim](tools.md#internet-explorer-compatibility)。

---

### CDNs (Content Delivery Networks)
### CDN（内容分发网络）

You can also find Mithril in [cdnjs](http://cdnjs.com/libraries/mithril/) and [jsDelivr](http://www.jsdelivr.com/#!mithril).

Content delivery networks allow the library to be cached across different websites that use the same version of the framework, and help reduce latency by serving the files from a server that is physically near the user's location.

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

NPM is the default package manager for [NodeJS](http://nodejs.org/). If you're using NodeJS already or planning on using [Grunt](http://gruntjs.com/) to create a build system, you can use NPM to conveniently keep up-to-date with Mithril versions.

Assuming you have NodeJS installed,  you can download Mithril by typing this:

NPM 是 [NodeJS](http://nodejs.org/) 默认使用的包管理器。如果你已经在使用 NodeJS 或者打算使用 [Grunt](http://gruntjs.com/) 来建立一个构建系统，你可以使用 NPM 以便于保持 Mithril 版本的更新。

假设你已经装好了 NodeJS，你可以输入这行代码来下载 Mithril：

```
npm install mithril
```

Then, to use Mithril, point a script tag to the downloaded file:

然后，用一个 script 标签指向下载的文件，来使用 Mithril：

```markup
<script src="/node_modules/mithril/mithril.min.js"></script>
```

---

### Bower

[Bower](http://bower.io) is a package manager for [NodeJS](http://nodejs.org/). If you're using NodeJS already or planning on using [Grunt](http://gruntjs.com/) to create a build system, you can use Bower to conveniently keep up-to-date with Mithril versions.

Assuming you have NodeJS installed, you can install Bower by typing this in the command line:

[Bower](http://bower.io) 也是一个 [NodeJS](http://nodejs.org/) 的包管理器。如果你已经在使用 NodeJS 或者打算使用 [Grunt](http://gruntjs.com/) 来建立一个构建系统，你可以用 Bower 以便于保持 Mithril 版本的更新。

假设你已经装好了 NodeJS，你可以用以下指令安装 Bower：

```
npm install -g bower
```

And you can download Mithril by typing this:

然后你可以用以下指令安装 Mithril：

```
bower install mithril
```

Then, to use Mithril, point a script tag to the downloaded file:

然后，用一个 script 标签指向下载的文件，来使用 Mithril：

```markup
<script src="/bower_components/mithril/mithril.min.js"></script>
```

---

### Component

[Component](http://component.io) is another package manager for [NodeJS](http://nodejs.org/). If you're using NodeJS already or planning on using [Grunt](http://gruntjs.com/) to create a build system, you can use Component to conveniently keep up-to-date with Mithril versions.

Assuming you have NodeJS installed, you can install Component by typing this in the command line:

[Component](http://component.io) 又是另一个 [NodeJS](http://nodejs.org/) 的包管理器。如果你已经在使用 NodeJS 或者打算使用 [Grunt](http://gruntjs.com/) 来建立一个构建系统，你可以用 Bower 以便于保持 Mithril 版本的更新。（译者注：这段话是不是很熟？）

假设你已经装好了 NodeJS，你可以用以下指令安装 Component：

```
npm install -g component
```

And you can download Mithril by typing this:

然后你可以用以下指令安装 Mithril：

```
component install lhorie/mithril
```

Then, to use Mithril, point a script tag to the downloaded file:

然后，用一个 script 标签指向下载的文件，来使用 Mithril：

```markup
<script src="/components/lhorie/mithril/master/mithril.js"></script>
```

---

### Rails

Jordan Humphreys created a gem to allow integration with Rails:

[Mithril-Rails](https://github.com/mrsweaters/mithril-rails)

It includes support for the [MSX](https://github.com/insin/msx) HTML templating syntax from Jonathan Buchanan.

Jordan Humphreys 写了一个 gem 来将 Mithril 整合到 Rails 中：

[Mithril-Rails](https://github.com/mrsweaters/mithril-rails)

这个 gem 包括对 [MSX](https://github.com/insin/msx) 的支持，MSX 是一种 HTML 模板语法，由 Jonathan Buchanan 创建。

---

### Github

You can also fork the latest stable project [directly from Github](https://github.com/lhorie/mithril).

If you want to use the bleeding edge version, you can [fork the development repository](https://github.com/lhorie/mithril.js).

Be aware that even though Mithril has tests running in a continuous integration environment, the bleeding edge version might occasionally break. If you're interested in helping improve Mithril, you're welcome to use the bleeding edge version and report any bugs that you find.

In order to update a forked version of Mithril, [follow the instructions on this page](https://help.github.com/articles/syncing-a-fork).

你也可以 [直接从 Github](https://github.com/lhorie/mithril) fork 最新的稳定版项目。

如果你希望使用最新的（开发）版本，你可以 [fork 开发项目](https://github.com/lhorie/mithril.js)。

需要注意，尽管 Mithril 有集成环境下的测试，最新的（开发）版本可能偶尔会崩溃。如果你对帮忙改进 Mithril 感兴趣，欢迎你使用最新的（开发）版本，并反映你找到的 bug。

[按照这个网页上的指示](https://help.github.com/articles/syncing-a-fork)，你可以对 fork 后的 Mithril 版本进行更新。

#### Using bleeding edge from NPM
#### 在 NPM 上使用最新的开发版本

To use the bleeding edge version from npm, use the following command:

在 NPM 中，用以下指令来使用最新的开发版本：

```
npm install git://github.com/lhorie/mithril.js/tarball/next --save
```

