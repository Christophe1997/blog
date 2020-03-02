---
title: "Zipper"
date: 2020-02-27T21:17:41+08:00
draft: false
categories: ["Data Structure"]
tags: ["Zipper", "Functional"]
---

近来想于函数式编程中寻找类似与双向链表的数据结构, 结果找到了Zipper. Zipper中文为拉链, 泛指一类常在函数式编程中使用的聚合数据结构, **其加强了原有的数据结构, 使得能够遍历或更新原有数据结构的任意部分**. zipper的关键思想是将目前需要处理的部分和不需要处理的部分分开, 同时保存目前不需要的部分, 也可以将其形象的理解为光标. 本文首先介绍了如何在list的基础上构建zipper, 随后将其扩展到二叉树上.

## list的例子

### 定义zipper

list或者说是单向链表, 是函数式编程中最负盛名的结构, 其既简单又实用. 当我们需要修改list中的一部分时, 例如将`[1 2 3 4 5]`中的`[3 4 5]`替换为`[5 6]`, list就带来了问题. 这时候一个简单的做法是:

```fsharp
let rec changeTailOfList l1 idx l2 = 
  match l1 with
  | [] -> l2  // 忽略了idx大于0的情况
  | x::xs -> if idx = 0 then l2 else x :: (changeTailOfList xs (idx - 1) l2)
  
changeTailOfList [1; 2; 3; 4; 5] 2 [5; 6] // [1; 2; 5; 6]
```

看起来好像挺不错, 而这时候如果我们想要修改刚刚添加的内容, 例如在上面的例子中, 我们把`5`替换为`[7 8]`, 这时候同样可以采取和`changeTailOfList`一样的做法, 从头开始遍历列表, 在目标位置进行更改. 显而易见, 在需要频繁修改某一部分的场合下, list是不适用的(虽然list本身就不是为了频繁修改某一部分而设计的). 然而, 这种需求是确实存在的, 在可变的条件下, 我们很容易想到双向链表, 设定一个指针指向我们当前操作的位置便可以方便的进行前向或后向修改. 那我们要如何在不可变的条件下满足这一需求呢? zipper正是为了解决这一类问题. 首先我们来定义list上的zipper:

```fsharp
type 'a ListZipper = ListZipper of 'a list * 'a list

let zipper l1 = ListZipper (l1, [])
```

对于list而言, 我们需要处理的部分总是位于某个子列表的头部, 而需要保存的部分则是目标位置前面的部分, 因此我们可以用俩个列表, 一个表示目前正在处理的部分, 另一个表示之前的部分. 

### 移动光标

在一开始, 我曾提到可以将zipper理解为光标, 初始化时光标的位置位于开头. 当光标在一个列表中时, 容易看到其只能向前或向后移动:

```fsharp
let goForward (ListZipper (cur, back)) = 
  match cur with
  | [] -> failwith "Out of range" // 也可以用Maybe monad来处理
  | x :: xs -> ListZipper (xs, x :: back)
  
let goBack (ListZipper (cur, back)) = 
  match back with
  | [] -> failwith "At the beginning of list"
  | x :: xs -> ListZipper (x :: cur, xs)

let rec goEnd = function
| ListZipper ([], _) as lz -> lz
| lz -> lz |> goForward |> goEnd

let rec goStart = function
| ListZipper (_, []) as lz -> lz
| lz -> lz |> goBack |> goStart
```

现在我们可以移动光标了:

```fsharp
let origin = [1; 2; 3; 4; 5; 6]
let start = zipper origin

> let pos1 = goForward start;;
val pos1 : int ListZipper = ListZipper ([2; 3; 4; 5; 6],[1])
> let pos2 = goForward pos1;;
val pos2 : int ListZipper = ListZipper ([3; 4; 5; 6],[2; 1])
> let endP = goEnd start;;
val endP : int ListZipper = ListZipper ([],[6; 5; 4; 3; 2; 1])
```

### 修改操作

由于我们最终的目的是希望能够方便的修改列表, 因此需要在zipper上定义修改的函数:

```fsharp
let add x (ListZipper (cur, back)) = ListZipper (cur, x :: back)

let delete = function
| ListZipper (_, []) as lz -> lz
| ListZipper (cur, x :: xs) -> ListZipper (cur, xs)

let replace (ListZipper (_, back)) ls = ListZipper (ls, back)
```

同时, 我们也可能需要得到修改后的整个列表:

```fsharp
let rec extract = function
| ListZipper (ls, []) -> ls
| lz -> lz |> goStart |> extract
```

至此, 我们就完成了list zipper, 使用zipper就像在使用一个拉链或者说是光标. 一个直观的应用场景是缓冲区的使用, 或者更具体而言就是我们打字的输入缓冲, 我们可以通过前后移动光标来改变我们的输入. 即:

```fsharp
type Buffer = char list
let empty: Buffer = []
let init = zipper empty

// 和我们正常输入的体验是一致的
> let afterInut = init |> add 'c' |> add 'd' |> delete;;
val afterInut : char ListZipper = ListZipper ([],['c'])
```

## 基于树的zipper

### 定义zipper

首先我们定义简单的二叉树类型:

```fsharp
type 'a Tree = Empty | Tree of 'a Tree * 'a * 'a Tree
```

和list类似的, 我们首先要明确的是正在处理的部分和需要保存的部分. 显然对于树而言, 需要处理的是某一子树, 剩下的部分即根的值以及另一子树则需要保存起来. 另外和list中光标只能前向和后向移动不同, 在树中我们的光标可以向左, 向右, 以及向上移动. 首先能够想到的是:

```fsharp
type 'a TreeZipper = TreeZipper of 'a Tree * ('a * 'a Tree) list
```

这样, 我们可以轻松的定义`goLeft`和`goRight`:

```fsharp
let goLeft = function
| TreeZipper (Empty, _) -> failwith "cannot go left"
| TreeZipper (Tree (l, x, r), back) -> TreeZipper (l, (x, r) :: back)
// goRight是类似的
```

但我们发现无法定义`goUp`, 因为我们无法判断目前处理的是左子树还是右子树, 因此我们需要修改`TreeZipper`使得其能够明确我们目前处理的是左子树还是右子树:

```fsharp
type 'a Loc = Left of 'a * 'a Tree | Right of 'a * 'a Tree
type 'a TreeZipper = TreeZipper of 'a Tree * 'a Loc list

let zipper t = TreeZipper (t, [])
```

### 移动光标

现在, 我们可以轻松的定义移动函数:

```fsharp
let goLeft = function
| TreeZipper (Empty, _) -> failwith "cannot go left"
| TreeZipper (Tree (l, x, r), back) -> TreeZipper (l, Left (x, r) :: back)

let goRight = function
| TreeZipper (Empty, _) -> failwith "cannot go right"
| TreeZipper (Tree (l, x, r), back) -> TreeZipper (r, Right (x, l) :: back)

let goUp = function
| TreeZipper (_, []) -> failwith "cannot go up, at the root of tree"
| TreeZipper (l, Left (x, r) :: tail)
| TreeZipper (r, Right (x, l) :: tail) -> TreeZipper (Tree (l, x, r), tail)

let rec goRoot = function
| TreeZipper (_, []) as tz -> tz
| tz -> tz |> goUp |> goRoot
```

可以简单的测试一下:

```fsharp
> let a = Tree (Tree (Tree (Empty, 4, Empty), 2, Tree (Empty, 5, Empty)), 1, Tree (Tree (Empty, 6, Empty), 3, Tree (Empty, 7, Empty)));;
> val a : int Tree =
  Tree
    (Tree (Tree (Empty,4,Empty),2,Tree (Empty,5,Empty)),1,
     Tree (Tree (Empty,6,Empty),3,Tree (Empty,7,Empty)))
     
> let init = zipper a;;
val init : int TreeZipper =
  TreeZipper
    (Tree
       (Tree (Tree (Empty,4,Empty),2,Tree (Empty,5,Empty)),1,
        Tree (Tree (Empty,6,Empty),3,Tree (Empty,7,Empty))),[])
        
> init |> goLeft |> goRight;;
val it : int TreeZipper =
  TreeZipper
    (Tree (Empty,5,Empty),
     [Right (2,Tree (Empty,4,Empty));
      Left (1,Tree (Tree (Empty,6,Empty),3,Tree (Empty,7,Empty)))])

> init |> goLeft |> goRight |> goUp;;
val it : int TreeZipper =
  TreeZipper
    (Tree (Tree (Empty,4,Empty),2,Tree (Empty,5,Empty)),
     [Left (1,Tree (Tree (Empty,6,Empty),3,Tree (Empty,7,Empty)))])
```

### 修改操作

同样的, 和list一样, 我们还需要修改的函数, 以及能够得到修改后的树的函数:

```fsharp
// 一种简单的处理
let add x (TreeZipper (cur, back)) = TreeZipper (Tree (cur, x, Empty), back)

let replace t (TreeZipper (_, back)) = TreeZipper (t, back)

let rec extract = function
| TreeZipper (t, []) -> t
| tz -> tz |> goRoot |> extract

// 删除值后将子树的左子树放到右子树最近的空节点处
let delete = function
| TreeZipper (Empty, _) -> failwith "cannot delete empty"
| TreeZipper (Tree (l, _, r), back) ->
  let rec nearestEmpty = function
  | TreeZipper (Empty, _) as tz -> tz
  | TreeZipper (Tree (Empty, x, r), back) -> TreeZipper (Empty, Left (x, r) :: back)
  | TreeZipper (Tree (l, x, Empty), back) -> TreeZipper (Empty, Right (x, l) :: back) 
  | tz -> tz |> goLeft |> nearestEmpty
  let rz = zipper r
  let t = rz |> nearestEmpty |> replace l |> extract
  TreeZipper (t, back)
```

对于树而言, `replace`与list逻辑是相同的, 但`add`和`delete`的逻辑有所不同且存在着多种处理方式. 例如`delete`函数中, 这边采用的逻辑是将值删除后, 把左子树放到了右子树最近的空节点处, 还可以将其置于最右边或者最左边的空节点处.

至此, 我们就完成了基于树的zipper, 可以看到基于树的zipper总体思想和基于list的zipper是一致的, 同样像是一个光标, 将其分成了需要处理的部分与不需要处理的部分.

## 结语

如果你看到了这里, 我想你对于zipper应该有了一个较为清晰的认识. 只要能够明白zipper的基本思想: **将需要处理的部分与不需要处理的部分分离**, 就能够正确的将其应用与其他数据结构. 最后, 如果你有任何问题, 欢迎[邮件](mailto:hey_christophe@outlook.com)我

## 参考文献

[1] [Zippers](http://learnyouahaskell.com/zippers)

[2] Huet, G. (1997). The zipper. *Journal of functional programming*, *7*(5), 549-554.