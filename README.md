# Sigma-with-Wiki Tutor

> **Wiki Tutor** — 先构建知识库，再用知识库教学。
> Wiki 是持久化的知识源，Sigma 是教学引擎。

基于 Sigma SKILL，融合了 Karpathy 风格的互链 Markdown 知识库。学习者在学习之前，先通过 wiki 流程生产知识库；教学时直接从 wiki 读取相关页面作为教学内容来源；学习过程中沉淀的误解、掌握度等数据再写回 wiki，形成 **知识→教学→沉淀** 的闭环。

<p align="center">
  <img src="https://img.shields.io/badge/Agent_Skill-Wiki_Tutor-blue" alt="Agent Skill" />
  <img src="https://img.shields.io/badge/Method-Bloom's_2--Sigma-green" alt="Bloom's 2-Sigma" />
  <img src="https://img.shields.io/badge/Integration-Wiki_+_Tutor-orange" alt="Wiki + Tutor" />
  <img src="https://img.shields.io/badge/License-MIT-yellow" alt="MIT License" />
</p>

---

## 核心流程

```
                  ┌──────────────────────────────────────┐
                  │         Sigma-with-Wiki              │
                  │                                      │
  /wiki <action>  │                     /study <topic>   │
       │          │                         │            │
       ▼          │                         ▼            │
  ┌────────┐      │                   ┌────────────┐     │
  │ Ingest │      │                   │ 读取 Wiki  │     │
  │   ↓    │      │                   │    ↓       │     │
  │ Simple │      │                   │ 诊断       │     │
  │   ↓    │      │                   │    ↓       │     │
  │Teaching│──────┼────→              │ Roadmap    │     │
  └────────┘      │                   │    ↓       │     │
                  │                   │ 教学循环    │     │
                  │                   │    ↓       │     │
                  │                   │ 沉淀回写    │     │
                  │                   └────────────┘     │
                  └──────────────────────────────────────┘
```

---

## 功能特性

### 知识库（由 `/wiki` 管理）

- **Ingest** — 从 URL、PDF、本地文件摄取源材料，自动生成简单知识页
- **自动 Teachify** — 摄取后自动将简单知识转化为完整教学内容（含讲解、示例、误解、练习题）
- **Query** — 查询知识库并合成回答（优先简单知识，用户追问再引用教学深度内容）
- **Search** — 搜索知识库，返回文件路径和匹配行片段，用于定位源材料
- **Lint** — 健康检查：孤儿页、死链、标签合规、教学完整性等

### 教学（由 `/study` 管理）

- **概念依赖展开** — 自动从教学页面的关联引用中递归展开整个概念依赖链，按拓扑排序逐概念教学
- **苏格拉底式提问** — 从不直接给出答案，通过问题引导学习者自己发现
- **掌握度门控** — 评定量规评分（正确性 + 解释 + 新场景应用 + 区分能力），≥80% 才能进阶
- **误解追踪** — 识别错误背后的心智模型，设计反例让学习者自己发现矛盾
- **间隔重复** — 复习已掌握概念，正确则翻倍间隔（1d→2d→4d→...→32d）
- **学习沉淀** — 教学会话结束后，误解入库、掌握度更新、学习者画像持续累积

### 目录结构

```
sigma-wiki/                  ← 所有输出（相对当前运行目录）
├── SCHEMA.md                # Wiki 约定与标签分类法
├── index.md                 # 内容目录索引
├── log.md                   # 操作日志
├── raw/                     # 源材料（不可变）
├── entities/                # 实体知识页
├── concepts/
│   ├── simple/              # 简单知识（条目级定义）
│   └── teaching/            # 教学知识（完整教学内容）
├── comparisons/             # 对比分析
├── queries/                 # 查询存档
├── sessions/{topic-slug}/   # 学习会话（含 roadmap.html、concept-map、visuals）
├── misconceptions/          # 跨主题误解库
└── profile/
    └── learner-profile.md   # 学习者画像
```

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

## 命令参考

### `/wiki <action> [args]`

| 子命令 | 示例 | 说明 |
|--------|------|------|
| `ingest <url>` | `/wiki ingest https://...` | 摄取源材料（支持多条 URL） |
| `teachify <topic>` | `/wiki teachify python-decorators` | 将简单知识转化为教学知识 |
| `query <question>` | `/wiki query 闭包是什么？` | 查询知识库并合成回答 |
| `search <query>` | `/wiki search closure` | 搜索知识库，返回文件路径+行片段 |
| `lint` | `/wiki lint` | 知识库健康检查 |

### `/study <topic> [options]`

| 用法 | 说明 |
|------|------|
| `/study <topic>` | 启动新教学会话 |
| `/study <topic> --level <level>` | 指定起始水平（beginner / intermediate / advanced） |
| `/study <topic> --lang <code>` | 指定教学语言（如 `zh`、`en`） |
| `/study <topic> --resume` | 恢复已有会话 |

---

## 安装

该 skill 与 Hermes Agent 配合使用。将 skill 目录放入 Hermes 的 skills 目录：

```bash
# 如果使用技能管理器
npx skills add <repo-url>

# 或手动复制
cp -r sigma-with-wiki ~/.hermes/skills/
```

安装后，Hermes 会自动识别此 skill。

> **注意：** 此 skill 需要以下 Hermes 工具的支持：`web_extract`（URL 摄取）、`search_files`（搜索）、`read_file`/`write_file`（文件操作）、`terminal`（浏览器打开）。

---

## 与原始 skills 的关系

| Skill | 输出目录 | 说明 |
|-------|---------|------|
| `sigma`（原） | `sigma/` | 独立的苏格拉底导师（不依赖知识库） |
| `llm-wiki`（原） | `~/wiki/` | 独立的知识库管理（不含教学功能） |
| **`sigma-with-wiki`（本）** | **`./sigma-wiki/`** | **知识库驱动的教学系统（推荐）** |

三个 skill 可同时安装，目录互不重叠。

---

## 参考

- [Bloom's 2-Sigma Effect (Bloom, 1984)](https://doi.org/10.3102/0013189X013006004)
- [Karpathy's LLM Wiki Pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- [Sigma Tutor（原 skill）](https://github.com/sanyuan0704/sanyuan-skills)

---

## License

MIT
