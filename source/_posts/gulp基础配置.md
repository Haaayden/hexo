---
title: gulp基础配置
date: 2017-07-01 18:48:55
tags:
    - 工程化
    - gulp
---
# gulp 入门与自用配置
最近来了新活，搭建公司官网。用什么技术还真是让我好生纠结了一番。
我司的官网主要是展示性网页，所以第一时间排除了 vue，react 等框架，接下来好像只剩 jq 了，但是简单地用 jq 是不是太 low 了？现在是2017年，前端工程化都是炒烂的概念了，咱也不能落后。
开始考虑是想弄个简单点的，最先想到的是百度的 fis3，各种配置简单易学，但是思虑再三，还是放弃了。
第一点考虑是 fis3 的环境比较封闭，gulp 背靠 npm，各种插件安装使用都很方便；
第二点是 gulp 每一个 task 都是自己配置，比较方便我们熟悉整个流程，可控性更强，而且 gulp 可以配合 webpack 等等用在 react 等框架的使用中，至于 webpack 也是以后必学的，也算是个铺垫了；
综上，最后我们选择了 gulp。
<!--more-->
## Gulp 入门
1. 全局安装 gulp
```bash
$ npm install gulp -g
```
2. 进入项目目录，安装 gulp 作为本项目的依赖之一
```bash
$ cd ~/Code/Demo
$ npm install --save-dev gulp
```

3. 在项目根目录下创建 `gulpfile.js` 文件
```javascript
var gulp = require('gulp');
gulp.task('default', function() {
    // 任务代码
});
```

4. 运行 gulp
```bash
$ gulp
```

## gulp 常用的四个函数
### gulp.src
输出符合匹配模式的文件，将返回一个 stream，可以被 pipe 到其他插件中。
```javascript
gulp.src("app/css/*.css")
    .pipe(mincss())
    .pipe(gulp.dest("dist/css"));
```
### gulp.sest
能够写文件，将会返回 stream，同样可以被 pipe 到其他插件中。
### gulp.task
定义一个任务。
```javascript
gulp.task('taskname', function() {
    // task code
});
```
### gulp.watch
监听文件变动。
```javascript
gulp.watch("app/scss/**/*.scss", function() {
    // do something
});
```

## 常用 gulp 插件（不定期更新）
- `browser-sync`
可以实现本地服务器，实现监听文件变动自动刷新浏览器等功能。详细配置见[文档](http://www.browsersync.cn/)

- `gulp-sass` 编译 scss 文件
```javascript
var sass = require('gulp-sass');
gulp.task('sass', function() {
    return gulp.src("app/scss/*.scss")
        .pipe(sass())
        .pipe(gulp.dest("app/css"));
});
```

- `gulp-ejs`在 html 中使用 ejs 模板引擎
```javascript
var ejs = require('gulp-ejs');
gulp.task('ejs', function() {
    return gulp.src("app/templetes/*.html")
            .pipe(ejs())
            .pipe(gulp.dest("app/pages"));
});
```

- `gulp-autoprefixer`自动添加浏览器前缀
- `gulp-uglify`压缩混淆 js
- `ggulp-mini-css`压缩 css
- `gulp-htmlmin`压缩 html
- `gulp-imagemin`压缩图片

## gulp 路径匹配

### `*`匹配路径中0个或多个字符，但不会匹配路径分隔符，除非分隔符出现在末尾。
```javascript
// 匹配 style 路径下所有 js 文件
./style/*.js

// 匹配./style目录下所有文件
./style/*.*

// 只要层级相同，可以匹配任意目录下的任意js文件 比如./style/a/b.js
./style/*/*.js
```

### `**` 匹配0个或多个层级的目录
```javascript
// 匹配style目录及其所有子目录下的所有js文件，如能匹配
// ./style/a.js
// ./style/lib/res.js
// ./style/mudules/b/a.js
./style/**/*.js

// 匹配style目录下的所有目录和文件，比如能匹配
// ./style/a.js
// ./style/bb
// ./style/images/c.png
./style/**/*
```

## 参考链接
- [Gulp中文网](http://www.gulpjs.com.cn/docs/api/)
- [文件路径匹配模式globs](http://yangbo5207.github.io/gulp/2016/08/10/new.html)