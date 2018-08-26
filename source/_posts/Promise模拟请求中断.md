---
title: Promise模拟请求中断
date: 2018-08-26 22:05:55
tags:
    - Javascript
---

# Promise 模拟请求中断
#语言/JavaScript


![](https://ws1.sinaimg.cn/large/006tNbRwly1funghu18coj30sf0jjmzk.jpg)


## 背景
我司 app 一级入口的个人中心页面采用 Hybrid 开发方式，每次进入页面都会请求用户最新数据，当快速在「个人中心」页与其他页面切换时，可能会发生重复的网络请求造成列表数据混乱。具体表现为当第一个网络请求发出，未返回时，切到其他页面，再快速切回来，就会发第二个网络请求。后果是分页的第一页数据重复。

<!--more-->

## 解决方案
  1. 后端在每页返回的数据中标明当前数据是第几页，前端在使用前判断如果是第一页数据就直接使用当前数据替换原有列表数据。
  2. 前端主动取消重复的网络请求。

## 实践
  1. 后端生病了，无法及时响应需求😂。
  2. 我们的 Hybrid 方案是公司前端与客户端开发 SDK，作为业务前端，发现现有的 fetch 方法其实是客户端提供原生的网络请求方法，经过 JS-SDK 封装后暴露出来的方法，目前 SDK 不支持请求中断，所以最后方案是我这边使用 Promise hack 一下。
  
## code
> The Promise.race(iterable) method returns a promise that resolves or rejects as soon as one of the promises in the iterable resolves or rejects, with the value or reason from that promise.
Promise.race 方法将多个 Promise 对象组成的迭代器包装成一个 Promise 对象，当迭代器中的某个 Promise resolve 或者 reject 时，该对象就会 resolve 或者 reject。
race 就是竞赛的意思，谁速度快（率先返resolve 或者 reject）谁就赢，整个迭代器其他的 Promise 的状态我们就不关心了。

基于此，封装一个包装 fetch 的方法如下
```javascript
export function fetchWithAbort(fetchPromise) {
  let abort = null
  const abortPromise = new Promise((resolve, reject) => {
    abort = () => {
      reject('abort')
      console.log('=== fetchWithAbort abort ===')
    }
  })
  let promiseWithAbort = Promise.race([fetchPromise, abortPromise])
  promiseWithAbort.abort = abort
  return promiseWithAbort
}
```
使用(示例代码是在 .vue 中使用)
```javascript
if (this.productionFetchPromise && this.productionFetchPromise.abort) {
  this.productionFetchPromise.abort()
}
this.productionFetchPromise = fetchWithAbort(getProductionHttp(this.page, PAGE_SIZE))
this.productionFetchPromise.then(res => {
  console.log('=== fetch success ==='. res)
}).catch(err => {
  if (err === 'abort') {
    console.log('=== fetch abort ===', err)
    return
  }
  console.log('=== fetch err ===', err)
})
```
在每次发起网络请求前，先 abort 掉之前的 fetch，其实并没有真的中断请求，只是让之前的请求 Promise reject 掉，使我们不必关心之前的网络请求是成功还是失败。

## 联想
当 abort 写出来了，其实自定义超时的方法也就出来了，甚至更加简单
```javascript
function fetchWithTimeout(fetchPromise) {
  const timePromise = new Promise((resolve, reject) => {
    setTimeout(() => {
      reject('timeout')
    }, 5000)
  })
  return new Promise.race([fetchPromise, timePromise])
}
```

## 问题
使用这种方案虽然解决了业务需求，但是我这边一直都是使用 async await 处理网络事件，这种方案让这边的代码风格变得不一致，虽然工作量不大，但是修改这的网络请求相关代码还是费了点时间，之后可以考虑下可否保持原先的 async await 代码，无痛切换到 fetchWithAbort。

以上。

## 参考文献
  1. [Promise.race() - JavaScript | MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/race)
  2. [Fetch进阶指南 | louis blog](http://louiszhai.github.io/2016/11/02/fetch/)
