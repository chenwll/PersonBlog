# 常见loader





# 常见Plugin

html-webpack-plugin 的作用是：当使用 webpack打包时，创建一个 html 文件，并把 webpack 打包后的静态文件自动插入到这个 html 文件当中。





# 体积优化

## 压缩资源

针对`Html`代码，使用`html-webpack-plugin`就可以开启压缩功能

```js
import HtmlPlugin from "html-webpack-plugin";

export default {
	// ...
	plugins: [
		// ...
		HtmlPlugin({
			// ...
			minify: {
				collapseWhitespace: true,
				removeComments: true
			} // 压缩HTML
		})
	]
};
```

如果不添加任何配置的话，会生成一个默认index.html 文件，并自动注入所有的 chunk 和压缩。

```js
const HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
  plugins: [new HtmlWebpackPlugin()],
};
```

也可以通过自定义[配置参数](https://github.com/jantimon/html-webpack-plugin#minification)，以下几个是常见的参数：

- `template`：模板的路径，默认会去寻找 `src/index.ejs` 是否存在。
- `filename`：输出文件的名称，默认为 `index.html`。
- `inject`：是否将资源注入到模版中，默认为 `true`。
- `minify`：压缩参数。在生产模式下（`production`），默认为 `true`；否则，默认为`false`。

如果 `minify` 为 `true`，生成的 HTML 将使用 [html-minifier-terser](https://github.com/terser/html-minifier-terser) 和以下选项进行压缩：

```js
js复制代码{
  collapseWhitespace: true,
  keepClosingSlash: true,
  removeComments: true,
  removeRedundantAttributes: true,
  removeScriptTypeAttributes: true,
  removeStyleLinkTypeAttributes: true,
  useShortDoctype: true
}
```



## 代码分包





## 动态垫片

https://juejin.cn/post/7076424056693587975

https://segmentfault.com/a/1190000021188054

https://zhuanlan.zhihu.com/p/361874935

babel-loader作用

[core-js](https://github.com/zloirock/core-js) 是关于 ES 标准最出名的 `polyfill`，polyfill 意指当浏览器不支持某一最新 API 时，它将帮你实现，中文叫做垫片。core-js已经集成到babel里面了，并成为babel的重要特性

[babel-polyfill](https://github.com/zloirock/core-js#babelpolyfill)已经被废弃，就不多说了

### babel/preset-env

[`@babel/preset-env`](https://babeljs.io/docs/babel-preset-env.html)，但是需要对useBuiltIns进行配置

> @babel/preset-env is a smart preset that allows you to use the latest JavaScript without needing to micromanage which syntax transforms (and optionally, browser polyfills) are needed by your target environment(s). This both makes your life easier and JavaScript bundles smaller!

- `false`:不论要兼容的浏览器版本，默认引入全量的polyfill，引入量很大，有几百Kb

- `entry`:根据配置的要兼容的浏览器的版本，引入对应的polyfill.但是不会根据代码使用情况来引入。

- `usage`:根据配置的要兼容的浏览器版本和代码使用情况，使用哪些引入哪些polyfill。比如使用了Promise，没有使用Map，那么就只引入Promise，不引入Map

@babel/proset-env实现polyfill的原理是，增加全局对象(Promise)或者在原型链上(Array.prototype.some)增加方法，这就造成了全局污染。

在我们的项目中，除了Babel外，可能引入多个第三方模块，如果其他模块也实现了同样的方法，那么就会造成冲突。比如两个第三方模块都实现了 Array.prototype.some，那么就不知道该用谁的，造成了冲突。

如果想要避免冲突，我们就要用导入的方法来引入polyfill，而不是修改原型链或者全局对象的方法。

```js
//如果useBuiltIns实现polyfill
//全局定义Promise，再使用
window.Promise = function(){
...
}

//手动引入
//某个文件内引入，不会影响到其他地方
var Promise = require("./node_modules/@babel/runtime-corejs3/core-js-stable/promise.js")
```



[`@babel/runtime`](https://babeljs.io/docs/plugins/transform-runtime/)不会造成全局的变量污染，需要手动引入这些api

```js
import from from 'core-js-pure/stable/array/from';
import flat from 'core-js-pure/stable/array/flat';
import Set from 'core-js-pure/stable/set';
import Promise from 'core-js-pure/stable/promise';

from(new Set([1, 2, 3, 2, 1]));
flat([1, [2, 3], [4, [5]]], 2);
Promise.resolve(32).then(x => console.log(x));

// you can just write
Array.from(new Set([1, 2, 3, 2, 1]));
[1, [2, 3], [4, [5]]].flat(2);
Promise.resolve(32).then(x => console.log(x));
```

但是如果每个都要手动引入的话，又会非常麻烦，所以@babel/plugin-transform-runtime插件就出现了。它可以帮你引入当前文件需要的polyfill.

