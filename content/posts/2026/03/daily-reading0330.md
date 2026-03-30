---
title: "Daily Reading 20260330"
date: 2026-03-30T23:14:02+08:00
draft: falise
tags: ["Agents"]
categories: ["Daily Reading"]
showToc: true
TocOpen: false
---

## Daily Reading 20260330

> [Thoughts on slowing the fuck down](https://mariozechner.at/posts/2026-03-25-thoughts-on-slowing-the-fuck-down/)

fuck 含量极高的一篇文章~, 探讨了agent带来的技术负债. 过去我们人开发项目的时候, 会由于技术负债的痛苦不得不对项目进行迭代/重构来减少负债. 但现在
完全由AI开发, 由于AI只做风格迁移, 也会产生技术负债, 只是这种负债的痛苦我们不再需要感知, 最后的结果就是改不动, 无法继续迭代. 当然这里有人会说, 现在有
很多工具, 像[superpowers](https://github.com/obra/superpowers), [ce](https://github.com/EveryInc/compound-engineering-plugin), 
[gstack](https://github.com/garrytan/gstack), 都可以来工程化, 更好的理解, 迭代项目. 这些工具都能很好的教LLM基于你的项目来写需求, 但我觉得这里有个问题,
我们使用的时候不得不维护大量的文档来构建项目的知识, 但实际上"A sufficiently detailed spec is code", 代码永远都是最新的规范, 而我们维护的大量md其实存在新鲜
度的问题. 当然, 我们可以做一个post-hook来更新, 但这里也正是问题可能出现的地方. 另外, 最近nodejs社区也有关于拒绝在node.js core中使用AI生成的提交的[请愿](https://github.com/indutny/no-ai-in-nodejs-core).