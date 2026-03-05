---
title: "Daily Reading 20260305"
date: 2026-03-05T23:23:02+08:00
draft: false
tags: ["Agentic Engineering"]
categories: ["Daily Reading"]
showToc: true
TocOpen: false
---

## Daily Reading 20260305

> [Agentic Engineering Patterns](https://simonwillison.net/guides/agentic-engineering-patterns/)

这俩天抽空把已经更新的都看完了，整体还是有收获的。原则上，"Writing code is cheap now", 但我们还是要为生成内容负责，这应该是共识。同时，之前在golang-dev中看到过相关讨论，
Ross Cox的[观点](https://groups.google.com/g/golang-dev/c/4Li4Ovd_ehE/m/8L9s_jq4BAAJ?pli=1)认为我们不应该添加"Co-Authored-By agent"在commit message中，
这和我的观点也是一致的。由于claude code的默认提交风格总是会倾向于添加"Co-Authored-By", 最近我在自己的plugin中严格禁止了这一项:
```markdown
### Rule: No AI Attribution

**Never add `Co-Authored-By` footer for AI agents** (Claude Code, GitHub Copilot, OpenAI Codex, Cursor, etc.).

AI is a tool, not a co-author. The human takes full ownership of every commit in their repository.
```

第二部分介绍了测试优先的模式，这我觉得也是非常重要的，通过"Red,Green,Refactor"来保证AI生成的质量。 同时在会话开始的时候，优先让AI跑一次测试，这样后续会话AI都倾向于用测试来验
证改动。虽然我很认同这个模式，但实践起来还是有难度的，比如在我日常的工作中，经历的很多项目可能都没有很好的测试，这时候我先花时间跑一边测试的生成又好像有点麻烦，最近在思考怎么增量
的，动态的做这个工程。

第三部分是讨论如何理解AI生成的代码，这一块其实也是验证并保证AI生成质量的关键一环。里面提到的[showboat](https://github.com/simonw/showboat)我体验了一下，是一个有意思的工
具，他能够还原整个markdown的构建流程(由一系列showboat命令组成)，但我第一次尝试生成完再跑`verify`就报错了。但这个思路我感觉是合理的，目标就是保证AI生成的质量，把整个过程变
得可重复的。对于一些依赖项目实现生成的文档是非常有用的，我只需要重新跑一下`verify`就可以来确认文档是否还匹配当前项目。我最近在思考怎么跟我在开发中的插件[llm-doc](https://github.com/Christophe1997/agent-extentions/tree/main/plugins/llm-doc)结合来动态管理整个文档的生命周期。
