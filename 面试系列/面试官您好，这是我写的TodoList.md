前段时间看到掘金上有人二面被面试官要求写一个TodoList，今天趁着上班没啥事情，我也来写一个小Demo玩玩。


![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6355a132a80340d9b4c22a649540532a~tplv-k3u1fbpfcp-watermark.image?)

# 功能

一个TodoList大致就是长成这个样子，有一个输入框，可以通过输入任务名称进行新增，每个任务可以进行勾选，切换已完成和未完成状态，还可以删除。

# 组件设计

## 组件拆分

接下来，我们可以从功能层次上来拆分组件

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4d65fac35fae4d2f9458f50a006eb0d1~tplv-k3u1fbpfcp-watermark.image?)

1. 最外层容器组件，只做一个统一的汇总(红色)
2. 新增组件，管理任务的输入(绿色)
3. 列表组件，管理任务的展示(紫色)，同时我们也可以将每一个item拆分成为单独的组件(粉色)

## 数据流

组件拆分完毕之后，我们来管理一下数据流向，我们的数据应该存放在哪里？

我们的数据可以放在新增组件里面吗？不可以，我们的数据是要传递到列表组件进行展示的，他们两个是兄弟组件，管理起来非常不方便。同理，数据也不能放在列表组件里面。所以我们把数据放在我们的顶级组件里面去管理。

我们在最外层容器组件中把数据定义好，并写好删除，新增的逻辑，然后将数据交给列表组件进行展示，列表组件只管数据的展示，不管具体的实现逻辑，我只要把列表id抛出来，调用你传递的删除函数就可以了

现在，我们引出组件设计时的一些原则

1. 从功能层次上拆分一些组件
2. 尽量让组件原子化，一个组件只做一个功能就可以了，可以让组件吸收复杂度。每个组件都实现一部分功能，那么整个大复杂度的项目自然就被吸收了
3. 区分容器组件和UI组件。容器组件来管理数据，具体的业务逻辑；UI组件就只管显示视图


![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7eaba68ae6cb40da83c2bc09b09173d0~tplv-k3u1fbpfcp-watermark.image?)

# 数据结构的设计

一个合理的数据结构应该满足以下几点：
1. 用数据描述所有的内容
2. 数据要结构化，易于操作遍历和查找
3. 数据要易于扩展，方便增加功能

```js
[
    {
        id:"1",
        title:'标题一',
        completed:false
    },
    {
        id:"2",
        title:'标题二',
        completed:false
    }
]
```

# coding
https://codesandbox.io/s/todolist-tfmzkv

# 反思

看了下Antd表单组件的设计，它将一个Form拆分出了Form和Form.item

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bf777bac5e6a4cf9b17028ef09a0d234~tplv-k3u1fbpfcp-zoom-1.image)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e75956a21de24f85a21e375e1c8f70be~tplv-k3u1fbpfcp-zoom-1.image)

为什么要这么拆分呢？

上文说到，我们在设计一个组件的时候，需要从功能上拆分层次，尽量让组件原子化，只干一件事情。还可以让容器组件（只管理数据）和渲染组件（只管理视图）进行分离

通过Form表单的Api，我们可以发现，Form组件可以控制宏观上的布局，整个表单的样式和数据收集。Form.item控制每个字段的校验等。

个人拙见，如有不妥，还请指教！！！