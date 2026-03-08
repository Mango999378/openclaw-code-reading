## OpenClaw 关键模块与目录分布

* **CLI (命令行)**
  * 入口脚本: `src/entry.ts`
  * 命令连接: `src/cli/`
  * Gateway CLI 文档: `docs/cli/gateway.md`
* **Gateway (网关)**
  * 网关启动与核心运行时: `src/gateway/` (入口文件为 `src/gateway/server.impl.ts`)
  * WebSocket 运行时与方法: 位于 `src/gateway/server-*` 相关模块中
* **Channels (渠道)**
  * 核心渠道实现: 位于各渠道专属文件夹下 (例如 `src/telegram`, `src/discord`, `src/imessage`, `src/web` 等)
  * 共享渠道逻辑与路由辅助: `src/channels/`
* **Auto-reply / agent turns (自动回复 / 代理轮次)**
  * 回复管道: 位于 `src/auto-reply/`
  * 核心编排器: `src/auto-reply/reply/agent-runner.ts` 负责运行轮次、管理会话更新、打字信号、工具输出规则和后续行动
* **Config schema (配置模式)**
  * 类型定义: 从 `src/config/types.ts` 导出 (拆分为多个 `types.*.ts` 文件)
* **Daemon (守护进程)**
  * 目录: `src/daemon/`
  * 功能: 负责跨平台服务管理 (Linux 的 systemd、macOS 的 launchd、Windows 的 schtasks)，将 Gateway 作为后台服务处理其生命周期
* **Docs (文档)**
  * 目录: `src/docs/`
  * 功能: 包含文档系统的辅助程序和实用工具
* **Memory (记忆系统)**
  * 目录: `src/memory/`
  * 功能: 提供持久化知识层，结合 SQLite + sqlite-vec 存储、Markdown 分块、嵌入模型提供商和混合搜索功能
* **Plugins (插件)**
  * 目录: `src/plugins/`
  * 功能: 通过 jiti 加载插件、注册钩子并集成插件市场
* **Security (安全)**
  * 目录: `src/security/`
  * 功能: 包含安全审计 (`openclaw security audit`)、扫描路径辅助程序和修复应用
* **Browser (浏览器控制)**
  * 目录: `src/browser/`
  * 功能: 通过 Playwright CDP 进行 Chromium 自动化，包括屏幕截图标准化、AX 树遍历和扩展中继
* **TTS (文字转语音)**
  * 目录: `src/tts/`
  * 功能: 通过 ElevenLabs/OpenAI/Edge TTS API 提供文字转语音功能
* **Cron (定时任务)**
  * 目录: `src/cron/`
  * 功能: 通过计时器循环和运行日志记录来执行计划任务
* **Media (多媒体)**
  * 目录: `src/media/`
  * 功能: 负责媒体获取、图像优化、输入文件处理和 MIME 检测
* **Hooks (钩子)**
  * 目录: `src/hooks/`
  * 功能: 包含系统捆绑的钩子 (命令记录器、会话记忆) 以及用户可配置的钩子
* **TUI (终端用户界面)**
  * 目录: `src/tui/`
  * 功能: 包含用于 CLI 交互的终端 UI 组件
* **Link understanding (链接理解)**
  * 目录: `src/link-understanding/`
  * 功能: 负责 URL 内容提取和总结
* **Canvas host (画布宿主)**
  * 目录: `src/canvas-host/`
  * 功能: 用于丰富输出的交互式画布渲染


---


## 核心入口点 (Key entrypoints)

### 命令行界面 (CLI)

- `src/entry.ts` — CLI 入口（生成/设置环境变量，然后加载 `src/cli/run-main.ts`）
- `src/cli/` — CLI 命令定义
- `src/commands/` — 命令具体实现（约 329 个文件）

### 网关 (Gateway)

- `src/gateway/server.impl.ts` — 网关启动、配置验证/迁移、运行时连接
- `src/gateway/server-ws-runtime.ts` （及其同级文件）— WebSocket 服务器 + RPC 处理程序管道
- `docs/gateway/index.md` — 与生产环境操作方式匹配的运行手册

### 渠道 (Channels)

- `src/channels/` — 共享渠道逻辑（身份识别、白名单、门控、注册表）
- 包含完整适配器的独立渠道文件夹：`src/telegram/`, `src/discord/`, `src/slack/`, `src/signal/`, `src/imessage/`, `src/web/`, `src/line/`, `src/whatsapp/`
- 仅通过配置实现的渠道（无 `src/` 目录）：`googlechat`, `msteams`, `feishu`
- `docs/channels/` — 渠道文档（29 个文件，包含配对、路由、群组说明）

### Agent 轮次 (Agent turns)

- `src/auto-reply/` — 回复流水线
- `src/auto-reply/reply/agent-runner.ts` — 核心的 Agent 轮次编排器
- `src/agents/` — Agent 框架（约 751 个文件，12 个子目录）：工具、沙盒、授权配置文件、技能、多 Agent 协作

### 路由 (Routing)

- `src/routing/` — 专用的路由模块：`session-key.ts`, `resolve-route.ts`, `bindings.ts`

### 安全 (Security)

- `src/security/` — 安全相关逻辑（审计、策略、外部内容包装）
- `docs/gateway/security/index.md` — 面向运维人员的威胁模型 + 检查清单


---


## OpenClaw 仓库根目录的顶层结构如下：
```text
openclaw/
├── src/                        # 核心源码 (TypeScript)
├── apps/                       # 原生客户端应用
│   ├── macos/                  # macOS 菜单栏应用 (Swift)
│   ├── ios/                    # iOS 节点应用 (Swift)
│   ├── android/                # Android 节点应用 (Kotlin)
│   └── shared/                 # 跨平台共享代码 (Swift)
├── ui/                         # Web 控制台 UI (Lit + Vite)
├── extensions/                 # 可选通道扩展 (31 个)
├── packages/                   # 内部共享包
├── skills/                     # 内置技能 (52 个)
├── docs/                       # 官方文档源码
├── scripts/                    # 构建与工具脚本
├── test/                       # 全局测试配置
├── vendor/                     # 第三方代码
├── patches/                    # pnpm patch 补丁
├── git-hooks/                  # Git 钩子
├── package.json                # 主包配置
├── pnpm-workspace.yaml         # Monorepo 工作区定义
├── tsconfig.json               # TypeScript 配置
├── tsdown.config.ts            # 打包配置
├── vitest.config.ts            # 测试配置
├── docker-compose.yml          # Docker 编排
├── Dockerfile                  # 主容器镜像
├── Dockerfile.sandbox          # 沙箱容器镜像
├── Dockerfile.sandbox-browser  # 带浏览器的沙箱镜像
└── openclaw.mjs                # npm 全局安装的 CLI 入口
```


---


## `src/` 主要子目录结构
```text
src/
├── agents/                     # Agent 框架、工具、沙盒、授权配置文件、技能 (约 751 个文件)
├── gateway/                    # 网关服务器、WebSocket 运行时、RPC 处理、配置验证 (约 321 个文件)
├── auto-reply/                 # 回复流水线、Agent 轮次编排 (约 272 个文件)
├── cli/                        # CLI 命令定义与解析 (约 274 个文件)
├── commands/                   # CLI 命令具体实现 (约 329 个文件)
├── infra/                      # 基础设施：网络、SSRF 防护、执行安全、归档 (约 348 个文件)
├── config/                     # 配置模式、类型、验证、迁移 (约 217 个文件)
├── browser/                    # 浏览器自动化 (CDP/Puppeteer) (约 142 个文件)
├── channels/                   # 共享渠道逻辑、身份识别、白名单、注册表 (约 157 个文件)
├── memory/                     # 内存/上下文管理、QMD (约 89 个文件)
├── secrets/                    # 凭据引用解析、环境变量替换、审计 (约 44 个文件)
├── cron/                       # 定时任务调度 (约 93 个文件)
├── plugins/                    # 插件运行时与加载 (约 78 个文件)
├── media-understanding/        # 基于 AI 的图像/音频/视频理解 (约 63 个文件)
├── tui/                        # 终端用户界面 (Terminal UI) (约 45 个文件)
├── daemon/                     # 后台守护进程
├── hooks/                      # 生命周期钩子系统
├── media/                      # 媒体处理 (上传、下载、转换)
├── plugin-sdk/                 # 面向扩展开发者的插件 SDK
├── providers/                  # LLM 提供商集成 (Anthropic, OpenAI, Ollama 等)
├── routing/                    # 消息路由、会话密钥派生
├── sessions/                   # 会话管理与压缩
├── tts/                        # 文本转语音 (Text-to-speech)
├── wizard/                     # 安装设置向导
├── logging/                    # 日志记录、数据脱敏
├── link-understanding/         # URL 预览 / 链接理解
├── node-host/                  # 用于沙盒执行的 Node.js 宿主环境
├── pairing/                    # 设备配对流程
├── process/                    # 进程管理
├── terminal/                   # 终端集成
├── types/                      # 共享的 TypeScript 类型定义
├── utils/                      # 通用工具函数
├── shared/                     # 跨模块共享代码
├── acp/                        # ACP (Agent 通信协议)
├── canvas-host/                # 画布/绘图宿主环境
├── compat/                     # 兼容层
├── docs/                       # 应用内文档辅助工具
├── macos/                      # macOS 原生集成
├── markdown/                   # Markdown 解析处理
├── scripts/                    # 源码级别的脚本
├── test-helpers/               # 测试辅助工具
└── test-utils/                 # 测试工具函数
```