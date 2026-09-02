# XingTu MCPs · 专业自述

> 定位：行途开源矩阵的 MCP 服务资产仓，收录可独立部署、可被任意 MCP 客户端复用的 MCP server。

## 1. 这个项目是干什么的（作用）

- **给谁用**：需要在 Agent 会话里拿到真实工具能力（读写外部系统、调业务接口、执行有副作用的操作）的开发者；想快速起一个自用 MCP server 而不想从协议层踩坑的人。
- **解决什么问题**：Agent 只靠文本提示只能"说"，MCP 让它"做"。本仓把散落在真实工程里的可复用工具能力收敛成标准 MCP server，任何支持 MCP 的宿主（Claude Code、Claude Desktop、Cursor、CodeBuddy 等）都能通过同一种方式发现和调用。
- **核心价值**：
  1. **协议先行**：所有 server 走 MCP 规范（JSON-RPC 2.0 + 工具 schema），不绑死某个 Agent 宿主，换客户端不用重写。
  2. **标准化起步**：每个 server 从 `servers/_TEMPLATE` 起步，统一目录与 README 结构（安装 / 配置 / 工具列表），把"起一个 server"的成本压到最低。
  3. **可独立部署**：每个 server 独立运行，stdio 或 HTTP(S) 传输都可挂载，本地、远端皆可。
  4. **可审计**：工具带参数 schema 声明，调用有明确的入参/出参边界，Agent 调用行为可记录、可回放。
  5. **与 harness 打通**：作为子模块被 xingtu-harness 聚合，场景安装时按需拉取，而不是全量堆进 Agent 环境。

## 2. 在行途 harness 中的定位

- **位置**：执行层（L3）的工具供给端。harness 六层架构里，mcps 与 skills、tools、cli 共同构成"Agent 能干活的能力面"，但 mcps 走的是**协议化工具通道**——能力以外部服务形式存在，Agent 通过工具调用触发，而不是把逻辑读进上下文。
- **协作关系**：

```
             ┌──────────────────────────────────────────┐
             │  Agent 宿主（Claude Code / Cursor / ...）  │
             └───────────────────┬──────────────────────┘
                                 │ MCP 协议（JSON-RPC 2.0）
                 ┌───────────────┴───────────────┐
                 │       xingtu-mcps (servers/)   │  ← 本仓
                 │  server-A   server-B   ...     │
                 └───────────────────────────────┘
                                 ▲
                 ┌───────────────┴───────────────┐
                 │  xingtu-harness（submodule 聚合）│
                 │  xingtu-tools/cli（本机脚本，无协议）│
                 └───────────────────────────────┘
```

- **与相邻仓的关系**：tools / cli 是**本机进程内**脚本，直接执行、零协议；mcps 是**跨进程、跨宿主**的协议化工具，Agent 侧看到的是带 schema 的"工具列表"而不是一段 shell。

## 3. 与其他项目的差异与区别

| 对比项 | xingtu-mcps | xingtu-tools / xingtu-cli | xingtu-skills |
|--------|-------------|---------------------------|---------------|
| 能力形态 | 独立运行的 server 进程 | 本机脚本 / 命令行 | 文档式技能（SKILL.md + 辅助脚本） |
| 调用方式 | Agent 经 MCP 协议调工具（带 schema） | 人或 Agent 直接执行命令 | Agent 按需读取技能说明后自行执行 |
| 边界 | 工具即服务，有生命周期、可远端部署 | 无协议，进程内、跑在宿主机器上 | 教 Agent "怎么想怎么做"，不产副作用工具 |
| 触发机制 | 宿主侧工具调用 | shell 调用 | LLM 检索 description 后加载 |
| 何时选它 | 需要真实读写外部系统、且要跨宿主复用 | 简单本机操作，不想要协议层 | 需要改变 Agent 行为方式/流程约束 |

一句话：**skills 给方法，tools/cli 给本机命令，mcps 给跨宿主的服务化工具**。三者不重叠，可同时挂载。

## 4. 在 Agent 体系中的应用

### 4.1 Work Agent（业务/内容工作流）

- **作用方法**：内容/业务工作流里需要真实数据操作时（查企业库、写表格、发消息、调接口），挂载对应 server，Agent 以工具调用方式完成，而不是让 Agent 靠提示词"猜"或自己拼脚本。
- **触发方法**：宿主配置 `mcp add` 后即成为 Agent 的工具集；Agent 在需要时自动选择调用，无需人工干预。
- **典型场景**：调研类任务里查结构化数据源、运营任务里批量读写业务表、内容任务里把素材落库。

### 4.2 Coding Agent（编码 Agent，如 Claude Code）

- **作用方法**：`claude mcp add <name> <command>` 把 server 接入会话，工具出现在 Agent 的可用工具列表里。
- **触发方法**：**工具调用**，不是提示词——Agent 判断需要外部能力时直接发起调用，参数由 schema 校验。
- **典型场景**：需要读写远端系统/数据源的编码任务、需要在代码改动外联动外部服务的任务、需要把 Agent 行为接入自有后端能力的场景。

## 5. 升级方法与迭代开发

- **新增 server**：`cp -r servers/_TEMPLATE servers/<name>`，实现 tools / resources / prompts，写独立 README（安装 / 配置 / 工具列表）。
- **质量门禁**：
  - 每个工具必须可测：能跑通一次真实调用、参数 schema 校验通过。
  - 有副作用的工具必须有幂等或保护设计（如备份、确认参数），避免 Agent 误操作造成不可逆影响。
  - 不出现在 README 里的工具不作为发布资产。
- **演进路径**：先补齐首批高频 server（网络诊断、操作前备份、安全删除、契约校验等日常场景），再逐步扩展业务向工具；每新增一个即回填 `servers/<name>/README.md` 与顶层清单。
- **当前状态**：目录规范与 `_TEMPLATE` 已就绪，首批 server 发布是当前主任务。

## 6. 基础概念

- **MCP（Model Context Protocol）**：模型上下文协议，定义模型如何发现和调用外部工具、读取资源的标准接口，让能力与模型解耦。
- **host / client / server**：host 是用户界面（如 Claude Code），client 是 host 内的协议连接端，server 提供工具/资源/提示词的独立进程。
- **tool / resource / prompt 三类原语**：tool 是可执行操作（有副作用），resource 是只读数据（如文件、文档），prompt 是模板化提示词。三类能力统一走协议。
- **JSON-RPC 2.0**：MCP 底层消息协议，请求/响应/通知的标准化格式。
- **transport（stdio / Streamable HTTP）**：stdio 适合本机子进程挂载，HTTP 适合远程服务；选哪种取决于 server 部署位置。
- **tool schema**：工具入参的 JSON Schema 声明，是 Agent 正确调用的前提——schema 越严谨，误调用越少。
- **mcp marketplace**：以目录文件（$schema + items）描述一组可用 server 的清单格式，客户端可一键发现多个 server。为什么重要：它让"装工具"从手工配置变成目录化发现。

## 7. 专业背书

- MCP 由 Anthropic 于 2024 年 11 月以 MIT 许可开源，随后被 OpenAI、Google、Microsoft 等生态广泛采用，是"模型与工具解耦"方向的事实标准。本仓 server 结构遵循其规范。
- server 的目录与实现参照 Anthropic 官方 MCP SDK 及官方 servers 参考实现（Python / TypeScript SDK），保证协议兼容。
- 与 `mcp-marketplace` 清单格式对齐，可被支持目录发现的客户端一键接入——这也呼应了 harness "一份资产，多处复用"的设计。
