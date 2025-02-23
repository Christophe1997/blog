---
title: "My LLM codegen workflow atm"
date: 2025-02-23T21:05:33+08:00
draft: false
tags: ["Harper Reed"]
categories: ["Things I've Found"]
showToc: true
TocOpen: false
---

[My LLM codegen workflow atm](https://harper.blog/2025/02/16/my-llm-codegen-workflow-atm/), Harper Reed在文中介绍了基于LLM的代码生成工作流。主要介绍了两种场景，开发一个新项目（Greenfield）以及老项目的持续迭代(Non-greenfield)。

新项目基于需求细化（spec），计划制定（todo）以及代码生成三部分来开展。老项目则通过生成代码上下文（[repomix](https://github.com/yamadashy/repomix)）来制定测试回归和代码审查任务。这对我来说是一个巨大的启发，准备找时间试一下其中描述的工作流程。

同时Harper Reed也给出了具体的Prompt，局限于目前LLM的特性，仍然需要通过[提示工程](https://www.promptingguide.ai)来引导AI生成我们需要的内容。前一阵子较火的[DeepSeek从入门到精通](https://mp.weixin.qq.com/s/3Igd0u3ToUmPE-od_wzRKw)也指出掌握提示语设计是AIGC时代的必备技能，在平时使用这些LLM工具中也感受到怎么清晰的向AI表达需求非常重要，因为你的提示语完全决定了AI生成的质量，进而决定了AI是否好用或者为你带来提效。另外，由于中文的特性（高上下文依赖）以及模型的训练数据分布，可能会出现提示效果不如英文的情况。