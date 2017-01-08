```
// Tutorial 0 - introduction.js 
```

## 教程0 - 简介
 
```
// Why this tutorial? 
```

### 为什么会有该教程?

```
// While trying to learn Redux, I realized that I had accumulated incorrect knowledge about flux through
// articles I read and personal experience. I don't mean that articles about flux are not well written
// but I just didn't grasp concepts correctly. In the end, I was just applying documentation of different
// flux frameworks (Reflux, Flummox, FB Flux) and trying to make them match with the theoretical concept I read
// about (actions / actions creators, store, dispatcher, etc).
```

当我尝试学习Redux的时候，我发现我通过阅读flux相关文章和个人经验所积累的flux知识是不正确的.这并不是说这些文章写的不好，而是我并没有正确地掌握其概念. 最后,我勉强地将其运用到不同的flux框架(Reflux, Flummon, FB Flux)文档上并尝试去匹配我所读到的那些相关理论概念(actions / action构造器, store, dispatcher等)

```
// Only when I started using Redux did I realize that flux is more simple than I thought. This is all
// thanks to Redux being very well designed and having removed a lot of "anti-boilerplate features" introduced
// by other frameworks I tried before. I now feel that Redux is a much better way to learn about flux
// than many other frameworks. That's why I want now to share with everyone, using my own words,
// flux concepts that I am starting to grasp, focusing on the use of Redux.
```

只有当我开始使用Redux的时候才使我意识到flux要比我认为的更简单. 这全都要感谢Redux良好的设计以及移除了一大堆我以前尝试过的其他框架所引入的"反模版特性".这就是为什么现在我想用自己的语言来与大家分享那些我正开始掌握并聚焦于Redux使用的flux概念

```
// You may have seen this diagram representing the famous unidirectional data flow of a flux application:
```
你有可能已经见过这样一张图, 它展现了flux程序中有名的单向数据流

```
                 _________               ____________               ___________
                |         |             |            |             |           |
                | Action  |------------▶| Dispatcher |------------▶| callbacks |
                |_________|             |____________|             |___________|
                     ▲                                                   |
                     |                                                   |
                     |                                                   |
 _________       ____|_____                                          ____▼____
|         |◀----|  Action  |                                        |         |
| Web API |     | Creators |                                        |  Store  |
|_________|----▶|__________|                                        |_________|
                     ▲                                                   |
                     |                                                   |
                 ____|________           ____________                ____▼____
                |   User       |         |   React   |              | Change  |
                | interactions |◀--------|   Views   |◀-------------| events  |
                |______________|         |___________|              |_________|


// In this tutorial we'll gradually introduce you to concepts of the diagram above. But instead of trying
// to explain this complete diagram and the overall flow it describes, we'll take each piece separately and try to
// understand why it exists and what role it plays. In the end you'll see that this diagram makes perfect sense
// once we understand each of its parts.
```

在本教程中,我会逐步的向你介绍上述图表中的那些概念. 但我不会尝试去解释整个图表及其描述的全部数据流, 我会将每一部分单独拿出来并尝试的去理解为什么需要它，它又扮演着怎样的角色. 最后你将看到只要我们明白了这张图的各个部分那么它就非常清楚了.

```
// But before we start, let's talk a little bit about why flux exists and why we need it...
// Let's pretend we're building a web application. What are all web applications made of?
// 1) Templates / html = View
// 2) Data that will populate our views = Models
// 3) Logic to retrieve data, glue all views together and to react accordingly to user events,
//    data modifications, etc. = Controller
```
但在开始前, 我想简单地聊一下为什么会有flux, 为什么我们需要它...
假如我们正在构建一个web程序，那整个web程序由什么组成呢?
* 视图 **View** = 模版 / HTML
* 模型 **Models** = 用于生成视图的数据
* 控制器 **Controller** = 获取数据的逻辑部分, 将所有视图粘合在一起的部分, 以及对用户事件或数据修改作出相应的反应部分等

```
// This is the very classic MVC that we all know about. But it actually looks like concepts of flux,
// just expressed in a slightly different way:
// - Models look like stores
// - user events, data modifications and their handlers look like
//   "action creators" -> action -> dispatcher -> callback
// - Views look like React views (or anything else as far as flux is concerned)
```
这就是我们所熟知的非常经典的MVC. 但实际上它看起来也有些像flux的概念,只不过flux以另一种稍微不同的方式来表达:
* 模型Model就好比Store
* 用户事件,数据的改动以及相应的处理函数就好比:**"action构造器"->action->分发器->回调**部分
* 视图View就好比React视图

```
// So is flux just a matter of new vocabulary? Not exactly. But vocabulary DOES matter, because by introducing
// these new terms we are now able to express more precisely things that were regrouped under
// various terminologies... For example, isn't a data fetch an action? Just like a click is also an action?
// And a change in an input is an action too... Then we're all already used to issuing actions from our
// applications, we were just calling them differently. And instead of having handlers for those
// actions directly modify Models or Views, flux ensures all actions go first through something called
// a dispatcher, then through our stores, and finally all watchers of stores are notified.
```
那么flux仅仅只是个新词而已吗? 并非完全如此. 但是词汇也蛮重要的, 因为通过引入这些新的术语, 现在我们就能够更准确的表达那些用各种各样术语重新组合起来的东西. 比如: 获取数据并非是一个action? 还是如点击事件一样也是一个action呢? 一个输入的变化也是一个action... 我们都已经习惯了从我们的程序中触发actions, 只不过称呼不同而已. 并且flux并不会拥有那些能直接修改模型或视图的actions的处理函数，而是确保所有的actions先要通过一个被称为dispatcher的部分, 然后再通过我们的stores,最后所有stores的关注者都会被通知.

```
// To get more clarity how MVC and flux differ, we'll // take a classic use-case in an MVC application:
// In a classic MVC application you could easily end up with:
// 1) User clicks on button "A"
// 2) A click handler on button "A" triggers a change on Model "A"
// 3) A change handler on Model "A" triggers a change on Model "B"
// 4) A change handler on Model "B" triggers a change on View "B" that re-renders itself
```
为了更加的明白MVC与flux有什么不同, 我们将用MVC程序中一个经典的例子,在这个例子中,你能很容易想到如下场景:
* 用户点击按钮"A"
* 按钮"A"的点击处理函数将触发模型"A"的一个变化
* 模型"A"的变化处理函数将触发模型"B"的一个变化
* 模型"B"的变化处理函数将触发视图"B"的一个变化并渲染自己

```
// Finding the source of a bug in such an environment when something goes wrong can become quite challenging
// very quickly. This is because every View can watch every Model, and every Model can watch other Models, so
// basically data can arrive from a lot of places and be changed by a lot of sources (any views or any models).
```
在这样的环境里, 当问题出现的时候, 找出bug的源头很快地变得颇有挑战性. 这是因为每个视图都关注着每个模型, 而每个模型又可以关注着其他的模型, 所以从根本从根本上来说，数据可以来自许多的地方并且被许多的源头(任何的视图或者模型)改变

```
// Whereas when using flux and its unidirectional data flow, the example above could become:
// 1) user clicks on button "A"
// 2) a handler on button "A" triggers an action that is dispatched and produces a change on Store "A"
// 3) since all other stores are also notified about the action, Store B can react to the same action too
// 4) View "B" gets notified by the change in Stores A and B, and re-renders
```
与之相反当使用flux和它的单向数据流的时候, 上面的例子就变成:
* 用户点击按钮"A"
* 按钮"A"的处理函数将触发一个action, 这个action将被分发并使模型A产生一个变化
* 由于其他的模型也都会收到一个有关该action的通知消息, Store B也能对此作出相应的处理
* 视图"B"接收到模型A和B的变化通知后进行重绘

```
// See how we avoid directly linking Store A to Store B? Each store can only be
// modified by an action and nothing else. And once all stores have replied to an action,
// views can finally update. So in the end, data always flows in one way:
//     action -> store -> view -> action -> store -> view -> action -> ...
```
看一看我们是怎样避免直接的将Store A 和Store B关联起来的? 每一个Store只能够被一个Action修改. 那么一旦所有的stores都应答了这个action,视图最终将进行更新. 因此最后，数据总是如下一样流动:
**action -> store -> view -> action -> store -> view -> action -> ...**

```
// Just as we started our use case above from an action, let's start our tutorial with
// actions and action creators.
```
正如在上述的例子中我们是从action开始一样，我们的教程也将从action和action构造器出发.
