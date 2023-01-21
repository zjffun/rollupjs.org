---
title: 常见问题
---

#### 为什么ES模块比CommonJS更好?(Why are ES modules better than CommonJS modules?)

ES模块是官方标准，也是JavaScript语言明确的发展方向，而CommonJS模块是一种特殊的传统格式，在ES模块被提出之前做为暂时的解决方案。 ES模块允许进行静态分析，从而实现像 tree-shaking 的优化，并提供诸如循环引用和动态绑定等高级功能。

#### 什么是 ‘tree-shaking’?(What is "tree-shaking?")

Tree-shaking, 也被称为 "live code inclusion," 它是清除实际上并没有在给定项目中使用的代码的过程，但是它可以更加高效。词汇来源查看：[与清除无用代码相似](https://medium.com/@Rich_Harris/tree-shaking-versus-dead-code-elimination-d3765df85c80#.jnypozs9n) 

#### 我如何在 CommonJS 模块中使用 Rollup ?(How do I use Rollup in Node.js with CommonJS modules?)

Rollup 力图实现 ES 模块的规范，而不一定是 Node.js, npm, `require()`, 和 CommonJS 的特性。 因此，加载 CommonJS 模块和使用 Node 模块位置解析逻辑都被实现为可选插件，默认情况下不在 Rollup 内核中。 你只需要执行 `npm install` 安装 [CommonJS](https://github.com/rollup/rollup-plugin-commonjs) 和 [node-resolve](https://github.com/rollup/rollup-plugin-node-resolve) 插件然后使用 `rollup.config.js` 配置文件启用他们，那你就完成了所有设置。

#### Rollup 是用来构建库还是应用程序？(Is Rollup meant for building libraries or applications?)

Rollup 已被许多主流的 JavaScript 库使用，也可用于构建绝大多数应用程序。但是 Rollup 还不支持一些特定的高级功能，尤其是用在构建一些应用程序的时候，特别是代码拆分和运行时态的动态导入 [dynamic imports at runtime](https://github.com/tc39/proposal-dynamic-import). 如果你的项目中更需要这些功能，那使用 [Webpack](https://webpack.js.org/)可能更符合你的需求。


#### Why do additional imports turn up in my entry chunks when code-splitting?

By default when creating multiple chunks, imports of dependencies of entry chunks will be added as empty imports to the entry chunks themselves. [Example](https://rollupjs.org/repl/?shareable=JTdCJTIybW9kdWxlcyUyMiUzQSU1QiU3QiUyMm5hbWUlMjIlM0ElMjJtYWluLmpzJTIyJTJDJTIyY29kZSUyMiUzQSUyMmltcG9ydCUyMHZhbHVlJTIwZnJvbSUyMCcuJTJGb3RoZXItZW50cnkuanMnJTNCJTVDbmNvbnNvbGUubG9nKHZhbHVlKSUzQiUyMiUyQyUyMmlzRW50cnklMjIlM0F0cnVlJTdEJTJDJTdCJTIybmFtZSUyMiUzQSUyMm90aGVyLWVudHJ5LmpzJTIyJTJDJTIyY29kZSUyMiUzQSUyMmltcG9ydCUyMGV4dGVybmFsVmFsdWUlMjBmcm9tJTIwJ2V4dGVybmFsJyUzQiU1Q25leHBvcnQlMjBkZWZhdWx0JTIwMiUyMColMjBleHRlcm5hbFZhbHVlJTNCJTIyJTJDJTIyaXNFbnRyeSUyMiUzQXRydWUlN0QlNUQlMkMlMjJvcHRpb25zJTIyJTNBJTdCJTIyZm9ybWF0JTIyJTNBJTIyZXNtJTIyJTJDJTIybmFtZSUyMiUzQSUyMm15QnVuZGxlJTIyJTJDJTIyYW1kJTIyJTNBJTdCJTIyaWQlMjIlM0ElMjIlMjIlN0QlMkMlMjJnbG9iYWxzJTIyJTNBJTdCJTdEJTdEJTJDJTIyZXhhbXBsZSUyMiUzQW51bGwlN0Q=):

```js
// input
// main.js
import value from './other-entry.js';
console.log(value);

// other-entry.js
import externalValue from 'external';
export default 2 * externalValue;

// output
// main.js
import 'external'; // this import has been hoisted from other-entry.js
import value from './other-entry.js';
console.log(value);

// other-entry.js
import externalValue from 'external';
var value = 2 * externalValue;
export default value;
```

This does not affect code execution order or behaviour, but it will speed up how your code is loaded and parsed. Without this optimization, a JavaScript engine needs to perform the following steps to run `main.js`:

1. Load and parse `main.js`. At the end, an import to `other-entry.js` will be discovered.
2. Load and parse `other-entry.js`. At the end, an import to `external` will be discovered.
3. Load and parse `external`.
4. Execute `main.js`.

With this optimization, a JavaScript engine will discover all transitive dependencies after parsing an entry module, avoiding the waterfall:

1. Load and parse `main.js`. At the end, imports to `other-entry.js` and `external` will be discovered.
2. Load and parse `other-entry.js` and `external`. The import of `external` from `other-entry.js` is already loaded and parsed.
3. Execute `main.js`.

There may be situations where this optimization is not desired, in which case you can turn it off via the [`output.hoistTransitiveImports`](guide/en/#outputhoisttransitiveimports) option. This optimization is also never applied when using the [`output.preserveModules`](guide/en/#outputpreservemodules) option.

#### How do I add polyfills to a Rollup bundle?

Even though Rollup will usually try to maintain exact module execution order when bundling, there are two situations when this is not always the case: code-splitting and external dependencies. The problem is most obvious with external dependencies, see the following [example](https://rollupjs.org/repl/?shareable=JTdCJTIybW9kdWxlcyUyMiUzQSU1QiU3QiUyMm5hbWUlMjIlM0ElMjJtYWluLmpzJTIyJTJDJTIyY29kZSUyMiUzQSUyMmltcG9ydCUyMCcuJTJGcG9seWZpbGwuanMnJTNCJTVDbmltcG9ydCUyMCdleHRlcm5hbCclM0IlNUNuY29uc29sZS5sb2coJ21haW4nKSUzQiUyMiUyQyUyMmlzRW50cnklMjIlM0F0cnVlJTdEJTJDJTdCJTIybmFtZSUyMiUzQSUyMnBvbHlmaWxsLmpzJTIyJTJDJTIyY29kZSUyMiUzQSUyMmNvbnNvbGUubG9nKCdwb2x5ZmlsbCcpJTNCJTIyJTJDJTIyaXNFbnRyeSUyMiUzQWZhbHNlJTdEJTVEJTJDJTIyb3B0aW9ucyUyMiUzQSU3QiUyMmZvcm1hdCUyMiUzQSUyMmVzbSUyMiUyQyUyMm5hbWUlMjIlM0ElMjJteUJ1bmRsZSUyMiUyQyUyMmFtZCUyMiUzQSU3QiUyMmlkJTIyJTNBJTIyJTIyJTdEJTJDJTIyZ2xvYmFscyUyMiUzQSU3QiU3RCU3RCUyQyUyMmV4YW1wbGUlMjIlM0FudWxsJTdE):

```js
// main.js
import './polyfill.js';
import 'external';
console.log('main');

// polyfill.js
console.log('polyfill');
```

Here the execution order is `polyfill.js` → `external` → `main.js`. Now when you bundle the code, you will get

```js
import 'external';
console.log('polyfill');
console.log('main');
```

with the execution order `external` → `polyfill.js` → `main.js`. This is not a problem caused by Rollup putting the `import` at the top of the bundle—imports are always executed first, no matter where they are located in the file. This problem can be solved by creating more chunks: If `polyfill.js` ends up in a different chunk than `main.js`, [correct execution order will be preserved](https://rollupjs.org/repl/?shareable=JTdCJTIybW9kdWxlcyUyMiUzQSU1QiU3QiUyMm5hbWUlMjIlM0ElMjJtYWluLmpzJTIyJTJDJTIyY29kZSUyMiUzQSUyMmltcG9ydCUyMCcuJTJGcG9seWZpbGwuanMnJTNCJTVDbmltcG9ydCUyMCdleHRlcm5hbCclM0IlNUNuY29uc29sZS5sb2coJ21haW4nKSUzQiUyMiUyQyUyMmlzRW50cnklMjIlM0F0cnVlJTdEJTJDJTdCJTIybmFtZSUyMiUzQSUyMnBvbHlmaWxsLmpzJTIyJTJDJTIyY29kZSUyMiUzQSUyMmNvbnNvbGUubG9nKCdwb2x5ZmlsbCcpJTNCJTIyJTJDJTIyaXNFbnRyeSUyMiUzQXRydWUlN0QlNUQlMkMlMjJvcHRpb25zJTIyJTNBJTdCJTIyZm9ybWF0JTIyJTNBJTIyZXNtJTIyJTJDJTIybmFtZSUyMiUzQSUyMm15QnVuZGxlJTIyJTJDJTIyYW1kJTIyJTNBJTdCJTIyaWQlMjIlM0ElMjIlMjIlN0QlMkMlMjJnbG9iYWxzJTIyJTNBJTdCJTdEJTdEJTJDJTIyZXhhbXBsZSUyMiUzQW51bGwlN0Q=). However there is not yet an automatic way to do this in Rollup. For code-splitting, the situation is similar as Rollup is trying to create as few chunks as possible while making sure no code is executed that is not needed.

For most code this is not a problem, because Rollup can guarantee:

> If module A imports module B and there are no circular imports, then B will always be executed before A.

This is however a problem for polyfills, as those usually need to be executed first but it is usually not desired to place an import of the polyfill in every single module. Luckily, this is not needed:

1. If there are no external dependencies that depend on the polyfill, it is enough to add an import of the polyfill as first statement to each static entry point.
2. Otherwise, additionally making the polyfill a separate entry or [manual chunk](guide/en/#outputmanualchunks) will always make sure it is executed first.

#### Is Rollup meant for building libraries or applications?

Rollup is already used by many major JavaScript libraries, and can also be used to build the vast majority of applications. However if you want to use code-splitting or dynamic imports with older browsers, you will need an additional runtime to handle loading missing chunks. We recommend using the [SystemJS Production Build](https://github.com/systemjs/systemjs#browser-production) as it integrates nicely with Rollup's system format output and is capable of properly handling all the ES module live bindings and re-export edge cases. Alternatively, an AMD loader can be used as well.

#### How do I run Rollup itself in a browser

While the regular Rollup build relies on some NodeJS features, there is also a browser build available that only uses browser APIs. You can install it via

```
npm install @rollup/browser
```

and in your script, import it via

```js
import { rollup } from '@rollup/browser';
```

Alternatively, you can import from a CDN, e.g. for the ESM build

```js
import * as rollup from 'https://unpkg.com/@rollup/browser/dist/es/rollup.browser.js';
```

and for the UMD build

```html
<script src="https://unpkg.com/@rollup/browser/dist/rollup.browser.js"></script>
```

which will create a global variable `window.rollup`. As the browser build cannot access the file system, you need to provide plugins that resolve and load all modules you want to bundle. Here is a contrived example that does this:

```js
const modules = {
  'main.js': "import foo from 'foo.js'; console.log(foo);",
  'foo.js': 'export default 42;'
};

rollup
  .rollup({
    input: 'main.js',
    plugins: [
      {
        name: 'loader',
        resolveId(source) {
          if (modules.hasOwnProperty(source)) {
            return source;
          }
        },
        load(id) {
          if (modules.hasOwnProperty(id)) {
            return modules[id];
          }
        }
      }
    ]
  })
  .then(bundle => bundle.generate({ format: 'es' }))
  .then(({ output }) => console.log(output[0].code));
```

This example only supports two imports, `"main.js"` and `"foo.js"`, and no relative imports. Here is another example that uses absolute URLs as entry points and supports relative imports. In that case, we are just re-bundling Rollup itself, but it could be used on any other URL that exposes an ES module:

```js
rollup
  .rollup({
    input: 'https://unpkg.com/rollup/dist/es/rollup.js',
    plugins: [
      {
        name: 'url-resolver',
        resolveId(source, importer) {
          if (source[0] !== '.') {
            try {
              new URL(source);
              // If it is a valid URL, return it
              return source;
            } catch {
              // Otherwise make it external
              return { id: source, external: true };
            }
          }
          return new URL(source, importer).href;
        },
        async load(id) {
          const response = await fetch(id);
          return response.text();
        }
      }
    ]
  })
  .then(bundle => bundle.generate({ format: 'es' }))
  .then(({ output }) => console.log(output));
```

#### 谁制作了 Rollup 的 Logo。太可爱了!(Who made the Rollup logo? It's lovely.)

我就知道! 是[Julian Lloyd.](https://twitter.com/jlmakes)制作的。
