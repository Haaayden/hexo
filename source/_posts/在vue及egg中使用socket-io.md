---
title: 在vue及egg中使用socket.io
date: 2017-07-23 16:20:32
tags:
    - faramework
    - vue
    - egg
---
# vue-socket.io 及 egg-socket.io 的简单使用
这是一篇究极文不对题的博客，因为我要介绍的并不是 socket.io 在 vue 以及 egg 的使用。
![](https://ws4.sinaimg.cn/large/006tNc79ly1fhu0l936k7j306s06swec.jpg)
既然你还想知道然后。。。。。。。我就告诉你
<!-- more -->
我要介绍的是 vue-socket.io 以及 egg-socket.io。
因为最近手又痒了想写个聊天室熟悉下 vue 以及 nodeJs。
真实原因是，我身边最后一个单身的程序员竟然也找到了女朋友
噩耗传来
![](https://ws4.sinaimg.cn/large/006tNc79ly1fhtx7zrnzmj306s061wee.jpg)
一番思考
![](https://ws4.sinaimg.cn/large/006tNc79ly1fhtxf5va7qj309106s747.jpg)
我想通了
![](https://ws3.sinaimg.cn/large/006tNc79ly1fhtxgbf9u3j306u06sq2y.jpg)
故事讲完了。
总结：聊天室项目，前端 vue，后端 egg，使用 vue-socket.io，egg-socket.io

## egg-socket.io 的使用
官方文档看这里 [egg-socket.io](https://github.com/eggjs/egg-socket.io)
接下来的内容其实与文档里差不多，介意的童鞋略过就好，目前只是简单的引入，下周往后会写复杂些的逻辑，在后面的文章会介绍。
贴下目录结构
![](https://ws3.sinaimg.cn/large/006tNc79ly1fhtxvvtlg9j30c60osaaa.jpg)

1. 下载安装
```bash
$ npm install --save egg-socket.io
```
2. 开启插件以及插件配置
开启插件
```javascript
// app/config/plugin.js
exports.io = {
    enable: true,
    package: 'egg-socket.io'
};
```
插件配置
```javascript
// app/config/config.default.js
// 这里的 auth 以及 filter 是待会会编写的两个中间件，用于不用依据自己的情况选择即可
exports.io = {
    namespace: {
        '/': {
            connectionMiddleware: [ 'auth' ],
            packetMiddleware: [ 'filter' ],
        }
    }
};
```
3. 编写中间件
```javascript
// app/io/middlewware/auth.js
// 这个中间件的作用是提示用户连接与断开的，连接成功的消息发送到客户端，断开连接的消息在服务端打印
module.exports = app => {
    return function* (next) {
        this.socket.emit('res', 'connected!');
        yield* next;
        console.log('disconnection!');
    };
};

// app/io/middleware/filter.js
// 这个中间件的作用是将接收到的数据再发送给客户端
module.exports = app => {
    return function* (next) {
        this.socket.emit('res', 'packet received!');
        console.log('packet:', this.packet);
        yield* next;
    };
};
```
4. 编写控制器
```javascript
// app/io/controller/chat.js
// 将收到的消息发送给客户端
module.exports = app => {
    return function* () {
        const self = this;
        const message = this.args[0];
        console.log('chat 控制器打印', message);
        this.socket.emit('res', `Hi! I've got your message: ${message}`);
    };
};
```
5. 编写路由
```javascript
// app/router.js
// 这里表示对于监听到的 chat 事件，将由 app/io/controller/chat.js 处理
module.exports = app => {
    app.io.of('/').route('chat', app.io.controllers.chat);
};
```

### tip：
```javascript
// controller 中
// 发送给自己
this.socket.emit('eventName', 'value');
// 发送给除了自己外的所有人
this.socket.broadcast.emit('eventName', 'value');
// 发送给所有人，包括自己
this.server.sockets.emit('eventName', 'value');
```
前两种书写方式都与 socket.io 一致，最后一种 socket.io 使用的是 io.sockets.emit，但是无奈小白怎么都找不到这个 io，但是这怎么可能阻止机制的我呢，直接开 debug 模式，将当前环境下的对象打印出来，找到了一个个 server，里面有 sockets 对象，试了下，竟然成功了，暂时没时间研究为什么，等项目写的差不多了，对 egg 了解多点再回头填这个坑吧，先立个 flag。
写到这里 `npm run dev` 没报错的话应该是问题不大，但是要测试是否成功，我们需要在前端也用上 socket。

## vue-socket.io 的使用
这里是没有加上 vuex 的食用方法，后续会补上。
文档在这里[vue-socket.io](https://github.com/MetinSeylan/Vue-Socket.io)。
因为涉及到的文件比较少，这里就不贴目录结构了，这边的项目是 vue-cli 初始化的，所以看客老爷脑补下就好。

1. 下载
```bash
$ npm install --save vue-socket.io
```
2. 连接服务器，以及接收服务端消息
```javascript
// src/main.js
import VueSocketio from 'vue-socket.io';

Vue.use(VueSocketio, 'http://127.0.0.1:7001/');

new Vue({
    el: '#app',
    router,
    template: '<App/>',
    components: { App },
    sockets: {
        connect: function () {
            console.log('socket connected');
        },
        res: function (val) {
            console.log('接收到服务端消息', val);
        }
    }
});
```
`Vue.use()`里面的 url 是你服务器地址。
connect 是 socket.io 默认的事件，看这名字就知道是干啥的了，另外一个 res 是自定义的监听事件，表示监听服务端发送的名为 res 的事件。
3. 向服务端发送消息
```html
<script>
    // ...
    methods: {
        sendMessageToServer: function() {
            this.$socket.emit('chat', '111111111111');
        }
    }
</script>
```
这里我们使用的事件名为 chat，所以服务端会将这条消息交给 chat.js（就是上面服务器端项目里面的文件啦） 这个控制器处理。

## 效果如图
![](https://ws1.sinaimg.cn/large/006tNc79ly1fhu0etwztjg30zg0k4e8j.gif)
开始报的错误是因为后端服务器没打开，连接失败，至于那个 undefined 也可以忽略，这是安装的一个 chrome 插件报的，好像是收趣搞得鬼。
左边窗口是服务端打印的内容，右边自然是客户端的打印啦。

以上。

## 参考链接
- [egg-socket.io](https://github.com/eggjs/egg-socket.io)
- [vue-socket.io](https://github.com/MetinSeylan/Vue-Socket.io)
