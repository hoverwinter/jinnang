# wiki-skill 变更记录

记录 skill 规则的演进。每次把仓库中验证过的规则提升到本 skill，或修改/移除现有规则，就在此追加一行。

格式：`- YYYY-MM-DD [类型] 变更描述（来源：xxx）`

类型：`add`（新增规则）/ `change`（修改）/ `remove`（删除）/ `template`（模板调整）

---

- 2026-04-23 [init] skill 首次建立，平台规则从 finance-knowledge-base 仓库抽取
- 2026-04-23 [add] 新增 commands/ 目录（wiki-init / wiki-ingest / wiki-query / wiki-lint / wiki-stats / wiki-recent 六个 slash command），与 skill 并存。plugin.json 版本升至 1.1.0
