```
// Tutorial 06 - dispatch-action.js
```

## 06 - 分发action

```
// So far we've focused on building our reducer(s) and we haven't dispatched any of our own actions.
// We'll keep the same reducers from our previous tutorial and handle a few actions:
```
到目前为止, 我们主要集中在构建我们的reducers, 而我们还没有分发过任何的actions.
我们将保留在前面几节中所写的reducer函数，并用他们来处理一些actions:

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
该代码会输出: 初始化后store_0的状态: { user: {}, items: [] }

```
// Let's dispatch our first action... Remember in 'simple-action-creator.js' we said:
//     "To dispatch an action we need... a dispatch function." Captain obvious
```
让我们开始分发第一个action... 记住我们在"简单的action构造器"一节中所提到的:

  "为了分发一个action, 我们需要的... 显然是一个分发函数."

```
// The dispatch function we're looking for is provided by Redux and will propagate our action
// to all of our reducers! The dispatch function is accessible through the Redux
// instance property "dispatch"
```
Redux为我们提供了我们所期望的分发函数,
而且该函数会将action分发到我们所有的reducers中!
通过Redux实例的"dispathc"属性来调用分发函数.

```
// To dispatch an action, simply call:
```
简单的如下调用来分发一个函数

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
输出:
传入参数state: {}和action { type: 'AN_ACTION' }到userReducer并调用
传入参数state: []和action { type: 'AN_ACTION' }到itemsReducer并调用

```
// Each reducer is effectively called but since none of our reducers care about this action type,
// the state is left unchanged:
```
每一个reducer都被有效的调用到了, 但是由于没有一个reducer处理该类型的action,
所以State依然褒词不变:

```js
console.log('store_0 state after action AN_ACTION:', store_0.getState())
// Output: store_0 state after action AN_ACTION: { user: {}, items: [] }
```
输出: 收到action: AN_ACTION后store_0的状态: { user: {}, items: [] }

```
// But, wait a minute! Aren't we supposed to use an action creator to send an action? We could indeed
// use an actionCreator but since all it does is return an action it would not bring anything more to
// this example. But for the sake of future difficulties let's do it the right way according to
// flux theory. And let's make this action creator send an action we actually care about:
```
但是, 等一下!
我们不是应该用action构造器来分发action吗？我们实际上可以用actionCreator,
但由于actionCreator所做的只是返回一个action,
对于我们这个例子将不会带来任何更多的东西. 不过出于为日后制造一些难处的缘故,
让我们根据flux理论使用正确的方式用action构造器来发送我们实际上所关注的那些action吧:

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

输出:
- 传入state: {}和action { type: 'SET_NAME', name: 'bob'
  }参数给userReducer并调用
- 使用state: []和action { type: 'SET_NAME', name: 'bob' } 参数调用itemsReducer

```js
console.log('store_0 state after action SET_NAME:', store_0.getState())
// Output:
// store_0 state after action SET_NAME: { user: { name: 'bob' }, items: [] }
```
输出:
- 接受到action: SET_NAME后, store_0状态是: { user: { name: 'bob' }, items: [] }


```
// We just handled our first action and it changed the state of our application!

// But this seems too simple and not close enough to a real use-case. For example,
// what if we'd like do some async work in our action creator before dispatching
// the action? We'll talk about that in the next tutorial "dispatch-async-action.js"

// So far here is the flow of our application
// ActionCreator -> Action -> dispatcher -> reducer
```
我们刚刚处理了我们的第一个action, 并且该action改变了我们程序的State!
但是这似乎也太简单了，还不足以接近真实的用例. 比如,
我们想在分发action前在action构造器中完成一个异步工作该怎么办呢？这将在下一节中揭晓!

目前我们程序的流程变成了:

** ActionCreator -> Action -> dispatcher -> reducer **

