---
title: Front End Review
tags: [前端,JS,Node,React,Vue]
excerpt: 对前端知识的一些总结回顾
index_img: /img/avatar.jpg
date: 2022-10-10 10:00:00
---
# 基础回顾

## JS

1. [js事件循环](https://juejin.cn/post/6844903968292749319)
2. Promise静态方法

   1. **race** 有一个状态敲定就返回
      ```JavaScript
      functionrace(promises) {
      returnnewPromise((re, rj) => {
              promises.forEach((item) => {
      Promise.resolve(item).then(re, rj)
              })
          })
      }
      ```
   2. **all **所有promise都返回成功或者有一个失败
      ```JavaScript
      functionall(arr) {
      let count = 0
      const res = []
      returnnewPromise((re, rj) => {
              arr.forEach((item, i) => {
                  count++
      Promise.resolve(item).then(val => {
                      res[i] = val
      if (res.length === count) re(res)
                  }, rj)
              });
          })
      }
      ```
   3. allSettled所有状态都敲定
      ```JavaScript
      functionallSettled(promises) {
      let count = 0
      const res = []
      returnnewPromise((re, rj) => {
              arr.forEach((item, i) => {
      Promise.resolve(item).then(val => {
                      count++
                      res[i] = { status: 'fulfilled', val: val }
      if (promises.length === count) re(res)
                  }, (err) => {
                      count++
                      res[i] = { status: 'rejected', err: err }
      if (promises.length === count) rj(res)
                  })
              });
          })
      }
      ```
   4. **any**  返回第一个结果是成功的
      ```JavaScript
      functionany(promises) {
      let arr = [],
              count = 0
      returnnewPromise((resolve, reject) => {
              promises.forEach((item, i) => {
      Promise.resolve(item).then(resolve, err => {
                      arr[i] = { status: 'rejected', val: err }
                      count += 1
      if (count === promises.length) reject(newError('没有promise成功'))
                  })
              })
          })
      }
      ```
3. 链式调用

   1. JQuery形式  把当前的实例返回出去
      ```JavaScript
      var user = function (name, age) {
          this.name = name;
          this.age = age;
      };
      user.prototype.getName = function () {
          console.log("姓名是", this.name);
          return this;
      };
      user.prototype.getAge = function () {
          console.log("年龄是", this.age);
          return this;
      };
      var user1 = new user("zjf", 22);
      user1.getAge().getName()Ï
      ```
   2. Koa形式 中间件 通过next调用下一个
      ```JavaScript
       compose(middlewares) {
              return function (ctx) {
                  return dispatch(0);
                  function dispatch(idx) {
                      const fn = middlewares[idx];
                      if (!fn) {
                          return Promise.resolve();
                      }
                      return Promise.resolve(
                          fn(ctx, function next() {
                              return dispatch(idx + 1);
                          })
                      );
                  }
              };
          }
      ```
4. **instanceof**

   ```JavaScript
   function new_instance_of(leftVaule, rightVaule) {
       let rightProto = rightVaule.prototype; // 取右表达式的 prototype 值
       leftVaule = leftVaule.__proto__; // 取左表达式的__proto__值
       while (true) {
           if (leftVaule === null) {
               return false;
           }
           if (leftVaule === rightProto) {
               return true;
           }
           leftVaule = leftVaule.__proto__
       }
   }
   ```
5. **new**

   ```JavaScript
   function newFn(fn, ...args) {
       let obj = new Object();
       obj.__proto__ = Object.create(fn.prototype);
       const res = fn.apply(obj, args)
       return typeof res === "object" ? res : obj
   }
   ```
6. call、bind、apply
7. Event Bus
8. **不可变对象**

   1. seal 会使对象的所有属性的**configurable为**false，但是仍然可以修改现有的属性，因为**writable**为true。
      ```JavaScript
      const obj = { author: "DevPoint" };
      console.log(Object.getOwnPropertyDescriptors(obj));
      /*
      {
        author: {
          value: 'DevPoint',
          writable: true,
          enumerable: true,
         configurable: true
        }
      }
      */
      Object.seal(obj)
      console.log(Object.getOwnPropertyDescriptors(obj));
      /*
      {
        author: {
          value: 'DevPoint',
          writable: true,
          enumerable: true,
          configurable: false
        }
      }
      */
      ```
   2. freeze 同理，使得**writable，configurable**均为false，使得其不可修改和扩展。
      ```JavaScript
      const obj = { author: "DevPoint" };
      Object.freeze(obj)
      console.log(Object.getOwnPropertyDescriptors(obj));
      /*
      {
        author: {
          value: 'DevPoint',
          writable: false,
          enumerable: true,
          configurable: false
        }
      }
      */
      ```
   3. 缺点 二者均只能监控第一层，对于有多层嵌套的对象，可以采用**递归**的方式进行封闭 / 冻结。
      ```JavaScript
      const obj = {
          author: {
              name: "author",
          },
      };
      Object.freeze(obj);
      console.log(Object.getOwnPropertyDescriptors(obj.author));
      /*
      {
        name: {
          value: 'author',
         writable: true,
      enumerable: true,
          configurable: true
        }
      }
      */
      // deepSeal同理
      function deepFreeze(object) {
          const propsNames = Object.getOwnPropertyNames(object);
          for (const name of propsNames) {
              const value = object[name];
              if (value && typeof value === "object") {
                  deepFreeze(value);
              }
          }
          return Object.freeze(object);
      }
      ```
9. [闭包](https://segmentfault.com/a/1190000039042550)

   ```JavaScript
   function debounce(fn, wait) {
       let timer = null;
       return function () {
           clearTimeout(timer);
           timer = setTimeout(() => {
               fn.call(this, ...arguments);
           }, wait);
       };
   }
   function throttle(fn, delay = 500) {
       let timer = null;
       return function (...args) {
           if (!timer) {
               timer = setTimeout(() => {
                   fn.apply(this, args);
                   timer = null;
               }, delay);
           }
       };
   }
   ```
10. defer async区别

    1. async js脚本的加载不会阻塞渲染，但是当js加载完成会立即运行，此时会阻塞html的渲染，而且多个async脚本的执行顺序不一致
    2. defer与async类似，但是它会在html加载完成后才执行，且保证执行顺序。
11. 数组的filter、every、flat

    1. filter

       ```js
       Array.prototype.myFilter = function (fn, thisValue) {
           if (typeof fn !== "function") {
               throw new Error(`${fn} 不是一个函数`);
           }
           if ([null, undefined].includes(this)) {
               throw new Error(`this 是null 或者 undefined`);
           }
           const arr = Object(this);
           const filterArr = []; // 没有符合条件的返回空数组
           for (let i = 0; i < arr.length; i++) {
               const res = fn.call(thisValue, arr[i], i, arr);
               if (res) {
                   filterArr.push(arr[i]);
               }
           }
           return filterArr;
       };
       ```
    2. every**如果所有元素都通过检测返回 true，否则返回 false**

       ```js
       Array.prototype.myEvery = function (fn, thisValue) {
           if (typeof fn !== "function") {
               throw new Error(`${fn} 不是一个函数`);
           }
           if ([null, undefined].includes(this)) {
               throw new Error(`this 是null 或者 undefined`);
           }
           const arr = Object(this);
           let flag = true;
           for (let i = 0; i < arr.length; i++) {
               const res = fn.call(thisValue, arr[i], i, arr);
               if (!res) {
                   return false;
               }
           }
           return flag;
       };
       ```
    3. flat 数组扁平化

       ```js
        Array.prototype.Myflat = function (dep = 1) {
           let res = [];
           this.forEach((item, index) => {
               if (Array.isArray(item) && dep > 0) {
                   dep--;
                   res = res.concat(item.Myflat(dep));
               } else {
                   res.push(item);
               }
           });
           return res;
       };
       ```
12. [Map和Set,Object的区别](https://juejin.cn/post/7077430645844082702)
13. JS场景手写题目

    1. 数组转树
    2. 数组洗牌
    3. Promise并发

## CSS

1. **display:none opacity:0 以及visibility:hidden区别**

   1. display不会被子元素继承 但是父元素不在了，子元素也就不在了
   2. opacity可以被子元素继承，但是不能设置子元素opacity:1使子元素显示
   3. visibility可以被子元素继承，子元素设置visibility:visibile显示
2. **flex:1**

   1. flex-grow 定义放大比例 默认为0 不放大
   2. flex-shrink 定义缩小比例 默认为1  空间不足就缩小
   3. flex-basis 定义分配多余空间之前，项目占据的主轴空间 默认auto 即元素本来的大小| **语法** | **等值** |
      | -------------- | -------------- |
      | flex: initial  | flex: 0 1 auto |
      | flex: 0        | flex: 0 1 0%   |
      | flex: none     | flex: 0 0 auto |
      | flex: 1        | flex: 1 1 0%   |
      | flex: auto     | flex: 1 1 auto |
3. 0.5px的线  transform: scaleY(0.5);
4. [BFC](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Block_formatting_context)

   1. [面试官：请说说什么是BFC？大白话讲清楚](https://juejin.cn/post/6950082193632788493)
   2. 三栏布局
      1. 浮动方式
         ```HTML
         <div class="left"></div>
         <div class="right"></div>
         <div class="main"></div>

          <style>
             .left {
                 width: 100px;
                 height: 150px;
                 float: left;
                 background: #f66;
             }
             .right {
                 width: 100px;
                 height: 150px;
                 float: right;
                 background: #f66;
             }
             .main {
                 height: 200px;
                 overflow: hidden;
                 margin: 0 100px;
                 background: #fcc;
             }
         </style>
         ```
      2. flex布局
      3. 绝对定位
5. 选择器权重 !important > 内联 > id > 类 伪类 属性 > 标签 伪元素 > 通配符 兄弟选择器
6. [z-index](https://juejin.cn/post/6844903667175260174)
7. 浮动的产生、浮动带来的影响、消除影响的方式

## 智力题

* 赛马
* 变色龙

## H5

# 前端框架

## Vue2

1. [前端框架用vue还是react？清晰对比两者差异](https://juejin.cn/post/6844903974437388295#heading-0)

## Vue3

## React

# NodeJS

## Express

## Koa

## Egg

# 跨端

## 小程序

## React Native

## Flutter

## Weex

## Electron

# 工程化

## Webpack

## Vite

# 浏览器相关

## 缓存

## 存储

## 协议

## 其他

1. [前端网页如何打开一个PC本地应用](https://juejin.cn/post/6844903989155217421#heading-15)
2. [手写echarts图表](https://juejin.cn/post/6950684708443258894)

# CI/CD相关

## Docker

## SCM
