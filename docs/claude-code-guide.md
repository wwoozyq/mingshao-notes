# Claude Code 常用操作指南

## 1. 斜杠命令速查表

### 会话与对话管理

| 命令 | 用途 |
|:---|:---|
| `/clear` | 清空上下文，开始新对话（别名 `/reset`、`/new`） |
| `/compact [指令]` | 压缩对话节省上下文，可指定压缩重点 |
| `/resume [会话]` | 恢复之前的对话（别名 `/continue`） |
| `/branch [名称]` | 从当前对话分叉出新分支（别名 `/fork`） |
| `/rewind` | 回退对话到之前某个节点（别名 `/undo`） |
| `/rename [名称]` | 重命名当前会话 |
| `/exit` | 退出 CLI |

### 模型与思考深度

| 命令 | 用途 |
|:---|:---|
| `/model [模型]` | 切换模型，左右箭头调整 effort 等级 |
| `/effort [级别]` | 设置思考深度：`low`、`medium`、`high`、`xhigh`、`max` |
| `/fast [on|off]` | 切换快速模式（更快的模型，更低延迟） |

### 权限与模式

| 命令 | 用途 |
|:---|:---|
| `/permissions` | 管理工具权限的 allow/ask/deny 规则 |
| `/plan [描述]` | 进入计划模式（只读不修改） |
| `/sandbox` | 切换沙箱模式 |

### 上下文与记忆

| 命令 | 用途 |
|:---|:---|
| `/memory` | 编辑 CLAUDE.md 记忆文件 |
| `/context` | 可视化当前上下文用量 |
| `/cost` | 显示 token 使用统计 |

### 文件与代码

| 命令 | 用途 |
|:---|:---|
| `/add-dir <路径>` | 添加额外工作目录 |
| `/diff` | 打开交互式 diff 查看器 |
| `/copy [N]` | 复制最近一条回复到剪贴板 |
| `/init` | 初始化项目 CLAUDE.md |

### 审查与分析

| 命令 | 用途 |
|:---|:---|
| `/review [PR]` | 在本地审查 PR |
| `/security-review` | 对当前分支做安全审查 |

### 配置与调试

| 命令 | 用途 |
|:---|:---|
| `/config` | 打开设置界面（主题、模型、输出风格等） |
| `/doctor` | 诊断安装问题，按 `f` 自动修复 |
| `/status` | 显示版本、模型、账号信息 |
| `/hooks` | 查看钩子配置 |
| `/mcp` | 管理 MCP 服务器连接 |
| `/theme` | 切换颜色主题 |

### 并行与自动化

| 命令 | 用途 |
|:---|:---|
| `/batch <指令>` | 大规模并行修改（5-30 个隔离 worktree） |
| `/loop [间隔] [指令]` | 定时重复执行指令 |
| `/tasks` | 查看后台任务列表 |

---

## 2. 键盘快捷键

### 基本控制

| 快捷键 | 说明 |
|:---|:---|
| `Ctrl+C` | 取消当前输入或生成 |
| `Ctrl+D` | 退出会话 |
| `Ctrl+L` | 清屏重绘 |
| `Ctrl+G` / `Ctrl+X Ctrl+E` | 在默认编辑器中编辑提示词 |
| `Ctrl+O` | 切换 transcript 查看器（详细工具调用） |
| `Ctrl+R` | 反向搜索命令历史 |
| `Ctrl+B` | 将后台命令/agent 移到后台运行 |
| `Ctrl+T` | 切换任务列表显示 |
| `Esc` + `Esc` | 回退对话（rewind） |
| `Shift+Tab` | 循环切换权限模式 |

### 模型切换

| 快捷键 | 说明 |
|:---|:---|
| `Option+P` (macOS) / `Alt+P` | 切换模型 |
| `Option+T` / `Alt+T` | 切换扩展思考 |
| `Option+O` / `Alt+O` | 切换快速模式 |

### 多行输入

| 方式 | 快捷键 |
|:---|:---|
| 反斜杠换行 | `\` + `Enter` |
| Option 换行 | `Option+Enter`（需配置 Meta） |
| Shift+Enter | iTerm2/WezTerm/Ghostty 等原生支持 |
| Ctrl 换行 | `Ctrl+J`（任何终端可用） |
| 粘贴模式 | 直接粘贴多行文本 |

### 快捷前缀

| 前缀 | 说明 |
|:---|:---|
| `/` | 斜杠命令 |
| `!` | Bash 模式——直接运行命令并添加输出到会话 |
| `@` | 文件路径引用——触发路径自动补全 |

### 粘贴图片

| 快捷键 | 说明 |
|:---|:---|
| `Ctrl+V` / `Cmd+V` (iTerm2) / `Alt+V` | 从剪贴板粘贴图片 |

---

## 3. 核心工作流

### 提交代码

直接说 "commit these changes" 或 "commit my work"。Claude 会自动暂存文件、撰写提交信息并执行 `git commit`。

### 创建 PR

1. 直接说 "create a PR for my changes"
2. 也可以分步：先 "summarize my changes"，再 "create a PR"
3. 创建后用 `claude --from-pr <编号>` 继续跟进

### 审查 PR

- `/review [PR]` 本地审查
- `/security-review` 安全审查
- `/autofix-pr` 自动监控 PR 并修复 CI 失败

### Worktree 并行工作

- `claude --worktree feature-auth` 创建隔离工作树
- 可同时开多个 session 各自修改不同功能
- 工作树位于 `<repo>/.claude/worktrees/<名称>`

### 非交互式使用（管道模式）

```bash
cat file | claude -p "解释这段代码"
claude -p "explain this function"            # 单次查询后退出
claude -p --output-format json "query"       # JSON 格式输出
claude -p --max-turns 3 "query"              # 限制 agentic 轮数
claude -p --max-budget-usd 5.00 "query"      # 限制花费
```

---

## 4. 权限模式详解

| 模式 | 自动放行范围 | 适用场景 |
|:---|:---|:---|
| `default` | 只读操作 | 入门、敏感工作 |
| `acceptEdits` | 读取 + 文件编辑 + 常见文件命令 | 日常写代码 |
| `plan` | 只读（不做任何修改） | 探索规划阶段 |
| `auto` | 所有操作（后台安全检查） | 长任务、减少打断 |
| `dontAsk` | 仅预先批准的工具 | CI/脚本 |
| `bypassPermissions` | 除保护路径外全部放行 | 容器/VM 隔离环境 |

**切换方式**：按 `Shift+Tab` 循环切换，或启动时 `claude --permission-mode plan`。

**权限规则语法**：
- `Bash(npm run build)` — 精确匹配命令
- `Bash(npm run *)` — 通配符匹配
- `Read(./.env)` — 匹配读取 .env
- `Edit(/src/**/*.ts)` — 匹配编辑 src 下 TS 文件
- `mcp__puppeteer__*` — 匹配 MCP 服务器所有工具

---

## 5. 设置文件层级

| 层级 | 位置 | 影响范围 | 可共享？ |
|:---|:---|:---|:---|
| 托管 | 管理员配置 | 全机器所有用户 | 是（IT 部署） |
| 用户 | `~/.claude/settings.json` | 你，所有项目 | 否 |
| 项目共享 | `.claude/settings.json` | 所有协作者 | 是（提交到 git） |
| 项目本地 | `.claude/settings.local.json` | 你，仅此项目 | 否（gitignored） |

**优先级**（高→低）：托管 > CLI 参数 > 项目本地 > 项目共享 > 用户

---

## 6. 实用技巧

### 上下文管理

- **主动 `/compact`**：上下文快满时压缩，可指定重点如 `/compact focus on auth module`
- **`/context`** 可视化 token 用量，了解上下文消耗
- **切换任务用 `/clear`**：旧对话可通过 `/resume` 找回
- **`/btw`** 旁注提问：不进入对话历史，适合快速小问题

### 多文件编辑

- 用 `@` 引用文件：`explain the logic in @src/utils/auth.js`
- 引用多个文件：`@file1.js and @file2.js`
- 引用目录：`what is the structure of @src/components?`

### 减少权限弹窗

- 用 `Shift+Tab` 切换到 `acceptEdits` 模式
- 在 `settings.json` 预批准常用命令：
  ```json
  "permissions": {
    "allow": ["Bash(npm run test *)", "Bash(git diff *)"]
  }
  ```

### 会话命名与恢复

- 启动时命名：`claude -n "payment-integration"`
- 会话中改名：`/rename auth-refactor`
- 恢复会话：`claude --resume auth-refactor` 或 `claude --continue`
- 从 PR 恢复：`claude --from-pr 123`

### 高效习惯

- 复杂任务先用 `/plan` 规划再动手
- 用 `/init` 创建 CLAUDE.md 让后续会话了解项目惯例
- 用 `/simplify` 检查最近改动的代码质量
- 长时间运行用 `Ctrl+B` 移到后台
- 设置 `alwaysThinkingEnabled: true` 常驻扩展推理