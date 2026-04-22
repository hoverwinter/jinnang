---
description: 初始化一个新的知识库仓库骨架
---

使用 wiki-skill 的「初始化」流程在当前目录建立知识库骨架。

领域：$ARGUMENTS

按 SKILL.md 中定义的 6 步流程执行：
1. 与用户确认领域名 / 主要语言 / 一句话描述（若 $ARGUMENTS 已提供领域，仅确认其余）
2. 建目录骨架（raw/ 和 wiki/ 及其子目录）
3. 从 `templates/CLAUDE.md` 生成仓库 CLAUDE.md，替换占位符
4. 写入 index.md / log.md / overview.md 初始内容
5. 不生成示例概念或伪造内容
6. 提示下一步可用 /wiki-ingest 摄入第一份资料
