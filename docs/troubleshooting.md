# 故障排查

## 启动后没有加载

1. 确认独立模组包已安装到 `ue4ss/Mods/PalServerBridgeRE/`。
2. 确认模组根目录存在 `enabled.txt`，并且 `dlls/main.dll` 存在。
3. 确认使用 Windows x64 专用服务器和兼容的 UE4SS。
4. 查看 UE4SS 日志中的第一条 PalServerBridgeRE 错误。

本仓库只提供文档，不提供 DLL 或本地图形测试工具。不要尝试从 GitHub 文档仓库寻找模组文件。

## REST 服务未启动

如果日志提示没有加载已启用令牌：

1. 确认 `RESTConfig.json` 的 `enabled` 为 `true`。
2. 确认至少有一个 `Tokens/*.json` 文件，并且令牌文件的 `enabled` 为 `true`。
3. 确认令牌长度和 JSON 格式有效。
4. 检查全局和令牌的 `allowed_ips`。
5. 重启服务器。

## HTTP 状态码

| 状态 | 常见原因 | 处理 |
| --- | --- | --- |
| 401 | 缺少 Bearer 令牌或令牌错误 | 从令牌文件重新读取，不要手工截断。 |
| 403 | 来源 IP 未放行或权限不足 | 同时检查全局和令牌 `allowed_ips`，再检查权限。 |
| 404 | 路径错误、指令不存在或目标玩家不存在 | 核对 `base_path`、端点和玩家标识。 |
| 409 | 物品数量不足 | 查看响应中的 `data.OwnedCount`。 |
| 503 | 当前游戏状态不满足操作，或游戏更新导致相应能力暂不可用 | 停止重复写请求，保存日志并确认版本兼容性。 |
| 504 | 游戏处理未在配置时间内响应 | 检查服务器是否卡顿；不要通过无限增大超时掩盖问题。 |

错误响应中的 `error.code` 比 HTTP 状态更具体，应优先记录该值。

## 手工测试

本仓库不提供 Python GUI 或其他可执行测试工具。可以使用 PowerShell 自带的 HTTP 客户端：

```powershell
$headers = @{ Authorization = 'Bearer 你的令牌' }
Invoke-RestMethod `
  -Uri 'http://127.0.0.1:17994/v1/psbre/version' `
  -Headers $headers
```

先测试 `/version`，再测试 `system.game_thread`，最后才测试踢人、物品或帕鲁写操作。

## 提交问题

请提供：

- PalServerBridgeRE 版本。
- Palworld 专用服务器版本。
- UE4SS 版本。
- 请求方法和路径。
- HTTP 状态、`error.code` 和脱敏后的响应。
- 问题发生时间附近的脱敏日志。

提交前必须删除：

- `Authorization` 和所有令牌内容。
- 服务器公网地址。
- 玩家 IP、Steam/EOS 标识和昵称。
- 不希望公开的帕鲁、物品、存档和管理数据。

不要上传完整令牌文件、真实 `RESTConfig.json`、整个日志目录或 `Pals/Deleted` 留证目录。
