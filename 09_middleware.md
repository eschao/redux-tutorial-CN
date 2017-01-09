```
// Tutorial 09 - middleware.js
```

## 09 - 中间件middleware

```
// We left dispatch-async-action-2.js with a new concept: "middleware". Somehow middleware should help us
// to solve async action handling. So what exactly is middleware?
```
在上一节中我们留下了一个新的概念: "中间件middleware".
不管怎样middleware会帮助我们解决异步action的处理问题. 那么到底什么是middleware?

```
// Generally speaking middleware is something that goes between parts A and B of an application to
// transform what A sends before passing it to B. So instead of having:
// A -----> B
// we end up having
// A ---> middleware 1 ---> middleware 2 ---> middleware 3 --> ... ---> B
```
总的来说middleware是一种运行在程序A部分和B部分之间用于将A发送给B的数据在传送给B之前先做一些变换的组件.  因此将不再是:

  **A -----> B**

而最终会是:

  **A ---> middleware 1 ---> middleware 2 ---> middleware 3 ---> ... ---> B**

```
// How could middleware help us in the Redux context? Well it seems that the function that we are
// returning from our async action creator cannot be handled natively by Redux but if we had a
// middleware between our action creator and our reducers, we could transform this function into something
// that suits Redux:
```
在Redux的上下文中middleware是如何帮组我们的呢？哦看起来从我们异步action构造器中返回的函数是不能够直接地被Redux处理, 但如果有一个中间件位于我们的action构造器和reducers之间， 我们就能够将该函数转换成适合Redux的东西:

```
// action ---> dispatcher ---> middleware 1 ---> middleware 2 ---> reducers
```
**action ---> dispatcher ---> middleware 1 ---> middleware 2 ---> reducers**

```
// Our middleware will be called each time an action (or whatever else, like a function in our
// async action creator case) is dispatched and it should be able to help our action creator
// dispatch the real action when it wants to (or do nothing - this is a totally valid and
// sometimes desired behavior).
```
我们的中间件在每次action被分发的时候都会被调用,
并且它能够在需要的时候帮助我们的action构造器去分发一个真实的action

```
// In Redux, middleware are functions that must conform to a very specific signature and follow
// a strict structure:
```
在Redux中, 中间件就是一堆必须遵循特殊声明和规则的函数, 比如:

```js
    var anyMiddleware = function ({ dispatch, getState }) {
        return function(next) {
            return function (action) {
                // your middleware-specific code goes here
            }
        }
    }
```

```
// As you can see above, a middleware is made of 3 nested functions (that will get called sequentially):
// 1) The first level provides the dispatch function and a getState function (if your
//     middleware or your action creator needs to read data from state) to the 2 other levels
// 2) The second level provides the next function that will allow you to explicitly hand over
//     your transformed input to the next middleware or to Redux (so that Redux can finally call all reducers).
// 3) the third level provides the action received from the previous middleware or from your dispatch
//     and can either trigger the next middleware (to let the action continue to flow) or process
//     the action in any appropriate way.
```
正如你在上面所看到的, 一个中间件由3个内嵌的函数组成(他们会被依次调用):
* 第一层提供一个dispatch函数和getState函数作为参数(如果你的中间件或action构造器需要从状态中读取数据)给第二层的函数
* 第二层会提供一个next函数参数给第三层,
  该参数允许你显示地将变换后的输入交给下一层middleware或者Redux(以便Redux能最终调用到所有的reducers).
* 第三层提供一个action参数, 该参数来自于前一个中间件或你自己dispatch函数,
  并且能够触发下一个middleware或者正确地处理掉该action.

```
// Those of you who are trained to functional programming may have recognized above an opportunity
// to apply a functional pattern: currying (if you aren't, don't worry, skipping the next 10 lines
// won't affect your Redux understanding). Using currying, you could simplify the above function like that:
```
那些具有函数编程背景的人可能意识到有机会将一个函数编程范式应用到上述中:
currying柯里化(如果你不知道，不用担心，可以跳过下面10行,
这并不会影响到你对Redux的理解). 使用Currying, 就能将上述函数简化为:
```js
    // "curry" may come from any functional programming library (lodash, ramda, etc.)
    var thunkMiddleware = curry(
        ({dispatch, getState}, next, action) => (
            // your middleware-specific code goes here
        )
    );
```

```
// The middleware we have to build for our async action creator is called a thunk middleware and
// its code is provided here: https://github.com/gaearon/redux-thunk.
// Here is what it looks like (with function body translated to es5 for readability):
```

```js
var thunkMiddleware = function ({ dispatch, getState }) {
    // console.log('Enter thunkMiddleware');
    return function(next) {
        // console.log('Function "next" provided:', next);
        return function (action) {
            // console.log('Handling action:', action);
            return typeof action === 'function' ?
                action(dispatch, getState) :
                next(action)
        }
    }
}
```

```
// To tell Redux that we have one or more middlewares, we must use one of Redux's
// helper functions: applyMiddleware.
```

```
// "applyMiddleware" takes all your middlewares as parameters and returns a function to be called
// with Redux createStore. When this last function is invoked, it will produce "a higher-order
// store that applies middleware to a store's dispatch".
// (from https://github.com/rackt/redux/blob/v1.0.0-rc/src/utils/applyMiddleware.js)
```

```js
// Here is how you would integrate a middleware into your Redux store:

import { createStore, combineReducers, applyMiddleware } from 'redux'

const finalCreateStore = applyMiddleware(thunkMiddleware)(createStore)
// For multiple middlewares, write: applyMiddleware(middleware1, middleware2, ...)(createStore)

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
                return state
        }
    }
})

const store_0 = finalCreateStore(reducer)
```
```
// Output:
//     speaker was called with state {} and action { type: '@@redux/INIT' }
//     speaker was called with state {} and action { type: '@@redux/PROBE_UNKNOWN_ACTION_s.b.4.z.a.x.a.j.o.r' }
//     speaker was called with state {} and action { type: '@@redux/INIT' }
```

```js
// Now that we have our middleware-ready store instance, let's try again to dispatch our async action:

var asyncSayActionCreator_1 = function (message) {
    return function (dispatch) {
        setTimeout(function () {
            console.log(new Date(), 'Dispatch action now:')
            dispatch({
                type: 'SAY',
                message
            })
        }, 2000)
    }
}

console.log("\n", new Date(), 'Running our async action creator:', "\n")

store_0.dispatch(asyncSayActionCreator_1('Hi'))
```
```
// Output:
//     Mon Aug 03 2015 00:01:20 GMT+0200 (CEST) Running our async action creator:
//     Mon Aug 03 2015 00:01:22 GMT+0200 (CEST) 'Dispatch action now:'
//     speaker was called with state {} and action { type: 'SAY', message: 'Hi' }
```

```
// Our action is correctly dispatched 2 seconds after our call the async action creator!
```

```
// Just for your curiosity, here is how a middleware to log all actions that are dispatched, would
// look like:
```

```js
function logMiddleware ({ dispatch, getState }) {
    return function(next) {
        return function (action) {
            console.log('logMiddleware action received:', action)
            return next(action)
        }
    }
}
```

```
// Same below for a middleware to discard all actions that are dispatched (not very useful as is
// but with a bit of more logic it could selectively discard a few actions while passing others
// to next middleware or Redux):
```
```js
function discardMiddleware ({ dispatch, getState }) {
    return function(next) {
        return function (action) {
            console.log('discardMiddleware action received:', action)
        }
    }
}
```

```
// Try to modify finalCreateStore call above by using the logMiddleware and / or the discardMiddleware
// and see what happens...
// For example, using:
//     const finalCreateStore = applyMiddleware(discardMiddleware, thunkMiddleware)(createStore)
// should make your actions never reach your thunkMiddleware and even less your reducers.
```

```
// See http://redux.js.org/docs/introduction/Ecosystem.html#middleware, section Middleware, to
// see other middleware examples.
```

```
// Let's sum up what we've learned so far:
// 1) We know how to write actions and action creators
// 2) We know how to dispatch our actions
// 3) We know how to handle custom actions like asynchronous actions thanks to middlewares
```

```
// The only missing piece to close the loop of Flux application is to be notified about
// state updates in order to react to them (by re-rendering our components for example).
```

```
// So how do we subscribe to our Redux store updates?
```
