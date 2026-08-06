> **📦 agents-team 发布 / 分发仓（public release repo）**
> 本仓库**只放发布说明（landing）**；插件以**自包含 npm 装机包**（`.tgz`，包内自带完整插件本体）随各版 **Releases** 分发，源码在私有仓维护。
> ✅ 装机（Windows / Linux / macOS 通用）：到 [Releases](https://github.com/KDevSec/agents-team-release/releases/latest) 下载 `.tgz` → `npm i -g ./agents-team-1.2.2.tgz && agents-team`（不碰 npm registry / git 源仓，`npx agents-team` 不可用）。
> 各版本制品见 Releases。

# agents-team

数字员工集群——自包含单插件（编排引擎 + 记忆底座 + 业务员工 + 能力 skill）。**0.8.0 起一包多宿主**：同一个装机包除 Claude Code 外，还可把数字员工装到 [opencode](https://opencode.ai) 宿主运行（见下「多宿主」）。

从 [KDevSec/kdev-agents](https://github.com/KDevSec/kdev-agents) clean-room 抽取并通用化（去公司定制前缀、去第三方依赖、单插件化）。源仓保持冻结，本仓为通用产品going-forward 主线。

## 安装

前置：[Claude Code](https://docs.claude.com/claude-code) CLI（`claude`）+ `python3`（3.10+，编排引擎与状态栏需要）。方法论 skill（TDD/brainstorming/writing-plans 等 5 个，fork 自 superpowers，MIT）已随包内置，无需第三方 marketplace。code-graph 后端 understand-anything（MIT）是跨 marketplace 依赖的官方远程条目（marketplace `understand-anything`，上游仓 [Egonex-AI/Understand-Anything](https://github.com/Egonex-AI/Understand-Anything)），装机时**自动联网级联安装**；缺它只影响 code-graph（build/trace/impact/spec-link），不阻断核心员工。可选：Playwright MCP server（`qa` / `ui-autotest` 黑盒 UI 测试需要，缺失则这两类测试 env-gated 跳过）。

> 🌐 **弱网 / 无 GitHub SSH 环境注意**：UA 是 34M 的 git 仓，宿主内部克隆无续传，弱网下反复断连会导致装机失败（qoder/codebuddy 对依赖全有全无：UA 缺 = agents-team 被拒装）。请先手动浅克隆再注册目录型 marketplace，然后重跑装机（以 qoder 为例，codebuddy 换 `codebuddy` + `~/.codebuddy`）：
>
> ```bash
> git clone --depth 1 --single-branch --filter=blob:none https://github.com/Egonex-AI/Understand-Anything ~/.qoder/understand-anything-marketplace-src
> qodercli plugin marketplace add ~/.qoder/understand-anything-marketplace-src
> agents-team   # 重跑装机
> ```
>
> 该目录别删（目录型 marketplace 从它读）；以后更新 UA 用 `git -C <目录> pull`。

> 📦 本插件**不发布到 npm registry**、也**不依赖 git 源码仓**——只通过 GitHub Release 分发一个**自包含 npm 装机包**（`.tgz`，包内自带完整插件本体）。**插件本体自包含、本地安装，Windows / Linux / macOS 通用**；依赖的 understand-anything 不在包内，需另行联网从上游装（见上）。

### 1. 下载装机包

最新版直达：<https://github.com/KDevSec/agents-team-release/releases/latest>（网页点 `.tgz` 即可，所有平台）。或命令行：

```sh
# 跨平台（需 gh，自动取最新版）
gh release download --repo KDevSec/agents-team-release --pattern '*.tgz'
# Linux / macOS（指定版本直链）
curl -fL -O https://github.com/KDevSec/agents-team-release/releases/download/v1.2.2/agents-team-1.2.2.tgz
```

```powershell
# Windows PowerShell
iwr https://github.com/KDevSec/agents-team-release/releases/download/v1.2.2/agents-team-1.2.2.tgz -OutFile agents-team-1.2.2.tgz
```

### 2. 本地装（npm + node，三平台通用）

```sh
npm i -g ./agents-team-1.2.2.tgz
agents-team
```

装机做三件事，**幂等、可重跑**：注册 marketplace（用**包内本地路径**）→ 装插件 `agents-team@ieidev` → 接状态栏。装好后状态栏出现 `agents-team …`；重载插件（`/reload-plugins`）或重启 session 后生效。

> **为什么不碰 registry / 源码仓**：installer（`bin/cli.js` / `install.sh`）检测到包内 `.claude-plugin/marketplace.json`，直接用**包内本地路径** `marketplace add`、`plugin install` 把插件复制进 `~/.claude/plugins/cache/`。所以装机只需这个 `.tgz`，无需 npm 账号、无需访问源码仓。`npx agents-team` **不可用**（不发 registry）。
> **unix / WSL / Git Bash** 另可解包跑脚本：`tar xzf agents-team-1.2.2.tgz && cd package && bash install.sh`（与上面 npm 装法共用同一幂等决策核）。

常用开关（`agents-team --help` 看全部）：

```sh
agents-team --project                       # 写项目级状态栏（当前目录 .claude/settings.json，默认用户级）
agents-team --marketplace-source <本地路径>   # 覆盖 marketplace 源（离线/自定义；默认=包内自带插件本地路径）
agents-team --host all|claude-code|opencode|codebuddy  # 选装宿主（默认 all：探测本机在场宿主逐个装）
agents-team --owner auto|ieidev              # opencode 主配置家归属决策（默认 auto 探测，绝不覆盖既有配置）
```

### 3. 或用 `/plugin` 安装（需搭配 `/agents-team:setup` 接状态栏）

在 Claude Code 会话里直接装插件（无需 npm）：

```text
/plugin marketplace add <本仓或 release 内 .claude-plugin 路径>
/plugin install agents-team@ieidev
```

**与 npm 装机的差别**（Claude Code 插件机制的固有限制）：
- **HUD 后台服务**：装后**下一个新会话**由 SessionStart hook 自动起（幂等、非阻塞），无需手动。
- **主状态栏**：插件**无法**自动注册主 statusLine（只能写用户/项目级 settings.json）。下一个会话会**一次性提醒**你运行下面命令接入：

  ```text
  /agents-team:setup            # 写用户级状态栏（--project 写项目级）
  ```

**卸载**：先清状态栏再卸插件，避免 settings.json 残留断链命令：

```text
/agents-team:setup --uninstall
/plugin uninstall agents-team@ieidev
```

### 4. 多宿主：装到 opencode（0.8.0+）

同一个 `.tgz` 也能把数字员工装到 opencode。默认 `--host all` 探测本机在场宿主逐个装（opencode 需已安装且**至少启动过一次**——配置目录尚未生成时视为未在场跳过）；`--host opencode` 只装 opencode 侧。

- **隔离共存**：本插件资产落**专属配置家**（默认 `~/.config/opencode-ieidev/opencode`，worker 经 XDG 定向消费），不触碰你现有的 opencode 全局配置；与 oh-my-openagent（oma）同机时归属自动探测让位（`--owner` 可显式指定），**绝不覆盖既有 oma / 个人配置**。
- **行为对齐**：agent 发现、拦截插件加载、人工审核闸在 opencode worker 上端到端生效；员工模型由 5 档语义 tier 统一映射到各宿主模型，宿主可独立换模型不改员工定义。
- Windows 上 opencode 宿主的配置隔离为 best-effort，未经真机验证（CC 宿主不受影响）。

### 5. 多宿主：装到 CodeBuddy

同一个 `.tgz` 也能把数字员工装到 [CodeBuddy Code](https://www.codebuddy.ai)（`@tencent-ai/codebuddy-code`，Claude Code 家族衍生，插件清单/plugin CLI/钩子协议与 CC 近乎同构）。前置：本机已装 `codebuddy` CLI 且已登录；`--host codebuddy` 只装 codebuddy 侧，`--host all` 探测到 codebuddy 在场时一并装。

- **落地方式**：复用 codebuddy 自带 plugin CLI（`codebuddy plugin marketplace add` + `codebuddy plugin install agents-team@ieidev`），对称于 Claude Code 装机段——不像 opencode 需要自研文件落地/协议翻译层。
- **模型档位**：5 档语义 tier 映射到 codebuddy 模型面（强档 `glm-5.2`、执行档 `deepseek-v4-flash`），宿主可独立换模型不改员工定义。
- **编排驱动（L2）**：`flow-driver`/`goal` 可用 `codebuddy -p` spawn worker；判停/续跑与 Claude Code 一致（主判据读 `.ieidev/` 引擎状态，宿主无关）。驱动为单测覆盖 + 与 CC/opencode 同构；**真机 L2「待审核→续跑」全链路待坐实**。
- 实测（CodeBuddy Code 2.117.0）：装机后交互会话内 36 个 agent / 6 个 command / 25 个 skill / 18 个钩子被发现，且钩子运行时实际触发（SessionStart/UserPromptSubmit 等）。
- 已知：2 个 SKILL.md（`ieidev-test-cases`/`ieidev-test-points`）的 description 字段触发 codebuddy 更严格的 YAML 解析器告警（不影响功能），待后续修 frontmatter。
- Qoder 宿主适配尚未提供。

## 用法（按步骤）

装好后你有两件事可做：**指挥数字员工干活**，和**起 HUD 看它们干到哪了**。

> **数字员工是什么**：不是常驻进程，而是按需实例化的 **agent 角色**——由主控（你的 Claude Code 会话）按各自的 SOP flow 编排调度，干完一步即归。进度账实时落在项目 `.ieidev/features/<slug>/`（flow-state + events），状态栏与 HUD 都从这里派生。
>
> **工程记忆零配置（v0.4.0+）**：**首次调用数字员工即自动建立** `.ieidev/memory/` 工程记忆——静默、零提示、按工程，无需显式初始化；记忆仓为独立 git 仓（无远程时本地自动建仓 + 持续提醒去建远程）。详见 `memory` 能力 skill（见下「集群概览」）。

### 一、和数字员工交互

**A. 一个目标端到端（推荐）—— `goal` 总编排**

1. 在你的项目目录起 Claude Code 会话，一句话说目标：

   ```text
   /agents-team:goal 给登录页加「记住我」并出可上线版本
   ```

2. `goal` 把目标路由到交付生命周期，渲**一屏编排结论**（要跑哪几段、各段哪个员工、人工闸停在哪），让你确认或微调。
3. 确认后它**顺序链式**跑，同 slug 接力：
   需求架构师（澄清 → SR 规格 → 拆解 AR/用户故事 → 原型 → 方案）
   → 开发工程师（环境对齐 → 实施计划 → 编码/前端 → 安全自评 → E2E 视觉验收 → 部署）
   → 测试工程师（测试点 → 用例 → UI/API 自动化）。
4. 段与段之间在**人工闸**停下等你确认；关键节点自动发函 **CQO 监督员**做质量复核（建议非拦截，不替你拍板）。
5. 全程在状态栏 / HUD 看进展（见下「二」）。

**B. 只让某一个员工跑它的 SOP —— `flow-driver`**

```text
/agents-team:flow-driver dev-engineer --task "按 PLAN 实现登录失败锁定策略"
```

不走总编排，直接驱动指定数字员工跑自己的 flow。员工 id 见下方「集群概览」（`req-architect` / `dev-engineer` / `test-engineer` …）。

### 二、启动 HUD 服务（观测层，只读，不改任何状态）

HUD 把数字员工的进展可视化。三个通道，按需选。命令里的 `${CLAUDE_PLUGIN_ROOT}` 是装机后的插件根目录（在插件目录里也可直接用 `PYTHONPATH=pyieidev`）。

**① 状态栏（装机即有，零操作）**
Claude Code 底部 `agents-team …` 单行，显示在跑的需求 / 当前节点 / 活动；无在跑任务时显示 `agents-team │ 暂无在跑需求`。

**② 实时全局台（0.2.0 新，推荐）—— 一个浏览器台子看本机所有项目**

1. 起服务（读本机 registry，聚合你用过数字员工的**所有项目**，无需在某个项目目录里）：

   ```sh
   PYTHONPATH=${CLAUDE_PLUGIN_ROOT}/pyieidev python3 -m ieidev_hud serve --global --open
   ```

   `--open` 自动开浏览器；默认 `http://127.0.0.1:8765`，`--port N` 改端口。

2. **左侧任务树**：按项目分组列各 goal——活跃 `◐` / 完成 `✓`（灰显折叠）/ `⚠` stale（workspace 已失联）；点任一 goal。
3. **右栏**（选中 goal）：链进度 % + 阶段路线（需求→开发→测试，当前段高亮）+ worktree + **Story TODO** 清单 + **监督员告警** + **评审流水**（时间 · 评审项 · 状态）。点 story 行或进度环可**钻入详情抽屉**（验收标准 / 阶段明细 / 最近事件流）。
4. **顶部「在跑总览」**：此刻全机有哪些活跃派单（哪个项目/goal 正在跑哪一步、跑了多久；久无回执的孤儿派单标 stale）。
5. 页面走 **SSE 实时推送**——有变化秒级局部刷新，**不整页闪、不丢你的选中/展开态**（告别旧版 2 秒全页 reload）。

**③ 单项目实时台**
在某个项目目录里去掉 `--global`，只看当前项目：

```sh
PYTHONPATH=${CLAUDE_PLUGIN_ROOT}/pyieidev python3 -m ieidev_hud serve --open
```

**④ 静态快照（离线 / 留档 / 截图）**

```sh
PYTHONPATH=${CLAUDE_PLUGIN_ROOT}/pyieidev python3 -m ieidev_hud render --global   # 全局，出 hud.html
PYTHONPATH=${CLAUDE_PLUGIN_ROOT}/pyieidev python3 -m ieidev_hud render            # 仅当前项目
```

`render` 把当前 model 内联进一个**自包含 hud.html**（零外链），浏览器直接打开，可离线浏览 / 截图存档。

> 全局台依赖本机 registry——首次用数字员工立项时自动登记，跑过的项目才会出现在全局台上。`--help` 看全部子命令与开关。

## 集群概览

**数字员工**（`staff.yml` 注册，各自有编排 SOP）：

| 员工 | 职责 |
|------|------|
| 需求架构师 `req-architect` | 需求澄清 → SR 规格 → 拆解（AR + 用户故事）→ 原型 → 方案设计 |
| 开发工程师 `dev-engineer` | 环境对齐 → 实施计划 → 编码/前端实现 → 安全自评 → E2E 视觉验收 → 部署 |
| 测试工程师 `test-engineer` | 黑盒测试点/用例设计 → UI/API 自动化执行 |
| 调研员 `researcher` | 探索调研：立题（问题域/成功判据/方法集）→ 外部/内部取证 → 综合成文（引用 + 置信度分级）|
| 评审专家 `reviewer` | 方案/SR/故事/原型/代码/安全/测试设计/测试覆盖/调研 多维度百分制评审 gate |

> **评审密度可调**：req-architect 的 8 个复合评审闸默认走 `review_profile: standard`（合并 4 个 checkpoint，全程专家、覆盖不丢），`full` 恢复 1.0 细粒度 8 闸、`light` 合并 2 闸。确认屏 `[rp light|standard|full]` 切档；结构与策略分层，改 SOP = 改源 node-table 或 `review-merge.yml` + 跑 `compile-review-tables` 重生。详见 `gate-decision-logic.md`「review_profile 档位」段。
| **CEO 总编排** = `goal` skill | 高层目标 → 交付链编排 + 人工闸 + 发函 CQO |
| **CQO 监督员** = `cqo-orchestrator` | 跨 flow 质量监督（L-a 逐事件全检 + L-b circuit-breaker 聚合 + L-c 棒间建议），建议非拦截 |

**能力 skill**（员工与主控按需调用）：

- 需求/设计撰写：`sr-authoring` · `ar-authoring` · `detailed-design-authoring` · `constitution-authoring`
- 前端/原型：`frontend-design`
- 测试：`test-points` · `test-cases` · `ui-autotest` · `env-recon` · `python-testing-patterns` · `qa`
- 代码图：`codegraph-build` · `codegraph-impact` · `codegraph-trace` · `codegraph-spec-link`
- 工程底座：`memory`（持久工程记忆 + 召回 + 蒸馏）· `env-recon`（被测环境踩点）· `secure-coding`（Python 安全编码规范）· `flow-driver`（通用 flow 引擎驱动）

> 编排引擎、记忆底座与 HUD 由 `pyieidev/` 下 4 个自包含 python 包（`ieidev_core` / `ieidev_team` / `ieidev_hud` / `ieidev_ingestor`）支撑，均以 `python -m` 行内自带 `PYTHONPATH` 方式调用，无需 pip 安装。
