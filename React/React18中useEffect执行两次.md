本文用于记录笔者在看React18官方文档中`严格模式`那一节的一些收获，涉及到严格模式的作用，React为什么要将useEffect执行两次以及如何优化的问题。
# 严格模式(Strict Mode)

## 什么是严格模式
strict mode就是一个工具，用来检查应用中可能存在的问题。
严格模式只会在开发环境下运行，在生产环境下它是没有影响的。
```js
import React from 'react';

function ExampleApplication() {
  return (
    <div>
      <Header />
      <React.StrictMode>        <div>
          <ComponentOne />
          <ComponentTwo />
        </div>
      </React.StrictMode>      <Footer />
    </div>
  );
}
```
你可以用`React.StrictMode`包裹你想要检查的应用，没有包裹的应用是不会受到影响的。比如上述的代码中`<Header />`组件是不会受到严格模式的作用的，只有我们包裹的`<ComponentOne />`和`<ComponentTwo>`才会受到严格模式的影响。
## 严格模式的作用
严格模式会进行如下的检查
- 识别不安全的生命周期，比如componentWillMount。后面升级成了componentDidMount


![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/55e781540b31448fa322c32012da6cf6~tplv-k3u1fbpfcp-watermark.image?)
- 对ref这个API以前版本的检查，希望你换成新的版本
- findDOMNode被弃用的警告
- 检测意外的副作用
- 发现context过期的API
- 确保可重复使用的state
```js
React官方文档中给出了一个例子，意思应该是说如果我们从路由A跳转到路由B，再从路由B返回去的时候。
React希望立即展示原来的状态，也就我可以把A的状态缓存起来，那么我们返回的时候就可以根据缓存中的状态立刻渲染出来。
```
# 为什么React将useEffect设置为执行两次
刚刚我们提到，React 将支持使用卸载前已有的组件状态重新挂载到树上。但需要组件对多次挂载和销毁的副作用具有弹性。这里需要我们好好理解React中的副作用，笔者也还没有完全理解这句话的意思，但是我们可以根据React官方文档给出的例子来好好理解useEffect的使用场景，然后再回来理解这句话表达的含义。

意思好像是如果React不将Effect设置成执行两次，那么React在做这方面优化的时候可能会引发一些问题。并且这些问题并不会让开发者注意到，所以React将Effect设置成执行两次，就是为了让问题提前暴露出来。React还认为Effect执行两次在大部分情况下是没有问题的

非严格模式
```js
* React mounts the component. //挂载组件
  * Layout effects are created. //layout执行
  * Effects are created. //Effects执行
```

严格模式
```js
* React mounts the component. //挂载组件
    * Layout effects are created. //layout执行
    * Effect effects are created. // Effects执行
* React simulates effects being destroyed on a mounted component. //React模拟组件销毁
    * Layout effects are destroyed. // layout销毁
    * Effects are destroyed.       // Effects销毁
* React simulates effects being re-created on a mounted component. // React模拟重新挂载
    * Layout effects are created  // layout重新创建
    * Effect setup code runs     //  Effect重新执行
```
# 如何在应用中处理Effect执行两次的问题
React认为执行两次在大多数情况下并没有有什么影响，我们应该考虑的是在重新挂载之后，如何让Effect正确的执行，而不是怎么才能让Effect执行一次。下面我们看些优化的场景。

## 请求数据
我们可以通过一个变量来控制请求的时候只发出去一次，也可以通过取消请求的方式来进行改善。
```js
useEffect(() => {  
    let ignore = false;  
    async function startFetching() {  
        const json = await fetchTodos(userId);  
        if (!ignore) { 
            setTodos(json);  
        }  
}  

    startFetching();  
    return () => {  
       ignore = true;  
    };  
}, [userId]);
```
## 弹窗问题
在第二次挂载之前，将弹窗close掉，就不会有两次调用showModal()的问题了
```js
useEffect(() => {  
    const dialog = dialogRef.current;  
    dialog.showModal();  
    return () => dialog.close();  
}, []);
```
## Not an Effect: Buying a product
不知道标题怎么翻译比较合适，直接拿过来用吧
```js
useEffect(() => {  
    fetch('/api/buy', { method: 'POST' });  
}, []);
```
React认为在useEffect里面发送post请求是不正确的，这种需求场景是不存在的。你应该是在点击事件中发送某个post请求，而不是直接在useEffect里面。
## setTimeout
记得在useEffect里面取消定时器
```js
  useEffect(() => {
    function onTimeout() {
      console.log('⏰ ' + text);
    }

    console.log('⏰  Schedule "' + text + '" log');
    const timeoutId = setTimeout(onTimeout, 3000);

    return () => {
      console.log('⏰  Cancel "' + text + '" log');
      clearTimeout(timeoutId);
    };
  }, [text]);
```

# 原文链接

[严格模式 – React (reactjs.org)](https://zh-hans.reactjs.org/docs/strict-mode.html)

[Synchronizing with Effects (reactjs.org)](https://beta.reactjs.org/learn/synchronizing-with-effects#what-are-effects-and-how-are-they-different-from-events)