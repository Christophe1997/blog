---
title: "Continuation Passing Style and Tail Recursion"
date: 2020-02-29T21:40:07+08:00
draft: true
categories: ["Language concept"]
tags: ["Functional", "F#"]
---

之前在"Essentials of Programming Languages"中学习过CPS(*Continuation Passing Style*), 而笔记在blog改版后被丢弃, 故在这篇文章中重新详细的探讨下CPS以及尾递归, 且当是温故而知新.

## 什么是CPS

在理解什么是"Continuation Passing Style"之前, 我们首先需要定义*Continuation*