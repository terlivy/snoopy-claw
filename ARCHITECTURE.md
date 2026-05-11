# SnoopyClaw 架构文档

> **版本**：v1.5（本地更新中，待推送）
> **更新**：2026-05-12
> **Gateway**：2026.5.10-beta.3 (6d7dcd9)
> **本文档**：唯一真实来源（SSOT），每次架构变更后同步到 GitHub

---

## 核心身份

| 属性 | 值 |
|------|-----|
| **名称** | SnoopyClaw（SC，代号 9527） |
| **物种** | AI 伙伴 |
| **Vibe** | 好奇、勤奋、直接、可靠、有点技术范儿 |
| **Emoji** | 🐾🦞 |
| **主脑模型** | MiniMax-M2.7-highspeed（minimax provider） |
| **主脑 Token Plan** | MiniMax Token Plan |
| **Gateway 端口** | 18789（loopback） |
| **Gateway Token** | local-dev-token |

---

## 插件系统（8/99 启用）

| 插件 | ID | 版本 | 状态 | 来源 |
|------|----|------|------|------|
| SAS Engine | sas-engine | 1.0.0 | ✅ enabled | 本地扩展 |
| lossless-claw-enhanced | lossless-claw | 0.5.2 | ✅ enabled | win4r fork（martian-engineering/lossless-claw 0.5.2 + CJK修复） |
| Memory LanceDB Pro | memory-lancedb-pro | 1.1.0-beta.9 | ✅ enabled | 本地扩展 |
| Browser | browser | 2026.5.10-beta.3 | ✅ enabled | stock |
| DeepSeek | deepseek | 内置 | ✅ enabled | stock provider |
| MiniMax | minimax | 内置 | ✅ enabled | stock provider |
| ZAI Provider | zai | 内置 | ✅ enabled | stock provider |
| Feishu（Lark） | lark | 2026.5.7 | ✅ enabled | 飞书插件 |

> ⚠️ stock `active-memory` 已禁用（使用 memory-lancedb-pro 替代）

### lossless-claw-enhanced 详情

- **Fork 来源**：win4r/lossless-claw-enhanced（git remote 已设置）
- **源码**：~/.openclaw/extensions/lossless-claw-enhanced/
- **CJK Token 修正**：CJK 字符 1.5x（原本 0.25 → 1.5），Emoji 2.0x
- **ContextEngine Slot**：✅ 已配置（`plugins.slots.contextEngine: lossless-claw`）
- **编译产物**：dist/estimate-tokens.js 已确认含 CJK 1.5x 修正

### memory-lancedb-pro 详情

- **数据目录**：~/.openclaw/memory/lancedb-pro/memories.lance/
- **Embedding**：SiliconFlow BAAI/bge-large-zh-v1.5（1024 维）
- **检索模式**：hybrid（BM25 权重 0.3 + 向量 0.7）
- **smartExtraction**：✅ ON
- **autoCapture / autoRecall**：✅ 全开
- **Skill 配置**：~/.openclaw/workspace/skills/memory-lancedb-pro-skill/SKILL.md
- **Plan A-D**：支持自定义配置（embedding provider/apiKey/model/baseURL/dimensions、reranker、decay 等）

### 插件 Slots 配置

```json
"plugins": {
  "slots": {
    "memory": "memory-lancedb-pro",
    "contextEngine": "lossless-claw"
  }
}
```

---

## 模型配置（5 Providers）

### Provider 总览

| Provider | API 类型 | 模型数 | 主要用途 |
|----------|---------|--------|---------|
| **apimart** | openai-completions | 180 | 产研 Agent 主力（Gpt-5.2/Codex/Claude/Gemini） |
| **deepseek** | openai-completions | 4 | 编程/推理（DeepSeek-V4-Pro/V4-Flash/Chat/Reasoner） |
| **minimax** | openai-completions | 6 | 主脑高速推理（M2.7/M2.5/M2.1/M2） |
| **siliconflow** | openai-completions | 4 | 免费备用（DeepSeek-V3/Qwen3.5-397B/Qwen3-Coder-480B） |
| **zai** | openai-completions | 6 | 备用/轻量任务（GLM-5-Turbo/4.7/4.5-Air） |

### apimart 模型列表（180 个）

```
GPT 系列：gpt-5, gpt-5.2, gpt-5.2-codex, gpt-5.2-pro, gpt-5.3-codex, gpt-5.4,
         gpt-4o, gpt-4.1, gpt-4.1-mini, gpt-4.1-nano, gpt-3.5-turbo
Claude 系列：claude-opus-4.6, claude-opus-4.6-thinking, claude-sonnet-4.6, claude-sonnet-4.6-thinking,
             claude-haiku-4-5
Gemini 系列：gemini-3-pro-preview, gemini-3-pro-preview-thinking, gemini-3.1-pro-preview,
             gemini-2.5-pro, gemini-2.5-flash
MiniMax 系列：minimax-m2.7, minimax-m2.5, minimax-m2.1, minimax-m2
DeepSeek 系列：deepseek-v4-pro, deepseek-v4-flash, deepseek-v3.2, deepseek-v3.1-terminus,
               deepseek-r1
其他：kimi-k2, kimi-k2.5, doubao-seedance, qwen-image, icl-ocr, flux-2-pro,
      wan2.5/2.6/2.7, kling-video, sora-2, imagen-4, whisper-1,
      text-embedding-3-small/large
```

### deepseek Provider（2 个）

```
deepseek-v4-pro, deepseek-v4-flash
```

> 注：`deepseek-chat`（DeepSeek V3）和 `deepseek-reasoner`（DeepSeek R1）已从 DeepSeek API 移除，仅保留 v4 系列。

### minimax Provider（6 个）

```
MiniMax-M2.7, MiniMax-M2.7-highspeed, MiniMax-M2.5, MiniMax-M2.5-highspeed,
MiniMax-M2.1, MiniMax-M2
```

### siliconflow Provider（4 个）

```
deepseek-ai/DeepSeek-V3, deepseek-ai/DeepSeek-R1,
Qwen/Qwen3.5-397B-A17B, Qwen/Qwen3-Coder-480B-A35B-Instruct
```

### zai Provider（6 个）

```
glm-5-Turbo, glm-4.7, glm-4.7-flash, glm-4.7-flashx, glm-4.5-air, glm-4.6v
```

---

## Agent 系统（20 个）

### Agent 索引

| Agent | 模型 | 职责 | Skills 数量 |
|-------|------|------|------------|
| main（SC） | minimax/MiniMax-M2.7-highspeed | 主脑，统筹协调 | 6（内置） |
| weather | zai/glm-4.5-air | 天气查询 | 0 |
| hr | zai/glm-5-Turbo | 招聘、入职、团队管理 | 0 |
| tech-news | zai/glm-4.5-air | 技术新闻收集 | 0 |
| doc-expert | zai/glm-4.7 | 文档生成 | 0 |
| sas-sop-expert | apimart/gpt-5.2 | SAS 准则优化 | 0 |
| sas-default | deepseek/deepseek-v4-flash | 默认 SAS 执行 | 4 |
| sas-leader | apimart/gemini-3-pro-preview | SAS Leader 协调 | 1 |
| **prod-leader** | apimart/gemini-3-pro-preview | 产研 Leader 协调 | 2 |
| requirement-analyst | apimart/gpt-5.2 | 需求分析 | 7 |
| product-manager | apimart/gpt-5.2 | 产品规划 | 7 |
| technical-architect | apimart/gpt-5.2 | 技术架构 | 6 |
| ui-designer | apimart/gemini-3-pro-preview | UI/UX 设计 | 2 |
| developer | apimart/gpt-5.2-codex | 编码实现 | 7 |
| code-reviewer | apimart/claude-sonnet-4.6-thinking | 代码审查 | 5 |
| security-reviewer | apimart/claude-opus-4.6-thinking | 安全审查 | 3 |
| tester | apimart/gpt-5.2 | 功能测试 | 5 |
| performance-tester | apimart/gpt-5.2 | 性能测试 | 4 |
| devops | apimart/gpt-5.2-codex | DevOps | 5 |
| operations-agent | apimart/gpt-5.2 | 运维监控 | 5 |

> **prod-leader**（2026-05-12 新注册）：统筹 11 个产研 Agent，使用 sessions_yield 分配子任务，workspace 隔离。

### Agent 模型分配原则

| 任务类型 | 推荐模型 | Provider |
|---------|---------|---------|
| 复杂推理/规划 | MiniMax-M2.7-highspeed | minimax |
| 产研核心（GPT 系） | GPT-5.2 / GPT-5.2-codex | apimart |
| 代码审查（深度） | Claude Sonnet 4.6 Thinking | apimart |
| 安全审查（深度） | Claude Opus 4.6 Thinking | apimart |
| UI/视觉设计 | Gemini 3 Pro | apimart |
| 编程/代码 | DeepSeek-V4-Pro | deepseek |
| 文档生成 | GLM-4.7 | zai |
| 轻量查询 | GLM-4.5-Air | zai |
| Leader 协调 | Gemini 3 Pro Preview | apimart |
| 备用/免费 | DeepSeek-V3 / Qwen3.5-397B | siliconflow |

### Agent 工具授权（alsoAllow）

所有 Agent 默认 alsoAllow：
- memory_recall, memory_store, memory_forget
- task_decompose

额外授权：
- sas-engine 工具：sas_check_gate, sas_log_transition, sas_watchdog_check, sas_get_task_state
- sas-leader：exec
- developer/code-reviewer/security-reviewer/tester 等：见各 agent.json

---

## Skills 系统（22 个）

### Skills 目录

```
~/.openclaw/workspace/skills/
├── memory-lancedb-pro-skill/    # memory-lancedb-pro 配置 Skill
├── harness-leader/              # Leader 能力 Skill
├── system-healer/                # 自愈机制 Skill
├── workspace-manager/            # 工作区管理 Skill
├── sas-default/                  # SAS 默认执行 Skill
├── sas-task-planner/             # SAS 任务规划 Skill
├── task-planner/                 # 通用任务规划
├── self-improving-agent/         # 自优化 Agent
├── skill-vetter/                 # Skill 审查
├── summarize-pro/                # 强化摘要
├── clawteam/                     # 团队协作
├── excel-xlsx/                   # Excel 处理
├── word-docx/                   # Word 处理
├── powerpoint-pptx/             # PPT 处理
├── diagram-generator/            # 图表生成
├── frontend-design-3/            # 前端设计
├── playwright-scraper-skill/     # 网页爬取
├── websearch/                    # 网页搜索
├── doc-handler/                  # 文档处理
├── graphify-out/                  # （其他）
├── free-ride/                    # 自由乘驾
├── official-document-template/   # 官文档模板
```

### P0/P1 Skill 绑定（产研 Agent v2.0）

| Agent | P0 Skills | P1 Skills |
|-------|----------|----------|
| requirement-analyst | websearch, summarize, doc-handler, task-planner | diagram-generator, playwright-scraper, memory-lancedb-pro-skill |
| product-manager | word-docx, pptx, excel-xlsx, diagram-generator, websearch | doc-handler, task-planner |
| technical-architect | diagram-generator, coding-agent, github | excel-xlsx, mcporter, taskflow |
| ui-designer | frontend-design-3, diagram-generator | powerpoint-pptx, playwright-scraper |
| developer | coding-agent, github | gh-issues, diagram-generator, taskflow, websearch |
| code-reviewer | coding-agent, github | gh-issues, session-logs, self-improving-agent |
| security-reviewer | healthcheck, github | 1password, session-logs, postdoc-anticheat |
| tester | coding-agent, playwright-scraper, github | task-planner, session-logs |
| performance-tester | healthcheck, system-healer | model-usage, coding-agent, github |
| devops | healthcheck, system-healer, github | tmux, taskflow, node-connect |
| operations-agent | healthcheck, system-healer | blogwatcher, node-connect, taskflow, model-usage |

---

## 自动化系统

### SAS 工作准则（v1.8）

- **方法论**："慢即是快"，六阶段门控（接收→计划→执行→检查→交付→归档）
- **核心工具**（sas-engine 插件）：
  - sas_check_gate：阶段门审批
  - sas_log_transition：阶段转换记录
  - sas_watchdog_check：看门狗检查
  - sas_get_task_state：任务状态查询
- **SAS-SOP 专家**：每天优化 SAS 准则，SC 审核后执行

### 自愈机制

- **systemd OnFailure** → openclaw-fix.sh（自动检查修复 Gateway）
- **健康检查脚本**：snoopy_evolver/ops/health_check.py（P0 进程检查、磁盘、context 使用）

### 记忆系统

- **向量数据库**：LanceDB Pro（~/.openclaw/memory/lancedb-pro/）
- **每日同步**：02:00 cron 自动全渠道同步（sync_session_memory.py）
- **SmartExtraction**：ON（LLM 自动 6 类提取）

### AI Monitor

- **后端**：FastAPI + SQLite（端口 8000）
- **前端**：Vue + Tailwind CSS（端口 3000）
- **任务接入**：Sessions As Tasks 融合层（后端 state.py）
- **数据源**：Gateway WebSocket sessions + SQLite tasks

---

## snoopy-evolver 模块

| 子模块 | 功能 |
|--------|------|
| agent_tracker/ | 子 Agent 性能追踪（spawn 记录、质量评分） |
| evolver/events/ | 事件日志（task_started/completed/failed/gene_applied） |
| evolver/signals/ | 信号驱动（task_complete/failure/git_push/migration） |
| evolver/genes/ | 基因库（记忆模式、最佳实践） |
| skill_router/ | Skill 路由匹配（查询 inventory.yaml） |
| ops/health_check.py | P0 Ops 健康检查 |

---

## GitHub 仓库

| 仓库 | 内容 |
|------|------|
| terlivy/snoopy-claw | 主架构文档（ARCHITECTURE.md） |
| SAS.git | SAS 准则文档 |
| SAS-script.git | 脚本目录（/home/openclaw/scripts） |
| SAS-plug-in.git | sas-engine 插件代码 |
| snoopyclaw-skills.git | Skills 资产 |

---

## 版本历史

| 版本 | 日期 | 变更 |
|------|------|------|
| v1.4 | 2026-05-09 | 初始版本，19 agents，22 skills |
| v1.5 | 2026-05-12 | 升级到 OpenClaw 2026.5.10-beta.3；新增 prod-leader（20 agents）；lossless-claw-enhanced CJK 1.5x；memory-lancedb-pro beta.9；slots 补全 contextEngine；providers 调整为 5 个（zai 替代 ollama）； |

---

*本文档为唯一真实来源（SSOT）。每次架构变更后同步到 GitHub。*
