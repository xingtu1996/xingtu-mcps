# XingTu MCPs · MCP 服务集合

> 可部署的服务参考实现，供 AI Agent 工具调用。

![MIT](https://img.shields.io/badge/license-MIT-green.svg)

## 这是什么

`xingtu-mcps` 是行途开源矩阵的 **MCP 服务资产仓**。收录可独立部署的服务参考实现（长连接机器人 / MCP server 骨架），为 AI Agent 提供真实可用的工具能力。

## Servers 清单

| Server | 说明 | 状态 |
|--------|------|:---:|
| servers/feishu-bot | 飞书长连接机器人参考实现（命令 + LLM + 上下文）| ✅ 可运行 |

> 逐步填充中：将按"真实服务 → MCP 封装"路线持续新增。

## 标准

- 服务遵循 MCP / 飞书开放平台等标准协议
- 每个 server 独立 README（配置 / 运行 / 工具列表）
- **密钥一律环境变量**，代码不落盘

## 目录结构

```
servers/
  feishu-bot/   # 飞书长连接机器人（参考实现 + MCP 适配方向）
```

## 许可证

MIT License
