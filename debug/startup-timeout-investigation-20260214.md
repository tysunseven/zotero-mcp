# Zotero MCP 启动超时排查（2026-02-14）

## 现象
- `codex exec` 多次出现：`mcp: zotero failed: ... timed out after 60 seconds`

## 关键证据
1. 启动脚本原先健康检查在 `pwsh` 下单次约 11 秒：
   - `Test-NetConnection` 约 8-9 秒
   - `Invoke-WebRequest` 约 2-3 秒
2. 在 Windows PowerShell 5.1 下，`Invoke-WebRequest` 存在偶发异常（NullReference），稳定性较差。
3. MCP 配置原先使用 `command = "powershell"`（即 5.1）。

## 根因判断
- 启动超时不是 Zotero API 本身不可用，而是 **MCP 启动链路中使用 Windows PowerShell 5.1 + 重型/不稳定探测命令** 导致启动握手不稳定。

## 已实施修复
1. `C:\Users\28613\.codex\config.toml`
   - `mcp_servers.zotero.command` 从 `powershell` 改为 `C:\Program Files\PowerShell\7\pwsh.exe`
2. `C:\Users\28613\.codex\run-zotero-mcp.ps1`
   - 端口检查改为 `TcpClient`（1.5s）
   - API 检查改为 `HttpClient`（3s）
   - 使用 `127.0.0.1`，避免 `localhost` 解析差异

## 修复后验证
- 连续 5 次端到端 `codex exec + Zotero MCP`：
  - `mcp startup: ready: zotero` 5/5
  - `startup timeout` 0/5
  - 请求成功 5/5（返回 `ok`）
