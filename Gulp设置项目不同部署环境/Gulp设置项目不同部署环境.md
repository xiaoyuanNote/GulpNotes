##  Gulp设置项目不同部署环境

> 在开发过程中，需要连接本地后台，进行前后端对接开发；在项目上线后，需要连接正式服务器；或者连接其他测试服务器，UAT环境。
>
> 我们不可能每次都手动修改代码内容，操作不仅繁琐，而且有风险。这就需要针对不同的部署环境进行项目构建配置。



参考网上众多内容，我使用了`gulp-preprocess插件`，该插件构建一个自定义上下文或环境配置，来预处理HTML、JavaScript和其他文件。



### 官方例子

##### Gulpfile

```js
var preprocess = require("gulp-preprocess");
 
gulp.task("html", function() {
  gulp
    .src("./app/*.html")
    .pipe(preprocess({ context: { NODE_ENV: "production", DEBUG: true } })) // To set environment variables in-line
    .pipe(gulp.dest("./dist/"));
});
 
gulp.task("scripts", function() {
  gulp
    .src(["./app/*.js"])
    .pipe(preprocess())
    .pipe(gulp.dest("./dist/"));
});
```



##### HTML

```html
<head>
  <title>Your App
 
  <!-- @if NODE_ENV='production' -->
  <script src="some/production/lib/like/analytics.js"></script> 
  <!-- @endif -->
 
</head>
<body>
  <!-- @ifdef DEBUG -->
  <h1>Debugging mode - <!-- @echo RELEASE_TAG --> </h1>
  <!-- @endif -->
  <p>
  <!-- @include welcome_message.txt -->
  </p>
</body>
```



Javascript

```js
var configValue = "/* @echo FOO */" || "default value";
 
// @ifdef DEBUG
someDebuggingCall();
// @endif
```

以上例子所示，首先选中指定的文件，通过`preprocess`插件引入配置好的自定义上下文，然后在文件里使用注解，根据上下文的`NODE_ENV`，来正确的输出预期内容。



### 实际操作

##### 下载插件

```cmd
npm install --save-dev gulp-preprocess
```



##### 运行环境

- `development` 开发环境
- `production` 正式环境
- `uat` 客户测试环境

```JSON
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "serve": "set NODE_ENV=development && gulp script",
    "build": "set NODE_ENV=production && gulp script",
    "uat": "set NODE_ENV=uat && gulp script"
},
```



##### 代码支持

###### gulpfile.js

掉用相关插件进行配置，这里，我们通过`process.env.NODE_ENV`传入一个运行环境的参数

```js
const gulp = require('gulp')
const preprocess = require('gulp-preprocess')

// 定义一个任务
gulp.task('script', function () {
  return gulp.src('./src/js/test.js')
    .pipe(preprocess({
      context: {
        // 此处可接受来自调用命令的 NODE_ENV 参数，默认为 development 开发测试环境
        NODE_ENV: process.env.NODE_ENV.trim() || 'development',
      },
    }))
    .pipe(gulp.dest('./dist'))
})
```



###### test.js

我们使用**注解**的方式，根据上下文中的`NODE_ENV`有对应的选项，正确的输出预期的结果

```js
var publicjs = {
  // @if NODE_ENV = 'development'
  env: '开发测试环境',
  // @endif
  // @if NODE_ENV = 'production'
  env: '正式环境',
  // @endif
  // @if NODE_ENV = 'uat'
  env: '客户验收测试环境',
  // @endif
}
```



##### 运行代码

```cmd
npm run serve
```

```js
var publicjs = {
  env: '开发测试环境',
}
```

```
npm run build
```

```js
var publicjs = {
  env: '正式环境',
}
```

```
npm run uat
```

```js
var publicjs = {
  env: '客户验收测试环境',
}
```



### 所遇问题

##### 1. npm 中设置环境NODE_ENV变量，打印process.env.NODE_ENV确实是"production",但是判断process.env.NODE_ENV === "production" 是false

```json
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "serve": "set NODE_ENV=production && gulp dev && gulp server",
    "build": "set NODE_ENV=production && gulp dev && gulp build"
},
```

这个问题真是很搞人，折腾了一个上午。我刚开始以为是`gulp-preprocess`插件和运行代码的问题，前后测试了很多次，都没法修改过来，后来我注意到了`process.env.NODE_ENV`参数，打印出了production，但是却无法让process.env.NODE_ENV = production

![](D:\github\GulpNotes\Gulp设置项目不同部署环境\md.assets\1.png)

我后来是使用`typeof process.env.NODE_ENV`来测试参数值类型，得到的是String。这让我很是费解，到底两个值相同的String为何不能相等。

没办法只能Google！

![](.\md.assets\2.png)

卧槽！！！

因为我的NODE_ENV是production，多了一个空格

![](.\md.assets\3.png)

呜呜呜...



### 参考链接

- ##### [Gulp 不同部署环境设置](https://www.jianshu.com/p/bd2f97e16d86)

- ###### [npm 中设置环境NODE_ENV变量，判断失败打印process.env.NODE_ENV确实是"development",但是判断process.env.NODE_ENV === "development" 是false](https://www.cnblogs.com/sugartang/p/12402191.html)