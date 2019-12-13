+++
title = "value restriction"
author = ["christophe"]
lastmod = 2019-12-13T19:47:37+08:00
categories = ["language concepts"]
tags = ["OCaml", "F#"]
draft = true

+++

## Value Restriction 是什么？

Value restriction是用于控制类型推断能否对值声明进行多态泛化的规则（[MLton原文](http://mlton.org/ValueRestriction)：“The value restriction is a rule that governs when type inference is allowed to polymorphically generalize a value declaration.”）。常出现在ML系的语言中，如[SML](https://www.smlnj.org/)，[OCaml](https://ocaml.org/)，[F#](https://fsharp.org/)中，其实value restriction产生的本质原因是为了保证类型系统在结合参数多态与命令式特征（_imperative feature_，如`ref`）时候的可靠性（_soundness_）。一个典型的例子就是：

```F#
// 如果没有value restriction
let x = ref None  // 'a option ref
let y: int option ref = x // type checked
let z: string option ref = x // type checked
let () = y := Some 2  // type checked
let  v: string = !z  // 破坏了类型安全
```

## 限制了什么？

简单来讲，value restriction限制了类型泛化只能发生在表达式的右边是句法意义上的值。