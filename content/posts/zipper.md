---
title: "Zipper"
date: 2020-02-27T21:17:41+08:00
draft: true
categories: ["Data Structure"]
tags: ["Zipper", "Functional"]
---

近来想于函数式编程中寻找类似与双向链表的数据结构, 结果找到了Zipper. Zipper中文为拉链, 泛指一类常在函数式编程中使用的聚合数据结构, **其加强了原有的数据结构, 使得能够遍历或更新原有数据结构的任意部分**. zipper的关键思想是将目前需要处理的部分和不需要处理的部分分开, 同时保存目前不需要的部分, 也可以将其形象的理解为光标. 本文首先介绍了如何在list的基础上构建zipper, 随后将其扩展到二叉树上, 最后则介绍了原作者论文中的zipper.

## list的例子

list或者说是单向链表, 是函数式编程中最负盛名的结构, 其既简单又实用. 当我们需要修改list中的一部分时, 例如将`[1 2 3 4 5]`中的`[3 4 5]`替换为`[5 6]`, list就带来了问题. 这时候一个简单的做法是:

```F#
let rec changeTailOfList l1 idx l2 = 
  match l1 with
  | [] -> l2  // 忽略了idx大于0的情况
  | x::xs -> if idx = 0 then l2 else x :: (changeTailOfList xs (idx - 1) l2)
  
changeTailOfList [1; 2; 3; 4; 5] 2 [5; 6] // [1; 2; 5; 6]
```

看起来好像挺不错, 而这时候如果我们想要修改刚刚添加的内容, 例如在上面的例子中, 我们把`5`替换为`[7 8]`, 这时候同样可以采取和`changeTailOfList`一样的做法, 从头开始遍历列表, 在目标位置进行更改. 显而易见, 在需要频繁修改某一部分的场合下, list是不适用的(虽然list本身就不是为了频繁修改某一部分而设计的). 然而, 这种需求是确实存在的, 在可变的条件下, 我们很容易想到双向链表, 设定一个指针指向我们当前操作的位置便可以方便的进行前向或后向修改. 那我们要如何在不可变的条件下满足这一需求呢? zipper正是为了解决这一类问题. 首先我们来定义list上的zipper:

```F#
type 'a ListZipper = ListZipper of 'a list * 'a list
let zipper l1 = ListZipper (l1, [])
```

在一开始, 我曾提到可以将zipper理解为光标, 初始化时光标的位置位于开头. 当光标在一个列表中时, 容易看到其只能向前或向后移动:

```F#
let goForward (ListZipper (cur, back)) = 
  match cur with
  | [] -> failwith "Out of range" // 也可以用Maybe monad来处理
  | x :: xs -> ListZipper (xs, x :: back)
  
let goBack (ListZipper (cur, back)) = 
  match back with
  | [] -> failwith "At the beginning of list"
  | x :: xs -> ListZipper (x :: cur, xs)
```

现在我们可以移动光标了:

```F#
let origin = [1; 2; 3; 4; 5; 6]
let start = zipper origin
> let pos1 = goForward start;;
val pos1 : int ListZipper = ListZipper ([2; 3; 4; 5; 6],[1])
> let pos2 = goForward pos1;;
val pos2 : int ListZipper = ListZipper ([3; 4; 5; 6],[2; 1])
```

