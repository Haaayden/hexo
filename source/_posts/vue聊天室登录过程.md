---
title: vue聊天室登录过程
date: 2017-07-30 22:22:13
tags:
    - vue
---
# 使用 vue 的 SPA 的登录逻辑
这两天将之前分开写的几个聊天室的页面给串起来了，这里顺便记录下登录的逻辑。
其实说白了就是用户状态的判断，根据结果跳转不同的页面。
再往下分析的话，就三个点。
- 用户是否是已登录的状态
- 在何时去判断用户状态
- 未登录用户跳转登录页面（vue-router）

<!-- more -->

## 判断用户登录状态
这里我使用的是由纯前端判断用户的登录状态，在用户登录时，后端返回token，前端存储到本地，依据本地 token 是否存在判断用户是否登录。
```javascript
// src/main.js
new Vue({
    methods: {
        checkLogin: function () {
            // 检测本地是否存在 id cookie
            if (this.getCookie('token')) {
                // 用户已登录时跳转到内容页
                this.$router.push('/');
            } else {
                // 未登录时跳转到登录页
                this.$router.push('/login');
            }
        },
        getCookie: function (name) {
            return window.localStorage.getItem(name);
        }
    }
});
```

## 何时判断用户的状态
- 用户打开页面时
- 路由发生变化时
跳转到某些需要权限的页面时判断用户是否有权访问该页面也可以使用此方法。
```javascript
// src/main.js
new Vue({
    created () {
        // 打开页面的时候检测登录状态
        this.checkLogin();
    },
    watch: {
        // 监听路由变化
        '$router': 'checkLogin'
    }
});
```

这个周末比较匆忙，有时间的话补一下结合 vuex 登录的具体操作吧。
