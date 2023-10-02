在某些场景下，我们需要对第三方的依赖进行一点改进，但是这时候我们应该怎么办呢？

# pnpm patch
pnpm 官方新增了两个命令来实现了这个功能，我们一起来看一下
```js
// 生成包的一个修改路路径
pnpm patch <package-name><package-version>

// 生成patch目录，保存修改的diff信息
pnpm patch-commit <file-path>
```

# coding

我们一起来走一遍官网的例子

## 搭建环节

新建一个`patch-demo`文件
```js
pnpm init
```

安装一个第三方依赖
```
pnpm i is-odd@3.0.1
```
新建`index.js`
```js
const isOdd = require('is-odd')

console.log(isOdd(1));
```

配置一下package.json文件
```js
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node index.js"
  },
```

整体的目录结构

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/35f6a56d818d4b209dbe5d4033775ea8~tplv-k3u1fbpfcp-watermark.image?)

`pnpm start`运行一下

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2e67a262d63f499f96be79fe98126f61~tplv-k3u1fbpfcp-watermark.image?)

## 修改第三方库的代码

```js
pnpm patch is-odd@3.0.1
```
返回了一个依赖的修改路径

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/906cc5f5ed564bdabd351b9106a9f149~tplv-k3u1fbpfcp-watermark.image?)

使用`code`打开一下
```js
code C:\Users\19633\AppData\Local\Temp\9b9ee94d390ef48a98a8d57159039b9f
```
修改一下源码，加一个console.log()

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3fd09326c9344be18a0023c6609aacf8~tplv-k3u1fbpfcp-watermark.image?)

保存commit信息
```js
pnpm patch-commit C:\Users\19633\AppData\Local\Temp\9b9ee94d390ef48a98a8d57159039b9f
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9aa718719f2847c5b28afb665f8cbfa7~tplv-k3u1fbpfcp-watermark.image?)

重新运行一下
```js
pnpm start
```
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d15e8aa61ce544da875c5180b7dfba1e~tplv-k3u1fbpfcp-watermark.image?)

此时生成了一个patches的目录，通过将它push到远程仓库，可以保存我们修改第三方库的信息

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/495f9ccdc4a643ba85d9a1f6d1e9408f~tplv-k3u1fbpfcp-watermark.image?)

如果我们直接改变patches下面的文件，也会直接生效

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/65ff1f631b1649c691f73741177eca82~tplv-k3u1fbpfcp-watermark.image?)

记得运行前要重新`install`一下

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e7caca7c96004b1596b120c18dc002bb~tplv-k3u1fbpfcp-watermark.image?)

# 补充
除了上述的方法，也可以将公有依赖fork到私有仓库，通过URI的方式进行安装，如：

如果我们安装`lodash`这个库的话，除了使用包名安装，还可以直接通过URI的形式

```js
  "dependencies": {
    "is-odd": "3.0.1",
    "my-loadsh": "git+https://github.com/lodash/lodash.git"
  },
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4d4986563df54e50b3870c3fac57e102~tplv-k3u1fbpfcp-watermark.image?)

结束收工！！！