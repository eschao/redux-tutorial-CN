```
// Tutorial 05 - combine-reducers.js
```
## 05 - 组合多个reducers

```
// We're now starting to get a grasp of what a reducer is...
```
现在我们去了解一下 reducer 函数包含了什么...

```js
var reducer_0 = function (state = {}, action) {
    console.log('reducer_0 was called with state', state, 'and action', action)

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
```

```
// ... but before going further, we should start wondering what our reducer will look like when
// we'll have tens of actions:
```
...但在进一步深入之前，我们可能想知道, 当我们拥有数十个 actions 的时候，我们的 reducer 会变成什么样？

```js
var reducer_1 = function (state = {}, action) {
    console.log('reducer_1 was called with state', state, 'and action', action)

    switch (action.type) {
        case 'SAY_SOMETHING':
            return {
                ...state,
                message: action.value
            }
        case 'DO_SOMETHING':
            // ...
        case 'LEARN_SOMETHING':
            // ...
        case 'HEAR_SOMETHING':
            // ...
        case 'GO_SOMEWHERE':
            // ...
        // etc.
        default:
            return state;
    }
}
```

```
// It becomes quite evident that a single reducer function cannot hold all our
// application's actions handling (well it could hold it, but it wouldn't be very maintainable...).
```
非常明显单个 reducer 函数是无法包含程序中处理所有 actions 的代码 (好吧，或许它能，但这样的代码将非常难以维护...).

```
// Luckily for us, Redux doesn't care if we have one reducer or a dozen and it will even help us to
// combine them if we have many!
```
幸运的是, Redux 并不关心我们是有一个 reducer 还是有多个 reducer. 如果我们有多个 reducer 函数的话，Redux甚至能够帮助我们将他们合并在一起!

```
// Let's declare 2 reducers
```
让我们声明2个 reducers 试试:

```js
var userReducer = function (state = {}, action) {
    console.log('userReducer was called with state', state, 'and action', action)

    switch (action.type) {
        // etc.
        default:
            return state;
    }
}
var itemsReducer = function (state = [], action) {
    console.log('itemsReducer was called with state', state, 'and action', action)

    switch (action.type) {
        // etc.
        default:
            return state;
    }
}
```

```
// I'd like you to pay special attention to the initial state that was actually given to
// each reducer: userReducer got an initial state in the form of a literal object ({}) while
// itemsReducer got an initial state in the form of an array ([]). This is just to
// make clear that a reducer can actually handle any type of data structure. It's really
// up to you to decide which data structure suits your needs (an object literal, an array,
// a boolean, a string, an immutable structure, ...).
```
我希望你特别地注意下实际传给每个 reducer 的 state 初始化值:
* **userReducer** 获得一个以字面量对象形式表示的 state 初始化值.
* **itemsReducer** 获得到一个以 array 形式表示的 state 初始化值.

这些不同的初始化值只不过是为了说明一个 reducer 实际能够处理不同的数据结构类型.
而你真正地决定了什么样的数据结构适合于你的需求(字面量对象, 数组, 布尔, 字符串或者不可变的数据结构, ....).

```
// With this new multiple reducer approach, we will end up having each reducer handle only
// a slice of our application state.
```
随着这些新的 reducer 方法的引入, 最终我们所拥有的每个 reducer 只会处理程序 state 的一部分.

```
// But as we already know, createStore expects just one reducer function.
```
但正如我们所知, **createStore** 值允许传入一个 reducer 函数.

```
// So how do we combine our reducers? And how do we tell Redux that each reducer will only handle
// a slice of our state?
// It's fairly simple. We use Redux combineReducers helper function. combineReducers takes a hash and
// returns a function that, when invoked, will call all our reducers, retrieve the new slice of state and
// reunite them in a state object (a simple hash {}) that Redux is holding.
// Long story short, here is how you create a Redux instance with multiple reducers:
```
那么如何将我们的 reducers 合并起来呢？我们又该怎样让 Redux 明白我们的每个 reducer 只会处理程序 state 的一部分呢?

其实很简单. 我们使用 Redux 的 **combineReducers** 辅助函数来达到目的. **combineReducers** 生成一个 hash 对象并返回一个函数，该函数被调用的时候实际上会调用我们提供的 reducers 函数去获取 state 更新的那一部分, 然后将多个新的 state 小块组合起来并存放在 Redux 持有的 state 对象中(一个简单的 hash {} 对象).

长话短说吧, 下面这段代码展示了如何用多个 reducers 来创建一个 Redux 实例:

```js
import { createStore, combineReducers } from 'redux'

var reducer = combineReducers({
    user: userReducer,
    items: itemsReducer
})
```
```
// Output:
// userReducer was called with state {} and action { type: '@@redux/INIT' }
// userReducer was called with state {} and action { type: '@@redux/PROBE_UNKNOWN_ACTION_9.r.k.r.i.c.n.m.i' }
// itemsReducer was called with state [] and action { type: '@@redux/INIT' }
// itemsReducer was called with state [] and action { type: '@@redux/PROBE_UNKNOWN_ACTION_4.f.i.z.l.3.7.s.y.v.i' }
```
运行后输出:
```js
    userReducer was called with state {} and action { type: '@@redux/INIT' }
    userReducer was called with state {} and action { type: '@@redux/PROBE_UNKNOWN_ACTION_9.r.k.r.i.c.n.m.i' }
    itemsReducer was called with state [] and action { type: '@@redux/INIT' }
    itemsReducer was called with state [] and action { type: '@@redux/PROBE_UNKNOWN_ACTION_4.f.i.z.l.3.7.s.y.v.i' }
```

```js
var store_0 = createStore(reducer)
```
```
// Output:
// userReducer was called with state {} and action { type: '@@redux/INIT' }
// itemsReducer was called with state [] and action { type: '@@redux/INIT' }
```
运行后输出:
```js
    userReducer was called with state {} and action { type: '@@redux/INIT' }
    itemsReducer was called with state [] and action { type: '@@redux/INIT' }
```

```
// As you can see in the output, each reducer is correctly called with the init action @@redux/INIT.
// But what is this other action? This is a sanity check implemented in combineReducers
// to assure that a reducer will always return a state != 'undefined'.
// Please note also that the first invocation of init actions in combineReducers share the same purpose
// as random actions (to do a sanity check).
```
正如你在输出结果中看到的一样, 每个 reducer 都因初始化 action @@redux/INIT 而被正确地调用.

但其他那些非初始化 action 又是什么呢？他们是在 **combineReducers** 函数中实现的健全性检查, 确保 reducer 函数总能够返回一个非 **undefined** 的  state.

同样也请注意在 **combineReducers** 中第一次调用的初始化 actions 与那些随机 actions 具有相同的目的.

```js
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
// It's interesting to note that Redux handles our slices of state correctly,
// the final state is indeed a simple hash made of the userReducer's slice and the itemsReducer's slice:
// {
//     user: {}, // {} is the slice returned by our userReducer
//     items: [] // [] is the slice returned by our itemsReducer
// }
```
值得注意的是 Redux 正确地处理了我们程序 state 的每一部分, 最终的 state 实际上是一个由 userReducer 的 state 部分和 itemsReducer 的 state 部分组成的一个简单 hash 对象:
```js
    {
        user: {}, // {} 是由 userReducer 返回的部分
        items: [] // [] 是由 itemsReducer 返回的部分
    }
```

```
// Since we initialized the state of each of our reducers with a specific value ({} for userReducer and
// [] for itemsReducer) it's no coincidence that those values are found in the final Redux state.
```
由于在每个 reducer 中我们都用特殊的值去初始化 state ( {} 用于初始化 userReducer, [] 用于初始化 itemsReduce), 那么这些值出现在最终的 Redux state 中就不足为怪了.

```
// By now we have a good idea of how reducers will work. It would be nice to have some
// actions being dispatched and see the impact on our Redux state.
```
到目前为止, 我们清楚地明白了多个 recuder 是如何运转的. 拥有多个被分发的 actions 并看到他们对 Redux state的影响也很不错.
