# 配置说明

本仓库只展示配置格式，不提供可直接加载的配置文件。实际文件位于独立模组目录：

```text
<服务器目录>/Pal/Binaries/Win64/ue4ss/Mods/PalServerBridgeRE/dlls/RESTAPI/
```

目录结构：

```text
RESTAPI/
├─ RESTConfig.json
└─ Tokens/
   └─ <你的令牌>.json
```

首次加载时，如果 `RESTConfig.json` 不存在，模组会生成默认关闭的配置。修改配置或令牌后应重启服务器。

## RESTConfig.json

```json
{
  "enabled": true,
  "bind_address": "127.0.0.1",
  "port": 17994,
  "base_path": "/v1/psbre",
  "max_body_bytes": 65536,
  "io_timeout_seconds": 10,
  "game_thread_timeout_ms": 2000,
  "mirror_requests_to_console": false,
  "allowed_ips": ["127.0.0.1", "::1"]
}
```

| 字段 | 说明 |
| --- | --- |
| `enabled` | 是否启动 REST 服务。默认 `false`。 |
| `bind_address` | 监听地址。只在本机使用时保持 `127.0.0.1`。 |
| `port` | 监听端口，默认 `17994`。 |
| `base_path` | API 基础路径，默认 `/v1/psbre`。 |
| `max_body_bytes` | 单次请求正文大小上限。 |
| `io_timeout_seconds` | HTTP 读写超时。 |
| `game_thread_timeout_ms` | 需要等待游戏处理的请求超时。不要把它当作卡服问题的修复项。 |
| `mirror_requests_to_console` | 是否额外向 UE4SS 控制台输出请求摘要，不影响文件日志。 |
| `allowed_ips` | 全局来源 IP 白名单。支持精确 IPv4、精确 IPv6 和 `*`。 |

## 创建令牌

令牌文件应放在 `RESTAPI/Tokens/`，并使用随机生成的长令牌。下面的 PowerShell 片段只作为文档示例，不会由本仓库提供脚本文件：

```powershell
$tokenDirectory = 'C:\path\to\PalServerBridgeRE\dlls\RESTAPI\Tokens'
$tokenPath = Join-Path $tokenDirectory 'AdminPanel.json'
[IO.Directory]::CreateDirectory($tokenDirectory) | Out-Null
$bytes = New-Object byte[] 48
$rng = [Security.Cryptography.RandomNumberGenerator]::Create()
try { $rng.GetBytes($bytes) } finally { $rng.Dispose() }
$secret = [Convert]::ToBase64String($bytes).TrimEnd('=').Replace('+', '-').Replace('/', '_')
$value = [ordered]@{
  enabled = $true
  name = 'AdminPanel'
  token = $secret
  permissions = @('REST.*')
  allowed_ips = @('127.0.0.1', '::1')
}
$json = $value | ConvertTo-Json -Depth 8
$utf8WithoutBom = New-Object System.Text.UTF8Encoding($false)
[IO.File]::WriteAllText($tokenPath, $json + [Environment]::NewLine, $utf8WithoutBom)
```

不要把令牌提交到 GitHub、发送到聊天群或写入日志。每个后台、机器人或自动化服务应使用独立令牌，便于单独撤销。

令牌文件格式：

```json
{
  "enabled": true,
  "name": "ReadOnlyPanel",
  "token": "由安全随机源生成的令牌",
  "permissions": [
    "REST.Version.Read",
    "REST.Players.Read",
    "REST.Pals.Read",
    "REST.Items.Read"
  ],
  "allowed_ips": [
    "127.0.0.1"
  ]
}
```

请求使用：

```text
Authorization: Bearer <令牌>
```

## 权限

| 权限 | 能力 |
| --- | --- |
| `REST.Version.Read` | 读取插件版本。 |
| `REST.Diagnostics.Read` | 执行只读诊断。 |
| `REST.Players.Read` | 读取玩家列表。 |
| `REST.Players.Kick` | 踢出在线玩家。 |
| `REST.Messages.Write` | 发送通知。 |
| `REST.Pals.Read` | 读取玩家帕鲁。 |
| `REST.Pals.Write` | 给予、扣除或生成帕鲁。 |
| `REST.Items.Read` | 读取玩家物品。 |
| `REST.Items.Write` | 给予或扣除物品。 |

权限支持精确值、末尾通配符（如 `REST.*`）和完整通配符 `*`。生产环境应使用最小权限。

每次请求必须同时通过全局 IP 白名单、令牌匹配、令牌 IP 白名单和权限检查。配置中不需要额外的指令白名单。
