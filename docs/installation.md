# 安装说明

本仓库只提供文档。PalServerBridgeRE 模组文件由维护者在其他发布页面单独提供，请不要把本仓库当作模组安装包。

## 前置条件

- Windows x64 Palworld 专用服务器。
- 与当前服务器版本兼容且能够加载 C++ 模组的 UE4SS。
- 安装目录的写入权限。

建议先在测试服验证新版本，并在操作前备份服务器存档和现有模组配置。

## 首次安装

1. 从维护者的独立发布页面下载 PalServerBridgeRE 模组包。
2. 完全停止 Palworld 专用服务器。
3. 将独立发布包中的 `PalServerBridgeRE` 文件夹复制到：

   ```text
   <服务器目录>/Pal/Binaries/Win64/ue4ss/Mods/
   ```

4. 最终路径应为：

   ```text
   <服务器目录>/Pal/Binaries/Win64/ue4ss/Mods/PalServerBridgeRE/enabled.txt
   <服务器目录>/Pal/Binaries/Win64/ue4ss/Mods/PalServerBridgeRE/dlls/main.dll
   ```

5. 启动服务器，检查 UE4SS 日志，确认模组已加载且没有初始化错误。

只安装 `main.dll` 而遗漏 `enabled.txt`，可能导致 UE4SS 不加载模组。不要把本仓库的 Markdown 文件复制到服务器 Mods 目录。

## 配置位置

REST 配置位于独立模组目录的：

```text
<服务器目录>/Pal/Binaries/Win64/ue4ss/Mods/PalServerBridgeRE/dlls/RESTAPI/
```

首次加载时，如果 `RESTConfig.json` 不存在，模组会生成一份默认关闭的配置。令牌文件放在该目录下的 `Tokens/` 子目录。配置格式见 [配置说明](configuration.md)。

## 更新

1. 停止服务器。
2. 备份以下运行时数据：

   ```text
   PalServerBridgeRE/dlls/RESTAPI/RESTConfig.json
   PalServerBridgeRE/dlls/RESTAPI/Tokens/
   PalServerBridgeRE/dlls/Logs/
   PalServerBridgeRE/dlls/Pals/
   ```

3. 用独立发布页面的新模组包替换旧的 `PalServerBridgeRE` 文件夹。
4. 不要用文档中的示例配置覆盖真实的 `RESTConfig.json` 和令牌文件。
5. 启动服务器，先测试 `/version` 和线程诊断，再恢复外部后台的写操作。

每次启动会在 `PalServerBridgeRE/dlls/Logs/` 创建一个新的通用日志文件，文件名包含启动时间戳。更新或排障时应保留该文件以及 `Logs/RESTAPI/` 下对应的请求日志。

## 卸载

1. 停止服务器。
2. 备份需要保留的日志、令牌和删除留证。
3. 删除服务器 Mods 目录中的 `PalServerBridgeRE` 文件夹。
4. 再次启动服务器，并确认没有外部服务继续请求原 REST 端口。

## 安装验证

安装完成后，使用 [REST API](rest-api.md) 中的版本请求验证服务：

```powershell
$headers = @{ Authorization = 'Bearer 你的令牌' }
Invoke-RestMethod `
  -Uri 'http://127.0.0.1:17994/v1/psbre/version' `
  -Headers $headers
```

如果请求失败，请先阅读 [故障排查](troubleshooting.md)，不要反复重启生产服或重复发送写请求。
