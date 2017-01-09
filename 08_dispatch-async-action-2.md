```
// Tutorial 08 - dispatch-async-action-2.js
```

## 08 - 分发异步action(2)

```
// Let's try to run the first async action creator that we wrote in dispatch-async-action-1.js.
```
让我们试着运行一下在上一节中所写的第一个异步action构造器

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

console.log("\n", 'Running our async action creator:', "\n")
store_0.dispatch(asyncSayActionCreator_1('Hi'))
```
```
// Output:
//     ...
//     /Users/classtar/Codes/redux-tutorial/node_modules/redux/node_modules/invariant/invariant.js:51
//         throw error;
//               ^
//     Error: Invariant Violation: Actions must be plain objects. Use custom middleware for async actions.
//     ...
```
程序运行后输出:
```js
     ...
     /Users/classtar/Codes/redux-tutorial/node_modules/redux/node_modules/invariant/invariant.js:51
         throw error;
               ^
     Error: Invariant Violation: Actions must be plain objects. Use custom middleware for async actions.
     ...
```

```
// It seems that our function didn't even reach our reducers. But Redux has been kind enough to give us a
// tip: "Use custom middleware for async actions.". It looks like we're on the right path but what is this
// "middleware" thing?
```
似乎我们的函数根本就没有运行到我们的reducers.
但Redux已给与我们一个足够善意的提示: "为异步actions使用自定义的middleware".
看起来我们正走在正确的路上, 但什么是"middleware"呢？

```
// Just to reassure you, our action creator asyncSayActionCreator_1 is well-written and will work as expected
// as soon as we've figured out what middleware is and how to use it.
```
只是为了打消你的疑虑,
我们的action构造器asyncSayActionCreator_1写的没错,
只要我们明白了什么是middleware并且知道如何使用它，异步action构造器就能如期的工作了.
