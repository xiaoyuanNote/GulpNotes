## Gulp解决跨域的配置browser-sync + http-proxy-middleware

> 在前后端开发过程中，我使用gulp + browser-sync在本地运行了一网页，便于开发调试。该页面运行来localhost环境下，如果直接请求后台同事服务器，就是报跨域错误！
>
> 为了解决这个问题，必须作域名转发处理，以下就是使用http-proxy-middleware，来进行跨域请求。



### 简介

http-proxy-middleware用于后台将请求转发给其它服务器。

例如：我们当前主机A为[http://localhost:3000/](https://link.jianshu.com/?t=http://localhost:3000/)，现在浏览器发送一个请求，请求接口./api，这个请求的数据在另外一台服务器B上（http://10.119.168.87:4000），这时，就可通过在A主机设置代理，直接将请求发送给B主机。

以下是官网给出的例子代码：

```js
// javascript
 
const express = require('express');
const { createProxyMiddleware } = require('http-proxy-middleware');
 
const app = express();
 
app.use('/api', createProxyMiddleware({ target: 'http://www.example.org', changeOrigin: true }));
app.listen(3000);
 
// http://localhost:3000/api/foo/bar -> http://www.example.org/api/foo/bar
```



### 安装

```cmd
npm install --save-dev http-proxy-middleware
```



### 使用

###### gulpfile.js

```js
const browserSync = require('browser-sync').create()
const { createProxyMiddleware } = require('http-proxy-middleware')

const serverProxy = createProxyMiddleware('/api', {
  target: 'https://zwljewelry.com/branddev',
  // target: 'http://192.168.3.167:8081',
  changeOrigin: true,
  pathRewrite: {
    '^/api': ''
  },
  logLevel: 'debug'
})
```



###### test.js

```js
axios.get(`${host}/SDK/getSDKSign`)
  .then(function (response) {
  	// 操作
  })
  .catch(function (error) {
    console.log(error);
  });
```



### 所遇问题

##### 1. 启动报错：proxy is not a function

![](.\md.assets\1.png)

如图所示，查看了http-proxy-middleware的官方文档，发现最新的1.0.0版本已经对模块的引用作了明确的要求

###### 0.x.x版本的引用方式

```js
const proxy = require('http-proxy-middleware');
```

###### 1.0.0版本的引用方式

```js
const { createProxyMiddleware } = require('http-proxy-middleware');
```



### 参考链接

- [在gulp-server 中使用 http-proxy-middleware 配置代理跨域](https://blog.csdn.net/Charissa2017/article/details/104861908)
- [Http-proxy-middleware安装报错：proxy is not a function](https://blog.csdn.net/Benz_s600/article/details/105782913)
- [[http-proxy-middleware使用方法和实现原理（源码解读）](https://www.cnblogs.com/zhaoweikai/p/9969282.html)]

