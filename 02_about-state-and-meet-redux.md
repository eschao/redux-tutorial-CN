```
// Tutorial 02 - about-state-and-meet-redux.js
```

## 关于State

```
// Sometimes the actions that we'll handle in our application will not only inform us
// that something happened but also tell us that data needs to be updated.
```

在我们的程序中，有时候我们将要处理的actions不仅会通知我们什么事会发生同时也会告诉
我们哪些数据要被更新

```
// This is actually quite a big challenge in any app.
// Where do I keep all the data regarding my application along its lifetime?
// How do I handle modification of such data?
// How do I propagate modifications to all parts of my application?
```
事实上在任何的程序中，这都是一个相当大的挑战.
我该在什么地方保持我程序中所有数据的生命期呢?
我该怎样处理这种数据的修改呢？
我该如何将修改广播到我程序的其它部分呢？

```
// Here comes Redux.
```
Redux出现了.

```
// Redux (https://github.com/rackt/redux) is a "predictable state container for JavaScript apps"
```
Redux是一个"为JavaScript程序提供一个可预知的状态容器"

```
// Let's review the above questions and reply to them with
// Redux vocabulary (flux vocabulary too for some of them):
```
我们先回顾一下上述问题并用Redux术语来回答

```
// Where do I keep all the data regarding my application along its lifetime?
//     You keep it the way you want (JS object, array, Immutable structure, ...).
//     Data of your application will be called state. This makes sense since we're talking about
//     all the application's data that will evolve over time, it's really the application's state.
//     But you hand it over to Redux (Redux is a "state container", remember?).
// How do I handle data modifications?
//     Using reducers (called "stores" in traditional flux).
//     A reducer is a subscriber to actions.
//     A reducer is just a function that receives the current state of your application, the action,
//     and returns a new state modified (or reduced as they call it)
// How do I propagate modifications to all parts of my application?
//     Using subscribers to state's modifications.
```
我该在什么地方保持我程序中所有数据的生命期呢?
- 你可以以任何的方式保存它(JS对象, 数组, 不可变结构体, ...)
- 你程序中的数据被称为State. 这合乎情理, 因为我们所涉及的程序中的所有数据都会
  随着事件而改变, 这其实就是程序的状态State.
- 不过你将他们交给了Redux(Redux是一个"状态容器", 还记得吗？)

我该怎样处理这种数据的修改呢？
- 使用reducers(在传统的flux中被称为"stores")
- 一个reducer就是对actions的一个订阅者subscriber
- 一个reducer只不过是一个可以接受你程序当前状态, action,
  并返回一个新的被修改后的状态的函数

我该如何将修改广播到我程序的其它部分呢？
- 使用对于状态修改的订阅者subscribers


```
// Redux ties all this together for you.
// To sum up, Redux will provide you:
//     1) a place to put your application state
//     2) a mechanism to dispatch actions to modifiers of your application state, AKA reducers
//     3) a mechanism to subscribe to state updates
```
Redux为你捆绑了这一切.
总之, Redux将为你提供:
* 一个存放你程序状态的地方
* 一个将action分发到你程序状态修改者的机制, 又名reducers
* 一个对订阅状态更新的机制

```js
// The Redux instance is called a store and can be created like this:
// Redux实例被称为store, 可以如下来创建:

    import { createStore } from 'redux'
    var store = createStore()
```

```
// But if you run the code above, you'll notice that it throws an error:
// 但如果你运行上面的代码, 你将发现它会抛出一个异常:
//     Error: Invariant Violation: Expected the reducer to be a function.
```

```
// That's because createStore expects a function that will allow it to reduce your state.
```

这是因为createStore需要传入一个函数用于reduce你程序的状态

```
// Let's try again
```
让我们再试一试

```js
import { createStore } from 'redux'

var store = createStore(() => {})
```

```
// Seems good for now...
```
现在看起来就对了...

