---
name: github-trending
description: 当需要采集 GitHub 热门开源项目时使用此技能
allowed-tools:
  - Read
  - Grep
  - Glob
  - WebFetch
---

# GitHub Trending 采集技能

## 使用场景

当需要采集 GitHub 热门开源项目，特别是 AI/LLM/Agent 领域的技术动态时使用此技能。通过定期抓取 GitHub Trending 页面，过滤筛选有价值的技术项目，并生成结构化的知识条目。

## 执行步骤

1. **搜索热门仓库**：使用 GitHub API 或 WebFetch 抓取 GitHub Trending 页面，获取当前热门开源项目列表
2. **提取信息**：从抓取的数据中提取项目名称、URL、星标数、编程语言、主题标签等基本信息
3. **过滤**：按关键词过滤，纳入 AI/LLM/Agent 相关的项目（GPT、Claude、LangChain、RAG、embedding 等），排除 Awesome 列表类项目和纯粹的资源合集
4. **去重**：基于项目 URL 或名称进行去重，避免重复采集相同项目
5. **撰写中文摘要**：为每个项目撰写简洁的中文摘要，遵循公式「项目名 + 做什么 + 为什么值得关注」
6. **排序取 Top 15**：按星标数或综合热度排序，选取排名前 15 的项目
7. **输出 JSON**：将结果按指定 JSON 结构输出到 `knowledge/raw/github-trending-YYYY-MM-DD.json` 文件

## 注意事项

- 仅采集 AI/LLM/Agent 相关领域的项目，确保内容质量
- 排除列表类项目（如 awesome-xxx），聚焦实际工具和框架
- 摘要需使用中文，控制在 50-100 字以内
- 星标数需真实反映项目热度，优先选取近期增长迅速的项目
- 输出的 JSON 文件采用追加写入方式，不得覆盖已有数据

## 输出格式

```json
{
  "source": "github_trending",
  "skill": "github-trending",
  "collected_at": "2026-06-03T12:00:00Z",
  "items": [
    {
      "name": "project-name",
      "url": "https://github.com/author/project",
      "summary": "项目名称是一个用于XXX的工具，值得关注是因为它提供了YYY功能。",
      "stars": 12345,
      "language": "Python",
      "topics": ["llm", "rag", "agent"]
    }
  ]
}
```
