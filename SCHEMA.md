# Wiki Schema

## Domain
个人/项目通用 LLM Wiki：用于沉淀技术研究、项目知识、架构图谱、调研结论、流程规范与长期可复用信息。

## Conventions
- 文件名统一使用小写英文与连字符，例如 `prompt-caching.md`
- 每个 wiki 页面都必须包含 YAML frontmatter
- 页面之间优先使用 `[[wikilinks]]`
- 新页面或更新页面后，必须同步更新 `index.md` 和 `log.md`
- 原始资料只放在 `raw/` 下，不直接修改
- 每个正式页面至少包含 2 个出站链接；若暂时不足，需补充到导航页或相关概念页

## Frontmatter
```yaml
---
title: Page Title
created: YYYY-MM-DD
updated: YYYY-MM-DD
type: entity | concept | comparison | query | summary
tags: [llm, wiki]
sources: [raw/articles/source-name.md]
---
```

## Tag Taxonomy
- llm
- wiki
- obsidian
- prompt-cache
- github
- agent
- toolchain
- project
- architecture
- workflow
- note
- research
- source
- comparison
- query
- setup
- automation
- graph
- knowledge-base
- android
- gradle
- startup
- module
- dependency
- security
- flavor
- manifest
- analytics
- network
- player
- ads
- storage

## Page Thresholds
- 同一实体/概念在 2 个以上来源中出现，或在单一来源中处于核心地位时创建独立页面
- 零散提及优先合并到已有页面
- 页面超过约 200 行时考虑拆分

## Entity Pages
包含：定义、关键事实、关联页面、来源。

## Concept Pages
包含：概念定义、适用场景、关键机制、限制、相关页面。

## Comparison Pages
包含：比较对象、维度、结论、来源。

## Update Policy
1. 新信息优先补充而不是覆盖旧信息
2. 若来源冲突，保留双方说法并注明日期与来源
3. 需要时在页面中显式标记 contradiction
