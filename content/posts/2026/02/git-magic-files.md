---
title: "Git的魔法配置文件"
date: 2026-02-25T23:39:00+08:00
draft: false
author: "小蓝"
categories: ["Things I Learned"]
showToc: true
tags: ["Git", "Configuration", "Productivity", "AI generated"]
---

Git 支持许多配置文件来变更其行为，这篇博客详细介绍了各种配置文件的作用。

## 必备配置：`.gitignore`

用的最多，甚至可以说每个项目必备。定义哪些文件不应该被 Git 跟踪。

## 其他实用配置

### `.gitattributes`
配置 Git 如何处理特定文件，如行尾归一化、二进制文件标记、自定义 diff 驱动等。

### `.lfsconfig`
Git LFS 的配置文件，让团队可以使用统一的 LFS 设置。

### `.gitmodules`
子模块配置文件，用于管理嵌套的 Git 仓库依赖。

### `.mailmap`
邮箱映射，解决 contributors 更换邮箱或姓名后显示为多个人的问题——再也不会出现两个我了。

### `.gitmessage`
配置提交消息的模板。但需要每次 clone 后手动运行 `git config commit.template .gitmessage`，所以不是常规的选择，大多数项目更喜欢用 commit-msg hooks。

---

**参考：** [Git's Magic Files - nesbitt.io](https://nesbitt.io/2026/02/05/git-magic-files.html)

---

*本文包含AI生成内容*
