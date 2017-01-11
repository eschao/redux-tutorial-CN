```
// Tutorial 07 - dispatch-async-action-1.js
```

## 07 - 分发异步 action -1

```
// We previously saw how we can dispatch actions and how those actions will modify
// the state of our application thanks to reducers.
```
我们在之前的章节中已经看到了如何的分发 action 以及这些 action 是如何的修改我们程序 state 的, 这要感谢 reducers 函数.

```
// But so far we've only considered synchronous actions or, more exactly, action creators
// that produce an action synchronously: when called an action is returned immediately.
```
但迄今为止我们仅仅考虑到了同步 actions , 或者更准确点说, 就是那些同步地产生 actions 的 action creator: 也就是说当调用 action creator时,
action 会立即的产生并返回给调用者.

```
// Let's now imagine a simple asynchronous use-case:
// 1) user clicks on button "Say Hi in 2 seconds"
// 2) When button "A" is clicked, we'd like to show message "Hi" after 2 seconds have elapsed
// 3) 2 seconds later, our view is updated with the message "Hi"
```
我们来思考一个简单的异步用例:
* 用户点击按钮 "2秒后显示Hi"
* 当按钮 "A" 被点击后, 我们想在2秒过后显示一个消息 "Hi"
* 2秒之后, 我们的 view 会被更新并显示消息 "Hi"

```
// Of course this message is part of our application state so we have to save it
// in Redux store. But what we want is to have our store save the message
// only 2 seconds after the action creator is called (because if we were to update our state
// immediately, any subscriber to state's modifications - like our view - would be notified right away
// and would then react to this update 2 seconds too soon).

// If we were to call an action creator like we did until now...
```
当然, 这个消息也是我们程序 state 的一部分了. 所以我们要将它保存在 Redux 的 store 中. 但我们想要的是让我们的 store 在 action creator被调用2秒后保存下该消息(因为如果我们立即更新程序的 state , 任何一个关注 state 修改的 subscriber - 比如 view -- 都将马上被通知并可能未及2秒就进行了更新)

如果我们还像目前为止我们所做过的那样去调用 action creator ...

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
运行后输出(忽略掉初始化部分):
```js
     Sun Aug 02 2015 01:03:05 GMT+0200 (CEST)
     speaker was called with state {} and action { type: 'SAY', message: 'Hi' }
     Sun Aug 02 2015 01:03:05 GMT+0200 (CEST)
     store_0 state after action SAY: { speaker: { message: 'Hi' } }
```

```
// ... then we see that our store is updated immediately.
```
...然后我们看到我们的 store 立即就被更新了.

```
// What we'd like instead is an action creator that looks a bit like this:
```
而我们想要的这个 action creator 看起来有点像这样:

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
可这样的话, 我们的 action creator 并不会返回一个 action , 它将返回 **"undefined"**. 所以这并非就是一个我们想要的解决方案.

```
// Here's the trick: instead of returning an action, we'll return a function. And this function will be the
// one to dispatch the action when it seems appropriate to do so. But if we want our function to be able to
// dispatch the action it should be given the dispatch function. Then, this should look like this:
```
而技巧在于: 我们将返回一个函数而不是一个 action . 而这个函数将在恰当的时机去分发一个 action .
但如果我们想要该函数能够分发一个 action 而又要接受一个 dispatch 函数. 那么, 这个函数看起来应该像这样:

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
再一次，你会发现我们的 action creator 不是返回一个 action , 而是返回一个函数. 所以我们的 reducer 函数很有可能不知道该怎么去处理. 但你却并不清楚，那么让我们试试看并找出到底会发生什么 ...

