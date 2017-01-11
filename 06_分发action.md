```
// Tutorial 06 - dispatch-action.js
```

## 06 - 分发 action

```
// So far we've focused on building our reducer(s) and we haven't dispatched any of our own actions.
// We'll keep the same reducers from our previous tutorial and handle a few actions:
```
到目前为止, 我们主要集中在构建我们的 reducers上, 而我们还没有分发过任何的 actions .

我们将保留在前面几节中所写的 reducer 函数，并用他们来处理一些 actions :

```js
var userReducer = function (state = {}, action) {
    console.log('userReducer was called with state', state, 'and action', action)

    switch (action.type) {
        case 'SET_NAME':
            return {
                ...state,
                name: action.name
            }
        default:
            return state;
    }
}
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

import { createStore, combineReducers } from 'redux'

var reducer = combineReducers({
    user: userReducer,
    items: itemsReducer
})
var store_0 = createStore(reducer)


console.log("\n", '### It starts here')
console.log('store_0 state after initialization:', store_0.getState())
```
```
// Output:
// store_0 state after initialization: { user: {}, items: [] }
```
运行后输出:
```js
    store_0 state after initialization: { user: {}, items: [] }
```

```
// Let's dispatch our first action... Remember in 'simple-action-creator.js' we said:
//     "To dispatch an action we need... a dispatch function." Captain obvious
```
让我们开始分发第一个 action ... 记住我们在[简单的action构造器]()一节中所提到的:

>>>
  "为了分发一个 action , 我们需要的... 显然是一个分发函数."

```
// The dispatch function we're looking for is provided by Redux and will propagate our action
// to all of our reducers! The dispatch function is accessible through the Redux
// instance property "dispatch"
```
Redux 为我们提供了我们所期望的分发函数, 而且该函数会将 action 分发到我们所有的 reducers 中! 通过 Redux 实例的 **dispatch** 属性我们就能调用该分发函数.

```
// To dispatch an action, simply call:
```
为了分发一个 action , 调用如下:

```js
store_0.dispatch({
    type: 'AN_ACTION'
})
```
```
// Output:
// userReducer was called with state {} and action { type: 'AN_ACTION' }
// itemsReducer was called with state [] and action { type: 'AN_ACTION' }
```
运行后输出:
```js
    userReducer was called with state {} and action { type: 'AN_ACTION' }
    itemsReducer was called with state [] and action { type: 'AN_ACTION' }
```

```
// Each reducer is effectively called but since none of our reducers care about this action type,
// the state is left unchanged:
```
每一个 reducer 都被有效的调用到了, 但是由于没有一个 reducer 处理该类型的 action , 所以 state 依然保持不变:

```js
console.log('store_0 state after action AN_ACTION:', store_0.getState())
// Output: store_0 state after action AN_ACTION: { user: {}, items: [] }
```
运行后输出: 
```js
    store_0 state after action AN_ACTION: { user: {}, items: [] }
```

```
// But, wait a minute! Aren't we supposed to use an action creator to send an action? We could indeed
// use an actionCreator but since all it does is return an action it would not bring anything more to
// this example. But for the sake of future difficulties let's do it the right way according to
// flux theory. And let's make this action creator send an action we actually care about:
```
但是, 等一下! 我们不是应该用 action creator 来发送 action 吗？我们实际上可以用 actionCreator ,
但由于 actionCreator 所做的只是返回一个 action 对象, 对于本例子将不会带来更多的东西. 不过为了日后可能出现的问题, 让我们遵循 flux 理论使用正确的方式来做吧. 用 action creator 来发送那些我们实际关注的 action :

```js
var setNameActionCreator = function (name) {
    return {
        type: 'SET_NAME',
        name: name
    }
}

store_0.dispatch(setNameActionCreator('bob'))
```
```
// Output:
// userReducer was called with state {} and action { type: 'SET_NAME', name: 'bob' }
// itemsReducer was called with state [] and action { type: 'SET_NAME', name: 'bob' }
```

运行后输出:
```js
    userReducer was called with state {} and action { type: 'SET_NAME', name: 'bob' }
    itemsReducer was called with state [] and action { type: 'SET_NAME', name: 'bob' }
```

```js
console.log('store_0 state after action SET_NAME:', store_0.getState())
// Output:
// store_0 state after action SET_NAME: { user: { name: 'bob' }, items: [] }
```
运行后输出:
```js
    store_0 state after action SET_NAME: { user: { name: 'bob' }, items: [] }
```

```
// We just handled our first action and it changed the state of our application!

// But this seems too simple and not close enough to a real use-case. For example,
// what if we'd like do some async work in our action creator before dispatching
// the action? We'll talk about that in the next tutorial "dispatch-async-action.js"

// So far here is the flow of our application
// ActionCreator -> Action -> dispatcher -> reducer
```
我们刚刚处理了我们第一个 action , 并且该 action 也改变了我们程序的 state!

但这似乎也太简单了，并不足以接近真实的用例. 比如, 如果我们想在分发 action 前在 action creator 中完成一些异步操作呢？这将在下一节中揭晓!

目前我们程序的流程变成了:

**action creator -> action -> dispatcher -> reducer**

