# DramaWave Engineering Wiki

这个仓库用于沉淀 DramaWave Android 工程知识库。

定位：
- DramaWave 外部独立 wiki 仓库
- Obsidian vault
- Git/GitHub 版本化知识库

仓库路径：
- 本地：`/Users/tinycoder/wiki`
- 远端：`https://github.com/TinyCodar/drama-wiki.git`

推荐使用方式：
1. 在 Obsidian 中打开 `/Users/tinycoder/wiki`
2. 从 `index.md` 开始浏览 DramaWave 工程知识图谱
3. 新增或更新页面时，同步维护 `SCHEMA.md`、`index.md`、`log.md`
4. 所有正式工程知识优先围绕 DramaWave 代码库展开，不再维护通用 llm-wiki 初始化说明页
5. 使用 Git 提交与同步远端

当前目录约定：
- `overviews/`：工程级与跨模块总览
- `inventory/`：模块/文档/脚本清单
- `modules/`：单模块 deep dive
- `flows/`：运行时链路与系统流程
- `_meta/`：仓库自身使用说明
- `.obsidian/`：Obsidian 配置

核心入口：
- `SCHEMA.md`：DramaWave wiki 结构规范
- `index.md`：导航索引
- `log.md`：维护日志

清理原则：
- 删除与 DramaWave 无关或偏通用的早期 llm-wiki 页面
- 避免新旧两套结构长期并存
- 若旧页面内容已被新页面替代，则删除旧页而非继续保留重复副本
