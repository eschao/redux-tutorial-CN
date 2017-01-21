```
// Tutorial 04 - get-state.js
```
## 04 - 获取 state

```
// How do we retrieve the state from our Redux instance?
```
我们如何从 Redux 实例中获取到程序的 state 呢? 请看如下这段代码:

```js
import { createStore } from 'redux'

var reducer_0 = function (state, action) {
    console.log('reducer_0 was called with state', state, 'and action', action)
}

var store_0 = createStore(reducer_0)
```
```
// Output: reducer_0 was called with state undefined and action { type: '@@redux/INIT' }
```
运行后输出: 
```js
    reducer_0 was called with state undefined and action { type: '@@redux/INIT' }
```

```
// To get the state that Redux is holding for us, you call getState
```
你可以通过调用 **getState** 函数来获取 Redux 为我们持有的程序 state

```js
console.log('store_0 state after initialization:', store_0.getState())
```
```
// Output: store_0 state after initialization: undefined
```
运行后输出:
```js
    store_0 state after initialization: undefined
```

```
// So the state of our application is still undefined after the initialization? Well of course it is,
// our reducer is not doing anything... Remember how we described the expected behavior of a reducer in
// "about-state-and-meet-redux"?
//     "A reducer is just a function that receives the current state of your application, the action,
//     and returns a new state modified (or reduced as they call it)"
// Our reducer is not returning anything right now so the state of our application is what
// reducer() returns, hence "undefined".
```
那么我们程序的 state 在初始化后仍然是 **undefined** 吗? 好吧, 它当然就该如此, 这是因为我们的 reduce r什么也没做...

还记得我们在第二节中是如何描述我们所期望的 reducer 行为吗?
>>>
"Reducer 仅仅是一个函数，这个函数接受你程序当前的 state 和 action , 并返回一个被修改后的新 state "
  
由于目前我们的 reducer 什么也没有返回, 因此程序的 state 也就是 ```reducer()``` 函数返回的 **"undefined"**.

```
// Let's try to send an initial state of our application if the state given to reducer is undefined:
```
如果传给 reducer 函数的 state 是 undefined 的话, 我们就试着赋予程序一个初始化的 state :
```js
var reducer_1 = function (state, action) {
    console.log('reducer_1 was called with state', state, 'and action', action)
    if (typeof state === 'undefined') {
        return {}
    }

    return state;
}

var store_1 = createStore(reducer_1);
console.log('store_1 state after initialization:', store_1.getState());
```
```
// Output: reducer_1 was called with state undefined and action { type: '@@redux/INIT' }
// Output: store_1 state after initialization: {}
```
运行后输出:··
```js
    reducer_1 was called with state undefined and action { type: '@@redux/INIT' }
    store_1 state after initialization: {}
```
```
// As expected, the state returned by Redux after initialization is now {}
```
正如我们所期望的一样, 在初始化之后 Redux 返回的程序 state 如今变为 **{}**

```
// There is however a much cleaner way to implement this pattern thanks to ES6:
```
这要感谢 ES6 的语法，我们可以用一个更清晰的方式来实现同样的功能:

```js
var reducer_2 = function (state = {}, action) {
    console.log('reducer_2 was called with state', state, 'and action', action)

    return state;
}

var store_2 = createStore(reducer_2)
console.log('store_2 state after initialization:', store_2.getState())
```
```
// Output: reducer_2 was called with state {} and action { type: '@@redux/INIT' }
// Output: store_2 state after initialization: {}
```
运行后输出: 
```js
    reducer_2 was called with state {} and action { type: '@@redux/INIT' }
    store_2 state after initialization: {}
```
```
// You've probably noticed that since we've used the default parameter on state parameter of reducer_2,
// we no longer get undefined as state's value in our reducer's body.
```
你可能已经注意到由于我们在 reducer_2 函数的 state 参数上使用了默认值，我们再也不会在 reducer 的函数体内获得 **undefined** 的 state

```
// Let's now recall that a reducer is only called in response to an action dispatched and
// let's fake a state modification in response to an action type 'SAY_SOMETHING'
```
回想一下, 一个 reducer 函数仅仅当响应一个被分发的 action 时才会被调用, 既然如此我们就伪造一个修改 state 的行为用来响应类型为 'SYA_SOMETHING' 的 action

```js
var reducer_3 = function (state = {}, action) {
    console.log('reducer_3 was called with state', state, 'and action', action)

    switch (action.type) {
        case 'SAY_SOMETHING':
            return {
                ...state,
                message: action.value
            }
        default:
            return state;
    }
}

var store_3 = createStore(reducer_3);
console.log('store_3 state after initialization:', store_3.getState());
```
```
// Output: reducer_3 was called with state {} and action { type: '@@redux/INIT' }
// Output: store_3 state after initialization: {}
```
运行后输出: 

```js
    reducer_3 was called with state {} and action { type: '@@redux/INIT' }
    store_3 state after initialization: {}
```
```
// Nothing new in our state so far since we did not dispatch any action yet. But there are few
// important things to pay attention to in the last example:
//     0) I assumed that our action contains a type and a value property. The type property is mostly
//        a convention in flux actions and the value property could have been anything else.
//     1) You'll often see the pattern involving a switch to respond 
//        to an action received in your reducers
//     2) When using a switch, NEVER forget to have a "default: return state" because
//        if you don't, you'll end up having your reducer return undefined (and lose your state).
//     3) Notice how we returned a new state made by merging current state with { message: action.value },
//        all that thanks to this awesome ES7 notation (Object Spread): { ...state, message: action.value }
//     4) Note also that this ES7 Object Spread notation suits our example because it's doing a shallow
//        copy of { message: action.value } over our state (meaning that first level properties of state
//        are completely overwritten - as opposed to gracefully merged - by first level property of
//        { message: action.value }). But if we had a more complex / nested data structure, you might choose
//        to handle your state's updates very differently:
//        - using Immutable.js (https://facebook.github.io/immutable-js/)
//        - using Object.assign (https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)
//        - using manual merge
//        - or whatever other strategy that suits your needs and the structure of your state since
//          Redux is absolutely NOT opinionated on this (remember, Redux is a state container).
```
目前为止我们程序的 state 没有任何变化，这是因为我们还没有分发任何的 action . 但在最近的这个例子中，有一些重要的东西需要我们注意:
* 该例子假设我们的 action 包含一个 **type** 属性和一个 **value** 属性. **type** 属性是 Flux actions 中最主要的一个约定, 而 **value**属性可以拥有其他的任何值
* 你将在你的 reducers 函数中经常看到这样的代码范式: 使用 ```switch``` 语句来合适地处理 reducer 函数接受到的 action.
* 当使用 ```switch``` 语句的时候，永远也不要忘记这一行语句: **"defualt: return state"**, 因为一旦缺失这一行，你的 reducer 函数就可能返回一个 **undefined** 的 state .
* 注意我们是如何通过合并当前的 stat e和 **{message: action.value}** 来返回一个新的 state 的, 这都要归功于 ES7 超棒的对象展开语法: { ...state, message: action.value }
* 同时我们也要注意之所以 ES7 的对象展开语法适合本例子, 是因为它基于我们的 state 做了一个 {message: action.value} 的浅拷贝 (...). 但如果我们拥有一个更加复杂且内嵌的数据结构, 你可能就会选择一个完全不同的方式来更新你程序的 state , 比如:
  - 使用[Immutable.js](https://facebook.github.io/immutable-js/)
  - 使用[Object.assign](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)
  - 手工合并
  - 或者其他任何适合你需求和程序数据结构的方法, 因为 Redux 对此绝无任何的限制.(记住, Redux 是一个 state 容器)

```
// Now that we're starting to handle actions in our reducer let's talk about having multiple reducers and
// combining them.
```
既然我们已开始在 reducer 中处理 actions 了，那么让我们看看拥有多个 reducer 函数的情况以及如何将他们合并在一起.

