---
title: "Memoization in Haskell"
date: 2020-06-19T21:39:35+08:00
draft: false
categories: ["Optimization"]
showToc: true
tags: ["Haskell", "Memoization", "DP"]
---

Memoization是动态规划(*Dynamic Programming*)中自顶向下处理问题采用的策略, 其基本想法是通过将子问题的解保存起来避免重复计算来优化算法. 这个概念本身很简单, 在其他有明显mutable语义的语言中, 实现起来也非常简单. 但是在Haskell中问题就变的复杂了不少, 对于一个原始的函数`f :: a -> b`你如果要用ref, 比如说IORef, 你必须要把它放到IO monad中, 你的memoize函数就变成了`... -> IO (a -> b)`. 我们希望是能够找到一个`memoize :: ... -> (a -> b)`, 这样memoize之后得到的和原函数类型是一致的. 为了讨论的方便, 我们主要关注两个例子的memoization, 一个是经典的Fibonacci数列:

```haskell
fib :: Int -> Integer
fib 0 = 0
fib 1 = 1
fib n = fib (n - 2) + fib (n - 1)
```

另一个则是动态规划(自底向上)中典型的最小编辑距离的问题, 所谓的最小编辑距离就是一个字符串通过增加, 删除, 替换的操作得到另一个字符串所需要的操作次数:

```haskell
minEditDist :: String -> String -> Int
minEditDist []     []     = 0
minEditDist s      []     = length s
minEditDist []     s      = length s
minEditDist (x:xs) (y:ys) | x == y    = minEditDist xs ys
                           | otherwise = 1 + minimum [minEditDist xs ys, minEditDist xs (y:ys), minEditDist (x:xs) ys]
```

### Memoizing with specific problem

首先来看`fib`的问题, [wiki](https://wiki.haskell.org/Memoization)给出了一个非常elegant的解(就`fib`本身而言, 还有更经典的解, `fib = (fibs !!) where fibs = 0 : 1 : zipWith (+) fibs (tail fibs)`):

```haskell
import Data.Function (fix)

memoize :: (Int -> a) -> (Int -> a)
memoize f = (map f [0..] !!)

fib :: (Int -> Integer) -> Int -> Integer
fib f 0 = 1
fib f 1 = 1
fib f n = f (n - 1) + f (n - 2)

fibMemo :: Int -> Integer
fibMemo = fix (memoize . fib)
```

虽然这个`memoize`和我们想要的`(a -> b) -> a -> b`有点差距, 但仍然值得分析一下. 首先来看`fix`, `fix`的定义很简单:

```haskell
fix :: (a -> a) -> a
fix f = let x = f x in x
```

关于`fix`的详细解释这里略去, 简单而言, 可以将`fix`理解为一个构建递归的函数. 例如, `fix (1:)`按定义展开后就是`1:(1:(1:(...)))`, 很容易看到是一个元素为1的无限列表. 这里的`fibMemo = fix (memoize . fib)`同样我们按定义展开:

```haskell
fibMemo = fix (memoize . fib)
        -- fix定义
        = let x = (memoize . fib) x in x
        = (memoize . fib) fibMemo
        = memoize (fib fibMemo)
        -- memoize定义
        = (map (fib fibMemo) [0..] !!)

-- 等价于
fibMemo = (map fib [0..] !!) where
  fib 0 = 0
  fib 1 = 1
  fib n = fibMemo (n - 2) + fibMemo (n - 1)
```

这种memoization实现利用了Haskell的laziness, `fibMemo`变成了从一个无限的列表里面取值, 我们已经构建好了每一个元素的表达式, 在需要的时候计算, 这样那些已经计算过的元素就保存在列表里面. 更详细的讲, 我们在定义完fibMemo时其结构为:

```haskell
fibMemo = ([0, 1,
            fibMemo 0 + fibMemo 1,
            fibMemo 1 + fibMemo 2..] !!)
```

在调用`fibMemo 3`之后其结构变为:

```haskell
fibMemo = ([0, 1, 1, 2, fib 2 + fib 3..] !!)
```

可以看到`fibMemo 2`的结果已经被保存了, 这就实现了memoization.

我们再来看最小编辑距离的问题, 我们显然没法把fib中的`memoize`直接拿过来. 因为在这个问题上, 我们希望保存的是任意两个子串的最小编辑距离, 从之前fib的memoization借鉴, 开始我们的第一次尝试:

```haskell
minEditDistMemo :: String -> String -> Int
minEditDistMemo s1 s2 = lookupS s1 s2
  where lookupS x1 x2 = maybe undefined id $ lookup (x1, x2) ds
        ds            = map g [(x1, x2) | x1 <- tails s1, x2 <- tails s2]
        g (s1, s2)    = ((s1, s2), f s1 s2)
        f [] []       = 0
        f s []        = length s
        f [] s        = length s
        f (x:xs) (y:ys) | x == y    = minEditDistMemo xs ys
                        | otherwise = 1 + minimum [minEditDistMemo xs ys, minEditDistMemo xs (y:ys), minEditDistMemo (x:xs) ys]
```

可以看到, 每次递归调用`minEditDistMemo`, 它都会构建一个新的ds, 而这是有问题的. 当然这也很容易解决, 只要把每次递归调用`minEditDistMemo`的地方换成`lookupS`就行:

```haskell
minEditDistMemo :: String -> String -> Int
minEditDistMemo s1 s2 = lookupS s1 s2
  where lookupS x1 x2 = maybe undefined id $ lookup (x1, x2) ds
        ds            = map g [(x1, x2) | x1 <- tails s1, x2 <- tails s2]
        g (s1, s2)    = ((s1, s2), f s1 s2)
        f []     []   = 0
        f s      []   = length s
        f []     s    = length s
        f (x:xs) (y:ys) | x == y    = lookupS xs ys
                        | otherwise =1 + minimum [lookupS xs ys, lookupS xs (y:ys), lookupS (x:xs) ys]
```

### generic memoization

通过上面的分析, 可以看到, 我们总是可以根据特定的问题构建特定的数据结构来实现memoization. 也就是说, 对于任意的一个函数`f :: a -> b`(如果f有多个参数, 可以先uncurry), 我们希望能够用一个数据结构来保存计算结果, 也就是`(a, b)`, 显然, Map就是最理想的数据结构. 问题是Haskell的Map是immutable, 我们没法像imperative programming那样方便的修改, 这个时候就需要用到State, State能够帮助我们解决共享状态的问题(以下实现来源于[Memoizing function in Haskell](http://www.nadineloveshenry.com/haskell/memofib.html)):

```haskell
import qualified Data.Map as M
import Control.Monad.State

type MemoState a b = State (M.Map a b) b

memorize :: Ord a => ((a -> MemoState a b) -> (a -> MemoState a b)) -> a -> b
memorize t x = evalState (f x) M.empty where
  f x = get >>= \m -> maybe (g x) return (M.lookup x m)
  g x = do
        y <- t f x
        m <- get
        put $ M.insert x y m
        return y
```

这里`t`就是我们要memoized的函数, `x`是`t`的参数. `memorize`从一个`empty`的Map开始运行`f x :: MemoState a b`并返回它的值. 而`f`首先用`get`拿到了当前的状态(也就是Map), 随后检查是否计算过参数为`x`的结果, 如果是则返回包含结果的`MemoState a b`, 否则返回`g x :: MemoState a b`. `g`的话, 它首先计算参数为`x`的值, 注意到这个`t`的类型是`(a -> MemoState a b) -> (a -> MemoState a b)`, 这和我们之前讨论利用`fix`的函数类似, 都不递归调用自身, 而是调用额外的函数. 随后, 用`get`拿到了当前的状态(Map), 再用`put`更新状态(Map), 最后返回了一个包含结果和新状态的`MemoState a b`.

注意到这个`t`的类型, 意味着我们要改写原函数, 我们原先的`minEditDist`需要改为:

```haskell
-- minEditDistM :: ((String, String) -> MemoState (String, String) Int) -> (String, String) -> MemoState (String, String) Int
minEditDistM :: Monad m => ((String, String) -> m Int) -> (String, String) -> m Int
minEditDistM f ([],     [])     = return 0
minEditDistM f (s,      [])     = return $ length s
minEditDistM f ([],     s)      = return $ length s
minEditDistM f ((x:xs), (y:ys)) | x == y    = f (xs, ys)
                                | otherwise = (+1) . minimum <$> (sequenceA $ f <$> [(xs, ys), (xs, (y:ys)), ((x:xs), ys)])
```

所幸的是, 我们可以把`minEditDistM`, 也就是`t`的类型定义的更generic. 这样一来, 我们的`minEditDist`就可以实现为:

```haskell
-- memoized version
minEditDist :: String -> String -> Int
minEditDist s1 s2 = memorize minEditDistM (s1, s2)
```

至此, 我们就得到了泛用的`memorize`, 我们要做的仅仅是改写原先的函数, 即:

```haskell
origin :: a1 -> a2 ... -> b
-- 1. uncurry所有参数, (a1, a2, ...) -> b
-- 2. 添加额外的f, 替换调用自身的情况, ((a1, a2, ...) -> b) -> (a1, a2, ...) -> b
-- 3. 修改返回值为monad
modified :: Monad m => ((a1, a2, ...) -> m b) -> (a1, a2, ...) -> m b

-- memoized version
originMemo a1 a2 ... = memorize modified (a1, a2, ...)
```

### 总结

本文讨论了Haskell中两种memoization的手段, 一种根据具体问题具体的分析, 构建需要的数据结构来保存子问题的结果; 另外一种则利用一个泛用的`memoize`函数, 按特定的模式修改原函数即可实现memoization. 总体而言, 两种方式各有优劣, 第一种方法需要更精致能够得到更适合问题的解, 第二种方法则提供了泛用性. 如果你有任何问题, 欢迎[邮件](mailto:hey_christophe@outlook.com)我

### 参考

1. [Memoization](https://wiki.haskell.org/Memoization)
2. [Dynamic programming](https://en.wikipedia.org/wiki/Dynamic_programming)
3. [Memoizing function in Haskell](http://www.nadineloveshenry.com/haskell/memofib.html)
4. [Lazy Dynamic Programming](https://jelv.is/blog/Lazy-Dynamic-Programming/)
5. [Haskell/Understanding monads/State](https://en.wikibooks.org/wiki/Haskell/Understanding_monads/State)