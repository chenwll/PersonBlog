从0到1构建一个vue3的脚手架，熟悉一下webpack的基础配置

# 项目结构搭建
```
// 生成package.json
npm init -y

// 生成ts配置文件
// 如果没有tsc 安装npm install typescript -g
tsc --init
```
并创建如下的文件目录结构

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ed06a3fcb7ff48bbbb11b39810c152c7~tplv-k3u1fbpfcp-watermark.image?)

# webpack相关配置安装

## 基本配置
我这里统一使用了pnpm进行安装，也可用npm, yarn...

安装webpack和webpack-cli
```
pnpm add webpack

pnpm add webpack-cli
```

启动服务webpack-dev-server
```
pnpm add webpack-dev-server
```
安装解析html插件
```
pnpm add html-webpack-plugin
```

webpack.config.js是基于node环境的，node是采用commonJs模块的



添加两条指令

`build`是进行打包的命令
`dev` 是启动开发服务器
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a3057ddfbe724b48a84e78360037337c~tplv-k3u1fbpfcp-watermark.image?)

main.ts里面编写
```
const a = 1
```

编写webpack.config.js
```
// 引入这个,编写config文件可以有提示
const {Configuration} = require('webpack')

const path = require("path");
const htmlWebpackPlugin = require('html-webpack-plugin')

const config = {
    // 开发模式
    mode:"development",
    
    // 入口文件
    entry:'./src/main.ts',
    
    // 配置loader
    module:{
        rules:[
        ]
    },

    // 输出的文件位置
    output: {
        filename:'[hash].js',
        path:path.resolve(__dirname,'dist')
    },

    // plugin
    plugins:[
        new htmlWebpackPlugin({
            template:'./public/index.html'
        }),
    ],
}

// 记得导出config
module.exports = config
```
npm run build，查看打包结果，打包出来的dist没有问题

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/94f293afb8694379a14a5a696f1d6a4a~tplv-k3u1fbpfcp-watermark.image?)

## 引入Vue

初始化App.vue
```
<template>
  <div>webpack搭建vue的脚手架</div>
</template>

<script setup></script>

<style lang="scss" scoped></style>
```
在`index.html`中创建挂载节点
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>webpack demo</title>
    <script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>

</head>
<body>
    // 创建app节点
    <div id="app"></div>
</body>
</html>
```

安装引入vue
```
pnpm install vue
```
main.ts里面引入vue
```
import {createApp} from "vue";
import App from './App.vue'

const app = createApp(App)
// 挂载到id=app的节点上面
app.mount('#app')
```
注意：ts是不认识`.vue`结尾的文件的，所以我们要弄一个声明文件
在src下新建一个`env.d.ts`文件，并配置vue

 ```
 // env.d.ts
declare module "*.vue" {
    import { DefineComponent } from "vue"
    const component: DefineComponent<{}, {}, any>
    export default component
}
 ```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1f73c73b376b44498ad9eb86b3203041~tplv-k3u1fbpfcp-watermark.image?)
安装vue-loader,解析vue和vue文件
```
pnpm add vue-loader@next

pnpm add @vue/compiler-sfc
```

使用插件，每次打包完自动删除
```
 pnpm add clean-webpack-plugin
```

处理css相关样式
```
pnpm add css-loader
pnpm add style-loader
pnpm add less-loader
```
在assets文件夹下新建`index.css`文件，添加背景颜色
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dadeaab019bd4930825155267fb2d006~tplv-k3u1fbpfcp-watermark.image?)

识别ts
```
pnpm add typescript
pnpm add ts-loader
```

现在我们每次打包之前需要手动删除dist文件夹，现在我们引入plugin自动清除dist文件夹
```
pnpm add clean-webpack-plugin
```

再次打包的时候，现在我们看到输出的东西是这么一坨，看着非常难受，我们来美化一下

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8424178f05994f5f90210d1d878f0434~tplv-k3u1fbpfcp-watermark.image?)
```
pnpm add friendly-errors-webpack-plugin
```
还可以添加路径别名，让`@`指向src目录

现在的webpack.config.js
```webpack.config.js
// 引入这个,编写config文件可以有提示
const {Configuration} = require('webpack')
const path = require("path");
const htmlWebpackPlugin = require('html-webpack-plugin')

// 引入vue的plugin
const { VueLoaderPlugin } = require('vue-loader/dist/index');
// 清空dist
const { CleanWebpackPlugin } = require('clean-webpack-plugin')

// 美化webpack输出
const FriendlyErrorsWebpackPlugin = require("friendly-errors-webpack-plugin");
const config = {
    // 开发模式
    mode:"development",
    // 入口文件
    entry:'./src/main.ts',

    module:{
        rules:[
            {
                test:/\.vue$/,
                use:'vue-loader'
            },
            {
                test: /\.less$/, //解析 less
                use: ["style-loader", "css-loader", "less-loader"],
            },
            {
                test: /\.css$/, //解析css
                use: ["style-loader", "css-loader"],
            },
            {
                test: /\.ts$/,  //解析ts
                loader: "ts-loader",
                options: {
                    configFile: path.resolve(process.cwd(), 'tsconfig.json'),
                    appendTsSuffixTo: [/\.vue$/]
                },
            },
        ]
    },

    // 输出的文件位置
    output: {
        filename:'[hash].js',
        path:path.resolve(__dirname,'dist')
    },

    // 配置别名
    resolve: {
        alias:{
            '@':path.resolve(__dirname,'src')
        },
        // 自动补全后缀
        extensions:['.vue','.ts','.js']
    },

    // plugin
    plugins:[
        new htmlWebpackPlugin({
            template:'./public/index.html'
        }),

        // 注册vue的plugin
        new VueLoaderPlugin(),
        // 注册清除dist的plugin
        new CleanWebpackPlugin(),
        //美化webpack
        new FriendlyErrorsWebpackPlugin({
            compilationSuccessInfo:{ //美化样式
                messages:['You application is running here http://localhost:9001']
            }
        }),
    ],
}

// 记得导出config
module.exports = config
```

现在内容就简洁了许多了

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c614726b800c413cbaf9e3a28a89bbc1~tplv-k3u1fbpfcp-watermark.image?)


## 其他配置
配置devServer
```
devServer: {
        proxy: {},
        // 改变端口
        port: 9001,
        hot: true,
        open: true,
    },
```

将Vue通过cdn引入的模式,不将Vue打包到dist文件夹中
```
externals:{
    vue:'Vue'
},
```

不使用cdn之前

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0a487e72b89d48229c25bc465938314a~tplv-k3u1fbpfcp-watermark.image?)

使用cdn引入之后的大小，可以发现大大减小了

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/708ff715b58a4ad6b370f2443f7807e5~tplv-k3u1fbpfcp-watermark.image?)

使用cdn之后，就不会把vue打包进去，可以通过script标签的方式引入，我们现在打开看效果肯定是不行的，会报`vue`is not defined，我们去官网上找个cdn放入`index.html`中
```
<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
```


![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/37ba0e5f55a745de91769a5dbcaa6e13~tplv-k3u1fbpfcp-watermark.image?)
我们再打包一下，就可以正常显示了

cdn优势：减少体积，减少服务器请求压力
但是考验网速，有利有弊

## webpack.config.js文件
```
// 引入这个,编写config文件可以有提示
const {Configuration} = require('webpack')
const path = require("path");
const htmlWebpackPlugin = require('html-webpack-plugin')

// 引入vue的plugin
const { VueLoaderPlugin } = require('vue-loader/dist/index');
// 清空dist
const { CleanWebpackPlugin } = require('clean-webpack-plugin')

// 美化webpack输出
const FriendlyErrorsWebpackPlugin = require("friendly-errors-webpack-plugin");
const config = {
    // 开发模式
    mode:"development",
    // 入口文件
    entry:'./src/main.ts',

    module:{
        rules:[
            {
                test:/\.vue$/,
                use:'vue-loader'
            },
            {
                test: /\.less$/, //解析 less
                use: ["style-loader", "css-loader", "less-loader"],
            },
            {
                test: /\.css$/, //解析css
                use: ["style-loader", "css-loader"],
            },
            {
                test: /\.ts$/,  //解析ts
                loader: "ts-loader",
                options: {
                    configFile: path.resolve(process.cwd(), 'tsconfig.json'),
                    appendTsSuffixTo: [/\.vue$/]
                },
            },
        ]
    },

    // 输出的文件位置
    output: {
        filename:'[hash].js',
        path:path.resolve(__dirname,'dist')
    },

    // 配置别名
    resolve: {
        alias:{
            '@':path.resolve(__dirname,'src')
        },
        // 自动补全后缀
        extensions:['.vue','.ts','.js']
    },

    // plugin
    plugins:[
        new htmlWebpackPlugin({
            template:'./public/index.html'
        }),

        // 注册vue的plugin
        new VueLoaderPlugin(),
        // 注册清除dist的plugin
        new CleanWebpackPlugin(),
        //美化webpack
        new FriendlyErrorsWebpackPlugin({
            compilationSuccessInfo:{ //美化样式
                messages:['You application is running here http://localhost:9001']
            }
        }),
    ],
    externals:{
        vue:'Vue'
    },
    devServer: {
        proxy: {},
        // 改变端口
        port: 9001,
        hot: true,
        open: true,
    },
    stats:"errors-only", //取消提示
}

// 记得导出config
module.exports = config
```

## 参考
[小满Vue3第四十三章（webpack 构建 Vue3项目）](https://xiaoman.blog.csdn.net/article/details/126634893)