## 深入了解 middleware

在上一节中我们简单的介绍了 middleware 并展示了其基本的用法. 但我们可能会疑惑: 
* 为什么 middleware 函数要遵循那样的签名及结构? 
* 为什么 middleware 需要三层函数? 
* Redux 的 ```applyMiddleware``` 函数到底又做了什么?

明白 middleware 的前因后果, 也是我们使用和理解 Redux 的一个重要的部分. 在本节中, 我会以翻译官方的 [Middleware](http://redux.js.org/docs/advanced/Middleware.html#) 一节为主并辅以我自己的浅薄理解去尝试剖析一下 middleware , 希望能帮助你更进一步地理解 middleware.

### 问题 1: 记录日志

Redux 所带来的一个好处就是它让 state 的更改变得可预测且透明. 每次当 action 被分发的时候, 就会计算出新的 state 并保存下来. state 是不能被自身所改变的, 它只能随着特定 action 的结果而改变.
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

你可以将日志记录部分提取出来并放在一个函数里:
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

### 第3次尝试: 给 Dispatch 打 "猴子补丁"

如果我们只是替换掉 store 实例的 ```dispatch``` 函数会怎样呢? Redux 的 store 只不过是拥有一些方法的简单对象, 而由于我们使用的是 JavaScript , 所以我们能够 monkeypatch 掉 ```dispatch``` 的实现:
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
这已经很接近我们想要的了! 无论我们在何处分发一个 action , 都能够保证日志被记录下来. 虽然 Monkeypatching 从来都不讨人喜欢, 但目前我们用着还好.

### 问题 2: Crash报告

如果我们要对 ```dispatch``` 添加不止一个这样的转换函数又该怎么办呢?
我想到的另一个有用的转换功能就是在产品中上报 JavaScript 的错误. 然而全局的 ```window.onerror``` 事件并不可靠, 这是因为它在一些老的浏览器中无法提供堆栈信息, 而这些信息对于我们明白错误为什么产生是至关重要的.

如果对于任何时候因分发一个 action 而抛出的一个错误, 我们能像 [Sentry](https://getsentry.com/welcome/) 一样将堆栈信息, 导致错误的 action 以及当前的 state 一起发送给 crash 报告服务, 不就有用了吗? 这样的话, 我们就可以在开发中重现该错误了.

不过, 保持分离日志和 crash 报告功能是很重要的. 理想情况下, 我们想让它们成为不同的模块, 并可能位于不同的包中. 否则我们就无法拥有如此的工具生态系统. (提示: 我们正在慢慢地接近什么是 middleware!)

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
此处开始逐步的将 '替换 dispatch' 方式转换为 '三层内嵌函数', 并引出 'applyMiddleware' 函数及其实现

```js
function logger(store) {
    let next = store.dispatch

    // 注释掉之前 '替换 dispatch' 的方式, 改成返回一个新的 dispatch 函数
    // store.dispatch = function dispatchAndLog(action) {

    // 现在我们返回一个新的dispatch函数
    return function dispatchAndLog(action) {
        console.log('dispatching', action)
        
        // next 即是原有的 store.dispatch 
        let result = next(action)
        console.log('next state', store.getState())
        return result
    }
}

function crashReporter(store) {
    let next = store.dispatch
    
    // 注释掉之前 '替换 dispatch' 的方式, 改成返回一个新的 dispatch 函数
    // store.dispatch = function dispatchAndReportErrors(action) {
    
    return function dispatchAndReportErrors(action) {
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
我们可以在 Redux 内部提供一个帮助函数, 该函数会将我们之前所写的 monkeypatching 作为一个实现细节:
```js
// 慢慢开始接近 applyMiddleware 函数了
function applyMiddlewareByMonkeypatching(store, middlewares) {
    // middlewares 是一个数组, 包含你所传入的所有 middleware, 此处相当于复制一份
    middlewares = middlewares.slice();
    
    // 将数组按顺序颠倒, 这应该是为什么要先复制一份的原因, 保证原有的middlewares数组不会改变, 这也是函数编程的思想:
    // 不改变传入的参数值, 至于为什么要颠倒数组顺序, 将用于下面的串连替换 dispatch
    middlewares.reverse();

    // 依次用每个 middleware 来转换 dispatch 函数. 这是applyMiddleware的关键, 也跟 middleware 三层函数相关
    // forEach接受的是一个函数作为参数, 然后对每个数组的元素调用该函数来处理. 这里使用了ES6的新语法: 箭头函数
    // 下面这段代码相当于:
    // middlewares.forEach( function(middleware) {
    //     store.dispatch = middleware(store);
    // });
    // 只不过箭头函数表达方式更简洁 
    middlewares.forEach(middleware =>
        store.dispatch = middleware(store)
    );
}
```

然后我们可以如下这样将其应用到多个 middleware 上:
```js
applyMiddlewareByMonkeypatching(store, [ logger, crashReporter ])

// 我们来看看当传入这些参数后, applyMiddlewareByMonkeypathcing内做了什么
{
    
    middlewares = middlewares.slice(); 
    // 运行后 middlewares 内容还是 [logger, crashReporter], 只不过是复制的一份, 保证不会导致参数被修改
    
    middlewares.reverse();
    // 将 middlewares 数组反转后, 数组就变成了[crashReporter, logger]
    
    middlewares.forEach(middleware =>
        store.dispatch = middleware(store)
    );
    // 遍历数组中每个元素, 首先是 crashReporter, 其次是logger
    // 还记得上面logger, crashReporter函数吗？ 
    // 两个函数中都有先将 store.dispatch 保存在一个 next 变量中, 我们看看它如何在此循环中变化的
    
    // 1. 先作用在 crashReporter 上, 相当于执行的是:
    store.dispatch = crashReporter(store);
    // 运行后等于: store.dispatch 被替换成 dispatchAndReportErrors(actions); 而这时候dispatchAndReportErrors中的 next 指向的是
    // 原始的 store.dispatch
    
    // 2. 作用在 logger 上:
    store.dispatch = logger(store);
    // 运行后等于: store.dispatch 被替换成 dispatchAndLog(actions); 由于crashReporter先执行导致store.dispatch已被替换成
    // dispatchAndReportErrors函数, 所以 dispatchAndLog中的 next 指向的则是 dispatchAndReportErrors
    
    // 那么数组遍历完后 store.dispatch 实际变成了这样(展开):
    store.dispatch = dispatchAndLog(actions) {
        ...
        //let result = next(actions); next 指向 logger 的 dispatchAndReportErrors()
        let result = dispatchAndReportErrors(actions) {
            ...
            //let result = next(actions); next 指向原始的 store.dispatch()
            let result = store.dispatch(actions);
            ...
        }
        ...
     }
     // 如此就形成了一个链式的函数调用, 在函数的最里层是最原始的 store.dispatch 调用, 然后依次是从数组最左边的 middleware 开始调用, 
     // 每一层对应一个 middleware 的 dispatch 函数, 最外层就是数组中第一个 middlerware 的 dispatch 函数, 且该函数也被赋给当前的
     // store.dispatch. 这也是为什么要把数组先做个反转后再执行. 
     // 那么这就是最终的 applyMiddleware 函数的实现吗？ 还不是, 因为还传入的 middleware 还不是一个三层函数, 虽然大体上已经一致了,
     // 但在语法上还可以在精简一下, 这就是接下来为什么要去掉如下这句:
     let next = store.dispatch;
}
```
然而, 这还是一个 monkeypatching. 我们将它隐藏在 Redux 框架里的事实并没有改变其本质.

### 第5次尝试: 移除 Monkeypathcing

为什么我们要覆盖掉 ```dispatch``` 函数呢? 当然是为了之后能调用 middleware 的 dispatch 函数. 但还有另外一个原因: 就是为了让每个 middleware 都能访问被包装的 ```store.dispatch``` :
```js
function logger(store) {
  // 准备要去掉这一行了, 注意 store.dispatch 不一定就是原始的 dispatch 函数, 如上分析, 它可能是上一层 middleware 的 dispatch 函数
  let next = store.dispatch

  return function dispatchAndLog(action) {
    console.log('dispatching', action)
    let result = next(action)
    console.log('next state', store.getState())
    return result
  }
}
```
本质其实就是将 middleware 串连了起来!
如果 ```applyMiddlewareByMonkeypatching``` 没有在处理完第一个 middleware 后立即给 ```store.dispatch``` 赋值, ```store.dispatch``` 就会保持指向原始的 ```dispatch``` 函数. 那么第二个 middleware 就能绑定到原始的 ```dispatch``` 函数上.

但这儿还有另外一种方式去串连. middleware 可以接受一个参数名为 ```next()``` 的 dispatch 函数而不是从 store 的实例读取 dispatch 函数:
```js
function logger(store) {
    // 结合上述对 applyMiddlewareByMonkeypatching 的分析, 在那个循环中, 我们不再返回 dispatchAndLog 而是返回一个 
    // wrapDispatchToAddLogging 函数. 这样之后 next 参数如何传入呢? 且看后续对 applyMiddleware 的变动.
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
这到了我们需要更进一步的时候了. 因此可能需要花点时间去理解这种写法. 这种瀑布式的函数会令人感到一丝恐惧. ES6的箭头函数使得这样的 currying(柯里化) 看起来更容易点:
```js
const logger = store => next => action => {
    console.log('dispatching', action)
    let result = next(action)
    console.log('next state', store.getState())
    return result
}

// 此处写法等同于这样:
const logger = function(store) { 
    return function(next) {
        return function(action) {
            ...
        }
     }
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
这正是 Redux 框架提供的 middleware.

现在 middleware 接受 ```next()``` 函数 (实际是一个 dispatch 函数) 作为参数, 并返回一个新的 dispatch 函数, 然后从左开始依次将其作为 ```next()``` 传给下一个 middleware 等等. 而 store 被保留下来作为一个顶层的参数, 用于访问类似 ```getState()``` 这样的方法.

### 第6次尝试: 天真地应用 Middleware

与 ```applyMiddlewareByMonkeypatching()``` 不同的是, 我们会写一个新的 ```applyMiddleware()``` 函数, 主要用于获得一个最终的完全包装好的 ```dispatch``` 函数, 并返回一个拷贝的 store 用来调用 dispatch 函数 (此处同样遵守函数编程的准则, 由于要改变 store.dispatch , 因而返回的是一个 store 的拷贝保证传入的 store 参数没有被修改):
```js
// Warning: Naïve implementation!
// That's *not* Redux API.

function applyMiddleware(store, middlewares) {
    middlewares = middlewares.slice()
    middlewares.reverse()

    let dispatch = store.dispatch
    middlewares.forEach(middleware =>
        dispatch = middleware(store)(dispatch)
        // 相对于 applyMiddlewareByMonkeypatching, 此处有2点变化
        // 1) middleware(store)之后再跟了一个(dispatch)相当于2次函数
        // 调用, 用logger来展开就变成:
        // logger(store)(dispatch) 
        // 相当于先执行 logger => store 箭头函数, 然后再执行返回的函数
        // 也就是第二个箭头函数 => next , 而 dispatch 就作为 next 参数
        // 被传入, 这样就用第二层函数来去掉了以前的 next = store.dispatch;
        // 2) 返回的dispatch并不如之前那样直接赋值给了 store 而是临时变量
        // 这主要是为了不破坏传入的 store 实例
        
        // 每遍历一个 middleware, 局部变量 dispatch 就被赋予当前 middleware
        // 返回的 dispatch 函数, 而该 dispatch 函数在遍历下一个 middleware 又
        // 被当做参数 next 传入. 这样就将所有的 middleware 串连了起来.
    }

    // 使用JS的Object.assign语法返回一个 store 的拷贝用于实际的 dispatch 调用
    return Object.assign({}, store, { dispatch })
}
```
这与 Redux 自带的 ```applyMiddleware()``` 实现基本相同. 但在以下三个重要方面有所不同:
* 对于 middleware, 它只暴露了 store API 中的一部分: ```dispatch(action)``` 和 ```getState()```.
* 它使用了一点点手段用于确保: 如果你是从你的 middleware 而非 ```next(action)``` 函数上去调用 ```store.dispatch(action)``` , 那么 action 实际就能再次穿越整个 middleware 链, 也包含当前的 middleware . 这对于异步方式的 middleware 很有用.
* 为了保证你只应用了 middleware 一次. 它会作用在 ```createStore()``` 而非 store 自身上. 因而它的签名是 ```(...middlewares) => (createStore) => createStore``` 而不是 ```(store, middlewares) => store```.

由于在使用之前将 middleware 函数应用到 ```createStore()``` 上有些繁琐, 所以 ```createStore()``` 可以接受一个可选的末尾参数用于传入这些函数.

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
下面是如何将它应用到 Redux 的 ```store``` 上:
```js
import { createStore, combineReducers, applyMiddleware } from 'redux'

let todoApp = combineReducers(reducers)
let store = createStore(
    todoApp,
    // applyMiddleware() 告诉 createStore() 如何处理 middleware
    applyMiddleware(logger, crashReporter)
)
```
就是这样! 现在任何对于该 ```store``` 实例分发的 actions 都将经过 ```logger``` 和 ```crashReporter``` 两个 middlewares .

### Thunk Middlware
最后我们再分析一下上一节中提到的 **Thunk Middleware** , 看看它是如何完成一个异步分发的 action

Thunk Middleware 的源码很简单, 只有短短的 10 来行, 如下:
```js
function createThunkMiddleware(extraArgument) {
  return ({ dispatch, getState }) => next => action => {
    if (typeof action === 'function') {
      return action(dispatch, getState, extraArgument);
    }

    return next(action);
  };
}

const thunk = createThunkMiddleware();
```

在上一节中, 我们的 reducer 函数如下:
```js
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
```
然后, 我们用 createStore 来创建 store:
```js
let store = createStore( 
    reducer, 
    applyMiddleware(logger, thunk)
);
```
此处, 我们用了2个 middleware , 一个 logger, 一个 thunk, 我们将
