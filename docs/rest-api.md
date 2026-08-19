# REST API

以下示例使用默认地址：

```text
http://127.0.0.1:17994/v1/psbre
```

所有请求都需要 Bearer 令牌。JSON 写请求应设置 `Content-Type: application/json`。

## 端点

| 方法 | 路径 | 权限 |
| --- | --- | --- |
| `GET` | `/version` | `REST.Version.Read` |
| `GET` | `/players` | `REST.Players.Read` |
| `POST` | `/notifications` | `REST.Messages.Write` |
| `GET` | `/pals/{player_identifier}` | `REST.Pals.Read` |
| `POST` | `/pals/{player_identifier}/give` | `REST.Pals.Write` |
| `POST` | `/pals/{player_identifier}/remove` | `REST.Pals.Write` |
| `POST` | `/pals/spawn` | `REST.Pals.Write` |
| `GET` | `/items/{player_identifier}` | `REST.Items.Read` |
| `POST` | `/items/{player_identifier}/give` | `REST.Items.Write` |
| `POST` | `/items/{player_identifier}/remove` | `REST.Items.Write` |
| `POST` | `/commands/execute` | 由具体指令决定 |

`player_identifier` 接受玩家 `UserId` 或 `PlayerUID`。对在线状态有要求的操作会在玩家离线时返回 404。

## 日志

模组每次启动都会在模组目录的 `dlls/Logs/` 创建一个新的通用日志文件：

```text
Logs/
├─ <启动时间戳>.log
└─ RESTAPI/
   └─ <启动时间戳>.log
```

通用日志不可关闭，包含启动开始、Unreal 初始化、每个功能模块的加载结果、模块卸载和关键错误。REST API 请求详细记录仍单独写入 `Logs/RESTAPI/`，包含请求方法、路径、来源 IP、HTTP 状态和脱敏后的请求/响应摘要。

通用日志不记录 Bearer 令牌；提交问题前仍应检查并删除服务器地址、玩家标识、IP 和其他私人数据。

## 通用响应

成功：

```json
{
  "ok": true,
  "data": {}
}
```

失败：

```json
{
  "ok": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "错误说明"
  }
}
```

部分业务失败还会带有 `data`，调用方不能只依据是否存在 `data` 判断成功。

## 玩家列表

`GET /players` 返回 `Meta` 和 `Players`。玩家字段包括 `Name`、`IP`、`PlayerUID`、`UserId`、`GuildName`、`GuildUUID`、`Status`、`WorldLocation` 和 `MapLocation`。

- `Meta.PlayerCount`：返回的账号数。
- `Meta.OnlineCount`：当前在线数。
- 离线玩家的 IP 为空，坐标为零值。
- `WorldLocation` 是游戏世界坐标。
- `MapLocation` 是适合显示在世界地图上的归一化坐标；无法转换时静默返回零值。

## 通知

`POST /notifications` 支持四种 `type`：

```json
{
  "type": "chat",
  "sender": "Server",
  "message": "服务器将在 10 分钟后重启"
}
```

```json
{
  "type": "private_chat",
  "user_id": "steam_76561198000000000",
  "sender": "Server",
  "message": "只有该玩家能看到"
}
```

```json
{
  "type": "floating",
  "message": "活动已经开始",
  "priority": "normal"
}
```

```json
{
  "type": "warning",
  "message": "服务器即将重启"
}
```

`floating.priority` 可选 `normal`、`important`、`very_important`，默认 `normal`。

## 玩家帕鲁

### 读取

`GET /pals/{player_identifier}` 返回队伍和终端中的帕鲁。帕鲁 ID 与等级以 `SaveParameter.CharacterID` 和 `SaveParameter.Level` 为准。容器页、槽位和队伍位置是对象同级的定位信息。

返回字段可能随游戏版本增加，调用方应按字段名读取，并保留未知字段。

### 给予

`POST /pals/{player_identifier}/give`：

```json
{
  "data": {
    "SaveParameter": {
      "CharacterID": "SheepBall",
      "Level": 20,
      "NickName": "测试帕鲁",
      "Talent_HP": 100
    }
  },
  "count": 1
}
```

`data.SaveParameter.CharacterID` 和 `data.SaveParameter.Level` 必填。`count` 默认为 1，范围为 1 至 100。其他字段为递归覆盖值；未提供的字段由游戏生成。每只新帕鲁都会获得独立实例 ID。

### 扣除

`POST /pals/{player_identifier}/remove`：

```json
{
  "data": {
    "SaveParameter": {
      "CharacterID": "SheepBall",
      "Level": 20,
      "Talent_HP": 100
    }
  },
  "count": 1
}
```

`data.SaveParameter.CharacterID` 必填，其他提交字段全部参与递归子集匹配。数组按完整数组匹配。匹配到多个帕鲁时，先按终端槽位从近到远删除，最后处理队伍槽位。

成功响应中的 `Removed` 数组包含每个删除位置和删除前的完整 `Pal` 对象。迁移系统必须以此响应中的 `Pal` 为准，不能用请求前快照替代。插件也会在 `dlls/Pals/Deleted/<玩家 UserId>/` 保存独立留证文件。

## 世界生成帕鲁

`POST /pals/spawn`：

```json
{
  "data": {
    "SaveParameter": {
      "CharacterID": "SheepBall",
      "Level": 20,
      "PassiveSkillList": ["Legend"]
    }
  },
  "count": 1,
  "position": {
    "x": 1000.0,
    "y": 2000.0,
    "z": 300.0
  },
  "rotation": {
    "pitch": 0.0,
    "yaw": 90.0,
    "roll": 0.0
  },
  "Snap": true
}
```

`CharacterID`、`Level` 和 `position.x/y/z` 必填。`rotation` 可省略，`Snap` 默认 `true`，`count` 范围为 1 至 100。响应可能表示请求已接受但实体仍在后续游戏帧完成初始化，调用方应检查 `Complete`、`SpawnComplete`、`AIComplete` 和 `RemainingError`。

## 玩家物品

`GET /items/{player_identifier}` 返回各库存容器和槽位。字段包括容器、槽位、物品 ID、堆叠数量及该物品可读取的附加数据。

给予和扣除使用相同的基础请求：

```json
{
  "item_id": "Wood",
  "count": 100
}
```

`count` 默认为 1。扣除前会统计所有库存容器；数量不足时不修改库存，返回 HTTP 409：

```json
{
  "ok": false,
  "error": {
    "code": "INSUFFICIENT_ITEMS",
    "message": "Not enough matching items were found"
  },
  "data": {
    "Success": false,
    "ItemId": "Wood",
    "RequestedCount": 50,
    "BeforeCount": 12,
    "OwnedCount": 12,
    "RemovedCount": 0
  }
}
```

`data.OwnedCount` 是请求执行时玩家实际拥有的总数。

## 指令入口

`POST /commands/execute`：

```json
{
  "command": "system.version",
  "arguments": {}
}
```

当前用户可用指令包括：

| 指令 | 权限 | 说明 |
| --- | --- | --- |
| `system.version` | `REST.Version.Read` | 读取版本。 |
| `system.game_thread` | `REST.Diagnostics.Read` | 执行只读线程诊断。 |
| `player.list` | `REST.Players.Read` | 读取玩家列表。 |
| `player.kick` | `REST.Players.Kick` | 踢出在线玩家。 |
| `notifications.send` | `REST.Messages.Write` | 发送通知，参数与通知端点相同。 |

踢人示例：

```json
{
  "command": "player.kick",
  "arguments": {
    "user_id": "steam_76561198000000000",
    "reason": "管理员踢出"
  }
}
```
