---
title: vue项目个人配置
date: 2017-07-16 14:39:36
tags:
---

# Vue 自用配置
一般项目都会使用 vue-cli 初始化项目，大部分使用的都是默认配置，但是也有一些配置并不符合个人的使用习惯，所以存在定制化的需求，当然如果有团队规范的话，那就直接使用团队规范，没什么好说的。最近开了一个聊天室的小项目，发现有些默认配置存在一些不顺手的情况，就自己折腾了一下，在这里做个记录。
<!-- more -->
## ESlint
vue 默认使用的是 standard 模式，这里是 standard 模式的[相关文档](https://github.com/standard/standard/blob/master/docs/RULES-zhcn.md)。
vue-cli init项目时如果选择了 eslint，会在根目录生成 .eslintignore 和 .eslintrc.js文件，.eslintignore 文件类似于 .gitignore，描述哪些文件不必使用 eslint 规范，.eslintrc.js 文件中就是 eslint 的具体规则配置文件。
大部分还是按照默认配置来，只是修改了以下选项：
```javascript
    'rules': {
        // 0-off, 1-warn, 2-error
        // 关闭缩进使用两个空格
        'indent': 'off',
        // 关闭句尾不允许加分号
        'semi': 'off',
        // 将定义未使用变量的提示改为警告
        'no-unused-vars': 'warn'
    }
```
属性值 0 与 off 代表不启用该规则，1 与 warn 代表违反该规则时显示警告，2 与 error 代表违反该规则时显示错误提示。
关于是否使用分号，现在的讨论很多，个人觉得这是个人编码习惯的事情，像我从写第一行 js 开始就使用分号，习惯了，既然他没有错误，也不算编码恶习，那我就接着用，不必纠结。
关于缩进，那就更是如此了，我习惯用四个空格那就四个喽。

## 使用 scss

1. 安装以下三个依赖：
- node-sass
- sass-loader
- style-loader

2. 在 build 文件夹下的 webpack.base.conf.js 中添加如下规则：
```javascript
module: {
    rules: [
        {
            test: /\.scss$/,
            loaders: ['style', 'css', 'sass']
        }
    ]
}
```

3. 在 style 中添加 `lang="scss"`
```html
<style lang="scss">
@import './styles/index.scss';
</style>
```
这里我们使用的是 scss，如果填写 sass，需要注意 scss 与 sass 的语法差异，例如 sass 不允许花括号等等。
