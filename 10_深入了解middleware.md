## 深入了解 middleware

在上一节中我们简单的介绍了 middleware 并展示了其基本的用法. 但我们可能会疑惑: 
* 为什么 middleware 函数要遵循那样的签名及结构? 
* 为什么 middleware 需要三层函数? 
* Redux 的 applyMiddleware 函数到底又做了什么?

明白 middleware 的前因后果, 也是我们理解 Redux 设计和使用的一个重要的部分. 在本节中, 我会以翻译官方的 [Middleware](http://redux.js.org/docs/advanced/Middleware.html#) 一节为主并辅以我自己的浅薄理解尝试剖析一下 middleware , 希望能帮助你更进一步地理解 middleware.

### 问题 1: 记录日志

Redux 所带来的一个好处就是它让 state 的变更变得可预测且透明. 每次当 action 被分发的时候, 就会计算出新的 state 并保存下来. state 是不能被自身所改变的, 它只能随着特定 action 的结果而改变.
要是能记录下在程序中发生的每一个 action 和其后被计算的 state 是不是很好呢? 当某个环节出现问题的时候, 我们就能够回溯所有的日志, 然后找出是哪个 action 破坏了 state.

我们该如何借助 Redux 来实现呢?

### 首次尝试: 手工记录

最天真的方法就是在每次调用 ``` store.dispatch(action)``` 之后你手工的记录下 action 以及下一个 state. 这并非真正的解决方案, 而只是理解问题的第一步.

>##### 注意
如果你正在使用 [react-redux](https://github.com/gaearon/react-redux) 或其他类似的绑定框架, 你可能不会在你的组件里直接访问 store 实例. 因而对于接下来的内容, 假定你显示地将 store 实例传递了下来

比如说, 你会这样的来创建一个 [todo](http://redux.js.org/docs/basics/ExampleTodoList.html) (官方文档中的待办事项例子):
```js
// 发送一个'添加待办事项'的 action
store.dispatch(addTodo('Use Redux'))
```
为了记录下 action 和 state , 你可能会改成这样:
```js
let action = addTodo('Use Redux')

// 分发前记录 action 对象
console.log('dispatching', action)
store.dispatch(action)
// 分发后记录 store 实例
console.log('next state', store.getState())
```
这些代码能达到期望的效果, 但你不会每次都想这么写.

### 第2次尝试: 包装 Dispatch

你可以将日志记录部分提取出来放在一个函数里:
```js
function dispatchAndLog(store, action) {
    console.log('dispatching', action)
    store.dispatch(action)
    console.log('next state', store.getState())
}
```
然后你就可以在任何的地方用 ```dispatchAndLog``` 来代替 ```store.dispatch()``` 了:
```js
dispatchAndLog(store, addTodo('Use Redux'))
```
我们可以就此结束, 但每次都要导入一个特殊的函数是十分不方便的.

### 第3次尝试: 对 Dispatch 打 "猴子补丁"

如果我们只是替换掉 store 实例的 ```dispatch``` 函数会怎样呢? Redux 的 store 只不过是拥有一些方法的简单对象, 而因为我们使用的是 JavaScript , 因此我们能够 monkeypatch 掉 ```dispatch``` 的实现:
```js
// 先将原始的 dispatch 保存下来, 注意 'next' 变量名, 在后续改进中有一定的含义
let next = store.dispatch

// 将 dispatch 替换成我们写的可以记录日志的: dispatchAndLog
store.dispatch = function dispatchAndLog(action) {
    console.log('dispatching', action)
    // 在这里我们会调用原始的 dispatch 来真正的分发 action
    let result = next(action)
    console.log('next state', store.getState())
    return result
}
```
这已经很接近我们想要的了! 无论我们在何处分发一个 action , 都能够保证日志被记录下来. Monkeypatching 从来都不讨人喜欢, 但目前我们用着还好.

### 问题 2: Crash报告

如果我们要对 ```dispatch``` 添加不止一个这样的转换函数又该怎么办呢?
我想到的另一个有用的转换功能就是在产品中报告 JavaScript 的错误. 全局的 ```window.onerror``` 事件并不可靠,这是因为它在一些老的浏览器中无法提供堆栈信息, 而这些信息对于我们明白错误为什么产生是至关重要的.

如果对于任何时候因分发一个 action 而抛出的一个错误, 我们能像 [Sentry](https://getsentry.com/welcome/) 一样将堆栈信息, 导致错误的 action 以及当前的 state 一起发送给 crash 报告服务, 不就有用了吗? 这样的话, 我们就可以在开发中重现该错误了.

然而, 保持分离日志和 crash 报告功能是很重要的. 理想情况下, 我们想让它们成为不同的模块, 并可能位于不同的包中. 否则我们就无法拥有如此的工具生态系统. (提示: 我们正在慢慢地接近什么是 middleware!)

如果日志和 crash 报告是两个独立的工具函数, 它们看起来可能像这样:
```js
// 记录日志, 如前一样使用 '替换 dispatch' 的方式
function patchStoreToAddLogging(store) {
    let next = store.dispatch
    store.dispatch = function dispatchAndLog(action) {
        console.log('dispatching', action)
        let result = next(action)
        console.log('next state', store.getState())
        return result
    }
}

// 上报 crash, 也是用 '替换 dispatch' 的方式
function patchStoreToAddCrashReporting(store) {
    let next = store.dispatch
    store.dispatch = function dispatchAndReportErrors(action) {
        try {
            return next(action)
        } catch (err) {
            console.error('Caught an exception!', err)
            // 使用 Raven 来记录 exception
            Raven.captureException(err, {
                extra: {
                    action,
                    state: store.getState()
                }
            })
            throw err
        }
    }
}
```
如果这些函数以独立模块的方式发布, 我们以后就可以用他们来修补 store :
```js
patchStoreToAddLogging(store)
patchStoreToAddCrashReporting(store)
```
然而, 这还是不好.

### 第4次尝试: 隐藏 Monkeypathcing

Monkeypathcing有点多余. "替换任何你喜欢的方法", 该是一种什么 API 呢? 相反让我们看看其本质. 之前, 我们的函数替换掉了 ```store.dispatch```. 而如果他们返回的是一个新的 ```dispatch``` 函数呢?
>##### 注意

```js
function logger(store) {
    let next = store.dispatch

    // 之前是替换掉store的dispatch
    // store.dispatch = function dispatchAndLog(action) {

    // 现在我们返回一个新的dispatch函数
    return function dispatchAndLog(action) {
        console.log('dispatching', action)
        let result = next(action)
        console.log('next state', store.getState())
        return result
    }
}
```
我们可以在 Redux 内部提供一个帮助函数, 该函数会将我们之前所写的 monkeypatching 作为一个实现细节:
```js
// 慢慢开始接近 applyMiddleware 函数了
function applyMiddlewareByMonkeypatching(store, middlewares) {
    middlewares = middlewares.slice()
    middlewares.reverse()

    // 依次用每个 middleware 来转换 dispatch 函数.
    middlewares.forEach(middleware =>
        store.dispatch = middleware(store)
    )
}
```

然后我们可以如下这样将其应用到多个 middleware 上:
```js
applyMiddlewareByMonkeypatching(store, [ logger, crashReporter ])
```
然而, 这还是一个 monkeypatching. 我们将它隐藏在 Redux 库里的事实并没有改变其本质.

### 第5次尝试: 移除 Monkeypathcing

为什么我们还要覆盖掉 ```dispatch``` 函数呢? 当然这是为了以后能调用它. 但这儿还有一个另外的原因: 是为了让每个 middleware 都能访问被包装的 ```store.dispatch``` :
```js
function logger(store) {
  // Must point to the function returned by the previous middleware:
  let next = store.dispatch

  return function dispatchAndLog(action) {
    console.log('dispatching', action)
    let result = next(action)
    console.log('next state', store.getState())
    return result
  }
}
```
其本质就是将 middleware 串连起来!
如果 ```applyMiddlewareByMonkeypatching``` 没有在处理完第一个 middleware 后立即给 ```store.dispatch``` 赋值, ```store.dispatch``` 就会保持指向原始的 ```dispatch``` 函数. 那么第二个 middleware 就能绑定到原始的 ```dispatch``` 函数上.

但这儿还有另外一种方式去串连. middleware 可以接受一个名为 ```next()``` 的 dispatch 函数参数而不是从 store 的实例读取 dispatch 函数:
```js
function logger(store) {
    return function wrapDispatchToAddLogging(next) {
        return function dispatchAndLog(action) {
            console.log('dispatching', action)
            let result = next(action)
            console.log('next state', store.getState())
            return result
        }
    }
}
```
这是我们需要更进一步的时刻了. 因此可能需要花点时间去理解. 这种瀑布式的函数会令人感到一丝恐惧. ES6的箭头函数使得这样的 currying(柯里化) 看起来更容易点:
```js
const logger = store => next => action => {
    console.log('dispatching', action)
    let result = next(action)
    console.log('next state', store.getState())
    return result
}

const crashReporter = store => next => action => {
    try {
        return next(action)
    } catch (err) {
        console.error('Caught an exception!', err)
        Raven.captureException(err, {
            extra: {
                action,
                state: store.getState()
            }
        })
        throw err
    }
}
```
这正是 Redux middleware 的样子
现在 middleware 接受 ```next()``` dispatch 函数参数, 并返回一个新的 dispatch 函数, 然后从左开始依次作为 ```next()``` 传给下一个 middleware 等等. 访问类似 ```getState()``` 这样的其他 store 方法仍然有用. 所以 store 被保留下来作为一个顶层的参数.

### 第6次尝试: 天真地应用 Middleware

与 ```applyMiddlewareByMonkeypatching()``` 不同的是, 我们会写一个新的 ```applyMiddleware()``` 用于首次获得一个最终的完全包装好的 ```dispatch``` 函数, 并返回一个使用它的 store 拷贝:
```js
// Warning: Naïve implementation!
// That's *not* Redux API.

function applyMiddleware(store, middlewares) {
    middlewares = middlewares.slice()
    middlewares.reverse()

    let dispatch = store.dispatch
    middlewares.forEach(middleware =>
        dispatch = middleware(store)(dispatch)
    )

    return Object.assign({}, store, { dispatch })
}
```
这基本与 Redux 内部的 ```applyMiddleware()``` 实现相同. 但在以下三个重要方面有所不同:
* 对于 middleware, 它只暴露了 store API 中的一部分: ```dispatch(action)``` 和 ```getState()```.
* 它使用了一点点手段用于确保当你从你的 middleware 而不是 ```next(action)``` 去调用 ```store.dispatch(action)``` 时, action 实际会再次穿越整个 middleware 链, 也包含当前的 middleware . 这对于异步方式的 middleware 很有用.
* 为了保证你只应用了 middleware 一次. 它会作用在 ```createStore()``` 而非 store 自身上. 因而它的签名是 ```(...middlewares) => (createStore) => createStore``` 而不是 ```(store, middlewares) => store```.

由于在使用之前将函数应用到 ```createStore()``` 有些繁琐, 所以 ```createStore()``` 可接受一个可选的末尾参数用于传入这些函数.

### 最后的方法
鉴于我们刚刚所写的 middlewares :
```js
const logger = store => next => action => {
    console.log('dispatching', action)
    let result = next(action)
    console.log('next state', store.getState())
    return result
}

const crashReporter = store => next => action => {
    try {
        return next(action)
    } catch (err) {
        console.error('Caught an exception!', err)
        Raven.captureException(err, {
            extra: {
                action,
                state: store.getState()
            }
        })
        throw err
    }
}
```
下面是如何将它应用到 Redux ```store``` 的:
```js
import { createStore, combineReducers, applyMiddleware } from 'redux'

let todoApp = combineReducers(reducers)
let store = createStore(
    todoApp,
    // applyMiddleware() tells createStore() how to handle middleware
    applyMiddleware(logger, crashReporter)
)
```
就是如此, 现在任何对于该 ```store``` 实例分发的 actions 都将经过 ```logger``` 和 ```crashReporter``` 两个 middlewares .
