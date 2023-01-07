---
title: 介绍
---

### 概述

Rollup 是一个 JavaScript 模块打包器，可以将小块代码编译成大块复杂的代码，例如库或应用程序。Rollup 对代码模块使用新的标准化格式，这些标准都包含在 JavaScript 的 ES6 版本中，而不是以前的特殊解决方案，如 CommonJS 和 AMD。ES 模块可以使你自由、无缝地使用你最喜爱的库中那些最有用独立函数，而你的项目不必携带其他未使用的代码。ES 模块最终还是要由浏览器原生实现，但当前 Rollup 可以使你提前体验。

### Installation

```
npm install --global rollup
```

这样安装可以让 Rollup 全局可用，你也可以将其安装于局部，详见[局部安装 Rollup](guide/en/#installing-rollup-locally).

### 快速入门

Rollup 可以通过[命令行](https://github.com/rollup/rollup/wiki/Command-Line-Interface)配合可选配置文件来调用，或者可以通过 [JavaScript API](https://github.com/rollup/rollup/wiki/JavaScript-API) 来调用。运行 `rollup --help` 可以查看可用的选项和参数。

> 查看 [rollup-starter-lib](https://github.com/rollup/rollup-starter-lib) 和 [rollup-starter-app](https://github.com/rollup/rollup-starter-app) 中那些使用 Rollup 的示例类库与应用项目。

这些命令假设应用程序入口起点的名称为 main.js，并且你想要所有的依赖都编译到一个名为 bundle.js 的单个文件中。

对于浏览器：

```bash
# compile to a <script> containing a self-executing function ('iife')
$ rollup main.js --file bundle.js --format iife
```

对于 Node.js:

```bash
# compile to a CommonJS module ('cjs')
$ rollup main.js --file bundle.js --format cjs
```

对于浏览器和 Node.js:

```bash
# UMD format requires a bundle name
$ rollup main.js --file bundle.js --format umd --name "myBundle"
```

### 为什么

如果你将项目拆分成小的单独文件中，这样开发软件通常会很简单，因为这通常会消除无法预知的相互影响，以及显著降低了所要解决的问题的复杂度，并且可以在项目最初时，就简洁地编写小的项目（[不一定是标准答案](https://medium.com/@Rich_Harris/small-modules-it-s-not-quite-that-simple-3ca532d65de4)）。不幸的是，JavaScript 以往并没有将此功能作为语言的核心功能。

ES6 版本的 Javascript 最终带来了 import 和 export 的语法，该语法可以让我们在多个脚本中共享函数和数据。虽然这一语法标准已确定下来，但目前仅在较新的浏览器中可以使用，并且 Node.js 也尚未完成该标准的导入。有了 Rollup，你现在就可以放心地使用新的模块标准来书写代码，并将其编译为当前被广泛支持的 CommonJS、AMD 以及 IIFE 风格等多种格式。也就是说，你可以用符合未来标准的代码来赢得当下广大用户的芳心和敬意。

### Tree-shaking

除了使用 ES 模块之外，Rollup 还静态分析代码中的 import，并将排除任何未实际使用的代码。这允许您架构于现有工具和模块之上，而不会增加额外的依赖或使项目的大小膨胀。

例如，在使用 CommonJS 时，必须导入完整的工具或库。

```js
// 使用 CommonJS 导入完整的 utils 对象
const utils = require('utils');
const query = 'Rollup';
// 使用 utils 对象的 ajax 方法
utils.ajax(`https://api.example.com?search=${query}`).then(handleResponse);
```

但是在使用 ES6 模块时，无需导入整个 `utils` 对象，我们可以只导入我们所需的 `ajax` 函数：

```js
// 使用 ES6 import 语句导入 ajax 函数
import { ajax } from 'utils';
const query = 'Rollup';
// 调用 ajax 函数
ajax(`https://api.example.com?search=${query}`).then(handleResponse);
```

因为 Rollup 只引入最基本最精简代码，所以可以生成轻量、快速，以及低复杂度的库和应用程序。因为这种基于显式的 `import` 和 `export` 语句的方式，它远比在编译后的输出代码中，简单地运行自动压缩工具检测未使用的变量更有效。

### 兼容性

#### 导入 CommonJS

Rollup 可以[通过插件](https://github.com/rollup/rollup-plugin-commonjs)导入已存在的 CommonJS 模块。

#### 发布 ES6 模块

为了确保你的 ES6 模块可以直接与运行在 CommonJS（例如 Node.js 和 webpack）中的工具(tool)使用，你可以使用 Rollup 编译为 UMD 或 CommonJS 格式，然后在 `package.json` 文件的 `main` 属性中指向当前编译的版本。如果你的 `package.json` 也具有 `module` 字段，像 Rollup 和 [webpack 2+](https://webpack.js.org/) 这样的 ES 模块识别工具将会直接[导入 ES6 模块版本](https://github.com/rollup/rollup/wiki/pkg.module)。
