---
title: 设计模式（一） 策略模式
date: 2021-04-28 15:42:25
tags: 设计模式
---

## 前言

一直以来对设计模式很是头疼，在当初PHP提升路上学习了几次但依然难以运用之后容易遗忘。虽然平常开发过程中总是会注意一些架构的合理性，方便后续的扩展，但没有理论的支撑实施起来的方案还是问题很多。刚好最近团队在设计模式上一起学习提升，记录学习笔记并归纳在JavaScript和TypeScript中的运用。接下来，首先从策略模式开始

## 正文

先从一个需求出发吧，如果在我的空间发言，需要满足几个条件才能发送

- 普通用户
- 是否通过实名认证
- 等级大于10
- 发言数大于100

从以往来看，大多都会使用`if else`来条件判断是否符合发言条件

```Javascript
function checkCondition(data) {
  if (data.role !== 'user') {
    console.log('不是普通用户');
    return false;
  }
  if (!data.userAuth) {
    console.log('未通过实名认证');
    return false;
  }
  if (data.level < 10) {
    console.log('等级不足');
    return false;
  }
  if (data.comment < 100) {
    console.log('发言数不足');
    return false;
  }
}
```

初步看好像可行，但实际上出现了这些问题

- 后续条件不断增加下去checkCondition里需要不断增加判断
- 函数内逻辑过于复杂，违反单一功能原则
- 条件没法复用
- 违反开闭原则

为了解决这些问题，我们需要引入策略模式

### 策略模式

定义 : 要实现某一个功能，有多种方案可以选择。我们定义策略，把它们一个个封装起来，并且使它们可以相互转换。

使用策略模式修改后

```Javascript
// 策略
var strategies = {
  checkRole: function(value) {
    return value === 'user';
  },
  checkAuth: function(value) {
    return value;
  },
  checkLevel: function(value) {
    return value < 10;
  },
  checkComment: function(value) {
    return value < 100;
  }
};
```

```Javascript
// 调用
function checkOne(name, value) {
  return strategies[name](value)
}
```

这边我们先通过将条件判断的逻辑拆解出不同方法来，解决单一功能原则和复用的问题。后续为了满足开闭原则利用对象映射的灵活性将条件判断的方法置于strategies对象内。后面在调用时只需要通过方法名和参数就能使用相应方法了。

接着进行优化，添加一个验证器

```Javascript
// 校验规则
var Validator = function() {
  this.cache = [];

  // 添加策略事件
  this.add = function(value, method) {
    this.cache.push(function() {
      return strategies[method](value);
    });
  };

  // 检查
  this.check = function() {
    for (let i = 0; i < this.cache.length; i++) {
      let valiFn = this.cache[i];
      var data = valiFn(); // 开始检查
      if (!data) {
        return false;
      }
    }
    return true;
  };
};
```

设置一个用户进行测试

```Javascript
var user = function() {
  var validator = new Validator();
  const data = {
    role: 'user',
    auth: true,
    level: 8,
    comment: 123
  };
  validator.add(data.role, 'checkRole');
  validator.add(data.auth, 'checkAuth');
  validator.add(data.level, 'checkLevel');
  validator.add(data.comment, 'checkComment');
  const result = validator.check();
  return result;
};

```

最重要的是学习之后什么时候使用呢？

- 各判断条件下的策略相互独立且可复用
- 策略内部逻辑相对复杂
- 策略需要灵活组合
