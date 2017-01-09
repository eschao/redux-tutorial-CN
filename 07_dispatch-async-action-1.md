```
// Tutorial 07 - dispatch-async-action-1.js
```

## 07 - 分发一个异步action-1

```
// We previously saw how we can dispatch actions and how those actions will modify
// the state of our application thanks to reducers.
```
我们在之前的章节中已经看到了是怎样的分发action以及这些action是如何的修改我们程序的状态的, 这要感谢reducers函数

```
// But so far we've only considered synchronous actions or, more exactly, action creators
// that produce an action synchronously: when called an action is returned immediately.
```
但迄今为止我们仅仅考虑到了同步actions, 或者更准确点,
就是那些同步地产生actions的action构造器: 也就是说当调用action构造器时,
action会立即的产生并返回给调用者

```
// Let's now imagine a simple asynchronous use-case:
// 1) user clicks on button "Say Hi in 2 seconds"
// 2) When button "A" is clicked, we'd like to show message "Hi" after 2 seconds have elapsed
// 3) 2 seconds later, our view is updated with the message "Hi"
```
我们来思考一个简单的异步用例:
* 用户点击按钮"2秒后显示Hi"
* 当按钮"A"被点击后, 我们想在2秒过后显示一个消息"Hi"
* 2秒之后, 我们的视图会被更新并显示消息"Hi"

```
// Of course this message is part of our application state so we have to save it
// in Redux store. But what we want is to have our store save the message
// only 2 seconds after the action creator is called (because if we were to update our state
// immediately, any subscriber to state's modifications - like our view - would be notified right away
// and would then react to this update 2 seconds too soon).

// If we were to call an action creator like we did until now...
```
当然, 这个消息也是我们程序状态的一部分了. 所以我们要将它存在Redux的store中.
但我们想要的是让我们的store在action构造器别调用2秒后保存下该消息(...)
如果我们还想目前为止我们所做过的那样去调用action构造器...

```js
import { createStore, combineReducers } from 'redux'

var reducer = combineReducers({
    speaker: function (state = {}, action) {
        console.log('speaker was called with state', state, 'and action', action)

        switch (action.type) {
            case 'SAY':
                return {
                    ...state,
                    message: action.message
                }
            default:
                return state;
        }
    }
})
var store_0 = createStore(reducer)

var sayActionCreator = function (message) {
    return {
        type: 'SAY',
        message
    }
}

console.log("\n", 'Running our normal action creator:', "\n")

console.log(new Date());
store_0.dispatch(sayActionCreator('Hi'))

console.log(new Date());
console.log('store_0 state after action SAY:', store_0.getState())
// Output (skipping initialization output):
//     Sun Aug 02 2015 01:03:05 GMT+0200 (CEST)
//     speaker was called with state {} and action { type: 'SAY', message: 'Hi' }
//     Sun Aug 02 2015 01:03:05 GMT+0200 (CEST)
//     store_0 state after action SAY: { speaker: { message: 'Hi' } }
```
程序会输出(忽略掉初始化部分):
```js
     Sun Aug 02 2015 01:03:05 GMT+0200 (CEST)
     speaker was called with state {} and action { type: 'SAY', message: 'Hi' }
     Sun Aug 02 2015 01:03:05 GMT+0200 (CEST)
     store_0 state after action SAY: { speaker: { message: 'Hi' } }
```

```
// ... then we see that our store is updated immediately.
```
...这样我们发现我们的store立即就被更新了.

```
// What we'd like instead is an action creator that looks a bit like this:
```
想法我们想要的这个action构造器看起来有点像这样:

```js
var asyncSayActionCreator_0 = function (message) {
    setTimeout(function () {
        return {
            type: 'SAY',
            message
        }
    }, 2000)
}
```

```
// But then our action creator would not return an action, it would return "undefined". So this is not
// quite the solution we're looking for.
```
可这样我们的action构造器并不会返回一个action, 它将返回"undefined".
所以这并非就是一个我们想要的解决办法.

```
// Here's the trick: instead of returning an action, we'll return a function. And this function will be the
// one to dispatch the action when it seems appropriate to do so. But if we want our function to be able to
// dispatch the action it should be given the dispatch function. Then, this should look like this:
```
技巧在于: 我们将返回一个函数而不是一个action.
而这个函数将在恰当的时机去分发一个action.
但如果我们想要该函数能够分发一个action而又应该接受一个dispatch函数. 那么,
这个函数应该看起来像这样:

```js
var asyncSayActionCreator_1 = function (message) {
    return function (dispatch) {
        setTimeout(function () {
            dispatch({
                type: 'SAY',
                message
            })
        }, 2000)
    }
}
```

```
// Again you'll notice that our action creator is not returning an action, it is returning a function.
// So there is a high chance that our reducers won't know what to do with it. But you never know, so let's
// try it out and find out what happens...
```
再一起，你会发现我们的action构造器不会返回一个action, 而是返回一个函数.
所以我们的reducer函数很有可能不知道该怎么处理.
但你是知道他们没法处理的，那么让我们试试看并找出到底会发生什么...

