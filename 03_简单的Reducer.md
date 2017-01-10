```
// Tutorial 03 - simple-reducer.js
```

## 03 - 简单的Reducer

```
// Now that we know how to create a Redux instance that will hold the state of our application
// we will focus on those reducer functions that will allow us to transform this state.
```
现在我们知道了如何创建一个Redux实例来持有我们程序的State. 接着我们将聚焦于那些可以让我们改变程序State的Reducer函数上.

```
// A word about reducer VS store:
// As you may have noticed, in the flux diagram shown in the introduction, we had "Store", not
// "Reducer" like Redux is expecting. So how exactly do Store and Reducer differ?
// It's more simple than you could imagine: A Store keeps your data in it while a Reducer doesn't.
// So in traditional flux, stores hold state in them while in Redux, each time a reducer is
// called, it is passed the state that needs to be updated. This way, Redux's stores became
// "stateless stores" and were renamed reducers.
```
说一说Reducer与Store吧:

你或许已注意到, 在[简介一节](0_简介.md)所展现的那张Flux图表中, 我们有一个"Store", 而不是Redux所期望的"Reducer". 那么到底Store与Reducer有何不同呢?

这要比你想象的更简单: 一个Store会保存你的数据而Reducer则不会. 因此在传统的Flux中Stores持有State, 而在Redux中每次Reducer被调用时, 它(Reducer)就被传入一个需要被更新的State. 以此类推, Redux的Stores就变成了一个"无状态的Stores"并改名为Reducers.

```
// As stated before, when creating a Redux instance you must give it a reducer function...
```
正如之前所提到的, 当创建一个Redux实例的时候你必须要给一个Reducer函数...

```js
import { createStore } from 'redux'

var store_0 = createStore(() => {})
```

```
// ... so that Redux can call this function on your application state each time an action occurs.
// Giving reducer(s) to createStore is exactly how Redux registers the action "handlers" (read reducers) we
// were talking about in section 01_simple-action-creator.js.
```
... 这样Redux就可以在每次Action发生时基于你程序的State调用Reducer函数.

传入一个Reducer函数给**createStore**正是我们在第一节中提到的Redux是如何注册Action的"处理函数"

```
// Let's put some log in our reducer
```
让我们加一些日志在Reducer函数中看看

```js
var reducer = function (...args) {
    console.log('Reducer was called with args', args)
}

var store_1 = createStore(reducer)
```

```
// Output: Reducer was called with args [ undefined, { type: '@@redux/INIT' } ]
```
程序运行后输出: 
```
    Reducer was called with args [ undefined, { type: '@@redux/INIT' } ]
    使用参数: **[undefined, { type: '@@redux/INIT' } ]**调用Reducer
```

```
// Did you see that? Our reducer is actually called even if we didn't dispatch any action...
// That's because to initialize the state of the application,
// Redux actually dispatches an init action ({ type: '@@redux/INIT' })
```
你看到这样的输出内容了吗？即使我们并没有分发任何的Action, 我们的Reducer实际上也会被调用... 这是由于程序State的初始化所引起的, Redux实际上会分发一个初始化的Action **({type: '@@redux/INIT' })**

```
// When called, a reducer is given those parameters: (state, action)
// It's then very logical that at an application initialization, the state, not being
// initialized yet, is "undefined"
```
当Reducer被调用的时候，它将被传入这些参数: **(state, action)**

总之在程序初始化阶段这是非常符合逻辑的, 而还没有被初始化的程序State就是**"undefined"**

```
// But then what is the state of our application after Redux sends its "init" action?
```
但在Redux发送了**"init"** Action后我们程序的State又会怎样呢？

