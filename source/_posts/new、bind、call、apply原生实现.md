---
title: new、bind、call、apply原生实现
date: 2019-06-14 16:08:32
tags:
---

### 面试期间最常见的几种手写原生方法

- new
- bind
- call
- apply

1. `new`事件过程
	
> - 创建一个新对象
> - this指向更改
> - 继承父类原型
> - 返回子类

``````js
// new事件过程
const newFunc = (parent) => {
  if (!(parent instanceof Function)) { return {}; }
  // 获取其他参数
  let arg = Array.prototype.slice.call(arguments, 1);
  // 1、创建一个新对象
  let child = Object.create({});
  // 2、this指向更改
  let result = parent.apply(child, arg);
  // 3、继承父类原型
  child.__proto__ = parent.prototype;
  // 4、返回子类，区分返回类型是值类型还是引用类型
  return (typeof result === 'object') ? result : child;
}

``````

2. `bind`事件过程

> - this指向明确
> - this指向更改
> - 原型关系维护
> - 返回函数

``````js
// bind事件过程
Function.prototype.bind = function(thisArg) {
  if (typeof this !== 'function') {
    throw new Error('需在Function上进行');
  }
  var _self = this;
  // 获取额外参数
  var arg = Array.prototype.slice.call(arguments, 1);
  // 定义一个空函数
  var fnNop = function () {}
  var fnBound = function () {
    // 区分new还是直接引用
    // new创建this指向，其他指向传参对象
    // this instanceof fBound === true时,说明返回的fBound被当做new的构造函数调用
    var _this = this instanceof fnBound ? this : thisArg;
    return _self.apply(_this, arg.concat([].slice.call(arguments)));
  }
  // 维护原型关系
  if (this.prototype) {
    fnNop.prototype = this.prototype;
  }
  // 因为如果直接fnBound.prototype = this.prototype时，
  // (new fnBound).__proto__.constructor === this
  // 造成原型链混乱
  // 使用下面装换后
  // (new fnBound).__proto__.constructor === fnBound
  // 下行的代码使fBound.prototype是fNOP的实例,因此
  // 返回的fBound若作为new的构造函数,new生成的新对象作为this传入fBound,新对象的__proto__就是fNOP的实例
  fnBound.prototype = new fnNop();

  return fnBound;
}

``````

3. `call`事件过程

> - this指向改变
> - 函数执行了

> 简单实现步骤：
> 1. 将函数设为对象的属性
> 2. 执行该函数
> 3. 删除该函数
>
> 第一步
> foo.fn = bar
> 第二步
> foo.fn()
> 第三步
> delete foo.fn

``````js
// call事件过程
Function.prototype.call = function(oThis) {
  if (typeof this !== 'function') {
    throw new Error('需在Function上进行');
  }
  oThis = oThis || window;
  oThis.fn = this;

  // 获取参数
  var args = [];
  // 因为arguments是类数组对象，所以可以用for循环
  for (var i = 1, len = arguments.length; i < len; i++) {
    // 因为下面要是用eval来执行函数，所以需要这样写
    // ['arguments[1]', 'arguments[2]', ...]
    args.push('arguments[' + i + ']');
  }
  // 这里 args 会自动调用 Array.toString() 这个方法=》'arguments[1],arguments[2],arguments[3]...'
  // 在eval中会执行arguments[1]来获取值
  // 在执行函数
  var result = eval('oThis.fn(' + args + ')');

  delete oThis.fn;

  return result;
}

``````

4. `apply`事件过程

`apply 的实现跟 call 类似`

``````js
// apply 的实现跟 call 类似
Function.prototype.apply = function(oThis, arr) {
  if (typeof this !== 'function') {
    throw new Error('需在Function上进行');
  }

  oThis = oThis || window;
  oThis.fn = this;

  // 获取参数
  var args = [];
  for (var i = 0, len = (arr || []).length; i < len; i++) {
    // 因为下面要是用eval来执行函数，所以需要这样写
    // ['arr[0]', 'arr[1]', ...]
    args.push('arr[' + i + ']');
  }
  // 这里 args 会自动调用 Array.toString() 这个方法=》'arr[0],arr[1],arr[2]...'
  // 在eval中会执行arr[0]来获取值
  // 在执行函数
  var result = eval('oThis.fn(' + args + ')');

  delete oThis.fn;

  return result;
}

``````