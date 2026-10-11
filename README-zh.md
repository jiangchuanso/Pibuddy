# Orbit

[![CI](https://github.com/imrj05/orbit/actions/workflows/ci.yml/badge.svg)](https://github.com/imrj05/orbit/actions/workflows/ci.yml)
[![License: Apache-2.0](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](LICENSE)
[![Rust](https://img.shields.io/badge/rust-1.94%2B-orange.svg)](https://rustup.rs/)

**Orbit** 是 [pi 编程智能体](https://github.com/earendil-works/pi) 的原生桌面工作台 —— 一个聊天式 GUI，完全用 Rust 基于 GPUI（Zed 所采用的 GPU 加速 UI 框架）渲染。它直接通过 stdio 与 pi CLI 的 RPC 协议通信：**没有浏览器、没有 webview、没有 Node**。

你在 Orbit 中创建的会话与在终端中创建的会话是**同一批会话**（`~/.pi/agent/sessions/`），由 pi 自带的会话管理器统一管理。

## 界面截图

一个实时会话 —— 正在流式输出的转录、工具行与输入框：

![一个已完成的 Orbit 会话：智能体关于 Windows IME 输入法修复的笔记，以及一张“已更改 4 个文件”卡片（含每个文件的行数），上方是输入框。](marketing/public/screens/session.png)

用量页面 —— 请求数、token、成本与缓存，全部读取自你自己的会话：

![Orbit 的用量页面，上方展示请求、token、成本和缓存指标，下方是 token 随时间变化图表。](marketing/public/screens/usage.png)

| 新建任务 | 审查 |
| --- | --- |
| ![Orbit 的新建任务页面：工作区选择器、输入框，以及展开“受监督 / 自动接受编辑 / 完全访问”的访问模式菜单。](marketing/public/screens/new-task.png) | ![Orbit 的审查面板，在会话旁打开，左侧展示所选文件的 diff，右侧是已更改文件树。](marketing/public/screens/review.png) |
| **Git** | **Providers** |
| ![Orbit 的 Git 页面，列出已暂存与已更改文件及行数，并带有提交框。](marketing/public/screens/git.png) | ![Orbit 的 Providers 页面，展示 Ollama、OpenCode Go、Bedrock、Anthropic 的提供商卡片及用量表。](marketing/public/screens/providers.png) |
| **插件** | **技能** |
| ![Orbit 的插件页面，列出已安装的 pi 包，含更新与移除操作。](marketing/public/screens/plugins.png) | ![Orbit 的技能页面，列出项目级与全局技能，右侧渲染所选技能的 SKILL.md。](marketing/public/screens/skills.png) |
| **模型** | **外观** |
| ![Orbit 的模型页面，按提供商分组列出模型，含上下文大小与收藏开关。](marketing/public/screens/models.png) | ![Orbit 的外观设置，含主题选择器、字体与疏密预览、字体选择器。](marketing/public/screens/appearance.png) |
| **通用** | **智能体** |
| ![Orbit 的通用设置，展示已连接的 pi 智能体、本地会话存储、工作区和通知开关。](marketing/public/screens/general.png) | ![Orbit 的智能体设置，含后续回复投递、自动压缩、自动重试、立即压缩和会话重命名。](marketing/public/screens/agent.png) |

## 功能特性

**聊天与转录**

- 聊天式智能体会话 —— 输入框支持选择模型、思考强度、后续追问排队、运行中转向、取消与流式回复
- **自定义回车行为**：在 设置 → 智能体 → 行为 中设置：默认将回车作为“排队追问”，或用于“转向运行中的任务”。Alt/Option+Enter 使用另一种模式；⌘⇧Enter / Ctrl+Shift+Enter 始终用于转向。当智能体工作时，上下文窗口指示器左侧的紧凑提示会同时显示两种发送模式。转向在当前回复与工具执行完成后生效；空闲时回车正常发送，Shift+Enter 换行
- **工作流模式** —— 将会话限定为 **计划（Plan）**、**构建（Build）** 或 **提问（Ask）**：计划与提问为只读模式（写工具被禁用，bash 受限），便于你在切换到构建前先探索与规划
- GPU 渲染的转录 —— 采用虚拟化，因此开销与消息数量无关，支持贴附最新、合并提交
- 支持 GFM markdown 与语法高亮的代码；高亮为纯绘制，流式代码块永远不会重排，Mermaid 围栏仍保持为可复制的代码块
- 转录内查找（⌘F）与全窗口图片灯箱
- 工具活动 —— bash、编辑、读取与思考行均由原生绘制，可展开为“参数 / 输出”卡片，每个区块可单独复制

**会话**

- 按项目分组、持久化、可重新打开、跨工作区
- Orbit 自有的项目列表 —— 可从侧边栏移除某个工作区，而不删除 pi 的会话
- 按最近活动、最近添加、名称或会话数量排序项目
- 重命名、置顶、克隆或删除会话，并复制会话 id；置顶的会话排在前面
- 按位置定位会话 —— **⌘1…⌘9**（`Ctrl+1…9`）打开第 N 个可见会话，**Ctrl+Tab / Ctrl+Shift+Tab** 循环切换
- CLI 或另一个窗口写入的会话会实时刷新侧边栏
- 后台轮次通知 —— 当窗口不在最前时弹出桌面横幅与提示音，否则显示应用内 toast；当运行等待某个扩展对话框时也会给出提示

**安全护栏**

- 访问模式 —— 受监督、自动接受编辑、完全访问，从输入框中选择
- 由内置的 pi 扩展强制执行：该扩展钩住 `tool_call`，并对当前模式未自动批准的变更调用进行确认
- 输入框上方的行内栏提供 **允许一次 / 总是允许此工具 / 拒绝**（↑ ↓ ⏎ esc），每种模式各自记录白名单
- 这是确认式护栏，而非沙箱 —— pi 本身不带沙箱，Orbit 也不会额外添加

**审查与 Git**

- 审查侧边栏，可选来源 —— **上一轮（Last Turn）**（某次智能体轮次确切更改的内容，来自逐轮检查点）、未提交、未暂存、已暂存、已提交、分支 —— 在运行结束时刷新
- diff 视图控制：折叠/展开每个文件、长行换行或平移、统一/并排行、停靠/整页/最小化，以及完整键盘操作（Tab 焦点切换、方向键、`n` / `p` 切换文件、`[` / `]` 切换区块）
- Git 页面涵盖 更改 / 历史 / 图谱 / 暂存：暂存、提交（⌘↵）、stash 的 push/pop/apply/drop、同步、推送（含 force-with-lease）、拉取、获取、合并与变基，并为被拒推送与冲突提供恢复操作
- Git 页面的 议题（Issues）与 拉取请求（Pull requests） 标签页通过 `gh` 实现：浏览与筛选、提交 issue 与 PR、评论、应用标签、关闭/重开、审查、合并、检出 PR 分支 —— 并支持仓库模板，以及 AI **生成（Generate）** 动作：根据分支、其提交、可选备注和仓库自身模板起草标题与正文
- Git worktree 作为一等工作区 —— Worktrees 页面列出并创建 worktree（`repo/.wt/113` → `feature/issue-113`），将其作为活动工作区打开，使资源管理器、终端、Git 页面、审查与智能体都在此处运行，并支持重命名 / 移动 / 锁定 / 移除 / 修剪 / 修复；状态栏的 **Work in** 芯片可在 Local 与任意 worktree 之间切换，或打开带自定义 worktree 名称的快速创建；可选的每仓库设置脚本会以 `ORBIT_ROOT_PATH` 与 `ORBIT_WORKTREE_PATH` 异步运行

**资源管理器与文件**

- 项目面板（⌘⇧E）—— 感知 gitignore 的工作区树，支持展开/折叠、过滤、隐藏文件开关、devicons、`M/A/D/R/U` git 徽标与键盘导航
- 从树进行文件操作：新建文件、新建文件夹、重命名、删除到回收站，以及打开 / 在文件管理器中显示 / 复制路径
- 文件页面带标签条与活动工作区的编辑器：语法高亮的 editable 缓冲区、Markdown 编辑/预览、图片、防抖自动保存（及 ⌘S，含脏/保存中状态），以及对二进制、超大、被截断和非 UTF-8 文件的诚实只读保护

**终端**

- 集成终端（⌘J）—— 可缩放底部面板中的真实登录 shell，基于 `alacritta_terminal` 实现 PTY 与 VT/ANSI 仿真，由 GPUI 原生渲染：滚动回溯、点击拖选（⌘C/⌘V）、 bracketed paste、闪烁光标、每主题 ANSI 调色板与重启；shell 跟随活动工作区

**工作台**

- 用量、技能、插件、模型、提供商、MCP、外观、智能体/运行时设置，全部以 pi 的磁盘数据为基础
- **MCP** —— 在 设置 → MCP 中管理 pi 的 MCP 服务器（全局与每项目）：添加 / 编辑 / 校验、`${NAME}` 密钥存储、来自 `pi mcp list --json` 的实时状态与工具列表，并通过重启 pi（保留会话）来应用变更
- 提供商 —— pi 的实时目录加自定义端点（`models.json`），支持 API key 或 OAuth 登录与用量表
- 插件 —— 从 npm、git 或本地路径安装 pi 包，全局或每项目；就地更新检查与更新/移除
- 模型 —— 读取自 pi 自身的运行时，含 **默认会话模型** 与思考级别（设置 → 智能体）并应用于每个新会话；该选择存在于 Orbit 自身存储中，不动 pi 的全局设置
- **会话标题自动命名** —— 内置 pi 扩展在首轮后命名会话，含模型选择器与 设置 → 智能体 中的开关
- 外观 —— **浅色 / 深色 / 跟随系统**，独立的浅色与深色主题调色板、背景图、字体、字号、间距疏密与 **界面语言**（英语、简体中文、日本語、한국어、Español、Français、Deutsch、Português do Brasil、Русский、Italiano，或系统语言）。跟随系统会实时响应操作系统外观变化；全新安装使用“跟随系统”，已有主题选择则保留

**原生体验**

- 命令面板（⌘P / ⌘K）与基于注册表的快捷键层：键位映射、面板芯片与 设置 → 快捷键 参考均读取自同一张命令表，支持 Tab 焦点遍历、上下文感知按键（审查树、选择器、对话框）、自定义 macOS 窗口装饰、原生对话框与减弱动态效果
- Zed 的设计令牌（间距与疏密、字体、图标与按钮尺寸、层级、动效）位于 `theme/tokens.rs`，随 UI 字号与间距疏密设置缩放 —— 如今设置、用量、侧边栏、转录外壳、面板、选择器、模态框、资源管理器、Git 面板与终端均已使用，输入框是最后一个主要界面
- **打开于（Open in）** —— 直接从标题栏在检测到的编辑器或终端中打开工作区，并带各自真实图标
- 首次运行对 `pi`、`node`、`git` 的依赖检查，缺失项会报告其安装命令
- 签名发布：macOS `.dmg`（签名 + 公证，通用架构）与 `.tar.gz`、Windows `.exe` / `.zip`、Linux `.deb` / `.tar.gz`，各平台均支持签名的应用内更新

## 架构

```
┌───────────────────────────────────────────┐
│  Orbit (Rust) — GPUI，Metal 渲染 UI         │
│  crates/orbit-pi                          │
│  window, transcript, markdown, workbench  │
└──────────────┬────────────────────────────┘
               │ 基于换行分隔的 JSON RPC，走 stdio
┌──────────────▼────────────────────────────┐
│  pi CLI（子进程，每个打开的会话一个）        │
│  pi --mode rpc — 会话、模型、工具           │
└──────────────┬────────────────────────────┘
               │
┌──────────────▼────────────────────────────┐
│  ~/.pi/agent/ — 会话、配置、用量           │
│  （与 pi CLI 使用的数据相同）              │
└───────────────────────────────────────────┘
```

Orbit 为每个打开的会话将 `pi` CLI 作为子进程启动，并与其 RPC 协议通信：stdin 上的 JSON 请求、stdout 上的 JSON 行事件（文本/思考增量、工具调用、提问对话框、结算信号）。所有面向用户的内容都由 GPUI 在 Metal 上绘制 —— 没有 DOM、没有 CSS、没有浏览器引擎。智能体运行时仍是 `pi` 本身，因此模型目录、思考级别与持久化仍归属 pi。

### 项目结构

```
crates/orbit-pi/       GPUI 应用 —— 窗口、转录、markdown、工作台
crates/orbit-analytics/  隐私优先、与提供商无关的产品遥测
crates/orbit-rpc/      pi CLI 的 RPC 客户端（进程生命周期 + JSONL 协议）
contrib/              内置的 pi 扩展（访问护栏、认证/配额桥接）
scripts/make-dmg.sh   构建签名 .app + DMG（arm64 / 通用）
assets/icons/         应用图标（logo-icon.png 源、icon.icns、icon.png）
marketing/            Next.js 落地页
PRODUCT.md            产品定义、能力、约束
INTENT.md             架构决策 + 阶段计划
AGENT.md              在本仓库工作的智能体与人类约定
PRIVACY.md            匿名用量分析发送与未发送的内容
```

## 路线图

Orbit 是一个工作台，不只是一个聊天窗口。已确认的方向见 [PRODUCT.md](PRODUCT.md)，架构决策见 [INTENT.md](INTENT.md)，已交付的工作见 [CHANGELOG.md](CHANGELOG.md)。下方每个未勾选项都欢迎提交 PR —— 开始之前请先阅读 [CONTRIBUTING.md](CONTRIBUTING.md) 并认领对应的 issue。

### 计划中的功能

| 功能 | 领域 | 描述 | 状态 | 跟踪 |
| --- | --- | --- | --- | --- |
| 工作流模式 | 输入框 | 将会话限定为 **计划模式**、**构建模式** 或 **提问模式**，而非单一无差别聊天。 | 已交付 | — |
| 结算时跟进 | 转录 | 运行结束后在行内显示排队的追问，而不仅在输入队列中。 | 计划中 | — |
| 建议的后续追问 | 转录 | 运行结束后，给出 2–3 条贴合上下文的下一条提示，作为输入框插入项。 | 提案 | [#10](https://github.com/imrj05/orbit/issues/10) |
| GitHub 客户端 | 工作台 | 在应用内浏览并管理远程提交、图谱、issue 与拉取请求。 | 进行中 —— Issues 与 Pull requests 已支持评论、审查、合并与分支检出；远程提交浏览仍待完成 | [#8](https://github.com/imrj05/orbit/issues/8) |
| Pi 扩展支持 | 工作台 | 对 pi 扩展的一等支持：列出 / 安装 / 启用 / 配置 / 调试，含社区扩展。 | 进行中 —— 插件已支持安装、更新与移除包；带启用/配置/调试的专属扩展界面仍待完成 | [#7](https://github.com/imrj05/orbit/issues/7) |
| 全局默认模型 | 模型 | 在 Orbit 内固定 pi 的全局默认模型与思考强度。 | 已交付 —— 设置 → 智能体 为每个新会话设定起始模型与思考级别（Orbit 自身存储；不动 pi 全局设置） | [#9](https://github.com/imrj05/orbit/issues/9) |
| 语音听写 | 输入框 | 向输入框口述提示。 | 计划中 | — |
| diff 审查与终端 | 工作台 | 行内 diff 审查与集成终端。 | 已交付 —— 带逐轮检查点与视图控制的审查面板，加上登录 shell 终端（⌘J） | — |
| 并行会话 | 会话 | 并排运行多个智能体 / 会话。 | 计划中 | — |
| 资源管理器缺口 | 资源管理器 | 文件树中的快速打开、粘性滚动与目录折叠。 | 计划中 —— 展开/折叠、过滤、隐藏开关与 git 徽标已交付 | — |
| 对话分支/回退 | 会话 | 分支并回退一段对话（克隆现已交付）。 | 计划中 | — |
| 滚动性能测量 | 性能 | 在设备上测量转录滚动性能。 | 计划中 | — |
| Zed 令牌迁移 | 设计 | 将剩余界面（设置、选择器、命令面板、模态框、转录外壳、侧边栏行）迁移到 Zed 设计令牌。 | 进行中 —— 设置、用量、侧边栏、转录外壳、面板、选择器、模态框、资源管理器、Git 面板与终端已基于令牌；输入框是最后一个主要界面 | — |

### 贡献者清单

- [x] **工作流模式** —— 计划模式 / 构建模式 / 提问模式
- [ ] **结算时跟进** —— 运行结束时行内显示排队的追问
- [ ] **建议的后续追问** —— 贴合上下文的下一条提示（[#10](https://github.com/imrj05/orbit/issues/10)）
- [ ] **GitHub 客户端** —— 远程提交与图谱（issue 与 PR 现已交付）（[#8](https://github.com/imrj05/orbit/issues/8)）
- [ ] **Pi 扩展支持** —— 专属扩展界面；插件目前管理包（[#7](https://github.com/imrj05/orbit/issues/7)）
- [x] **全局默认模型** —— 为新会话设定默认模型与思考级别（[#9](https://github.com/imrj05/orbit/issues/9)）
- [ ] **语音听写** —— 向输入框口述提示
- [x] **diff 审查与终端** —— 行内审查加集成终端
- [ ] **并行会话** —— 同时多个智能体 / 会话
- [ ] **资源管理器缺口** —— 快速打开、粘性滚动、目录折叠
- [ ] **对话分支/回退** —— 分支并回退会话
- [ ] **滚动性能测量** —— 设备端转录滚动基准

---

环境搭建、检查与提交约定见 [CONTRIBUTING.md](CONTRIBUTING.md)。Orbit 基于 [Apache-2.0](LICENSE) 许可；漏洞请按 [SECURITY.md](SECURITY.md) 报告。匿名用量分析详见 [PRIVACY.md](PRIVACY.md)。
