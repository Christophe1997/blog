---
title: "Effective Java 3"
date: 2020-06-12T19:02:59+08:00
draft: true
categories: ["Notes"]
tags: ["Java"]
---

"effective java"第三部分总结, 内容主要包括泛型.

### 减少unchecked warnning

这部分是我之前写Java代码最头疼的地方, 老是出现*unchecked warnning*, 主要有以下几类

- unchecked cast warnnings
- unchecked method invocation warnnings
- unchecked parameterized vararg type warnnings
- unchecked conversion warnnings

每一个unchecked warnning都有可能导致`ClassCastException`的抛出, 只有再确定类型安全的情况下才可以添加`@SuppressWarnings("unchecked")`, 同时要为其添加注释并最小化作用范围.

### 偏向于使用List而不是Array

- 在Java中Array是协变的, 而泛型本身是不变的
- Array的实现是*reified*(具体化), 即Array保证其在运行时的类型, 而泛型的实现是*erasure*(擦除), 即泛型只能在编译期保证类型, 因此泛型Array在Java中是非法的.