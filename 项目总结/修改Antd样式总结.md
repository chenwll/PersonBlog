最近要修改一下Antd的默认样式，遇到了一些奇奇怪怪的地方。今天来做个复现

# 全局修改某一组件样式

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5fd2036063e54eadac714ffe4a069876~tplv-k3u1fbpfcp-watermark.image?)

比如我有两个`Button`按钮，现在他们都是默认`Antd`中`primary`样式，如果我们想统一修改`primary`的默认样式呢？这时候我们应该怎么办
```js
<div>
     <Button type="primary">按钮一</Button>
    <br />
    <br />
    <Button type="primary">按钮二</Button>
</div>
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/633f0f2153a84fb9bc724fb1a8fc0ce3~tplv-k3u1fbpfcp-watermark.image?)

我们发现`Antd`中的`ant-btn-primary`就是控制`primary`的样式的，我们只要覆盖对应的样式即可

## 方法一：通过id选择器
通过id选择器的高优先级覆盖原本的样式
```css
#root .ant-btn-primary {
    background-color: green 
}
```
在任意地方引入这个css文件都可以，但是为了统一规范，在我的Umi项目中，是放在`global.less`文件夹下面

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b3b7d2a8fb814963b726761daa80396d~tplv-k3u1fbpfcp-watermark.image?)

在`.umi`文件夹下面它会先引入`global.less`文件

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9166a110ca34446dba459b4dabe09df2~tplv-k3u1fbpfcp-watermark.image?)

它现在已经变成我喜欢的绿色了

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/08232864479e4f8c94ffb275a63d0000~tplv-k3u1fbpfcp-watermark.image?)


![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/99ebe318a8d24722a9a02f28e78913ac~tplv-k3u1fbpfcp-watermark.image?)

## 方法二：通过:global
```css
:global(.ant-btn-primary) {
  background-color: green;
}
```
此时发现没有生效，样式没有改变的原因，因为**原本的.ant-btn-primary中已经存在background-color**，同时：**global的优先级又没有#root高**，因此不会覆盖，解决这个问题的方法就是加上 **！ important**；

```css
:global(.ant-btn-primary) {
  background-color: green !important;
}
```

但是在`Antd 5.x`中，当我们使用如下覆盖样式时，会发现还是没有生效
```css
:global .ant-btn-primary{
    background-color: green !important;
}
```
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3e416afc1b01446e8132c93f0eaef993~tplv-k3u1fbpfcp-watermark.image?)

看一下样式，发现前面有个没有看到过的选择器(:where(.css-dev-only-do-not-override-ph9edi))

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3de751ab2bc24036b95c682d9a326489~tplv-k3u1fbpfcp-watermark.image?)

查了一下，发现他们做了一些更新 [What is this `css-dev-only` class?](https://github.com/ant-design/ant-design/discussions/38753)，我们不需要使用`global`和`!import`，直接可以覆盖

```css
.ant-btn-primary{
    background-color: green;
}
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c0aec91cbabc4ddea805ca9fae89f06d~tplv-k3u1fbpfcp-watermark.image?)


![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b5184d70e659431289146ef95fb5fd0e~tplv-k3u1fbpfcp-watermark.image?)

# 修改某一个组件样式
有一个需求就是我不需要全局修改组件的样式，我只要改变其中的一个就可以，这时候我们需要给要修改的组件加个`className`


```css
<div>
  <Button className={styles.primary}>按钮一</Button>
  <Button>按钮二</Button>
</div>
```

```css
.primary {
  background-color: green;
}
```
我们写的样式名虽然是`primary`，但是它会经过`css-loader`编译，转化成另一个样式名，这样也可以直接达到效果

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/959d745c7b90434da797bc254d4829bc~tplv-k3u1fbpfcp-watermark.image?)



当我们将className赋值为一个字符串的时候，是不会被css-loader编译的
```js
<div>
  <Button className='primary'>按钮一</Button>
  <br />
  <br />
  <Button>按钮二</Button>
</div>
```
CSS Modules 允许使用`:global(.className)`的语法，声明一个全局规则。凡是这样声明的`class`，都不会被编译成哈希字符串。

因为这时候className没有被编译，global中的className也没有被编译，正好又可以对应上，样式也是同样可以生效

```css
:global(.primary) {
  background-color: green;
}
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dbd3dbb624a5495883af64466b643773~tplv-k3u1fbpfcp-watermark.image?)


![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/034563974c154340b3803081674b6c55~tplv-k3u1fbpfcp-watermark.image?)

如果我们将样式写成这个样子
```js
<div>
  <Button className='primary'>按钮一</Button>
  <br />
  <br />
  <Button>按钮二</Button>
</div>
```

```
.primary {
  background-color: green !important;
}
```
这时候样式就会失效了，因为className不会被编译，但是css文件里面的primary还是会被编译，就找不到对应的样式了，从而失效


![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8ba3c26e4e6d48d896b1c1e41fbe0fac~tplv-k3u1fbpfcp-watermark.image?)

# 总结
当我们想要修改`Antd`的默认样式的时候，可以使用

1. 通过`root`节点，更改对应的属性名
2. 通过`:global`加上`!important`


修改局部样式的时候，要注意`css-loader`会将`className={xxx.xxx}`编译成哈希字符串，可以通过将className设置成为一个字符串解决，同时使用`:global`不会进行编译

另外：CSS Module并不是React的能力，当我们用官方的脚手架创建一个项目的时候，并不支持CSS Module，需要我们手动配置。


# 思考
在 Webpack 中处理 CSS 文件，通常需要用到：

-   [`css-loader`](https://link.juejin.cn/?target=https%3A%2F%2Fwebpack.js.org%2Floaders%2Fcss-loader%2F "https://webpack.js.org/loaders/css-loader/")：该 Loader 会将 CSS 等价翻译为形如 `module.exports = "${css}"` 的JavaScript 代码，使得 Webpack 能够如同处理 JS 代码一样解析 CSS 内容与资源依赖；
-   [`style-loader`](https://link.juejin.cn/?target=https%3A%2F%2Fwebpack.js.org%2Floaders%2Fstyle-loader%2F "https://webpack.js.org/loaders/style-loader/")：该 Loader 将在产物中注入一系列 runtime 代码，这些代码会将 CSS 内容注入到页面的 `<style>` 标签，使得样式生效；
-   [`mini-css-extract-plugin`](https://link.juejin.cn/?target=https%3A%2F%2Fwebpack.js.org%2Fplugins%2Fmini-css-extract-plugin "https://webpack.js.org/plugins/mini-css-extract-plugin")：该插件会将 CSS 代码抽离到单独的 `.css` 文件，并将文件通过 `<link>` 标签方式插入到页面中

`css-loader` 提供了很多处理 CSS 代码的基础能力，包括 CSS 到 JS 转译、依赖解析、Sourcemap、css-in-module 等，基于这些能力，Webpack 才能像处理 JS 模块一样处理 CSS 模块代码。


![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4f378217c0b1486598c96ce69fdcba74~tplv-k3u1fbpfcp-watermark.image?)

经过`css-loader`处理之后的结果，我手动换行了一下
```
Module\n___CSS_LOADER_EXPORT___.push(
[module.id, \"body,html {\\r\\n    
height: 100%;\\r\\n    
background: red;\\r\\n}\\r\\n\\r\\n.primary {\\r\\n    
background-color: green;\\r\\n}\", \"\"
]);
```
但这段字符串只是被当作普通 JS 模块处理，并不会实际影响到页面样式，后续还需要：

1.  **开发环境**：使用 `style-loader` 将样式代码注入到页面 `<style>` 标签；
1.  **生产环境**：使用 `mini-css-extract-plugin` 将样式代码抽离到单独产物文件，并以 `<link>` 标签方式引入到页面中。

经过 `css-loader` 处理后，CSS 代码会被转译为等价 JS 字符串，但这些字符串还不会对页面样式产生影响，需要继续接入 `style-loader` 加载器。

与其它 Loader 不同，`style-loader` 并不会对代码内容做任何修改，而是简单注入一系列运行时代码，用于将 `css-loader` 转译出的 JS 字符串插入到页面的 `style` 标签。

经过 `style-loader` + `css-loader` 处理后，样式代码最终会被写入 Bundle 文件，并在运行时通过 `style` 标签注入到页面。这种将 JS、CSS 代码合并进同一个产物文件的方式有几个问题：

-   JS、CSS 资源无法并行加载，从而降低页面性能；
-   资源缓存粒度变大，JS、CSS 任意一种变更都会致使缓存失效。

因此，生产环境中通常会用 [`mini-css-extract-plugin`](https://link.juejin.cn/?target=https%3A%2F%2Fwebpack.js.org%2Fplugins%2Fmini-css-extract-plugin "https://webpack.js.org/plugins/mini-css-extract-plugin") 插件替代 `style-loader`，将样式代码抽离成单独的 CSS 文件。


在我们使用的Umi项目中，会将className编码，应该是Umi配置好了`css-loader`，自带`modules`功能

这样每个样式都是一个独立的小模块，类名不会出现重复，样式不会覆盖


 # 参考：

https://www.ruanyifeng.com/blog/2016/06/css_modules.html

https://webpack.docschina.org/loaders/css-loader/#modules