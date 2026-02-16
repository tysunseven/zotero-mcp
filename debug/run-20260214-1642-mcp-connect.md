# MCP 接入落地记录（2026-02-14）

## 1) 新增启动脚本
- 文件: C:\Users\28613\.codex\run-zotero-mcp.ps1
- 功能: 加载 C:\Users\28613\.codex\zotero-mcp-secrets.env，并启动 zotero-mcp（默认 stdio）
- 自测: `run-zotero-mcp.ps1 version` 输出 `Zotero MCP v0.1.2`

## 2) Codex MCP 配置
- 文件: C:\Users\28613\.codex\config.toml
- 新增段:
  [mcp_servers.zotero]
  command = "powershell"
  args = ["-NoProfile", "-ExecutionPolicy", "Bypass", "-File", "C:\\Users\\28613\\.codex\\run-zotero-mcp.ps1"]

## 3) 握手问题修复
- 现象: Codex 侧 `initialize response` 失败
- 原因: FastMCP banner 污染 stdio
- 修复文件: C:\Users\28613\Documents\GitHub\zotero-mcp\src\zotero_mcp\cli.py
- 修复内容: `mcp.run(transport="stdio", show_banner=False)`

## 4) 结果验证
- `codex mcp list --json` 可见 `zotero` 服务器，状态 enabled
- `codex exec` 日志显示: `mcp: zotero ready`
- 当前剩余问题: provider 流连接到 `https://right.codes/codex/v1/responses` 中断（与 Zotero MCP 启动无关）
