# xingtu-mcps · 生产级 MCP 服务集合

> 行途开源矩阵的 MCP 服务资产仓：收录可独立部署、可被任意 MCP 客户端复用的 MCP server。协议先行，不绑死某个 Agent 宿主——换客户端不用重写。

## 这是什么

Agent 只靠文本提示只能「说」，MCP（Model Context Protocol）让它「做」。本仓把散落在真实工程里的可复用工具能力收敛成标准 MCP server：任何支持 MCP 的宿主（Claude Code、Claude Desktop、Cursor、CodeBuddy 等）都能通过同一种方式发现和调用。

## 核心价值

1. **协议先行**：所有 server 走 MCP 规范（JSON-RPC 2.0 + 工具 schema），不绑死某个 Agent 宿主，换客户端不用重写。
2. **标准化起步**：每个 server 从 `servers/_TEMPLATE` 起步，统一目录与 README 结构（安装 / 配置 / 工具列表），把「起一个 server」的成本压到最低。
3. **可独立部署**：每个 server 独立运行，stdio 或 HTTP(S) 传输都可挂载，本地、远端皆可。
4. **可审计**：工具带参数 schema 声明，调用有明确的入参 / 出参边界，Agent 调用行为可记录、可回放。
5. **与 harness 打通**：作为子模块被 [xingtu-harness](https://github.com/xingtu1996/xingtu-harness) 聚合，场景安装时按需拉取，而不是全量堆进 Agent 环境。

## 目录结构

```
xingtu-mcps/
├── servers/
│   └── feishu-bot/          # 飞书长连接机器人参考实现
├── SELF.md                  # 仓内专业自述（定位 / 边界 / 概念）
├── CONTRIBUTING.md
└── LICENSE
```

## 当前服务

### feishu-bot · 飞书长连接机器人参考实现

`servers/feishu-bot/` 是一个可运行的飞书 WebSocket 长连接机器人骨架：接收群聊消息、按命令回复、带 LLM 对话与上下文管理。所有密钥（飞书 AppID / Secret、LLM Key）只从环境变量读取，代码与配置不落盘。

| 文件 | 说明 |
|---|---|
| `config.py` | 配置（密钥全走环境变量，无硬编码） |
| `bot_longconn.py` | 主入口：长连接 + 消息分发 |
| `commands.py` | 快捷命令处理（/help /status /tasks 等） |
| `context.py` | 群聊上下文保留与过期 |
| `llm.py` | LLM 对话封装（OpenAI 兼容格式） |
| `debug_ws.py` | 长连接连通性调试脚本 |

## 与相邻仓的边界

| 对比项 | xingtu-mcps | xingtu-tools / xingtu-cli | xingtu-skills |
|---|---|---|---|
| 能力形态 | 独立运行的 server 进程 | 本机脚本 / 命令行 | 文档式技能（SKILL.md + 辅助脚本） |
| 调用方式 | Agent 经 MCP 协议调工具（带 schema） | 人或 Agent 直接执行命令 | Agent 按需读取技能说明后自行执行 |
| 边界 | 工具即服务，有生命周期、可远端部署 | 无协议，进程内跑在宿主机器上 | 教 Agent「怎么想怎么做」，不产副作用工具 |

一句话：**skills 给方法，tools/cli 给本机命令，mcps 给跨宿主的服务化工具**。三者不重叠，可同时挂载。

## 新增 server

1. `cp -r servers/_TEMPLATE servers/<name>`。
2. 实现 tools / resources / prompts，写独立 README（安装 / 配置 / 工具列表）。
3. 质量门禁：每个工具可跑通一次真实调用、参数 schema 校验通过；有副作用的工具必须有幂等或保护设计；README 未列出的工具不作为发布资产。

## 与聚合仓的关系

- 运行时镜像：[harness-kit/mcps/](https://github.com/xingtu1996/harness-kit/tree/main/mcps)（harness 生成器聚合同一份服务集合）。
- 技能编排：[xingtu-skills](https://github.com/xingtu1996/xingtu-skills)（工作流编排技能库）。
- CLI 工具：[xingtu-tools](https://github.com/xingtu1996/xingtu-tools)（本机工程脚本集）。

## 关于作者 · 行途 XingTu

- 公众号「行途技术手记」：微信搜索关注，看 AI 工程化落地实战。
- GitHub：[@xingtu1996](https://github.com/xingtu1996)
- 方法论旗舰仓：[xingtu-ai-engineering](https://github.com/xingtu1996/xingtu-ai-engineering)

## License

MIT License。
