```
// Tutorial 05 - combine-reducers.js
```
## 05 - 组合reducers

```
// We're now starting to get a grasp of what a reducer is...
```
我们现在开始去了解一下reducer包含什么...

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
...但在进一步的了解之前，我们应该会很好奇当我们拥有数十个actions的时候，我们的reducer会是什么样子？

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
十分明显的就是单个reducer函数是无法处理我们程序所有的actions的(好吧，它能够处理所有的actions，但这将变得难以维护...).

```
// Luckily for us, Redux doesn't care if we have one reducer or a dozen and it will even help us to
// combine them if we have many!
```
幸运的是, Redux并不关心我们是有一个reducer还是有多个reducer,
如果我们有多个的话，Redux其实会帮我们将他们组合在一起!

```
// Let's declare 2 reducers
```
让我们声明2个reducers试试

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
我希望你特别的注意下实际传入给每个reducer的状态初始化值:
userReducer获得是一个以字面量对象形式表示的状态初始化值,
而itemsReducer得到的是一个以array形式表示的状态初始化值.
这些只不过是为了表明一个reducer事实上是能够处理不同的数据结构类型.
什么样的数据结果更适合于你的需求这都是由你自己来决定的.

```
// With this new multiple reducer approach, we will end up having each reducer handle only
// a slice of our application state.
```
随着这些新的reducer方法的引入,
我们最终会让每个reducer仅仅处理我们程序状态的一小块.

```
// But as we already know, createStore expects just one reducer function.
```
但正如我们已获悉的, **createStore**仅仅期望传入一个reducer函数

```
// So how do we combine our reducers? And how do we tell Redux that each reducer will only handle
// a slice of our state?
// It's fairly simple. We use Redux combineReducers helper function. combineReducers takes a hash and
// returns a function that, when invoked, will call all our reducers, retrieve the new slice of state and
// reunite them in a state object (a simple hash {}) that Redux is holding.
// Long story short, here is how you create a Redux instance with multiple reducers:
```
那么如何将我们的reducers组合起来呢？我们又该怎样的告诉Redux我们每个reducer只会处理程序状态的一小块呢?
其实很简单. 我们使用Redux的combineReducers辅助函数来帮我们达到目的.
combineReducers生成一个hash对象并返回一个函数，该函数被调用的时候实际上会调用我们提供的reducers函数去获取状态被更新的那一部分,
然后将多个新的状态小块组合起来并放在Redux持有的状态对象中(一个简单的
hash{}对象).
长话短说吧, 下面这段代码就是告诉我们如何用多个reducers来创建一个Redux实例:

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
运行这段代码会输出:
使用参数state: {}和action { type: '@@redux/INIT' }调用 userReducer
使用参数state: {}和action { type:
'@@redux/PROBE_UNKNOWN_ACTION_9.r.k.r.i.c.n.m.i' }调用 userReducer
使用参数state: []和action { type: '@@redux/INIT' }调用 itemsReducer
使用参数state: []和action { type:
'@@redux/PROBE_UNKNOWN_ACTION_4.f.i.z.l.3.7.s.y.v.i' }调用itemsReducer

```js
var store_0 = createStore(reducer)
```
```
// Output:
// userReducer was called with state {} and action { type: '@@redux/INIT' }
// itemsReducer was called with state [] and action { type: '@@redux/INIT' }
```
该代码会输出:
使用参数state: {}和action { type: '@@redux/INIT' }调用userReducer
使用参数state: []和action { type: '@@redux/INIT' }调用itemsReducer

```
// As you can see in the output, each reducer is correctly called with the init action @@redux/INIT.
// But what is this other action? This is a sanity check implemented in combineReducers
// to assure that a reducer will always return a state != 'undefined'.
// Please note also that the first invocation of init actions in combineReducers share the same purpose
// as random actions (to do a sanity check).
```
正如你在运行后输出的结果中看到的一样, 每个reducer都因初始化action
@@redux/INIT而被正确地调用.
但其他的action又是什么呢？他们是在combineReducers函数中实现的健全性检查用于确保reducer总能够返回一个undefined的state.
同样也请注意在combineReducers中初始化actions的第一次调用与随机actions具有西安共同的目的.

```js
console.log('store_0 state after initialization:', store_0.getState())
```
```
// Output:
// store_0 state after initialization: { user: {}, items: [] }
```
该代码会输出: 在初始化后store_0的状态: { user: {}, items: [] }

```
// It's interesting to note that Redux handles our slices of state correctly,
// the final state is indeed a simple hash made of the userReducer's slice and the itemsReducer's slice:
// {
//     user: {}, // {} is the slice returned by our userReducer
//     items: [] // [] is the slice returned by our itemsReducer
// }
```
值得一提的是Redux正确地处理我们程序状态的每一块,
最终的状态实际上是一个由userReducer部分和itemsReducer部分组成的一个简单的hash对象:
```js
    {
        user: {}, // {} 是由userReducer返回的部分
        items: [] // [] 是由itemsReducer返回的部分
    }
```

```
// Since we initialized the state of each of our reducers with a specific value ({} for userReducer and
// [] for itemsReducer) it's no coincidence that those values are found in the final Redux state.
```
由于在每个reducer中我们都用特殊的值去初始化状态,
那么这些值出现在最终的Redux状态中也就不足为怪了.

```
// By now we have a good idea of how reducers will work. It would be nice to have some
// actions being dispatched and see the impact on our Redux state.
```
到目前为止,我们有了一个很好的想法让我们多个recuder运作起来.
这也有利于拥有多个被分发的actions并观察其对Redux状态的影响.

