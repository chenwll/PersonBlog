# 为什么需要构建工具

- 转化ES6语法
- 转化JSX
- CSS前缀补全/预处理器
- 代码压缩
- 图片压缩

# 安装webpack

```js
npm install webpack webpack-cli -D
```

weback和webpack-cli关系：在命令行中执行 webpack 命令，会执行 node_modules 下的 .bin 目录下的 webpack 文件，webpack 的执行依赖 webpack-cli，如果没有安装webpack-cli 就会报错。在 webpack-cli 中代码执行时，才是真正利用 webpack 进行编译和打包的过程

# 基本配置

```js
const path = require(path);

module.exports ={
	mode: 'production',
    entry: './src/index.js',
    output: {
		path: path.resolve(__dirname, 'dist'),
    	filename: 'bundle.js'
	}
};
```

## entry

entry：入口文件，默认是'src/index.js'

单入口，entry是一个字符串：

```js
module.exports = {
	entry: './path/to/my/entry/file.js'
};
```

多入口，entry是一个对象：

```js
module.exports = {
	entry: {
		app: './src/app.js',
		adminApp: './src/adminApp.js'
	}
};
```

## output

output ⽤来告诉 webpack 如何将编译后的⽂件输出到磁盘

单入口配置：

```js
module.exports = {
	entry: './path/to/my/entry/file.js'
	output: {
		filename: 'bundle.js’,
		path: __dirname + '/dist'
	}
};
```

多入口配置：

```js
module.exports = {
	entry: {
		app: './src/app.js',
		search: './src/search.js'
	},
	output: {
		filename: '[name].js', // 通过占位符确保⽂件名称的唯⼀
		path: __dirname + '/dist'
	}
};
```

## optimization

用于控制如何优化产物包体积，内置 Dead Code Elimination、Scope Hoisting、代码混淆、代码压缩等功能

当mode为`production`时，webpack会自动帮我们开启一系列优化措施：Tree-Shaking、Terser压缩代码、SplitChunk提取公共代码，通常用于生产环境构建

# 常见loader

webpack 开箱即用只支持 JS 和 JSON 两种文件类型，通过 Loaders 去支持其它文件类型并且把它们转化成有效的模块，并且可以添加到依赖图中。

loader本身是一个函数，接受源文件作为参数，返回转换的结果。

## 处理样式

- css-loader：css-loader ⽤于加载 .css ⽂件，并且转换成 commonjs 对象
- style-loader：将样式通过` <style>` 标签插⼊到 head 中
- less-loader：⽤于将 less 转换成 css

## 处理资源

- file-loader：可以用于处理文件和字体
- url-loader：也可以处理图片和字体，可以设置较小资源自动base64
- raw-loader：将文件以字符串的形式导入

## 处理js资源

- babel-loader：转换ES6、ES7等JS新特性语法

详细可以看这篇总结：[从0开始了解babel配置](https://github.com/chenwll/PersonBlog/blob/master/%E5%89%8D%E7%AB%AF%E5%B7%A5%E7%A8%8B%E5%8C%96/%E4%BB%8E0%E5%BC%80%E5%A7%8B%E4%BA%86%E8%A7%A3babel%E9%85%8D%E7%BD%AE.md)

若要解析JSX

```js
{
	"presets": [
		"@babel/preset-env",
		+ "@babel/preset-react" // 新增一个babel的preset
	],
	"plugins": [
		"@babel/proposal-class-properties"
	]
}
```

## loader基本用法

```js
const path = require('path');
module.exports = {
	output: {
		filename: 'bundle.js'
	},
	module: {
		rules: [
			{ 
                test: /\.txt$/, // 匹配规则
             	use: 'raw-loader' // loader名称
            }
		]
	}
};
```

# 常见Plugin

插件⽤于 bundle ⽂件的优化，资源管理和环境变量注⼊，作⽤于整个构建过程

## html-webpack-plugin

由于 `webpack` 只认识 `js`，因此需通过 `html-webpack-plugin` 插件打包 html 文件。当使用 webpack打包时，创建一个 html 文件，并把 webpack 打包后的静态文件自动插入到这个 html 文件当中（创建html文件去承载输出的bundle）。

```js
let HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
    plugins: [
        new HtmlWebpackPlugin({
            template: "./src/index.html"        
        })    
    ]
}
```

`production` 模式下可以开启 html 文件的压缩配置：

```js
plugins: [
    new HtmlWebpackPlugin({
      template: "./src/index.html",
      minify: { removeAttributeQuotes: true, collapseWhitespace: true }, 
      hash: true
    })
]
```

配置详解：

- template：html模板文件的相对/绝对路径
- minify：压缩配置
  - removeAttributeQuotes：删除属性双引号
  - collapseWhitespace：代码压缩成为一行
- hash：引入文件带上hash值

> **TIP：** 如果不指定模板 `template` 配置，将是插件默认的 html 文件，而不是项目中的 html 文件

## clean-webpack-plugin

清理构建出来的目录

```js
const { CleanWebpackPlugin } = require("clean-webpack-plugin"); 

module.exports = {
    plugins: [
        new CleanWebpackPlugin(),
    ]
}
```

## CommonsChunkPlugin

将chunks相同的模块代码提取成为公共js

```js
module.exports = {
  //...
  entry: {
    vendor: ['jquery', 'other-lib'],
    app: './entry',
  },
  plugins: [
    new webpack.optimize.CommonsChunkPlugin({
      name: 'vendor',
      // filename: "vendor.js"
      // (Give the chunk a different name)

      minChunks: Infinity,
      // (with more entries, this ensures that no other module
      //  goes into the vendor chunk)
    }),
  ],
};
```

## ExtractTextWebpackPlugin

将css从bunlde文件里面提取成一个独立的css文件

## CopyWebpackPlugin

将文件或者文件夹拷贝到构建的输出目录

## ZipWebpackPlugin

将打包出的资源生成一个zip包



## thread-loader

Webpack 5 中的 thread- 是一个用于多线程构建的 loader，可以将一些耗时的操作（比如 babel 转译）放到单独的 worker pool 中执行，以此来提高构建性能。



# 其他

## 文件监听

webpack 开启监听模式，有两种⽅式：

- 启动 webpack 命令时，带上 --watch 参数
- 在配置 webpack.config.js 中设置 watch: true

> 文件监听是通过轮询判断⽂件的最后编辑时间是否变化，某个⽂件发⽣了变化，并不会⽴刻告诉监听者，⽽是先缓存起来，收集一段时间**(**可在watchOptions.aggregateTimeout中设置)的变化后，再一次性告诉监听者。防止在编辑代码的过程中可能会高频的输入文字导致文件变化的事件高频的发生

```js
module.export = {
	//默认 false，也就是不开启
	watch: true,
	//只有开启监听模式时，watchOptions才有意义
	wathcOptions: {
		//默认为空，不监听的文件或者文件夹，支持正则匹配
		ignored: /node_modules/,
		//监听到变化发生后会等300ms再去执行，默认300ms
		aggregateTimeout: 300,
		//判断文件是否发生变化是通过不停询问系统指定文件有没有变化实现的，默认每秒问1000次
		poll: 1000
	}
}
```

监听到文件变化后需要去刷新浏览器，这部分在webpack-dev-server模块中进行。(**在使用 webpack-dev-server 模块去启动 webpack 模块时，webpack 模块的监听模式默认会被开启**)



## webpack-dev-server

可通过`devServer.proxy`配置解决跨域问题

假设接口地址为 `http://localhost:3000/api/users`，对 `/api/users` 的请求可如下配置 

```js
devServer: {
    proxy: {
      "/api": {
        target: "http://localhost:3000/",
        pathRewrite: {
          "/api": ""
        },
      },
    }
}
```

配置详解：

- port - 端口号
- compress - 开启 `gzip` 压缩
- open - 启动后自动把页面打开
- client
- `progress`：在浏览器中以百分比显示编译进度

https://segmentfault.com/a/1190000038392557#item-1



## 热更新

以上自动刷新是会刷新整个页面，这种方式的缺点就是时间长，同时不能保存页面的状态。

而模块热更新即可在不刷新整个页面的情况下来实时预览。它只会在代码发生变化时，只编译发生变化的模块，并替换浏览器中的老模块。

### **热更新原理**

- bundle.js：构建输出的文件

- Webpack Compiler：将js源代码编译成`Bundle`（也就是最好打包好输出的文件）
- Bundle Server：提供文件在浏览器的访问

- HMR Server：将打包好的热更新文件输出给HMR Runtime
- HMR Runtime：在打包阶段，会将HMR的runtime注入到bundle.js文件里面，即会被注入到浏览器里面，这样的话浏览器端的bundle.js就会和服务器建立一个连接（websocket），更新文件的变化

热更新的过程：

> 启动阶段：将文件进行一个编译打包，将编译好的文件传递给bundle server，bundle server就可以让浏览器访问这个文件

> 更新阶段：将编译好的文件发送给HMR Server，HMR Server知道哪些模块发生变化，传递给HMR Runtime，HMR Runtime就会更新我们的代码



![image-20230927100615819](http://120.53.221.17/blog/1695780378037.png)



## 文件指纹（hash）

文件指纹是指打包后输出的文件名的后缀

![image-20230927103743458](http://120.53.221.17/blog/1695782265020.png)



文件的hash值有三种：

1. Hash：和整个项⽬的构建相关，只要项⽬⽂件有修改，整个项⽬构建的 hash 值就会更改
2. Chunkhash：和 webpack 打包的 chunk 有关，不同的 entry 会⽣成不同的 chunkhash 值
3. Contenthash：根据⽂件内容来定义 hash ，⽂件内容不变，则 contenthash 不变

### js hash

设置 output 的 filename，使⽤ [chunkhash]

```js
module.exports = {
	entry: {
		app: './src/app.js',
		search: './src/search.js'
	},
	output: {
		filename: '[name][chunkhash:8].js',
		path: __dirname + '/dist'
	}
};
```

### css hash

设置 MiniCssExtractPlugin 的 filename，使⽤ [contenthash]

```js
module.exports = {
	entry: {
		app: './src/app.js',
		search: './src/search.js'
	},
	output: {
		filename: '[name][chunkhash:8].js',
		path: __dirname + '/dist'
	},
	plugins: [
+ 		new MiniCssExtractPlugin({ 
+ 		filename: `[name][contenthash:8].css`
+ 		});
	]
};
```



> 我们通常在js中使用Chunkhash，而css资源中使用Contenthash。如果我们使用了css-loader，style-loader处理css资源，一般会由style-loader插入到<style>标签里面，并放到head头部，并没有一个独立的css文件。所以一般我们会使用MiniCssExtractPlugin单独抽取css文件，并使用contenthash。但是js为什么不使用contenthash缺使用chunkhash值呢？第一个原因就是js其实是没有contenthash值的，其次，webpack通常会将js源码按照entry合并成为几个大文件，js资源的关联度更高，使用chunkhash更便于寻找资源。而css的关联度不高，可以单独拆分成一个个模块，也不用其他资源修改重新加载css，所以就使用了contenthash



### 图片hash

设置 file-loader 的 name，使⽤ [hash]

```js
const path = require('path');
module.exports = {
	entry: './src/index.js',
	output: {
		filename: 'bundle.js',
		path: path.resolve(__dirname, 'dist')
	},
	module: {
		rules: [
			{
				test: /\.(png|svg|jpg|gif)$/,
				use: [{
					loader: 'file-loader’,
+ 					options: {
+ 						name: 'img/[name][hash:8].[ext] '
+ 					}
				}]
			}
		]
	}
};
```

> 图片的hash值是指文件内容的hash，默认是md5生成的

<img src="http://120.53.221.17/blog/1695794424514.png" alt="image-20230927140024227" style="zoom: 50%;" />

## 代码压缩

### TerserWebpackPlugin压缩JS

Webpack5.0 后默认使用 Terser 作为 JavaScript 代码压缩器，简单用法只需通过 `optimization.minimize` 配置项开启压缩功能即可：

```js
module.exports = {
  //...
  optimization: {
    minimize: true
  }
};
```

> 提示：使用 `mode = 'production'` 启动生产模式构建时，默认也会开启 Terser 压缩。

> TerserWebpackPlugin插件已经默认开启了并行压缩，开发者也可以通过设置`parallel`参数（默认值为require('os').cpus() - 1）设置具体的并发进程数量

多数情况下使用默认 Terser 配置即可，必要时也可以手动创建 [terser-webpack-plugin](https://github.com/webpack-contrib/terser-webpack-plugin) 实例并传入压缩配置实现更精细的压缩功能，例如：

```js
const TerserPlugin = require("terser-webpack-plugin");

module.exports = {
  optimization: {
    // 用于控制是否开启压缩，为true才会调用minimzer声明的压缩器数组
    minimize: true,
    minimizer: [
      new TerserPlugin({
        terserOptions: {
          compress: {
            reduce_vars: true,
            pure_funcs: ["console.log"],
          },
          // ...
        },
      }),
    ],
  },
};
```

tarser-webpack-plugin配置项：

1. test：只有命中该配置的产物路径才会执行压缩
2. include：在该范围内的产物才会执行压缩
3. exclude：与include相反，在该范围内的产物不会被压缩
4. parallel：是否启动并行压缩，默认值为true
5. minify：用于配置压缩器，支持传入自定义压缩函数，也支持`swc/esbuild/uglify.js`等值
6. terserOptions：具体压缩选项配置
7. extractComments：是否将代码中的备注抽取为单独文件，可配合特殊备注如 `@license` 使用。

```js
new TerserPlugin({
    minify: (file, sourceMap) => {
      const extractedComments = [];

      // Custom logic for extract comments

      const { error, map, code, warnings } = require('uglify-module') // Or require('./path/to/uglify-module')
        .minify(file, {
          /* Your options for minification */
        });

      return { error, map, code, warnings, extractedComments };
    },
  }),
```



```js
module.exports = {
  optimization: {
    minimize: true,
    minimizer: [
      new TerserPlugin({
        include: /\/includes/,  
        exclude: /\/excludes/,  
        parallel: true,  
        terserOptions: {
          ecma: undefined,
          parse: {},
          compress: {},
          mangle: true, // Note `mangle.properties` is `false` by default.
          module: false,
          // Deprecated
          output: null,
          format: null,
          toplevel: false,
          nameCache: null,
          ie8: false,
          keep_classnames: undefined,
          keep_fnames: false,
          safari10: false,
        },
      }),
    ],
  },
};
```

tarser-webpack-plugin插件并不只是Tarser的简单包装，它更像是一个代码压缩功能骨架，底层还支持使用 SWC、UglifyJS、ESBuild 作为压缩器，使用时只需要通过 `minify` 参数切换即可，例如：

```js
module.exports = {
  optimization: {
    minimize: true,
    minimizer: [
      new TerserPlugin({
        minify: TerserPlugin.swcMinify,
        // `terserOptions` 将被传递到 `swc` (`@swc/core`) 工具
        // 具体配置参数可参考：https://swc.rs/docs/config-js-minify
        terserOptions: {},
      }),
    ],
  },
};
```

> 提示：TerserPlugin 内置如下压缩器：
>
> - `TerserPlugin.terserMinify`：依赖于 `terser` 库；
> - `TerserPlugin.uglifyJsMinify`：依赖于 `uglify-js`，需要手动安装 `yarn add -D uglify-js`；
> - `TerserPlugin.swcMinify`：依赖于 `@swc/core`，需要手动安装 `yarn add -D` `@swc/core`；
> - `TerserPlugin.esbuildMinify`：依赖于 `esbuild`，需要手动安装 `yarn add -D esbuild`。

> 另外，`terserOptions` 配置也不仅仅专供 `terser` 使用，而是会透传给具体的 `minifier`，因此使用不同压缩器时支持的配置选项也会不同。

> 不同压缩器功能、性能差异较大，据我了解，ESBuild 与 SWC 这两个基于 Go 与 Rust 编写的压缩器性能更佳，且效果已经基本趋于稳定，虽然功能还比不上 Terser，但某些构建性能敏感场景下不失为一种不错的选择。

### 压缩css

```js
const CssMinimizerPlugin = require("css-minimizer-webpack-plugin");
const MiniCssExtractPlugin = require("mini-css-extract-plugin");

module.exports = {
  //...
  module: {
    rules: [
      {
        test: /.css$/,
        // 注意，这里用的是 `MiniCssExtractPlugin.loader` 而不是 `style-loader`
        use: [MiniCssExtractPlugin.loader, "css-loader"],
      },
    ],
  },
  optimization: {
    minimize: true,
    minimizer: [
      // Webpack5 之后，约定使用 `'...'` 字面量保留默认 `minimizer` 配置
      "...",
      new CssMinimizerPlugin(),
    ],
  },
  // 需要使用 mini-css-extract-plugin 将 CSS 代码抽取为单独文件
  // 才能命中 css-minimizer-webpack-plugin 默认的 test 规则
  plugins: [new MiniCssExtractPlugin()],
};
```







## 持久化缓存

牺牲空间来提高构建过程中的时间效率

## postcss-loader

自动补全前缀



## px2rem-loader





## devtool

[devtool](https://webpack.docschina.org/configuration/devtool/)此选项控制是否生成，以及如何生成 source map。

官方文档中有很多sourceMap选项，如下：

| 类型                         | 含义                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| source-map                   | 原始代码 最好的sourcemap质量，有完整的结果，但是会很慢       |
| eval-source-map              | 原始代码 同样道理，但是最高的质量和最低的性能                |
| cheap-module-eval-source-map | 原始代码（只有行内） 同样道理，但是更高的质量和更低的性能    |
| cheap-eval-source-map        | 转换代码（行内） 每个模块被eval执行，并且sourcemap作为eval的一个dataurl |
| eval                         | 生成代码 每个模块都被eval执行，并且存在@sourceURL,带eval的构建模式能cache SourceMap |
| cheap-source-map             | 转换代码（行内） 生成的sourcemap没有列映射，从loaders生成的sourcemap没有被使用 |
| cheap-module-source-map      | 原始代码（只有行内） 与上面一样除了每行特点的从loader中进行映射 |

看似配置项很多， 其实只是五个关键字eval、source-map、cheap、module和inline的任意组合

| 关键字     | 含义                                                         |
| ---------- | ------------------------------------------------------------ |
| eval       | 使用eval包裹模块代码                                         |
| source-map | 产生.map文件                                                 |
| cheap      | 不包含列信息（关于列信息的解释下面会有详细介绍)也不包含loader的sourcemap |
| module     | 包含loader的sourcemap（比如jsx to js ，babel的sourcemap）,否则无法定义源文件 |
| inline     | 将.map作为DataURI嵌入，不单独生成.map文件                    |

最佳实践：

- 首先在源代码的列信息是没有意义的，只要有行信息就能完整的建立打包前后代码之间的依赖关系。因此，不管是开发还是生产环境都会增加cheap属性来忽略模块打包后的列信息关联
- 不管是生产环境还是开发环境，我们都需要定位debug到最原始的资源，比如定位错误到jsx，ts的原始代码，而不是把jsx经编译后的js代码。所以不可以忽略掉module属性



hidden-source-map：编译后，可以查看**错误代码准确信息，但是无法查看源代码的位置**



eval模式可以看到出错的行数吗？eval模式下不会生成`.map`文件，所以不会产生具体的行列信息，只知道是哪个文件出错



生产环境到底要不要sourcemap



线上环境关闭source map，调式的时候开启

eval: 使⽤eval包裹模块代码

source map: 产⽣.map⽂件

cheap: 不包含列信息

inline: 将.map作为DataURI嵌⼊，不单独⽣成.map⽂件

module:包含loader的sourcemap

- 如果你以`inline`内联的方式直接打包进文件，会。因为通常`map`会增大包的大小从而影响文件的加载。所以通常**不会**在生产环境下选择`inline`的方式嵌入source map，通常用于开发模式，提高构建/重构建的速度
- 如果你是**独立**的`map`文件，几乎不会。`map`文件只有浏览器开启开发者工具（F12），且开启source map的时候才会去请求相应的`map`文件。

> **⚠️注意**：你在浏览器的network找不到相应的map请求的，浏览器把`.map`文件过滤掉了



eval为什么就快：https://segmentfault.com/q/1010000013570644



## output

target



## HMR热更新





## changelog文档



https://juejin.cn/post/7140619769853509640



webpack擦除没有使用的css

http://www.zhufengpeixun.com/front/html/1.4.webpack-sourcemap.html#t01.%20sourcemap