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

*Base Solution*: `minfree xs = head $ [0..] \\ xs `

然而这和要求的线性时间复杂度不符. 第二个想法就是设计一个fold的函数遍历一遍列表, 这样时间复杂度符合要求. 但是越来越多的边界条件让我意识到思路不对. 看了原文才发现忽略了解题的一个重要条件.

**Fact**: `[0..n]`中的所有自然数不可能都在X(`xs`)中, 其中`n = length xs`.

这也很容易证明, 因为$ n + 1 = length\ [0..n] > n $, 因此不属于集合X的最小自然数就是`[0..n]`中不属于X的最小自然数. 至此,该问题很容易解决, 只需要一个marked的array来表示`[0,,n]`中的自然数是否在X中即可. 下面描述基于分治的算法, 首先给出一个基本的结论.

**Theorem**: `(as ++ bs) \\ (us ++ vs) == (as \\ us) ++ (bs \\ vs)`, 如果`as \\ vs == as && bs \\ us == bs`.

这显然是符合集合论的. 显然, `[0..n]`可以拆分为两个不想交的集合`[0..b-1]`以及`[b..n]`, 因此`[0..b]`可以拆分为

```haskell
([0..b-1] \\ us) ++ ([b..] \\ vs) where
  (us, vs) = partition (< b) xs
```

而`minfree`则可以改写为

```haskell
minfree xs = if null $ [0..b-1] \\ us
             then head $ [b..] \\ vs
             else head $ [0..b-1] \\ us
             where (us, vs) = partition (< b) xs
                   b        = 
```

很容易发现`null $ [0..b-1] \\ us`等价于`length us == b`, 后者更加高效. 同时, 我们也可以进一步的抽象`minfree`, 因为我们在上面限制了从0开始:

```haskell
minfrom :: Int -> [Int] -> Int
minfrom a xs = head $ [a..] \\ xs
```

至此, 我们的`minfree`可以改为:

```haskell
minfree = minfrom 0

minfrom :: Int -> [Int] -> Int
minfrom a xs | null xs            = a
             | length us == b - a = minfrom b vs
             | otherwise          = minfrom a us
               where (us, vs) = partition (< b) xs
                     b        = 
```

接下来的问题是`b`应该是多少, 显然`b`可以是$(a, n=length\ xs)$中的任意一个自然数, 不过`b`的选择应该使得`us`和`vs`的长度尽可能的小, 否则的会导致算法在最坏情况下开销的增加. 因此比较理想的取值是```b = a + 1 + n `div` 2```, 这样如果`length us < b - a`的, 那么```length us < b - a  < n `div` 2 + 1 <= n `div` 2```, 而如果`length us == b - a`, 那么```length vs = n - b + a = n - n `div` 2 - 1 <= n `div` 2```. 此时可以看到算法的复杂度是$O(n)$的. 在`minfree`的最终版本, 为了避免重复计算, 我们可以传入`(length xs, xs)`.

*Final Solution*:

```haskell
minfree :: [Int] -> Int
minfree xs = minfrom 0 (length xs, xs)

minfrom :: Int -> (Int, [Int]) -> Int
minfrom a (n, xs) | n == 0     = a
                  | m == b - a = minfrom b (n - m, vs)
                  | otherwise  = minfrom a (m, us)
                    where (us, vs) = partition (< b) xs
                          b        = a + 1 + n `div` 2
                          m        = length us
```

## 后记

整个pearl看下来给我最大的感受就是首先给出一个比较naive的解, 然后利用分治的思想一步步的分解问题并优化解. 同时, 虽然函数式的算法总是会比相应的命令式的算法差一个对数阶(因为函数式的算法中无法保证array的更新是常数级的, 通常是一个对数级的), 但是在这个pearl上, 作者通过不断的迭代算法缩小了这个差距.