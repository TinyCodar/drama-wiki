---
title: LLM Wiki Stack
created: 2026-04-16
updated: 2026-04-16
type: concept
tags: [llm, wiki, obsidian, github, toolchain, setup]
sources: []
---

# LLM Wiki Stack

当前本地工具栈目标是把 `llm-wiki`、Obsidian、Git/GitHub 与 Hermes 配置打通，形成可持续沉淀知识的闭环。

## Components
- `[[prompt-caching]]`
- `[[wiki-bootstrap-status]]`

## Local Paths
- Wiki/Vault: `/Users/tinycoder/wiki`
- Hermes config: `~/.hermes/config.yaml`
- Hermes env: `~/.hermes/.env`

## Workflow
1. 原始资料进入 `raw/`
2. Hermes 生成/更新实体页、概念页、比较页、查询页
3. Obsidian 用于图谱浏览、双链导航与手工补充
4. Git 记录版本
5. GitHub 承载远端同步与协作
