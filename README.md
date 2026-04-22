# 锦囊 · jinnang

> hoverwinter 的个人 Claude Code 插件合集。
> 锦囊里是针对特定场景的预制方案——打开即用。

## 安装

在 Claude Code 中添加本 marketplace：

```
/plugin marketplace add https://github.com/hoverwinter/jinnang
```

然后按需安装单个 plugin：

```
/plugin install wiki@jinnang
```

后续升级：

```
/plugin update wiki@jinnang
```

## 目录中的 Plugins

### wiki — 个人知识库管理

把一个空目录变成一个由 Claude 维护的知识库仓库：原始资料（`raw/`）+ 结构化页面（`wiki/`），页面之间双向链接。

**主要命令：**

| 命令 | 用途 |
|---|---|
| `/wiki-init <领域名>` | 在当前目录初始化一个新知识库 |
| `/wiki-ingest <文件>` | 把一份原始资料提炼进知识库 |
| `/wiki-query <问题>` | 在知识库内查询并引用来源 |
| `/wiki-lint` | 扫描矛盾、孤立页面、死链、统计漂移等 |
| `/wiki-stats` | 页面总数与分类统计 |
| `/wiki-recent` | 读 `wiki/log.md` 显示最近变更 |

**设计原则：**

- 机制上移到本 skill，分类/词汇下沉到各仓库 CLAUDE.md
- 原始资料只读；结构化页面由 Claude 维护
- 一切引用走 `[[页面名]]` 双向链接
- 冲突保留不覆盖，标注双方来源

详见 `plugins/wiki/skills/wiki-skill/SKILL.md`。

## 目录结构

```
jinnang/
├── .claude-plugin/
│   └── marketplace.json          # marketplace 元信息
└── plugins/
    └── wiki/                     # 一个 plugin
        ├── .claude-plugin/
        │   └── plugin.json       # plugin 元信息
        └── skills/
            └── wiki-skill/       # skill 主体
                ├── SKILL.md
                ├── CHANGELOG.md
                └── templates/    # 页面 / CLAUDE.md 模板
```

新增 plugin：在 `plugins/` 下建一个新目录，添加 `.claude-plugin/plugin.json` 和 `skills/<name>/SKILL.md`，并在顶层 `marketplace.json` 的 `plugins` 数组里加一条。

## 版本策略

- 每个 plugin 的 `plugin.json` 用语义化版本：破坏性变更升 major，加规则/模板升 minor，文案升 patch
- skill 级的细粒度演进记在各 skill 自己的 `CHANGELOG.md` 里
- `marketplace.json` 的 plugin 条目可加 `"sha": "..."` 钉到具体 commit，不加则跟主分支最新

## License

MIT
