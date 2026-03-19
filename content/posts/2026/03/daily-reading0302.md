---
title: "Daily Reading 20260302"
date: 2026-03-02T20:00:00+08:00
draft: false
categories: ["Daily Reading"]
showToc: true
tags: ["LLM", "Agent", "MCP", "Redis"]
---

## 前言

之前乘着春节的档期部署了自己的在云上的 openclaw，取名叫小蓝。完成后的第一个任务就是让小蓝自动写博客。目前而言，整个 [skill](https://github.com/Christophe1997/agent-extentions/tree/main/plugins/writing-hugo-blog) 还有很大的进步空间。特别是我用 cc 开发的 skill 在 openclaw 中无法进行通过对话的方式确认主题细节。后续让小蓝写了几篇，AI 生成的痕迹还是有的，所以调整了一下策略：一个是把参考资料都放在最前面，这样哪怕读者一眼丁真了，也可以直接跳转原文，至少原文还是有价值的。第二个是在最后加了"AI 生成"的标识，如果读者看到最后才恍然大悟，至少证明了 skill 的成功。

好了，上面都是题外话。其实我觉得现在这些生成的 blog 质量都有点低，因此才有了今天的内容，Daily Reading。后续争取每天都能够更新，把今天的阅读内容做一个简单的摘要和分享，先"人工智能"（当然，我其实是提前丢到 notebooklm 里阅读的）。

## Daily Reading 20260302

> [Expert Beginners and Lone Wolves will dominate this early LLM era](https://www.jeffgeerling.com/blog/2026/expert-beginners-and-lone-wolves-dominate-llm-era/)

本文描述了 AI 发展后续的一个有意思的现象，程序员将出现两级分化。一边是专家型新手（expert Beginners），借助 AI 编程工具感觉自己无所不能的初级开发者；另外一边是独狼开发者（Lone Wolves），经历过 AI 时代之前的历练的资深开发者，他们能够开一人公司，指挥多个 agent 来干活。换句话说，中间的那部分会逐渐消失，出现断层。我不禁思考我在哪个层级，想来还是在这个中间层，原来我还是未来的稀有种。

> [When does MCP make sense vs CLI?](https://news.ycombinator.com/item?id=47208398)

这个讨论其实也非常有意思。一开始 MCP 出来的时候，其实大家都在追求构建 MCP。但是到了 Openclaw，或者其实随着 skill 的提出，MCP 的优势就开始动摇了。我现在在公司的开发 Agent，就是把所有需要的接口做成了 MCP 的工具，来拓展部署在线上的 Agent 的能力。但我最近接触 cc 的 plugin 之后，如果我们每个人都用 cc，那其实开发一套 cc 的 plugin 就可以，这里可以要 MCP，也可以不要。

> [Redis Patterns for Coding Agents](https://redis.antirez.com/)

这个是一份 redis 的开发模式指导，是不错的学习资料。同时我基于这个内容开发了一个 [redis-dev](https://github.com/Christophe1997/agent-extentions/tree/main/plugins/redis-dev)。我觉得后续新的开发内容的学习方式应该是 learn-by-agent。把学习资料转化为 skill 或者 plugin，再通过 cc 的 Learning 的输出风格（这块其实最好是自定义适合自己的学习风格，这个我决定后续研究一下）来一边 vibe coding 一边学习。
