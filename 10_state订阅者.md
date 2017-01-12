```
// Tutorial 10 - state-subscriber.js
```

## 10 - State 订阅者

```
// We're close to having a complete Flux loop but we still miss one critical part:
```
我们快要拥有一个完整的 Flux 环路了, 但我们仍然缺少一个重要的部分:

```
//  _________      _________       ___________
// |         |    | Change  |     |   React   |
// |  Store  |----▶ events  |-----▶   Views   |
// |_________|    |_________|     |___________|
```

```
// Without it, we cannot update our views when the store changes.
```
没有这部分, 我们就无法在 store 改变的时候更新 views.

```
// Fortunately, there is a very simple way to "watch" over our Redux store's updates:
```
幸运地是, 这儿有一个非常简单的方法去"监视" Redux store 的更新:

```js
    store.subscribe(function() {
        // retrieve latest store state here
        // Ex:
        console.log(store.getState());
    })
```

```
// Yeah... So simple that it almost makes us believe in Santa Claus again.
```
没错... 就如此简单, 这几乎让我们再次相信有圣诞老人.

```
// Let's try this out:
```
让我们试着运行一下这段代码:

```js
import { createStore, combineReducers } from 'redux'

var itemsReducer = function (state = [], action) {
    console.log('itemsReducer was called with state', state, 'and action', action)

    switch (action.type) {
        case 'ADD_ITEM':
            return [
                ...state,
                action.item
            ]
        default:
            return state;
    }
}

var reducer = combineReducers({ items: itemsReducer })
var store_0 = createStore(reducer)

store_0.subscribe(function() {
    console.log('store_0 has been updated. Latest store state:', store_0.getState());
    // Update your views here
})

var addItemActionCreator = function (item) {
    return {
        type: 'ADD_ITEM',
        item: item
    }
}

store_0.dispatch(addItemActionCreator({ id: 1234, description: 'anything' }))
```

```
// Output:
//     ...
//     store_0 has been updated. Latest store state: { items: [ { id: 1234, description: 'anything' } ] }
```
运行后输出:
```js
     ...
     store_0 has been updated. Latest store state: { items: [ { id: 1234, description: 'anything' } ] }
```

```
// Our subscribe callback is correctly called and our store now contains the new item that we added.
```
我们的 subscribe 回调函数被正确地执行, 并且我们的 store 也包含了我们刚刚添加的新数据项.

```
// Theoretically speaking we could stop here. Our Flux loop is closed, we understood all concepts that make
// Flux and we saw that it is not that much of a mystery. But to be honest, there is still a lot to talk
// about and a few things in the last example were intentionally left aside to keep the simplest form of this
// last Flux concept:
```
理论上讲, 我们可以就此打住了. 我们的 Flux 环路已完整, 我们知道了创建 Flux 程序的所有概念, 并且我们也看到它其实没有多么的神秘. 但老实说, 我们仍然有许多话题可以讨论, 而且出现在最后这个例子中的一些东西是有意被留下来让最后一个 Flux 概念保持着简单的形式.

```
// - Our subscriber callback did not receive the state as a parameter, why?
// - Since we did not receive our new state, we were bound to exploit our closured store (store_0) so this
//     solution is not acceptable in a real multi-modules application...
// - How do we actually update our views?
// - How do we unsubscribe from store updates?
// - More generally speaking, how should we integrate Redux with React?
```
* 我们的订阅回调函数并没有接受一个 state 参数, 为什么呢？
* 由于我们并没有接受到一个新的 state , 我们必须使用我们的闭合的 store (store_0) , 而该方案在真实的多模块程序中是无法被接受的...
* 我们实际上是如何更新 views 的呢？
* 我们如何取消对 state 更新的订阅呢?
* 更广泛地说, 我们如何整合 Redux 和 React?

```
// We're now entering a more "Redux inside React" specific domain.
```
目前我们正走进一个特殊领域: "位于 React 内的 Redux ".

```
// It is very important to understand that Redux is by no means bound to React. It is really a
// "Predictable state container for JavaScript apps" and you can use it in many ways, a React
// application just being one of them.
```
明白 Redux 绝不会与 React 绑在一起这一点非常重要. Redux 事实上是一个"JavaScript应用程序的可预测状态容器", 而你可以将其应用在许多的方面, React 程序只不过是其中之一而已.

```
// In that perspective we would be a bit lost if it wasn't for react-redux (https://github.com/rackt/react-redux).
// Previously integrated inside Redux (before 1.0.0), this repository holds all the bindings we need to simplify
// our life when using Redux inside React.
```
从此角度来看, 如果这一切都不是为了 [react-redux](https://github.com/rackt/react-redux), 我们会感到一丝的困惑.

在1.0.0以前, React 已经整合了 Redux , 并包含了所有我们需要的绑定以此简化了我们在 React 中使用 Redux日常工作.

```
// Back to our "subscribe" case... Why exactly do we have this subscribe function that seems so simple but at
// the same time also seems to not provide enough features?
```
回到我们的 "subscribe" 例子... 究竟为什么我们需要一个看起来简单但同时似乎又没有提供足够多特性的 subscribe 函数呢？

```
// Its simplicity is actually its power! Redux, with its current minimalist API (including "subscribe") is
// highly extensible and this allows developers to build some crazy products like the Redux DevTools
// (https://github.com/gaearon/redux-devtools).
```
它的力量实际上正源于它的简洁! 有着极简主义思想API的Redux是高度可扩展的,
它允许开发者去构建一些疯狂的作品, 比如Redux DevTools.

```
// But in the end we still need a "better" API to subscribe to our store changes. That's exactly what react-redux
// brings us: an API that will allow us to seamlessly fill the gap between the raw Redux subscribing mechanism
// and our developer expectations. In the end, you won't need to use "subscribe" directly. Instead you will
// use bindings such as "provide" or "connect" and those will hide from you the "subscribe" method.
```
不过最终我们仍需要一个"较好"的API去订阅Store的变化.
那正是react-redux给我们带来的:
一个好的API能够无缝地将开发者的期望与纯Redux订阅机制衔接起来. 最后,
开发者无需直接使用"订阅". 而是使用诸如"rpovide"或"connect"之类的绑定,
并且这些绑定会对开发者隐藏"subscribe"方法.

```
// So yeah, the "subscribe" method will still be used but it will be done through a higher order API that
// handles access to Redux state for you.
```
所以呢,
"subscribe"方法扔将被使用只不过它是通过一个更高阶且能替你访问Redux状态的API来完成的.

```
// We'll now cover those bindings and show how simple it is to wire your components to Redux's state.
```
现在我们将介绍这些绑定并展示如何简单地将你的组件和Redux状态接通.
