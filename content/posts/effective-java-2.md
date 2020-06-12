---
title: "Effective Java 2"
date: 2020-06-05T20:27:43+08:00
draft: false
categories: ["Notes"]
tags: ["Java"]
---

"effective Java"的第二部分总结, 内容主要关于`Object`的可重写的方法(`equals`, `toString`, `hashCode`, `clone`, `finalize`)以及类和接口的使用。

## Object中可重写的方法

### equals实现等价关系

意味着其要满足自反性, 对称性和传递性, 同时还应该保持一致性, 另外对于任何对象, `x.equals(null) == false`. 通常一个`equals`方法满足以下几条:

1. 用`==`检查是否是同一个引用, 如果是, 返回`true`;
2. 用`instanceof`检查是否了正确的类型, 如果不是, 返回`false`;
3. 类型转换;
4. 检查满足逻辑等价的字段.

重写`equals`的同时也需要重写`hashCode`, 可以使用[AutoValue](https://github.com/google/auto/tree/master/value)或者[Lombok](https://github.com/rzwitserloot/lombok)自动生成代码.

### 总是重写toString

### 谨慎的重写clone

一个类只有实现了`Cloneable`, 其`clone`方法才是合法的, 否则会抛出`CloneNotSupportedException`, `clone`的约束是创建并返回一个该对象的拷贝, 也就是说

1. `x.clone() != x -> true`
2. `x.clone().getClass() == x.getClass() -> true`
3. `x.clone().equals(x) -> true`(可选)

实现`clone`方法时, 首先调用`super.clone`, 如果类的字段都是primitive或者是不可变对象, 此时方法的实现就结束了, 否则, 还需要对每一个可变对象的字段递归的调用`clone`(有没法递归调用的情况). 而实际上, 除了数组以外, 其他情况下使用具有copy语义的构造函数或静态工厂方法通常是一个比`clone`更好的选择.

### Comparable实现序关系

可以用`Comparator`来实现`compareTo`.

## 类和接口

### 最小化类和成员的访问权限

- 尽可能的使类和成员不可访问，即采取必要时放宽的策略, 即一开始, 除了定义的API外其他都应该是私有的;
- 顶层的类和接口两个访问级别: *package-private*, *public*;
- 成员四个访问级别: *private*, *package-private*, *protected*(子类和package中的其他类能够访问), *public*;

### public的类使用getter/setter

### Immutable

- 没有修改对象状态的方法(如setter);
- 保证类无法被继承;
- 所有字段为*final*;
- 所有字段为*private*;
- 尽可能的保证类的不可变, 否则限制其可变性;
- 尽可能的声明所有字段为`private final`.

### 偏向组合而不是继承

- 用一个中间的*forwarding class*(包装, 仅仅转发原始类的方法)实现复用(装饰模式);
- 为继承提供良好的设计和文档.

### 偏向接口而不是抽象类

### 接口只用于提供类型

### 嵌套类偏向于静态的

- 如果一个嵌套类不需要访问实例非公有的方法和字段, 使用`static`.

## 总结

这两部分总的看下来除了最小化访问权限外没啥意外的收获.