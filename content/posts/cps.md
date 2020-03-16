---
title: "Continuation Passing Style and Tail Recursion"
date: 2020-02-29T21:40:07+08:00
draft: true
categories: ["Language concept"]
tags: ["Functional", "F#"]
---

之前在"Essentials of Programming Languages"中学习过CPS(*Continuation Passing Style*), 而笔记在blog改版后被丢弃, 故在这篇文章中重新详细的探讨下CPS以及尾递归, 且当是温故而知新.

## Continuation

在理解什么是"Continuation Passing Style"之前, 我们首先需要定义*Continuation*(惭愧的是我都不知道中文叫啥, 查了下好像是"续延"). *Continuation*是计算机程序控制状态的抽象表示, 换言之, 其就是一种表示程序执行中间某一计算步骤的控制上下文的数据结构, 即表示某一计算的未来. 一个形象的例子是阶乘函数:

```fsharp
let rec fact n = 
if n = 0
then 1
else n * (fact (n - 1))
```

当我们计算`(fact 4)`时, 其执行过程为(用lisp的形式是因为更加形象):

```scheme
(fact 4)
=> (* 4 (fact 3))
=> (* 4 (* 3 (fact 2)))
=> (* 4 (* 3 (* 2 (fact 1))))
=> (* 4 (* 3 (* 2 (* 1 (fact 0)))))
=> (* 4 (* 3 (* 2 (* 1 1))))
=> (* 4 (* 3 (* 2 1)))
=> (* 4 (* 3 2))
=> (* 4 6)
=> 24
```

这过程中, `(* 4 #)`就是一个*continuation*, 是`(fact 3)`的控制上下文, 等待着`(fact 3)`的计算结果传入我们的"#", `(* 4 (* 3 #))`也是一个*continuation*. 容易看到的是在这样一个计算过程中, *continuation*在不断增长, 因为调用栈在不断增长, 程序不得不保存`*`的左操作数, 计算右操作数. 接下来, 我们来看`fact`的尾递归版本, 所谓的**尾递归**(*tail recursion*)指的是递归函数的递归调用部分只有函数调用, 即:

```fsharp
let fact n = 
  let rec aux n acc = 
    if n = 0
    then acc
    else aux (n - 1) (acc * n)
  aux n 1
```

这里, `aux`是尾递归的, 其递归调用部分是一个函数调用. 这时候我们再来看`(fact 4)`的执行过程:

```scheme
(fact 4)
=> (aux 4 1)
=> (aux 3 4)
=> (aux 2 12)
=> (aux 1 24)
=> (aux 0 24)
=> 24
```

这过程中, 我们的*continuation*始终是`#`, 程序不需要保存额外的信息, 每次只需要调用函数`aux`就行.因此, 一个直接的结论是**尾递归不增长控制上下文**, 也就是说如果表达式e1返回的是表达式e2的值, 那么e1和e2应该在同一个*continuation*中. 因此, 如果一个递归函数是尾递归的, 那么它的调用栈可以不增长, 具体取决于语言是否有尾递归优化.

## Continuation Passing Style(CPS)

CPS指得是将*continuation*作为显示参数传递的风格, 例如, 阶乘函数`fact`的CPS版本为:

```fsharp
let rec factCPS n k = 
  if n = 0
  then k 1
  else fact (n - 1) (fun x -> k (n * x))
let fact n = factCPS n id
```

这里我们用单参数函数来表示*continuation*, 因为函数本身就是接受输入在执行后续的计算, 和*continuation*表示未来的计算不谋而合, 通常用`id`来表示一个终止的*continuation*. 在CPS中, 我们总是把下一步需要的计算作为函数的显示参数传入. 再来看一个fold的[例子](https://en.wikibooks.org/wiki/Yet_Another_Haskell_Tutorial/Type_basics#Continuation_Passing_Style):

```fsharp
let rec foldCPS f z = function
| [] -> z
| x :: xs -> f x z (fun y -> foldCPS f y xs)

let foldl f z ls = foldCPS (fun x z g -> g (f z x)) z ls
let foldr f z ls = foldCPS (fun x z g -> f x (g z)) z ls
```

这里`foldCPS`的`f`是CPS的, 其第一个参数是列表头部的元素, 第二个参数是累加的值, 第三个参数是一个*continuation*. 我们还通过`foldCPS`定义了`foldl`和`foldr`, `foldl f z [1; 2; 3; 4]`的逻辑是`(f (f (f (f z 1) 2) 3) 4)`, 即我们先计算头部元素和累加值的运算结果再将其传入*continuation*(接下来的计算); 而`foldr f z [1; 2; 3; 4]`的逻辑是`(f 1 (f 2 (f 3 (f 4 z))))`, 即我们先将累加值传入*continuation*(接下来的计算)得到结果后再计算其与头部元素的计算结果. 这和我们正常版本的`foldl`和`foldr`的逻辑是一致的:

```fsharp
let rec foldl f z = function
| [] -> z
| x :: xs -> foldl f (f z x) xs

let rec foldr f z = function
| [] -> z
| x :: xs -> f x (foldr f z xs)
```

CPS的一个重要特征就是所有函数都是尾递归的. 因此, 不难想到, 如果我们所有函数都能够写成CPS的形式, 那么就可以在具有尾递归优化的语言中受益. 下面就描述如何将普通的函数转化为CPS风格的函数:

1. 增加一个额外的参数表示我们的*continuation*(通常用`k`或`cont`)

2. 如果函数返回一个常量`c`, 就将其传入*continuation*返回, 即改为`k c`

3. 如果是一个尾调用, 即返回的是一个函数调用, 就改成在相同的*continuation*下调用该函数

4. 如果返回的一个表达式, 函数调用作为操作数, 就改为在一个新的*continuation*下调用该函数, 

   具体而讲就是构造一个新的函数, 其参数是函数调用的结果, 函数体是在旧的*continuation*下完成计算, 例如上文`factCPS`中的`else fact (n - 1) (fun x -> k (n * x))`.

   