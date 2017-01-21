```
// Tutorial 1 - simple-action-creator.js
```

## 01 - 简单的 action 构造器

```
// We started to talk a little about actions in the introduction but what exactly are those "action creators"
// and how are they linked to "actions"?
```
在[简介一节](/00_简介.md)中我们稍稍地涉及了一下 actions , 但究竟什么是" action creator "? 它们又是怎样与" actions "关联起来的呢？

```
// It's actually so simple that a few lines of code can explain it all!
```
事实上它们如此的简单以至于了了数行代码就可以完全地解释!

```js
// The action creator is just a function...
// action creator只不过是一个函数...
var actionCreator = function() {
    // ...that creates an action (yeah, the name action creator is pretty obvious now) and returns it
    // ...这会构建一个action并返回
    return {
        type: 'AN_ACTION'
    }
}
```
```
// So is that all? yes.
```

这难道就是所有的吗? 没错.

```
// However, one thing to note is the format of the action. This is kind of a convention in flux
// that the action is an object that contains a "type" property. This type allows for further
// handling of the action. Of course, the action can also contain other properties to
// pass any data you want.
```

然而，需要特别提到的是 action 的格式. 在 Flux 中规定 action 对象要包含一个 **type** 属性. **type** 允许了对 action 的进一步处理. 当然,  action 也可以包含其他的属性用于传递任何你想要传递的数据.

```
// We'll also see later that the action creator can actually return something other than an action,
// like a function. This will be extremely useful for async action handling (more on that
// in dispatch-async-action.js).
```

同样我们将在后续中看到 action creator 实际上可以返回一些非 action 的东西，比如返回一个函数. 这对于异步处理极为有用.

```js
// We can call this action creator and get an action as expected:
// 我们调用该action creator可以获得一个期望的action
console.log(actionCreator())
// Output: { type: 'AN_ACTION' }
```

```
// Ok, this works but it does not go anywhere...
// What we need is to have this action be sent somewhere so that
// anyone interested could know that something happened and could act accordingly.
// We call this process "Dispatching an action".
```

好吧, 这虽然能工作但还是待在原地不动...

我们所需要的是把该 action 发送到某个地方以便对其感兴趣的人能够得知某事发生了并相应地完成一些动作. 我们称这个过程为 "分发一个 action “

```
// To dispatch an action we need... a dispatch function ("Captain obvious").
// And to let anyone interested know that an action happened, we need a mechanism to register
// "handlers". Such "handlers" to actions in traditional flux application are called stores and
// we'll see in the next section how they are called in Redux.
```

为了分发一个 action ，我们需要的... 显然是一个分发函数. 为了让感兴趣的人得知该 actio n发生了, 我们需要一个机制去注册 "handlers" . 这些对 action的 "handlers" 在传统的 Flux 程序中被称之为 stores , 而我们将在接下来的章节中看到他们在 Redux 中被称之为什么.

```
// So far here is the flow of our application:
// ActionCreator -> Action
```
到目前为止我们程序的流向就是:

**Action Creator -> Action**


```
// Read more about actions and action creators here:
// http://rackt.org/redux/docs/recipes/ReducingBoilerplate.html
```

