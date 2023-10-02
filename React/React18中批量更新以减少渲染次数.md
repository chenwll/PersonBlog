本文是笔者在学习setState更新机制时，在github的Discussions中看到的一篇文章。这让我更加了解了React17和React18中setState更新的不同之处。顺便自己翻译总结一下。

# 综述
React18默认通过批处理机制来添加开箱即用的性能改进，在应用或者代码仓库中移除了手动批量更新的机制。这个文章将会解释什么是批处理，它之前是怎么工作的，并且它改变了什么。

# 什么是批处理
批处理是React团队为了更好的性能，将多次更新合并成为单个渲染。React只会渲染一次即使你改变了两次state中的值。

```js
function App() {
  const [count, setCount] = useState(0);
  const [flag, setFlag] = useState(false);

  function handleClick() {
    setCount(c => c + 1); // Does not re-render yet
    setFlag(f => !f); // Does not re-render yet
    // React will only re-render once at the end (that's batching!)
  }

  return (
    <div>
      <button onClick={handleClick}>Next</button>
      <h1 style={{ color: flag ? "blue" : "black" }}>{count}</h1>
    </div>
  );
}
```
✅ [Demo:React 17](https://codesandbox.io/s/spring-water-929i6?file=/src/index.js)在React17中，如果是内部的事件处理，React也会进行批量更新


批量更新带来了更好的性能因为它避免了不必要的重复渲染。它也阻止了你的state只进行了一次更新，从而成为了一个半成品，同时也减少了bug的产生。这就相当于当你就点了一个菜的时候，餐馆的服务员不会立刻跑到厨房告诉后厨，而是等你点完所有的菜之后，才会把菜单提交上去。

然而，React并没有在所有的更新事件中都采用批处理的方式。比如，当你点击之后访问数据，更新state 状态等，React并没有采用批处理的方式进行更新，而是分别的进行更新。

这是因为React只会在浏览器事件中采用批处理(像点击事件)，但是我们更新state的状态是在fetch的回调函数中进行的。

```
function App() {
  const [count, setCount] = useState(0);
  const [flag, setFlag] = useState(false);

  function handleClick() {
    fetchSomething().then(() => {
      // React 17 and earlier does NOT batch these because
      // they run *after* the event in a callback, not *during* it
      setCount(c => c + 1); // Causes a re-render
      setFlag(f => !f); // Causes a re-render
    });
  }

  return (
    <div>
      <button onClick={handleClick}>Next</button>
      <h1 style={{ color: flag ? "blue" : "black" }}>{count}</h1>
    </div>
  );
}
```

🟡 Demo: React 17不会在它不能控制的事件中采用批处理。
在React 18之前，我们只在React能控制的事件处理程序中批量更新。默认情况下，promise、setTimeout、原生事件处理程序或任何其他事件内部的更新不会在React中批量执行。

# 什么是自动批量更新
React18中用createRoot代替了render，所有的更新都会是自动的批量更新，无论他们是从哪里产生的。

这意味着timesout,promise,原生的事件处理或者其他的事件都会被当成React内部事件进行批量更新。这将会减少rendering的次数，也会带来更好的性能。

```js
function App() {
  const [count, setCount] = useState(0);
  const [flag, setFlag] = useState(false);

  function handleClick() {
    fetchSomething().then(() => {
      // React 18 and later DOES batch these:
      setCount(c => c + 1);
      setFlag(f => !f);
      // React will only re-render once at the end (that's batching!)
    });
  }

  return (
    <div>
      <button onClick={handleClick}>Next</button>
      <h1 style={{ color: flag ? "blue" : "black" }}>{count}</h1>
    </div>
  );
}
```
✅ [Demo1:React 18](https://codesandbox.io/s/morning-sun-lgz88?file=/src/index.js)在React18中使用createRoot都会采用批处理更新的方式

🟡 [Demo2: React 18 ](https://codesandbox.io/s/jolly-benz-hb1zx?file=/src/index.js)没有使用createRoot不会进行批处理更新

```js
function handleClick() {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React will only re-render once at the end (that's batching!)
}
```

```
setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React will only re-render once at the end (that's batching!)
}, 1000);
```

```
fetch(/*...*/).then(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React will only re-render once at the end (that's batching!)
})
```

```
elm.addEventListener('click', () => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React will only re-render once at the end (that's batching!)
});
```
上述几种方式都会产生一样的效果。

原文地址：https://github.com/reactwg/react-18/discussions/21

# 总结
之前的React版本分成了半自动批量更新和手动批量更新。手动批量更新就是React内部提供了batchedUpdates方法，用于在batchedContext上下文环境中执行回调函数。它是将原先的上下文环境保存起来，然后通过或等于的操作，代表当前属于BatchedContext上下文环境，函数执行完成之后再恢复成之前的上下文环境。开发者可以手动调用batchedUpdates方法按需合并更新，所以称为手动批量更新。

V18版本之前会在合适的时机使用batchedUpdates方法执行回调函数。legacy 模式在合成事件中有自动批处理的功能，但仅限于一个浏览器任务。非 React 事件想使用这个功能必须使用 unstable_batchedUpdates。在 blocking 模式和 concurrent 模式下，所有的 setState 在默认情况下都是批处理的。会在开发中发出警告。

因为更改batchedContext的环境是同步的，当异步事件执行的时候，早已经跳出来batchedContext的执行环境了，executionContext已经不包含batchUpdates环境了，所以异步触发的更新是不能自动批量更新的。

React18之后，全部采用了自动批量更新的模式。交给了schedule阶段的调度策略完成了