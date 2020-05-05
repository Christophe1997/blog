---
title: "The Smallest Free Number"
date: 2020-05-05T21:01:11+08:00
draft: false
categories: ["Algorithm"]
tags: ["Functional", "Haskell", "Pearls"]
---

**Pearl 1**: 给定一个自然数的有限集X, 计算不属于X的最小自然数. X表示为不包含重复元素的无序列表. 时间复杂度要求$O(n)$.

**Type**: `minfree :: [Int] -> Int`(也可以额外的定义自然数类型, 不过这不是我们的重点)

"Pearls of Functional Algorithm Design"的第一章, 其描述了一个分治的算法和一个基于array的算法, 这里按个人的思路讲解一下基于分治的算法, 基于array的算法具体可以查阅原文. 首先拿到这个问题, 我觉得最直接的想法就是

*Version 1*: `minfree xs = head $ [0..] \\ xs `

然而这和要求的线性时间复杂度不符. 第二个想法就是设计一个fold的函数遍历一遍列表, 这样时间复杂度符合要求. 但是越来越多的边界条件让我意识到思路不对. 看了原文才发现忽略了解题的一个重要条件.

**Fact**: `[0..n]`中的所有自然数不可能都在X(`xs`)中, 其中`n = length xs`.

这也很容易证明, 因为$ n + 1 = length [0..n] > n $, 因此不属于集合X的最小自然数就是`[0..n]`中不属于X的最小自然数. 至此,该问题很容易解决, 只需要一个marked的array来表示`[0,,n]`中的自然数是否在X中即可. 下面描述基于分治的算法, 首先给出一个基本的结论.

**Theorem**: `(as ++ bs) \\ (us ++ vs) == (as \\ us) ++ (bs \\ vs)`, 如果`as \\ vs == as && bs \\ us == bs`.

这显然是符合集合论的. 显然, `[0..n]`可以拆分为两个不想交的集合`[0..b-1]`以及`[b..n]`, 因此`minfree`可以拆分为

```haskell
minfree xs = ([0..b-1] \\ us) ++ ([b..] \\ vs) where
  (us, vs) = partition (< b) xs
```



