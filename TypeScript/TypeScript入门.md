# 搭建本地ts环境

## tsc

TypeScript 官方提供的编译器叫做 tsc，可以将 TypeScript 脚本编译成 JavaScript 脚本。本机想要编译 TypeScript 代码，必须安装 tsc。tsc 的作用就是把`.ts`脚本转变成`.js`脚本。

全局安装tsc，也可以作为一个项目依赖安装

```js
npm install -g typescript
```

安装之后就可以，就可以编译ts脚本了。

```js
// 生成tsconfig.json文件
tsc --init
```

> TypeScript 允许将`tsc`的编译参数，写在配置文件`tsconfig.json`。只要当前目录有这个文件，`tsc`就会自动读取。

```js
# ./src/index.ts
console.log("Hello TypeScript");
```

```js
tsc ./src/index.ts
```

## ts-node

[ts-node](https://github.com/TypeStrong/ts-node) 是一个非官方的 npm 模块，可以直接运行 TypeScript 代码。有点像`node index.js`直接运行js文件一样，`ts-node`可以直接运行ts脚本

```js
$ ts-node ./src/index.ts

'Hello TypeScript'
```

我们也可以指定`ts-node`的配置项

- `-P,--project`：指定你的 tsconfig 文件位置。默认情况下 ts-node 会查找项目下的 tsconfig.json 文件，如果你的配置文件是 `tsconfig.script.json`、`tsconfig.base.json` 这种，就需要使用这一参数来进行配置了。
- `-T, --transpileOnly`：禁用掉执行过程中的类型检查过程，这能让你的文件执行速度更快，且不会被类型报错卡住。这一选项的实质是使用了 TypeScript Compiler API 中的 transpileModule 方法，我们会在后面的章节详细讲解。
- `--swc`：在 transpileOnly 的基础上，还会使用 swc 来进行文件的编译，进一步提升执行速度。
- `--emit`：如果你不仅是想要执行，还想顺便查看下产物，可以使用这一选项来把编译产物输出到 `.ts-node` 文件夹下（需要同时与 `--compilerHost` 选项一同使用



## ts-node-dev

有时候我们需要像`nodemon`自动监听文件变化，然后自动重新执行，但是`ts-node`并不提供这样的能力，这时候我们可以使用`ts-node-dev`来实现

```js
npm i ts-node-dev -g
```

```js
tsnd  --respawn  ./src/index.ts
```

`--respawn`就相当于`--watch`开启监听

![image-20231001161037547](http://120.53.221.17/blog/1696147839186.png)



# TsConfig

importsNotUsedAsValues：通过控制它可以来控制没有被使用的导入语句将会被如何处理，它提供三个不同的选型

baseUrl：这一配置可以定义文件进行解析的根目录，它通常会是一个相对路径，然后配置`tsconfig.json`所在的路径来确定根目录的位置



# 类型声明

## 推荐文章

- [TypeScript 全面进阶指南—第20节](https://juejin.cn/book/7086408430491172901/section/7105947956183826464#heading-4)

- [TypeScipt入门教程—声明文件](https://ts.xcatliu.com/basics/declaration-files.html#declare-var)
- [阮一峰Ts教程—declare](https://wangdoc.com/typescript/declare)

## declare

- [`declare var`](https://ts.xcatliu.com/basics/declaration-files.html#declare-var) 声明全局变量
- [`declare function`](https://ts.xcatliu.com/basics/declaration-files.html#declare-function) 声明全局方法
- [`declare class`](https://ts.xcatliu.com/basics/declaration-files.html#declare-class) 声明全局类
- [`declare enum`](https://ts.xcatliu.com/basics/declaration-files.html#declare-enum) 声明全局枚举类型
- [`declare namespace`](https://ts.xcatliu.com/basics/declaration-files.html#declare-namespace) 声明（含有子属性的）全局对象
- [`interface` 和 `type`](https://ts.xcatliu.com/basics/declaration-files.html#interface-和-type) 声明全局类型
- [`export`](https://ts.xcatliu.com/basics/declaration-files.html#export) 导出变量
- [`export namespace`](https://ts.xcatliu.com/basics/declaration-files.html#export-namespace) 导出（含有子属性的）对象
- [`export default`](https://ts.xcatliu.com/basics/declaration-files.html#export-default) ES6 默认导出
- [`export =`](https://ts.xcatliu.com/basics/declaration-files.html#export-1) commonjs 导出模块
- [`export as namespace`](https://ts.xcatliu.com/basics/declaration-files.html#export-as-namespace) UMD 库声明全局变量
- [`declare global`](https://ts.xcatliu.com/basics/declaration-files.html#declare-global) 扩展全局变量
- [`declare module`](https://ts.xcatliu.com/basics/declaration-files.html#declare-module) 扩展模块
- [`/// `](https://ts.xcatliu.com/basics/declaration-files.html#san-xie-xian-zhi-ling) 三斜线指令

在所有的声明语句中，`declare var` 是最简单的，如之前所学，它能够用来定义一个全局变量的类型。与其类似的，还有 `declare let` 和 `declare const`，使用 `let` 与使用 `var` 没有什么区别。而当我们使用 `const` 定义时，表示此时的全局变量是一个常量，不允许再去修改它的值了

declare module 和 declare namespace 里面，加不加 export 关键字都可以。**同名接口会进行合并**

导出一个空对象是干啥用？

```js
export {}; 

https://zhuanlan.zhihu.com/p/655660979
```

## 无类型定义的 npm 包

如果你要使用一个npm包，但它发布的时间太早，根本没有携带类型定义，于是你的项目里就出现了这么一处没有被类型覆盖的地方。

```js
import foo from 'pkg';

const res = foo.handler();
```

假设这里的`pkg`是一个没有类型定义的npm包，我们怎么添加类型提示呢？

```js
declare module 'pkg' {
  const handler: () => boolean;
}
```

除了npm包可以这样，我们导入其他类型的文件时，也可以使用`declare module`语法

```js
// index.ts
import raw from './note.md';

const content = raw.replace('NOTE', `NOTE${new Date().getDay()}`);

// declare.d.ts
declare module '*.md' {
  const raw: string;
  export default raw;
}
```



## DefinitelyTyped





## 三斜线指令

```js
/// <reference path="./other.d.ts" />
/// <reference types="node" />
/// <reference lib="dom" />
```

**三斜线指令必须被放置在文件的顶部才能生效**

三条指令作用其实都是声明当前文件依赖的外部类型声明，只不过使用的方式不同：分别使用了 path、types、lib 这三个不同属性，我们来依次解析。

使用 path 的 reference 指令，其 path 属性的值为一个相对路径，指向你项目内的其他声明文件。而在编译时，TS 会沿着 path 指定的路径不断深入寻找，最深的那个没有其他依赖的声明文件会被最先加载。

```typescript
// @types/node 中的示例
/// <reference path="fs.d.ts" />
```

使用 types 的 reference 指令，其 types 的值是一个包名，也就是你想引入的 `@types/` 声明，如上面的例子中我们实际上是在声明当前文件对 `@types/node` 的依赖。而如果你的代码文件（`.ts`）中声明了对某一个包的类型导入，那么在编译产生的声明文件（`.d.ts`）中会自动包含引用它的指令。

```js
/// <reference types="node" />
```

使用 lib 的 reference 指令类似于 types，只不过这里 lib 导入的是 TypeScript 内置的类型声明，如下面的例子我们声明了对 `lib.dom.d.ts` 的依赖：

```typescript
// vite/client.d.ts
/// <reference lib="dom" />
```





# 模块和命令空间

## 模块

任何包含 import 或 export 语句的文件，就是一个模块（module）。相应地，如果文件不包含 export 语句，就是一个全局的脚本文件。

模块本身就是一个作用域，不属于全局作用域。如果想要使用模块里面的方法，就要使用`export`

如果一个文件不包含 export 语句，但是希望把它当作一个模块（即内部变量对外不可见），可以在脚本头部添加一行语句。

```js
export {};
```

我们经常能看见`.d.ts`文件中有如上语句，如：

<img src="http://120.53.221.17/blog/1696249184683.png" alt="image-20231002201942979" style="zoom:80%;" />

上面这行语句不产生任何实际作用，但会让当前文件被当作模块处理，所有它的代码都变成了内部代码。

## 命令空间（namespace）

namespace 是一种将相关代码组织在一起的方式，中文译为“命名空间”。

它出现在 ES 模块诞生之前，作为 TypeScript 自己的模块格式而发明的。但是，自从有了 ES 模块，官方已经不推荐使用 namespace 了。



# 工具类型

- [typeof](https://www.51cto.com/article/711374.html)
- 





# 参考资料

- https://ts.xcatliu.com/basics/declaration-files.html#declare-var)