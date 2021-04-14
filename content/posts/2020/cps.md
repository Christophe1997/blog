---
title: "Continuation Passing Style and Tail Recursion"
date: 2020-02-29T21:40:07+08:00
draft: false
categories: ["Functional"]
tags: ["Language concepts", "Continuation", "F#"]
showToc: true
---

之前在"Essentials of Programming Languages"中学习过CPS(*Continuation Passing Style*), 而笔记在blog改版后被丢弃, 故在这篇文章中重新详细的探讨下CPS以及尾递归, 就当是温故而知新.

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

我们来看一个斐波那契数列的例子:

```fsharp
let rec fib n =
	if n < 2
	then 1
	else fib (n - 1) + fib (n - 2)
```

我们首先添加一个额外的参数`k`. 随后, 该函数总共有俩处返回, `then`分支返回的是常量, 则改为`k 2`; `else`分支返回了一个表达式, 而且俩个操作数都是函数调用, 我们首先改写左边的操作数, 即`fib (n - 1) (fun v1 -> v1 + fib (n - 2))`, 随后改写第二个操作数, 并将结果传入*continuation*中, 即`fib (n - 2) (fun v2 -> v1 + v2 |> k)`, 将这些整合就得到了CPS版本的`fib`:

```fsharp
let rec fibCPS n k = 
	if n < 2
	then k 1
	else fibCPS (n - 1) <| fun v1 -> fibCPS (n - 2) <| fun v2 -> v1 + v2 |> k
let fib n = fibCPS n id
```

可以看到, `fib`的CPS版本相较于原始版本显得更加不直观, 而这确实是CPS的缺点, 由于显示的传递控制上下文, 我们的代码变得不够直观.

## CPS与尾递归优化

所谓的尾递归优化(*tail call optimization*, TCO)指的是对于一个尾递归的函数, 例如我们有函数f, 其尾递归调用了g, 由于不需要额外的信息, 我们可以直接传递f的返回地址. 这样当g返回时, 其可以直接返回到f的调用者. 从上文我们知道, CPS总是尾递归的, 因此CPS可以和TCO同时使用来消除递归函数的调用栈的增长. 因此CPS可以用于那些具有尾递归优化的语言来使避免我们的递归函数栈溢出, 但由于CPS使得代码变得不够直观, 因此其效果可能并不如使用*accumulator*, 同时CPS也可以作为编译器的IR(*intermediate representation*), [SML/NJ](https://www.smlnj.org/)就是一个例子, 具体可以参考[Andrew](https://www.cs.princeton.edu/~appel/)的"Compiling with Continuations".

## .NET中的tail call

.NET的CIL中存在着`tail.`的opcode, 不过C#的编译器本身不会做TCO, 而F#的编译器则会处理TCO, 对于简单的尾递归, 例如递归调用自身, F#编译器通常将其优化为循环, 对于其他的情况才会使用`tail.`, 具体可以参考"[Tail calls in F#](https://devblogs.microsoft.com/fsharpteam/tail-calls-in-f/)".

## 总结

以上就是关于CPS与尾递归的介绍, 然而这仅仅是关于*continuation*的一点皮毛而已, 也没有涉及到`call/cc`的内容, 如果你对*continuation*有兴趣, 可以参考Haskell中的"[Continuation monad](https://wiki.haskell.org/All_About_Monads#The_Continuation_monad)". 另外, 如果你关于本文有任何问题, 欢迎[邮件](mailto:hey_christophe@outlook.com)我
