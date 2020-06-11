---
title: "Effective Java 2"
date: 2020-06-05T20:27:43+08:00
draft: false
categories: ["Notes"]
tags: ["Java"]
---

"effective Java"的第二部分, 内容主要关于`Object`的可重写的方法(`equals`, `toString`, `hashCode`, `clone`, `finalize`).

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