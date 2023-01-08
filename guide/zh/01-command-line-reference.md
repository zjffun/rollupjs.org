---
title: 命令行
---

我们一般在命令行中使用 Rollup。你也可以通过可选的 Rollup 配置文件来简化命令行操作，同时可以通过配置文件启用 Rollup 的高级特性。

### 配置文件

Rollup 的配置文件并不是必须的，但是配置文件非常强大而且很方便，所以**推荐使用**。配置文件是一个 ES 模块，它对外导出一个对象，这个对象配置我们想要的选项：

```javascript
export default {
	input: 'src/main.js',
	output: {
		file: 'bundle.js',
		format: 'cjs'
	}
};
```

通常，这个配置文件位于项目的根目录，并且命名为 `rollup.config.js` 或 `rollup.config.mjs`。除非使用 [`--configPlugin`](guide/en/#--configplugin-plugin) 或 [`--bundleConfigAsCjs`](guide/en/#--bundleconfigascjs) 选项，否则 Rollup 会直接使用 Node 导入文件。注意，有一些[使用原生 Node ES 模块时的注意事项](guide/en/#caveats-when-using-native-node-es-modules)，因为 Rollup 将遵守 [Node ESM 语义](https://nodejs.org/docs/latest-v14.x/api/packages.html#packages_determining_module_system）。

如果你想要使用 `require` 和 `module.exports` 将配置文件写成一个 CommonJS 模块，则应该将文件扩展名更改为 `.cjs`。

配置文件也可以使用其他语言，例如 TypeScript。为此，请安装相应的 Rollup 插件，如 `@rollup/plugin-typescript` 并使用 [`--configPlugin`](guide/en/#--configplugin-plugin) 选项：

```
rollup --config rollup.config.ts --configPlugin typescript
```

使用 `--configPlugin` 选项将始终强制您的配置文件首先被转译为 CommonJS。请查看 [配置代码提示](guide/en/#config-intellisense)，了解在配置文件中使用 TypeScript 类型的更多方法。

配置文件支持的选项如下所示。关于选项的详情请查看 [配置项的完整列表](guide/en/#big-list-of-options)：

```javascript
// rollup.config.js

// 可以是一个数组（用于多个输入的情况）
export default {
	// 核心的输入选项
	external,
	input, // 必要项
	plugins,

	// 高级输入选项
	cache,
	onwarn,
	preserveEntrySignatures,
	strictDeprecations,

	// 危险区
	acorn,
	acornInjectPlugins,
	context,
	moduleContext,
	preserveSymlinks,
	shimMissingExports,
	treeshake,

	// 实验性
	experimentalCacheExpiry,
	perf,

	// 必要项 (可以是一个数组，用于多输出的情况)
	output: {
		// 核心的输出选项
		dir,
		file,
		format, // 必要项
		globals,
		name,
		plugins,

		// 高级输出选项
		assetFileNames,
		banner,
		chunkFileNames,
		compact,
		entryFileNames,
		extend,
		footer,
		hoistTransitiveImports,
		inlineDynamicImports,
		interop,
		intro,
		manualChunks,
		minifyInternalExports,
		outro,
		paths,
		preserveModules,
		sourcemap,
		sourcemapExcludeSources,
		sourcemapFile,
		sourcemapPathTransform,
		validate,

		// 危险区
		amd,
		esModule,
		exports,
		externalLiveBindings,
		freeze,
		indent,
		namespaceToStringTag,
		noConflict,
		preferConst,
		strict,
		systemNullSetters
	},

	watch: {
		buildDelay,
		chokidar,
		clearScreen,
		skipWrite,
		exclude,
		include
	}
};
```

甚至在监听模式下，如果你想一次从几个无关的输入项构建 bundle，就可以在配置文件中导出一个**数组**。为了使用相同的输入项构建不同的 bundle，你可以为每个输入项提供一个输出项的数组：

```javascript
// rollup.config.js (构建多个 bundle)

export default [
	{
		input: 'main-a.js',
		output: {
			file: 'dist/bundle-a.js',
			format: 'cjs'
		}
	},
	{
		input: 'main-b.js',
		output: [
			{
				file: 'dist/bundle-b1.js',
				format: 'cjs'
			},
			{
				file: 'dist/bundle-b2.js',
				format: 'es'
			}
		]
	}
];
```

如果你想异步创建配置，Rollup 也能处理结果是一个对象或者数组的 `Promise`。

```javascript
// rollup.config.js
import fetch from 'node-fetch';
export default fetch('/some-remote-service-or-file-which-returns-actual-config');
```

同样，你也可以这样:

```javascript
// rollup.config.js (Promise resolving an array)
export default Promise.all([fetch('get-config-1'), fetch('get-config-2')]);
```

如果你想使用 Rollup 的配置文件，可以在命令行加上 `--config` 或者 `-c` 的选项。

```
# 将自定义配置文件的路径传给 Rollup
rollup --config my.config.js

# 如果你不传文件名, Rollup 将会尝试
# 按照以下顺序加载配置文件：
# rollup.config.mjs -> rollup.config.cjs -> rollup.config.js
rollup --config
```

你也可以导出返回上述任何配置格式的函数。这个函数会接收当前命令行的参数，因此你可以动态调整配置以遵守 [`--silent`](guide/en/#--silent)。甚至你还可以在命令行加上 `config` 前缀来定义自己的命令行选项：

```javascript
// rollup.config.js
import defaultConfig from './rollup.default.config.js';
import debugConfig from './rollup.debug.config.js';

export default commandLineArgs => {
	if (commandLineArgs.configDebug === true) {
		return debugConfig;
	}
	return defaultConfig;
};
```

如果你现在运行 `rollup --config --configDebug`，就会使用 debug 配置。

默认情况下，命令行参数始终会覆盖从配置文件中导出的各个值。如果你想改变这样的方式，可以在 `commandLineArgs` 对象中删除命令行参数，使 Rollup 忽略这些命令行参数：

```javascript
// rollup.config.js
export default commandLineArgs => {
  const inputBase = commandLineArgs.input || 'main.js';

  // 这会使 Rollup 忽略命令行参数
  delete commandLineArgs.input;
  return {
    input: 'src/entries/' + inputBase,
    output: {...}
  }
}
```

#### 配置代码提示

Since Rollup ships with TypeScript typings, you can leverage your IDE's Intellisense with JSDoc type hints:
由于 Rollup 附带 TypeScript 类型，你可以通过 JSDoc 的注释使用 IDE 的代码提示：

```javascript
// rollup.config.js
/**
 * @type {import('rollup').RollupOptions}
 */
const config = {
	/* your config */
};
export default config;
```

或者你可以使用 `defineConfig` 函数，它提供代码提示而不需要 JSDoc 注释：

```javascript
// rollup.config.js
import { defineConfig } from 'rollup';

export default defineConfig({
	/* your config */
});
```

除了 `RollupOptions` 和封装此类型的 `defineConfig` 函数之外，以下类型也是有用的：

- `OutputOptions`：配置文件的`output`部分。
- `Plugin`：提供`name` 和一些钩子的插件对象。 所有钩子都是完全类型化的，以帮助插件开发。
- `PluginImpl`：将选项对象映射到插件对象的函数。 大多数公共 Rollup 插件都遵循这种模式。

你还可以通过 [`--configPlugin`](guide/en/#--configplugin-plugin) 选项直接在 TypeScript 中编写配置。使用 TypeScript，你可以直接导入 `RollupOptions` 类型：

```typescript
import type { RollupOptions } from 'rollup';

const config: RollupOptions = {
	/* your config */
};
export default config;
```

### 与 JavaScript API 的区别

尽管配置文件提供了配置 Rollup 的简单方式，但是它也限制了 Rollup 如何能被调用以及在何处进行的配置。尤其是如果你想要在另一个构建工具中重新打包 Rollup，或者想要把它集成到高级的构建进程中，那么最好是在脚本中用编程的方式调用 Rollup。

如果你想从配置文件切换为使用 [JavaScript API](guide/en/#javascript-api)，这里有一些需要注意的重要区别：

- 使用 JavaScript API 时，传给 `rollup.rollup` 的配置必须是一个对象，并且不能包装在 Promise 或者函数中。
- 你不能再使用数组作为配置。取而代之的是，你应该为每组 `inputOptions` 都运行一次 `rollup.rollup`。
- `output` 配置将会会被忽略。取而代之的是，你应该为每组 `outputOptions` 都运行一次 `bundle.generate(outputOptions)` 或者 `bundle.write(outputOptions)`。

### 从 Node 包中加载配置

为了实现互通性，Rollup 也支持从安装在 `node_modules` 目录下的包中加载配置文件。

```
# Rollup 首先会尝试加载 "rollup-config-my-special-config";
# 如果失败，Rollup 则会尝试加载 "my-special-config"
rollup --config node:my-special-config
```

### 使用原生 Node ES 模块时的注意事项

特别是从旧的 Rollup 版本升级时，在配置文件中使用原生 ES 模块时需要注意一些事项。

#### 获取当前目录

对于 CommonJS 文件，人们通常使用 `__dirname` 来访问当前目录并将相对路径解析为绝对路径。原生 ES 模块不支持此功能。相反，我们推荐以下方法，例如为外部模块生成绝对 ID：

```js
// rollup.config.js
import { fileURLToPath } from 'node:url'

export default {
  ...,
  // generates an absolute path for <currentdir>/src/some-external-file.js
  external: [fileURLToPath(new URL('src/some-external-file.js', import.meta.url))]
};
```

#### 导入 package.json

例如自动将依赖标记为“外部依赖”需要将 package.json 文件导入。根据你的 Node 版本，有不同的方法可以做到这一点：

- 对于 Node 17.5+，可以使用导入断言

  ```js
  import pkg from './package.json' assert { type: 'json' };

  export default {
  	// Mark package dependencies as "external". Rest of configuration omitted.
  	external: Object.keys(pkg.dependencies)
  };
  ```

- 对于较旧的 Node 版本，可以使用 `createRequire`

  ```js
  import { createRequire } from 'node:module';
  const require = createRequire(import.meta.url);
  const pkg = require('./package.json');

  // ...
  ```

- 或者直接从磁盘读取和解析文件

  ```js
  // rollup.config.mjs
  import { readFileSync } from 'node:fs';

  // Use import.meta.url to make the path relative to the current source file instead of process.cwd()
  // For more info: https://nodejs.org/docs/latest-v16.x/api/esm.html#importmetaurl
  const packageJson = JSON.parse(readFileSync(new URL('./package.json', import.meta.url)));

  // ...
  ```

### 命令行标志

很多选项都有等价的命令行参数。如果你使用的话，此处传递的所有参数都将覆盖配置文件。这是所有支持的选项列表：

```
-c, --config <filename>     使用配置文件（如果使用参数但是值没有
                              指定, 默认就是 rollup.config.js）
-d, --dir <dirname>         构建块的目录（如果不存在，就打印到标准输出）
-e, --external <ids>        逗号分隔列出排除的模块 ID
-f, --format <format>       输出类型 (amd, cjs, es, iife, umd, system)
-g, --globals <pairs>       逗号分隔列出 `moduleID:Global` 对
-h, --help                  显示帮助信息
-i, --input <filename>      输入 (替代 <entry file>)
-m, --sourcemap             生成 sourcemap (`-m inline` 生成行内 map)
-n, --name <name>           UMD 导出的名字
-o, --file <output>         单个的输出文件（如果不存在，就打印到标准输出）
-p, --plugin <plugin>       使用指定的插件（可能重复）
-v, --version               显示版本号
-w, --watch                 监听 bundle 中的文件并在文件改变时重新构建
--amd.autoId                根据块名称生成 AMD ID
--amd.basePath <prefix>     添加到自动生成 AMD ID 前的路径
--amd.define <name>         代替 `define` 使用的函数
--amd.forceJsExtensionForImports 在 AMD 导入中使用 `.js` 扩展名
--amd.id <id>               AMD 模块 ID（默认是匿名的）
--assetFileNames <pattern>  构建的资源命名模式
--banner <text>             插入 bundle 顶部（包装器之外）的代码
--chunkFileNames <pattern>  次要构建块的命名模式
--compact                   压缩包装器代码
--context <variable>        指定顶层的 `this` 值
--no-dynamicImportInCjs     将外部动态 CommonJS 导入转为 require
--entryFileNames <pattern>  入口构建块的命名模式
--environment <values>      设置传递到配置文件 (见示例)
--no-esModule               不增加 __esModule 属性
--exports <mode>            指定导出的模式 (auto, default, named, none)
--extend                    通过 --name 定义，拓展全局变量
--no-externalImportAssertions 在“es”输出中省略导入断言
--no-externalLiveBindings   不生成实施绑定的代码
--failAfterWarnings         如果构建产生警告，则带有错误退出
--footer <text>             插入到 bundle 末尾的代码（包装器外部）
--no-freeze                 不冻结命名空间对象
--generatedCode <preset>    使用哪些代码功能 (es5/es2015)
--generatedCode.arrowFunctions 在生成的代码中使用箭头函数
--generatedCode.constBindings 在生成的代码中使用“const”
--generatedCode.objectShorthand 在生成的代码中使用简写属性
--no-generatedCode.reservedNamesAsProps 始终引用保留名称作为属性
--generatedCode.symbols     在生成的代码中使用符号
--no-hoistTransitiveImports 不提升传递性的导入到入口构建块
--no-indent                 结果中不进行缩进
--inlineDynamicImports      使用动态导入时创建单个 bundle
--no-interop                不包含互操作块
--intro <text>              在 bundle 顶部插入代码（包装器内部）
--no-makeAbsoluteExternalsRelative 阻止外部导入标准化
--maxParallelFileOps <value> 并行读取多少个文件
--minifyInternalExports     强制或者禁用内部导出的压缩
--noConflict                为 UMD 全局变量生成 noConflict 方法
--outro <text>              在 bundle 的末尾插入代码（包装器内部）
--perf                      显示性能计时
--no-preserveEntrySignatures 避免表面的构建块作为入口
--preserveModules           保留模块结构
--preserveModulesRoot       将保留的模块放在此顶层路径下
--preserveSymlinks          解析文件时不要遵循符号链接
--no-sanitizeFileName       不要替换文件名中的无效字符
--shimMissingExports        给丢失的导出创建填充变量
--silent                    不打印警告
--sourcemapBaseUrl <url>    生成具有给定基础 URL 的绝对 sourcemap URL
--sourcemapExcludeSources   source map 中不包含源码
--sourcemapFile <file>      source map 中指定 bundle 的路径
--stdin=ext                 Specify file extension used for stdin input
--no-stdin                  不从标准输入中读取 "-"
--no-strict                 在生成的模块中不使用 `"use strict";`
--strictDeprecations        不推荐使用的特性抛出错误
--no-systemNullSetters      不用 `null` 替换空的 SystemJS setter
--no-treeshake              禁用 tree-shaking 优化
--no-treeshake.annotations  忽略纯的调用注释
--treeshake.correctVarValueBeforeDeclaration 取消优化变量直到声明
--treeshake.manualPureFunctions <names> 手动将函数声明为纯函数
--no-treeshake.moduleSideEffects 假设模块没有副作用
--no-treeshake.propertyReadSideEffects 忽略属性访问的副作用
--no-treeshake.tryCatchDeoptimization 不关闭 try-catch-tree-shaking
--no-treeshake.unknownGlobalSideEffects 假设未知的全局变量不抛出
--validate                  验证输出
--waitForBundleInput        等待 bundle 的输入文件
--watch.buildDelay <number> 监听重新构建的延时
--no-watch.clearScreen      重新构建时不进行清屏
--watch.exclude <files>     监听时排除的文件
--watch.include <files>     限制监听指定的文件
--watch.onBundleEnd <cmd>   `"BUNDLE_END"` 事件运行的 Shell 命令
--watch.onBundleStart <cmd> `"BUNDLE_START"` 事件运行的 Shell 命令
--watch.onEnd <cmd>         `"END"` 事件运行的 Shell 命令
--watch.onError <cmd>       `"ERROR"` 事件运行的 Shell 命令
--watch.onStart <cmd>       `"START"` 事件运行的 Shell 命令
--watch.skipWrite           监听时不写入文件到磁盘
```

下面列出来的选项只能在命令行接口使用。其他所有的选项都一一对应并会覆盖配置文件的等价配置项，具体细节可以参考[配置项的完整列表](guide/en/#big-list-of-options)。

#### `-h`/`--help`

打印帮助文档。

#### `-p <plugin>`, `--plugin <plugin>`

使用指定的插件。这里有几种方法可以指定插件：

- 通过相对路径：

  ```
  rollup -i input.js -f es -p ./my-plugin.js
  ```

  这个文件应该导出一个返回 plugin 对象的函数。

- 通过安装在项目或者全局的 `node_modules` 文件夹的插件名字：

  ```
  rollup -i input.js -f es -p @rollup/plugin-node-resolve
  ```

  如果插件的名字不是以 `rollup-plugin-` 或者 `@rollup/plugin-` 开头，Rollup 会自动尝试添加这些前缀：

  ```
  rollup -i input.js -f es -p node-resolve
  ```

- 通过内联实现：

  ```
  rollup -i input.js -f es -p '{transform: (c, i) => `/* ${JSON.stringify(i)} */\n${c}`}'
  ```

如果想要加载多个插件，则可以重复该选项，或者提供按逗号分隔的名称列表：

```
rollup -i input.js -f es -p node-resolve -p commonjs,json
```

默认情况下，导出方法的插件在创建时被调用应该是不接收参数的。然而你也可以传递自定义的参数：

```
rollup -i input.js -f es -p 'terser={output: {beautify: true, indent_level: 2}}'
```

#### `--configPlugin <plugin>`

允许指定 Rollup 插件来转译或以其他方式控制配置文件的解析。主要好处是它允许你使用非 JavaScript 配置文件。例如，如果你安装了“@rollup/plugin-typescript”，则以下内容将允许你使用 TypeScript 编写配置：

```
rollup --config rollup.config.ts --configPlugin @rollup/plugin-typescript
```

Typescript 的注意事项：确保在 `tsconfig.json` 的 `include` 中有 Rollup 配置文件。 例如：

```
"include": ["src/**/*", "rollup.config.ts"],
```

此选项支持与 [`--plugin`](guide/en/#-p-plugin---plugin-plugin) 选项相同的语法，即可以多次指定该选项，可以省略 `@rollup/plugin-` 前缀，只需编写 `typescript`，可以通过 `={...}` 指定插件选项。

使用此选项将使 Rollup 在执行之前先将您的配置文件转译为 ES 模块。要转译为 CommonJS，还要传递 [`--bundleConfigAsCjs`](guide/en/#--bundleconfigascjs) 选项。

#### `--bundleConfigAsCjs`

此选项将强制将您的配置转译为 CommonJS。

这允许您在配置中使用 CommonJS，如 `__dirname` 或 `require.resolve`，即使配置本身是作为 ES 模块编写的。

#### `-v`/`--version`

打印安装的版本号。

#### `-w`/`--watch`

当源文件在磁盘上发生更改时，重新构建 bundle。

_注意: 在监听模式下，环境变量 `ROLLUP_WATCH` 将会被 Rollup 的命令行设置成 `"true"`，并且可以被其他进程所检查到。插件应该改为检查 [`this.meta.watchMode`](guide/en/#thismeta)，因为它独立于命令行。_

#### `--silent`

不在控制台打印警告。如果配置文件包含一个 `onwarn` 的处理方法，那么这个方法就会被调用。要手动阻止这种情况，你可以按照[配置文件](guide/en/#configuration-files)末尾的说明访问配置文件中的命令行选项。

#### `--failAfterWarnings`

在构建完成后，如果有警告，则带有错误退出。

#### `--environment <values>`

通过 `process.ENV` 传递额外的设置给配置文件。

```sh
rollup -c --environment INCLUDE_DEPS,BUILD:production
```

将设置 `process.env.INCLUDE_DEPS === 'true'` 和 `process.env.BUILD === 'production'`。你可以多次使用这个选项。在这种情况下，后面设置的变量将覆盖前面的定义。这保证了你可以在 `package.json` 脚本中覆盖环境变量：

```json
{
	"scripts": {
		"build": "rollup -c --environment INCLUDE_DEPS,BUILD:production"
	}
}
```

如果通过以下的方式调用脚本：

```
npm run build -- --environment BUILD:development
```

那么配置文件就会接收 `process.env.INCLUDE_DEPS === 'true'` 和 `process.env.BUILD === 'development'`。

#### `--waitForBundleInput`

如果其中一个入口文件不可访问，将不会抛出错误。相反，它会等到所有文件都存在后再开始构建。当 Rollup 正在消费另一个进程的输出时，这个功能很有用，尤其是在监听模式下。

#### `--stdin=ext`

从 stdin 读取内容时指定虚拟文件扩展名。默认情况下，Rollup 将使用不带扩展名的虚拟文件名 `-` 从 stdin 读取内容。然而，一些插件依赖于文件扩展名来确定它们是否应该处理文件。另请参阅 [从标准输入读取文件](guide/en/#reading-a-file-from-stdin)。

#### `--no-stdin`

不从 `stdin` 读取文件。设置这个标志可以阻止传递内容到 Rollup，也保证了 Rollup 将 `-` 和 `-.[ext]` 解释为常规文件名，而不是将其解释为 `stdin`。另见[从标准输入读取文件](guide/en/#reading-a-file-from-stdin)。

#### `--watch.onStart <cmd>`, `--watch.onBundleStart <cmd>`, `--watch.onBundleEnd <cmd>`, `--watch.onEnd <cmd>`, `--watch.onError <cmd>`

在监听模式下，为监听事件运行 `<cmd>` Shell 命令。 另请参阅 [rollup.watch](guide/en/#rollupwatch)。

```sh
rollup -c --watch --watch.onEnd="node ./afterBuildScript.js"
```

### 从标准输入中读取文件

使用命令行时，Rollup 也能从标准输入读取内容：

```
echo "export const foo = 42;" | rollup --format cjs --file out.js
```

当文件包含导入时，Rollup 将尝试相对于当前工作路径来解析它们。当使用配置文件时，如果入口的文件名是 `-`，Rollup 将仅用 `stdin` 作为入口。要从标准输入读取一个非入口的文件，只需要给它取名为 `-`，它是内部引用标准输入的文件名。即：

```
import foo from "-";
```

在任何文件中，都将促使 Rollup 尝试从 `stdin` 读取导入文件，并且将其默认导出赋值给 `foo`。你可以通过 [`--no-stdin`](guide/en/#--no-stdin) 命令行选项来使得 Rollup 像对待常规文件名一样对待 `-`。

由于某些插件依赖文件扩展名来处理文件，你可以通过 `--stdin=ext` 为 stdin 指定文件扩展名，其中 `ext` 是所需的扩展名。在这种情况下，虚拟文件名将是“-.ext”：

```
echo '{"foo": 42, "bar": "ok"}' | rollup --stdin=json -p json
```

JavaScript API 始终将 `-` 和 `-.ext` 视为普通文件名。
