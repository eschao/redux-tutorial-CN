```
// Tutorial 03 - simple-reducer.js
```

## 03 - 简单的Reducer

```
// Now that we know how to create a Redux instance that will hold the state of our application
// we will focus on those reducer functions that will allow us to transform this state.
```
现在我们知道如何创建一个Redux实例来持有我们程序的状态.
接着我们将聚焦在哪些可以让我们改变程序状态的reducer函数上.

```
// A word about reducer VS store:
// As you may have noticed, in the flux diagram shown in the introduction, we had "Store", not
// "Reducer" like Redux is expecting. So how exactly do Store and Reducer differ?
// It's more simple than you could imagine: A Store keeps your data in it while a Reducer doesn't.
// So in traditional flux, stores hold state in them while in Redux, each time a reducer is
// called, it is passed the state that needs to be updated. This way, Redux's stores became
// "stateless stores" and were renamed reducers.
```
说说reducer与store:

你或许已注意到, 在简介一节所展现的那张flux图表中, 我们有一个"Store", 而不是像
Redux所期望的"Reducer". 那么到底Store与Reducer有何不同呢?

这要比你想象的更加简单: Store会保存你的数据而Reducer则不会.  因此在传统的flux中,
stores持有状态而在Redux中, 每次reducer被调用, 它(reducer)就被传入一个需要被更新
的状态. 按照这种, Redux的stores就变成了一个"无状态的stores"并改名为reducers.

```
// As stated before, when creating a Redux instance you must give it a reducer function...
```
正如之前提到的, 当创建一个Redux实例的时候你必须给一个reducer函数...

```js
import { createStore } from 'redux'

var store_0 = createStore(() => {})
```

```
// ... so that Redux can call this function on your application state each time an action occurs.
// Giving reducer(s) to createStore is exactly how Redux registers the action "handlers" (read reducers) we
// were talking about in section 01_simple-action-creator.js.
```
... 这样Redux就可以在每次action发生是基于你程序的状态调用该函数.
传递给createStore相应的reducer函数正是我们在第一节中提到的Redux如何注册action的
"处理函数"

```
// Let's put some log in our reducer
```
让我们加一些日志在我们的reducer函数中看看

```js
var reducer = function (...args) {
    console.log('Reducer was called with args', args)
}

var store_1 = createStore(reducer)
```

```
// Output: Reducer was called with args [ undefined, { type: '@@redux/INIT' } ]
```
输出: Reducer会带着参数[undefined, { type: '@@redux/INIT' } ]被调用

```
// Did you see that? Our reducer is actually called even if we didn't dispatch any action...
// That's because to initialize the state of the application,
// Redux actually dispatches an init action ({ type: '@@redux/INIT' })
```
你看到这样的输出内容了吗？即使我们并没有分发任何的action, 我们的reducer实际上也
会被调用... 这是因为程序状态的初始化引起的, Redux事实上会分发一个初始化的action
({type: '@@redux/INIT' })

```
// When called, a reducer is given those parameters: (state, action)
// It's then very logical that at an application initialization, the state, not being
// initialized yet, is "undefined"
```
当reducer被调用的时候，它被传入这些参数: (state, action)
那么在程序初始化阶段这是非常符合逻辑的, 还没有被初始化的程序状态是"undefined"

```
// But then what is the state of our application after Redux sends its "init" action?
```
但是在Redux发送了"init" action后我们程序的状态又是什么呢？

