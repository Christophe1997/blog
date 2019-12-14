+++
title = "value restriction"
author = ["christophe"]
lastmod = 2019-12-13T19:47:37+08:00
categories = ["language concepts"]
tags = ["OCaml", "F#"]
draft = false

+++

## Value Restriction是什么？

Value restriction是用于控制类型推断能否对值声明进行多态泛化的规则（[MLton原文](http://mlton.org/ValueRestriction)：“*The value restriction is a rule that governs when type inference is allowed to polymorphically generalize a value declaration.*”）。常出现在ML系的语言中，如[SML](https://www.smlnj.org/)，[OCaml](https://ocaml.org/)，[F#](https://fsharp.org/)中，其实value restriction产生的本质原因是为了保证类型系统在结合参数多态与命令式特性（_imperative feature_，如`ref`）时候的可靠性（_soundness_）。一个典型的例子就是：

```fsharp
// 如果没有value restriction
let x = ref None  // 'a option ref
let y: int option ref = x // type checked
let z: string option ref = x // type checked
let () = y := Some 2  // type checked
let  v: string = !z  // 破坏了类型安全
```

## 限制了什么？

简单来讲，value restriction限制了类型泛化只能发生在表达式的右边是句法意义上的值。那么什么是句法意义上的值呢，SML的[语言规范](http://sml-family.org/sml97-defn.pdf)上明确给出了什么样的表达式是句法意义上的值（准确来说是*non-expansive*）:

- 常量，如`13，"string"`
- 变量，如`x,y`
- 函数，如`fn x => e`
- 除了`ref`以外的构造函数在值上的调用，如`Foo v`
- 类型上受约束的值，如`v: t`
- 每一个元素都是值的**tuple**, 如`(v1, v2, v3)`
- 每一个字段都是值的**record**, 如`{l1 = v1, l2 = v2}`
- 每一个元素都是值的**list**, 如`[v1, v2, v3]`

确切的来讲，只要是协变（covariant）的类型并且不和可变的特性相结合，那么它总是可以类型安全的泛化（[OCaml manual](https://caml.inria.fr/pub/docs/manual-ocaml/polymorphism.html)原文：“*As a corollary, covariant variables will never denote mutable locations and can be safely generalized.*”）。即：

1. 是没有副作用的
2. 表达式的结果是一个不可变对象

## 在完备性上的问题

从上述规则来看，`let x = ref None`显然是非法的表达式，然而在引入value restriction的同时，类型系统损失了一定的完备性（_completeness_），因为以下代码同样违反了value restriction：

```fsharp
let id x = x  // 'a -> 'a
let listId = List.map id  // 违反了value restriction
```

即使我们只使用不可变特性，上述代码依然无法通过类型检查。因为函数调用不是句法意义上的值(因为编译器无法判断函数调用是否是pure的)。当然上述问题可以通过[eta-expansion](http://mlton.org/EtaExpansion)来避免，即：

```fsharp
let listId = fun x -> List.map id x  // 'a list -> 'a list
```

lambda表达式是句法意义上的值，因此上述代码是可以通过类型检查的。

### 如何避免value restriction

为了能够使得我们本身soundness的代码通过类型检查，在value restriction的限制下我们不得不做一些额外的工作。

1. eta-expansion

   向上一个例子那样，我们可以引入一个自由变量，使得函数调用变成了一个函数声明，从而通过了类型检查。

   ```fsharp
   let lsitId = fun x -> List.map id x
   ```

   在这种情况下，每一次`listId`被调用时，`List.map id`都会被调用。而不是像原来那样只在声明`listId`时调用一次，当然在有些情况下这可能会造成一个性能问题。

2. 引入局部变量，例如以下代码同样无法通过类型检查

   ```fsharp
   type 'a T = A of string | B of 'a
   let a = A (if true then "yes" else "no")  // failed
   ```

   但是可以修改为

   ```fsharp
   let s = if true then "yes" else "no" in 
   let a = A s
   ```

   使得其符合value restriction的规则。

## OCaml和F#中的value restriction

OCaml和F#同样存在着value restriction的完备性的问题，俩者通过不同的方式对其进行了relax。

### OCaml的relaxed value restriction

OCaml通过引入一个弱类型变量来放宽value restriction. 所谓弱类型变量是指编译器未知的变量，而一旦这个弱类型变量被编译器推断为一个具体的变量时，该弱类型变量就被具体的变量所替代，并且不在可变。例如：

```ocaml
# let a = ref None;;
val a : '_a option ref = {contents = None}
# let () = a := Some 2;;
# a;;let v<'T> : 'T option ref = ref None;;
val v<'T> : 'T option ref
- : int option ref = {contents = Some 2}
```

这和我们第一个例子是类似的，同意违反了value restriction。但是OCaml将a的类型推断为`'_a option ref`，这里的弱类型变量`'_a`指代的是未知的类型变量，在`let () = a := Some 2`中，编译器将`'_a`推断为`int`并且将a的类型固定为`int option ref`，通过这样的处理解决了第一个例子所展示的类型不安全的问题。换一种角度来看，所谓的弱类型变量是推迟了推断的具体的变量，即具体变量的占位符。这样确实解决了原有value restriction的完备性的问题，但同样导致了某些程序不在足够的泛化。例如

```ocaml
# let id x = x;;
val id : 'a -> 'a = <fun>
# let listId = List.map id;;
val listId : '_a list -> '_a list = <fun>
```

和前面一样，这同样是一个违反了value restriction的例子，于是OCaml使用了弱类型变量来处理，这意味着一旦我们在`int list`类型上调用完`listId`，例如`listId [1; 2; 3]`，之后`listId`就被固定为`int list -> int list`，这意味着我们无法再在`string list`上调用`listId`，而这同样不符合我们泛化的初衷，即`'a list -> 'a list`。当然我觉得OCaml的relaxed value restriction算是处理的非常优雅，有兴趣的可以阅读相关论文[6]。

### F#的处理

虽说F#参照了OCaml, 但还是存在着相当多的不同之处，在value restriction的处理上俩者也存在着区别。在F#中，上述违反了value restriction的例子依然是非法的。F#[语言规范](https://fsharp.org/specs/language-spec)中同样明确给出了可以泛化的情况（_generalizable_）:

- 函数表达式
- 实现接口的对象表达式
- 委托表达式
- 右边同样是可泛化的`let`表达式
- 右边同样是可泛化的`let rec`表达式
- 所有元素都是可泛化的tuple表达式
- 所有字段都是可泛化且不包含可变字段的record表达式
- 所有参数都是可泛化的`union case`表达式（即union类型表达式）
- 所有参数都是可泛化的`exception`表达式
- 空的`array`表达式
- 常量表达式
- 带有`GeneralizableValue`标签的类型函数的调用

因此在F#中`listId`同样是非法的。但是F#允许你引入一个显示的泛型参数来解决这个问题，即：

```fsharp
> let listId<'T> : 'T list -> 'T list = List.map id;;
val listId<'T> : ('T list -> 'T list)
```

这样的处理虽然不够优雅，但似乎是完美解决了这个问题，因为这里不会出现OCaml那样泛化不够的问题。但我们在看`ref`的问题：

```fsharp
> let v<'T> : 'T option ref = ref None;;
val v<'T> : 'T option ref
> v := Some 2;;
val it : unit = ()
> let x: int option = !v;;
val x : int option = None  // Oops
```

我们看到，这里x的值居然是`None`，而不是预期的`Some 2`。实际上这里的`v`并不是一个`ref`对象，而是一个泛型类，其接收一个泛型参数，产生一个具体的类，当我们对`v`赋值时，真正调用的是`(v<int>) := Some 2`，而此时会生成一个新的`ref`对象。即使我们使用`let x: int option = !v<int>`得到的依然是`None`，因为此时又生成了一个新的`ref`对象，这个行为是由IL所决定的（有兴趣可以参考[4]）。因此我们不得不声明类型变量:

```fsharp
> let v1 : int option ref = v<int>;;
val v1 : int option ref = { contents = None }
> let () = v1 := Some 2;;
> let x = !v1;;
val x : int option = Some 2
```

而这就又回到了OCaml的relaxed value restriction，并且比F#更加优雅：

```ocaml
# let v1 = ref None;;
val v1 : '_a option ref = {contents = None}
# let () = v1 := Some 2;;
# let x = !v1;;
val x : int option = Some 2
```

可见俩者在一定程度上是等价的。对于`lsitId`而言F#更有优势，因为泛型方法能够自动推断参数类型。而对于

`ref`对象而言，OCaml的处理更优雅，因为F#中，`v`变成了一个泛型类，而不是普通的值，而这是比较令人困惑的。在F#中，为了避免这样的问题，可以使用`[<RequiresExplicitTypeArguments>]`，即：

```fsharp
[<RequiresExplicitTypeArguments>]
let v<'T> : 'T option ref = ref None
```

在这样的情况下，你将无法使用`v := Some 2`，而必须使用`v<int> := Some 2`，这样就能清晰的表示`v`是一个泛型类而不再是一个普通的值。另外，值得一提的是F#还提供了`[<GeneralizableValue>]`（即上述可泛化对象的最后一条），来告诉编译器这是一个可泛化的值：

```fsharp
> [<GeneralizableValue>]
- let v<'T> : 'T option ref = ref None;;
val v<'T> : 'T option ref
> let a = v;;
val a : 'a option ref
```

如果没有`[<GeneralizableValue>]`，`let a = v`将违反value restriction.

## 结语

如果你看到了这里，我想你对value restriction应该有了一个清晰的认识，并且对OCaml和F#如何放宽value restriction有了充分的了解。而如果你使用F#编程，那么我的建议是除非你清楚的知道自己在做什么(即添加额外的泛型参数)，否则就按照MSDN的[建议](https://docs.microsoft.com/en-us/dotnet/fsharp/language-reference/generics/automatic-generalization#value-restriction)，我这边稍微扩展了一下：

1. 添加一个显示的参数，使得其变为具体的类型

   ```fsharp
   let counter = ref None
   // Adding a type annotation fixes the problem:
   let counter : int option ref = ref None
   ```

2. 使用eta-expansion将函数组合与部分调用展成一个lambda表达式或常规的函数

   ```fsharp
   let maxhash = max << hash
   // The following is acceptable because the argument 
   // for maxhash is explicit:
   let maxhash obj = (max << hash) obj
   // or
   let maxhash = fun obj -> (max << hash) obj
   ```

3. 引入局部变量来重写表达式

   ```fsharp
   type 'a T = A of string | B of 'a
   let a = A (if true then "yes" else "no")
   // introducing a local variable fixs the problem
   let s = if true then "yes" else "no" in 
   let a = A s
   ```

4. 通过添加一个额外的，无用的参数将表达式变成一个[thunk](https://en.wikipedia.org/wiki/Thunk)

   ```fsharp
   let emptyList10 = Array.create 10 []
   // Adding an extra (unused) parameter makes it a function,
   // which is generalizable.
   let emptyList10 () = Array.create 10 []
   ```

最后，如果你有任何问题或者关于该文章的任何建议，欢迎[邮件](mailto:hey%5Fchristophe@outlook.com)我。

## 参考文献

[1] [ValueRestriction](http://mlton.org/ValueRestriction)

[2] [Polymorphism and its limitations](https://caml.inria.fr/pub/docs/manual-ocaml/polymorphism.html)

[3] [Relaxed value restriction](https://ocamlverse.github.io/content/weak_type_variables.html#relaxed-value-restriction)

[4] [Finer Points of F# Value Restriction](https://blogs.msdn.microsoft.com/mulambda/2010/05/01/finer-points-of-f-value-restriction/)

[5] Wright, A. K. (1995). Simple imperative polymorphism. *Lisp and symbolic computation*, *8*(4), 343-355.

[6] Garrigue, J. (2004, April). Relaxing the value restriction. In *International Symposium on Functional and Logic Programming* (pp. 196-213). Springer, Berlin, Heidelberg.