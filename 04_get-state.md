```
// Tutorial 04 - get-state.js
```
## 04 - 获取状态

```
// How do we retrieve the state from our Redux instance?
```
我们如何从Redux实例中获取到程序的状态呢? 请看如下这段代码:

```js
import { createStore } from 'redux'

var reducer_0 = function (state, action) {
    console.log('reducer_0 was called with state', state, 'and action', action)
}

var store_0 = createStore(reducer_0)
```
```
// Output: reducer_0 was called with state undefined and action { type: '@@redux/INIT' }
```
输出: 程序使用参数state:undefined和action: { type: '@@redux/INIT' }调用reducer_0

```
// To get the state that Redux is holding for us, you call getState
```
你可以通过调用**getState**来获取Redux为我们持有的程序状态

```js
console.log('store_0 state after initialization:', store_0.getState())
```
```
// Output: store_0 state after initialization: undefined
```
输出: 初始化后store_0的状态为: undefined

```
// So the state of our application is still undefined after the initialization? Well of course it is,
// our reducer is not doing anything... Remember how we described the expected behavior of a reducer in
// "about-state-and-meet-redux"?
//     "A reducer is just a function that receives the current state of your application, the action,
//     and returns a new state modified (or reduced as they call it)"
// Our reducer is not returning anything right now so the state of our application is what
// reducer() returns, hence "undefined".
```
所以我们程序的状态在初始化后仍然是undefined? 好吧, 它当然就该如此,
因为我们的reducer什么也没做...
还记得我们在第二节中是如何描述我们所期望的reducer行为吗?
  - "Reducer仅仅是一个函数，这个函数接受你程序当前的状态和action作为参数然后返回一个新的被修改后的状态
由于目前我们的reducer什么也没有返回,
因此程序的状态也就是reducer()函数返回的"undefined".

```
// Let's try to send an initial state of our application if the state given to reducer is undefined:
```
如果reducer认为state是undefined时, 让我们试着赋予程序一个初始化的状态
```js
var reducer_1 = function (state, action) {
    console.log('reducer_1 was called with state', state, 'and action', action)
    if (typeof state === 'undefined') {
        return {}
    }

    return state;
}

var store_1 = createStore(reducer_1)
```
```
// Output: reducer_1 was called with state undefined and action { type: '@@redux/INIT' }
```
输出: 程序使用参数state: undefined和action { type: '@@redux/INIT' }
来调用reducer_1

```js
console.log('store_1 state after initialization:', store_1.getState())
```
```
// Output: store_1 state after initialization: {}
```
输出: 在初始化之后store_1的状态变为: {}

```
// As expected, the state returned by Redux after initialization is now {}
```
正如我们所期望的一样, 在初始化之后由Redux返回的程序状态如今变为{}

```
// There is however a much cleaner way to implement this pattern thanks to ES6:
```
这要感谢ES6的语法，我们可以用一个更清晰的方式来实现同样的功能:

```js
var reducer_2 = function (state = {}, action) {
    console.log('reducer_2 was called with state', state, 'and action', action)

    return state;
}

var store_2 = createStore(reducer_2)
```
```
// Output: reducer_2 was called with state {} and action { type: '@@redux/INIT' }
```
输出: 参数state: {} 和 action { type: '@@redux/INIT' }
被传入reducer_2函数并调用

```js
console.log('store_2 state after initialization:', store_2.getState())
```
````
// Output: store_2 state after initialization: {}

// You've probably noticed that since we've used the default parameter on state parameter of reducer_2,
// we no longer get undefined as state's value in our reducer's body.
```
输出: 在初始化之后store_2的状态为: {}
你可能已经注意到由于我们reducer_2函数的state参数上使用了默认值，我们将不会在reducer的函数内部得到undefined的状态值

```
// Let's now recall that a reducer is only called in response to an action dispatched and
// let's fake a state modification in response to an action type 'SAY_SOMETHING'
```
回想一下一个reducer仅仅是为了响应一个action的分发消息才会被调用,
那么让我们伪造一个状态修改的行为用来响应类型为'SYA_SOMETHING'的action分发

```js
var reducer_3 = function (state = {}, action) {
    console.log('reducer_3 was called with state', state, 'and action', action)

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

var store_3 = createStore(reducer_3)
```
```
// Output: reducer_3 was called with state {} and action { type: '@@redux/INIT' }
```
输出: 参数state: {}和action { type: '@@redux/INIT' }被传入reducer_3并调用

```js
console.log('store_3 state after initialization:', store_3.getState())
```
```
// Output: store_3 state after initialization: {}
```
输出: 在初始化之后store_3的状态为: {}

```
// Nothing new in our state so far since we did not dispatch any action yet. But there are few
// important things to pay attention to in the last example:
//     0) I assumed that our action contains a type and a value property. The type property is mostly
//        a convention in flux actions and the value property could have been anything else.
//     1) You'll often see the pattern involving a switch to respond appropriately
//        to an action received in your reducers
//     2) When using a switch, NEVER forget to have a "default: return state" because
//        if you don't, you'll end up having your reducer return undefined (and lose your state).
//     3) Notice how we returned a new state made by merging current state with { message: action.value },
//        all that thanks to this awesome ES7 notation (Object Spread): { ...state, message: action.value }
//     4) Note also that this ES7 Object Spread notation suits our example because it's doing a shallow
//        copy of { message: action.value } over our state (meaning that first level properties of state
//        are completely overwritten - as opposed to gracefully merged - by first level property of
//        { message: action.value }). But if we had a more complex / nested data structure, you might choose
//        to handle your state's updates very differently:
//        - using Immutable.js (https://facebook.github.io/immutable-js/)
//        - using Object.assign (https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)
//        - using manual merge
//        - or whatever other strategy that suits your needs and the structure of your state since
//          Redux is absolutely NOT opinionated on this (remember, Redux is a state container).
```
目前为止我们的状态没有任何变化，这是因为我们还没有分发action.
但在最后的这个例子中，有一些重要的东西我们需要注意:
* 该例子假设我们的action包含一个type属性和一个value属性. type属性是flux
  actions中最主要的一个约定, 而value属性可以是任何其他的值
* 你将在你的reducers函数中经常看到这样的代码范式:
  使用switch语句来恰当的响应传入的action.
* 当使用switch语句的时候，永远也不要忘记有一条"defualt: return state"语句,
  因为一旦忘记了加这条语句，你的reducer函数就可能返回一个undefined的程序状态
* 注意我们是如何通过合并当前状态和{message:
  action.value}来返回一个新的状态的, 这都要归功于ES7提供的超棒的理念
* 同时我们也要注意之所以ES7对象扩展的语法适合我们的例子,
  这是因为它基于我们的状态做了一个{message: action.value}的浅拷贝...
  但如果我们有一个更加复杂且内嵌的数据结果,
你可能选择一个完全不同的方式来更新你程序的状态, 比如:
  - 使用Immutable.js
  - 使用Object.assign
  - 手工合并
  - 或者其他任何适合你需求和程序数据结构的方法,
    因为Redux是绝对不会对此有任何的限制.(记住, Redux是一个状态容器)

```
// Now that we're starting to handle actions in our reducer let's talk about having multiple reducers and
// combining them.
```
既然我们已开始在reducer中处理actions了，让我们看一看拥有多个reducer函数的情况以及如何将他们组合在一起.

