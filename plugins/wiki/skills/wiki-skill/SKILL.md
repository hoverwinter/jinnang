---
name: wiki-skill
description: Initialize and maintain a personal knowledge wiki (raw sources + Claude-curated pages with bidirectional links). Use when the user wants to create a new wiki/knowledge-base repo, ingest source material (articles / PDFs / transcripts / books) into wiki pages, query the wiki, or run wiki health checks. Triggers on phrases like "wiki", "knowledge base", "知识库", "/wiki-init", "/wiki-ingest", "/wiki-query", "/wiki-lint", "/wiki-stats", "/wiki-recent".
---

# Wiki Skill

通用个人知识库的初始化与维护。不绑定具体领域（金融、法律、医学、学习笔记皆可），由用户在初始化时声明主题。

## 核心模型

知识库分两层：

- **raw/** — 原始资料，只读。作者、出处、时间戳的原始来源。
- **wiki/** — Claude 维护的结构化知识，由 raw 提炼而来，页面之间 `[[双向链接]]`。

目录约定（任意语种通用）：

- `raw/articles/` Markdown 文章 · `raw/papers/` PDF 论文 · `raw/transcripts/` 音频转录 · `raw/books/` 书籍摘录
- `wiki/index.md` 总索引 · `wiki/log.md` 变更日志 · `wiki/overview.md` 概览
- `wiki/concepts/` 概念 · `wiki/entities/` 实体 · `wiki/topics/` 主题 · `wiki/sources/` 来源摘要

## 触发命令

用户输入以下任一命令时进入对应流程：

| 命令 | 用途 | 详见 |
|---|---|---|
| `/wiki-init [领域名]` | 初始化一个全新的知识库仓库 | 下节「初始化」 |
| `/wiki-ingest <文件>` | 摄入一份原始资料 | 下节「摄入」 |
| `/wiki-query <问题>` | 在知识库内查询 | 下节「查询」 |
| `/wiki-lint` | 健康检查（矛盾/孤立/过时） | 下节「检查」 |
| `/wiki-stats` | 页面数、来源数统计 | 下节「统计」 |
| `/wiki-recent` | 读 log.md 显示最近变更 | 下节「最近」 |

触发时：
1. 先确认当前目录是目标仓库（有 `CLAUDE.md` 且提到 wiki-skill，或 init 时为空目录/新仓库）。
2. 按对应流程执行。
3. 除 init 外，所有写入操作完成后必须追加一行到 `wiki/log.md`。

---

## 初始化（/wiki-init）

在空目录或新仓库中建立知识库骨架。

**流程：**

1. **确认领域**：询问用户领域名（如「金融」「法律判例」「机器学习论文」）和一句话描述。不要自己假定。
2. **确认语言**：询问用户主要工作语言（中文/英文等）。影响 CLAUDE.md 模板中的正文文案，但**结构字段名保持英文**（`type`、`name`、`created`、`updated`）以便工具处理。
3. **建目录**：创建下列空目录与占位文件：
   ```
   raw/articles/  raw/papers/  raw/transcripts/  raw/books/
   wiki/concepts/  wiki/entities/  wiki/topics/  wiki/sources/
   wiki/index.md   wiki/log.md   wiki/overview.md
   ```
4. **写 CLAUDE.md**：复制 `templates/CLAUDE.md`（本 skill 目录下），替换占位符：
   - `{{DOMAIN_NAME}}` → 领域名
   - `{{DOMAIN_DESC}}` → 一句话描述
   - `{{LANGUAGE}}` → 主要语言（如 `中文`）
5. **写入 index.md / log.md / overview.md 的初始骨架**（见「初始骨架」小节）。
6. **提示用户**：下一步可以用 `/wiki-ingest <文件>` 摄入第一份资料。

**不要**：

- 不要在 init 阶段生成示例概念或伪造内容。知识库从用户第一份 raw 资料开始生长。
- 不要自动 `git init`，除非用户明确要求。
- 不要预先创建 `comparisons/` 之类不会立即用到的目录。

### 初始骨架

`wiki/index.md`：
```markdown
# 知识库索引

> 最后更新：YYYY-MM-DD

## 概览
- [[overview|知识库概览]]

## 概念 (Concepts)
*尚无内容，使用 /wiki-ingest 开始积累*

## 实体 (Entities)
*尚无内容*

## 主题 (Topics)
*尚无内容*

## 来源 (Sources)
*尚无内容*
```

`wiki/log.md`：
```markdown
# 变更日志

- YYYY-MM-DD 知识库初始化，领域：{{DOMAIN_NAME}}
```

`wiki/overview.md`：
```markdown
---
type: overview
name: 知识库概览
created: YYYY-MM-DD
updated: YYYY-MM-DD
---

# {{DOMAIN_NAME}} 知识库概览

{{DOMAIN_DESC}}

## 主要分类
*随内容积累逐步补充*

## 核心来源
*随内容积累逐步补充*
```

---

## 摄入（/wiki-ingest）

把一份原始资料提炼进知识库。

**流程（8 步，标注哪些步骤需要用户参与）：**

1. **读取 raw** — 识别类型（article/paper/transcript/book）并读取全文。若用户给的是外部路径，先复制到 `raw/<类型>/` 下再读。
2. **过滤广告噪声** — 清除推销、引流、联系方式、无关水印（见「广告过滤清单」）。
3. **与用户讨论要点** ⚠️ — 列出 3-8 个候选要点请用户确认／裁剪，不要自行全量摄入。
4. **创建来源摘要** — 在 `wiki/sources/` 下建页，含核心观点、关键内容、相关概念、相关实体。
5. **更新实体** — 作者、机构、涉及的公司/人物等；已存在则增补，不存在则新建。
6. **更新概念** — 每个确认的要点对应一个概念页。**先搜索已存在概念**再决定"更新"还是"新建"（见「概念粒度」）。
7. **更新 index.md** — 把新页面加到对应分类下；页面计数与实际文件数保持一致。
8. **写 log.md** — 一行记录：`- YYYY-MM-DD 摄入 <来源名>，新增 X 概念 / 更新 Y 概念`。

**需要用户确认的节点**：第 3 步（要点提炼）、矛盾裁决、概念合并决策。其余步骤 Claude 自主完成。

### 广告过滤清单

- 微信号 / 二维码 / 邮箱 / 电话等联系方式
- "点赞/关注/转发/在看" 等引流话术
- 课程推销、付费社群、知识星球、Patreon 等推广链接
- 作者无关水印、平台广告、尾部互推内容

**保留**：作者信息、创作时间、核心内容、必要版权声明。

### 概念粒度

创建新概念前先判断是否为已有概念的**参数差异或子情形**：

- ❌ 把 `EMA55均线支撑` 和 `EMA20均线支撑` 分成两页
- ✅ 合并为 `均线支撑`，下设参数小节

合并规则：**定义相同或仅参数差异**才合并；表达不同侧面则分开并互链。

### 矛盾标注

不同来源观点冲突时保留双方，不要覆盖：

```markdown
> ⚠️ 矛盾：[[来源A]] 认为 X，[[来源B]] 认为 Y。
```

---

## 查询（/wiki-query）

1. 读 `wiki/index.md` 定位可能相关页面。
2. 读相关页面（概念 + 来源摘要）。
3. 回答用户，**必须引用具体来源**（`[[来源名]]`），避免无据输出。
4. 若知识库内无相关内容，明确告知而不是从模型记忆编造。

---

## 检查（/wiki-lint）

扫描以下问题并报告（不自动修复，列清单请用户裁决）：

- **矛盾**：同一主题下未标注的观点冲突
- **孤立页面**：无任何反向链接的概念/实体
- **死链**：`[[xxx]]` 指向不存在的页面
- **过时**：超过设定阈值（默认 12 个月）未 `updated` 的页面
- **统计漂移**：index.md 的分类计数与实际文件数不符
- **frontmatter 缺失**：页面缺 `type` / `name` / `created` / `updated` 任一字段
- **分类漂移**：index.md 出现拼写/措辞差异的重复分类

---

## 统计（/wiki-stats）

输出：页面总数、概念数、实体数、主题数、来源数、最近 7/30 天新增。用 `ls`/`wc` 而不是全文读取。

## 最近（/wiki-recent）

读 `wiki/log.md`，列最近 N 条（默认 10）变更。

---

## 通用原则（所有命令共享）

1. **raw 不可修改** — 原始资料只读。如需勘误，在对应来源摘要页面内标注。
2. **知识累积** — 更新而非覆盖，冲突保留双方并标注。
3. **双向链接** — 所有交叉引用用 `[[页面名]]`，不用文件路径。页面底部维护「相关来源」「相关概念」两段。
4. **日期统一** — frontmatter 与正文一律 `YYYY-MM-DD`。
5. **frontmatter 必填** — 所有页面：`type`、`name`、`created`、`updated`。
6. **先搜后写** — 创建任何页面前先搜 index.md，避免重复。
7. **批量摄入用 Agent** — 大批量扫描时先用 Agent 并行探查，合并 JSON 后再落盘。

## 演进原则（平台 vs 领域）

本 skill 是**平台层**，仓库 CLAUDE.md 是**领域覆盖层**。两者分工：

| 内容 | 归属 |
|---|---|
| 目录结构、命令集、frontmatter 字段名 | skill |
| 链接/矛盾/日期格式 | skill |
| 通用广告模式（二维码、引流话术等） | skill |
| 概念粒度原则、Ingest 流程、Lint 检查项 | skill |
| 领域名、主要语言、一句话定位 | 仓库 CLAUDE.md |
| index.md 已长出的具体分类列表 | 仓库 CLAUDE.md |
| 领域专有词汇 / 特定来源的怪癖 | 仓库 CLAUDE.md |
| 对 skill 默认值的覆盖（如 Lint 过时阈值） | 仓库 CLAUDE.md |

原则一句话：**机制上移到 skill，分类和词汇下沉到仓库**。

### 何时提升规则到 skill

当用户给出反馈或 Lint 发现模式时，按此判断：

| 信号 | 处理 |
|---|---|
| 在 ≥ 2 个不同领域都成立 | 提议更新本 skill |
| 是流程 / 格式 / 检查规则 | 提议更新本 skill |
| 提到领域专有名词 | 落到仓库 CLAUDE.md |
| 是具体分类 / 术语列表 | 落到仓库 CLAUDE.md |
| 是特定来源的怪癖 | 落到仓库 CLAUDE.md |
| 不确定是否通用 | 先落仓库，等跨仓库验证再升 |

### 反馈触发点

1. **用户纠正时**：分类后分别更新对应层。不确定则先放仓库。
2. **Lint 发现重复模式**：同类偏差 ≥ N 次，暗示 skill 规则该补。
3. **跨仓库归纳**：用户启动第二个 wiki 时，把两边 CLAUDE.md 重合的部分抽到本 skill。

升级本 skill 时，同步在 `CHANGELOG.md`（skill 根目录）记录一行。

## 页面模板

见 `templates/` 目录下：

- `templates/CLAUDE.md` — 仓库级 CLAUDE.md 模板（init 时落盘）
- `templates/concept.md` — 概念页模板
- `templates/entity.md` — 实体页模板
- `templates/source.md` — 来源摘要模板

需要使用模板时，读取这些文件并替换占位符。
