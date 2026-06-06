---
name: sigma-with-wiki
description: "Wiki Tutor — 基于 Bloom's 2-Sigma 的知识库驱动 1-on-1 AI 导师。先构建知识库（/wiki），再用于教学（/study）。融合了 Karpathy's LLM Wiki 的知识沉淀能力和 Sigma 的苏格拉底式教学方法。"
version: 1.0.0
author: Dimo CYM
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [tutor, wiki, knowledge-base, learning, education, bloom-2-sigma, socratic-method]
    category: education
    related_skills: [sigma, llm-wiki]
---

# Sigma-with-Wiki Tutor

> **Wiki Tutor** — 先构建知识库，再用知识库教学。
> Wiki 是持久化的知识源，Sigma 是教学引擎。

基于 Bloom's 2-Sigma 效应的个性化 1-on-1 导师，结合 Karpathy 风格的互链 Markdown 知识库：

1. **先建知识** — 通过 `/wiki` 命令从网页/论文/笔记构建结构化知识库
2. **再用知识教学** — 通过 `/study` 命令启动苏格拉底式教学会话，从知识库中读取内容进行教学
3. **沉淀学习成果** — 教学过程中发现的误解、掌握度数据写回知识库，形成知识→教学→沉淀的闭环

---

## 目录

- [快速开始](#快速开始)
- [目录结构](#目录结构)
- [命令参考](#命令参考)
  - [`/wiki` 命令](#1-wiki-命令)
  - [`/study` 命令](#2-study-命令)
- [工作流详解](#工作流详解)
  - [Wiki 工作流](#wiki-工作流)
  - [Study 工作流](#study-工作流)
- [数据模型](#数据模型)
- [概念依赖图展开](#概念依赖图展开)
- [参考文件](#参考文件)
- [原 Skill 说明](#原-skill-说明)
- [常见问题](#常见问题)

---

## 快速开始

```bash
# 1. 构建知识库：摄取源材料
/wiki ingest https://example.com/python-decorators-guide

# 2. 查询已有知识
/wiki query Python 装饰器是什么？

# 3. 开始学习
/study Python装饰器

# 4. 恢复上次学习
/study Python装饰器 --resume
```

---

## 目录结构

所有输出统一存放于当前运行目录下的 `sigma-wiki/` 文件夹。

```
sigma-wiki/
├── SCHEMA.md                       # Wiki schema + tag taxonomy
├── index.md                        # 内容目录索引（按类型分区）
├── log.md                          # 操作日志（追加式，按年轮转）
│
├── raw/                            # Layer 1: 不可变源材料
│   ├── articles/                   #   网页文章
│   ├── papers/                     #   论文 PDF
│   ├── transcripts/                #   访谈/笔记
│   └── assets/                     #   图片、图表等资源
│
├── entities/                       # Layer 2: 实体页面（人、组织、产品、模型）
│   └── {slug}.md
│
├── concepts/                       # Layer 2: 概念/主题页面
│   ├── simple/                     #   简单知识（条目级定义/概述，ingest 默认产出）
│   │   └── {slug}.md
│   └── teaching/                   #   教学知识（含讲解逻辑、示例、典型误解、练习）
│       └── {slug}.md
│
├── comparisons/                    # Layer 2: 对比分析页面
│   └── {slug}.md
│
├── queries/                        # Layer 2: 有价值的查询结果存档
│   └── {slug}.md
│
├── sessions/                       # Layer 2（新增）: 学习会话记录
│   └── {topic-slug}/
│       ├── session.md              #   会话状态：诊断结果、概念列表、掌握度时间线
│       ├── roadmap.html            #   视觉路线图（HTML，每轮更新）
│       ├── concept-map/
│       │   └── full.json           #   Excalidraw 全图
│       ├── visuals/                #   HTML 讲解、图表
│       │   └── {concept-slug}.html
│       └── summary.html            #   会话总结（里程碑/结束时生成）
│
├── misconceptions/                 # Layer 2（新增）: 跨主题误解库
│   └── {misconception-slug}.md
│
└── profile/
    └── learner-profile.md          # Layer 2（新增）: 综合学习者画像
```

### 路径变量

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `$WIKI` | `./sigma-wiki/` | Wiki 根目录（相对于当前运行目录） |
| `$TOPIC_SLUG` | — | Topic 的 kebab-case 形式，如 `python-decorators` |
| `$MISCONCEPTION_SLUG` | — | 误解的 kebab-case 形式，如 `closure-value-copy-misconception` |

---

## 命令参考

### 1. `/wiki` 命令

管理知识库：摄取、转化、查询、搜索、检查。

| 子命令 | 用法 | 说明 |
|--------|------|------|
| `ingest` | `/wiki ingest <url> [<url> ...]` | 摄取一个或多个 URL/文件到知识库 |
| `ingest` | `/wiki ingest <filepath>` | 摄取本地文件 |
| `teachify` | `/wiki teachify <topic>` | 将 simple 知识转化为 teaching 知识 |
| `query` | `/wiki query <question>` | 查询知识库并合成回答 |
| `search` | `/wiki search <query>` | 搜索知识库，返回文件路径 + 匹配行片段 |
| `lint` | `/wiki lint` | 检查知识库健康状况 |

---

### 2. `/study` 命令

启动或恢复教学会话。

| 用法 | 说明 |
|------|------|
| `/study <topic>` | 启动新教学会话 |
| `/study <topic> --level beginner\|intermediate\|advanced` | 指定起始水平 |
| `/study <topic> --lang <code>` | 指定教学语言 |
| `/study <topic> --resume` | 恢复已有会话 |

---

## 工作流详解

### Wiki 工作流

#### `/wiki ingest` — 摄取源材料

摄取一个或多个源材料（URL / 文件）到知识库，自动完成从源材料到教学知识的完整流程。

**步骤：**

```
1. 摄取原始材料
   a. URL → web_extract 获取 markdown
      文件 → read_file 读取
   b. 计算 body sha256
   c. 检查 raw/ 中是否已有相同 source_url 的文件
      ① sha256 相同 → 跳过（内容未变）
      ② sha256 不同 → 覆盖写入（内容漂移，记录日志）
      ③ 新 URL → 创建新文件
   d. 写入 raw/articles/{slug}.md（含 frontmatter）

2. 生成 simple 知识页
   a. 提取核心概念/主题
   b. 检查 concepts/simple/ 中是否已有对应页面
      ① 存在 → 更新内容，bump updated
      ② 不存在 → 创建新页面
   c. 写入 concepts/simple/{slug}.md

3. 自动 teachify（见下方 teachify 流程）

4. 更新索引与日志
   a. 更新 index.md
   b. 追加 log.md

5. 提示用户结果摘要
   "已将 [{topic}] 转化为教学知识，随时可以使用 `/study {topic}` 开始学习。"
```

**多条 URL 处理：** 先全部摄取完所有 URL，再逐个自动 teachify，最后汇总报告。

**错误处理：**
- URL 无法访问：记录失败，继续处理下一条，最后汇总报告
- sha256 相同：静默跳过
- teachify 失败（材料不足）：记录日志，simple 页面保留，后续可重试

---

#### `/wiki teachify <topic>` — 将简单知识转化为教学知识

将 `concepts/simple/{slug}.md` 转化为 `concepts/teaching/{slug}.md`，补充完整教学结构。

**触发时机：**
- **自动触发：** `/wiki ingest` 完成后自动执行
- **自动触发：** `/study <topic>` 时如果只有 simple 知识没有 teaching 知识，自动执行
- **手动触发：** `/wiki teachify <topic>`

**步骤：**

```
1. 检查前置
   a. concepts/simple/{slug}.md 不存在 → 提示先 /wiki ingest
   b. concepts/teaching/{slug}.md 已存在 → 询问是否覆盖

2. 读取源材料
   a. 读取 concepts/simple/{slug}.md
   b. 如果 simple 页有 sources: 字段 → 读取 raw/ 中对应的源文件
   c. 如果材料不足以构建教学内容 → 提示用户提供更多源材料，终止

3. 生成 teaching 页面（结构见下方「数据模型」的 Teaching 知识页）

4. 写入 concepts/teaching/{slug}.md

5. 更新 index.md 和 log.md

6. 提示："已将 [{topic}] 转化为教学知识。"
```

**质量门控：**
- `## Practice Questions` 必须包含至少 3 道题
- `simple_source` 字段必须指向有效的 simple 页

---

#### `/wiki query <question>` — 查询知识库

查询知识库内容，优先使用 simple 知识回答，用户追问时引用 teaching 知识。

**步骤：**

```
1. 从问题中提取关键词
2. 在 concepts/simple/ 中搜索匹配页面
3. 基于 simple 知识合成概要回答
4. 回答末尾附带："如果需要更深入的教学讲解，可以使用 `/study <topic>` 开始学习。"
5. 如果用户继续追问细节 → 搜索 concepts/teaching/ 中对应内容，给出详细回答
6. 有价值的合成内容 → 可选归档到 queries/
7. 追加 log.md
```

---

#### `/wiki search <query>` — 搜索知识库

搜索知识库中的页面内容。**与 query 不同，search 不合成回答**，而是返回文件路径和匹配行片段，用于定位源材料。

**搜索范围：**
- `entities/`、`concepts/simple/`、`concepts/teaching/`
- `comparisons/`、`queries/`、`misconceptions/`
- 排除 `raw/`（源材料不可索引）

**输出格式：每个结果包含**
- 文件路径（相对 $WIKI）
- 匹配行内容片段（含行号）

---

#### `/wiki lint` — 知识库健康检查

沿用 llm-wiki 的 lint 流程，增加以下检查项：

| 检查项 | 说明 |
|--------|------|
| Orphan pages | 无入链引用的页面 |
| Broken wikilinks | 指向不存在页面的链接 |
| Index completeness | 所有页面是否都在 index.md 中 |
| Frontmatter validation | 必填字段 + 标签在 taxonomy 中 |
| Stale content | updated 日期 >90 天 |
| Contradictions | 同主题冲突声明 |
| **Simple/Teaching 一致性** | simple 存在但 teaching 不存在→建议 teachify |
| **Teaching 完整性** | Practice Questions 是否≥3 道 |
| **Misconception 一致性** | related_sessions 引用是否有效 |
| **Orphan teaching pages** | Related Teaching Pages 中的 wikilinks 是否有效 |
| **Page size** | 超过 200 行建议拆分 |
| **Tag audit** | 不在 taxonomy 中的标签 |

---

### Study 工作流

#### `/study <topic>` — 启动教学会话

**Step 0: 读取 Wiki 知识**

```
检查 concepts/teaching/{topic-slug}.md 是否存在？
├── 是 → 读取 teaching 内容作为教学素材
└── 否 ↓
    检查 concepts/simple/{topic-slug}.md 是否存在？
    ├── 是 → 自动执行 teachify → 再读取 teaching 内容
    └── 否 ↓
        提示用户：未找到 [{topic}] 的知识，请先 /wiki ingest
```

同时读取 `index.md` 获取相关概念的拓扑信息，用于 roadmap 构建。

**Step 1a: 概念依赖展开**

读取 teaching 页的 `## Related Teaching Pages` 中的 wikilinks，递归展开整个概念依赖链。具体算法见下方「概念依赖图展开」章节。

**Step 1b: 诊断**

结合 wiki 知识的概要和 `profile/learner-profile.md` 进行定向诊断。

- 读取 learner-profile.md（若存在）
- 提出 1-2 个诊断问题
- 确定起始概念在概念图中的位置
- 记录初始掌握度状态到 `sessions/{topic-slug}/session.md`

**Step 2: 构建 Roadmap**

```
1. 从概念展开结果生成概念列表
2. 标记掌握度状态（全部 not-started，起始概念标记为 in-progress）
3. 生成 roadmap.html（视觉路线图，首次生成时打开 browser）
4. 生成 concept-map/full.json（Excalidraw 全图）
```

**Step 3: Tutor 循环（基于 Sigma 方法，以下仅列出整合差异）**

每个概念按以下子步骤教学：

| 步骤 | 内容 | 整合差异 |
|------|------|---------|
| 3a. 引入 | 用问题而非讲解开启新概念 | 沿用 Sigma |
| 3b. 提问循环 | 结构化选择 + 开放式问题 + 交错练习 | 从 teaching 页的 `## Practice Questions` 提取问题素材，不足时 agent 补充 |
| 3c. 回应 | 根据答案质量自适应（正确→加深、错误→诊断） | 沿用 Sigma |
| 3d. 误解追踪 | 识别错误背后的错误心智模型，设计反例 | 新发现的误解写入 `misconceptions/{slug}.md`（agent 去重） |
| 3e. 视觉辅助 | HTML/Excalidraw/图片 | 沿用 Sigma |
| 3f. 同步进度 | 每轮更新 session.md + roadmap.html | 路径改为 `sessions/{topic-slug}/` |
| 3g. 掌握度检查 | 评定量规评分 + 自评估比对 | 从 teaching 页的 Practice Questions 选题 |
| 3h. 练习阶段 | 动手任务（写代码/设计/解释） | 沿用 Sigma |

**Step 4: 沉淀回写**

```
4a. 会话记录
    最终写入 sessions/{topic-slug}/session.md

4b. 误解入库
    遍历 session.md 中 ## Misconceptions 表：
    - 检查 misconceptions/ 中是否有语义相同的页面（agent 判断）
    - 有 → 更新（增加 related_sessions 引用、更新时间戳）
    - 无 → 创建 misconceptions/{slug}.md

4c. Learner Profile 更新
    更新 profile/learner-profile.md：
    - Learning Style（本次观察）
    - Misconception Patterns（仅记录出现 2+ 次的模式）
    - Mastered Topics（新增/更新）
    - Metacognition（自评偏差分析）

4d. 索引与日志
    - 更新 index.md
    - 追加 log.md
```

---

#### `/study <topic> --resume` — 恢复教学会话

**步骤：**

```
1. 读取 sessions/{topic-slug}/session.md
2. 读取 profile/learner-profile.md

3. 间隔复习检查
   for each mastered concept in session.md:
       if days_since_review >= review_interval:
           → 加入复习队列（最多 5 个，按最 overdue 优先）

4. 复习阶段
   if 复习队列非空:
       a. 告知用户进入间隔复习
       b. 对每个 overdue 概念，问 1 道检查题
          ① 正确 → 翻倍间隔（1d→2d→4d→8d→16d→32d），更新 session.md
          ② 错误 → 重置间隔至 1d，标记 in-progress

5. 继续教学
   a. 从 session.md 中找到第一个 in-progress 或 not-started 的概念
   b. 恢复 Step 3（Tutor 循环）

6. 会话结束时执行 Step 4（沉淀回写）
```

**间隔重复计划表：**

| 条件 | 下次复习间隔 |
|------|------------|
| 首次掌握 | 1 天 |
| 复习正确 | 翻倍（1d→2d→4d→8d→16d→32d） |
| 复习错误 | 重置到 1 天 |
| 最大间隔 | 32 天 |

---

## 数据模型

### Simple 知识页（`concepts/simple/{slug}.md`）

条目级定义/概述。由 `/wiki ingest` 自动产出。

```yaml
---
title: "Python Decorators"
type: concept
knowledge_level: simple
created: YYYY-MM-DD
updated: YYYY-MM-DD
tags: [python, metaprogramming, decorators]
sources: [raw/articles/python-decorators-guide.md]
---
```

正文：条目级定义、概述、关键事实。格式不限 markdown。

---

### Teaching 知识页（`concepts/teaching/{slug}.md`）

完整的教学内容。由 teachify 流程产出。

```yaml
---
title: "Python Decorators"
type: concept
knowledge_level: teaching
created: YYYY-MM-DD
updated: YYYY-MM-DD
tags: [python, metaprogramming, decorators]
sources: [raw/articles/python-decorators-guide.md]
simple_source: concepts/simple/python-decorators.md
---

## Overview
[简述，适合快速回顾]

## Core Explanation
[多角度解释，从直觉到严谨]

## Examples
[典型代码/示例，标注关键要点]

## Common Misconceptions
[典型误解 + 为什么不对 + 正确理解]

## Edge Cases & Gotchas
[边界情况和容易出错的细节]

## Practice Questions
[3-5 道自测/练习问题，用于 Socratic 提问的素材]

## Related Teaching Pages
[[closures]] [[higher-order-functions]]
```

> **注意：** `## Related Teaching Pages` 中的 wikilinks 是 `/study` 展开概念依赖链的权威来源。每个 teaching 页应链接到其前置依赖概念。

---

### Misconception 页（`misconceptions/{slug}.md`）

跨主题的误解库。由 `/study` 教学循环中写入。

```yaml
---
title: "Misconception: Closures copy variable values"
type: misconception
created: YYYY-MM-DD
updated: YYYY-MM-DD
related_concepts: [concepts/teaching/closures]
related_sessions: [sessions/python-decorators/session]
tags: [python, closures, variable-capture]
status: active  # active | resolved
---

## The Misconception
[学习者错误相信的具体内容]

## Root Cause
[为什么会产生这个误解]

## Counter-Example Used
[用于纠正的反例设计]

## Status
- Discovered in: [[sessions/python-decorators/session]]
- Resolution date: [YYYY-MM-DD or pending]
- Still active in: [links to sessions where it reappeared]
```

---

### Session 记录（`sessions/{topic-slug}/session.md`）

每个学习会话的状态记录。

```yaml
---
title: "Session: Python Decorators"
type: session
created: YYYY-MM-DD
updated: YYYY-MM-DD
topic: concepts/teaching/python-decorators
status: in-progress  # in-progress | completed | interrupted
---
```

正文沿用 sigma 原有 `session.md` 格式：概念列表与掌握度表、误解追踪表、会话日志。

---

### Learner Profile（`profile/learner-profile.md`）

跨 topic 的长期学习者画像。仅由 `/study` 会话结束时更新（teachify 不触发此更新）。

```markdown
# Learner Profile
Updated: YYYY-MM-DD

## Learning Style
- Preferred explanation mode: concrete / abstract / visual / ...
- Pace: fast / moderate / needs-time
- Responds best to: predict / debug / teach-back / ...

## Misconception Patterns
- [recurring pattern] (seen in: {topic1}, {topic2})

## Mastered Topics
| Topic | Concepts | Date | Gaps |

## Metacognition
- Self-assessment accuracy: over-confident / well-calibrated / under-confident
- Fluency illusion frequency: rare / occasional / frequent
```

---

### Raw 源材料（`raw/articles/{slug}.md`）

原始源材料。不可变，agent 只读不修改。

```yaml
---
source_url: https://example.com/article
ingested: YYYY-MM-DD
sha256: <hex digest of body below frontmatter>
---
```

---

## 概念依赖图展开

当用户执行 `/study <topic>` 时，agent 自动展开整个概念依赖链来确定学习范围。

### 算法

```
输入: root_topic_slug（用户的入口 topic）
输出: ordered_concepts_list（拓扑排序的概念列表）

1. 读取 concepts/teaching/{root_topic_slug}.md
2. 解析 ## Related Teaching Pages 章节，提取所有 [[wikilinks]]
3. 对每个 wikilink：
   a. 检查 concepts/teaching/{linked_slug}.md 是否存在
      ① 存在 → 递归解析该页面的 Related Teaching Pages（去重，防循环）
      ② 不存在 → 检查 concepts/simple/{linked_slug}.md 是否存在
          - 存在 → 标记为 simple-only 引用点，加入展开结果
          - 不存在 → 忽略（broken link，lint 阶段处理）
4. 去重：使用 seen_set 防止循环引用
5. 拓扑排序：
   a. 从无依赖的概念开始
   b. 逐步加入有依赖的概念
   c. 循环依赖 → 取先遇到的概念为准
6. 输出 ordered_concepts_list
```

### 展开阈值

| 参数 | 值 | 说明 |
|------|-----|------|
| 最大递归深度 | 5 层 | 超过则停止并标记「未完全展开」 |
| 最大概念数 | 15 个 | 超过则只保留最相关的 15 个（直接依赖优先于间接依赖） |
| 孤立概念处理 | 无依赖 | 如果入口 topic 没有 `## Related Teaching Pages`，只含该单个概念 |

### 示例

```
入口: python-decorators

展开依赖链：
  python-decorators
    ├── closures
    │   └── higher-order-functions
    │       └── functions-as-first-class-objects
    └── higher-order-functions (已展开, 跳过)

拓扑排序结果：
  1. functions-as-first-class-objects (teaching)
  2. higher-order-functions (teaching)
  3. closures (teaching)
  4. python-decorators (teaching)
```

---

## 参考文件

```
sigma-with-wiki/
├── SKILL.md                        # 本文件
├── README.md                       # 公开描述与安装说明（待编写）
├── doc/
│   ├── proposal.md                 # 需求文档
│   └── detailed-design.md          # 详细设计文档
└── references/
    ├── pedagogy.md                 # Bloom's 2-Sigma 理论、苏格拉底提问法、掌握度评分
    ├── html-templates.md           # 暗色 UI HTML 模板（roadmap、summary、visuals）
    └── excalidraw.md               # Excalidraw 元素格式、色板、布局
```

`references/` 目录中的文件从 sigma 原 skill 复制。

---

## 原 Skill 说明

同时安装了 `sigma`、`llm-wiki` 和 `sigma-with-wiki` 三个 skill。整合版 `sigma-with-wiki` 优先使用。

| Skill | 输出目录 | 用途 |
|-------|---------|------|
| sigma（原） | `sigma/` | 独立的 Socratic 导师（无知识库依赖） |
| llm-wiki（原） | `~/wiki/` | 独立的知识库（无教学功能） |
| sigma-with-wiki（本） | `./sigma-wiki/` | 知识库驱动的教学系统（推荐使用） |

三个目录互不重叠，可同时安装。

---

## 常见问题

**Q: 没有 teaching 知识的 topic 可以学习吗？**

A: `/study` 会自动触发 teachify 流程，将 simple 知识转化为 teaching 格式。如果 simple 知识也不存在，请先 `/wiki ingest` 摄取源材料。

**Q: 同一个 topic 学第二次会怎样？**

A: 使用 `--resume` 会触发间隔复习（先检查已掌握的概念，再继续之前中断的教学）。不使用 `--resume` 会开始全新会话，但会从 learner-profile 和 misconceptions 中引用已有数据来加速诊断。

**Q: Ingest 一个 URL 后可以直接学吗？**

A: 是的。ingest 完成后自动执行 teachify，所以你只需一步：`/wiki ingest <url>` 然后 `/study <topic>`。

**Q: 如何查看当前学习进度？**

A: `sessions/{topic-slug}/roadmap.html` 每轮更新，可以打开浏览器查看。或者再次执行 `/study <topic>` 查看当前状态。

**Q: 如何查看所有已学的知识？**

A: 用 `/wiki query <topic>` 或 `/wiki search <keyword>` 查询知识库。

---

## 参考

- [Bloom's 2-Sigma Effect (Bloom, 1984)](https://doi.org/10.3102/0013189X013006004)
- [Karpathy's LLM Wiki Pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- [Sigma Tutor (原 skill)](https://github.com/sanyuan0704/sanyuan-skills)
