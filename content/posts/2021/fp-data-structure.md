---
title: "函数式数据结构漫谈（一）"
date: 2021-04-06T23:26:20+08:00
draft: false
tags: ["Haskell", "Data Structure"]
categories: ["Things I Learned"]
showToc: true
TocOpen: false
---
近期计划开这个系列的坑，内容大多都是“Purely Functional Data Structures”内容加一点自己的理解（改一张牌就是我的了:），算是打磨文笔？
## 数据结构是什么
当我们在讨论数据结构的时候，我们在讨论什么。常见的介绍有“数据结构是一种数据组织、管理和存储的格式，它可以帮助我们实现对数据高效的访问和修改，更准确地说，
数据结构是数据值的集合，可以体现数据值之间的关系，以及可以对数据进行应用的函数或操作”。然而到更具体的场景，数据结构的概念还能够细化，比如我们经常会
讨论函数栈怎么怎么样，这里的“函数栈”也是数据结构，但它是一个泛指，是一个在程序执行过程中存在的概念，或者叫标识。Okasaki在他的“Purely Functional 
Data Structures”里指出，数据结构这一概念通常有四种含义：
1. 抽象，即抽象数据类型(_abstract data type_，可以用Java中的interface理解)，即表示数据的类型和一组适用于该类型的函数；
2. 实现，即对应于ADT的一个具体实现，通常是指对于该ADT做的具体设计；
3. 实例，即在程序运行中对应于一个数据类型的具体实例；
4. 泛指，即在程序运行中一个泛指的概念，不涉及具体的实例，例如上文提到的函数栈。
本系列将使用Haskell作为描述语言，则其中`class`可对应抽象的概念，`data`可对应实现的概念。具体到Java，可以用`interface`对应抽象的概念，用`class`对应实现
的概念。

## 函数式强调的是什么
抛开函数式编程本身强调的函数以外（不然就没法讲了），函数式编程通常还强调不可变（_immutable_）。因此，当我们说一个数据结构符合函数式的特性的时候主要在讨论不可变，
或者说持久性（_persistence_）。换句话说，在更新一个函数式的数据结构之后，它更新前的版本我们仍然能够访问到。这意味着所有有着破坏式更新的数据结构都不符合这一性质，
同时也表明相较于能够进行破坏式更新的数据结构，函数式数据结构的性能可能会更差，通常会有一个对数阶的更新代价在里面。实现持久性的方式非常简单，只需要将原有的数据结构
复制一遍，然后在复制后的数据结构上更新，由于没有破坏式的更，可以通过共享不变的部分来减少开销。下面讨论在函数式编程中经典的list。

### List
list在任何编程语言中都是非常常见的存在，函数式编程对其讨论则更多，著名的Lisp就取自“LISt Processor”。我们首先来看广泛的list定义
```haskell
class List t where
  empty :: t a
  isEmpty :: t a -> Bool 
  cons :: a -> t a -> t a
  -- error if the list is empty.
  head :: t a -> a
  tail :: t a -> t a
```
这里可以考虑将head和tail的结果包装一个`Maybe`，使之适合空的list，在这种情况下`isEmpty`就不再需要，因为`head ls = Nothing`或
`tail ls = Nothing`已经暗含`isEmpty ls = true`。本文为了偷懒就没用这种定义）。有了这些，我们就可以实现list上的各种“更高级的”
操作，例如经典的`map`：
```haskell
map :: List t => (a -> b) -> t a -> t b
map f ls = if isEmpty ls
  then emptll
  else cons (f $ head ls) (map f $ tail ls)
```
从这个定义上可以直接看出，这个List是不支持随机访问的。因此，我们额外定义支持“按下标”随机访问的`class`：
```haskell
class RandomAcess t where
  lookup :: Int -> t a -> a
  update :: Int -> a -> t a -> t a
```
将`List`和`RandomAcess`组合就得到了我们需要的`RandomAcessList`。虽然我们也可以直接用`List`的几个定义来实现`lookup`和`update`：
```haskell
lookup :: List t => Int -> t a -> a
lookup n ls = if isEmpty ls || n < 0 then error "Index out of range"
  else if n = 0 then head ls
  else lookup (n - 1) $ tail ls

update :: List t => Int -> a -> t a -> t a
update n v ls = if isEmpty ls || n < 0 then error "Index out of range"
  else if n = 0 then cons v $ tail ls
  else cons (head ls) (update (n - 1) v $ tail ls)
```
然而这样两者的效率都是$O(n)$的。即使无法在函数式的数据结构中讨论$O(1)$的随机访问，我们仍然希望能支持更高效的随机访问。而这其实可以从实现上着手，因此这里
将`RandomAcess`独立出来。

从`List`上的定义我们可以一窥所谓的持久性和共享不变的结构，主要考察几个返回类型为`t a`的函数：
1. empty返回一个空的list，对于一个确定的类型a来说，empty是不变的；
2. cons在一个已有的list上构建新的list，例如`ls1 = cons e ls`，此时ls1与ls共享ls的全部；
3. tail从一个list中获取除首部元素以外的部分，显然这也是共享的。

我们再来看一下list的经典实现，也就是单向链表：
```haskell
data LinkedList a = Nil | Cons a (LinkedList a) deriving(Eq, Show)

emptyError :: a
emptyError = error "Empty LinkedList"

instance List LinkedList where
  empty = Nil

  isEmpty Nil = True 
  isEmpty _ = False 

  cons = Cons

  head Nil = emptyError
  head (Cons e _) = e

  tail Nil =  emptyError
  tail (Cons _ ls) = ls
```
这个实现本身没有啥可以讨论的内容，因为和上面的定义几乎一致，好像只做了个翻译工作。