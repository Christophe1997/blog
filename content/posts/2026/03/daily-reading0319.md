---
title: "Daily Reading 20260319"
date: 2026-03-19T19:14:02+08:00
draft: true
tags: []
categories: ["Daily Reading"]
showToc: true
TocOpen: false
---

## Daily Reading 20260319

> [How coding agents work](https://simonwillison.net/guides/agentic-engineering-patterns/how-coding-agents-work/)

虽然相关的内容都已经掌握了, 还是重新review了下有没有什么概念是遗漏的.

> [You Might Debate It — If You Could See It](https://blog.jim-nielsen.com/2026/opacity-of-generative-tools/)

这个说了个有意思的现象, 如果你能够看到, 你就可能会质疑合理性. 但如果你看不到, 你可能会选择默默接受. 这个本身适合解释很多社会现象, 这里就不讨论了. 回到编程上,
就变成了, 如果给你制定规范, 你可能会质疑合理性, 但如果我说我给你提供了一个Skill, 你可能就高高兴兴的接受了我制定的规范. 我们在使用开源的Skill时候, 很多时候都默认的接受
了Skill开发者的品味和审美, 无论好坏.

> [A sufficiently detailed spec is code](https://haskellforall.com/2026/03/a-sufficiently-detailed-spec-is-code)

作者通过OpenAI的[Symphony](https://github.com/openai/symphony)例子指出, 最好的规范文档是代码本身. 我最近在现有项目上使用claude code还是倾向性于拆小需求给他, 因为
我之前追求构建好完整的项目理解(references)后再试试能不能让他自动修改完整需求. 但我发现, 要在一个已有的项目上构建完整的, 用于LLM理解的规范, 本身就是一件麻烦的事.