---
title: Prompt Caching
created: 2026-04-16
updated: 2026-04-16
type: concept
tags: [llm, prompt-cache, agent, workflow]
sources: []
---

# Prompt Caching

Hermes 当前已经内建 prompt cache 相关能力，但是否生效取决于模型/提供方。

## Current Situation
- Claude / Anthropic 路线可通过 `cache_control` 使用缓存
- OpenAI/Codex Responses 路线会传递 `prompt_cache_key`
- 为了保持缓存命中率，不应频繁在同一会话中改动系统提示或工具集

## Operational Rules
- 尽量复用同一会话处理连续任务
- 尽量避免无谓切换模型/提供方
- 技能和配置变更通常在新会话里生效更稳妥

## Related
- [[llm-wiki-stack]]
- [[wiki-bootstrap-status]]
