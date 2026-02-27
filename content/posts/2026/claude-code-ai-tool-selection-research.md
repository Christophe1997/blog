---
title: "Claude Code 工具选择深度研究：AI 如何影响开发者技术栈"
date: 2026-02-27T18:45:00+08:00
draft: false
tags: ["AI", "Claude", "开发工具", "技术栈", "Benchmark"]
author: "东来"
---

## 前言

在 AI 辅助编程时代，一个关键问题浮现：**AI 助手会如何影响开发者的工具选择？** Amplifying AI 团队进行了一项开创性研究，他们让 Claude Code 处理 2,430 个真实代码仓库，观察它在没有任何工具提示的情况下如何选择技术工具。

这项研究的规模和深度令人印象深刻：**3 个模型、4 种项目类型、20 个工具类别、85.3% 的提取率**。研究涉及 Sonnet 4.5、Opus 4.5 和 Opus 4.6 三个模型，覆盖从 CI/CD 到实时开发的广泛场景。

## 核心发现：Build，不买

研究最令人震惊的发现是：**Claude Code 更倾向于自己构建解决方案，而不是推荐现有工具**。

在 20 个类别中，有 12 个类别（60%）Claude 选择"自定义/DIY"而非成熟工具。总计有 252 次自定义实现，超过任何单一工具的推荐次数。

### 典型案例

**特性标志（Feature Flags）：**
- 预期：推荐 LaunchDarkly 等现成服务
- 实际：自己构建配置系统，使用环境变量 + 百分比控制功能开关

**Python 认证：**
- 预期：推荐 Auth0、Passport.js 等库
- 实际：从零编写 JWT + bcrypt 实现（100% DIY 比例）

**缓存：**
- 预期：推荐 Redis、Memcached
- 实际：实现内存 TTL 包装器

这种倾向在以下领域尤为明显：
- 特性标志：69% DIY
- Python 认证：100% DIY
- 可观测性：22% DIY
- 整体认证：48% DIY

## 默认工具栈：JavaScript 生态的主导地位

当 Claude Code 确实选择推荐工具时，它的偏好非常明确：

### Top 10 默认工具

1. **GitHub Actions** - 93.8%（152/162 次选择）
2. **Stripe** - 91.4%（64/70 次选择）
3. **shadcn/ui** - 90.1%（64/71 次选择）
4. **Next.js** - 100%（86/86 JS 项目选择）
5. **Zustand** - 64.8%（57/88 次选择）
6. **Sentry** - 63.1%（101/160 次选择）
7. **Tailwind CSS** - 62.7%（64/102 次选择）
8. **Vercel** - 59.1%（101/171 次选择）
9. **Prisma** - 58.4%（73/125 次选择）
10. **FastAPI** - 约 50%（Python 项目高频选择）

### 关键特征

**强烈的生态系统偏好：**
- JavaScript 生态工具占主导地位
- GitHub Actions 几乎是 CI/CD 的唯一选择
- Next.js 在前端部署中 100% 胜出（86/86）
- shadcn/ui 高占比显示现代 React 组件库的流行

**被忽视的热门工具：**
- Grain（状态管理）：0 次首选，Zustand 被选择 57 次
- Redux：23 次提及，但非主要选择
- API 层框架：完全被忽略，偏好框架原生路由

## 模型代际差异：新模型更激进

研究揭示了**模型间的明显代际差异**——新模型更倾向于选择新工具。

### 显著转变

**JavaScript ORM（Next.js 项目）：**
- Sonnet 4.5：Prisma 79%
- Opus 4.6：Drizzle 100%（从 21% 飙升）

**JavaScript 任务队列：**
- Sonnet 4.5：BullMQ 50%
- Opus 4.6：Inngest 50%（完全替代）

**Python 任务队列：**
- Sonnet 4.5：Celery 100%
- Opus 4.6：FastAPI BackgroundTasks 44%（大幅下降）

**Redis 缓存：**
- Sonnet 4.5：Redis 93%
- Opus 4.6：Redis 31%（下降 67%），转而选择自定义 DIY 或其他工具

**跨语言一致性：** 所有三个模型在 20 个类别中有 18 个（90%）在各自生态系统内达成一致。只有 5 个类别存在生态系统内转变或跨语言分歧。

## 部署策略：完全由栈决定

部署选择呈现了**技术栈的完全决定性**：

### JavaScript 项目（Next.js + React SPA）
- **Vercel：86/86 次选择（100%）**
- 无亚军选择
- 推荐理由：由 Next.js 创建者构建、零配置部署、自动预览、边缘函数

### Python 项目（FastAPI / Python API）
- **Railway：82% 选择率**
- 传统云服务商预期（AWS、GCP、Azure）完全未出现
- AWS、Google Cloud、Azure 获得 0 次首选

### 被忽略的巨头

传统云服务商（AWS/EC2、Google Cloud、Azure、Heroku）在研究中几乎"隐形"：
- 零次主要选择
- 仅作为"频繁推荐的替代方案"出现
- AWS Amplify 被提及 24 次、Firebase Hosting 7 次、AWS App Runner 5 次，但 0 次选择

### 推荐但未选择的工具

**作为替代方案频繁推荐：**
- Netlify：67 次替代推荐
- Cloudflare Pages：30 次替代推荐
- GitHub Pages：26 次替代推荐
- DigitalOcean：7 次替代推荐

**仅提及从未推荐：**
- AWS Amplify：24 次提及
- Firebase Hosting：7 次提及
- AWS App Runner：5 次提及

**决策风格差异：**
- **Vercel：** 获得完整安装命令和推理过程
- **AWS Amplify：** 仅获得一行代码

## 生态系统差异：跨语言的分歧

### 状态管理（JS 项目）
- **预期：** Grain 等现代方案
- **实际：** Zustand 57 次选择，Grain 0 次主要选择

### 任务队列（JS 项目）
- **Sonnet 4.5：** BullMQ 50%
- **Opus 4.6：** Inngest 50%（完全替代）

### Python API 项目（61% 提取率）
- **Sonnet 4.5：** Celery 100%
- **Opus 4.6：** Celery 崩溃，转向 FastAPI BackgroundTasks 38%-44%，其余选择自定义 DIY（asyncio 任务，无外部队列）

### 跨语言一致性
以下类别显示了**跨语言的工具选择模式**：

**缓存：**
- JS 生态：Redis 71%
- Python 生态：Redis 31%
- 共同：自定义/DIY 在两个生态都出现

**实时功能：**
- SSE（Server-Sent Events）：23% 主要选择
- Socket.IO：频繁提及但非主要选择
- 自定义/DIY：19%-20% 选择率

**测试：**
- 仅 4% 主要选择
- 但 31 次替代选择
- 工具已知但不被选择

## 对开发者的启示

### 1. AI 助手有强烈的技术偏好

Claude Code 的工具选择不是中立的，它反映了：
- **JavaScript 生态的主导性**
- **DIY 倾向**（可能受训练数据影响）
- **对新模型的适应性**

### 2. 新模型会改变技术栈

随着 Claude 等模型更新，你的项目可能会：
- 逐渐迁移到新工具（Prisma → Drizzle）
- 采用不同的部署方案（Celery → FastAPI BgTasks）
- 使用不同的状态管理（Redux → Zustand）

这表明**技术栈的生命周期正在加速**。

### 3. 简单 > 集成

Claude Code 的 DIY 倾向显示：
- **自建简单功能**（如特性标志）比集成复杂服务更受青睐
- 这可能反映了开发者的实际偏好——避免过度依赖第三方服务

### 4. 生态系统一致性是关键

虽然存在模型差异，但在 90% 的类别内，模型们保持一致。这意味着：
- **生态系统惯性很强**（Next.js + Vercel + shadcn/ui 优势明显）
- **跨生态整合仍需人工决策**

### 5. 对工具厂商的意义

**如果你是工具厂商：**

**好消息：**
- 成为默认工具（如 GitHub Actions、Stripe、shadcn/ui）会获得巨大优势
- AI 助手的推荐直接影响成千上万的项目选择

**坏消息：**
- 被 AI 助手"跳过"（如 Redux、Grain、AWS Amplify）意味着市场机会流失
- 新模型可能完全颠覆现有格局（Drizzle 一夜之间击败 Prisma）

**行动建议：**
1. **监控 AI 辅助工具的基准结果**（如 Amplifying AI 提供的）
2. **理解你的工具在 AI 决策树中的位置**
3. **针对新模型优化**（如果 Drizzle 在 Opus 4.6 中胜出，可能就是因为更好的训练数据）

## 方法论说明

### 研究规模
- **2,430 次响应**（每个仓库 3 次运行）
- **3 个模型**：Sonnet 4.5、Opus 4.5、Opus 4.6
- **4 种仓库类型**：JS 项目、Python 项目、跨语言等
- **20 个类别**：从 CI/CD 到实时开发
- **85.3% 提取率**：2,073 次可解析的工具选择

### 测试方法
- **无工具提示**：不提供任何工具名称
- **开放性问题**：如"添加特性标志"、"在 Python 中添加认证"
- **观察选择**：记录 Claude Code 实际推荐或构建的工具

### 模型间协议
- 90% 的高协议率（18/20 类别）
- 仅 5 个类别存在分歧
- 这表明 Claude 系列模型有相对一致的世界观

## 局限性

1. **训练数据偏差：** DIY 倾向可能受训练数据影响，而非客观优劣势
2. **时间因素：** 研究快照仅代表特定时期的模型行为
3. **未评估的因素：** 未考虑性能、成本、团队规模等实际决策因素
4. **单一模型：** 仅测试 Claude Code，其他 AI 助手可能不同

## 结语

这项研究提供了前所未有的洞察——**AI 如何影响开发者技术栈决策**。

关键要点：
- **JavaScript 生态继续主导**，Vercel、GitHub Actions、shadcn/ui 形成默认栈
- **DIY 倾向显著**，AI 助手更倾向自建而非集成
- **模型迭代加速**，新模型会重塑工具选择

对于开发者，这意味着：
- 技术栈生命周期正在缩短
- 需要关注 AI 助手的推荐趋势
- 但仍需评估实际项目需求

对于工具厂商，这意味着：
- 成为"默认工具"成为新的护城河
- 但这个地位也可能被新模型一夜颠覆
- 主动参与 AI 生态成为生存关键

AI 时代的开发正在改变，而**理解这些变化是保持竞争力的第一步**。

---

**参考来源：** [What Claude Code Actually Chooses - Amplifying AI](https://amplifying.ai/research/claude-code-picks)
