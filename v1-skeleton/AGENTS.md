# AI Knowledge Base Agent System

## 1. 项目概述

本项目是一个自动化 AI 技术动态采集与分析系统，通过定时抓取 GitHub Trending 和 Hacker News 上 AI/LLM/Agent 领域的技术内容，经 AI 结构化分析后存储为统一格式，支持多渠道（Telegram/飞书）分发。

## 2. 技术栈

- **运行时**: Python 3.12
- **Agent 框架**: OpenCode + 国产大模型
- **工作流编排**: LangGraph
- **工具生态**: OpenClaw

## 3. 编码规范

- **风格**: PEP 8
- **命名**: snake_case（变量/函数/方法）、PascalCase（类名）
- **文档**: Google 风格 docstring
- **日志**: 禁止裸 `print()`，统一使用 `logging` 模块
- **类型**: 推荐使用 type hints，公共接口必须标注

## 4. 项目结构

```
.
├── .opencode/
│   ├── agents/           # Agent 角色定义与配置
│   ├── skills/            # 可复用技能模块
│   └── config.yaml        # OpenCode 配置
├── knowledge/
│   ├── raw/               # 原始采集数据（JSON Lines）
│   └── articles/          # 结构化处理后的知识条目
├── src/
│   ├── collectors/        # 采集器（GitHub/HN）
│   ├── analyzers/         # AI 分析模块
│   ├── organizers/        # 知识整理模块
│   └── distributors/      # 分发渠道（Telegram/飞书）
├── tests/
├── pyproject.toml
└── AGENTS.md
```

## 5. 知识条目 JSON 格式

```json
{
  "id": "uuid-v4",
  "title": "string (必填, 最多 200 字符)",
  "source_url": "string (必填, 有效 URL)",
  "source_type": "github_trending | hacker_news",
  "summary": "string (必填, 100-500 字, AI 生成)",
  "tags": ["string"],
  "author": "string (可选)",
  "published_at": "ISO 8601 时间戳",
  "collected_at": "ISO 8601 时间戳",
  "status": "raw | analyzed | published | archived",
  "priority": "high | medium | low",
  "metadata": {}
}
```

### status 流转

```
raw → analyzed → published
                  ↘ archived
```

## 6. Agent 角色概览

| 角色 | 职责 | 输入 | 输出 |
|------|------|------|------|
| **Collector** | 从 GitHub Trending / HN 采集 AI 相关内容 | 定时触发 / 手动触发 | `knowledge/raw/*.jsonl` |
| **Analyzer** | AI 分析内容，提取摘要、标签、优先级 | `knowledge/raw/` | `knowledge/articles/` |
| **Organizer** | 去重、关联、更新状态、触发分发 | `knowledge/articles/` | 状态更新 + 分发事件 |

## 7. Agent 配置文件

Agent 配置位于 `.opencode/agents/` 目录，使用 YAML 格式定义：

### 7.1 Collector (采集器)
```yaml
name: collector
triggers:
  - type: schedule
    cron: "0 */6 * * *"  # 每6小时执行
  - type: manual
input:
  source: [github_trending, hacker_news]
  filter:
    keywords: [AI, LLM, Agent, GPT, Claude, LangChain, RAG, embedding]
output:
  path: knowledge/raw/{source_type}_{timestamp}.jsonl
config:
  github_trending:
    url: https://github.com/trending
    language: python
  hacker_news:
    url: https://news.ycombinator.com/
    max_items: 50
```

### 7.2 Analyzer (分析器)
```yaml
name: analyzer
trigger:
  event: raw_data_collected
input:
  path: knowledge/raw/*.jsonl
output:
  path: knowledge/articles/{id}.json
config:
  model: gpt-4o-mini
  prompt_template: (提取 summary, tags, priority, author)
```

### 7.3 Organizer (整理器)
```yaml
name: organizer
trigger:
  event: article_analyzed
input:
  path: knowledge/articles/*.json
output:
  - status_update
  - distribution_event
config:
  deduplication:
    key: source_url
  distribution:
    telegram: { enabled: true, threshold: medium }
    feishu: { enabled: true, threshold: high }
```

## 8. 红线（绝对禁止）

1. **禁止修改已发布的 `knowledge/articles/` 条目** — 只可追加，不可覆盖
2. **禁止在代码中硬编码密钥/Token** — 必须使用环境变量或 secrets 管理
3. **禁止向外部服务发送未经授权的用户数据**
4. **禁止在 `raw` 目录执行删除操作** — 仅追加写入
5. **禁止跳过 AI 分析环节直接分发** — 必须经过 Analyzer 处理
6. **禁止修改其他 Agent 的输出** — Agent 间通过事件总线交互，不得直接编辑对方文件
