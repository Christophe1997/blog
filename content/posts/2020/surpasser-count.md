---
title: "Surpasser Count"
date: 2020-05-06T20:06:38+08:00
draft: false
categories: ["Algorithm"]
tags: ["Functional", "Haskell", "Pearls"]
---

**Pearl 2**: 给定一个长度大于1的列表, 计算其元素的最大surpasser count, 要求算法复杂度$O(n log n)$.
**Type**: `msc: Ord a => [a] -> Int`

"Pearls of functional algorithm design"的第二章, 我们先来看surpasser的定义

**Definition surpasser**: 称列表中$X[j]$是$X[i]$的surpasser, 如果$X[i] < X[j]$且$i < j$.

因此一个元素的surpasser count就是其surpasser的数目.

同样, 一个naive的实现很容易:

```haskell
msc :: Ord a => [a] -> Int
msc xs = maximum [scount z zs | z:zs <- tails xs]
scount :: Ord a => a -> [a] -> Int
scount x xs = length $ filter (> x) xs
```

同时也很容易看到, 这个实现的时间复杂度是$O(n^2)$, 不符合要求的$O(n log n)$. 为了达到$ O(n log n)$的时间复杂度, 我们希望有个函数`f`能够递归的处理`xs = us ++ vs`, 并且存在一个线性复杂度的函数`join`, 使得`f xs = join (f us) (f vs)`, 这样整体的复杂度满足$T(n)=2 T(n/2)+O(n)=O(n log n)$. 原文中, 作者利用分治的思想通过一步步地推导获得了线性时间的`join`, 这里也仅仅是类似于复读的"再解释".

这里我们从所有surpasser count的表开始, 即`table xs = [(z, scount z zs) | z:zs <- tails xs]`, 这样的话`msc = maximum . map snd . table`. 如果我们能够找到一个线性复杂度的`join`, 使得`table (xs ++ ys) = join (table xs) (table ys)`, 那么就能够得到满足时间复杂度条件的算法. 首先, 给出一个非常直接的性质

**Theorem**: `tails (xs ++ ys) == map (++ ys) (tails xs) ++ tails ys`

利用上述性质, 我们可以进行简单的推导:

```haskell
table (xs ++ ys)
=> [(z, scount z zs) | z:zs <- tails (xs ++ ys)]
=> [(z, scount z zs) | z:zs <- map (++ ys) (tails xs) ++ tails ys]
-- ++的分配律
=> [(z, scount z $ zs ++ ys) | z:zs <- tails xs] ++ 
   [(z, scount z zs) | z:zs <- tails ys]
-- scount z $ zs ++ ys == scount z zs + scount z ys
=> [(z, scount z zs + scount z ys) | z:zs <- tails xs] ++ 
   [(z, scount z zs) | z:zs <- tails ys]
=> [(z, c + scount z ys) | (z, c) <- table xs] ++ table ys
-- ys == map fst $ table ys
=> [(z, c + scount z (map fst $ table ys)) | (z, c) <- table xs] ++ table ys
=>
join txs tys = [(z, c + tcount z tys) | (z, c) <- txs] ++ tys
tcount z tys = scount z $ map fst tys
```

这个也很容易理解, 我们合并`txs = table xs`和`tys = table ys`时, 最简单的就是对于`txs`的每一个`(z, c)`, 额外增加一个`z`在`ys`中的count, 然而我们知道这并不是一个线性复杂度的`join`, 而是一个$O(n^2)$的算法. 从上面可以看到, 作者多此一举的引入了一个`tcount`, 这表明了可以优化`tcount`, 如果`tys`是一个排序好的列表, 那么

```haskell
tcount z tys
=> length $ filter (> z) (map fst tys)
-- filter p . map f == map f . filter (p . f)
=> length (map fst $ filter ((> z) . fst) tys)
-- length . map f == length
=> length $ filter ((> z) . fst) tys
-- tys是一个递增的列表
=> length $ dropWhile ((<= z) . fst) tys
```

上面的推导表明, 如果我们在构建`table`的时候使其保持有序, 那么可以获得更好的性能. 对于排序好的俩个列表, 我们可以用一个线性复杂度的`merge`合并两个有序列表, 这样```join txs tys = [(z, c + tcount z tys) | (z, c) <- txs] `merge` tys```. 这启发我们可以设计一个排序的`join`. 首先最基本的条件很容易得到`join [] tys = tys, join txs [] = txs`, 对于递归的部分, 即`join txs @ ((x, c): txs') tys @ ((y, d):tys')`:

```haskell
join txs @ ((x, c): txs') tys @ ((y, d):tys')
=> ((x, c + tcount x tys):[(x, c + tcount x tys) | (x, c) <- txs]) `merge` tys
-- if x < y then tcount x tys == length tys
1=> (x, c + length tys):join txs' tys
-- if x == y then tcount x tys == tcount x tys' == d
2=> (y, d):join txs tys'
-- if x < y then same as x == y
3=> (y, d):join txs tys'
```

至此, 我们就可以得到优化后的`join`:

*Final Solution*:

```haskell
msc :: Ord a => [a] -> Int
msc = maximum . map snd . table

table :: Ord a => [a] -> [(a, Int)]
table [x] = [(x, 0)]
table xs = join (m - n) (table ys) (table zs)
           where m        = length xs
                 n        = m `div` 2
                 (ys, zs) = splitAt n xs

join :: Ord a => Int -> [(a, Int)] -> [(a, Int)] -> [(a, Int)]
join _ [] tys = tys
join _ txs [] = txs
join n txs@((x, c):txs') tys@((y, d):tys')
  | x < y     = (x, c + n) : join n txs' tys
  | otherwise = (y, d) : join (n - 1) txs tys'
```

注意到, 我们这里没法使用降序的`join`来进一步优化, 因为`tcount x tys == length tys`依赖于`x < y`.

## 后记

这个pearl看下来, 让我更清晰的感受到了一个优化的解的实现过程. 我拿到这个$O(n log n)$的复杂度要求, 虽然能够想到要通过切分, 递归的merge. 但我的思考过程并不算很连贯, 有可能想出类似的解, 但估计耗费的时间会很长. 还是一样, 最关键是要从基础的解开始一步步优化. 值得一提的是, 这个问题的array版本存在一个使用二分查找的[解](https://www.sciencedirect.com/science/article/pii/0167642388900536), 不过由于使用的符号太奇怪了, 我就没有仔细查看.