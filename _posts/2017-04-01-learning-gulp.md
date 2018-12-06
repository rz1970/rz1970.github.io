---
title: 前端构建工具之Gulp
tags: gulp
---

## 全局安装部署Gulp-Cli
1. gulp是基于[nodeJS](https://nodejs.org/en/)的一个前端构建工具。所以需要先到nodeJS的官网上下载安装nodejs。

2. 查看nodeJS是否安装成功
```
$ node -v
$ npm -v
```
3. Install gulp globally:
If you have previously installed a version of gulp globally, please run `npm rm --global gulp` to make sure your old version doesn't collide with `gulp-cli`.
```
$ npm install --global gulp-cli
```
## 每个工程中部署Gulp
1. create your project folder
```
$ mkdir project_folder;
$ git init
```
2. Initialize your project directory:
```
$ npm init
```
3. Install gulp in your project devDependencies:
```
$ npm install --save-dev gulp
```
4. Create a gulpfile.js at the root of your project:
```
var gulp = require('gulp');

gulp.task('default', function() {
  // place code for your default task here
});
```
5. Run gulp
```
$ gulp
```
The default task will run and do nothing.To run individual tasks, use `gulp <task> <othertask>`.

## gulp核心API
Gulp的核心API只有4个：`src`、`dest`、`task`、`watch`
* gulp.src(globs\[, options])：指明源文件路径
  + globs：路径模式匹配
  + options：可选参数
* gulp.dest(path\[, options])：指明处理后的文件输出路径
  + path：路径（一个任务可以有多个输出路径）
  + options：可选参数
* gulp.task(name\[, deps], fn)：注册任务
  + name：任务名称（通过 gulp name 来执行这个任务）
  + deps：可选的数组，在本任务运行中所需要所依赖的其他任务（当前任务在依赖任务执行完毕后才会执行）
  + fn：任务函数（function方法）；
* gulp.watch(glob\[, opts], tasks)：监视文件的变化并运行相应的任务
  + glob：路径模式匹配
  + opts：可以选配置对象
  + taks：执行的任务；

## 前端项目构建目录的基本目录结构
```
my-gulp（项目文件夹）
  + node_modules (Gulp组件目录)
  + dist (发布环境)
      + css (编译后的CSS文件)
        ─ etc...
      + images (压缩后的图片文件)
        ─ etc...
      + js (编译后的JS文件)
        ─ etc...
　　    ─ html (静态文件)
  + src 开发环境
      + sass SASS文件 (可以替换成stylus)
        ─ etc...
      + images (图片文件)
        ─ etc...
      + js (JS文件)
        ─ etc...
      ─ html (静态文件)
  ─ gulpfile.js (Gulp任务文件)
```