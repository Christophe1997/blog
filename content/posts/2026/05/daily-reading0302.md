---
title: "Daily Reading 20260531"
date: 2026-05-31T22:00:00+08:00
draft: false
categories: ["Daily Reading"]
showToc: true
tags: ["AI", "Claude Code"]
---

## 前言

不得不感慨，懒惰真的是很难抗衡的东西。之前打算保持更新的 Daily Reading 不知不觉又停了俩月。该说是没有反馈呢还是本身就很难坚持，虽然其实还是保持
着阅读，但这个反过来输送到 blog 又是另外一回事。最近在尝试 Obsidian，准备按照 Zettelkasten 的方法来整理思绪，关于 Zettelkasten 这里就不在
过多介绍（AI 知道），整体而言就是把写作当成思考的过程，让笔记与笔记之间产生链接并生成新的想法。由于太久没更新，本文可能包含过期的内容。

## Daily Reading 20260531

> [Beyond the Prompt: Claude Code](https://arps18.github.io/posts/claude-code-mastery/)

如果你在使用 Claude，可以看下这个内容对自己使用姿势做一次查漏补缺。比如 `/branch` 可以把当前 session fork 出来；`/insights` 可以让 Claude 来分析你
最近的使用模式（文中建议一个月使用一次，我感觉现在两周使用一次更好），Claude 会输出一份 html，把你最近使用方式上做的好的、差的、可复用沉淀成 skill 的都
帮你整理出来。可以通过这份报告来更新自己的使用姿势，比如我就在这个报告的基础上沉淀了一个 `zettel-sync` 的[skill](https://github.com/Christophe1997/agent-extentions/blob/main/plugins/zettel-sync/README.md)，可以把最近会话整理成闪念笔记（Fleeting Notes，Claude 给的翻译，没有考察真正的中文翻译是什么，我觉得还是太抽象了，可能灵感更好点）。本来 Claude 给的建议是直接沉淀成永久笔记（Permanent Notes）或文献笔记（Literature Notes），
但我还是保留了由我们自己来整理这部分工作，毕竟如果都交给 AI，我们做这件事就失去了意义。

> [I’m tired of talking to AI](https://orchidfiles.com/im-tired-of-ai-generated-answers/)
> 
> I’m tired of talking to AI.
> 
> I want to talk to real people.
> 
> But even when I talk to people, they forward my questions to AI and send me the AI’s answer.

这个还是很真实的，引发了我的一些共鸣，最近天天跟 AI 聊天，还是会有点存在主义危机（可能有点夸张）。
之前看 Deep 的一个短剧，里面男主把跟所有人的聊天都交给 AI，AI 会分析你的行为习惯、说话方式，以及说你朋友/爱人/家人的行为习惯来帮你聊天。
比如你可能会忘记妈妈的生日，但是 AI 会帮你给妈妈庆生；你可能忽略了朋友最近的情绪低落，但是 AI 会帮你安慰开导你的朋友。
这样的 AI 虽然离现在还有一定的距离，但我看完还是不寒而栗，人本质是社交动物，如果 AI 取代了我来构建和他人之间的联系，那是否说明我的社会性已经不存在了。
短剧里没有演出来的是，如果对面也用了同样的 AI，最后就变成了 AI 和 AI 聊，而我们只不过是 AI 的养料。
之前可能对这种科幻的会感觉很遥远，但最近 AI 的发展，特别是编码领域，已经可以产生比较高质量的内容了，取代了我的一部分工作是实实在在发送的事情。
唯一值得庆幸的是，现在还是以我思考为主导。