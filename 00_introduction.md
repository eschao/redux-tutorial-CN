```
// Tutorial 0 - introduction.js 
```

## 0 - 简介
 
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

在努力学习Redux的同时，我发现通过阅读相关文章和自身经验所积累的Flux知识是不正确的. 不是说这些文章写的不好，而是我并没有正确地掌握其概念. 最后,我勉强地将这些知识运用于不同的Flux框架(Reflux, Flummon, FB Flux)文档上并努力地匹配我在这些文档中读到的理论概念(比如:Actions / Action构造器, Store, Dispatcher等)

```
// Only when I started using Redux did I realize that flux is more simple than I thought. This is all
// thanks to Redux being very well designed and having removed a lot of "anti-boilerplate features" introduced
// by other frameworks I tried before. I now feel that Redux is a much better way to learn about flux
// than many other frameworks. That's why I want now to share with everyone, using my own words,
// flux concepts that I am starting to grasp, focusing on the use of Redux.
```

只有当我开始使用Redux的时候才让我意识到Flux要比我想的更简单. 这全都要归功于Redux的良好设计以及移除了一大堆由那些我曾试过的Flux框架所引入的"反模版功能".这就是为什么我现在想用自己的语言来跟大家分享那些我正开始掌握,并集中在Redux使用上的Flux概念

```
// You may have seen this diagram representing the famous unidirectional data flow of a flux application:
```
你有可能已经见过下面这张图表, 它展现的是Flux程序中著名的单向数据流
* Action - 动作
* Action Creators - 动作构造器
* Dispatcher - 分发器
* Callbacks - 回调
* Store - 存储
* User interactions - 用户交互
* React Views - React视图
* Change events - 更改事件
 
 
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
| Web API |     | Creators |                                        | Store   |   
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

在本教程中,我会逐步地向你介绍上述图表中的概念. 但我不会尝试去解释整个图表以及它所描述的整体数据流, 我会将图表中的每一部分单独拿出来并试着去理解为什么需要它，它又在图表中扮演了什么角色. 最后你将看到一旦我们明白了图表的各个部分, 那么整张图表就非常清晰了.

```
// But before we start, let's talk a little bit about why flux exists and why we need it...
// Let's pretend we're building a web application. What are all web applications made of?
// 1) Templates / html = View
// 2) Data that will populate our views = Models
// 3) Logic to retrieve data, glue all views together and to react accordingly to user events,
//    data modifications, etc. = Controller
```
但在开始之前, 我想简单地聊一下为什么会有Flux, 为什么我们需要它...

假如我们正在构建一个web应用程序，那整个web应用程序由哪些部分构成呢?
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
这就是我们所熟知的十分经典的MVC. 但实际上它也有些像Flux概念, 只不过Flux用一种略微不同的方式来表达:
* 模型Model = **Store(存储)**
* 用户事件,数据的改动以及相应的处理函数 = **"action creator(构造器)"->action->dispatcher(分发器)->callback(回调)**
* 视图View = **React View(React视图)**
```
// So is flux just a matter of new vocabulary? Not exactly. But vocabulary DOES matter, because by introducing
// these new terms we are now able to express more precisely things that were regrouped under
// various terminologies... For example, isn't a data fetch an action? Just like a click is also an action?
// And a change in an input is an action too... Then we're all already used to issuing actions from our
// applications, we were just calling them differently. And instead of having handlers for those
// actions directly modify Models or Views, flux ensures all actions go first through something called
// a dispatcher, then through our stores, and finally all watchers of stores are notified.
```
那么Flux仅仅只是个新词而已吗? 并非完全如此. 但词汇也很蛮重要, 因为这些新术语的引入, 我们如今就能够更准确的表达那些用各种各样术语重新组合起来的东西. 比如: 获取数据会像点击事件一样也是一个Action吗? 一个输入部分的变化也是一个Action吗?... 总之我们都已经习惯于从我们的程序中分发Actions, 只不过对此叫法不同而已. 然而Flux并不会拥有那些能够直接修改Models或Views的Actions处理函数，而是确保所有的Actions先要通过一个被称为Dispatcher的部分, 然后再通过我们的Stores部分, 最后所有Stores的关注者都会被通知.

```
// To get more clarity how MVC and flux differ, we'll 
// take a classic use-case in an MVC application:
// In a classic MVC application you could easily end up with:
// 1) User clicks on button "A"
// 2) A click handler on button "A" triggers a change on Model "A"
// 3) A change handler on Model "A" triggers a change on Model "B"
// 4) A change handler on Model "B" triggers a change on View "B" that re-renders itself
```
为了更加地明白MVC与Flux有什么不同, 我们将使用MVC应用程序中的一个经典例子来说明. 

在这个例子中, 你能很容易想到如下场景:
* 用户点击按钮"A"
* 按钮"A"的点击处理函数将触发模型"A"的一个变化
* 模型"A"的变化处理函数将触发模型"B"的一个变化
* 模型"B"的变化处理函数将触发视图"B"的一个变化并重绘视图"B"

```
// Finding the source of a bug in such an environment when something goes wrong can become quite challenging
// very quickly. This is because every View can watch every Model, and every Model can watch other Models, so
// basically data can arrive from a lot of places and be changed by a lot of sources (any views or any models).
```
当问题出现的时候, 在这样的环境里要找出Bug的源头很快就变得非常有挑战性. 这是因为每个View都监视着每个Model, 而每个Model又可以监视着其他的Models, 所以从根本上来说，数据可以来自很多的地方并且可以被很多的源头(任何的Views或者Models)所改变.

```
// Whereas when using flux and its unidirectional data flow, the example above could become:
// 1) user clicks on button "A"
// 2) a handler on button "A" triggers an action that is dispatched and produces a change on Store "A"
// 3) since all other stores are also notified about the action, Store B can react to the same action too
// 4) View "B" gets notified by the change in Stores A and B, and re-renders
```
与之相反, 当使用Flux及其单向数据流的时候, 上面的例子就变成:
* 用户点击按钮"A"
* 按钮"A"的处理函数将触发一个Action, 这个Action将被分发并使Store "A"产生一个变化
* 由于其他的Store(存储)也都会收到该Action的分发通知, 因而Store B也能对该Action作出相应的处理
* View "B"接收到Store A和B的变化通知后将进行重绘

```
// See how we avoid directly linking Store A to Store B? Each store can only be
// modified by an action and nothing else. And once all stores have replied to an action,
// views can finally update. So in the end, data always flows in one way:
//     action -> store -> view -> action -> store -> view -> action -> ...
```
看看我们是如何避免直接地将Store A 和Store B关联起来的? 每一个Store只能够被一个Action修改. 那么一旦所有的Stores都应答了这个Action,View最终将进行更新. 因此最后，数据总是沿着一个方向流动:

**action -> store -> view -> action -> store -> view -> action -> ...**

```
// Just as we started our use case above from an action, let's start our tutorial with
// actions and action creators.
```
正如我们在上述例子中是从Action开始的一样，本教程也将从Action和Action Creator出发.
