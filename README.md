# PalServerBridgeRE 用户文档

这里是 PalServerBridgeRE 的公开用户文档仓库，包含安装、REST 配置、接口说明、安全建议、故障排查和问题反馈模板。

本仓库不包含 PalServerBridgeRE 模组、DLL、源码、SDK、构建文件、运行时数据或测试工具。模组文件请从维护者提供的独立发布页面获取；下载后按照本文档安装和配置。

## 文档

- [安装说明](docs/installation.md)
- [配置说明](docs/configuration.md)
- [REST API](docs/rest-api.md)
- [故障排查](docs/troubleshooting.md)
- [安全说明](SECURITY.md)

## 快速开始

1. 从独立发布页面下载与 Palworld 专用服务器版本匹配的 PalServerBridgeRE 模组。
2. 按 [安装说明](docs/installation.md) 将模组放入 UE4SS 的 Mods 目录。
3. 按 [配置说明](docs/configuration.md) 创建 REST 配置和令牌。
4. 重启服务器后先调用 `/version`，确认服务正常，再开放写操作。

REST API 默认关闭，只建议绑定本机或受防火墙保护的内网地址。服务本身不提供 TLS，不应直接暴露到公网。

## 版本

当前文档对应模组版本：`0.5.0`。游戏或 UE4SS 更新后，先在测试服验证，再启用帕鲁、物品等写接口。

## 反馈

提交 Issue 前请删除令牌、服务器公网地址、玩家 IP、Steam/EOS 标识、昵称和私人游戏数据。安全漏洞请使用 GitHub 私密安全报告，不要公开发布。
