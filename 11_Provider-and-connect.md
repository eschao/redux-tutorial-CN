```
// Tutorial 11 - Provider-and-connect.js
```

## 11 - Provider和Connect

```
// This is the final tutorial and the one that will show you how to bind together Redux and React.
```
这是最后的一节, 也是展示如何将Redux和React绑定在一起的一节.

```
//In this example, you will need a browser.
```
在接下来的例子中, 你需要一个浏览器.

```
// All explanations for this example are inlined in the sources inside ./11_src/src/.
```
对于例子的所有解释都内嵌在源代码文件中 ./11_src/src/..

```
// Once you've read the lines below, start with 11_src/src/server.js.
```
一旦你阅读完下面这几行, 请从文件 11_src/src/server.js开始.

```
// To build our React application and serve it to a browser, we'll use:
// - A very simple node HTTP server (https://nodejs.org/api/http.html)
// - The awesome Webpack (http://webpack.github.io/) to bundle our application,
// - The magic Webpack Dev Server (http://webpack.github.io/docs/webpack-dev-server.html)
//     to serve JS files from a dedicated node server that allows for files watch
// - The incredible React Hot Loader http://gaearon.github.io/react-hot-loader/ (another awesome
//     project of Dan Abramov - just in case, he is Redux's author) to have a crazy
//     DX (Developer experience) by having our components live-reload in the browser
//     while we're tweaking them in our code editor.
```
为了构建一个React程序并运行在浏览器上, 我们会用到:
* 一个非常简单的HTTP服务器 (...)
* 非常不错的打包工具Webpack
* 拥有魔力的Webpack Dev Server
* 非常不错的React热加载

```
// An important point for those of you who are already using React: this application is built
// upon React 0.14.
```
对于已经开始使用React的你, 最重要的一点是: 该例子基于React 0.14.

```
// I won't detail Webpack Dev Server and React Hot Loader setup here since it's done pretty
// well in React Hot Loader's docs.
import webpackDevServer from './11_src/src/webpack-dev-server'
// We request the main server of our app to start it from this file.
import server from './11_src/src/server'

// Change the port below if port 5050 is already in use for you.
// if port equals X, we'll use X for server's port and X+1 for webpack-dev-server's port
const port = 5050

// Start our Webpack dev server...
webpackDevServer.listen(port)
// ... and our main app server.
server.listen(port)

console.log(`Server is listening on http://127.0.0.1:${port}`)

// Go to 11_src/src/server.js...
```
