---
title: 设计模式（一） 工厂方法模式
date: 2021-05-20 10:21:10
tags:
---

## 正文

```typescript
interface Pizza {
    Prepare: ()=> String,
    Bake: ()=> String,
    Cut: () => String,
    Box: () => String 
}
abstract class Pizza(){
    abstract Prepare():String;
    abstract Bake():String;
    abstract Cut():String;
    abstract Box():String;
}
class PizzaStore() {
    let name;
    OrderPizza: function(){
        switch(type)
    }
}
```
