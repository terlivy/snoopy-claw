# SC 当前完整架构图
## SnoopyClaw OpenClaw Architecture — 单一事实来源 (SSOT)

> **版本**: 1.2
> **更新日期**: 2026-05-02
> **维护**: SC 主脑
> **用途**: 回答架构问题时的唯一参考，避免每次回答不一致
> **本地验证**: 已与 `~/.openclaw/openclaw.json` 实际配置对比确认

---

## 一、整体架构

```
┌─────────────────────────────────────────────────────────────────────┐
│                    OpenClaw Gateway (2026.4.15)                    │
│                     端口: 18789 | 运行中                            │
└─────────────────────────────────────────────────────────────────────┘
         │                    │                    │
         ▼                    ▼                    ▼
┌─────────────┐    ┌─────────────────┐    ┌─────────────────────┐
│  Plugins    │    │  Agents (19个)   │    │  Skills (22个)     │
│  4个插件    │    │  主脑 + 18个专用  │    │  workspace技能     │
└─────────────┘    └─────────────────┘    └─────────────────────┘
```

---

## 二、已安装插件（4个）

| 插件 | 路径 | Slot | 功能 |
|------|------|------|------|
| `lossless-claw-enhanced` | `~/.openclaw/extensions/lossless-claw-enhanced` | **contextEngine** | DAG上下文压缩，SQLite持久化 |
| `memory-lancedb-pro` | `~/.openclaw/extensions/memory-lancedb-pro` | **memory** | 向量检索+BM25混合+CrossEncoder重排 |
| `openclaw-lark` | `~/.openclaw/extensions/openclaw-lark` | — | 飞书全栈集成（当前禁用）|
| `sas-engine` | `~/.openclaw/extensions/sas-engine` | — | SAS准则工具（sas_check_gate等）|

### 2.1 lossless-claw-enhanced
- **用途**: 上下文满了自动摘要，不丢信息
- **配置**: `slots.contextEngine: "lossless-claw"`
- **数据库**: `~/.openclaw/lcm.db`
- **命令**: `openclaw sessions list/status` — 部分子命令超时
- **文档**: win4r/lossless-claw-enhanced (MIT)

### 2.2 memory-lancedb-pro
- **用途**: 长期记忆向量存储+混合检索
- **Embedding**: SiliconFlow BAAI/bge-large-zh-v1.5 (1024维)
- **数据路径**: `~/.openclaw/memory/lancedb-pro/`
- **功能**: smartExtraction ON / Weibull遗忘 / 多作用域隔离
- **LLM**: deepseek-ai/DeepSeek-V3.2 (via SiliconFlow)
- **文档**: CortexReach/memory-lancedb-pro (MIT)

### 2.3 openclaw-lark
- **用途**: 飞书消息/日历/表格/文档/多维表格
- **状态**: 插件已加载，但 `entries.openclaw-lark.enabled: false`

### 2.4 sas-engine
- **用途**: SAS准则执行工具
- **工具**: sas_check_gate / sas_log_transition / sas_watchdog_check / sas_get_task_state

---

## 三、Agent 系统（19个）

### 3.1 主脑 Agent

| Agent ID | 名称 | 模型 | 飞书群 | Skills | 职责 |
|----------|------|------|--------|--------|------|
| **main** | SC主脑 | minimax/MiniMax-M2.7-highspeed | — | — | 统筹协调，CEO模式 |

### 3.2 基础辅助 Agent（4个）

| Agent ID | 名称 | 模型 | Skills | 职责 |
|----------|------|------|--------|------|
| **hr** | HR专家 | zai/glm-5-turbo | — | 招聘/入职/团队管理 |
| **doc-expert** | 文档专家 | zai/glm-4.7 | — | 文档处理/生成 |
| **weather** | 天气 | zai/glm-4.5-air | — | 天气查询 |
| **tech-news** | 科技新闻 | zai/glm-4.5-air | — | 新闻收集 |

### 3.3 SAS Agent（3个）

| Agent ID | 名称 | 模型 | Skills | 职责 |
|----------|------|------|--------|------|
| **sas-sop-expert** | SAS优化 | apimart/gpt-5.2 | — | SAS准则每日优化 |
| **sas-default** | SAS执行 | deepseek/deepseek-v4-flash | memory工具 | SAS准则执行 |
| **sas-leader** | SAS派发 | apimart/gemini-3-pro-preview | **harness-leader** | 任务分解+派发 |

### 3.4 产研 Agent（11个）v2.0

| Agent ID | 名称 | 模型 | Skills | 职责 |
|----------|------|------|--------|------|
| **requirement-analyst** | 需求分析师 | apimart/gpt-5.2 | doc-handler, memory-lancedb-pro-skill, playwright-scraper-skill, task-planner, websearch, diagram-generator | 需求分析+竞品调研 |
| **product-manager** | 产品经理 | apimart/gpt-5.2 | word-docx, doc-handler, powerpoint-pptx, excel-xlsx, task-planner, websearch, diagram-generator | 产品规划+PRD撰写 |
| **technical-architect** | 技术架构师 | apimart/gpt-5.2 | doc-handler, taskflow, github, excel-xlsx, diagram-generator | 技术方案设计 |
| **ui-designer** | UI设计师 | apimart/gemini-3-pro-preview | powerpoint-pptx, diagram-generator, playwright-scraper-skill, frontend-design-3 | 界面设计 |
| **developer** | 开发工程师 | apimart/gpt-5.2-codex | github, taskflow, websearch, diagram-generator, gh-issues | 代码开发 |
| **code-reviewer** | 代码评审 | apimart/gemini-3.1-pro-preview-thinking | session-logs, github, self-improving-agent, gh-issues | 代码审查 |
| **security-reviewer** | 安全审计 | apimart/gemini-3.1-pro-preview-thinking | healthcheck, session-logs, github | 安全审查 |
| **tester** | QA测试 | apimart/gpt-5.2 | session-logs, github, task-planner, playwright-scraper-skill | 测试执行 |
| **performance-tester** | 性能测试 | apimart/gpt-5.2 | system-healer, healthcheck, github | 性能测试 |
| **devops** | DevOps工程师 | apimart/gpt-5.2-codex | system-healer, healthcheck, tmux, github, taskflow, node-connect | 运维自动化 |
| **operations-agent** | 运营 | apimart/gpt-5.2 | system-healer, healthcheck, taskflow, node-connect | 运营支持 |

---

## 四、模型配置（5个Provider）

| Provider | 模型数 | API | 主要用途 |
|---------|--------|-----|---------|
| **apimart** | 4 | openai-completions | 产研Agent主力（GPT-5.2/Gemini/Codex） |
| **zai** | 3 | openai-completions | 基础辅助Agent（GLM系列） |
| **deepseek** | 4 | openai-completions | SAS执行/备用编程 |
| **siliconflow** | 4 | openai-completions | 免费备用（DeepSeek-V3等） |
| **minimax** | 4 | openai-completions | 主脑高速推理 |

### 模型能力表

| 模型 | Provider | 上下文 | 特长 |
|------|----------|--------|------|
| **MiniMax-M2.7-highspeed** | minimax | 200K | 主脑，复杂推理，极速 |
| **GLM-5-Turbo** | zai | 200K | HR/文档，深度定制OpenClaw |
| **GLM-4.7** | zai | 128K | 文档专家 |
| **DeepSeek-V3.2** | siliconflow | 128K | embedding/LLM backbone |
| **GPT-5.2** | apimart | 200K | 产研Agent主力 |
| **Gemini-3-Pro-Preview** | apimart | 200K | 复杂推理 |
| **GPT-5.2-codex** | apimart | 200K | 编程 |
| **Gemini-3.1-Pro-Preview-Thinking** | apimart | 200K | 深度思考任务 |

---

## 五、Workspace Skills（22个）

### 核心运维类（8个）
| Skill | 功能 |
|-------|------|
| `system-healer` | 自愈机制（Ollama/Gateway/端口检查+修复） |
| `workspace-manager` | 工作区审计、优化、维护 |
| `harness-leader` | 多Agent协作编排（sas-leader使用）|
| `sas-default` | SAS准则执行 |
| `sas-task-planner` | 任务全生命周期管理 |
| `task-planner` | 通用任务规划 |
| `self-improving-agent` | 持续改进 |
| `clawteam` | 多Agent协作（Python CLI）|

### 文档处理类（5个）
| Skill | 功能 |
|-------|------|
| `word-docx` | Word文档 |
| `excel-xlsx` | Excel处理 |
| `powerpoint-pptx` | PPT处理 |
| `doc-handler` | Word/PDF/Excel综合 |
| `official-document-template` | 公文排版（GB/T 9704-2012）|

### 设计与爬虫类（4个）
| Skill | 功能 |
|-------|------|
| `frontend-design-3` | 前端界面生成 |
| `diagram-generator` | 图表生成 |
| `playwright-scraper-skill` | 网页爬取 |
| `graphify` | 知识图谱生成 |

### 工具类（5个）
| Skill | 功能 |
|-------|------|
| `memory-lancedb-pro-skill` | 向量记忆管理 |
| `graphify-out` | 知识图谱输出 |
| `nano-pdf` | PDF编辑 |
| `video-frames` | 视频帧提取 |
| `websearch` | 网页搜索 |

### OpenClaw 内置 Skills（50+个）
`github` / `gh-issues` / `healthcheck` / `node-connect` / `weather` / `cron` / `feishu-*` / `clawhub` 等

---

## 六、自动化系统

| 系统 | 实现方式 | 触发时间 |
|------|---------|---------|
| **心跳巡检** | HEARTBEAT.md | 每30分钟 |
| **GitHub同步** | `python3 scripts/sas_github_sync.py` | 每日02:00 |
| **会话记忆同步** | `python3 scripts/sync_session_memory.py` | 每日02:00 |
| **SAS-SOP优化** | cron触发sas-sop-expert | 每日 |
| **资讯推送** | cron触发tech-news | 每日 |
| **天气推送** | cron触发weather | 每日 |

---

## 七、网络与监控

| 服务 | 技术栈 | 端口 |
|------|--------|------|
| Gateway | OpenClaw | 18789 |
| Ollama | bge-m3 embedding | 11434 |
| ClawTeam | tmux/subprocess | — |

---

## 八、外部集成

| 集成 | 状态 | 说明 |
|------|------|------|
| **飞书** | ⚠️ 插件已装但禁用 | channel已配置，entries.openclaw-lark.enabled: false |
| **GitHub** | ✅ CLI已集成 | `gh`命令可用 |
| **YouTube** | ✅ yt-dlp | cookies工具链 |
| **ClawTeam** | ✅ 已安装 | Python CLI |

---

## 九、snoopy-evolver 模块

路径：`~/.openclaw/workspace/snoopy-evolver/` | GitHub: `terlivy/SAS.git`

| 模块 | 文件 | 功能 |
|------|------|------|
| **P0 Ops** | `ops/health_check.py` | 6项健康检查矩阵 |
| **P1 基因库** | `evolver/genes/genes.json` | 基因管理 |
| **P1 选择器** | `evolver/selector.py` | 信号→基因匹配 |
| **P2 审计** | `evolver/events/logger.py` | EvolutionEvent日志 |
| **P3 共享** | `evolver/clawteam_integration.py` | 跨Agent经验共享 |

---

## 十、关键配置文件

| 文件 | 用途 |
|------|------|
| `~/.openclaw/openclaw.json` | Gateway主配置（agents/plugins/models/channels）|
| `~/.openclaw/workspace/SOUL.md` | SC核心身份+CEO模式 |
| `~/.openclaw/workspace/MEMORY.md` | 记忆索引+红线规则 |
| `~/.openclaw/workspace/HEARTBEAT.md` | 心跳巡检规则 |
| `~/.openclaw/workspace/AGENTS.md` | 工作区规范 |
| `~/.openclaw/workspace/USER.md` | 用户信息 |

---

## 十一、架构版本对照

| 版本 | 日期 | 变化 |
|------|------|------|
| 1.0 | 2026-04-22 | 初始版本（8 agents, 28 skills） |
| 1.1 | 2026-04-27 | 更新providers（apimart:180模型） |
| **1.2** | **2026-05-02** | **与本地实际配置对比修正（19 agents, 22 skills, 5 providers各4模型）** |

### 本地 vs GitHub 差异说明（v1.2修正）

| 项目 | GitHub旧版(v1.1) | 本地实际 | 差异 |
|------|------------------|---------|------|
| Agents | 8个 | **19个** | 产研v2.0新增11个agent未同步 |
| Skills | 28个 | **22个** | workspace skills实际22个 |
| Provider: zai | 6模型 | **3模型** | 修正 |
| Provider: deepseek | 4模型 | **4模型** | ✅ 一致 |
| Provider: minimax | 6模型 | **4模型** | 修正 |
| Provider: siliconflow | 4模型 | **4模型** | ✅ 一致 |
| Provider: apimart | 180模型 | **4模型** | openclaw.json仅列出4个 |
| memory-lancedb-pro | bge-m3(Ollama) | **BAAI/bge-large-zh-v1.5(SiliconFlow)** | embedding源变更 |
| smartExtraction | OFF（浪费） | **ON** | 已优化 |

---

## 十二、安全配置

```json
// ~/.openclaw/openclaw.json 关键配置

// 派发子Agent权限
"agents.defaults.subagents.allowAgents": ["*"]

// 跨Agent消息权限
"tools.sessions.visibility": "all"

// Agent间通信
"tools.agentToAgent.enabled": true
```

---

## 十三、架构一致性说明

### 为什么之前回答不一致？
因为没有单一事实来源（SSOT）。从现在起：
- **架构问题** → 以本文档为准
- **配置问题** → 以 `~/.openclaw/openclaw.json` 为准
- **代码问题** → 以实际文件为准

### 如何更新本文档？
每次架构变更（新增插件/Agent/Skill/配置修改），立即：
1. 更新本文档
2. 同步到 GitHub: `terlivy/snoopy-claw`
3. 写入铁律

---

*本文档由 SC 主脑生成，最后更新：2026-05-02*
*本地验证: 已与 ~/.openclaw/openclaw.json 对比确认*
