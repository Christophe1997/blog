---
title: "Details in Data.List"
date: 2020-06-17T20:36:04+08:00
draft: true
categories: ["Data Structure"]
tags: ["Haskell", "List"]
---

起源于Codewars上的[一道题](https://www.codewars.com/kata/521c2db8ddc89b9b7a0000c1), 求一个方阵从外到内的一个螺旋序列:

```
*->*->*
      |
*->*  *
|     |
*<-*<-*
```

我想到的是做一个递归的算法:

```haskell
strip :: [a] -> ([a], [a], [a])
strip ls = (start, mid, end)
           where (start, rest) = splitAt 1 ls
                 (mid, end)    = splitAt (size - 2) rest
                 size          = length ls 

snail :: [[Int]] -> [Int]
snail [] = []
snail [xs] = xs
snail ls = top ++ right ++ reverse bottom ++ reverse left ++ snail rest
           where ([top], mid, [bottom])         = strip ls
                 (left, rest, right)            = foldr stripAndCons ([], [], []) mid
                 stripAndCons ls (s, m, e)      = (x0 : s, xs : m, x1 : e)
                                                  where ([x0], xs, [x1]) = strip ls 
```

最后的实现自我感觉还算满意, 直到我看到了最高赞的答案:

```haskell
snail :: [[Int]] -> [Int]
snail [] = []
snail (xs:xss) = xs ++ (snail . reverse . transpose) xss
```

`transpose :: [[a]] -> [[a]]`是`Data.List`的一个函数, 实现了转置. 虽然我没有想到利用转置的解并不是因为不知道`transpose`(而是没有往转置那块想), 但这个也一定程度上促使我想要仔细考察下`Data.List`. 另一方面Haskell的`Data.List`模块是我用过的对list的各种操作支持最多最好的一个, 每当我在其他语言用List时就会怀念Haskell对List的支持. 所以, 有了这一篇, 本文主要考察`Data.List`中的一些函数, 更确切的来说是"base"的隐藏模块`Data.OldList`中的函数.

### transpose

就先把`transpose`放在第一个, `transpose`是将参数的行和列转置, 实现如下:

```
transpose               :: [[a]] -> [[a]]
transpose []             = []
transpose ([]   : xss)   = transpose xss
transpose ((x:xs) : xss) = (x : [h | (h:_) <- xss]) : transpose (xs : [ t | (_:t) <- xss])
```

值得注意的是`transpose ([] : xss) = transpose xss`这一行, 可以看到当某一行比较短的时候会发生这种情况, 策略就是忽略它. 这样的好处是扩大了`transpose`的输入域, 即使不是方阵也能转置.

