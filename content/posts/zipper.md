---
title: "Zipper"
date: 2020-02-27T21:17:41+08:00
draft: true
categories: ["Data Structure"]
tags: ["Zipper", "Functional"]
---

## Zipper

近来想于函数式编程中寻找类似与双向链表的数据结构, 结果找到了Zipper. Zipper中文为拉链, 泛指一类常在函数式编程中使用的聚合数据结构, **其加强了原有的数据结构, 使得能够遍历或更新原有数据结构的任意部分**. Zipper的关键思想是将目前需要处理的部分和不需要处理的部分分开, 同时保存目前不需要的部分, 也可以将其形象的理解为光标. 本文首先介绍了如何在list的基础上构建Zipper, 随后将其扩展到二叉树上, 最后则介绍了原作者论文中的Zipper.