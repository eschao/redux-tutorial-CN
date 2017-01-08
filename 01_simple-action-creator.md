```
// Tutorial 1 - simple-action-creator.js
```

## 教程1 - 简单的action构造器

```
// We started to talk a little about actions in the introduction but what exactly are those "action creators"
// and how are they linked to "actions"?
```
在简介中我们稍稍的涉及了一下actions, 但究竟什么是"action构造器"? 他们又怎样与"actions"关联起来的呢？

```
// It's actually so simple that a few lines of code can explain it all!
```
事实上它们如此的简单以至于了了数行代码就可以完全的解释!

```js
// The action creator is just a function...
// action构造器只不过是一个函数...
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

然而，需要特别提到的一件事就是action的格式. 在flux中这种约定就是action是一个包含有**type**属性的对象. **type**允许对action做进一步的处理. 当然, action也可以包含其他的属性用于传递任何你想要传递的数据.

```
// We'll also see later that the action creator can actually return something other than an action,
// like a function. This will be extremely useful for async action handling (more on that
// in dispatch-async-action.js).
```

同样我们将在后续中看到action构造器实际上可以返回一些非action的东西，比如一个函数. 这对于异步处理极为有用

```js
// We can call this action creator and get an action as expected:
// 我们可以调用这个action构造器去获得一个如期的action
console.log(actionCreator())
// Output: { type: 'AN_ACTION' }
```

```
// Ok, this works but it does not go anywhere...
// What we need is to have this action be sent somewhere so that
// anyone interested could know that something happened and could act accordingly.
// We call this process "Dispatching an action".
```

好吧, 这虽然能工作但还是在原地...
我们所需要的是使该action被发送到某个地方以便对其感兴趣的人能够得知有事发生了并相应的完成一些动作. 我们称这个过程为"分发一个action“

```
// To dispatch an action we need... a dispatch function ("Captain obvious").
// And to let anyone interested know that an action happened, we need a mechanism to register
// "handlers". Such "handlers" to actions in traditional flux application are called stores and
// we'll see in the next section how they are called in Redux.
```

为了分发一个action，我们需要的... 显然是一个分发函数了. 为了让感兴趣的人得知该action发生了, 我们需要一个机制去注册"处理函数". 这些对action的"处理函数"在传统的flux程序中被称之为stores, 而我们将在接下来的章节中看到他们在Redux中叫什么.

```
// So far here is the flow of our application:
// ActionCreator -> Action
```
到目前为止我们程序的流程就是:

**Action构造器 -> Action**


```
// Read more about actions and action creators here:
// http://rackt.org/redux/docs/recipes/ReducingBoilerplate.html
```

