# OpenClaw 部署迁移指南

> **版本**: v1.0
> **更新日期**: 2026-05-08
> **维护**: SC 主脑
> **适用版本**: OpenClaw 2026.4.15

---

## 一、迁移原理

OpenClaw 的数据分为两部分：

| 类别 | 路径 | 说明 |
|------|------|------|
| **运行时（npm 包）** | `~/.npm-global/lib/node_modules/openclaw/` | 由 npm 管理，版本锁定 |
| **配置数据** | `~/.openclaw/` | 用户数据，与包分离 |
| **工作区** | `~/.openclaw/workspace/` | 项目代码和配置 |

迁移本质：**只迁移 `~/.openclaw/` 目录**，新机器安装同版本 npm 包后替换即可。

---

## 二、新机器标准安装步骤

### 第一步：安装 Node.js（>=18）
```bash
# Ubuntu/WSL
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# 验证
node --version  # >= 18
npm --version
```

### 第二步：安装 openclaw（锁定版本）
```bash
npm install -g openclaw@2026.4.15
openclaw --version  # 应输出 OpenClaw 2026.4.15
```

### 第三步：初始化目录结构
```bash
# 首次运行会创建 ~/.openclaw/ 目录
openclaw gateway start

# 然后停止，用旧配置覆盖
openclaw gateway stop
```

---

## 三、配置迁移清单

> 把旧机器的以下文件/目录复制到新机器对应位置

### 3.1 核心配置（必须）
| 旧机器路径 | → 新机器路径 | 说明 |
|-----------|-------------|------|
| `~/.openclaw/openclaw.json` | `~/.openclaw/openclaw.json` | 主配置文件 |
| `~/.openclaw/credentials/` | `~/.openclaw/credentials/` | API Keys、认证信息 |
| `~/.openclaw/identity/` | `~/.openclaw/identity/` | 身份配置 |

### 3.2 Agents（必须，21个）
| 旧机器路径 | → 新机器路径 |
|-----------|-------------|
| `~/.openclaw/agents/` | `~/.openclaw/agents/` |

### 3.3 Providers（必须）
providers 配置在 `openclaw.json` 的 `providers` 字段中，API key 分布在：
- `~/.openclaw/credentials/` 下的密钥文件
- 或 `openclaw.json` 中的内嵌 key

**推荐方式**：复制整个 `credentials/` 目录，再在新机器的 `openclaw.json` 中确认 provider 指向正确。

### 3.4 Skills（必须）
| 旧机器路径 | → 新机器路径 |
|-----------|-------------|
| `~/.openclaw/workspace/skills/` | `~/.openclaw/workspace/skills/` |

### 3.5 Plugins（必须）
| 旧机器路径 | → 新机器路径 |
|-----------|-------------|
| `~/.openclaw/plugins/` | `~/.openclaw/plugins/` |

### 3.6 工作区（可选，按需）
| 旧机器路径 | → 新机器路径 |
|-----------|-------------|
| `~/.openclaw/workspace/` | `~/.openclaw/workspace/` |

包含：ai-monitor、debt-tracker、scripts、MyTasks 等

---

## 四、自动化迁移脚本

### 4.1 旧机器执行：打包
```bash
cd ~
tar --exclude='.openclaw/workspace/node_modules' \
    --exclude='.openclaw/workspace/venv' \
    --exclude='.openclaw/workspace/.git' \
    --exclude='.openclaw/node_modules' \
    --exclude='.openclaw/logs' \
    --exclude='.openclaw/memory/lancedb-pro' \
    -czf openclaw-migration.tar.gz \
    .openclaw/openclaw.json \
    .openclaw/credentials/ \
    .openclaw/identity/ \
    .openclaw/agents/ \
    .openclaw/plugins/ \
    .openclaw/workspace/skills/ \
    .openclaw/workspace/scripts/ \
    .openclaw/workspace/checklists/ \
    .openclaw/workspace/memory/ \
    .openclaw/workspace/AGENTS.md \
    .openclaw/workspace/SOUL.md \
    .openclaw/workspace/MEMORY.md \
    .openclaw/workspace/IDENTITY.md \
    .openclaw/workspace/USER.md \
    .openclaw/workspace/HEARTBEAT.md \
    .openclaw/workspace/TOOLS.md
```

### 4.2 新机器执行：恢复
```bash
# 把 tar 包传到新机器后
tar -xzf openclaw-migration.tar.gz -C ~/

# 重启 Gateway
openclaw gateway restart
```

---

## 五、版本一致性检查

迁移后在新机器执行以下检查：

```bash
# 1. 检查 openclaw 版本
openclaw --version
# 必须输出: OpenClaw 2026.4.15

# 2. 检查 agent 数量
ls ~/.openclaw/agents/ | wc -l
# 必须输出: 21

# 3. 检查 skills
ls ~/.openclaw/workspace/skills/ | wc -l
# 必须输出: >=17

# 4. 启动 Gateway 并检查状态
openclaw gateway start
openclaw status

# 5. 检查 providers 是否正常
openclaw providers list
```

---

## 六、已知版本差异

| 项目 | 旧版本 | 当前版本 | 迁移注意 |
|------|--------|---------|---------|
| openclaw | < 2026.4 | **2026.4.15** | 2026.4 有破坏性变更，必须用 2026.4.15 |
| providers | 分散 | MiniMax/apimart/deepseek/zai/siliconflow | credentials/ 需同步迁移 |
| agents | 8 | 21 | 迁移 agents/ 目录即可 |
| skills | workspace-skills | 22个（workspace+内置） | 迁移 skills/ 目录 |

---

## 七、独立 GitHub 仓库（按需克隆）

迁移 workspace 后，如果需要拉取各独立仓库最新代码：

```bash
# 架构文档
git clone https://github.com/terlivy/snoopy-claw

# 演化系统
git clone https://github.com/terlivy/snoopy-evolver

# SAS 工作准则
git clone https://github.com/terlivy/SAS

# SAS 自动化脚本
git clone https://github.com/terlivy/SAS-script

# SAS 插件
git clone https://github.com/terlivy/SAS-plug-in

# SC 专属 Skills
git clone https://github.com/terlivy/snoopyclaw-skills
```

---

## 八、故障排查

| 问题 | 解决方案 |
|------|---------|
| `openclaw: command not found` | npm 全局路径未加入 PATH：`export PATH="$PATH:$(npm root -g)/bin"` |
| agents 为空 | agents/ 目录未复制，或权限问题：`chmod -R 755 ~/.openclaw/agents/` |
| providers 报错 | credentials/ 未复制，或 API key 过期 |
| Gateway 无法启动 | 检查 `~/.openclaw/openclaw.json` 语法：`python3 -c "import json; json.load(open('~/.openclaw/openclaw.json'))"` |

---

## 九、版本升级 SOP（v2.0）

> 升级前必须阅读：`~/.openclaw/workspace/checklists/openclaw-upgrade.md`
> 脚本路径：`/home/openclaw/scripts/openclaw_upgrade_*.sh`

### 升级流程
```bash
# 阶段1：预检查（必须）
bash /home/openclaw/scripts/openclaw_upgrade_precheck.sh [目标版本]
# 解读 report.json：BLOCK_UPGRADE / REVIEW_WARNINGS / READY_TO_UPGRADE

# 阶段2：执行升级（获得授权后）
bash /home/openclaw/scripts/openclaw_upgrade.sh [目标版本]
# 自动：备份 → 安装 → 验证 → 失败自动回滚

# 阶段3：人工复核
openclaw --version  # 确认版本
openclaw gateway status  # 确认进程

# 阶段4：回滚（如需要）
bash /home/openclaw/scripts/openclaw_rollback.sh auto  # 自动回滚
bash /home/openclaw/scripts/openclaw_rollback.sh 2026.4.15  # 指定版本
```

### 升级脚本说明
| 脚本 | 用途 | 退出码 |
|------|------|--------|
| `openclaw_upgrade_precheck.sh` | 升级前全面检查 | 0=通过, 1=失败, 2=警告 |
| `openclaw_upgrade.sh` | 执行升级（自动回滚） | 0=成功, 1=已回滚 |
| `openclaw_upgrade_postverify.sh` | 升级后验证 | 0=通过, 1=失败 |
| `openclaw_rollback.sh` | 一键回滚 | 0=成功, 1=失败 |

### 备份文件
- 配置备份：`~/.openclaw/backup/YYYYMMDD_HHMMSS/openclaw.json`
- npm 快照：`~/.openclaw/backup/YYYYMMDD_HHMMSS/npm_versions.txt`
- 升级日志：`/tmp/openclaw_upgrade_*.log`
- 验证报告：`/tmp/openclaw_precheck_*.json` / `/tmp/openclaw_postverify_*.json`

| skills 不生效 | 检查 `openclaw.json` 中 skills 配置路径是否正确 |
