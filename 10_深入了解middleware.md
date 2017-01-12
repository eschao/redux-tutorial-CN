## 深入了解 middleware

在上一节中我们简单的介绍了 middleware 并展示了其基本的用法. 但我们可能会疑惑: 
* 为什么 middleware 函数要遵循那样的签名及结构? 
* 为什么需要三层函数? 
* Redux 支持 middleware 的 applyMiddleware 函数到底又做了什么?

明白 middleware 的前因后果, 是我们理解 Redux 设计的一个很重要的部分. 在本节中, 我会以翻译官方的 Middleware 一节为主并辅以我自己的理解来解析 middleware , 希望能让你更深入一步地了解 middleware.

### 问题: 记录日志

Redux 的带来的一个好处就是它使得 state 的变更是可预测且透明的. 每次当 action 被分发的时候, 新的 state 就会被计算并保存下来. state是不能被自身所改变的, 它只能因特殊 action 的结果而改变.
要是能记录下在程序中发生的每一个 action 和其后被计算的 state 是不是很好呢? 当某个地方出现问题的时候, 我们就能够回溯所有的日志, 然后找出是哪个 action 破坏了 state.

我们该如何使用 Redux 来做到呢?

### 首次尝试: 手工记录

最笨的方法就是在每次调用```js store.dispatch(action)```之后你手工的记录下 action 以及下一个 state. 这并非真正的解决方案, 但却向理解问题迈出了第一步.

>##### 注意
如果你正在使用 react-redux 或其他类似的绑定框架, 你可能不会在你的组件里直接的访问 store 实例. 因而对于接下来的内容, 假定你显示的将 store 实例传递了下来

比如说, 你会这样的来创建一个 todo:
```js
store.dispatch(addTodo('Use Redux'))
```
为了记录 action 和 state , 你可能会改成这样:
```js
let action = addTodo('Use Redux')

console.log('dispatching', action)
store.dispatch(action)
console.log('next state', store.getState())
```
这些代码能产生期望的结果, 但你不会想每次都这么做.

### 第二次尝试: 
