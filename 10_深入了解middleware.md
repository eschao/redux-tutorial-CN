## 深入了解 middleware

在上一节中我们简单的介绍了 middleware 并展示了其基本的用法. 但我们可能会疑惑: 
* 为什么 middleware 函数要遵循那样的签名及结构? 
* 为什么需要三层函数? 
* Redux 支持 middleware 的 applyMiddleware 函数到底又做了什么?

明白 middleware 的前因后果, 是我们理解 Redux 设计的一个很重要的部分. 在本节中, 我会以翻译官方的 Middleware 一节为主并辅以我自己的理解来解析 middleware , 希望能让你更深入一步地了解 middleware.

### 问题: 记录日志
